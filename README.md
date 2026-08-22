# ip-layer

Ọmọ Kọ́dà IP Layer — a portable IP identity and provenance layer built on Nostr. Every Ọmọ Kọ́dà
agent is born with one; usable by any agent framework or human creator. Coordinated (optionally) by
a Wyoming DAO LLC legal wrapper — the DAO does not take ownership of creators' content.

Status: **schema-draft stage**, nothing implemented yet. See `schemas/` for the two core event
schemas drafted so far (IP Root genesis, Creation Receipt). License-layer (NIP-32) and Attestation
event schemas are next.

100% universal wording — no Orisha/Yoruba naming on any public surface, per the ecosystem-wide
locked canon. This project is explicitly civic/legal-facing.

## v1 scope

In: IP Root at birth, Creation Receipts, license declaration, public provenance on Nostr, direct
payment hooks (NIP-57 zaps), basic attestation, optional Wyoming DAO binding.

Out: streaming platform/player, automated royalty accounting, automated legal enforcement, taking
ownership of user content, algorithmic discovery.

## Build order

1. IP Root genesis + Creation Receipt schemas (this commit)
2. License layer (NIP-32 labels) + Attestation event schema
3. Omo-Koda2 birth-flow hook (IP Root creation)
4. Creation Receipt emission on major "create" actions
5. Shared Nostr publish library
6. Wyoming DAO LLC binding (optional, per IP Root)
