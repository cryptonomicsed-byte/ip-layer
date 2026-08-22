# Attestation — event schema (draft v1)

Vouch/challenge/confirm on a Creation Receipt (or an IP Root) by another agent, human, or Zàngbétò.
Nostr **regular event**, kind `1902`.

## Event shape

```json
{
  "id": "<32-byte-hex>",
  "pubkey": "<attestor npub hex>",
  "created_at": 1755800200,
  "kind": 1902,
  "tags": [
    ["e", "<creation-receipt-or-ip-root-event-id>"], // subject being attested to
    ["p", "<subject-pubkey>"],                        // subject's identity, for easy filtering
    ["stance", "vouch"],                              // one of: vouch | challenge | confirm
    ["reason", "<short-code-or-free-text>"],           // required for challenge, optional otherwise
    ["L", "attestation"],                              // NIP-32 label namespace
    ["l", "vouch", "attestation"],                      // mirrors `stance`, machine-filterable
    ["weight", "<stake-or-reputation-weight>"]         // optional, e.g. Zàngbétò stake amount
  ],
  "content": "{\"notes\":\"...\"}",
  "sig": "<schnorr-sig>"
}
```

## Field notes

- **kind 1902**: custom regular event, adjacent to Creation Receipt's kind `1901`. Attestations are
  themselves immutable — a changed mind is a *new* attestation event, not an edit.
- **`stance` values**:
  - `vouch` — attestor affirms the claim (authorship, contribution, quality) is credible.
  - `challenge` — attestor disputes the claim; `reason` is required. Does not itself revoke or alter
    the receipt (receipts are immutable) — it's a public counter-claim an indexer/UI can surface
    alongside the original.
  - `confirm` — narrower than `vouch`: attestor has directly verified a specific fact (e.g. "I
    co-created this," "I ran this simulation and got the same hash"), not just a general credibility
    stance.
- **Zàngbétò as attestor**: no special-cased fields — Zàngbétò attests like any other pubkey, using
  `weight` to carry its stake/slashing-relevant amount if the ecosystem's slashing logic needs it.
  Keeps this schema attestor-agnostic (human, peer agent, or Zàngbétò all use the same shape).
- **No aggregation logic here**: this schema only defines the individual attestation event. Computing
  a rolled-up trust/reputation score from a set of attestations is downstream indexer/consumer logic,
  not part of the wire format.

## Relationship to Creation Receipt's `attest` tag

A Creation Receipt (kind `1901`) has a repeatable `attest` tag that an indexer backfills with
attestation event ids as they arrive (see `creation_receipt.md`). This event is the actual attestation
being pointed to — the receipt-side tag is just a convenience mirror, this event is the source of
truth.

## Open questions (ask before implementing)

1. Does a `challenge` ever need teeth beyond "publicly visible dispute" in v1 — e.g. should enough
   `challenge` weight auto-suppress a receipt from default UI surfacing (Vantage), or is that purely a
   v2/UI-layer decision with zero protocol-level effect?
2. Is Zàngbétò's `weight` value read from an on-chain stake record, or self-declared in the event
   content — i.e. does verifying an attestation's weight require an external lookup, or is it fully
   self-contained in the Nostr event?
