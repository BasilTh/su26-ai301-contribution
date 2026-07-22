# CodePath AI301 Contribution README: Basil Thomas

**Cohort:** AI301 Summer 2026, Section 1C (Wednesdays 4 to 6 PM PT)
**Cycle:** 2 — switched open source project (see note below)
**Status:** Phase IV In Progress — PR opened: [actualbudget/actual #8547](https://github.com/actualbudget/actual/pull/8547); CI running, awaiting maintainer review. (Out sick July 8–~20; resubmitted a re-scoped change.)

---

> **Note — I switched the open source project I'm contributing to.** I began this
> capstone on [apache/burr #276](https://github.com/apache/burr/issues/276) and
> completed Phase I–II there, but the burr maintainer never answered my Phase I
> intro questions (and another contributor had already expressed interest), so
> rather than stall I switched to a live, maintainer-engaged issue:
> **[actualbudget/actual #7391](https://github.com/actualbudget/actual/issues/7391)**.
> This README now tracks the actual/#7391 contribution end to end; the earlier burr
> Phase I–II write-up is preserved in this repo's git history.

---

> **Update (week of July 21) — scope narrowed after part of it merged upstream.** I was
> out sick from July 8 for about two weeks and could not work. While I was out, **PR
> [#8336](https://github.com/actualbudget/actual/pull/8336) merged (July 3)** and added
> scoped ErrorBoundaries to the mobile **budget, accounts, and transaction** pages — part
> of my original mobile scope. When I came back I re-checked `master` and the issue and
> confirmed the mobile **schedules, payees, and bank sync** routes were still unwrapped and
> unclaimed, so I re-scoped my contribution to exactly those three areas and matched the
> route-level pattern #8336 used. This README reflects that re-scoped change.

---

## The Issue

**Issue:** [actualbudget/actual #7391 — [Maintenance] Add scoped ErrorBoundaries to isolate feature-level crashes](https://github.com/actualbudget/actual/issues/7391)
**Repository:** [actualbudget/actual](https://github.com/actualbudget/actual) — a
local-first, open-source personal finance app (TypeScript/React, Yarn 4 monorepo,
MIT license)
**Labels:** `good first issue`, `help wanted`, `tech debt`, `maintenance`
**My fork:** https://github.com/BasilTh/actual
**Work branch:** `fix-7391-mobile-schedules-payees-banksync`
**My scope (re-scoped):** the mobile **schedules**, **payees**, and **bank sync** pages —
the mobile entries in the issue's table that still had no boundary after #8336. The issue
is the umbrella tracker for all platforms and stays open after my PR.

---

## Phase I: Problem Summary

Actual is a local-first personal finance app. When any component throws a rendering
error, the *entire* app crashes to a full-screen "Fatal Error" — because the app has
only two top-level `ErrorBoundary` wrappers (in `App.tsx`) plus a single
report-scoped boundary. A render error in any one feature (budget, accounts,
transactions, schedules, payees, bank sync, …) therefore takes down the whole app
and the user loses all context.

The maintainers documented this as a recurring pattern: 70+ open and closed issues
are fatal crashes that a feature-scoped boundary would have contained. The fix is to
wrap each major feature area in its own `ErrorBoundary` (from `react-error-boundary`,
already a dependency) that shows a contextual, recoverable fallback instead of
crashing the app. This issue is the umbrella; I took the **mobile** slice, and after
#8336 merged the remaining mobile work is schedules, payees, and bank sync.

---

## Phase I: Why I Chose This Issue

After my first issue (apache/burr #276) stalled on an unresponsive maintainer, I
needed a live, engaged issue rather than a waiting game. actual/#7391 is a tight fit:

1. **Squarely my stack.** A pure React + TypeScript frontend change — no backend or
   data-model work — which is exactly the territory I work in daily (recent work:
   Next.js 15 + React 19 + D3 product, D3 scrollytelling).

2. **Maintainer-scoped and pre-approved.** The umbrella issue spans every feature
   area (70+ historical fatal-crash reports). I claimed the mobile half on the
   thread, and the maintainer (@MatissJanis) confirmed "one PR is fine" for all
   mobile pages — so the scope is bounded and agreed, not speculative.

3. **A proven, repeatable pattern.** The repo already merged the same fix for other
   surfaces (#7658 report charts, #7437 rules, #7560 modals, #7497
   budget/accounts/transactions, #7382 widgets, and #8336 for mobile
   budget/accounts/transactions), so there's a clear convention to match rather than
   invent.

4. **Real-world OSS practice.** A large, active monorepo with CI, visual-regression
   tests, a contribution guide, and an explicit AI-usage policy — exactly the
   "codebase you didn't write" the capstone is meant to engage with.

---

## Phase I: Initial Engagement

- Forked actualbudget/actual to my GitHub account.
- Posted a claim comment on #7391 introducing myself as a CodePath AI301 student,
  proposing to take the mobile pages, listing the entry points I'd wrap and how I'd
  verify the fix.
- The maintainer (@MatissJanis) agreed to a single PR for all mobile pages and asked
  me to read the project's AI-usage policy.
- Read the AI-usage policy, then stood up the local dev environment (below).

---

## Phase II: Reproduction & Plan

### Reproduction Process

#### Environment Setup

Stack: TypeScript/React monorepo (Yarn 4 workspaces), Node ≥ 22.

1. Forked actualbudget/actual; `origin` = my fork, `upstream` = actualbudget/actual.
2. Branch: `fix-7391-mobile-schedules-payees-banksync` off the latest `master`.
3. Toolchain: Node ≥ 22 (`.nvmrc` = v22), `corepack enable` for Yarn 4, then
   `yarn install` from the repo root.
4. Run: `yarn start` (Vite dev server, port 3001).
5. In the app: "Don't use a server" → **Try the demo** (a pre-populated budget). Mobile
   emulation via a viewport < 512px (`breakpoints.small`).

#### Steps to Reproduce

On `master`, the fatal-crash behavior for the still-unwrapped mobile routes:

1. Open the demo budget and switch to a mobile viewport (< 512px).
2. Trigger a render error inside mobile schedules, payees, or bank sync — a temporary
   `throw new Error('test')` injected into that page's render is a deterministic baseline.
3. **Expected (desired):** the error is contained to that section with a recoverable
   fallback, header and nav intact.
4. **Actual (before the fix):** the whole app unmounts to the full-screen "Fatal Error."

#### Reproduction Evidence

- **Working branch:** https://github.com/BasilTh/actual/tree/fix-7391-mobile-schedules-payees-banksync
- **Baseline:** confirmed the full-app crash by injecting a temporary `throw` into
  `MobileSchedulesPage` at a mobile viewport; contained after the fix (see Testing).
- **Findings:** on `master`, `/schedules`, `/schedules/:id`, `/payees`, `/payees/:id`,
  `/bank-sync`, and `/bank-sync/account/:accountId/edit` in `FinancesApp.tsx` are bare
  route elements with no boundary, while `/budget`, `/accounts/:id`,
  `/transactions/:transactionId` (from #8336) and `/rules` are already wrapped.
  `react-error-boundary` and the shared `FeatureErrorFallback` already exist — so this is
  a pure rendering/composition change, no backend or data-model work.

### Solution Approach

#### Analysis

The crash propagates because there is no boundary between a mobile feature route and
the app root. The merged fixes — including the just-landed mobile boundaries (#8336) and
the rules boundaries — show the intended pattern: wrap the **route element** in
`FinancesApp.tsx` in an `ErrorBoundary` using the shared `FeatureErrorFallback` with
`resetKeys={[location.pathname]}` so navigation clears the error. Schedules, payees, and
bank sync simply hadn't been done yet.

#### Implementation Plan (UMPIRE)

**Understand:** A render error in mobile schedules/payees/bank sync crashes the entire
app; it should instead be contained to that section with a recoverable fallback, while
the mobile header and nav tabs stay alive.

**Match:** `react-error-boundary` v6 is already a dependency; `FeatureErrorFallback` and
`resetKeys={[location.pathname]}` are already used by the merged `/budget` and `/rules`
boundaries. Match that convention exactly rather than inventing a new one.

**Plan:**
1. Wrap the six mobile routes in `FinancesApp.tsx` (`/schedules`, `/schedules/:id`,
   `/payees`, `/payees/:id`, `/bank-sync`, `/bank-sync/account/:accountId/edit`) in an
   `ErrorBoundary` with `FallbackComponent={FeatureErrorFallback}` and
   `resetKeys={[location.pathname]}`.
2. Add a release note under `upcoming-release-notes/`.

**Implement:** done on `fix-7391-mobile-schedules-payees-banksync` — one commit (see
Phase III).

**Review:** self-review against repo conventions — reuse `FeatureErrorFallback`, match
the merged route-level pattern, no throwaway files, diff scoped to the issue, `[AI]`-free
human PR (I author the PR text myself) per the project's AI-usage policy, with an explicit
AI-disclosure line.

**Evaluate:** manual crash/recovery test at a mobile viewport plus `yarn typecheck` (see
Testing Strategy).

**Design decision — route-level vs inside-page-chrome.** I first implemented these
boundaries *inside* each page's chrome, worried that a route-level boundary would kill the
mobile nav (the `Page` component portals its header). On re-checking `master` I saw #8336
placed the merged mobile boundaries at the **route level**, and that the mobile nav
(`MobileNavTabs`) renders in a **separate route tree** in `FinancesApp.tsx` — so a
route-level boundary does *not* take down the header/nav. I switched to the route-level
approach: a smaller diff that matches the just-merged convention.

---

## Phase III: Implementation

### What I built

Route-level scoped error boundaries around the mobile schedules, payees, and bank sync
routes, so a render error is contained to that section (showing a recoverable "Try again"
fallback) instead of crashing the entire app — matching the pattern merged for mobile
budget/accounts/transactions (#8336) and rules.

- **Wrapped six routes** in `packages/desktop-client/src/components/FinancesApp.tsx`:
  `/schedules`, `/schedules/:id`, `/payees`, `/payees/:id`, `/bank-sync`, and
  `/bank-sync/account/:accountId/edit` — each in
  `<ErrorBoundary FallbackComponent={FeatureErrorFallback} resetKeys={[location.pathname]}>`.
- **Reused the shared `FeatureErrorFallback`** and the existing `ErrorBoundary` /
  `location` imports already in the file. No new components, no new dependencies.

### Code Changes

- **Branch:** https://github.com/BasilTh/actual/tree/fix-7391-mobile-schedules-payees-banksync
- **Commit:** `27f28f3` — *"fix: add scoped ErrorBoundaries to mobile schedules, payees,
  and bank sync pages"* (authored as me; no `[AI]` prefix, no Co-Authored-By, per the
  human-contributor track of the AI-usage policy).
- **Files:** `packages/desktop-client/src/components/FinancesApp.tsx` (+48 / −12) and a
  new release note `upcoming-release-notes/mobile-error-boundaries-schedules-payees-banksync.md`.

### Testing Strategy

- **Type checking:** `yarn typecheck` passes across all 10 packages (0 errors).
- **Manual, mobile viewport (primary):** drove the running dev app in a headless browser at
  a **360×720** mobile viewport on the demo budget. Injected a temporary render `throw`
  into `MobileSchedulesPage` and confirmed the crash is **contained** — the
  `FeatureErrorFallback` ("Something went wrong loading this section." + "Try again")
  renders while the **mobile header and bottom nav stay interactive**, instead of the
  full-app "Fatal Error." Removed the throw and confirmed the real Schedules list renders
  normally. (Screenshots captured for the before/after.) Payees and bank sync are the
  identical route-level wrap.
- **CI (on the PR):** the mobile Playwright suites and the VRT shards that mobile changes
  trigger run automatically.

---

## Phase IV: Pull Request & Iteration

_In progress — PR opened at [actualbudget/actual #8547](https://github.com/actualbudget/actual/pull/8547); awaiting review._

**Repo state re-verified before submitting.** After being out sick, I re-fetched
`upstream/master` and confirmed: `master` had not advanced past my branch base, the three
mobile routes were still unwrapped, no competing PR touched them, and no one else had
claimed them on #7391 after me. Then `yarn typecheck` clean (10/10) and the manual mobile
test above passed.

**PR plan.** Open the PR myself as a human contributor (no `[AI]` prefix), filling out the
project's PR template — Description / Related issue(s) / Testing / Checklist — in my own
words in English, with an explicit AI-disclosure line per the project's
[AI-usage policy](https://actualbudget.org/docs/contributing/ai-usage-policy). (The `[AI]`
prefix and blank-template rules apply to *autonomous* agents; human contributors using AI
assistance disclose and author the PR themselves.)

**PR title:** `Add scoped ErrorBoundaries to mobile schedules, payees, and bank sync pages`

**PR description (draft for GitHub):**

> **Description** — The mobile schedules, payees, and bank sync pages were the remaining
> entries in the #7391 mobile table without a scoped error boundary; a render error in any
> of them currently crashes the whole app to the "Fatal Error" screen. This wraps those
> routes in `FinancesApp.tsx` in an `ErrorBoundary` using the shared `FeatureErrorFallback`
> with `resetKeys={[location.pathname]}`, so a crash is contained to that section (with a
> "Try again") while the mobile header and nav stay usable. Same route-level pattern already
> merged for mobile budget/accounts/transactions (#8336) and rules — no new components or
> dependencies.
>
> *AI disclosure: I used Claude (Anthropic's AI) to help write and review this change. I
> read through the diff, tested it at a mobile viewport, and verified the behavior myself.*
>
> **Related issue(s)** — Relates to #7391 (mobile schedules, payees, and bank sync; the
> umbrella issue stays open for any remaining areas).
>
> **Testing** — At a mobile viewport (360×720) on the demo budget, injected a render error
> into a wrapped mobile page and confirmed the crash is contained to that section (fallback
> + "Try again") with the header/nav still interactive, instead of the full-app Fatal Error;
> removed the throw and confirmed normal rendering. `yarn typecheck` passes (10/10).

**PR Link:** https://github.com/actualbudget/actual/pull/8547

**Maintainer Feedback:** _(pending)_

**Status:** Open ([#8547](https://github.com/actualbudget/actual/pull/8547)) — CI running; awaiting maintainer review.

---

## Learnings & Reflections

- **Check the repo state before you submit.** Two weeks out sick, and part of my original
  scope had already merged (#8336). Re-verifying `master` turned a would-be redundant PR
  into a clean, still-needed one — and I did the same check again right before pushing.
- **Match the merged convention over your first design.** Route-level boundaries were the
  accepted pattern; switching to them (once I confirmed the mobile nav is a separate route
  tree, so nav survives) made the change smaller and more mergeable.
- **Disclose AI use and stay the human in the loop.** The project welcomes AI-assisted work
  as long as a human understands, tests, and authors the submission — so I verified the
  behavior myself and wrote the PR in my own words with a disclosure line.
