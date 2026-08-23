# 05 — ROMS / Ha'bem (OMS) — prefix ROMS, product name OMS

**Important caveat:** the ROMS/OMS repo (`restaurant-order-management-system`) is **not cloned on
this machine** — it's not present under `terra-initiative-home/` here (the `.gitignore` reserves
`oms/`/`restaurant-order-management-system/` for it, but neither folder exists locally). Everything
below is reconstructed from `HUB_STATE.md` and `ALL_TASKS.md`'s cross-reference of that repo's own
`TASKS.md` — re-read `oms/TASKS.md` directly on whichever machine has it cloned before acting, since
this is second-hand.

---

## 0. NEW 2026-08-23 — Fold `roms_gtm_strategy.html` into OMS as an admin page; split admin vs. customer links

**Raised by Will, real architecture decision — not scoped further than this yet.** Today,
`https://terra-hq.com/roms_gtm_strategy.html` lives as a static page inside `terra-hq-site` (GTM
strategy content — investor/business-facing). The proposal: **fold that content into OMS itself as
an admin/operator page**, so it lives with the product it's actually about instead of on the
marketing site. Two link targets going forward:
- **From terra-hq-site**, clicking through to this content should land on OMS's new admin page
  (not stay on terra-hq-site).
- **Customers** get a *separate*, genuinely customer-facing OMS page/link — not the GTM strategy
  content at all.

**Why this matters / how it fits the existing pattern:** this is structurally the same move
ADR-012 already made for Terra API — `terra-api-adr-012` pulled the internal operator view out of
being scattered/raw and gave it a real gated `/internal` route inside `terra-api-fe`, distinct from
the customer dashboard at `/dashboard`. OMS doing the same (an admin/GTM-strategy view distinct
from the customer ordering experience) is consistent with the ecosystem's existing audience-split
convention (`public`/`customer`/`investor`/`internal` per `terra-api-adr-010`) — worth explicitly
naming it that way rather than treating it as a one-off.

**What to decide/gather before any coding session (none of this needs the OMS repo checked out
here — it's all planning):**
1. **Audience/access model for the new OMS admin page**: does it need real authentication/gating
   (mirroring ADR-012's `role=internal` + `ops:read` pattern), or is it lower-stakes since it's
   GTM/strategy content rather than live customer/quarantine data? Decide the actual sensitivity
   before assuming it needs the same gate weight as Terra API's operator view.
2. **Where the admin page lives**: a new route inside OMS-fe (same-app pattern, like ADR-012 chose
   for Terra API) vs. a separate small internal tool. The ecosystem's stated precedent (ADR-012's
   own reasoning: reuse auth/theming/infra rather than standing up a second app) argues for
   same-app unless OMS-fe's architecture makes that awkward — confirm by looking at OMS-fe's actual
   structure once the repo is available.
3. **What "customer-facing" OMS page means today** vs. what it should become — check what
   `OMS-fe`'s current default/root route actually shows customers right now (per HUB_STATE, this
   is the ported `habem-pages.jsx` browse-grid/cart flow) to confirm it's genuinely distinct from
   the GTM content, not accidentally the same page today.
4. **Content migration**: `roms_gtm_strategy.html`'s actual content needs porting into OMS-fe as
   real components (not an iframe embed) — read it fully first to scope what's actually in it
   before estimating the port size; terra-hq-site's own THQ-010/011 merges (folding
   `terra_ecosystem.html`/`terra_business_structure.html` into other pages) are a recent, similar
   precedent for how this repo has done this kind of content move before.
5. **terra-hq-site's own link**: once the OMS admin page is live, the `roms_gtm_strategy.html` link
   on terra-hq-site (currently pointed at itself) needs to redirect/point at the new OMS admin URL
   instead — this is a small terra-hq-site edit, sequenced AFTER the OMS admin page actually exists,
   not before. Cross-reference `04-terra-hq-site.md` for the corresponding note.
6. **Worth its own ADR?** Given the ecosystem's convention of ADRs for structural moves like this
   (`roms-adr-*` series exists, currently at 001–005), consider drafting `roms-adr-006` documenting
   this decision before building it — matches how ADR-012 was written BEFORE the operator endpoints
   were built, not after.

**Sequencing relative to OMS-013 below:** independent — the `serviceId` rename and this admin-page
restructuring don't block each other, but both touch OMS-fe, so consider doing them in the same
session to avoid two separate rounds of OMS-fe changes.

---

## 1. OMS-013 — Coordinate cross-repo `serviceId: 'roms'` → `'oms'` rename

**Blocker CLEARED 2026-08-23.** terra-hq-site's uncommitted state (the ~80 file deletions / Assets/
reorg this was blocked on) has landed — the pending THQ-005–018 batch merged into `main` and pushed
(commit `29d269c`), working tree confirmed clean, zero divergence from origin. This repo is now
safe to touch for the rename.

**Once unblocked, before coding:** grep every repo (`terra-api`, `terra-api-fe`, `terra-hq-site`,
and the ROMS repo itself) for the literal string `roms` used as a `service_id`/`serviceId` value
(not the repo name — that stays "ROMS" as an internal codename) to get the full list of call sites
that would need updating together. This is a cross-repo coordinated rename — plan the commit order
(likely backend accepts both values during a transition window, then frontend switches, then
backend drops the old value) before writing any code.

---

## 2. OMS-015 — Redis connection root cause genuinely uncertain

**Why it matters:** a health check flipped DOWN→UP with no code change — the fix is committed but
the actual root cause was never confirmed. Needs a dedicated investigation session with better
tooling than was available when it was first hit.

**What to gather before that session:** pull CloudWatch metrics (or Docker container logs via SSM)
for the `oms-server` box (`i-04f3abfb579f2bd1d`, `100.60.7.24`) around the time window this was
first observed, looking for anything correlated (network blip, container restart, Redis's own
memory pressure). Having this data ready before the investigation session starts saves the first
hour of just gathering evidence.

---

## 3. OMS-018 — Favicon/touch-icon needs tighter cropping

**Blocked on Will supplying a new source image** — pure asset gate, same class as TFE-602's
favicon half. No coding involved once a properly-cropped source image exists; just drop the
replacement files in.

---

## 4. ROMS ADR-005 / terra-api-adr-010 — stale "found stopped, recoverable in place" text

**Why it matters:** both ADRs still describe an older recovery narrative, superseded by the actual
migration that happened since.

**Steps:**
1. Open the Notion pages for `roms-adr-005` and `terra-api-adr-010`.
2. Read the current "found stopped, recoverable in place" language against what actually happened
   (the real EC2 migration/redeploy history, per HUB_STATE's ROMS section).
3. Draft an amendment section (not a rewrite — ADRs get amended, not silently edited, per the hub's
   documentation conventions) describing the actual outcome, dated.

---

## 5. Business/brand decisions — Ha'bem (do these before further ROMS/OMS frontend work)

These are non-code, but block downstream work (the wordmark, the Grocery category, currency
pinning, and Terra Chain discount rate all wait on some of these):

1. **Final Ha'bem wordmark palette** — decide, since brand/icon system is drafted but palette isn't
   finalized.
2. **Domain + trademark availability** — check OAPI (Cameroon/CEMAC regional trademark office) for
   "Ha'bem" availability before committing to the brand publicly. Also check domain availability
   for whatever TLD is intended.
3. **Grocery category fulfillment model** — decide before building it as a separate tab; this is a
   real operational decision (sourcing, delivery), not a technical one.
4. **`CatalogItem` price-currency pinning** — decide the policy (pin at creation vs. dynamic) before
   backend work starts; the frontend pattern already exists as reference.
5. **Terra Chain settlement design** — the placeholder 8% Nkap discount rate is waiting on this
   being real; check in with wherever Terra Chain/Nkap design work lives (likely a separate,
   not-yet-started initiative) for status.
6. **Spa/Tours/Housekeeping category pages** — wait on `Booking`/`ServiceRequest` entities existing
   in the data model first; this is sequencing, not a decision, but worth noting so it isn't
   attempted out of order.

**Separately, lower-priority technical housekeeping (per HUB_STATE's ROMS "Next Step," CONFIRMED
still open 2026-08-23 via the Ha'bem (OMS) Notion project page's own log — this is the one real
fragment of the now-closed "Phase D" Notion row, see `07`):**
- Disable SonarCloud Automatic Analysis for the ROMS project (same class of fix TAPI-020 already
  did for terra-api — SonarCloud's own automatic analysis conflicts with CI-based analysis; an
  enforced quality gate was attempted and dropped per that same log, since this SonarCloud plan
  doesn't allow project webhooks — report-only analysis is the ceiling here, not a gate).
  **Site:** sonarcloud.io → the ROMS project → Administration → Analysis Method.
- Terminate/delete the old us-east-2 EC2 instance once confident it's no longer needed (the
  current prod box is `oms-server`/`i-04f3abfb579f2bd1d` in us-east-1 — the us-east-2 one is a
  leftover from before that migration). **Site:** AWS EC2 Console, switch region selector to
  us-east-2 first, confirm the instance ID before terminating anything.

---

## 6. Terra Agriculture cross-vertical sourcing (§13, conceptual only)

**Not build-ready — explicitly planning-only.** Ha'bem's F&B catalog sourcing from Terra
Agriculture is drafted conceptually but blocked on a real geography mismatch: Terra Agriculture's
only defined scope is a Ghana bamboo/calabash pilot (no produce plans yet), while Ha'bem is
Cameroon-only. The one real thread (both plan to use Terra Nkap) doesn't resolve the geography gap.

**What to do:** decide the open question flagged in HUB_STATE — is Terra Agriculture sourcing
scoped Cameroon-only to match Ha'bem, or should it be a standing cross-property principle? This
decision has ripple effects into `07-notion-and-business-decisions.md`'s Terra Agriculture items —
resolve them together, not independently.
