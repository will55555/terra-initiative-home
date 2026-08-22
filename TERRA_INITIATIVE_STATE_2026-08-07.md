# Terra Initiative — Consolidated State & ROMS Redeployment Readiness
<!-- Generated 2026-08-07. Read-only research pass. Sources: Notion (ADRs + project pages),
     local DEV_LOGs/TASKS.md/CLAUDE.md across all 6 repos, HUB_STATE.md, git log/status. -->

## 1. Executive Summary

Terra API's shared infrastructure — auth (JWT), health/quarantine orchestration, rate limiting,
audit logging, feature flags, Postgres/Redis, CI/CD with a dedicated Jenkins box, and a SonarQube
quality gate — is genuinely built and live in production, verified via HUB_STATE, TASKS.md, and
Notion ADRs that all agree with each other on this point. terra-api-fe (customer dashboard) and
terra-hq-site (public visualizer) both consume a shared `ecosystem-health` contract and are
feature-complete for a single-service ecosystem, but both have real uncommitted work sitting
locally (terra-api-fe: TFE-502-related redirect/test files; terra-hq-site: a pending tube-refresh
bug fix and file reorganization). **CORRECTED 2026-08-07 (post-publish fix):** this doc originally
claimed ROMS's Jenkinsfile had its Push-to-Docker-Hub/Deploy stages commented out, contradicting
ADR-005 — that was a misread of stale comment headers, not the actual code. Verified directly via
`git log -p` and `git diff` against the working file: all 7 stages, including push and deploy, are
live and uncommented, enabled since 2026-05-03/04 and untouched since. Comments fixed in the
Jenkinsfile the same day. **The single most important thing standing between "now" and "ROMS
redeployed and visible in the visualizer" is now purely infrastructural: does the original ROMS EC2
instance/Elastic IP still exist, or does this start from a fresh EC2 provision** — since every
other piece (Terra API's public health endpoint, the visualizer's tier-coloring, ROMS's own fully
enabled Jenkinsfile) is already built and just needs a live target to point at. Will is checking
AWS Global View directly; no AWS console/CLI access was available in this research pass.

---

## 2. Per-Project Sections

### Terra API
**Done:** Phases 1–6 complete (Notion proxy removed/replaced, JWT self-issued auth, health/
quarantine orchestration ADR-005, rate limiting ADR-006, audit log bus ADR-007 with Postgres
backfill, feature flags ADR-008, Redis migration, full Jenkins CI/CD with branch-tiered
staging/prod deploy). TAPI-013 prod-outage hardening (heap caps, CloudWatch alarm, swap) shipped
and verified 2026-08-02. TAPI-014 public unauthenticated `GET /api/v1/ecosystem/public-health`
endpoint shipped 2026-08-03 — this is the endpoint the visualizer needs and already has. TAPI-017
(ADR-012 operator endpoints against the `OperatorAccess` gate) closed 2026-08-04, `/internal` page
confirmed rendering live data. TAPI-018 (Postgres backup automation, S3) live-verified 2026-08-03.
TAPI-019 (Jenkins moved to its own dedicated EC2 box) closed 2026-08-05. **TAPI-020 (SonarQube
quality gate) closed 2026-08-07** per HUB_STATE and CLAUDE.md — `jacoco` plugin, `sonar-token`
credential, SonarCloud Automatic Analysis disabled in favor of CI-based analysis, GitHub webhook
wired, full pipeline green on every push to `master`.

**Genuinely open:**
- **TAPI-021 (EC2 right-size back toward t3.micro)** — Active Task per HUB_STATE and TASKS.md.
  Remaining sub-items: staging on-demand (biggest win), Alpine JRE base, trim snapd/SSM.
- **TAPI-023 (OS-level security patching automation, ecosystem-wide)** — TASKS.md: "Planned."
  Sequenced after TAPI-021 so it covers Jenkins's new box too.
- **TAPI-022 (domains + TLS for all ecosystem endpoints)** — TASKS.md: "Planned." Explicitly NOT
  satisfied by Jenkins's 2026-08-07 public-IP access (HUB_STATE is emphatic on this point — "NOT
  TAPI-022 closing... corrected 2026-08-07 after being wrongly marked closed earlier the same
  session"). Everything is still raw IP:port, ROMS included.
- **ADR-013 (Customer Identity & Login Strategy)** — Proposed, unscheduled (2026-08-07), the
  newest ADR. In its own words: "Deliberately unscheduled... last piece of the current Terra
  API/CI-CD work queue — after TAPI-021 and TAPI-023 close," blocked on "pending frontend rework"
  and "pending design rework" (both TBD scope). Also: "the `/internal` operator gate (ADR-012)
  also has no way to provision a real operator identity until an identity system exists."
- **ADR-012 (Internal Operator Dashboard)** — Proposed (2026-08-02), not Accepted. Endpoints are
  built (TAPI-017) and the `OperatorAccess.isOperator()` gate is live, but the ADR's own text says
  "An operator identity also does not exist yet: the seeded customer is `role=customer` with
  `read,write`, so nobody can currently pass the gate. Correct until an operator account is
  provisioned." Also still deferred, no trigger: per-customer drill-down, historical timelines
  (blocked on ADR-007's audit read API), any write actions.
- **ADR-002 (Notion → Obsidian Selective Push)** — Status "Designed, not built." Every checklist
  item in its own Build Status table is unchecked (n8n workflow, Notion DB property, RAG-Ready
  folder, Ollama ingestion all "Not built"/"Future"). Low urgency, not ROMS-relevant, but genuinely
  stale-open since 2026-05-22 with no recent activity.
- **Open Blocker (minor, per CLAUDE.md):** Docker port 8082 fix no longer blocked but
  `docker-compose build --no-cache` rebuild "not yet done, do whenever convenient."

**Contradiction found (needs Will's judgment call):** TASKS.md lists **TAPI-020 (SonarQube gate)
as "Planned"** (row status literally says "Planned"), while both CLAUDE.md's Current State and
HUB_STATE.md explicitly say it was "fully closed 2026-08-07" with concrete verification detail
(jacoco, sonar-token credential, webhook, green pipeline). This reads like TASKS.md's row simply
wasn't updated after the work shipped — HUB_STATE and CLAUDE.md agree with each other and carry
dated, specific verification language, so they're more likely current — but the task row itself
should be corrected rather than assumed.

---

### terra-api-fe
**Done:** Full auth shell (login, JWT bearer attach, protected routes), same-origin deploy wiring
into terra-api's Jenkins pipeline (TFE-201, live-verified 2026-08-02), backend health/entitlement
consumption (TFE-301/302/303), and a full visualizer port (TFE-401/402/403) — copy-paste parity
with terra-hq-site's Phase 5 Three.js implementation, cube filtering by customer entitlement,
health-tier coloring. TFE-501 (unauthenticated SPA reachability) done. Dashboard restyled with a
Montfort-referenced structural pass 2026-08-04.

**Genuinely open:**
- **TFE-502** — "Redirect unauthenticated users to login instead of leaving them on a broken
  page." TASKS.md: unchecked. **Git status confirms active work in progress right now**:
  `src/components/ProtectedRoute.js`, `src/pages/Login.js`, and `src/App.css` are all modified but
  uncommitted, alongside new untracked test files `ProtectedRoute.test.js` and `Login.test.js` —
  this strongly suggests TFE-502 (and possibly part of TFE-503) is mid-flight, not merely queued.
- **TFE-503** — "Expand frontend test coverage." TASKS.md: unchecked, and HUB_STATE (2026-08-04)
  says "10/12 modules untested." The new test files above are a partial start.
- **Not production-ready** per HUB_STATE: "only ever run via Docker dev compose / CRA dev server,
  no real ROMS/PIOS deployment to test against" — this is the direct dependency on ROMS actually
  being redeployed.
- **Refinement Notes (2026-08-06, DEV_LOG.md)** — a deliberately deferred future UX pass: redesign
  dashboard layout/hierarchy, redesign login experience, replace one-page flow with navigable
  tabs/sections. Explicitly scoped as its own future phase, not current work — but real and on
  record, and referenced by ADR-013 as one of the two blockers on customer-identity work.
- **8 uncommitted files as of 2026-08-04 per HUB_STATE** (TFE-401 rework + dashboard restyle) —
  git status today (2026-08-07) shows a *different* uncommitted set (App.css, ProtectedRoute.js,
  Login.js, DEV_LOG.md, TASKS.md, two new test files), meaning either the 2026-08-04 batch was
  committed since and new work has started, or HUB_STATE's file list is stale. Worth a `git log`
  check before assuming which.

**Contradictions found:** None beyond the note above about which uncommitted-file set is current
— not a real contradiction, just HUB_STATE being 3 days behind current git status.

---

### terra-hq-site
**Done:** THQ-001 (visualizer integration architecture) and THQ-002 (graduated health-tier
coloring, replacing binary connected/disconnected) both shipped and committed (`bf8d54c`,
2026-08-03) — the visualizer already polls Terra API's public-health endpoint and colors ROMS/PIOS
by real tier. Montfort-referenced structural pass (numbered badges + scroll-reveal) shipped across
11 of 13 pages, committed 2026-08-04 (`e0699aa`).

**Genuinely open:**
- **THQ-003** — pipeline extension tubes freeze their connected-state color at creation time and
  never refresh on later health updates (`createPipelineExtension()` uses `tube.userData.cube`
  singular, which the live-refresh loop doesn't recognize). A candidate fix is drafted and
  syntax-checked per HUB_STATE but explicitly **NOT visually confirmed**, and deliberately left
  uncommitted, local to the `test` machine only. This directly affects whether the visualizer will
  correctly show ROMS's live tier once ROMS starts reporting — a real dependency for the
  redeployment goal, not cosmetic.
- **THQ-004** (backlog) — add the public visualizer to the broader site experience once ROMS/PIOS
  are live and confirmed. Directly gated on the ROMS redeploy.
- **terra-hq-site-adr-003 (Multi-Tier Auth & Route Gating)** — Proposed, not Accepted. Its own
  "Open Questions" section says exact route-gating mechanics depend on ADR-010 landing first
  (ADR-010 IS Accepted, so this may just need re-review, not fresh design). Build Status table:
  `/customer` "Not started" (trigger: second Terra product with paying customers — i.e., ROMS
  again), `/internal` "Not started" (trigger: Google Workspace live, deliberately deferred),
  `/investor` "Placeholder only... 48+ months out."
- **Uncommitted local changes** (git status, today): TASKS.md modified; four image files
  moved/deleted from repo root into `archive/`; new untracked `Webpage design ideas/` directory
  and `design and improvement ideas.md`. Looks like in-progress file cleanup, not a bug — but
  uncommitted.

**Contradiction found:** None material. HUB_STATE and local TASKS.md/devlog agree closely.

---

### terra-jenkins
**Done:** Extracted into its own repo 2026-07-21 (out of terra-api). CRLF entrypoint bug fixed +
`.gitattributes` hardened (`01e80dc`, 2026-07-26). Given its own dedicated EC2 box 2026-08-05
(TAPI-019) and public IP access 2026-08-07 (Elastic IP `3.211.62.86`, security group opened on
port 8090). SonarQube Cloud CI wired 2026-08-07.

**Genuinely open:** No dedicated DEV_LOG/TASKS.md exists in this repo (by design — HUB_STATE notes
this is neutral shared infra, tracked via TAPI task IDs, not its own prefix). Nothing found that
isn't already captured under Terra API's TAPI-019/020/021/022/023. One thing to flag: **it is now
publicly reachable on the internet** (`3.211.62.86:8090`) with no TLS (that's TAPI-022's still-open
scope) — worth confirming Jenkins itself has adequate auth in front of it, since this wasn't
explicitly called out as checked in this pass.

---

### ROMS (restaurant-order-management-system)
**Done (per its own ADRs/Notion, all dated May 2026):** ROMS ADR-001–005 all "Accepted." ADR-005
specifically describes a complete, production CI/CD pipeline: Jenkins on a dedicated EC2 box,
Docker-outside-of-Docker, a 7-stage pipeline (Checkout → Build BE → Test BE → Build FE → Test FE →
Build Docker Images → Push to Docker Hub → Deploy to EC2), deployed via SSH agent to a t3.small
EC2 instance with an Elastic IP and Cloudflare DNS/proxy in front. Its own text: "Full
production-proven: every stage has already failed at least once and been fixed... this is a
battle-tested reference, not a theoretical design." The ROMS Notion project page (last synced
2026-05-30) says "Deployment complete and stable... Jenkins CI/CD pipeline operational" and frames
ROMS as in intentional "maintenance mode... portfolio reference." Locally, Phase 11 (WebSocket live
order updates), Phase 13 (Address CRUD), Google OAuth login, and 31 frontend tests are all
documented complete in DEVLOG.md.

**Genuinely open / contradicted:**
- **The single biggest gap: is the EC2 instance still there?** HUB_STATE (2026-07-29 finding,
  unresolved as of 2026-08-07): "the ROMS EC2 instance may no longer exist... the us-east-1 console
  listed 'Instances (1)', `terra-api-server` only, no ROMS box." Deliberately not chased yet —
  "Will's call to resolve it when ROMS integration actually starts." **This report could not verify
  this either way — no AWS console/CLI access was used in this pass.**
- **RESOLVED 2026-08-07 (this pass's read was wrong):** Original finding claimed the Jenkinsfile's
  Push-to-Docker-Hub and Deploy stages were commented out, contradicting ADR-005. Verified directly
  via `git log -p -- Jenkinsfile` and `git diff <last-commit> HEAD -- Jenkinsfile`: **both stages
  are live, uncommented, functional code**, enabled in commits `9bb9016`/`7d86ef3`/`22fcba9`/
  `20c34fb` (2026-05-03/04) and unchanged since. The stage *header comments* still said "currently
  disabled" — stale leftover text from before those enable commits — which is what caused the
  misread (twice, independently, in the same session). Comments corrected in the Jenkinsfile
  2026-08-07. **No contradiction exists between ADR-005 and the actual pipeline.**
- **Deploy target IP in the Jenkinsfile** (`ubuntu@3.135.55.219`) is a single hardcoded address —
  unverified whether this instance/IP still exists, separate from the "may no longer exist"
  question above (that finding referenced the AWS console generally; this is the specific IP the
  pipeline would actually SSH to).
- **`roms-key.pem` sits in the repo's working directory** (confirmed present via `ls`) but IS
  correctly gitignored (`git check-ignore` confirms `.gitignore:6:*.pem` covers it, and `git
  ls-files` shows it was never tracked) — not a live secrets leak, but worth noting it exists
  on-disk locally.
- **ROMS-001/002/003/004/005** (TASKS.md, all unchecked) — see Section 3 below for the full
  checklist derived from these plus HUB_STATE plus the findings above.
- **Resort Deployment Tracker** (separate Notion page, last updated 2026-06-15): reframes ROMS's
  actual go-to-market as in-house first — deployed inside Terra Real Estate's own resort/hotel
  properties, not sold externally. West Region Resort (Cameroon) is the first target site, status
  "Planning," no target go-live date. This is a distinct, later-stage goal from "redeploy ROMS and
  show it in the visualizer" — worth not conflating the two. The visualizer goal is infrastructure
  validation; the Resort Tracker is the actual business deployment, unstarted.

---

### PIOS
**Done:** Design-phase only. ADR-001–010 (capital governance principles) Accepted 2026-01-17.
ADR-011–014 (event-sourced write path, repository pattern, event schema versioning, projection
rebuild strategy) all Accepted. No code exists yet — HUB_STATE: "Design phase — no coding until
ADR-013 resolves."

**Genuinely open:**
- **ADR-013 (Event Schema Versioning) is the gating decision** per HUB_STATE's PIOS section — but
  the ADR index lists ADR-013 as "Accepted," which appears to directly contradict HUB_STATE's
  framing of it as still-unresolved and gating. **This is a contradiction worth flagging
  explicitly** — either HUB_STATE's PIOS section is stale (most likely, since Terra API/ROMS
  sections were updated 2026-08-07 but PIOS wasn't touched in the same pass) or the ADR index's
  "Accepted" label doesn't reflect the same resolution HUB_STATE is waiting on. Not independently
  verified against ADR-013's own page text in this pass (not fetched — out of scope of the ROMS
  redeploy question, flagged for Will's awareness only).
- **ADR-015 (Consumer Capital Layer — cashback ingestion, CRR tracking)** — "Accepted (design-level)"
  but its own text is explicit: "Treat the Decision section as directionally accepted, not
  implementation-ready." Open items: exact Gmail parsing rules, the Make/n8n workflow itself, the
  CRR formula (undefined), error-handling model. Sequenced after PIOS MVP and after Terra API
  Phase 3's WebSocket relay — i.e., after PIOS doesn't yet exist and after a Terra API phase that
  also isn't built. Low urgency, correctly framed by its own ADR as not build-ready.
- Not relevant to the ROMS redeploy path.

---

## 3. The ROMS Deployment Path — Ordered Checklist

Pulled from ROMS-001/002 in HUB_STATE, ROMS's own TASKS.md (ROMS-001–005), and the Jenkinsfile/
ADR-005 contradiction found above. Ordered by dependency, not by HUB_STATE's original numbering.

1. **Resolve whether the original ROMS EC2 instance/Elastic IP still exists.**
   Check AWS **Global View** (not just us-east-1 — the 2026-07-29 finding only checked N.
   Virginia) for any running or stopped instance. Also check **EBS Snapshots** (anything to
   restore from) and **Elastic IPs** (an unassociated one both confirms the box existed and is
   quietly billing). The Jenkinsfile's hardcoded deploy target (`3.135.55.219`) is the specific IP
   to check first. *(ROMS-003 in TASKS.md backlog; HUB_STATE flags this as the first-check item.)*

2. **Branch the decision on step 1's outcome:**
   - If the instance is recoverable: restart it, verify it still boots ROMS's stack (Spring Boot +
     Kafka + Postgres + Redis on t3.small, per ADR-005), confirm Docker/Docker Compose versions
     still match what the Jenkinsfile expects.
   - If not recoverable: provision a fresh EC2 instance matching ADR-005's spec (t3.small minimum
     — ADR-005 notes t3.micro OOM'd Kafka's JVM; Ubuntu 24.04; Kafka heap capped at
     `-Xmx512m -Xms256m` to coexist with Postgres/Redis/Spring Boot on 2GB). *(ROMS-004.)*

3. ~~Reconcile Jenkinsfile stages against ADR-005~~ — **RESOLVED 2026-08-07, not real work.** Both
   stages are already live/enabled (verified via git history); only stale comment headers needed
   fixing, done. Real remaining question: confirm `DOCKERHUB_CREDENTIALS_ID`/`server-ssh`
   credentials are still valid in whatever Jenkins instance ends up running this — check when
   ROMS-002's migration to the shared Jenkins box happens (step 4 below).

4. **Migrate ROMS's Jenkins to the shared Terra Jenkins EC2** before redeploying, per ROMS-002:
   back up the ROMS Jenkins data volume, inventory jobs/plugins/credentials, import jobs onto the
   shared Terra Jenkins box (`terra-jenkins`, already live at `3.211.62.86:8090`) without
   overwriting its existing `JENKINS_HOME`, recreate or migrate credentials (Docker Hub, SSH key
   for the new/recovered EC2 target), run a green ROMS pipeline end-to-end on the shared instance,
   then retire the old standalone ROMS Jenkins.

5. **Deploy ROMS and confirm its own health endpoint responds** on the (recovered or fresh) EC2
   instance.

6. **Wire ROMS's heartbeat to Terra API.** Terra API's `terra.roms.*` config block already exists
   (commented/aspirational in `application.yaml` per CLAUDE.md) — uncomment and point it at the
   real ROMS instance's address; confirm `POST /api/services/heartbeat` (ADR-005's quarantine
   protocol) receives real heartbeats and ROMS moves out of "never reported" state in the
   `QuarantineService` registry.

7. **Verify `GET /api/v1/ecosystem/public-health`** (TAPI-014, already live) correctly reflects
   ROMS's real tier once heartbeats are flowing — this endpoint and its consumer (terra-hq-site's
   visualizer, THQ-002) are both already built and don't need new work, only a live data source.

8. **Fix THQ-003 before declaring this done, not after.** The pipeline-tube refresh bug means
   ROMS's cube may show a stale/frozen connection state even after real heartbeats start — this
   was found live-testing against a real ROMS heartbeat and the fix is drafted but not visually
   confirmed or committed. Confirming this now avoids shipping a redeploy that looks broken in the
   visualizer for a UI reason unrelated to ROMS's actual health.

9. **Confirm the public visualizer on terra-hq-site actually shows ROMS live** — the end goal
   stated in this task. terra-api-fe's scoped visualizer (TFE-401/402/403) should be spot-checked
   too, since it reads the same underlying health model and is a second, independent consumer that
   would surface any entitlement-filtering issues terra-hq-site's unscoped view wouldn't catch.

10. **Optional but consistent with "launch and forget":** once ROMS is redeployed, its box has none
    of Terra API's hardening yet — CloudWatch alarms, automated Postgres backups, OS patching — all
    of which exist as patterns/scripts on Terra API's box (TAPI-013/018/023) but haven't been
    applied to ROMS. Not blocking for "visible in the visualizer," but worth sequencing shortly
    after if the goal is a durable redeploy, not just a demo.

---

## 4. All Open ADRs and Their Blockers

| ADR | Title | Status | What's Blocking | Anything Else Depends On It? |
|---|---|---|---|---|
| terra-api-adr-002 | Notion → Obsidian Selective Push Workflow | Designed, not built | n8n workflow never built; Notion DB property never added; RAG-Ready folder never created; Ollama ingestion is a future dependency (originally "Jul 2026," already past) | No — isolated, not on the ROMS critical path |
| terra-api-adr-012 | Internal Operator Dashboard | Proposed (2026-08-02) | No operator identity exists to actually use the built gate — "the seeded customer is `role=customer`... nobody can currently pass the gate." Also waiting on a design pass (tied to terra-hq-site design work, TBD) | Yes — ADR-013 is explicitly blocked on the same missing-identity piece |
| terra-api-adr-013 | Customer Identity & Login Strategy | Proposed, unscheduled (2026-08-07) | Deliberately unscheduled — sequenced after TAPI-021/023 close; blocked on "pending frontend rework" (scope TBD) and "pending design rework" (TBD, tied to terra-hq-site design queue) | Yes — ADR-012's operator identity gap depends on this too; ADR-011's entitlement table stays empty until this resolves |
| terra-hq-site-adr-003 | Multi-Tier Auth & Route Gating Strategy | Proposed (2026-07-16, folded 2026-08-01) | Exact route-gating claim mechanics need re-review against ADR-010 (which IS now Accepted, so this may be closer to resolvable than its "Proposed" label suggests). `/customer` section trigger: second Terra product with real paying customers — i.e., waits on ROMS. `/internal` trigger: Google Workspace live (no timeline). `/investor`: no trigger, 48+ months out | `/customer` section directly depends on ROMS having real customers, tying it to the same redeploy question |
| pios-adr-015 | Consumer Capital Layer (Cashback/CRR) | Accepted (design-level only) | Its own text: "not implementation-ready." Gmail parsing rules, Make/n8n workflow, CRR formula, and error-handling model all unspecified. Sequenced after PIOS MVP (doesn't exist) and Terra API Phase 3's WebSocket relay (not built) | No — PIOS has no code yet at all |
| pios-adr-013 | Event Schema Versioning | Listed "Accepted" in the ADR index, but HUB_STATE's PIOS section calls it the still-open gating decision ("no coding until ADR-013 resolves") | **Contradiction, not independently resolved in this pass** — see PIOS section above | Possibly all of PIOS's next coding phase, depending on which source is current |

*(terra-api-adr-001, 003–011 and viz-adr-001–005 and terra-hq-site-adr-001/002 and ROMS-adr-001–005
and pios-adr-001–012/014 are all "Accepted" with no open-blocker language found in their content —
not repeated here.)*

---

## 5. Everything Else Open, Uncategorized

- **terra-api TASKS.md TAPI-020 row says "Planned"** while CLAUDE.md/HUB_STATE say it shipped
  2026-08-07 — see Section 2 contradiction above. Small doc-hygiene fix, not urgent, but exactly
  the class of stale-doc problem flagged as a past pain point.
- **PIOS ADR-013's status label discrepancy** between the ADR index ("Accepted") and HUB_STATE's
  PIOS section (treats it as the still-open gating decision) — flagged in Section 4, not resolved.
- **terra-api-fe's uncommitted files today (2026-08-07)** don't match the file list HUB_STATE
  recorded on 2026-08-04 — likely just HUB_STATE being a few days stale rather than a real
  problem, but means HUB_STATE's terra-api-fe "Blockers"/uncommitted-file note should be treated
  as informational, not current, until cross-checked with `git log`.
- **terra-hq-site has uncommitted image-file reorganization** (four images moved root → `archive/`,
  two new untracked docs) that isn't mentioned in HUB_STATE at all — minor, but real uncommitted
  local state nobody has flagged yet.
- **Jenkins is now publicly internet-reachable** (`3.211.62.86:8090`, opened 2026-08-07) with no
  TLS yet (TAPI-022 still open) — not itself a documented problem, but worth Will confirming
  Jenkins' own auth is solid given the newly-expanded attack surface, since this pass didn't
  verify that.
- **A dedicated `will-cli` IAM identity exists** with `AdministratorAccess` per HUB_STATE's Terra
  API Blockers line — flagged there as "narrow... or move to IAM Identity Center as a separate
  security follow-up," still open, low urgency.
- **ADR-002's Notion→Obsidian workflow** has sat at "Designed, not built" since 2026-05-22 with an
  already-past target date ("Jul 2026") for the Ollama ingestion step — a deferred-with-no-active-
  trigger item worth surfacing per the task's own instructions, even though it's not urgent and
  not ROMS-related.
- **ROMS's `roms-key.pem`** sits in the local working directory (correctly gitignored, never
  committed) — noted for completeness, not a live leak.

---

## 6. Confidence / Gaps

- **Could NOT verify whether the ROMS EC2 instance currently exists.** This requires AWS
  console/CLI access, which this pass did not have. HUB_STATE's 2026-07-29 finding (console showed
  only `terra-api-server` in us-east-1) is the most recent evidence, and it was explicitly a
  partial check (single-region), not a Global View sweep. This is the single largest unresolved
  fact blocking a concrete ROMS redeploy plan.
- **Could NOT verify whether ROMS's Jenkins credentials** (`DOCKERHUB_CREDENTIALS_ID`, `server-ssh`)
  still exist/are valid on whatever Jenkins instance ROMS was using before — no access to that
  Jenkins UI in this pass.
- **Did NOT independently fetch every "Accepted" ADR's full text** (terra-api-adr-001, 003–008,
  010; viz-adr-001–005; ROMS-adr-001–004; PIOS-adr-001–012, 014) — these were treated as settled
  based on their "Accepted" status in the ADR index and no contradicting signal found elsewhere.
  If any of these have since-added amendments with new open items (the way ADR-005/009/011 did),
  this pass would have missed them unless HUB_STATE or a DEV_LOG also referenced the gap.
- **Did NOT fetch PIOS ADR-013's own page text** — the contradiction flagged in Section 4/5 is
  based on the ADR index's status label vs. HUB_STATE's framing only; resolving it would need
  reading ADR-013 directly.
- **Did NOT check whether Jenkins (terra-jenkins, now publicly reachable) has adequate
  authentication** in front of it — flagged as a gap, not verified either way.
- **Local DEV_LOG reads used tail/grep sampling on very large files** (terra-api's DEV_LOG.md,
  ROMS's DEVLOG.md) rather than reading every line from the beginning — recent entries (last
  several sessions, covering roughly the last month) were read in full; older Phase 1–10 history
  in each file was sampled/grepped rather than read end-to-end, since HUB_STATE and TASKS.md
  already provided a verified-current summary of that older work.
- **git commit timestamps in this environment show dates in 2026**, consistent with the stated
  current date (2026-08-07) — no clock-skew concerns found.
