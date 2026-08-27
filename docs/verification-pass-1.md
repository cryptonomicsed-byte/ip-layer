# Verification pass 1 — resolving pre-implementation open questions

Before writing the Omo-Koda2 birth-flow hook (build-order step 3), the schema-drafting stage's own
open questions needed real answers, not assumptions. This doc records what was actually checked.

## 1. Kind-number collision check (31900 / 1901 / 1902)

Checked against the local ecosystem's own NIP usage (OSOVM, Omo-Koda2, Vantage, minipae docs) and the
well-known NIP registry ranges:

- `31900` falls in the NIP-01/NIP-33 parameterized-replaceable range (30000–39999). No local or
  well-known NIP claims 31900.
- `1901` / `1902` fall in the regular-event custom range adjacent to NIP-94 (`1063`). No local or
  well-known NIP claims either number.
- No collision found. These remain **provisional but clear** — still subject to a real NIP submission
  before treating them as permanent, per the schema docs' own caveat, but safe to build against now.

**Status: resolved for v1 purposes.** Formal NIP submission is a separate, non-blocking follow-up.

## 2. OSOVM RECEIPT opcode payload shape — verified against real code

Read directly from `~/OSOVM/src/opcodes.jl` and `~/OSOVM/src/vm_core.jl` (not touched — read-only,
per the standing rule that `oso_vm.jl` and its test file belong to another agent).

Real definition:

```julia
:RECEIPT => 0x1f   # opcode table, opcodes.jl

function op_receipt(state::VMState, args::Dict{Symbol,Any})
    hash_val = String(get(args, :hash, "0x0"))
    verified = length(hash_val) >= 64
    return state, Dict{Symbol,Any}(:receipt => hash_val, :verified => verified)
end
```

And the wrapping `Receipt` struct (`vm_core.jl`):

```julia
struct Receipt
    receipt_id::String   # made via make_receipt_id(block_no, tx_index, instr_index, opcode)
    tx_id::String
    opcode::UInt8
    status::Symbol
    data::Dict
end
```

**Finding: the unification is real but narrower than the placeholder assumed.**

- OSOVM's RECEIPT opcode is a minimal in-VM primitive: input is just `{hash}`, output is
  `{receipt, verified}` where `verified` is purely a length check (`>= 64` chars, i.e. "looks like a
  SHA-256 hex digest"). It does **not** natively carry creator, license, timestamp, or entity-count
  fields — those are Creation Receipt-side additions, not something OSOVM's opcode already produces.
- The `x` tag (content hash) on a Creation Receipt is exactly the same kind of value as `op_receipt`'s
  `hash` argument — so unifying via `osovm_op` is legitimate, not cosmetic: the `<osovm-receipt-id>`
  placeholder should be OSOVM's `receipt_id` (from `make_receipt_id`), and the hash carried in `x`
  should be the same value passed as `args[:hash]` into `op_receipt`.
- Correction needed to `creation_receipt.md`: the `osovm_op` tag should be documented as cross-
  referencing OSOVM's `receipt_id` string (not a separate invented id), and it should be explicit that
  OSOVM's own `verified` flag is a shallow length check — any stronger authenticity claim (e.g. "this
  hash was actually witnessed/quorum-approved") comes from Zàngbétò's separate witness-receipt system
  (`~/OSOVM/src/zangbeto_receipts.jl`, `ZangbetoReceipt`/`ReceiptBundle`, 7/12 quorum), not from
  `op_receipt` itself. A Creation Receipt claiming OSOVM backing should distinguish "opcode emitted a
  receipt" from "Zàngbétò quorum-verified it" — these are different strength claims and both exist in
  the real codebase as separate structures.

**Status: resolved.** `osovm_op` unification is confirmed real; `creation_receipt.md` needs a small
correction (see follow-up below) to reflect the actual (narrower, two-tier) shape rather than the
placeholder guess.

## 3. Zàngbétò attestation `weight` semantics — still open, now with real context

`ZangbetoReceipt`/`ReceiptBundle` in `zangbeto_receipts.jl` shows Zàngbétò already has its own
quorum/stake machinery (`QUORUM_REQUIRED = 7`, `TOTAL_WITNESSES = 12`, `WitnessVote`). This means an
Attestation event's `weight` tag, if Zàngbétò is the attestor, *could* legitimately be a self-declared
mirror of a real on-chain/off-chain stake value Zàngbétò already tracks — not an arbitrary free-text
field. But whether the ip-layer schema should require external verification of that value (i.e. an
indexer must look up Zàngbétò's actual stake record to trust `weight`) or accept it self-declared in
the event is a policy decision, not something resolvable by reading more code.

**Status: still open — genuinely needs the owner's call, flagging as before.**

## Follow-up action taken

`creation_receipt.md`'s `osovm_op` field note updated to reflect the verified (narrower, two-tier)
shape instead of the placeholder guess. See that file's diff in this commit.
