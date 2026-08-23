# 07 — Notion Contradictions + Non-Code Business Decisions

**Updated 2026-08-23: Part A is now resolved.** Notion MCP access became available this session —
all 4 contradictions below were checked directly against Notion (not inferred) and corrected there.
Keeping the record here so it's clear what happened and why, plus one item that's still genuinely
open.

---

## Part A — Contradictions: RESOLVED 2026-08-23

1. **TAPI-020 (SonarQube gate).** No standalone Notion row existed to correct — it was only
   referenced inside Phase A's title and a since-closed meta-task. Nothing to close in Notion.
   **Still real, not yet fixed:** `terra-api/TASKS.md`'s own TAPI-020 row still says "Planned,"
   while HUB_STATE says it closed 2026-08-07 — this is a contradiction WITHIN the same repo's own
   docs, not a Notion drift. Fixing that row is a `terra-api/TASKS.md` edit, not done as part of
   this pass (out of scope for a Notion-focused correction) — still worth a 2-minute fix next time
   that repo is open. Verify in Jenkins first (recent `master` build shows the quality-gate stage
   passing) before flipping the status.
2. **JVM heap caps "not yet applied."** Notion row marked **Done**, referencing TAPI-013's
   `-XX:+PrintFlagsFinal` verification (2026-08-02, confirmed `MaxRAMPercentage=50` actually active).
3. **ADR-012 / operator account provisioning.** Notion's "Phase B" row marked **Done**. ADR-012's
   own page Status field also flipped **Proposed → Accepted** (its 2026-08-09 update note already
   documented live production verification, so the ADR's formal status just hadn't caught up).
4. **ROMS EC2 box / "Phase D."** Notion row marked **Done**, per the Ha'bem (OMS) Notion project
   page's own log confirming ROMS-001/002 closed 2026-08-08. **One real fragment survives, still
   open:** disabling SonarCloud Automatic Analysis for the OMS project — that page's own log still
   lists it as an open manual step. See `05-roms-oms.md` item 5.

**Also fixed in the same pass:**
- terra-api-fe's Notion project page wrongly said its own repo was `will55555/terra-api-home` —
  corrected to `will55555/terra-api-fe`.
- **"Commit terra-hq-site's ~13 pending-commit items"** — marked **Done** 2026-08-23. All of
  THQ-005–018 landed via a merge into `main` (commit `29d269c`), confirmed pushed, zero divergence
  from origin. This was a real, correctly-open Notion row until today.
- Two self-generated meta-tasks closed: "Correct stale Notion Tasks DB rows" (this work) and
  "Confirm whether a Machine Paths table exists in Notion" (confirmed via search: it does not).

**Left open in Notion, correctly — a scope call, not a fact to verify:**
- **Phase A** (Terra API branch consolidation): page is blank, names branches (`frontend-CI`,
  `public-health`, `customer-identity`) that don't exist under those names in `terra-api`'s current
  branch list. The underlying work looks done via differently-named merges — confirm with yourself
  before closing it, since I won't close a scope decision unilaterally.
- **Phase E** (SEC-001, Jenkins off laptops + Cloudflare Tunnel, real subdomains): TAPI-019
  (Jenkins to its own box) is Done. TAPI-022 (domains+TLS) is still genuinely open — see
  `02-terra-api-infra.md` item 3, now folded together with the domain-naming brainstorm below.
  **SEC-001 specifically** wasn't found under that ID anywhere — search Notion directly for
  "SEC-001" next time, since it may already be covered by TAPI-016's security hardening bundle
  without ever being cross-referenced.
- **Phase C** (verify login flows + live visualizer endpoint): still open, genuinely unverified —
  see `03-terra-api-fe.md` item 0.
- **"Hard-reset terra-api branches on the other laptop"** — housekeeping for `solan`, not urgent.
- **"Scope a Jenkins capacity/scaling session"** / **"Scope a service-load learning session"** —
  both explicitly want a learning/design pass before implementation. Future session topics.
- **"Terra API — decide staging: leave down, or restore on t3.small"** — same decision as
  TAPI-021's "staging on-demand" sub-item in `02-terra-api-infra.md` item 1.

---

## Part B — Non-Code Business Decisions (no repo dependency)

1. **Set up Cloudflare Access on terra-hq.com** — site is currently fully public. **Site:**
   Cloudflare dashboard → Zero Trust → Access → Applications. Decide which paths (if any) need
   gating, and whether this should apply to the new subdomains/domains being planned in
   `02-terra-api-infra.md` item 3 too — decide once, apply consistently.
2. **Domain-naming brainstorm for each ecosystem service (raised 2026-08-23, NEW)** — moved to
   `02-terra-api-infra.md` item 3, since it's really the same decision as TAPI-022's domain rollout,
   just widened from "pick subdomains" to "pick the whole naming scheme." See that file for the
   actual options.
3. **Verify Snorkel AI first invoice** — check whether the Terra Services LLC "Rung 2" formation
   trigger has fired. **Site:** wherever Terra Services LLC's invoicing/billing records live.
4. **Update Terra Apparel page/site to wearables-only scope** — business-side scope narrowing.
5. **Terra Agriculture sourcing scope (Cameroon-only vs. cross-property principle)** — see
   `05-roms-oms.md` item 6, decide these together since they're linked.
6. **Goods-line decisions** (bamboo/calabash hard goods): entity home first, then the goods-line
   area, then the SKU scheme (depends on both).
7. **Automated invisible-comment watermarking script** — decide which repo/service owns this before
   building anything.

---

## Part C — Parked, no action needed now (Phase F / Phase G)

Explicitly parked per Notion — visual/UX refinement pass (Phase F) and scale/load planning
(Phase G) are both deliberately deferred until Phases A–E are live / the ecosystem is fully live.
Nothing to do here.
