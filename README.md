# CodePath AI301 Contribution README: Basil Thomas

**Cohort:** AI301 Summer 2026, Section 1C (Wednesdays 4 to 6 PM PT)
**Cycle:** 2 — switched open source project (see note below)
**Status:** Phase III Complete

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

## The Issue

**Issue:** [actualbudget/actual #7391 — [Maintenance] Add scoped ErrorBoundaries to isolate feature-level crashes](https://github.com/actualbudget/actual/issues/7391)
**Repository:** [actualbudget/actual](https://github.com/actualbudget/actual) — a
local-first, open-source personal finance app (TypeScript/React, Yarn 4 monorepo,
MIT license)
**Labels:** `good first issue`, `help wanted`, `tech debt`, `maintenance`
**My fork:** https://github.com/BasilTh/actual
**Work branch:** `fix-issue-7391`
**My scope:** the **mobile** pages (the issue is the umbrella tracker for all
platforms and stays open after my PR)

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
crashing the app. This issue is the umbrella; I'm taking the **mobile** slice.

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
   budget/accounts/transactions, #7382 widgets), so there's a clear convention to
   match rather than invent.

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

1. Forked actualbudget/actual; rewired the existing clone —
   `git remote rename origin upstream`, then
   `git remote add origin https://github.com/BasilTh/actual.git`.
2. Branch: `git checkout -b fix-issue-7391`.
3. Toolchain: Node ≥ 22 (`.nvmrc` = v22), `corepack enable` for Yarn 4, then
   `yarn install` from the repo root (Husky hooks install via `prepare`).
4. Run: `yarn start` (Vite dev server, port 3001). **Windows flag:** `yarn start`
   chains a POSIX shell script, so it needs Git Bash `sh` on PATH; fallback is
   `yarn workspace @actual-app/web run start --mode=browser`.
5. In the app: "Don't use a server" → **View demo** (a pre-populated budget). Mobile
   emulation via DevTools viewport < 512px (`breakpoints.small`).

#### Steps to Reproduce

On `master`, the fatal-crash behavior:

1. Open the demo budget and switch to a mobile viewport (< 512px).
2. Trigger a render error inside a mobile feature page — either one of the issue's
   linked mobile crashes (#7108, #6650, #6467, #5692, #4204, #3263) where still
   reproducible, or a temporary `throw new Error('test')` injected into a mobile
   page's render as a deterministic baseline.
3. **Expected (desired):** the error is contained to that section with a recoverable
   fallback, header and nav intact.
4. **Actual:** the whole app unmounts to the full-screen "Fatal Error" and the user
   loses all context.

#### Reproduction Evidence

- **Working branch:** https://github.com/BasilTh/actual/tree/fix-issue-7391
- **Baseline:** confirmed the full-app crash by injecting a temporary `throw` into a
  mobile page at a mobile viewport; before/after captures will be attached to the PR.
- **Findings:** the app has only two top-level boundaries + one report boundary, so
  nothing contains a mobile feature crash. `react-error-boundary` (v6.0.3) and a
  shared `FeatureErrorFallback` already exist and are used by the merged desktop
  boundaries — so this is a pure rendering/composition change, no backend or
  data-model work.

### Solution Approach

#### Analysis

The crash propagates because there is no boundary between a mobile feature page and
the app root. The merged desktop fixes (reports, rules, modals,
budget/accounts/transactions, widgets) show the intended pattern: wrap each feature
in an `ErrorBoundary` using the shared `FeatureErrorFallback`, with `resetKeys` so
navigation clears the error. The mobile pages simply hadn't been done yet. One
subtlety: the mobile `Page` component *portals* its header, so a boundary placed at
the route element would also kill the header/nav on crash — the boundary must sit
*inside* the page chrome.

#### Implementation Plan

Using the UMPIRE framework (adapted):

**Understand:** A render error in any mobile feature page crashes the entire app; it
should instead be contained to that page with a recoverable fallback, while the
mobile header and nav tabs stay alive.

**Match:** `react-error-boundary` v6.0.3 is already a dependency; `FeatureErrorFallback`
and the `resetKeys={[location.pathname]}` pattern are already used by the merged
desktop boundaries (e.g. the report boundary). I match that convention exactly
rather than inventing a new one.

**Plan:**
1. Add a shared `MobilePageBoundary` helper that wraps `ErrorBoundary` with
   `FallbackComponent={FeatureErrorFallback}` and `resetKeys={[useLocation().pathname]}`.
2. Wrap each mobile feature page *inside* its page chrome — transactions, budget,
   accounts, transaction-edit, schedules, payees, bank sync — one commit per area.
3. No `onError` wiring (the merged precedents don't use it; match real convention).
4. Add a release note under `upcoming-release-notes/`.

**Implement:** done on `fix-issue-7391` — 8 commits, working tree clean (see
Phase III).

**Review:** self-review against repo conventions — TS-strict (no `@ts-strict-ignore`
on new files), reuse `FeatureErrorFallback`, no throwaway files, diff scoped to the
issue, descriptive commit messages, and the PR write-up left to me personally per the
project's AI-usage policy.

**Evaluate:** per-area manual throw test at a mobile viewport (contained fallback +
live header/nav + "Try again" reset), plus `yarn typecheck` and the desktop-client
unit suite (see Testing Strategy below).

---

## Phase III: Implementation

### Implementation Notes — Week 3 Progress

**What I built:** scoped error boundaries around every mobile feature page, so a
render error is contained to that section (showing a recoverable "Try again"
fallback) instead of crashing the entire app.

- **New shared helper** — `packages/desktop-client/src/components/mobile/MobilePageBoundary.tsx`:
  wraps `ErrorBoundary` with `FallbackComponent={FeatureErrorFallback}` and
  `resetKeys={[useLocation().pathname]}`, so navigating away clears the error.
- **Wrapped the mobile feature pages** — transactions, budget, accounts,
  transaction-edit, schedules, payees, and bank-sync — placing each boundary
  *inside* the page chrome so `MobilePageHeader` + `MobileNavTabs` stay alive when a
  section crashes.
- **Matched existing convention exactly:** no `onError` wiring (the merged
  precedents don't use it), reused `FeatureErrorFallback`, added no throwaway files.

**Challenges / decisions:**
- **Boundary placement.** A route-element boundary would have killed the nav bar (the
  `Page` component portals its header); placing the boundary inside page chrome keeps
  header + nav interactive — verified per area.
- **Test needed a router.** The boundary calls `useLocation()`, so
  `MobilePayeesPage.test.tsx` started failing for lack of router context; I wrapped
  its render in `<MemoryRouter>`.
- **Windows lint trap.** `yarn lint:fix` CRLF-rewrites thousands of files on my
  machine, so I rely on CI's oxlint/oxfmt rather than the local autofix.

### Code Changes

- **Branch:** https://github.com/BasilTh/actual/tree/fix-issue-7391 — 8 commits,
  working tree clean.
- **Files:** 13 changed (+558 / −498). New `MobilePageBoundary.tsx`; boundaries
  added to `AccountsPage`, `BudgetPage`, `TransactionListWithBalances`,
  `TransactionEdit`, `MobileSchedulesPage` + `MobileScheduleEditPage`,
  `MobilePayeesPage` + `MobilePayeeEditPage`, `MobileBankSyncPage` +
  `MobileBankSyncAccountEditPage`; test fix in `MobilePayeesPage.test.tsx`; release
  note `upcoming-release-notes/7391.md`.
- **Key commits:**
  - `d01c02b` Add scoped ErrorBoundary to mobile transaction list
  - `3ac1e3a` Wrap mobile budget page table in a scoped ErrorBoundary
  - `f9170b3` Wrap mobile accounts list in a scoped ErrorBoundary
  - `ed12062` Wrap mobile transaction edit in a scoped ErrorBoundary
  - `2eb8293` Wrap mobile schedules pages in scoped ErrorBoundaries
  - `33e9941` Wrap mobile payees pages in scoped ErrorBoundaries
  - `df17193` Wrap mobile bank-sync pages in scoped ErrorBoundaries
  - `ba4aaba` Add release note for mobile error boundaries

### Testing Strategy

- **Manual, per area:** at a mobile viewport, inject a temporary `throw` into each
  wrapped page → confirm the contained `FeatureErrorFallback` renders while the
  header/nav stay live → confirm "Try again" (`resetErrorBoundary` keyed on
  `pathname`) recovers → remove the throw → commit.
- **Unit:** updated `MobilePayeesPage.test.tsx` to render inside `<MemoryRouter>` so
  the boundary's `useLocation()` resolves.
- **Gates (verified locally):** `yarn typecheck` clean across all 10 packages
  (0 errors); `yarn workspace @actual-app/web run test` green — **42 files, 686
  passed / 1 skipped / 0 failed**, including the updated `MobilePayeesPage.test.tsx`.
  `yarn lint` is left to CI (local `lint:fix` CRLF-rewrites the tree on Windows). CI
  on the PR also runs the 9 `*.mobile.test.ts` Playwright suites (350×600 viewport)
  and the VRT shards that mobile changes trigger.

---

## Phase IV: Pull Request & Iteration

_In progress._ Opening the PR to actualbudget/actual personally — a normal title (no
`[AI]` prefix, per the project's AI-usage policy for human-reviewed work), a
human-written description plus an AI-disclosure line, leaving the PR template
untouched. Once the PR number exists, rename `upcoming-release-notes/7391.md` →
`<PR#>.md`. Then keep the branch current with `master` and respond to maintainer
review personally.
