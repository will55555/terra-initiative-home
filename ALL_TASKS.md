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

## Direct verification pass (2026-10-04) — checked every open task against current code/content

Per Will's request to confirm these are "actual current tasks and not things already addressed."
Re-fetched all 5 repos (clean, zero divergence from origin, no change since 2026-10-03). Checked
the most re-verifiable claims directly against code/content rather than trusting prior doc text.

**Clarified first:** Will asked about "the migration" being complete. Two different migrations
share the ROMS→OMS name: (1) OMS-006–012, the codebase/package/Docker-image/repo rename — **genuinely
done**, confirmed via `git log`/`git show` on commit `fd4c732` ("rename ROMS to OMS across backend
package, frontend, and build config," 2026-08-10) — matches `oms/TASKS.md`'s own Closed status,
already correctly reflected everywhere in this file. (2) OMS-013, the `service_id` wire-protocol
*value* sent in heartbeats + stored in prod's `customer_service_access` table — **confirmed still
open**: `TerraHeartbeatScheduler.java:32` still has `SERVICE_ID = "roms"` (not touched by the
`fd4c732` rename — that commit moved the file into the new package but didn't change this string
literal), `terra-api-fe`'s `domainConfig.js`/`productConfig.js`/`terraScene.js`/
`useEcosystemHealth.js` all still use `'roms'` throughout, and `terra-api`'s `seed-dev.sql` +
several test files still reference `service_id: "roms"`. No evidence anywhere the prod DB `UPDATE`
was run either. OMS-013 stays listed as open below — this was already correct, no file change
needed for this item specifically.

**Real finding — THQ-002 and THQ-003 are stale, describe a dead file:**
`terra_api_visualizer_phase5.js`/`.html` (the file both tasks are about) is **archived**, not at
the repo root — confirmed via direct file search: both files live at
`Assets/archive/terra_api_visualizer_phase5.{js,html}`. No live page references them: grepped
`index.html`/`terra_initiative.html`/`terra_tech.html`, zero hits. This matches
`terra-hq-site/CLAUDE.md`'s own documented 2026-08-09 decision ("terra_api_strategy.html archived
... this static HTML copy is a frozen reference only, no longer linked from anywhere on the live
site") — the visualizer moved into `archive/` alongside it the same day. All live "Terra API" links
on the site (`index.html` lines 488/592/1032, `terra_tech.html` line 229) point directly to
`https://api.terra-hq.com/` — the real, live, production visualizer, which is terra-api-fe's React
port. **Directly confirmed terra-api-fe's copy does NOT have THQ-003's bug**: `terraScene.js` lines
440-441 show `tube.userData.cube1 = cube` / `tube.userData.cube2 = cube` as live, uncommented code
— the fix is fully applied there, unlike the archived hq-site copy's half-applied version. THQ-002
(graduated health-tier coloring) is almost certainly also moot for the same reason — the live
visualizer customers/investors actually see is the terra-api-fe one, not the archived file. **Both
THQ-002 and THQ-003 moved to a new "Likely Stale / Needs Closing" section below** rather than
closed outright — closing a repo's own TASKS.md row isn't a call to make unilaterally, but leaving
them in the main Low Priority / In Progress tables as if still meaningfully open misrepresents
their status.

**Real finding — "Set up Cloudflare Access on terra-hq.com" (High Priority) is likely obsolete,
not just open:** the live site's own content states this is a **settled architecture decision**
against the approach the Notion task asks for. `index.html` line 751: "Route gating via Terra API
JWT (not Cloudflare Access)." `terra_tech.html` line 194: "protect terra-hq.com's /internal pages
via Terra API's own JWT — explicitly not Cloudflare Access, per the locked architecture decision."
**Contradicted by a different file in the same repo**: `terra-hq-site/CLAUDE.md` (last touched
2026-09-14, newer than index.html's 2026-08-20) still says "Cloudflare Access NOT enabled... setup
pending" — CLAUDE.md itself reads stale against the live site content, not the other way around.
This task stays in High Priority below but reworded to flag the contradiction — it's not a simple
"still Todo," it may be a Notion row describing an approach that's already been architecturally
rejected in favor of JWT gating. Needs Will's call, not a doc fix, since CLAUDE.md and index.html
actively disagree.

**Everything else directly spot-checked — confirmed accurate, no changes needed:**
TAPI-021/022/023/024 (`Planned`), TAPI-025 (`Closed 2026-08-09`), TAPI-026 (`Backlog`) — all
re-read directly from `terra-api/TASKS.md`, unchanged since 2026-10-03. TFE-502/503/601-605 —
re-read directly from `terra-api-fe/TASKS.md`, all still open/unchecked, unchanged. THQ-005–017
(`Done`), THQ-001/004/018 (`In Progress`/`Planned`) — re-read directly from
`terra-hq-site/TASKS.md`, unchanged. SEC-001 — confirmed it's real, live content on
`terra_tech.html` ("migration to Bitwarden in progress," not yet confirmed closed) rather than a
phantom Notion-only reference, matching what this file already said.

## Resolved This Pass (2026-10-04) — confirmed by Will, closed

All three items flagged in the "direct verification pass" above were confirmed and closed same day:

| Task | Resolution |
|---|---|
| THQ-001 — Visualizer frontend integration ADR + migration plan | **Confirmed by Will**: the migration happened — visualizer lives in `terra-api-fe` now, hq-site's copy is "only kept as a relic." Closed in `terra-hq-site/TASKS.md` (`82617ef`). |
| THQ-002 — Visualizer cube color: graduated health tier | Closed as moot — archived file, live coloring already shipped in terra-api-fe (TFE-403). Closed in `terra-hq-site/TASKS.md` (`82617ef`). |
| THQ-003 — Pipeline extension tubes freeze connected state | Closed as moot — archived file, live fix already applied in terra-api-fe's `terraScene.js`. Closed in `terra-hq-site/TASKS.md` (`82617ef`). |
| Set up Cloudflare Access on terra-hq.com | **Confirmed by Will**: "stale since terra-api-fe live — Cloudflare Access became kind of stale because secure pages need password for entry now" (JWT-based gating via Terra API, already live). Closed in Notion Tasks DB directly, with a note that `terra-hq-site/CLAUDE.md`'s "pending" framing was the actually-stale side. `CLAUDE.md` also fixed same pass (`bea97ac`) — replaced its stale Current State/Next Action/Open Blockers sections (dated 2026-07-18) with current status + a pointer to TASKS.md instead of restating task content that drifts. |
| ~~Decide entity home for goods line~~ | **Closed 2026-10-04 in Notion Tasks DB** (row previously left open while this file only flagged the staleness). Resolution note added pointing to the actual decision page — bamboo/calabash is cross-vertical, routed per-product. |
| Registered Agent Renewal note (Compliance Calendar) | **Fixed 2026-10-04** — the "x5 Wyoming entities" cost-basis note was stale inside the database row itself (`Notes` field), not just prose on the parent page as first assumed. Corrected to the Model B figure (~$125/year, Terra Inc only). |

## Full Terra Inc Notion space audit (2026-10-03) — first pass beyond the Tasks DB

**Scope:** per Will's request ("scour through the entire terra inc space, even subfiles/subfolders")
— not just the canonical Tasks DB, but every page reachable from the 🌍 Terra Inc hub (restructured
2026-09-14 into 6 sub-pages: Tech, Finance, Entities, Ops, Subsidiaries, Brand & Strategy), crawled
via 3 parallel research agents to leaf depth. **Explicitly excluded**: the personal Finance Hub
(credit repair, investment strategy, personal banking) one hop beyond Terra Inc's thin Finance
pointer page — real and substantial, but not Terra Inc scope; not reconciled here. Also excluded:
items already covered by a repo's own TASKS.md (terra-api, terra-api-fe, terra-hq-site, oms) per
the existing sections below — this audit is additive, not a re-verification of those.

**Top finding — a real, named gap, not just scattered drift:** Notion has a page literally called
["Task Queue (Manual Transfer)"](https://app.notion.com/p/3c189370d497810ab1beef13672466a9) under
Ops, explicitly built as a staging area for items "destined for HUB_STATE.md / other git-tracked
state" — a holding pen for exactly this kind of reconciliation, logged 2026-08-19, Status: Queued.
5 items have been sitting there over 6 weeks, confirmed found independently by 2 of 3 audit agents.
Most of these were *already* in this file's Low Priority section below (added 2026-08-19 from an
earlier Notion pull) — but one of them is now stale, corrected below.

**Stale item, since closed:** "Decide entity home for goods line" — Notion Tasks DB row showed
**Todo** despite being actually **resolved 2026-08-29** — see
[Entity Home Decision — RESOLVED](https://app.notion.com/p/3cc89370d497814dbde1e606a7b56b0a):
bamboo/calabash is a cross-vertical material line (not a single entity home) — individual products
route to whichever subsidiary fits (shoe → Apparel, resort hard goods → OMS/ROMS inventory). **Row
marked Done 2026-10-04**, see "Resolved This Pass" above. Follow-on opens from that resolution, not
previously captured anywhere: whether this changes the SKU/marker scheme approach (per-destination
tagging now, not one unified line — see the live draft scheme below), and legal/entity implications
of a cross-vertical product line, not yet addressed.

**New Terra-scoped findings from the full crawl, grouped by confidence:**

### Explicitly queued, not yet migrated (highest confidence — Notion's own page says so)
| Item | Source | Notes |
|---|---|---|
| Build automated invisible-comment watermarking script | [Task Queue](https://app.notion.com/p/3c189370d497810ab1beef13672466a9) | Target: `claude-skills/skills/ai-control/HUB_STATE.md`. Scope decision still open: apply to all files in a repo vs. core/business-logic classes only. |
| SKU/marker scheme — now has a live draft structure | [SKU Scheme Draft](https://app.notion.com/p/3cc89370d4978141a580cd81c8436f0c) | Already in this file below as "Define SKU/marker scheme" but with zero context — actual substance exists: a 4-segment composite tag (destination / design-line / season-drop / open) is drafted, directly downstream of the entity-home resolution above. Still open: exact segment order/delimiter, full segment list, ROMS/OMS technical integration. |

### Recurring checklists, currently unfilled (actionable now, no blocker)
| Item | Source | Notes |
|---|---|---|
| Weekly Review checklists (Entities, Ops, Tech sub-pages) | Entities / Ops / Tech pages | Monday-cadence checklists — compliance calendar check, documents-needing-action check, Rung 2 (1099) status check, active-project blockers check. Currently unchecked across all three (found independently by 2 agents). "Next Week Focus" on Ops page is blank (3 empty priority slots). |

### Entity formation — Rung 2 trigger status needs a direct check
| Item | Source | Notes |
|---|---|---|
| Terra Services LLC formation — may now be actionable | [Entity Strategy](https://app.notion.com/p/38d89370d497814192b7c1deb155e8e8) + [Formation Checklist](https://app.notion.com/p/36e89370d4978114a9cbfaf161282279) | Snorkel AI 1099 gig was secured 2026-07-18 (the Rung 2 trigger), but task work hadn't started as of that note ("time going to Terra API instead") — worth checking current status, since this gates a whole sequential checklist (choose state/agent → file Articles → EIN → bank account → Operating Agreement → Stripe → bookkeeping). **Data-quality flag**: the Formation Checklist page still says "Virginia" and references the retired "WT Ventures" plan in its state/fee references — needs a Maryland correction pass before anyone executes it (canonical is MD per Model B). Same gate gives the [Business Banking checklist](https://app.notion.com/p/38d89370d49781bf916ee1457b83105f) (Chase Business Checking/Savings, auto-sweep rule, draw cadence) — currently fully blocked on the same first-1099 trigger. |
| Q3 Estimated Tax Payment — possibly overdue, not confirmed | [Compliance Calendar](https://app.notion.com/p/36f89370d497816db646f2cdca6a4f17) | Due date 2026-09-15 has passed (today is 2026-10-03); row still shows "⏭️ Deferred." Worth a direct check — may just be a stale Notion row if it was actually paid, or a genuinely missed item. |
| Registered Agent Renewal note is stale | Compliance Calendar | Cost-basis note still says "x5 Wyoming entities," a pre-Model-B assumption. Not urgent (renewal itself is due 2027-01-01) but worth a text fix next time that row is touched. |

### Per-subsidiary open items (live, not blocked)
| Item | Source | Notes |
|---|---|---|
| Terra Agriculture — land title formalization | [Terra Agriculture](https://app.notion.com/p/38189370d497816daa98ce7dc961b099) | Flagged by the page's own Next Action list as the lead blocker before capital goes into permanent structures. Parallel, non-blocking: local operator search, Starlink coverage/pricing confirmation, Cameroon drone import regs, Parcel B power availability. |
| Terra Agriculture — Fruit Value Chain brainstorm cluster | [Fruit Value Chain](https://app.notion.com/p/3cc89370d497811c916df726b0484c14) (+ 6 sub-pages) | Entirely new since the last pass (logged 2026-08-29, not in Tasks DB). Real open items: crop sequencing/orchard candidate list, land/hive plot mapping against real Parcel B geography (100–150yd pollination-proximity guidance exists as a draft constraint), capital planning for juice/pulp/wine processing tiers. One item resolved, NOT open: "cassmango" species ID was flagged as a research question on the parent page but a child page explicitly marks it **"Proprietary, Not Disclosed, Drop This Thread"** — a direct self-contradiction within the same doc cluster; treat the child page as authoritative (closed), not the parent's phrasing. |
| Resort/Terra Real Estate — permit/regulatory research | [Resort Concept](https://app.notion.com/p/36f89370d4978115be61e48e712461c4) | Explicitly flagged "should happen before any spend occurs" — not yet started. Also open: real local supplier contacts (CCIMA Bafoussam branch suggested as starting point), architect brief prep, Phase 1 cost estimation with real local quotes. |
| Resort/OMS — deployment stage unresolved | [Resort Deployment Tracker](https://app.notion.com/p/36e89370d49781849ebbe20a7d754a08) | Stage = "Planning," Target Go-Live = unset. This is the named Phase-1 OMS-in-house target (per the Terra Inc hub's own status table: "OMS live at Terra RE resort" = 🔴 Not Started, Priority 1) — real scoping work hasn't started despite being the top business priority. |
| Bamboo shoe design — subsidiary home undecided | [Idea Capture](https://app.notion.com/p/3cc89370d497818faae0cfb1dad95286) | Hand-drawn concept exists, Gemini visual-iteration planned. Open question explicitly flagged on the page: does this belong under Terra Apparel (wearables) or the Bamboo/Calabash goods line (core material)? Not resolved. |
| Terra Nkap — 4 residual open threads | [Terra Nkap](https://app.notion.com/p/3cc89370d49781f8afb4ff16e573540c) | Mostly resolved (naming-risk question has a separate "CONFIRMED FINAL" child page — worth a consistency pass, not a fresh decision). Still open: physical card manufacturing plan (concept art only), Nkap-specific regulatory mapping (general Africa regulatory path exists, not Nkap-specific), cross-check the "three functions" framing on terra_africa_strategy.html against this page (not yet verified). |

### Lower-confidence / parked (listed for completeness, not immediately actionable)
Terra Real Estate's Dual-Use Global Property concept (trigger-gated, no hours committed — already in
this file's Low Priority section as "numeric criteria" item), Terra Tech Engineering Lab purchase
decisions (monitor/scope/printer choice — "parked, Black Friday 2026 target"), PIOS Frontend Stack
ADR + PIOS Travel Layer ADR (both explicitly "write once PIOS architecture drafting starts," not
now), Terra Solar (dormant until real land development triggers it).

**Not reconciled into the main tables below** — this audit surfaced real items but most need a
scoping/triage pass (several are self-tagged Brainstorm/Research Required, several are entity/legal
decisions outside repo scope) before they're Tasks-DB-ready rows. Treat this section as the source
list for that triage, not a finished reconciliation. Re-run this full-space crawl periodically —
the Task Queue page's own existence shows this kind of drift accumulates between passes.

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
| HQ Site: Move Terra API into Terra Tech page | Coding | Medium | terra-hq-site | **Resolved 2026-10-03** — closed as `terra-hq-site/TASKS.md` THQ-019. **Initial framing was wrong, corrected directly by Will**: first pass treated this as a content/link question (all live Terra API links already pointed at `https://api.terra-hq.com/`) and wrongly concluded "no new work needed." Will corrected this — the actual ask was a visual/structural fix: `index.html`'s Divisions sequence had Terra API as its own standalone top-level puzzle piece, duplicating content already present inside Terra Tech's own card ("Terra API Shipped"). Fixed by removing the standalone card entirely (commit `5de8ea6`) plus two division-count copy corrections; Terra Tech's own card was left untouched. Full writeup in `terra-hq-site/TASKS.md` → THQ-019. Separately, confirmed terra_tech.html already has the Terra API link "integrated" via two existing spots (Products & Infrastructure card + Overview paragraph link) — no further change needed there for Terra API specifically; surfaced a different real gap on that same page instead, see the next row. |
| HQ Site: terra_tech.html product cards not all clickable | Coding | Low | terra-hq-site | **Resolved 2026-10-03** — closed as `terra-hq-site/TASKS.md` THQ-020. Surfaced while verifying the row above: 2 of 5 Products & Infrastructure cards (Terra API Visualizer, Terra CI/CD) were plain non-clickable divs while their siblings were real links — an inconsistency, not a deliberate choice. Fixed (commit `97dd2d8`): Visualizer card links to the local archived copy, CI/CD card links to the Jenkins temp URL. Also repointed the Ha'bem (OMS) card from the old static GTM strategy page to the new OMS-fe internal dashboard (commit `3919b96`) once that dashboard was built — see the OMS internal dashboard row below. |
| OMS: internal strategy dashboard built in OMS-fe | Coding | Medium | oms | **Resolved 2026-10-03** — closed as `oms/TASKS.md` OMS-019 through OMS-022. Folds `terra-hq-site/roms_gtm_strategy.html`'s static GTM content (plus a new Architecture Notes tab from `oms-expansion-sketch.md`) into a real React page, `OMS-fe/src/internal/OmsDashboard.js` + 11 tabs, gated at `/internal/dashboard` with `requiredRole="INTERNAL"` (commit `3339af7`). Deployed and live-verified same day, surfacing and fixing three real bugs in the process: the `INTERNAL` role didn't exist in the backend `Role` enum OR the live Postgres `user_role_check` constraint (OMS-020, fixed via code + direct SQL — no migration file exists for the constraint change, flagged as a real gap); `ProtectedRoute` false-redirected role-gated routes on page refresh due to a `user=null` race before the profile fetch resolved, a pre-existing bug affecting every `requiredRole` gate including `/admin/menu`/`/admin/orders`, not just this new route (OMS-021, commit `5b17b37`); the dashboard rendered inside the customer storefront's navbar (OMS-022, commit `99f21af`). Tab layout also redesigned from cramped top-row tabs to a left sidebar (commit `8f14b37`) after Will reviewed it live. All work pushed. |
| OMS/terra-api: disk-full deploy failures — build cache, not just images | Infra | High | oms, terra-api | **Resolved 2026-10-03** — the 2026-10-04-dated image-prune fix (`045fe3a`) wasn't enough; root cause was Docker's BUILD CACHE (7.4GB, untouched for 8 weeks, invisible to `docker image prune`) plus old-but-tagged image versions that `-af` doesn't remove. Added `docker builder prune -af` to both oms's Jenkinsfile (`aa9e019`) and terra-api's Jenkinsfile (`6b1e072`, both GitHub + Bitbucket), same non-blocking placement before/after each deploy. Manually cleared oms-server's disk via SSM as an immediate unblock (79%→53% used). Recurred once more the same session (disk filled again mid-deploy, SSM itself became unreachable) — root-caused to the box's root EBS volume being genuinely undersized (6.8GB) for Postgres+Kafka+Redis+2 app images, not just accumulated junk; resized to 20GB via the EC2 console + in-OS `growpart`/`resize2fs`, confirmed durable (13GB free after a full stack rebuild). |
| terra-api: stale `roms` entry stuck in quarantine registry forever | Infra | Medium | terra-api | **Resolved 2026-10-03** — closed as `terra-api/TASKS.md` TAPI-027. `QuarantineService`'s in-memory registry had no eviction path at all; the old `roms` service-id, abandoned when OMS renamed its heartbeat `serviceId` to `oms`, sat permanently ORANGE (`missed_heartbeats`) since nothing ever removed it. Added a new `@Scheduled` `evictStaleServices()` method removing ORANGE/RED entries silent for 72+ hours (`quarantine-policy.stale-eviction-hours`, new policy-driven config property per ADR-005 §4), commit `d175395`. The current stale `roms` entry is left to clear naturally over that window, by Will's choice, rather than force-cleared via a restart. |
| Visualizer: confirm location (terra-api-fe?) + record 8-cube role model | Coding | Medium | terra-api-fe or terra-hq-site | **Resolved 2026-10-03** — closed as `terra-api-fe/TASKS.md` TFE-606. Live visualizer confirmed at `terra-api-fe/src/visualizer/` (React/Three.js, live at api.terra-hq.com); terra-hq-site's copy stays archived/relic-only. 8-cube role model recorded in TFE-606: only OMS (Hospitality) and PIOS (Ventures) carry a non-null `serviceId` and report live health; the other 6 domains (Finance/Nkap, Real Estate, Agriculture, Apparel, Africa, Solar) are placeholder children with `serviceId: null`. Terra API anchor confirmed architecturally separate from the 8 — rendered at the origin, not one of the eight corners. |
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
| Define SKU/marker scheme for guest-purchasable vs. resort-owned goods items | Notion | **Has a live draft now** — see "Resolved This Pass" above (4-segment composite tag drafted 2026-08-29). No longer blocked on the entity-home decision (closed 2026-10-04, see above); still open on exact segment order/format and ROMS/OMS technical integration. |
| Build automated invisible-comment watermarking script | Notion | No repo home identified yet. |
| [Research Required] Amazon seller plan for bamboo/calabash goods line | Notion | **New since last pass** (created 2026-08-23). |
| Define numeric criteria for "decent unit in good area" — Dual-Use Global Property concept | Notion | **New since last pass** (created 2026-08-29). Unrelated personal/business real-estate concept, not a Terra repo task. |
| TAPI-026 — Manual quarantine/release from the Operator tab | terra-api TASKS.md | Confirmed **Backlog** directly — deliberately unscoped until Will designs it properly. |
| TFE-601/604/605 | terra-api-fe TASKS.md | Confirmed **Open/Backlog** directly — tab split, menu popover ideas, cube pulse animation. All deliberately shelved for future design sessions. |
| THQ-004 — Design pass follow-ups (per-subsidiary card images, puzzle-piece shape, 3D backdrop asset) | terra-hq-site TASKS.md | Confirmed **Planned**, notes only, no code yet. |
| THQ-018 — Custom circular cursor (Montfort-style), site-wide | terra-hq-site TASKS.md | Confirmed **Planned**, design discussion only. See [[reference_montfort-design]] memory. |

## In Progress

Empty as of 2026-10-04 — THQ-001 and THQ-003 (the only rows previously here) both closed this
pass. See "Resolved This Pass" above.

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
