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
    ["L", "license"],                        // NIP-32 label namespace
    ["l", "cc-by-4.0", "license"],           // NIP-32 label value; overrides ip_root default if present
    ["split", "<npub-hex>", "<basis-points>"], // repeatable: payment split entries, sum <= 10000
    ["attest", "<attestation-event-id>"],    // repeatable, populated as attestations arrive
    ["osovm_op", "RECEIPT", "<osovm-receipt-id>"], // present when this event IS the Nostr
                                                    // publication of an OSOVM RECEIPT opcode emission
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
- **`L`/`l` tags**: real NIP-32 labels, not a bespoke `license` tag — matches the blueprint's stated
  audit flow ("OSOVM RECEIPT opcode + major actions emit structured receipts → published as Nostr
  events with NIP-32 labels"). Other label namespaces (e.g. content-category, jurisdiction) can be
  added the same way without a schema change.
- **`e` tag with "root" marker**: NIP-10-style thread reference, reused here to express derivative/
  remix lineage (receipt B built on receipt A).

## Relationship to OSOVM receipts — unified, not parallel

Per the endgame blueprint's audit flow, this is **not** a second receipt system bolted onto OSOVM's
RECEIPT opcode — it's the same artifact viewed from two audiences. When an agent action runs through
OSOVM and OSOVM's RECEIPT opcode fires, and that action is IP-significant, the resulting kind `1901`
event **is** the Nostr publication of that opcode emission (`osovm_op` tag carries the opcode name and
OSOVM's internal receipt id for cross-reference/replay). For actions that never touch OSOVM at all
(e.g. an external-framework agent, or a human creator with no OSOVM runtime), the `osovm_op` tag is
simply absent and the event stands alone as a plain Creation Receipt. One schema serves both cases —
there should never be two separate receipt records for the same action.

## Open questions (ask before implementing)

1. Exact authorization rule for "which pubkeys may emit receipts under a given IP Root" — needs a
   real answer before an indexer can safely validate receipts (currently just "root's own pubkey" is
   safe; multi-agent/owner-delegated signing needs a rule).
2. Should `split` basis-points be validated/enforced anywhere in v1, or purely advisory metadata for
   downstream zap-splitting tools to read? (`split` entries are consumed by NIP-57 zap-splitting
   today; NIP-60/61 Cashu is a future alternative settlement path, same tag shape.)
3. OSOVM's actual RECEIPT opcode payload shape isn't in hand yet — `osovm_op` tag above is a
   placeholder guess at the cross-reference shape. Needs a direct look at OSOVM's opcode definition
   before this is locked, to make sure the unification is real and not just cosmetic.
