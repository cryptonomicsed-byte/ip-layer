# ip-layer

Ọmọ Kọ́dà IP Layer — a portable IP identity and provenance layer built on Nostr. Every Ọmọ Kọ́dà
agent is born with one; usable by any agent framework or human creator. Coordinated (optionally) by
a Wyoming DAO LLC legal wrapper — the DAO does not take ownership of creators' content.

Status: **schema-draft stage, verified**, implementation not started. See `schemas/` for the four core
event schemas: IP Root genesis (kind `31900`), Creation Receipt (kind `1901`, unified with OSOVM's
RECEIPT opcode via `osovm_op`, licensed via real NIP-32 `l`/`L` tags, extended with a `twin` tag),
Attestation (kind `1902`, vouch/challenge/confirm), and Twin Binding (kind `1903`, covers 1:1
digital-twin ownership — see `schemas/twin_binding.md`). No separate license-layer schema was needed —
NIP-32 labels on the Creation Receipt cover it. See `docs/verification-pass-1.md` for the pre-
implementation verification pass (kind-collision check, real OSOVM RECEIPT opcode payload shape).

100% universal wording — no Orisha/Yoruba naming on any public surface, per the ecosystem-wide
locked canon. This project is explicitly civic/legal-facing.

## v1 scope

In: IP Root at birth, Creation Receipts, license declaration, public provenance on Nostr, direct
payment hooks (NIP-57 zaps), basic attestation, optional Wyoming DAO binding.

Out: streaming platform/player, automated royalty accounting, automated legal enforcement, taking
ownership of user content, algorithmic discovery.

## Build order

1. ✅ IP Root genesis + Creation Receipt schemas
2. ✅ License layer (NIP-32 labels, folded into Creation Receipt) + Attestation event schema
2.5. ✅ Pre-implementation verification pass (kind-collision check, real OSOVM RECEIPT opcode shape)
     + Twin Binding schema (kind `1903`, covers 1:1 VeilSim digital-twin ownership)
3. Omo-Koda2 birth-flow hook (IP Root creation) — **next up, not started**
4. Creation Receipt emission on major "create" actions
5. Shared Nostr publish library
6. Wyoming DAO LLC binding (optional, per IP Root)

## Open questions carried forward (see individual schema files for full detail)

- ~~Provisional kind numbers need a collision check~~ — **resolved**, see `docs/verification-pass-1.md`.
  Still provisional pending a real NIP submission, but no collision found.
- Authorization rule for which pubkeys may emit receipts under a given IP Root.
- ~~OSOVM's actual RECEIPT opcode payload shape~~ — **resolved**, see `docs/verification-pass-1.md`.
- Whether `challenge` attestations should have any protocol-level effect in v1, or stay UI-only.
- Whether Zàngbétò's attestation `weight` is self-declared or requires an external stake lookup
  (real `ZangbetoReceipt`/quorum machinery exists — see verification doc — but policy still undecided).
- Twin Binding: one active twin per IP Root, or multiple allowed? Is `fidelity` ever machine-checked?
