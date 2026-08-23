# 02 — Terra API Infrastructure (TAPI-020 through TAPI-026)

Sequence per HUB_STATE's own committed order: **TAPI-021 → TAPI-023 → TAPI-022 → ADR-013**, with
TAPI-024/025 cleanup and the TAPI-020 status contradiction handled alongside since they're quick.

---

## 0. TAPI-020 — status contradiction, resolve first (5 min)

**Still open as of 2026-08-23 — narrowed scope.** The Notion side of this is already resolved: no
standalone Notion row existed for TAPI-020 itself, so there was nothing to correct there. What's
still genuinely wrong is `terra-api/TASKS.md`'s own row, which still says "Planned," while
HUB_STATE (and CLAUDE.md, per `ALL_TASKS.md`) say it fully closed 2026-08-07 (jacoco plugin,
`sonar-token` Jenkins credential, SonarCloud Automatic Analysis disabled, GitHub webhook wired,
pipeline green on every push to master). This is a contradiction inside `terra-api`'s own repo,
not a Notion sync issue.

**Steps:**
1. Open Jenkins (`http://3.211.62.86:8090`) → the terra-api pipeline → confirm a recent `master`
   build shows the SonarQube/quality-gate stage passing.
2. If confirmed: edit `terra-api/TASKS.md` row TAPI-020, change Status from "Planned" to "Done."

---

## 1. TAPI-021 — EC2 right-size terra-api's box back toward t3.micro (Active Task)

**Why it matters:** the box was bumped `t3.micro`→`t3.small` (~$7.50→$15/mo) as an emergency
stabilizer after a 40-hour undetected outage (OOM — two idle JVMs plus no swap left 28Mi free).
JVM heap caps and the swapfile (2 of the original 5 mitigation options) are already done. Three
real options remain.

**Sites:** AWS EC2 Console → `i-044e35066f956d506` (`terra-api-server`, us-east-1).

**Steps, in order (biggest win first):**
1. **Staging on-demand** (biggest remaining win, ~250–300MB reclaimed):
   - Decide: keep staging `up` 24/7, or bring it `down` by default and only `up` it when a
     `phase-*` branch build actually deploys to it.
   - If going on-demand: this needs a Jenkinsfile change (the `phase-*` deploy stage would need to
     `docker compose -f docker-compose.staging.yml up -d` before deploying and NOT tear it back
     down automatically — decide the teardown policy too, e.g. auto-down after N hours idle, or
     manual). This is a real code/pipeline change, not a console click — flag it as its own
     follow-up task when ready to build it (out of scope for a "don't touch code" plan).
2. **Alpine JRE base image** (~20-40MB/container): swap the Dockerfile's base image from `jammy` to
   an Alpine JRE variant. Also a code change (Dockerfile edit) — same as above, log as a follow-up
   coding task rather than doing it from this plan.
3. **Trim snapd/SSM** (~30-50MB): SSM access is needed (TAPI-025 depends on it) — do NOT disable
   SSM. `snapd` may be safe to remove if nothing else on the box depends on it; verify via
   `snap list` over SSM Session Manager before removing anything.
4. Once any combination of the above frees enough headroom, actually attempt the downsize:
   AWS EC2 Console → select the instance → **Instance state → Stop** → **Actions → Instance
   Settings → Change instance type** → `t3.micro` → **Start**.
   ⚠️ This causes downtime (stop/start cycle) — schedule it for low-traffic hours, and confirm the
   CloudWatch alarm (`awsec2-i-044e35066f956d506-...-StatusCheckFailed`) and SNS subscription
   (`terra-api-prod-alerts`) are still correctly wired before AND after (instance type changes can
   sometimes require re-confirming an alarm's dimension).
5. After resize, re-verify memory headroom isn't back at the 28Mi-free danger zone — SSM into the
   box, `free -h`.

**Also open, not yet scheduled (do alongside step 4 above):** remove the old **8081** security
group rule on `sg-063cee0bc65d872c9` once confident nothing still references port 8081 (TAPI-021's
own port move to 8080 already happened 2026-08-08; 8081 was deliberately left open "during
transition"). Check Cloudflare's Origin Rule and any hardcoded `:8081` references in configs/docs
first, then remove the inbound rule in the EC2 Console → Security Groups.

---

## 2. TAPI-023 — OS-level security patching automation, ecosystem-wide

**Why it matters:** no automated OS patching exists on any box today. Sequenced after TAPI-019
(Jenkins's own box, already done) so this covers every box that will exist, not just the ones that
existed when it was first scoped.

**Sites:** AWS Systems Manager (SSM) Console → **Patch Manager**.

**Steps:**
1. Open AWS Console → Systems Manager → Patch Manager.
2. Confirm SSM is active on all three boxes needing patching: `terra-api-server`
   (`i-044e35066f956d506`), `terra-jenkins` (`i-04ef85c382ac39269`), `oms-server`
   (`i-04f3abfb579f2bd1d`) — SSM was confirmed working on the first and third per TAPI-025's work;
   confirm terra-jenkins's box has it too (check via `aws ssm describe-instance-information` or the
   console's Fleet Manager).
3. Create a **Patch Baseline** (or use the AWS-provided default for Ubuntu) — decide auto-approval
   delay (e.g. auto-approve security patches after 7 days, to catch regressions before they hit
   your boxes).
4. Create a **Maintenance Window** (e.g. weekly, low-traffic hour) and register all three instances
   as targets, running the `AWS-RunPatchBaseline` document in `Install` mode.
5. Do a manual **Scan** run first (not Install) on all three boxes to see current patch compliance
   before turning on automatic installs.
6. Once comfortable with the scan results, enable the Install-mode maintenance window schedule.

---

## 3. TAPI-022 — Domains + TLS for all remaining ecosystem endpoints

**Why it matters:** only `https://api.terra-hq.com` is real HTTPS today (Cloudflare Flexible
SSL + an Origin Rule to port 8080). Everything else — Jenkins, OMS/Ha'bem — is still raw IP:port.

**DECIDED 2026-08-23 — Option C (split by audience).** Ha'bem/OMS gets its own dedicated domain;
internal/operator tooling (Jenkins) stays on `terra-hq.com` subdomains; `api.terra-hq.com` stays
as-is. Original brainstorm kept below for the reasoning trail.

**Domain availability — my automated checks were USELESS (whois.com fetch artifact, not real
lookups — see prior note), IGNORE THEM.** Will checked directly instead: **`habem.net`, `habem.org`,
and `habemapp.com` are all confirmed available.** Plain `habem.com` status still unconfirmed —
worth checking who owns it and whether it's realistically acquirable, since `habemapp.com` (a real
`.com`, just with "app" appended) may be the better fallback if the bare `habem.com` isn't gettable
— a `.com` with "app" tacked on is a well-understood, common pattern (reads clean, e.g. joinapp.com
-style branding) and beats compromising on TLD.

**TLD choice, if it comes down to `.net` vs `.org` for a paid consumer ordering product:**
`.net` over `.org` — `.org` signals "nonprofit/organization," the wrong association for a product
that takes payments. `.net` carries no negative signal, just slightly less "default" than `.com`
(some users will still type `.com` from habit). Also worth a look if `.com` is truly unavailable:
`.app` (Google-run registry, forces HTTPS, reads as a modern product) and `.co` (widely understood
`.com` stand-in). **Not yet decided** — Will is weighing `.com`-acquisition first before settling.

**NEW 2026-08-23 — PIOS also needs its own domain**, per the same Option C logic (audience split):
once PIOS moves past its learning-stack gate (see `06-pios.md`) and has a real product surface,
it's a consumer/investor-facing product like Ha'bem, not internal tooling — so it should get its
own dedicated domain too, not a `terra-hq.com` subdomain. Nothing to check yet (no product name
locked in, no repo exists) — logged here so the domain-naming pattern is applied consistently when
PIOS gets there, not decided ad hoc later. See `06-pios.md` for the actual gating status.

The original
plan assumed subdomains of `terra-hq.com` for everything (e.g. `oms.terra-hq.com`,
`jenkins.terra-hq.com`). You flagged this isn't settled — specifically not keen on a pattern like
`oms-terra-hq.com`. Worth deciding the whole naming scheme once, rather than picking per-service ad
hoc. Options, not a recommendation:

| Option | Shape | Pros | Cons |
|---|---|---|---|
| **A. Subdomains of terra-hq.com** | `oms.terra-hq.com`, `jenkins.terra-hq.com`, `api.terra-hq.com` (already live) | Cheapest (no new domain purchases), reinforces "one ecosystem" branding, consistent with what's already shipped for the API | Ties every product's public identity to the parent holding company's domain — may not fit Ha'bem's own guest-facing brand identity (it's meant to feel like its own product, not "a Terra subdomain") |
| **B. Dedicated domain per consumer-facing product** | `habem.com`/`getHabem.com` for OMS, `terra-hq.com` stays for the corporate/investor site, internal tools (Jenkins) stay on subdomains | Matches Ha'bem's own branding effort (wordmark, naming registry, guest-facing identity work already done in `05-roms-oms.md`) — a guest ordering food shouldn't see "terra-hq" in the URL | New domain to buy/renew per product; more moving pieces (more Cloudflare zones, more cert management) |
| **C. Hybrid — customer-facing gets its own domain, internal/operator stuff stays on terra-hq.com subdomains** | `habem.com` (guest-facing OMS) + `jenkins.terra-hq.com` (internal only, arguably shouldn't even be public — see Part B item 1 in `07`) + `api.terra-hq.com` (stays, it's already the accepted internal/developer-facing convention) | Splits by AUDIENCE not by product — matches the ADR-012 pattern already established (`public`/`customer`/`investor`/`internal` are different audiences with different surfaces) | Slightly more to track than one uniform pattern |

**Recommendation to consider, not a decision:** Option C lines up with a distinction the ecosystem
has already made deliberately elsewhere (ADR-010's four audiences) — guest-facing products get
their own brand-appropriate domain, internal/operator tooling stays consolidated under
`terra-hq.com` subdomains where nobody but Will (and eventually other operators) ever sees the URL.

**Once the naming scheme is decided, steps per service:**
1. **Jenkins** (`3.211.62.86:8090`): if staying on a `terra-hq.com` subdomain (likely, it's internal
   tooling) — in Cloudflare DNS, add an A record → `3.211.62.86`, proxied (orange cloud). Add an
   Origin Rule rewriting destination port to 8090 (same pattern as the API's 8080 rewrite, since
   Cloudflare's free tier only proxies to a fixed origin-port allowlist). Decide whether Jenkins
   should be public at all vs. behind Cloudflare Access (see `07`, Part B item 1 — decide together).
2. **OMS/Ha'bem** (`100.60.7.24`): if going with a dedicated domain (Option B/C above) — first
   check availability/purchase a domain matching the Ha'bem brand (this is also blocked on the
   trademark/domain availability check already listed in `05-roms-oms.md` item 5 — do that check
   first, it may constrain the domain choice). Add the new domain as its own zone in Cloudflare,
   point an A/CNAME record at `100.60.7.24`, same Origin Rule pattern for the port.
3. For each new domain/subdomain: verify Cloudflare's SSL/TLS mode — Flexible (Cloudflare↔browser
   HTTPS, Cloudflare↔origin plain HTTP) is what's used for the API today; decide whether to upgrade
   any of these to Full/Full-Strict once the origin can present its own cert (bigger lift, not
   required to just "have a domain").
4. Update Jenkinsfiles / webhook URLs / any hardcoded IP:port references once a domain is live and
   confirmed working, so future deploys use the domain, not the raw IP (this is a code change per
   repo — log it as a follow-up coding task per repo rather than doing it here).

---

## 4. TAPI-024 — Grant prod `customer_service_access` for roms/oms to Will's real account

**Why it matters:** terra-api-fe's customer dashboard shows an empty topology in prod for Will's
own account — the dev-only seed data that grants this never runs in prod.

**Blocker to clear first:** Will's real prod `customer_id` needs to be confirmed.

**Sites:** prod Postgres, reached via SSM Session Manager into `terra-api-server`
(`i-044e35066f956d506`) — direct network access to port 5432 is closed from outside per TAPI-025's
findings.

**Steps:**
1. AWS Console → EC2 → `i-044e35066f956d506` → **Connect** → **Session Manager**.
2. From the session, exec into the running Postgres container (same pattern TAPI-025 used for the
   login-bug investigation) and query: `SELECT customer_id, email FROM customers WHERE email =
   '<Will's real prod email>';` to get the exact `customer_id`.
3. Run: `INSERT INTO customer_service_access (customer_id, service_id) VALUES ('<id>', 'roms'),
   ('<id>', 'oms');` — check the exact `service_id` values already in use for OMS (may be `roms` or
   `oms` depending on whether the OMS-013 rename has landed — see `05-roms-oms.md`; check the
   column's existing values for other rows first, don't guess).
4. Refresh the customer dashboard in a browser to confirm the topology now populates.

---

## 5. TAPI-025 — finish the temporary bootstrap controller cleanup

**Why it matters:** `OneShotOpsScopeGrantController` and its matching temporary Jenkinsfile block
are explicitly marked TEMPORARY in their own code comments — they exist only to grant Will's
account the `ops:read` scope once, and have no reason to survive past that.

**Status check first:** confirm whether the 5-step process documented in `terra-api/TASKS.md` (row
TAPI-025) was ever actually run. If Will's `/internal` access already works today, steps 1–4 below
are already done — skip straight to step 5 (deletion). If `/internal` still 403s, run all 5.

**Sites:** Jenkins (`http://3.211.62.86:8090`), a terminal/curl, the terra-hq.com login page.

**Steps:**
1. In Jenkins, create credential `bootstrap-ops-scope-secret` (Secret text, value of Will's
   choosing, never commit it anywhere).
2. Merge/push the branch containing `OneShotOpsScopeGrantController` to `master` to trigger deploy.
3. `curl -X POST https://api.terra-hq.com/_bootstrap-ops-scope?email=<will's prod email> -H
   "X-Bootstrap-Secret: <value from step 1>"`.
4. Log out and back in on the live site to pick up a fresh JWT carrying the new scope.
5. **Delete** `OneShotOpsScopeGrantController.java` and the temporary Jenkinsfile block added for
   it, then redeploy — this is a real code change (file deletion + Jenkinsfile edit), so treat step
   5 as its own small follow-up coding task once steps 1–4 are confirmed successful. Don't leave
   this controller live longer than necessary — it's an intentionally unauthenticated write path
   gated only by a shared secret.

---

## 6. Standing security follow-up: narrow `will-cli` IAM identity

**Why it matters:** currently full `AdministratorAccess`. HUB_STATE flags a same-session incident
where broad admin access let one slightly-off command corrupt a prod env file
(`docker-prod.env`/`TERRA_AUTH_PASSWORD`).

**Sites:** AWS IAM Console → Users/Identities → `will-cli`.

**Steps:**
1. Review CloudTrail or recent session history for what `will-cli` has actually been used for
   (EC2, S3, SSM, IAM policy attach, direct RDS/Postgres access were all mentioned in recent work).
2. Draft a least-privilege policy covering exactly those services/resources, OR migrate to IAM
   Identity Center with a permission set scoped the same way.
3. Test the narrowed policy against a real recent workflow (e.g. an SSM session + an S3 backup
   check) before removing `AdministratorAccess` entirely, to avoid getting locked out mid-task.

---

## 7. TAPI-026 — Manual quarantine/release from Operator tab (Backlog, explicitly not scoped yet)

**No action needed right now** — Will's own call was "we'll refine it" and it's logged as backlog.
Listed here only so it's not lost: when ready to design this, the real implementation needs (1) a
release/un-quarantine method on `QuarantineService` (doesn't exist yet — only the trigger
direction does), (2) two new gated endpoints on `InternalEcosystemController`, (3) a
click-to-confirm frontend interaction in `OperatorTab.js`/`terraScene.js`. All of this is code —
don't start it without a dedicated design pass first, per Will's own note.
