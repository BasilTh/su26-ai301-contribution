# CodePath AI301 Contribution README: Basil Thomas

**Cohort:** AI301 Summer 2026, Section 1C (Wednesdays 4 to 6 PM PT)

| Cycle | Issue | Status |
|-------|-------|--------|
| 1 | [apache/burr #276](https://github.com/apache/burr/issues/276) — link read/write state on telemetry UI | Paused at Phase II — maintainer unresponsive; pivoted to Cycle 2 |
| 2 | [actualbudget/actual #7391](https://github.com/actualbudget/actual/issues/7391) — scoped ErrorBoundaries (mobile) | **Phase III Complete** |

---

# Cycle 2 — actual #7391: Mobile ErrorBoundaries

**Issue:** [actualbudget/actual #7391 — Add scoped ErrorBoundaries to mobile pages](https://github.com/actualbudget/actual/issues/7391)
**Repository:** [actualbudget/actual](https://github.com/actualbudget/actual) — local-first personal finance app (TypeScript/React, Yarn 4 monorepo, MIT)
**My fork:** https://github.com/BasilTh/actual
**Work branch:** [`fix-issue-7391`](https://github.com/BasilTh/actual/tree/fix-issue-7391)
**Status:** Phase III Complete — 8 commits pushed, working tree clean; PR pending my personal write-up

---

## Phase I (Cycle 2): Why I Chose This Issue

After Cycle 1 (apache/burr #276) stalled — the maintainer never answered my intro
questions and another contributor had already expressed interest — I pivoted to a
live, maintainer-engaged issue instead of waiting. actual/#7391 is a textbook fit:

- **Squarely my stack.** A pure React + TypeScript frontend change (no backend or
  data-model work), exactly the territory I work in daily.
- **Maintainer-scoped and engaged.** The umbrella issue spans every feature area
  (70+ historical fatal-crash reports). I claimed the mobile half on the issue
  thread, and the maintainer (@MatissJanis) confirmed "one PR is fine" for all
  mobile pages — so the scope is bounded and pre-approved rather than speculative.
- **A proven, repeatable pattern.** The repo already merged the same fix for other
  surfaces (#7658 report charts, #7437 rules, #7560 modals, #7497
  budget/accounts/transactions, #7382 widgets), so there's a clear convention to
  match rather than invent.

---

## Phase II (Cycle 2): Reproduction & Plan

**The bug:** Actual has only two top-level `ErrorBoundary` wrappers (in `App.tsx`)
plus one report-scoped boundary. A render error *anywhere* in a mobile feature page
therefore unmounts the whole app to a full-screen "Fatal Error," losing the user's
context.

**Reproduction (baseline on `master`):** In the demo budget at a mobile viewport
(<512px, `breakpoints.small`), I confirmed the fatal-screen behavior by injecting a
temporary `throw new Error('test')` into a mobile page's render and observing the
full-app crash. Before/after captures will be attached to the PR.

**Plan:** Wrap each mobile feature page in a scoped `ErrorBoundary` (from
`react-error-boundary`, already a v6.0.3 dependency) using the shared
`FeatureErrorFallback` and `resetKeys={[location.pathname]}`, placed **inside** the
page chrome so the mobile header and nav tabs survive a contained crash — one commit
per feature area. Full plan: `.planning/actual/PLAN_7391.md` (local).

---

## Phase III (Cycle 2): Implementation

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
  precedents don't use it), reused `FeatureErrorFallback` for consistency, added no
  throwaway files.

**Challenges / decisions:**
- **Boundary placement.** A route-element boundary would have killed the nav bar on
  crash (the `Page` component portals its header). Placing the boundary inside page
  chrome keeps header + nav interactive — verified per area.
- **Test needed a router.** The new boundary calls `useLocation()`, so
  `MobilePayeesPage.test.tsx` started failing for lack of router context; I wrapped
  its render in `<MemoryRouter>` to fix it.
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
  `yarn lint` is left to CI (local `lint:fix` CRLF-rewrites the tree on Windows).
  CI on the PR also runs the 9 `*.mobile.test.ts` Playwright suites (350×600
  viewport) and the VRT shards that mobile changes trigger.

### What's left (→ Phase IV)

Open the PR to actualbudget/actual personally — normal title (no `[AI]` prefix),
human-written description + AI-disclosure line, leaving the template untouched —
then rename `upcoming-release-notes/7391.md` to the PR number and respond to review.

---

# Cycle 1 — apache/burr #276: link read/write state on telemetry UI

> **Status: paused at Phase II.** Pivoted build effort to Cycle 2 (above) after the
> maintainer went unresponsive. Phase I + II below are preserved as the documented
> Cycle 1 journey.

## The Issue

**Issue:** [apache/burr #276 - Add linking of read/write on telemetry UI](https://github.com/apache/burr/issues/276)
**Repository:** [apache/burr](https://github.com/apache/burr) - the Apache Software Foundation's framework for building 
stateful, observable AI agents and workflows
**Labels:** `good first issue`, `kind/improvement`, `area/ui`, `priority/low`
**My fork:** https://github.com/BasilTh/burr
**Work branch:** `feature/276-highlight-state-io`

---

## Phase I: Problem Summary

Burr is a Python framework for building stateful agent applications, and its
companion telemetry UI lets developers replay an agent run step by step. The
UI shows each `action` and the `state` fields it reads or writes - but today
the reads/writes are static labels. There's no way to click a state field
(e.g. `prompt`) and see which *other* actions produced it or consumed it
downstream. For larger agent graphs, this makes it hard to trace where a piece
of state originated or where it flows next.

The fix is a frontend improvement: when a user selects a state field, the
telemetry UI highlights the other actions in the graph that read or write that
same field, making the producer/consumer relationship visible inline. All the
data needed (each action's `reads` and `writes` arrays) is already present in
the existing telemetry model, this is purely a rendering and interaction state
change in the React UI, not a backend or data model change.

---

## Phase I: Why I Chose This Issue

I picked apache/burr #276 because it lines up tightly with my strongest skills
while plugging me into a real AI-adjacent codebase. The work is a React +
TypeScript frontend change on top of [React Flow](https://reactflow.dev/) and
Dagre, which is exactly the territory I've been working in for the past year,
my recent projects include a Next.js 15 + React 19 + D3 v7 product (limitless.io)
and a heavily customized D3 scrollytelling project (CSE 578 group project),
so SVG layout, graph interactions, and React state management are muscle memory.

Three things made this the right pick over alternatives I considered:

1. **AI-thematic fit for AI301.** Burr is the Apache Software Foundation's
   framework for stateful AI agents - exactly the kind of "real AI engineering
   tooling" the capstone is meant to engage with. Several other candidates I
   evaluated (e.g. Apache Superset chart fixes, Medusa admin bugs) had stronger
   brand recognition but weaker thematic alignment.

2. **Bounded, well scoped problem.** The maintainer's issue description lays
   out three concrete approach options. The data needed (each action's reads /
   writes arrays) already exists in `ActionModel`. This means no architectural
   debate - the work is a rendering + interaction layer change in a focused
   surface (~3 files in `telemetry/ui/src/components/routes/app/`).

3. **Genuine skills match.** The telemetry UI stack - `reactflow ^11.10.4`, 
   `dagre ^0.8.5`, React 18 + TypeScript, React Query,Tailwind - overlaps 
   almost completely with what I work in daily. I can ramp on the Burr 
   specific code without simultaneously learning a new framework or build system.

**Note on overlap:** A community contributor (`@hannguyen0712`) expressed
interest 8 days ago without follow up from the maintainer. I posted a comment
on the issue introducing myself, acknowledging their prior interest, and
offering to coordinate or defer if they've started. My fallback if the
maintainer prefers them is [apache/burr #402](https://github.com/apache/burr/issues/402)
("display state reads/writes on newline") - same repo, same area, unraced for
8 months, and equally well aligned with my SVG/layout strengths.

---

## Phase I: Initial Engagement

- Forked apache/burr to my GitHub account
- Created this Contribution README repository
- Posted intro comment on the issue introducing myself as a CodePath AI301
  student, acknowledging @hannguyen0712's prior interest, and asking three
  clarifying questions (graph library, highlight visual treatment, selection
  state location)
- Awaiting maintainer (@elijahbenizzy) response

---

## Phase II: Reproduction & Plan

### Reproduction Process

#### Environment Setup

Stack: Python backend (`burr` server) + React 18/TypeScript telemetry UI (CRA).

1. Python: `uv venv .venv` then `uv pip install -e ".[start]"` from the repo root
2. UI: `npm install` from `telemetry/ui/`
3. Demo data: ran `examples/multi-modal-chatbot` once to populate `~/.burr`

**Windows-specific blocker (solved):** `burr/examples` is a git *symlink*
(`-> ../examples`). Windows checks it out as a plain text file containing the
literal path, so the server crashed on `import burr.examples`. Fix: delete the
file, recreate it as an NTFS junction (`cmd /c mklink /J burr\examples examples`),
then `git update-index --skip-worktree burr/examples` so git ignores the local
difference. Documented here because any Windows contributor will hit this.

**Local lint quirk (not a bug):** with `core.autocrlf=true`, `npm run lint`
reports ~1300 prettier `Delete ␍` errors (CRLF on disk, LF in repo). Linux CI
doesn't see them. Local check that matches CI:
`npx prettier --check --end-of-line auto "src/**/*.tsx"`.

#### Steps to Reproduce

On upstream `main`, the issue behavior (missing feature):

1. Terminal 1 (repo root): `.\.venv\Scripts\Activate.ps1`, set
   `BURR_SERVE_STATIC=false`, run `burr --no-open`
2. Terminal 2: `cd telemetry/ui`, set `DISABLE_ESLINT_PLUGIN=true`, run
   `npx react-scripts start`
3. Open `localhost:3000` -> Projects -> `demo_chatbot` -> app `chat-1-giraffe`
4. Click the `check_safety` step, open the Code tab — the action's state
   reads/writes render as chips (e.g. `prompt`)
5. Click the `prompt` chip
6. **Expected (per #276):** the graph highlights which actions read or write
   `prompt`, so you can trace where state flows
7. **Actual:** nothing happens — the chips are static labels with no
   interaction

#### Reproduction Evidence

- **Working branch:** https://github.com/BasilTh/burr/tree/feature/276-highlight-state-io
- **Screenshots:** before (no selection) / after (`prompt` selected, 1 writer
  ringed red, 6 readers ringed blue) — captured via Playwright
- **Findings:** all data needed already exists client-side — each
  `ActionModel` carries `reads`/`writes` arrays — so this is purely a
  rendering/interaction change in the React UI. No backend or data-model work.

### Solution Approach

#### Analysis

The chips in `ActionView.tsx` (rendered by `ChipGroup` in `common/chip.tsx`)
have no click handler, and `GraphView.tsx`'s `ActionNode` has no notion of a
"selected state field" — its `NodeStateProvider` context only tracks
hover/selection of *actions*. Nothing connects the state-field labels to the
graph nodes.

#### Implementation Plan

**Understand:** The telemetry UI shows what state each action reads/writes,
but can't answer the inverse question: "given a state field, which actions
produce or consume it?" Selecting a field should highlight its readers and
writers in the graph.

**Match:** `ActionNode` already does conditional styling driven by
`NodeStateProvider` context (hover/current-action rings) — the highlight
mechanism extends that existing pattern. `AppView.tsx` already persists UI
state in the URL via `useSearchParams`, which the selected field follows.

**Plan:**
1. `common/chip.tsx` — give `ChipGroup` an optional `onChipClick` +
   `selectedChip`; ring the selected chip
2. `ActionView.tsx` — wire chips to selection; re-clicking the selected chip
   toggles it off
3. `AppView.tsx` — own the selection as a `state_field` URL param
   (`useSearchParams`, `replace: true`) so selection survives reload and is
   shareable
4. `StateMachine.tsx` — thread the field down to both `ActionView` and the
   graph tab
5. `GraphView.tsx` — extend `NodeStateProvider` with `selectedStateField`;
   `ActionNode` rings red (`ring-dwred`) when it *writes* the field, blue
   (`ring-dwdarkblue`) when it *reads* it

Design choice — URL param over local `useState`: survives navigation between
tabs, makes selections shareable/bookmarkable, and matches how the app
already manages view state.

**Implement:** done on the working branch — commit `28cc319`
`feat(ui): link read/write state fields to graph highlighting`
(5 files, +82/-8).

**Review:** self-review against repo conventions: conventional-commit title
(`feat(ui): ...`, becomes the squash commit), `Fixes #276` in the body, no
`console.*`, no new files (avoids ASF license-header/RAT exposure), and
manual dependency-array review since `react-hooks/exhaustive-deps` is
disabled in this repo.

**Evaluate:** `tsc --noEmit` clean, prettier clean, CRA compiles. Verified
end-to-end with Playwright on `chat-1-giraffe`: clicking the `prompt` chip
adds `&state_field=prompt` to the URL; DOM shows red ring on the 1 writer
(`prompt`) and blue rings on 6 readers (`check_safety`, `decide_mode`,
`generate_image`, `generate_code`, `answer_question`, `prompt_for_more`);
re-clicking clears all rings and the URL param. The telemetry UI has CRA's
default test tooling (`react-scripts test`, testing-library) but no existing
test files to extend, and the change is frontend-only, so the Python test
suite is unaffected.

## Phase III & IV (Cycle 1)

_Not completed for Cycle 1. The burr maintainer (@elijahbenizzy) never responded to
my Phase I intro questions and a prior contributor (@hannguyen0712) had expressed
interest, so I pivoted my build effort to [Cycle 2 (actual #7391)](#cycle-2--actual-7391-mobile-errorboundaries).
I'll reopen burr #276 if the maintainer re-engages._
