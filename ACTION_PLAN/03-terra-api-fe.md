# 03 — terra-api-fe (TFE-502/503/601/602/603/604/605 + Notion Phase C)

All of these are code changes — per the "do not touch the code" instruction, this file documents
what each one needs and what to gather/decide BEFORE writing any code, not the code itself. Treat
this as prep work you can do solo (design decisions, asset gathering, confirming the exact bug
repro) so the actual implementation session is fast when you get to it.

---

## 0. Notion Phase C — verify login + terra-hq-site login buttons + live visualizer endpoint

**Why it matters:** `ALL_TASKS.md` flags this Notion row as "Related repo work (TFE-501 done,
TFE-502/503 still open) suggests this is partially done already" — worth a direct verification pass
rather than leaving it as an open Notion row indefinitely.

**Steps (all just using the live site, no code):**
1. Visit `https://terra-hq.com` (or wherever terra-api-fe's login lives) and confirm login works
   end-to-end for a real account.
2. Check terra-hq-site's pages for login buttons/links — confirm they actually point at the live
   login flow, not a dead/placeholder link.
3. Open the public visualizer and confirm (via browser DevTools → Network tab) it's genuinely
   polling `GET /api/v1/ecosystem/public-health` and not stale/cached/mocked data.
4. If all three check out, update the Notion Phase C row to Done (or close whatever sub-pieces are
   confirmed, leaving only genuinely-unverified pieces open).

---

## 1. TFE-502 — Redirect unauthenticated users to login instead of a broken page

**What to confirm/gather before coding:** reproduce it yourself — visit a protected route
(`/dashboard` or `/internal`-equivalent tab) while logged out, confirm what currently renders (a
blank/broken page vs. silently nothing). Decide the exact redirect target (`/login`) and whether to
preserve the originally-requested URL as a `?redirect=` param so login sends the user back to where
they were headed.

**No sites needed** — just the live app in a browser, logged out.

---

## 2. TFE-503 — Expand frontend test coverage

**What to gather:** open `terra-api-fe/TASKS.md` and any test files already present
(`ProtectedRoute.test.js`, `Login.test.js` were added 2026-08-08) to see what's already covered.
List which of the 9 ApiDashboard tabs and which hooks (`useOperatorEcosystem.js`,
`useEcosystemHealth.js`) have zero test coverage today — that list becomes the actual scope for
whoever picks this up.

**No sites needed.**

---

## 3. TFE-601 — "My Services" / "Ecosystem" tab split (deliberately shelved, design already brainstormed)

**Why it matters:** today ALL 8 domain cubes + launchpad cards render identically for every
customer regardless of entitlement — `customer_service_access` only ever changes cube/card COLOR,
never visibility. The design direction is already agreed (two tabs inside `Dashboard.js`: "My
Services" filtered to entitled products, "Ecosystem" = today's full unfiltered view) — this is
shelved for a dedicated session, not because the design is unclear.

**What to do now (no code):** re-read the full design brainstorm already captured in
`terra-api-fe/TASKS.md` row TFE-601 before the dedicated session starts, and decide the "honest
empty state for zero entitlements" copy/treatment for the new "My Services" tab ahead of time —
that's the one open UX detail not already speced.

---

## 4. TFE-602 — Replace placeholder branding (nav logo + favicon)

**Blocked on Will's real designs existing** — this is the one item in this file that's a pure
asset/design gate, not a coding gate.

**What to do now (no code):**
1. Produce or commission: (a) an animated nav-logo mark for `ApiDashboard.js`'s
   `.nav-brand-placeholder` span, (b) a favicon set — `favicon.ico`, `logo192.png`, `logo512.png`.
2. Once real assets exist, this becomes a same-day drop-in (swap the placeholder span, replace the
   three files in `public/` in place, same filenames — no code changes needed for the favicon half
   per the task's own note).

**Sites:** wherever Will produces/commissions designs (design tool of choice) — no specific URL
prescribed by the hub.

---

## 5. TFE-603 — JWT session expiry has no user-facing handling

**Why it matters:** real UX bug — tokens expire after 1 hour, no refresh mechanism, and the
frontend just shows a permanent "STATUS UNAVAILABLE / Reconnecting" state that never resolves,
instead of prompting re-login. Confirmed: logging out/in immediately fixes it.

**Decision needed before coding:** pick (a) a real refresh-token flow (bigger — touches
`AuthService`/`TokenIssuer` on the backend too) vs. (b) minimum viable — detect a 401 from an
authenticated fetch and show "Session expired — log in again." Given this deserves "its own focused
pass" per the hub's own note, use this prep time to decide which of (a)/(b) you actually want before
the coding session starts, since it changes scope significantly (backend + frontend vs.
frontend-only).

**No sites needed** — this is a design decision, do it on paper/in your notes.

---

## 6. TFE-604 — Menu popover feature ideas (captured, not scoped)

**Not urgent — no action needed until the popover redesign gets picked up again.** Three
independent, additive ideas already captured in `terra-api-fe/TASKS.md`: account info display,
quick links to `/dashboard`/`/login`, live status dot. Nothing to prep — they're already
well-specified. Just don't lose the note.

---

## 7. TFE-605 — Cube slow-pulse animation

**What to clarify before coding (this is explicitly flagged as ambiguous in the task itself):**
confirm with yourself (or by testing) whether `shouldPulse(status)` (already wired in
`terraScene.js`/`healthColors.js` for health-driven pulsing) is actually firing/looking right today,
or whether this ask is about a SEPARATE always-on ambient pulse the original phase5 visualizer had
independent of health state. Test in a browser first — load the visualizer, watch a cube in each
health tier for ~10 seconds, note what pulsing (if any) you actually see — before scoping the real
animation-curve work (a per-cube sine/easing driver, offset per cube so they don't pulse in
lockstep — genuinely nontrivial math per Will's own framing).

---

## 8. Sign-out button looks wrong — ROOT-CAUSED 2026-08-23 (read-only code investigation, nothing edited)

**Found it.** `src/internal/ApiDashboard.js` line 114:
```jsx
<button type="button" className="theme-toggle nav-signout" onClick={logout} title="Sign out">⏻</button>
```
It reuses the `theme-toggle` class (same circular-button styling as the theme toggle right next to
it) plus a `nav-signout` modifier. In `api-dashboard.css` (lines 150–152):
```css
/* nav-signout is React-only (no HTML equivalent) — same circular affordance as theme-toggle,
   just a different glyph/action, so it reads as part of the same control cluster. */
.api-shell .nav-signout { font-size: 12px; }
```
This was a **deliberate design choice** (per the comment) to make sign-out look like part of the
same control cluster as the theme toggle — not a bug in the sense of unintended code. The glyph is
`⏻` (U+23FB, the ISO "standby/power" symbol). Likely candidates for why it "looks wrong" in
practice, ranked by likelihood:
1. **Font rendering inconsistency** — `⏻` is a less-common Unicode symbol; some system fonts render
   it as a generic/blank glyph, oddly proportioned, or vertically misaligned within the circular
   button, especially at the small `12px` override (the theme toggle's own icon size isn't
   overridden the same way — check `.theme-icon`'s actual rendered size against `.nav-signout`'s to
   see if they visually mismatch each other, which would look inconsistent side-by-side).
2. **Glyph choice itself** — a standby/power icon reads as "power off the app" to some users rather
   than "sign out of your account," which may be what looks "wrong" semantically rather than visually.

**What to decide before coding:**
1. Open the live app and look at the button directly — confirm which of the two above (or
   something else entirely) is the actual complaint.
2. If it's the glyph/rendering: swap `⏻` for a more universally-supported icon (a proper icon font
   already used elsewhere on the page, if one exists, beats another raw Unicode glyph) or a small
   inline SVG, sized to visually match `.theme-icon` exactly.
3. If it's semantic (power icon reads wrong for "sign out"): pick a clearer glyph/label (a door/arrow
   icon is the more conventional "sign out" affordance) — still keep the circular `theme-toggle`
   affordance if that visual language is otherwise working, per the original comment's intent.
4. Once decided, this is a small, well-scoped CSS/markup fix — log it as its own task ID (e.g.
   TFE-606) in `terra-api-fe/TASKS.md`.

**No sites needed** — code already read; just compare against the live rendered button to confirm
which failure mode this actually is.

---

## 9. Jenkinsfile webhook confirmation (small, not code — just a GitHub setting check)

**Why it matters:** `HUB_STATE.md` notes the GitHub webhook payload URL for terra-api-fe
(`http://3.211.62.86:8090/github-webhook/`) was identified but "not yet confirmed added on GitHub,"
and the `terra-api-fe-main`/`-branches` Jenkins jobs weren't confirmed to have "GitHub hook trigger
for GITScm polling" enabled.

**Sites:** GitHub repo settings (`github.com/will55555/terra-api-fe/settings/hooks`), Jenkins job
config pages (`http://3.211.62.86:8090`).

**Steps:**
1. On GitHub, confirm a webhook exists pointing at `http://3.211.62.86:8090/github-webhook/`
   (Content type `application/json`, triggered on push).
2. In Jenkins, open the `terra-api-fe-main` and `terra-api-fe-branches` (or equivalent multibranch)
   job configs and confirm "GitHub hook trigger for GITScm polling" is checked.
3. Test by pushing a trivial commit and confirming Jenkins picks it up automatically instead of
   needing a manual "Scan Multibranch Pipeline Now."
