# 09 — Terra Nkap (crypto) — NOT YET SCOPED, placeholder only

**Status: you said "not sure" — this file captures what's already scattered across the hub, it
does not commit you to starting anything.** Nkap is structurally different from everything else in
this action plan (PIOS/OMS/terra-hq-site are all conventional web apps; a crypto/token component
brings in custody, regulatory, and settlement concerns none of the other repos have) — worth
treating as its own decision, not folded into the general backlog.

---

## What's already on record, scattered across the hub (not a plan, just what exists)

- **Ventures structure**: `terra_ventures.html` has an existing "Nkap" tab (per terra-hq-site's
  THQ-011 merge — chain/Nkap content was consolidated there when `terra_business_structure.html`
  was folded in). Nkap is treated as a child of the Terra Ventures domain, alongside ROMS/PIOS.
- **Ha'bem/OMS integration point**: the checkout flow already has a placeholder "Terra Nkap"
  payment option with an **8% discount rate** — explicitly a placeholder, waiting on "Terra Chain
  settlement design" being real before it can be pinned to an actual rate.
- **Terra Agriculture cross-link**: a conceptual thread (§13 of `roms-expansion-sketch.md`) noted
  both Ha'bem's Nkap discount idea AND Terra Agriculture's harvest-tokenization idea independently
  plan to use Terra Nkap — the one concrete cross-vertical thread that exists today, though nothing
  is built on either side.
- **No dedicated ADR series exists yet** — unlike `terra-api-adr-*`, `roms-adr-*`, `pios-adr-*`,
  there's no `nkap-adr-*` sequence. If this becomes real, it likely wants one, following the
  ecosystem's own convention (per the hub's "read the spec before designing" rule, an ADR is where
  the architecture decisions below would need to land before code).
- **Domain**: per the Option C domain-naming decision (`02-terra-api-infra.md` item 3), if Nkap
  becomes a real user-facing product (a wallet, an exchange interface, anything a customer touches
  directly), it likely wants its own domain too, same logic as Ha'bem/PIOS — but this is even less
  settled than those two, since it's not clear yet whether Nkap is a customer-facing product at all
  or purely backend settlement infrastructure other products call into.

## Open questions that would need answering before any real scoping session

These aren't answered anywhere in the hub yet — surfacing them so the decision to start (or not)
is an informed one:
1. **What IS Nkap, concretely?** A settlement/discount mechanism only (backend infra other products
   call), or a real tradeable/holdable token customers interact with directly? The answer changes
   everything — custody requirements, regulatory exposure, and whether it needs its own
   customer-facing surface at all.
2. **Regulatory scope** — "crypto" spans an enormous range (a closed-loop loyalty-point system with
   no real transferability is a very different regulatory animal than an actual blockchain token).
   Worth deciding which end of that spectrum this is before any design work, since it changes
   whether this needs legal review before engineering even starts.
3. **Custody model** — if real value is held, who holds it, and under what entity (this ecosystem
   already tracks entity-formation status for its other ventures — Nkap would need the same
   question answered: does it need its own entity, or does it live inside an existing one).
4. **Relationship to Terra Chain** — HUB_STATE references "Terra Chain settlement design" as a
   separate, not-yet-started thing Nkap's discount rate depends on. Is Terra Chain the underlying
   ledger/infrastructure and Nkap the token that rides on it? That relationship isn't documented
   anywhere found this session — worth clarifying before treating them as two separate initiatives.

## Recommendation

Given how early and undefined this is — no ADR, no entity decision, no clarity on what kind of
"crypto" this even is — a dedicated **planning/design session** (matching the pattern already used
for PIOS's "scope a Jenkins capacity session" style deferrals) makes more sense than jumping into
implementation planning now. If you want to move forward, the next concrete step would be
answering the 4 open questions above, on paper, before this file grows any further.
