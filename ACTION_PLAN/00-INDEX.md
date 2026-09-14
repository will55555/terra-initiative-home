# Terra Initiative — Action Plan (generated 2026-08-22, last refreshed 2026-08-30)

**2026-08-30 refresh note:** re-pulled the live Notion Tasks DB and read `terra-api`,
`terra-api-fe`, `terra-hq-site`, and `oms` TASKS.md files directly on this machine (all four are
present here, correcting this file's 2026-08-23 claim that `oms` wasn't cloned locally). Found and
fixed: `terra-hq-site` was 2 commits behind `origin/main` (fast-forwarded); `terra-api/TASKS.md`'s
TAPI-020 row and `terra-hq-site/TASKS.md`'s THQ-005–017 rows both had stale status text despite
being done/pushed (both corrected directly in-repo); OMS-013's blocker is now cleared. No Notion
rows needed correction — live drift since 2026-08-23 was purely additive (new business-brainstorm
tasks). Full detail in `ALL_TASKS.md`'s "What changed" section — treat this file's per-item detail
below as accurate as of 2026-08-23 except where superseded by that section.

**Read-only research artifact.** No code was touched to produce this (a few docs/Notion corrections
were applied with explicit approval each time — see the "Status as of 2026-08-23" section below for
what actually changed). It's a synthesis of `ALL_TASKS.md`, `HUB_STATE.md` (claude-skills), every
repo's own `TASKS.md` (`terra-api`, `terra-api-fe`, `terra-hq-site`; `oms`/ROMS's TASKS.md is NOT on
this machine — see `05-roms-oms.md`), and the live Notion Tasks DB (re-pulled 2026-08-23). Anything
still requiring code/infra changes is *for Will to execute*, not something an agent ran —
consistent with the hub's standing rule that Claude never runs build/deploy/infra commands.

**This snapshot was refreshed 2026-08-23** — Notion access came online mid-session, resolving most
of the contradictions the 2026-08-22 version could only flag. Re-read the source files before
acting on anything here if it's been more than a session or two since.

## How to use this folder

Each file below covers one project/domain. Inside each file, tasks are listed in the order Will
should tackle them, with: what it is, why it matters, exact sites/consoles to open, and the
concrete manual steps. Anything flagged "**DECISION NEEDED**" means two sources disagree or a
judgment call is required before touching anything — resolve that first, don't act on the stale
reading.

| File | Covers |
|---|---|
| [01-urgent-uncommitted-work-risk.md](01-urgent-uncommitted-work-risk.md) | **Revised 2026-08-22, updated 2026-08-23:** nothing uncommitted anywhere in the ecosystem as of this refresh — the one real find (terra-hq-site's diverged batch) is now merged. |
| [02-terra-api-infra.md](02-terra-api-infra.md) | TAPI-020/021/022/023/024/025/026 — EC2, Jenkins, domain-naming brainstorm (NEW), patching, prod DB grants, branch protection |
| [03-terra-api-fe.md](03-terra-api-fe.md) | Notion Phase C verify + TFE-502/503/601/602/603/604/605 + sign-out button UI issue (NEW) |
| [04-terra-hq-site.md](04-terra-hq-site.md) | THQ-001/002/003/004/018 — the THQ-005-018 pending-commit batch has landed (2026-08-23); visualizer fixes, design parking lot remain |
| [05-roms-oms.md](05-roms-oms.md) | OMS-013/015/018, ADR-005 amendment, brand/domain decisions (repo not cloned on this machine) |
| [06-pios.md](06-pios.md) | Learning-stack gate check, ADR duplicate cleanup |
| [07-notion-and-business-decisions.md](07-notion-and-business-decisions.md) | **Updated 2026-08-23: contradictions resolved** via live Notion access — Part A is now a record of what was fixed, not an open task list. Non-code business decisions (Part B) + domain brainstorm pointer unchanged. |
| [08-hub-and-git-housekeeping.md](08-hub-and-git-housekeeping.md) | Machine Paths + repo rename + git drift **all fixed 2026-08-23**; **branch protection (NEW)** across every repo |
| [09-terra-nkap-crypto.md](09-terra-nkap-crypto.md) | **NEW 2026-08-23, placeholder only** — Nkap crypto isn't scoped anywhere; captures what's scattered across the hub + open questions, doesn't commit to a plan |

## Recommended overall order

1. **01 — uncommitted-work risk.** Confirmed clean as of 2026-08-23 — quick read, not a blocker.
2. **08 — git housekeeping, item 1 + item 5.** The remaining git-drift pull (~5 min) and branch
   protection setup (~20 min across 6 repos) — both quick, both close real gaps.
3. **02 item 3 — domain-naming brainstorm.** Decide this before any DNS work happens, since it
   changes the shape of TAPI-022 entirely.
4. **02 — Terra API infra**, remaining sequence: TAPI-021 → TAPI-023 → TAPI-022 (now gated on the
   domain decision above) → ADR-013, with TAPI-020's repo-doc fix and TAPI-024/025 cleanup alongside.
5. **04, 03, 05, 06** — the four product repos. terra-hq-site's backlog is much lighter now that the
   pending-commit batch has landed; terra-api-fe has the new sign-out UI issue plus existing UX
   debt; ROMS/OMS is stable in prod; PIOS hasn't started.

## Resource Directory (every site/console you'll need)

### AWS Console (root: https://console.aws.amazon.com — all resources below are `us-east-1`)
| Resource | ID | Used for |
|---|---|---|
| terra-api-server (EC2) | `i-044e35066f956d506` | TAPI-021/023/024/025, right-sizing, patching |
| terra-jenkins (EC2) | `i-04ef85c382ac39269` | TAPI-023 patching, Jenkins capacity scoping |
| roms-server / oms-server (EC2) | `i-04f3abfb579f2bd1d` | OMS tasks, TAPI-023 patching |
| terra-api-be security group | `sg-063cee0bc65d872c9` | Port 8081 cleanup (TAPI-021), CORS/port work |
| Jenkins security group | `sg-0a811821f4b2739bf` | Jenkins public-access review |
| Prod SSH rule (Jenkins→prod) | `sgr-05b94280fb11d357d` | Reference only — don't touch, working path |
| Prod SSH rule (personal IP) | `sgr-07a3dc0a970d1c562` | TAPI-025 follow-up if revisiting EC2 Instance Connect |
| S3 bucket | `terra-api-backups` | Postgres backup verification (`s3://terra-api-backups/postgres/prod/`), Jenkins migration archive |
| CloudWatch alarm | `awsec2-i-044e35066f956d506-...-StatusCheckFailed` | Confirm still active during any EC2 resize |
| SNS topic | `terra-api-prod-alerts` (aka `terra-api-alerts`) | Confirm subscription still live after any infra change |
| IAM identity | `will-cli` | TAPI-021 blocker — narrow from AdministratorAccess |
| IAM role | `terra-api-backup-role` | TAPI-025 SSM access, backup automation |

Direct console jump for any EC2 instance:
`https://console.aws.amazon.com/ec2/home?region=us-east-1#InstanceDetails:instanceId=<ID>`

### Notion
| Page | URL / ID | Used for |
|---|---|---|
| Terra API System Design doc | `https://app.notion.com/p/37089370d497814ab6cdf10732687fee` | Read before touching TAPI architecture |
| Terra API Notion project | `https://app.notion.com/p/37789370d49781908780e2b4e7a6c480` | ADRs 001–013 |
| 🌍 Terra Inc (root page) | `39289370d49780178d44c4e2c87c5488` | Now carries "Local machine" note for `terra-initiative-home` |
| Canonical Tasks DB | `collection://6002b34b-8bff-456a-aa43-4eed8f643dcd` | The contradictions in `07-notion-and-business-decisions.md` live here |
| ROMS Resort Deployment Tracker | (see ROMS Notion project) | OMS decisions |
| PIOS project page | (last synced 2026-05-30 — re-check freshness) | Learning-stack gate status |

### Live endpoints / dashboards
| What | Where |
|---|---|
| Terra API (prod, HTTPS) | `https://api.terra-hq.com` |
| Terra API (prod, raw) | `http://100.60.61.209:8080` (was 8081 — moving off, see TAPI-021) |
| Terra API management/actuator | `:8082` (deliberately unpublished — SSH/SSM only) |
| Jenkins (new dedicated box) | `http://3.211.62.86:8090` |
| Cloudflare dashboard | `https://dash.cloudflare.com` — zone `terra-hq.com` |
| terra-hq-site (public) | `https://terra-hq.com` (Cloudflare Pages) |
| OMS/Ha'bem prod | `http://100.60.7.24` (oms-server, no domain yet) |

### Git remotes
| Repo | GitHub | Bitbucket mirror |
|---|---|---|
| terra-api | `will55555/terra-api` | `terra-inc-dev/terra-api` |
| terra-api-fe | `will55555/terra-api-fe` | `terra-inc-dev/terra-api-fe` |
| terra-hq-site | `will55555/terra-hq-site` | — (single remote) |
| terra-jenkins | `will55555/terra-jenkins` | `terra-inc-dev/terra-jenkins` |
| terra-initiative-home (this repo) | `will55555/terra-initiative-home` (renamed on GitHub + local `origin` URL updated 2026-08-23) | — |
| claude-skills | `will55555/claude-skills` | — |

## Status as of 2026-08-23 — most of the original follow-ups are resolved

1. ~~Git drift on two repos~~ — **FIXED 2026-08-23.** Both repos pulled from GitHub, pushed to
   Bitbucket to resync, and upstream tracking switched to `origin` on both so this doesn't recur
   silently. Zero known git drift anywhere in the ecosystem as of this update.
2. ~~THQ-003's committed fix is half dead code~~ — **still open**, unchanged. Full detail in
   `04-terra-hq-site.md` §0.
3. ~~HUB.md's Machine Paths table is stale~~ — **FIXED 2026-08-23**, committed and pushed to
   `claude-skills` (`69774cb`).
4. ~~GitHub repo name mismatch~~ — **FIXED 2026-08-23**. Turned out the repo was already renamed on
   GitHub (discovered via its redirect notice); local `origin` URL updated to match directly.
5. **NEW 2026-08-23 — Notion contradictions resolved.** All 4 stale Notion rows this plan originally
   flagged for confirmation (JVM heap caps, ADR-012/Phase B, ROMS EC2/Phase D, plus "commit
   terra-hq-site's pending items") are now marked Done in Notion, each with a resolution note.
   ADR-012's own Status flipped Proposed→Accepted. Full detail in `07-notion-and-business-decisions.md`.
6. **NEW 2026-08-23 — terra-hq-site's pending-commit batch has landed.** THQ-005–018 (13 items)
   merged into `main`, pushed, zero divergence from origin. Full detail in `04-terra-hq-site.md`.
7. **NEW 2026-08-23 — branch protection needed on main/master, every repo.** No repo in the
   ecosystem appears to have branch protection configured. Given `terra-api`/`terra-api-fe` auto-
   deploy to prod on push, this is a real gap. Full detail + suggested baseline rule in
   `08-hub-and-git-housekeeping.md` item 5.
8. **ROOT-CAUSED 2026-08-23 — sign-out button on terra-api-fe.** Found via code read (no edits):
   `ApiDashboard.js` reuses the theme-toggle's circular button styling with a `⏻` Unicode glyph at
   a smaller font-size — likely a font-rendering/glyph-choice issue, not a layout bug. Full detail
   + fix options in `03-terra-api-fe.md` item 8.
9. **DECIDED 2026-08-23 — domain-naming for TAPI-022: Option C, split by audience.** Ha'bem/OMS
   AND PIOS (once it has a real product surface) both get their own dedicated domains; Jenkins/
   internal tooling stays on `terra-hq.com` subdomains; `api.terra-hq.com` unchanged.
   Domain-availability checks attempted this session are **worthless, not just inconclusive** —
   whois.com returned identical boilerplate for 7 different domains, meaning the fetch tool never
   got real data. Verify `habem.com` and every TLD variant directly at a registrar. Full detail in
   `02-terra-api-infra.md` item 3 and `06-pios.md` item 3.
10. **Still open, unchanged**: `terra-api/TASKS.md`'s own TAPI-020 row still says "Planned" against
    HUB_STATE's "Done 2026-08-07" — a same-repo doc contradiction, not a Notion one. See
    `02-terra-api-infra.md` item 0.
11. **NEW 2026-08-23 — fold `roms_gtm_strategy.html` into OMS as an admin page.** Real architecture
    decision, mirrors ADR-012's operator/customer split pattern. terra-hq-site's link should point
    at the new OMS admin page once it exists; customers get a separate page. Full detail (including
    a suggested `roms-adr-006`) in `05-roms-oms.md` item 0, cross-referenced from `04` item 6.
12. **NEW 2026-08-23 — Terra Nkap (crypto), placeholder only.** Raised as an open question, not a
    commitment — genuinely unscoped (no ADR series, no entity decision, unclear if it's a
    customer-facing token or backend settlement infra). New file `09-terra-nkap-crypto.md` captures
    what's known + 4 open questions worth answering before any real planning session.
13. **NEW 2026-08-23 — exploring `terra-hq.com` → `terrahq.net`, not decided.** Parent ecosystem
    domain, far wider blast radius than the product-level domain decisions above (CORS config,
    live HTTPS endpoint, Cloudflare Pages, every ADR/doc reference). Logged as an idea only — see
    `08-hub-and-git-housekeeping.md` item 4.
