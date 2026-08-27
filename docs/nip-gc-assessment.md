# NIP-GC (Geocaching Events) — relevance assessment

Owner flagged a draft NIP-GC spec + reference implementation
(`https://gitlab.com/chad.curtis/treasures.git`) as potentially relevant to ip-layer/Twin Binding and
agent-phone/flagship-device presence work. Investigated directly rather than assuming a connection.

## What NIP-GC actually is

A geocaching-specific Nostr NIP: `37516` cache listings (geohash, difficulty/terrain/size/hint),
`7516` found logs, `1111` NIP-22 threaded comments, `37517` curated cache collections, and the
interesting piece — `7517` **Verification Event**: proof a finder was physically at a cache location.

## How `7517` proves physical presence — read from real code

Cloned and read the reference repo (`src/utils/verification/events.ts`), not just the spec text. The
mechanism: each geocache has a dedicated Nostr keypair; the **private key (nsec)** is physically
printed/QR-coded at the cache site. A finder who reaches the site scans the QR code, gets the nsec,
and signs a kind-`7517` event with it (`createVerificationEvent`, using `NSecSigner` from
`nostr-tools`/`@nostrify/nostrify`) attesting `"Geocache verification for <finder-npub>"`. That signed
event gets embedded inside the finder's kind-`7516` found-log. Anyone can verify the Schnorr signature
without trusting the finder's word — standard, real, working code (`verifyEvent` from `nostr-tools`).

**Trust model, stated plainly: possession of a static secret, not continuous presence.** The nsec is a
single unchanging value tied to the cache. Once photographed/copied by anyone at the site, it can be
signed with from anywhere, at any later time — it proves "someone extracted this key from the site at
some point," not "the signer was physically there at signing time." No timestamp binding, no RSSI,
no liveness check. That's an appropriate tradeoff for a geocaching game (low stakes, honor-system
adjacent), but it's a materially weaker guarantee than what this ecosystem's own Witness-firmware
already aims for.

## Compared to OSOVM's Witness-firmware

Per `~/OSOVM/witness_bridge/README.md`: Witness-firmware (LoRa DePIN) attestation is "physics-proof
payload hash + Ed25519 signature," submitted via `submit_witness(attestation_id, witness_sensor_id,
signature)` — a live hardware sensor signs a fresh payload per submission, not a static shared secret.
(Note: `witness_bridge/src/lib.rs` is currently only 20 lines — this is a stub/skeleton locally, not a
mature implementation yet, so the comparison is spec-vs-spec more than code-vs-code.) Even as a stub,
the intended design is stronger than NIP-GC's mechanism: per-event fresh signing from real hardware
beats a single static leaked-once-compromised-forever key.

## Answers

**1. Should ip-layer or the flagship device adopt NIP-GC's verification-key-at-location pattern?**
Not the security mechanism — it's weaker than what Witness-firmware already targets, and adopting a
static-shared-secret model would be a downgrade. But the **event-shape pattern is worth borrowing**:
a small, separately-signed "presence/verification" event embedded inside a tag of a larger claim event
is a clean, reusable technique. If the flagship device ever needs a "prove I was physically at this
device/location" claim (distinct from Twin Binding's "this sim IS my digital twin" claim), the right
move is a similar embedded-verification-event shape, but signed by a per-session or hardware-derived
key (closer to Witness-firmware's model) rather than a static QR-printed nsec. Not building this now —
no concrete flagship-device presence requirement exists yet to build it against; noting the pattern
for when one does.

**2. Is the reference repo worth learning from?** Yes, concretely: it's a real, tested, production-
shaped app (React + Capacitor, `nostr-tools`/`@nostrify/nostrify`, `src/tests/verification-flow.test.ts`
exists and is a good template for how to test a signed-event-embedded-in-event flow), and the
`naddr` construction/parsing helpers (`geocacheToNaddr`/`parseNaddr`) are a clean reference for how
ip-layer's own `ip_root`/Creation Receipt cross-referencing could be tested, if a shared Nostr publish
library (build-order step 5) ends up needing similar addressable-event helpers. Geohash/location
indexing itself (`src/utils/geo.ts`, `coordinates.ts`) is standard and not something ip-layer needs.

**3. Is this actually relevant to agent-phone?** No — not forcing it. Agent-phone is telecom
(identity/discovery/calling), not location/presence. Geocaching's difficulty/terrain/size/hint/mission
fields have no analog there. The only genuine touchpoint is the flagship *device* (physical
board+screen), and only as a future-reference pattern, not a dependency or adoption today.

## Conclusion

No schema change made. This is an evaluated-and-shelved reference, not an adopted pattern — recording
the assessment so it doesn't need re-investigating later, and flagging the embedded-verification-event
shape as the one idea worth reaching for if/when the flagship device gets a real presence-proof
requirement.
