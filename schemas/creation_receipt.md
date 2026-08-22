# Creation Receipt — event schema (draft v1)

Emitted on every significant creative output. Nostr **regular event**, kind `1901`
(custom, adjacent to NIP-94 file-metadata kind `1063` which it references/extends).

## Event shape

```json
{
  "id": "<32-byte-hex>",
  "pubkey": "<creator/agent npub hex, must match or be authorized by an ip_root>",
  "created_at": 1755800100,
  "kind": 1901,
  "tags": [
    ["ip_root", "<ip-root-d-tag-value>"],   // links back to the IP Root (kind 31900)
    ["x", "<sha256-content-hash>"],          // NIP-94 style content hash
    ["url", "<optional-content-location>"],  // optional, off-Nostr storage pointer
    ["m", "<mime-type>"],                    // optional, NIP-94 style
    ["license", "cc-by-4.0"],                // overrides ip_root default if present
    ["split", "<npub-hex>", "<basis-points>"], // repeatable: payment split entries, sum <= 10000
    ["attest", "<attestation-event-id>"],    // repeatable, populated as attestations arrive
    ["osovm_receipt", "<osovm-receipt-id>"], // optional: formal OSOVM receipt cross-link
    ["e", "<parent-creation-receipt-id>", "", "root"] // optional: derivative-work lineage
  ],
  "content": "{\"title\":\"...\",\"description\":\"...\"}",
  "sig": "<schnorr-sig>"
}
```

## Field notes

- **kind 1901**: custom regular (non-replaceable) event — receipts are immutable facts, never
  edited. Provisional pending NIP submission, same caveat as `ip_root` kind 31900.
- **`ip_root` tag**: required. Every receipt must trace back to exactly one IP Root. A relay/indexer
  should reject receipts whose signing pubkey isn't authorized under the referenced root (root's own
  pubkey, or an agent pubkey the root's `owner` has vouched for — exact authorization rule TBD, flag
  as open question below).
- **`x` tag**: the actual provenance anchor — content hash of the creative output, independent of
  where it's hosted. This is the NIP-94 pattern reused for a receipt rather than a raw file-metadata
  event.
- **`split` tags**: repeatable, one per payee. Basis points (1/100 of a percent) so splits sum to
  10000 for 100%. Purely a *declared intent* in v1 — no automated enforcement (explicitly out of
  scope per the brief).
- **`attest` tags**: empty at emission time, populated as separate Attestation events (see below)
  reference this receipt's id — receipts stay immutable, so attestation *linkage* is tracked by having
  attestation events point at the receipt, and this tag is a convenience mirror an indexer can
  backfill, not something the original signer edits.
- **`e` tag with "root" marker**: NIP-10-style thread reference, reused here to express derivative/
  remix lineage (receipt B built on receipt A).

## Relationship to OSOVM receipts

`osovm_receipt` is an optional cross-link only — this schema does not require OSOVM. When an agent's
output is significant enough to also warrant a formal OSOVM receipt, that receipt's id gets attached
here rather than duplicating OSOVM's receipt fields into this event.

## Open questions (ask before implementing)

1. Exact authorization rule for "which pubkeys may emit receipts under a given IP Root" — needs a
   real answer before an indexer can safely validate receipts (currently just "root's own pubkey" is
   safe; multi-agent/owner-delegated signing needs a rule).
2. Should `split` basis-points be validated/enforced anywhere in v1, or purely advisory metadata for
   downstream zap-splitting tools to read?
