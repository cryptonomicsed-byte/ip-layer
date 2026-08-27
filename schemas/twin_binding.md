# Twin Binding — event schema (draft v1)

Covers 1:1 twin ownership (VeilSim digital-twin concept: a simulated counterpart of a physical
agent/device) — this was flagged as a gap: the schema set only covered content-creation receipts,
not "this simulation IS the IP-bearing twin of that physical thing."

## Why this isn't just another Creation Receipt

A VeilSim twin isn't a single creative output — it's an ongoing simulated entity that produces a
stream of state over time (per real code: `ZangbetoReceipt` in `~/OSOVM/src/zangbeto_receipts.jl`
already carries `sim_id`, `veil_ids::Vector{Int}`, `entity_count`, `trajectory_hash` for veil
executions). Two separate needs:

1. **The binding itself** — a one-time (or rarely-updated) claim: "this `sim_id` IS the digital twin
   of this IP Root." This is identity/ownership, not a creative event. → new kind `1903`.
2. **Ongoing twin-state snapshots** — ordinary Creation Receipts (kind `1901`), reusing the *existing*
   `osovm_op` unification: when a veil execution produces a `ZangbetoReceipt`, its `trajectory_hash`
   becomes the Creation Receipt's `x` tag, and a new `twin` tag (see below) marks it as a twin-state
   snapshot rather than a generic creation. No new receipt-emission schema needed here — the existing
   Creation Receipt already fits, it just needed one more tag.

## New event: Twin Binding, kind `1903`

Parameterized replaceable (same range as IP Root, `31900`-adjacent reasoning doesn't apply here since
this isn't addressable by a stable `d` id the same way — kept as a plain regular event, immutable,
same as Creation Receipt/Attestation; a *re-binding* to a different sim_id is a new event, old one
stands as history).

```json
{
  "id": "<32-byte-hex>",
  "pubkey": "<IP Root owner's npub hex>",
  "created_at": 1755800300,
  "kind": 1903,
  "tags": [
    ["ip_root", "<ip-root-d-tag-value>"],     // the physical agent/device's IP Root
    ["sim_id", "<veilsim-sim-id>"],            // matches ZangbetoReceipt.sim_id
    ["twin_kind", "veilsim-1to1"],             // extensible: other twin types later use a different value
    ["fidelity", "<free-text-or-score>"],      // optional, self-declared claim about sync fidelity
    ["osovm_op", "VEIL", "<first-veil-receipt-id>"]  // optional, points at the genesis veil execution
  ],
  "content": "{\"notes\":\"...\"}",
  "sig": "<schnorr-sig>"
}
```

## Creation Receipt extension: `twin` tag

Add one optional repeatable-free tag to the existing Creation Receipt schema (no kind change needed):

```
["twin", "<twin-binding-event-id>"]   // present when this receipt is a twin-state snapshot,
                                        // points back at the kind-1903 event that established the binding
```

This keeps twin-state snapshots inside the existing, already-unified Creation Receipt / OSOVM RECEIPT
pipeline — a snapshot is just a Creation Receipt with `osovm_op` (pointing at the VEIL/RECEIPT opcode
emission, per `docs/verification-pass-1.md`'s real payload shape) plus this new `twin` tag pointing at
the binding event. No parallel receipt system for twins.

## What this deliberately does NOT do (v1)

- No automated royalty/value flow from twin-state changes — same "declared intent, not enforced"
  posture as `split` tags elsewhere in this schema set.
- No claim about *which* is authoritative (physical vs simulated) when they diverge — that's a
  downstream dispute, handled the same way any Creation Receipt dispute is: an Attestation event
  (`challenge` stance) pointing at the relevant snapshot.
- Does not require Zàngbétò quorum-verification to exist — a Twin Binding can exist for a self-run
  VeilSim instance with no witness quorum; if/when Zàngbétò's `ReceiptBundle.status` is `VERIFIED`,
  that's a stronger claim carried by a separate Attestation (`confirm` stance, real verification),
  not baked into the Twin Binding event itself.

## Open questions (ask before implementing)

1. Should a physical agent/device be limited to exactly one active Twin Binding at a time (one `sim_id`
   per `ip_root`), or can it have multiple twins for different purposes? Real `ZangbetoReceipt` data
   doesn't constrain this — it's a policy call.
2. Is `fidelity` ever machine-checked (e.g. against `ZangbetoReceipt.robustness`/`f1_score`), or is it
   purely a self-declared free-text field in v1, matching this schema set's general "advisory unless
   stated otherwise" posture?
