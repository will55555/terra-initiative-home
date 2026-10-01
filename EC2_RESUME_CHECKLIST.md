# EC2 Resume Checklist

<!-- Written 2026-10-01 — all three ecosystem EC2 instances were stopped (not terminated) this
     date to cut AWS costs while Will works on something else. This is the reverse of that
     action: what to do, in order, when picking this project back up. Update the "Stopped on"
     date if this gets reused for a future pause. -->

## Why this exists

On 2026-10-01, all three EC2 instances backing the Terra ecosystem were **stopped** (not
terminated) to eliminate EC2 compute billing while this project was paused:

| Instance | ID | Type | Role |
|---|---|---|---|
| `terra-api-server` | `i-044e35066f956d506` | t3.small | Terra API prod + staging (both tiers, one box) |
| `oms-server` | `i-04f3abfb579f2bd1d` | t3.small | Ha'bem/OMS prod |
| `terra-jenkins` | `i-04ef85c382ac39269` | t3.medium | Jenkins CI/CD controller |

**Stopping ≠ terminating.** EBS volumes (and everything on them — Postgres/Redis data, Jenkins'
`infra_jenkins_home` volume, all repo clones on each box) are untouched while stopped. All three
instances have an **Elastic IP** (not a dynamic public IP) matching their current public address,
so the public IP is retained across the stop/start cycle — no DNS or webhook URL changes needed
on resume, unlike a dynamic-IP instance which would get a new address on every start.

**What was offline during the pause:** `api.terra-hq.com` (Terra API prod, customer dashboard,
public ecosystem-health endpoint), Ha'bem/OMS prod, and Jenkins (so no CI/CD ran on any push
during this window — don't expect any auto-deploys to have happened).

---

## Resume steps, in order

### 1. Start the instances

AWS Console → EC2 → Instances → select each → **Instance state → Start instance**. Order doesn't
strictly matter, but starting `terra-api-server` and `oms-server` first (then `terra-jenkins`)
means the product boxes are already coming up while you're still working through Jenkins.

Wait for status checks to go green (3/3) on each before proceeding — `terra-jenkins` was
showing **2/3 checks passed** before it was stopped (noted 2026-10-01, not investigated); worth
watching whether that's still the case after this restart or was transient.

### 2. Confirm containers actually came back

`restart: unless-stopped` on every service in each compose file means Docker *should* bring
containers back up automatically when the instance boots and the Docker daemon starts — but
don't assume, verify:

```bash
# SSH into terra-api-server
ssh ubuntu@100.60.61.209   # or whatever the current Elastic IP shows in the console

# Check both tiers are up
cd ~/terra-api-prod && docker-compose-v2 -p terra-api-prod -f docker-compose.prod.yml ps
cd ~/terra-api-staging && docker-compose-v2 -p terra-api-staging -f docker-compose.staging.yml ps
```

```bash
# SSH into oms-server
ssh ubuntu@100.60.7.24   # or current Elastic IP

docker ps   # confirm oms-be / postgres / redis / kafka (per OMS's own compose) all Up
```

```bash
# SSH into terra-jenkins
ssh ubuntu@3.211.62.86   # or current Elastic IP

docker ps   # confirm the single 'terra-jenkins' container is Up
```

If anything isn't `Up`/healthy, `docker-compose ... up -d` (or plain `docker start <container>`)
in the relevant directory should bring it back — same recovery as any other unexpected restart.

### 3. Spot-check the live endpoints

- `https://api.terra-hq.com` — should load the ApiDashboard (`/`) or prompt for operator login,
  not a connection error / 522.
- `https://api.terra-hq.com/actuator/health` via the management port path, or SSM into the box
  and `curl localhost:8082/actuator/health` directly (`:8082` is deliberately unpublished).
- Ha'bem/OMS prod — `http://100.60.7.24` (no domain yet, per the last-known state) — confirm it
  actually serves the storefront, not a blank/error page.
- `https://terra-hq.com` (the public visualizer, Cloudflare Pages — unaffected by any of this
  stop/start, since it's not EC2-hosted) — confirm cubes light up again now that Terra API's
  public-health endpoint is reachable (it will have shown "unreachable"/grey the whole time
  everything was stopped).
- `http://3.211.62.86:8090` — Jenkins UI loads, your jobs/credentials/build history are intact
  (confirms the `infra_jenkins_home` volume survived the stop, as expected).

### 4. Re-arm anything that depends on "currently running"

- **CloudWatch `StatusCheckFailed` alarm** (on `terra-api-server`) — this watches for an
  *unexpected* failure; it should resume normal monitoring automatically once the instance is
  running again, nothing to manually re-enable. Worth a quick look at the alarm's history in the
  Console to confirm it didn't fire/notify anything confusing during the intentional stop window.
- **Automated Postgres backup cron** (`terra-api-server`, daily 3am UTC per `TAPI-018`) — resumes
  on its own once the instance (and therefore cron) is running again. The backup window(s) during
  the stop were simply skipped, not failed — nothing to clean up, just don't expect backups dated
  during the pause.
- **Jenkins job triggers** — any GitHub push during the pause window did NOT trigger a build
  (Jenkins wasn't reachable to receive the webhook). GitHub doesn't queue/retry webhook deliveries
  indefinitely, so if anything was pushed to `master`/`phase-*` while Jenkins was down, you'll
  need to manually trigger that job ("Scan Multibranch Pipeline Now" or a manual build) once
  Jenkins is back — it won't auto-catch-up on its own.

### 5. Normal resume

At this point, treat it like any other `load hub` session start — check `git status` on each
repo for anything that happened or was left uncommitted before the pause, re-read HUB_STATE's
Next Step for whichever project you're resuming, and continue from there.

---

## If costs need to come back down again later

See `terra-api/TASKS.md` → `TAPI-021` for the already-scoped (not yet built) staging-on-demand
plan — the actual long-term fix so staging doesn't sit up 24/7 between deploys, without needing
a full stop of everything every time. Also worth a look next time: `terra-jenkins` is `t3.medium`
(~$30/mo), the single biggest EC2 line item in the ecosystem for what is, today, a single-controller
Jenkins instance with no build agents — right-sizing that box specifically (not just stopping it)
is a separate, not-yet-scoped opportunity if cost keeps mattering once this resumes for real.
