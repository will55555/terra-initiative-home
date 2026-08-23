# All Open Tasks — Cross-Referenced

<!-- Generated 2026-08-21. Cross-references three sources: the canonical Notion Tasks DB
     (collection://6002b34b-8bff-456a-aa43-4eed8f643dcd, Terra-domain rows), and each repo's
     own local TASKS.md (oms/, terra-api/, terra-api-fe/, terra-hq-site/) under this folder.
     Regenerate each session by re-pulling the Notion query + re-reading each TASKS.md — this
     is a snapshot, not a live view. Personal/Finance/non-Terra Notion rows are intentionally
     excluded; this file is scoped to the terra-initiative-home folder's own work. -->

## How to read this

Each repo tracks its own tasks with its own ID prefix (TAPI, TFE, THQ, OMS/ROMS). The Notion
canonical Tasks DB is a separate, higher-level list — some Notion rows map 1:1 to a repo task ID,
most don't (they're coarser "Phase" groupings or cross-cutting decisions). Where a Notion row
clearly corresponds to specific repo-level task IDs, both are listed together below.

---

## 🔴 High Priority — Open

| Task | Source | Notes |
|---|---|---|
| Phase A: Terra API branch consolidation + merge (ancestor check, SonarQube, merge order: frontend-CI → public-health → customer-identity) | Notion | No matching repo TASKS.md entry found — likely predates/spans terra-api's TASKS.md granularity. Verify current branch state before resuming. |
| Phase B: ADR-012 formalization — accept ADR, provision operator account (role=internal + scope=ops:read), build /api/v1/internal endpoints | Notion + terra-api TAPI-017/TAPI-025 | TAPI-017 (endpoints) and TAPI-025 (ops:read scope grant) both show **Done** in terra-api/TASKS.md, TAPI-025 resolved via SSM 2026-08-09. **Contradiction**: Notion still lists this as open High priority — TASKS.md is more current/detailed here, Notion row likely stale. Confirm with Will before treating as still-blocking. |
| Phase E: Resolve SEC-001 (plaintext creds), relocate Jenkins off laptops to own box + Cloudflare Tunnel, real subdomains, migrate creds/webhooks | Notion + terra-api TAPI-019/TAPI-022 | TAPI-019 (Jenkins → dedicated EC2 box) is **Done** (2026-08-05). TAPI-022 (domains + TLS) is still **Planned** — this is the still-genuinely-open part of Phase E. SEC-001 plaintext-creds item not found by that ID in any TASKS.md — needs its own lookup. |
| Terra API — deploy JVM heap caps to prod (committed 157f923, not yet applied) | Notion + terra-api TAPI-013 | **Contradiction**: terra-api/TASKS.md's TAPI-013 says heap caps were confirmed **active and verified** in prod as of 2026-08-02 (`MaxRAMPercentage=50` confirmed via `-XX:+PrintFlagsFinal`). Notion row saying "not yet applied" looks stale — TASKS.md's verification detail is more convincing. Recommend re-checking prod directly before spending time on this. |
| Set up Cloudflare Access on terra-hq.com — site fully public | Notion | No corresponding terra-hq-site TASKS.md entry found. Genuinely appears open. |
| TAPI-020 — SonarQube gate wired into Jenkins CI/CD | terra-api TASKS.md | Listed **Planned** in TASKS.md, sequenced after TAPI-019 (which is done, so this is unblocked now). CLAUDE.md/HUB_STATE reportedly say this actually closed 2026-08-07 — this exact contradiction was already flagged in `TERRA_INITIATIVE_STATE_2026-08-07.md` (2026-08-07 research doc) as needing Will's correction. Still unresolved as of this file. |
| TAPI-021 — EC2 right-size terra-api's box back toward t3.micro | terra-api TASKS.md | **Planned**. Remaining real sub-items: staging on-demand (biggest win), Alpine JRE base, trim snapd/SSM. |
| TAPI-023 — OS-level security patching automation, ecosystem-wide | terra-api TASKS.md | **Planned**, sequenced after TAPI-019 (now unblocked) so it covers Jenkins's box too. |
| TAPI-024 — Grant prod `customer_service_access` entitlement for roms/pios to Will's real account | terra-api TASKS.md | **Planned**. Needs Will's real prod `customer_id` confirmed before writing the exact SQL. |

## 🟡 Medium Priority — Open

| Task | Source | Notes |
|---|---|---|
| Phase C: Verify terra-api-fe login, wire terra-hq-site login buttons, confirm visualizer reads live public-health endpoint | Notion | Related repo work (TFE-501 done, TFE-502/503 still open — see below) suggests this is partially done already. |
| Phase D: ROMS own EC2 box — provision, onboard to terra-jenkins, redeploy, SonarQube cleanup branch | Notion + oms TASKS.md | oms/TASKS.md's ROMS-001/002/004/005 all show **Done/superseded** — ROMS is already live in production on its own EC2, redeployed, heartbeat verified. This Notion row reads as stale; the actual remaining piece may just be the "SonarQube cleanup branch" fragment. |
| Hard-reset terra-api branches on the other laptop after history rewrite | Notion | No matching repo-level task ID. Housekeeping item, do when next on that machine. |
| Scope a Jenkins capacity/scaling session | Notion | Not yet scoped anywhere else. |
| Scope a service-load learning session (OMS under high concurrency, etc.) | Notion | Not yet scoped anywhere else. |
| Terra API — decide staging: leave down, or restore on t3.small | Notion | Related to TAPI-021's "staging on-demand" sub-item — likely the same decision, not a separate task. |
| Verify Snorkel AI first invoice — check Terra Services LLC Rung 2 trigger | Notion | Business/entity-formation trigger check, not a repo task. |
| Update Terra Apparel page/site to wearables-only scope | Notion | Business-side, not represented in any repo TASKS.md. |
| TFE-502 — Redirect unauthenticated users to login instead of leaving them on a broken page | terra-api-fe TASKS.md | Open. |
| TFE-503 — Expand frontend test coverage | terra-api-fe TASKS.md | Open. |
| TFE-602 — Replace placeholder branding (/internal nav logo + favicon) | terra-api-fe TASKS.md | Blocked on Will's real designs existing. Same class as oms/TASKS.md's OMS-018 (favicon crop). |
| TFE-603 — JWT session expiry has no user-facing handling | terra-api-fe TASKS.md | Real UX bug: silent infinite-retry loop on 401 instead of a "session expired" prompt. Not fixed by design (deserves its own focused pass). |
| OMS-013 — Coordinate cross-repo `serviceId: 'roms'` → `'oms'` rename | oms TASKS.md | **Blocked**: terra-hq-site has ~80 unrelated uncommitted file deletions/an Assets/ reorg that need Will's attention before that repo is safe to touch for this. |
| OMS-015 — Redis connection root cause genuinely uncertain (health check DOWN→UP with no code change) | oms TASKS.md | Fix is committed but root cause unconfirmed — needs a dedicated investigation session with better tooling. |
| OMS-018 — Favicon/touch-icon needs tighter cropping | oms TASKS.md | Blocked on Will supplying a new source image. |

## 🟢 Low Priority / Parked

| Task | Source | Notes |
|---|---|---|
| Phase F: visual/UX refinement pass across terra-hq-site, terra-api-fe, ROMS UI | Notion | Explicitly parked until Phases A–E are live. |
| Phase G: scale/load planning (concurrent users, DB pooling, Kafka throughput, EC2 sizing) | Notion | Explicitly parked until ecosystem is fully live. |
| Decide: Terra Agriculture sourcing for OMS — Cameroon-only or cross-property principle | Notion | Business decision, no code dependency. |
| Stand up separate goods-line area (bamboo/calabash hard goods) | Notion | Business decision. |
| Decide entity home for goods line | Notion | Business decision. |
| Define SKU/marker scheme for guest-purchasable vs. resort-owned goods items | Notion | Depends on the goods-line decisions above. |
| Build automated invisible-comment watermarking script | Notion | No repo home identified yet. |
| TAPI-026 — Manual quarantine/release from the Operator tab (click a cube) | terra-api TASKS.md | Backlog — deliberately unscoped further until Will designs it properly. |
| TFE-601 — "My Services" / "Ecosystem" tab split | terra-api-fe TASKS.md | Design brainstormed, deliberately shelved for a dedicated session. |
| TFE-604 — Menu popover feature ideas (account info, quick links, live status dot) | terra-api-fe TASKS.md | Captured for a future refinement pass, not scoped. |
| TFE-605 — Cube slow-pulse animation | terra-api-fe TASKS.md | Deferred — real animation-curve work, not a quick add. |
| THQ-002 — Visualizer cube color: graduated health tier vs. binary connected | terra-hq-site TASKS.md | **Planned**, likely superseded — THQ-017 already ported FE's tier-color fixes into this file 2026-08-09. Worth confirming whether THQ-002 is actually still needed or can be closed. |
| THQ-004 — Design pass follow-ups (per-subsidiary card images, puzzle-piece shape, 3D backdrop asset) | terra-hq-site TASKS.md | Notes only, no code yet. |
| THQ-018 — Custom circular cursor (Montfort-style), site-wide | terra-hq-site TASKS.md | Design discussion only — see [[reference_montfort-design]] memory. |

## ⚠️ Uncommitted / Pending-Commit Work (not "open tasks" but real risk)

A large amount of terra-hq-site work is marked **Done — pending commit** in its own TASKS.md
(THQ-005 through THQ-017, minus THQ-002/004/018). None of this has landed in git yet. This is
the single biggest "silent risk" in the whole list — a machine issue, branch mixup, or accidental
`git checkout .` could lose real, already-approved work. Recommend committing this batch before
anything else on this list.

## In Progress

| Task | Source | Notes |
|---|---|---|
| THQ-001 — Visualizer frontend integration ADR + migration plan | terra-hq-site TASKS.md | Marked In Progress since 2026-07-17 — check if this is stale given how much visualizer work (THQ-002/003/017) has shipped since. |
| THQ-003 — Pipeline extension tubes freeze connected state at creation, never refresh | terra-hq-site TASKS.md | Fix drafted and syntax-checked, **not yet visually confirmed**, uncommitted, local to the `test` machine only. Needs the visual-verification repro steps run before committing. |

## Known Contradictions — RESOLVED 2026-08-23

Notion MCP access became available this session; each row below was checked directly against
Notion (not inferred) and corrected there where TASKS.md was clearly more current.

1. **TAPI-020 (SonarQube gate)** — no standalone Notion row existed to correct; it was only
   referenced inside Phase A's title and the meta-correction task. Nothing to close.
2. **JVM heap caps "not yet applied"** — Notion row marked **Done**, referencing TAPI-013's
   verified `-XX:+PrintFlagsFinal` confirmation (2026-08-02).
3. **ADR-012 / operator account provisioning** — Notion's "Phase B" row marked **Done**. ADR-012's
   own page Status field also flipped **Proposed → Accepted**, since its 2026-08-09 update note
   already documented live production verification.
4. **ROMS EC2 box / Phase D** — Notion row marked **Done**, per the Ha'bem (OMS) Notion project
   page's own log confirming ROMS-001/002 closed 2026-08-08. **One real fragment is NOT closed**:
   disabling SonarCloud Automatic Analysis for the OMS project — that same page's log still lists
   it as an open manual step.

**Also found and fixed in the same pass:**
- terra-api-fe's Notion project page wrongly said its own repo was `will55555/terra-api-home` —
  corrected to `will55555/terra-api-fe` (confirmed via `git remote -v`).
- Two Notion meta-tasks this file's 2026-08-21 pass had itself generated are now closed: "Correct
  stale Notion Tasks DB rows" (this work) and "Confirm whether a Machine Paths table exists in
  Notion" (confirmed via search — it does not; Machine Paths is a `claude-skills`-only artifact).

**Left open, not closed — a scope call, not a factual correction:** "Phase A: Terra API branch
consolidation" — its Notion page is blank and names branches (`frontend-CI`, `public-health`,
`customer-identity`) that don't exist under those names in `terra-api`'s current branch list. The
underlying work looks done via differently-named merges, but confirm with Will before closing it.

**Also corrected this session (found while re-verifying, not part of the original 4):**
`terra-hq-site/TASKS.md`'s THQ-003 row said "uncommitted, local to `test` machine only" — false on
this machine (the fix IS committed, `43805a9a`). Real finding: the commit only half-applies the
fix (child tube's live-refresh assignment is real code; the parent tube's matching fix is
commented out) — TASKS.md now describes this precisely. See `ACTION_PLAN/04-terra-hq-site.md` §0
for full detail.
