# CodePath AI301 Contribution README: Basil Thomas

**Cohort:** AI301 Summer 2026, Section 1C (Wednesdays 4 to 6 PM PT)
**Cycle:** 1
**Status:** Phase II Complete

---

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

## Phase III: Implementation
_To be filled in during Weeks 3+._

## Phase IV: Pull Request & Iteration
_To be filled in during Weeks 4+._
