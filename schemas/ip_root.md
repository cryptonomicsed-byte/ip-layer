# IP Root — event schema (draft v1)

Birth-time identity event for an IP Root. One per creator (human or agent).
Nostr **parameterized replaceable event**, kind `31900` (NIP-01/NIP-33 addressable range),
so a creator can update default license stance etc. without minting a new identity.

## Event shape

```json
{
  "id": "<32-byte-hex>",
  "pubkey": "<creator/agent npub hex>",
  "created_at": 1755800000,
  "kind": 31900,
  "tags": [
    ["d", "<ip-root-id>"],            // stable addressable id, e.g. same as pubkey or a uuid
    ["owner", "<owner-npub-hex>"],    // optional: distinct from pubkey if agent has a human owner
    ["soul", "<bipon39-derived-pubkey>"],  // optional BIPON39 soul-binding reference
    ["license", "cc-by-4.0"],         // default license stance, overridable per creation
    ["dao", "<wyoming-dao-llc-id>"],  // optional Wyoming DAO LLC membership link
    ["genesis", "<genesis-receipt-event-id>"] // link to birth/genesis receipt if one exists
  ],
  "content": "{\"display_name\":\"...\",\"framework\":\"omo-koda2|external\",\"notes\":\"...\"}",
  "sig": "<schnorr-sig>"
}
```

## Field notes

- **kind 31900**: custom parameterized-replaceable kind. Not yet in the NIP registry — treat as
  provisional pending an actual NIP submission; document the number here as the source of truth
  until then.
- **`d` tag**: the addressable IP Root id. For Omo-Koda2-born agents this can be the agent's own
  Nostr pubkey; for human creators or external-framework agents it's a self-chosen stable string.
- **`owner` tag**: optional. Distinguishes the *acting* identity (pubkey signing events) from the
  *responsible* identity (a human who birthed/deployed an agent). Mirrors the "creator = birth-deployer"
  rule from the constitution's Tier-7 wallet-inheritance definition — reused here for IP attribution,
  not governance.
- **`license` tag**: default license stance (SPDX-style id, or a custom license URI). Applies to all
  Creation Receipts under this root unless overridden per-creation.
- **`dao` tag**: optional. Only present if this IP Root has bound itself to the Wyoming DAO LLC legal
  wrapper. Absence means fully independent — the DAO does not take ownership by default.
- **content**: free-form JSON metadata, non-normative (display name, framework origin, notes).

## Universal-wording compliance

No Yoruba/Orisha terms anywhere in tag names, kind names, or content — per
[[universal-wording-locked-canon]]. "Soul" here is a plain-English term (soul-binding = key
derivation binding), not a translated Orisha concept.

## Open questions (ask before implementing)

1. Exact kind number (31900) is provisional — confirm no collision with other ecosystem NIPs before
   locking it in Omo-Koda2/Vantage.
2. Should `owner` be mandatory for agent-born IP Roots (liability clarity) or stay optional?
