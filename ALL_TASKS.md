# All Open Tasks — Cross-Referenced

<!-- Regenerated 2026-08-30. Cross-references the canonical Notion Tasks DB (live pull,
     collection://6002b34b-8bff-456a-aa43-4eed8f643dcd, Terra-domain rows) against each repo's own
     TASKS.md, read DIRECTLY from this machine's clones — terra-api/, terra-api-fe/, terra-hq-site/,
     oms/ all live nested inside terra-initiative-home/ (gitignored siblings, confirmed present and
     git-clean on this machine as of this pass; terra-jenkins/ has no TASKS.md, infra-only repo).
     Regenerate each session by re-pulling Notion + re-reading each TASKS.md — this is a snapshot,
     not a live view. Personal/Finance/non-Terra Notion rows are intentionally excluded. -->

## How to read this

Each repo tracks its own tasks with its own ID prefix (TAPI, TFE, THQ, ROMS/OMS). The Notion
canonical Tasks DB is a separate, higher-level list — some Notion rows map 1:1 to a repo task ID,
most don't (they're coarser "Phase" groupings or cross-cutting decisions).

## What changed since the last snapshot (2026-08-30 → 2026-10-03)

**Context: this pass found a real gap, not just drift.** 8 new Notion tasks were created between
2026-09-28 and 2026-10-02 that no prior session's `load hub` ever surfaced — `load hub`'s Linear
Fetch Mode only reads local hub/repo files, never live Notion, so task-DB-only planning sessions
(Notion-only, no repo touched) go invisible to this file until someone explicitly re-pulls Notion.
Also found during this pass: 4 Obsidian Queue sub-pages sitting under the Terra API Notion project
page, 3 of which were genuinely undrained (1 note missing from the vault, written this pass to
`Software Development/Testing/linter-driven-fixes-can-introduce-real-bugs.md`; the other 3 notes
across those pages had already landed in the vault despite the page never being marked drained —
spot-checked one byte-for-byte to confirm, not assumed). All 3 pages now marked drained in Notion
(page-delete tool unavailable, same as the existing 2026-08-05 precedent — left in place, safe to
delete manually). Also **separately found 2026-09-14**: `notion-space-audit` restructured the 🌍
Terra Inc page itself (per its own page history) — not investigated further this pass, worth a
look if Terra Inc's page structure matters for the next session that touches it.

**New Notion tasks (2026-09-28 → 2026-10-02) — none yet have a matching repo TASKS.md ID:**

| Task | Domain | Priority | Likely repo | Notes |
|---|---|---|---|---|
| HQ Site: Subsidiary emergence v1 (vanilla puzzle block) | Coding | High | terra-hq-site | No THQ-### ID assigned yet — scope this into one if picked up. |
| Ha'bem — QR codes for scan-to-order (per room/table — confirm granularity) | Coding | High | oms | No ROMS/OMS-### ID yet. Granularity question (room vs. table) is an open design call, not just a build task. |
| Mobile audit — everything shipped (Brainstorm Required) | Coding | High | ecosystem-wide | Explicitly tagged Brainstorm Required — a scoping session, not ready to implement. |
| Customer-facing app shell — standard footer, legal/compliance links, share + bottom nav | Engineering | Medium | unclear — terra-api-fe or oms, or a new shared component | Cross-cutting; worth deciding which repo owns this before starting, since "customer-facing app shell" could mean either product's frontend. |
| Payments layer for customer-facing apps — multi-provider abstraction (Stripe + Africa mobile money) | Engineering | Medium | likely oms (Ha'bem already has mobile-money adapter groundwork per ROMS's brand/data-model design) | Real architecture decision — multi-provider abstraction is a design-before-code item, not a quick add. |
| App store readiness — Google Play + Apple App Store (Brainstorm Required) | Coding | Medium | oms (if Ha'bem goes mobile) | Tagged Brainstorm Required — implies no mobile app exists yet to submit; scope unclear from the title alone. |
| HQ Site: Move Terra API into Terra Tech page | Coding | Medium | terra-hq-site | Small, mechanical-sounding — a content/nav reorg, consistent with past THQ-010/011 merge pattern. |
| Visualizer: confirm location (terra-api-fe?) + record 8-cube role model | Coding | Medium | terra-api-fe or terra-hq-site | Sounds like a documentation/decision task (which visualizer owns the 8-cube model going forward), not new code. |
| HQ Site: Subsidiary emergence v2 — upgrade to 3D globe (later) | Coding | Low | terra-hq-site | Explicitly deferred ("later") in its own title — parked, not actionable now. |

**Not reconciled further this pass** — these are real findings surfaced, not resolved. Each would
need its own scoping session (most are tagged Brainstorm/Research Required by their own Notion
title, or are cross-cutting enough that picking the right repo home is itself a decision) before
they're ready to become a repo TASKS.md row. Re-pull Notion next session to confirm none of these
flipped status since.

## What changed since the last snapshot (2026-08-23 → 2026-08-30)

1. **terra-hq-site was 2 commits behind `origin/main` on this machine** — the THQ-005–017 merge
   (`29d269c`, done from a different clone 2026-08-23) had never been pulled down here. Fast-forwarded
   clean, zero conflicts (1-line TASKS.md diff only).
2. **TAPI-020 fixed** — `terra-api/TASKS.md`'s own row said "Planned" against Jenkins/HUB_STATE
   evidence of a 2026-08-07 close. Corrected to Done, committed (`5ff2841`), pushed to both GitHub
   and Bitbucket remotes.
3. **THQ-005–017 status labels fixed** — all 13 rows in `terra-hq-site/TASKS.md` still said "Done —
   pending commit" despite being confirmed merged and pushed. Corrected to plain "Done" (or "Done,
   confirmed committed/pushed 2026-08-30" where the row had its own trailing commit-status note),
   committed (`78c4e6e`), pushed.
4. **OMS-013's blocker confirmed cleared** — the rename was blocked on `terra-hq-site` having ~80
   uncommitted file deletions; that landed with the THQ-005–017 merge. Noted in `oms/TASKS.md`
   (`ff7db76`), pushed. The rename itself has NOT been started — only the prerequisite is clear.
5. **TAPI-025 re-read directly**: its own Status column literally says "Closed 2026-08-09" — fully
   done (SSM access provisioned, operator account created, live-verified), not merely "steps may
   have run" as the prior pass's secondhand framing suggested.
6. **No Notion rows required correction this pass** — the 4 rows corrected 2026-08-23 (JVM heap
   caps, ADR-012/Phase B, ROMS EC2/Phase D, "commit terra-hq-site's pending items") are all still
   marked Done in the live pull, consistent. Live drift since is purely additive: several new
   personal/business-brainstorm tasks were added to the Notion DB between 2026-08-21 and 2026-08-29
   (see "New Notion tasks since last pass" below) — none contradict repo state.

---

## 🔴 High Priority — Open

| Task | Source | Notes |
|---|---|---|
| Phase A: Terra API branch consolidation + merge (ancestor check, SonarQube, merge order: frontend-CI → public-health → customer-identity) | Notion | Still Todo. Notion page is blank; names branches (`frontend-CI`, `public-health`, `customer-identity`) not present under those names in `terra-api`'s current branch list (`phase-2-auth`, `phase-3-resilience`, `phase-4-governance`, `phase-5-redis`, `phase-6-cicd`, `phase-8-customer-identity`, `sonarqube-quality-gate`, `rename/roms-to-oms-mentions`). The underlying work looks done via differently-named merges — this is a scope call for Will, not a fact to verify, so left open. |
| Phase E: Resolve SEC-001 (plaintext creds), relocate Jenkins off laptops to own box + Cloudflare Tunnel, real subdomains, migrate creds/webhooks | Notion + terra-api TAPI-019 | TAPI-019 (Jenkins → dedicated EC2 box) confirmed **Done** directly in `terra-api/TASKS.md`. TAPI-022 (domains + TLS, the other real piece of Phase E) is still **Planned** — see below. SEC-001 not found under that ID in any TASKS.md read this pass. |
| Set up Cloudflare Access on terra-hq.com — site fully public | Notion | Still Todo. No corresponding terra-hq-site TASKS.md entry — genuinely open, business/security decision. |
| TAPI-021 — EC2 right-size terra-api's box back toward t3.micro | terra-api TASKS.md | Confirmed **Planned** directly in the file. Remaining real sub-items per the row's own text: staging on-demand (biggest win), Alpine JRE base, trim snapd/SSM. |
| TAPI-022 — Domains + TLS for all remaining ecosystem endpoints | terra-api TASKS.md | Confirmed **Planned** directly. Only `api.terra-hq.com` has real HTTPS today. |
| TAPI-023 — OS-level security patching automation, ecosystem-wide | terra-api TASKS.md | Confirmed **Planned** directly, sequenced after TAPI-019 (done) so it covers Jenkins's box too. |
| TAPI-024 — Grant prod `customer_service_access` entitlement for roms/oms to Will's real account | terra-api TASKS.md | Confirmed **Planned** directly. Blocked on confirming Will's real prod `customer_id` first. |

## 🟡 Medium Priority — Open

| Task | Source | Notes |
|---|---|---|
| Phase C: Verify terra-api-fe login, wire terra-hq-site login buttons, confirm visualizer reads live public-health endpoint | Notion | Still Todo. Not independently re-verified this pass (would need live-site browsing, not a repo-file read) — genuinely open. |
| OMS-013 — Coordinate cross-repo `serviceId: 'roms'` → `'oms'` rename | oms TASKS.md | **Blocker cleared 2026-08-30** (see "What changed" above) — safe to start now, not yet started. Sequencing note in the row itself: `terra-api`'s prod DB update, `TerraHeartbeatScheduler.java`'s `SERVICE_ID`, and `terra-api-fe`'s `domainConfig.js`/`productConfig.js` should land in one tight deploy window. |
| Hard-reset terra-api branches on the other laptop after history rewrite | Notion | Still Todo. No matching repo-level task ID — housekeeping for a different machine. |
| Scope a Jenkins capacity/scaling session | Notion | Still Todo. Explicitly wants a learning/design pass before implementation. |
| Scope a service-load learning session (OMS under high concurrency, etc.) | Notion | Still Todo. Same as above. |
| Terra API — decide staging: leave down, or restore on t3.small | Notion | Still Todo. Same decision as TAPI-021's "staging on-demand" sub-item. |
| Verify Snorkel AI first invoice — check Terra Services LLC Rung 2 trigger | Notion | Still Todo. Business/entity-formation check, not a repo task. |
| Update Terra Apparel page/site to wearables-only scope | Notion | Still Todo. Business-side, no repo TASKS.md entry. |
| [Brainstorm Required] Apparel page/site architecture + sourcing (Alibaba vs. POD) | Notion | **New since last pass** (created 2026-08-21, not previously captured here). Business decision. |
| [Brainstorm Required] Terra Chain/Nkap: build trigger + geographic scope (Africa-only vs. also US) | Notion | **New since last pass** (created 2026-08-21). See `ACTION_PLAN/09-terra-nkap-crypto.md` for the fuller open-questions writeup — this Notion row is the narrower "build trigger + geography" slice of that same undecided space. |
| Terra API logo: animated dot-wave technique (canvas/sine) | Notion | **New since last pass** (created 2026-08-24). Coding idea, not scoped into any repo TASKS.md yet. |
| [Brainstorm Required] Terra Ventures investment thesis | Notion | **New since last pass** (created 2026-08-29). Business decision. |
| TFE-502 — Redirect unauthenticated users to login instead of leaving them on a broken page | terra-api-fe TASKS.md | Confirmed **Open** directly in the file. |
| TFE-503 — Expand frontend test coverage | terra-api-fe TASKS.md | Confirmed **Open** directly. |
| OMS-015 — Redis connection root cause genuinely uncertain | oms TASKS.md | Confirmed still **uncertain** directly in the file — the row is explicit that the `spring.redis.*`→`spring.data.redis.*` fix is committed but unverified as the actual cause; needs a dedicated investigation session with better tooling. |
| OMS-018 — Favicon/touch-icon needs tighter cropping | oms TASKS.md | Confirmed **Open**, blocked on Will supplying a new source image. |
| TFE-602 — Replace placeholder branding (/internal nav logo + favicon) | terra-api-fe TASKS.md | Confirmed **Open**, blocked on Will's real designs. Same class as oms's OMS-018. |
| TFE-603 — JWT session expiry has no user-facing handling | terra-api-fe TASKS.md | Confirmed **Open** — real UX bug (silent infinite-retry on 401 instead of a "session expired" prompt), deserves its own focused pass. |

## 🟢 Low Priority / Parked

| Task | Source | Notes |
|---|---|---|
| Phase F: visual/UX refinement pass across terra-hq-site, terra-api-fe, ROMS UI | Notion | Explicitly parked until Phases A–E are live. |
| Phase G: scale/load planning | Notion | Explicitly parked until ecosystem is fully live. |
| Decide: Terra Agriculture sourcing for OMS — Cameroon-only or cross-property principle | Notion | Business decision, no code dependency. |
| Stand up separate goods-line area (bamboo/calabash hard goods) | Notion | Business decision. |
| Decide entity home for goods line | Notion | Business decision. |
| Define SKU/marker scheme for guest-purchasable vs. resort-owned goods items | Notion | Depends on the goods-line decisions above. |
| Build automated invisible-comment watermarking script | Notion | No repo home identified yet. |
| [Research Required] Amazon seller plan for bamboo/calabash goods line | Notion | **New since last pass** (created 2026-08-23). |
| Define numeric criteria for "decent unit in good area" — Dual-Use Global Property concept | Notion | **New since last pass** (created 2026-08-29). Unrelated personal/business real-estate concept, not a Terra repo task. |
| TAPI-026 — Manual quarantine/release from the Operator tab | terra-api TASKS.md | Confirmed **Backlog** directly — deliberately unscoped until Will designs it properly. |
| TFE-601/604/605 | terra-api-fe TASKS.md | Confirmed **Open/Backlog** directly — tab split, menu popover ideas, cube pulse animation. All deliberately shelved for future design sessions. |
| THQ-002 — Visualizer cube color: graduated health tier vs. binary connected | terra-hq-site TASKS.md | Confirmed still **Planned** directly in the file, though HUB_STATE separately claims this already shipped 2026-08-03. Worth a live-site check (load `terra-hq.com`, confirm graduated tier colors) to close this out — not verified this pass since it needs a browser, not a file read. |
| THQ-004 — Design pass follow-ups (per-subsidiary card images, puzzle-piece shape, 3D backdrop asset) | terra-hq-site TASKS.md | Confirmed **Planned**, notes only, no code yet. |
| THQ-018 — Custom circular cursor (Montfort-style), site-wide | terra-hq-site TASKS.md | Confirmed **Planned**, design discussion only. See [[reference_montfort-design]] memory. |

## In Progress

| Task | Source | Notes |
|---|---|---|
| THQ-001 — Visualizer frontend integration ADR + migration plan | terra-hq-site TASKS.md | Confirmed still **In Progress** directly in the file, open since 2026-07-17. Worth checking whether terra-api-fe's TFE-401 (feature-complete Three.js port) already IS this migration — a Notion/ADR-009 cross-check, not done this pass. |
| THQ-003 — Pipeline extension tubes freeze connected state at creation, never refresh | terra-hq-site TASKS.md | Confirmed directly: **"In Progress — committed but INCOMPLETE."** The fix (`43805a9a`) is committed but only half-applied — the child tube's live-refresh assignment is real code; the parent tube's matching two lines are still commented out in `terra_api_visualizer_phase5.js` (~lines 666–667). This is a real, still-open bug, not a commit-risk item. Fix is a 2-line uncomment + a visual repro pass (see the file's own row for exact repro steps). |

## Recently Confirmed Done (this pass, direct repo reads)

| Task | Source | Notes |
|---|---|---|
| TAPI-020 — SonarQube gate wired into Jenkins CI/CD | terra-api TASKS.md | Row corrected this pass (was stale "Planned") — confirmed Done via HUB_STATE's Jenkins evidence, fixed in the repo doc itself. |
| TAPI-025 — Grant `ops:read` scope to Will's prod account | terra-api TASKS.md | Row's own Status column: "Closed 2026-08-09." Fully resolved via direct SSM access; the one-shot bootstrap controller was the fallback path, not what actually shipped. |
| THQ-005 through THQ-017 (gold rollout, circuit backdrop, resort rebuild, nav menu, visualizer transparency/theme-sync, THQ-017's FE-parity port, etc.) | terra-hq-site TASKS.md | All 13 rows corrected this pass (was stale "Done — pending commit") — confirmed pushed and merged (`29d269c`), zero divergence from origin as of this machine's fast-forward pull. |
| terra-hq-site git sync | This machine | Was 2 commits behind `origin/main`; fast-forwarded clean. |

## Known Contradictions — status as of 2026-08-30

All 4 contradictions resolved 2026-08-23 (JVM heap caps, ADR-012/Phase B, ROMS EC2/Phase D, commit
terra-hq-site's pending items) remain correctly resolved — re-verified against the live Notion pull
this pass, no regression. The two same-repo doc contradictions found in that pass (TAPI-020,
THQ-005–017) are now also fixed, directly in the repos, per "What changed" above.

**Still left open, not closed — a scope call, not a factual correction:** "Phase A: Terra API
branch consolidation" (see High Priority table above).

**Not independently re-verified this pass (would need a live browser, not a file read):**
- Phase C (login/visualizer live-endpoint verification)
- THQ-001 and THQ-002's "likely already shipped" claims
