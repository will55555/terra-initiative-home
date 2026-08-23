# 08 — Hub & Git Housekeeping

**Updated 2026-08-23: items 1–3 are resolved.** Keeping the record for context; item 4 (monthly
audit) is still upcoming.

---

## 1. Multi-remote git drift — RESOLVED 2026-08-23

Both repos pulled from `origin` (GitHub) and pushed to `bitbucket` to resync the mirror:
- `terra-api` `master`: pulled `74a199a` (DEV_LOG entry), pushed to bitbucket — confirmed in sync
  with both remotes.
- `terra-api-fe` `main`: pulled `09ae916` (package-lock regen, app name update, DEV_LOG entry),
  pushed to bitbucket — confirmed in sync with both remotes.

**Also done 2026-08-23:** upstream tracking switched from `bitbucket` to `origin` on both repos
(`git branch --set-upstream-to=origin/master master` / `...origin/main main`) — a plain `git pull`/
`git status` now compares against GitHub by default, so this class of drift surfaces immediately
instead of hiding behind a Bitbucket-synced status. Bitbucket still needs an explicit
`git push bitbucket <branch>` going forward — that's now the thing to remember, not GitHub.

---

## 2. HUB.md's Machine Paths table — FIXED 2026-08-23

All `(machine: test)` rows for terra-api/terra-api-fe/terra-jenkins/terra-hq-site corrected to
point at `terra-initiative-home\<repo>\` (each a gitignored nested-repo subdirectory of one outer
repo), replacing the stale "New folder\" sibling-layout description. Committed and pushed to
`claude-skills` (`69774cb`). Freshness stamp bumped to 2026-08-23, v2.0.

---

## 3. GitHub repo name — FIXED 2026-08-23

The GitHub repo was already renamed to `will55555/terra-initiative-home` (discovered mid-push via
GitHub's redirect notice — the "not yet renamed" note in the original version of this file was
itself stale). The local `origin` remote URL has now been updated to match directly:
`https://github.com/will55555/terra-initiative-home.git` — no longer relying on the redirect.

---

## 4. NEW 2026-08-23 — Exploring replacing terra-hq.com with terrahq.net (parent domain, NOT a decision yet)

**Just an idea Will is floating — explicitly not ready to decide, logged here so it isn't lost.**
Unlike the OMS/PIOS domain decisions (new products getting their own domain), this would replace
the **parent ecosystem domain itself** — `terra-hq.com` is already live in production and referenced
far more widely than any single product:
- `https://api.terra-hq.com` — Terra API's real HTTPS endpoint (Cloudflare Flexible SSL + Origin
  Rule to port 8080)
- `terra-api`'s CORS config is narrowly scoped to `https://terra-hq.com` specifically (the
  deliberate exception documented for terra-hq-site's public visualizer, TAPI-014/THQ-002 work)
- Cloudflare Pages deploy for terra-hq-site itself
- Referenced throughout ADRs, Notion pages, and `HUB_STATE.md`/`ALL_TASKS.md` as the standing
  ecosystem identity
- The planned Jenkins/internal-tooling subdomains (`02-terra-api-infra.md` item 3) would all sit
  under whichever domain wins here

**If this is ever pursued for real**, it's a coordinated multi-repo migration, not a DNS-only
change: new Cloudflare zone + cert, CORS config update in `terra-api`, Cloudflare Pages re-point
for terra-hq-site, every hardcoded `terra-hq.com` reference across ADRs/docs updated, old domain
kept alive with a redirect during transition (not dropped immediately, to avoid breaking any
external links/bookmarks/webhooks still pointing at it). Worth its own dedicated planning pass with
a full reference-audit (grep every repo + Notion for `terra-hq.com`) before scoping further — not
attempted here since Will isn't ready to decide yet.

---

## 5. Monthly Hub Audit is coming due

`Last Audit: 2026-08-02` in `HUB_STATE.md` — fires after **2026-09-01**. Items 1–3 above already
resolve 2 of that audit's 5 checklist items (multi-remote sync is still item 1's open half —
Machine Paths accuracy is now clean). Worth running the remaining checklist items proactively next
`load hub`: HUB_STATE claims vs. git for every ACTIVE project, Reference Links resolving, rules
within the 80-line read window, and re-checking multi-remote sync once item 1 above is actually
pulled.

---

## 6. NEW 2026-08-23 — Branch protection needed on main/master, every repo

**Why it matters:** none of the ecosystem's repos currently appear to have branch protection rules
on their primary branch (not checked via API this session — `gh` CLI isn't installed on this
machine — but worth confirming and fixing regardless, since an unprotected `master`/`main` on a
repo with a live CI/CD pipeline means a force-push or direct bad commit can hit prod with no gate).

**Sites:** GitHub → each repo → Settings → Branches → Add branch protection rule.

**Repos needing this** (their default/deploy branch):
| Repo | Branch | Notes |
|---|---|---|
| `terra-api` | `master` | Deploys to prod on push per the Jenkinsfile's branch-tiering — highest-stakes one to protect |
| `terra-api-fe` | `main` | Same-origin deploy rides inside terra-api's build, but still worth protecting independently |
| `terra-hq-site` | `main` | Public-facing, Cloudflare Pages auto-deploys on push |
| `terra-jenkins` | `master` | Infra-as-code for the CI/CD system itself — a bad push here is especially high blast-radius |
| `terra-initiative-home` | `main` | Orchestration/docs repo — lower stakes but still worth a baseline rule |
| `claude-skills` | `master` | Governs the hub itself — the Hub Self-Sync Exception already lets Claude push here unattended, which is exactly the kind of path worth a protection rule around (e.g. still allow direct pushes but require the check to at least exist) |

**Suggested baseline rule (adjust per repo's actual workflow — you're solo, so keep it light):**
1. Require a pull request before merging — **or**, if staying on direct-push-to-master (matches the
   "no PR ceremony while solo" convention already established in the hub's Commit Conventions),
   skip this and just enable the items below instead.
2. **Require status checks to pass before merging** (for `terra-api`/`terra-api-fe`: the Jenkins
   CI/CD pipeline result) — this is the one that actually matters most, since it stops a broken
   build from reaching a branch that auto-deploys.
3. **Block force pushes** and **block branch deletion** on the protected branch — cheap insurance,
   no workflow cost for a solo dev who isn't force-pushing `master` intentionally anyway.
4. Leave "Require pull request reviews" OFF for now, consistent with the solo-dev convention —
   revisit only when a second contributor exists.

**Do this per repo, in rough priority order:** `terra-api` → `terra-jenkins` → `terra-api-fe` →
`terra-hq-site` → `terra-initiative-home` → `claude-skills`.
