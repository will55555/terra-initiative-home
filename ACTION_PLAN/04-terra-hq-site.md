# 04 — terra-hq-site (THQ-001/002/003/004/018)

**Updated 2026-08-23: the big pending-commit batch has landed.** THQ-005 through THQ-018 (gold
color-tone rollout, circuit backdrop, resort page rebuild, 7+ merges/renames, visualizer
transparency/theme-sync, nav menu, THQ-017's FE-parity port) — 13 items previously "Done — pending
commit" from the `solan` machine — merged into `main` this session (commit `29d269c`) alongside a
local THQ-003 status fix. Confirmed pushed, zero divergence from `origin/main`. Nothing to commit
for that batch anymore. The items below are the ones still genuinely open.

---

## 0. THQ-003 — REAL BUG CONFIRMED: the committed fix is half dead code

**Re-verified 2026-08-22 by reading the actual file, not just the notes — this is a genuine,
still-open bug, not a commit-risk item (superseding what the old file 01 said about this).**

`terra-hq-site/TASKS.md` currently describes THQ-003 as "In Progress — UNCOMMITTED, local to `test`
machine only." That's stale. The fix commit (`43805a9a`, 2026-08-03) **is** in this machine's git
history. But reading `terra_api_visualizer_phase5.js` lines 660–668 directly shows the fix was
committed as a **comment**, not as active code:

```js
const tube = new THREE.Mesh(tubeGeometry, tubeMaterial);
// cube1/cube2 (both = cube), not the old singular .userData.cube — matches the main radial
// tubes' naming so updateCubeConnection's existing live-refresh loop picks this up on every
// tick instead of freezing connected1/connected2 at whatever they were the moment this tube
// was created (the actual bug: a cube expanded before its service reported healthy stayed
// red forever after, even once the real data caught up).
// tube.userData.cube1 = cube;
// tube.userData.cube2 = cube;
tube.userData.isExtension = true;
```

The two assignment lines that would actually fix the bug are commented out. By contrast, the
**child** tube a few lines below (line 681–682) DOES have the real fix applied:
```js
childTube.userData.cube1 = cube;
childTube.userData.cube2 = cube;
```

**Net effect:** `updateCubeConnection()`'s live-refresh loop only updates a tube when both
`userData.cube1` AND `userData.cube2` are set. The child extension tube refreshes correctly. The
**parent** extension tube (the one from a cube's original pipeline position to its expanded
position) never gets `cube1`/`cube2` set at all — so it's still frozen at creation time, which is
the exact bug THQ-003 was opened to fix. The bug is only half-fixed.

**What to do (this is a 2-line code change once you're ready — not done here per "don't touch the
code"):** uncomment lines 666–667 in `terra_api_visualizer_phase5.js`. Then run the original repro
from `TASKS.md`: hard refresh → wait ~6s for heartbeat data → click Hospitality to expand → hover
the ROMS/OMS child cube → confirm the tube connecting the cube to ITS ORIGINAL POSITION (not just
the child-tube) goes gold/animated and updates live as health data changes, not just at the moment
of the click. Once visually confirmed, update `terra-hq-site/TASKS.md`'s THQ-003 row to Done — the
current "uncommitted" status is doubly wrong (it's committed, and the reason to keep testing isn't
"needs committing," it's "needs the actual fix uncommented").

---

## 1. THQ-001 — Visualizer frontend integration ADR + migration plan (In Progress since 2026-07-17)

**DECISION NEEDED — likely stale.** This has been "In Progress" for over a month, and a huge amount
of visualizer work (THQ-002/003/017, TFE-401's full port) has shipped since it was opened.

**Steps:**
1. Open the Notion Terra API project page and find `terra-api-adr-009` (the ADR this task was
   meant to produce).
2. Compare ADR-009's content against what's actually been built: is terra-api-fe's Three.js port
   (TFE-401, feature-complete against ADR-009's Build Sequence per HUB_STATE) the "migration" this
   task was tracking?
3. If yes — mark THQ-001 **Done** in `terra-hq-site/TASKS.md`, pointing at ADR-009 and TFE-401 as
   the completed migration.
4. If no — identify specifically what's still missing from the original migration plan and
   re-scope the task with a concrete remaining checklist instead of leaving it open-ended.

---

## 2. THQ-002 — Visualizer cube color: graduated health tier vs. binary connected

**DECISION NEEDED — likely superseded.** `ALL_TASKS.md` flags this as "likely superseded" since
THQ-017 already ported the frontend's tier-color fixes into this file 2026-08-09, and HUB_STATE
separately says THQ-002 itself already **shipped** 2026-08-03 (public visualizer polling
`GET /api/v1/ecosystem/public-health`, real tier coloring, single-poll-not-per-cube design
resolved).

**Steps:**
1. Load the live public visualizer (`https://terra-hq.com`) and confirm cubes actually show
   graduated HEALTHY/YELLOW/ORANGE/RED colors, not just binary connected/grey.
2. If confirmed working: mark THQ-002 **Done** in `terra-hq-site/TASKS.md` — it's currently listed
   "Planned," which is stale.

---

## 3. THQ-004 — Design pass follow-ups (per-subsidiary card images, puzzle-piece shape, 3D backdrop)

**Notes only, no code yet — this is legitimately still open, not a contradiction.** Resolved
2026-08-23: after the merge landed the full TASKS.md from `solan`, there is only ONE THQ-004 row in
this file — the "Design pass follow-ups" one below. The separate "Farm Renders tab" work
`HUB_STATE.md` described under the same ID doesn't have its own TASKS.md row here (it may live only
as inline content in `terra_africa_strategy.html` itself, or that HUB_STATE note was describing
work that never got a formal task row) — no actual ID collision to renumber after all.

**What to gather before a design session:** per-subsidiary card images for each of the 8 domain
cubes (need source images or a generation prompt per domain, similar to the Farm Renders approach
already used for Terra Agriculture), a decision on the puzzle-piece shape concept, and a 3D backdrop
asset. None of this needs a specific site — it's asset creation/sourcing, same class of work as the
Farm Renders tab.

---

## 4. THQ-018 — Custom circular cursor (Montfort-style), site-wide

**Design discussion only, not scoped to code yet.** Reference: `https://mont-fort.com/` (the
standing visual north star for terra-hq-site, per Will's own framing — also referenced in Claude
memory as `reference_montfort-design`).

**What to do now:** visit mont-fort.com, study their custom cursor behavior (size, easing, hover
states over links/images), and write a short spec (cursor states needed, which elements trigger
which state) so a future coding session has a clear target instead of "make it like Montfort."

---

## 6. NEW 2026-08-23 — `roms_gtm_strategy.html` link needs to point at OMS's new admin page (sequenced AFTER 05's item 0)

**Don't do this yet.** Once OMS gets its own admin page for the GTM strategy content (see
`05-roms-oms.md` item 0 for the full decision/plan), this repo's link to
`roms_gtm_strategy.html` needs to change from serving the content itself to redirecting/pointing at
the new OMS admin page's live URL. Also relevant to the domain-naming decision (`02-terra-api-infra.md`
item 3) — the target URL will be on OMS's own new domain, not a `terra-hq.com` path, once that
domain exists. Logged here now so it isn't forgotten once the OMS-side work is actually done; no
action possible from this repo alone until then.

---

## 5. Cross-check: is terra-hq-site's own Cloudflare Pages deploy still healthy?

**Why it matters:** HUB_STATE notes this was found silently broken since mid-July 2026 (a
bot-generated `wrangler.jsonc` never merged into `main`) and was fixed in the same session as the
CORS/mixed-content debugging (2026-08-08). Worth a quick spot-check since it's the kind of thing
that fails silently.

**Sites:** Cloudflare dashboard → Pages → the terra-hq-site project.

**Steps:**
1. Open the Cloudflare Pages dashboard, find the terra-hq-site project, confirm the most recent
   deploy matches the latest `main` commit (check the commit hash shown in Cloudflare against
   `git log -1` on `terra-hq-site`'s `main`).
2. If they've drifted again, re-check `wrangler.jsonc` is actually on `main` (not stuck on a
   feature branch again).
