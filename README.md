# CodePath AI301 Contribution README: Basil Thomas

**Cohort:** AI301 Summer 2026, Section 1C (Wednesdays 4 to 6 PM PT)
**Cycle:** 1
**Status:** Phase I Complete

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
_To be filled in during Week 2._

## Phase III: Implementation
_To be filled in during Weeks 3+._

## Phase IV: Pull Request & Iteration
_To be filled in during Weeks 4+._

