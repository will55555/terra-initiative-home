# 06 — PIOS

No code exists for PIOS yet — everything here is a status check and a decision, not implementation.

---

## 1. Confirm the learning-stack prerequisite gate status (DECISION NEEDED — this is the real blocker)

**Why it matters:** all of PIOS's ADRs (011–015) are Accepted — the ADR sequence is NOT what's
blocking coding. The actual blocker is a learning-stack prerequisite: **CS50P + Karpathy Zero to
Hero (Python) → Angular → DSA**, required before any PIOS code starts. PIOS's own Notion project
page (last synced 2026-05-30) had this marked "not started" as of that sync — but that page itself
may be stale by now.

**Sites:** the PIOS Notion project page.

**Steps:**
1. Open the PIOS Notion project page and check the last-synced date again — if it's still
   2026-05-30, the page itself hasn't been touched in ~3 months and needs a fresh look regardless.
2. Honestly assess: has any of CS50P, Karpathy's Zero to Hero, Angular, or the DSA progression
   (tracked separately in this same ecosystem as `DSA-001` onward, currently just starting Arrays
   per HUB_STATE) actually progressed since May?
3. Update the PIOS Notion page with the real current status either way — even "still not started"
   is worth recording explicitly rather than leaving a 3-month-stale page as the only signal.
4. If the gate genuinely hasn't moved: PIOS stays correctly blocked, no further action.
5. If it HAS progressed (or you want to explicitly waive/reorder the gate): PIOS is then ready to
   start on the event-sourced write path per ADR-011/012/013 — that's a real coding project start,
   worth its own dedicated planning session rather than jumping in.

---

## 2. Archive the duplicate ADR-013 page (low priority, cosmetic)

**Why it matters:** two Notion pages exist for the same ADR-013 decision (event schema versioning)
— an early 2026-05-12 draft filed under a superseded "Investment System App" page, and the formal
2026-07-13 `pios-adr-013` filed properly under PIOS. Both say the same thing (Accepted), not a real
conflict, just clutter.

**Steps:**
1. Confirm both pages exist and say the same thing.
2. Archive (don't delete — keep history) the older 2026-05-12 draft, with a note/link pointing to
   the canonical `pios-adr-013`.

---

## 3. NEW 2026-08-23 — PIOS will need its own domain (no action yet, just noted)

Per the domain-naming decision in `02-terra-api-infra.md` item 3 (Option C — split by audience),
PIOS is a consumer/investor-facing product like Ha'bem, so it gets its own dedicated domain once it
has a real product surface — not a `terra-hq.com` subdomain. Nothing to do now: no product name is
locked in and no repo exists yet (blocked on item 1's learning-stack gate). Logged here purely so
the same naming pattern gets applied when PIOS actually gets to that stage, rather than being
decided separately/inconsistently later.
