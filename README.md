# ip-layer

Ọmọ Kọ́dà IP Layer — a portable IP identity and provenance layer built on Nostr. Every Ọmọ Kọ́dà
agent is born with one; usable by any agent framework or human creator. Coordinated (optionally) by
a Wyoming DAO LLC legal wrapper — the DAO does not take ownership of creators' content.

Status: **schema-draft stage**, nothing implemented yet. See `schemas/` for the three core event
schemas drafted so far: IP Root genesis (kind `31900`), Creation Receipt (kind `1901`, unified with
OSOVM's RECEIPT opcode via `osovm_op`, licensed via real NIP-32 `l`/`L` tags), and Attestation
(kind `1902`, vouch/challenge/confirm). No separate license-layer schema was needed — NIP-32 labels
on the Creation Receipt cover it.

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
3. Omo-Koda2 birth-flow hook (IP Root creation)
4. Creation Receipt emission on major "create" actions
5. Shared Nostr publish library
6. Wyoming DAO LLC binding (optional, per IP Root)

## Open questions carried forward (see individual schema files for full detail)

- Provisional kind numbers (31900 / 1901 / 1902) need a collision check against other ecosystem NIPs.
- Authorization rule for which pubkeys may emit receipts under a given IP Root.
- OSOVM's actual RECEIPT opcode payload shape — needed to confirm `osovm_op` unification is real, not
  cosmetic.
- Whether `challenge` attestations should have any protocol-level effect in v1, or stay UI-only.
- Whether Zàngbétò's attestation `weight` is self-declared or requires an external stake lookup.
