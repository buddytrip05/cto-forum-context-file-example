# The PM-Engineering Context File
### Companion repo — LA CTO Forum, April 10, 2026
*"Product and Engineering Working Together in the Era of Agentic Coding"*

---

## What this repo is

When engineering teams use agentic coding tools — Cursor, Claude Code, GitHub Copilot, and others — the quality of what gets built is determined by the quality of what goes in. Every gap in a spec is a product decision the agent makes on your behalf, silently, without asking.

This repo contains three artifacts that represent the new PM-engineering operating model: a real context file, a prompt library for leaders, and a guide to structuring your codebase so AI tools have the context they need without you having to paste it into every session.

They are designed to be read, forked, and adapted. Nothing here is theoretical.

---

## What's in this repo

### `CONTEXT.md` — A real context file

This is a complete example of the artifact that replaces the PRD in the agentic development model. It was built based on a real feature: an inventory system for a fulfillment center.

It is not a requirements document. It is not a user story. It is a machine-executable brief — structured so that multiple engineers can each build an independent version of the same feature from this single document, without coordinating, and without inventing answers to fill gaps.

**What makes it different from a PRD:**
- Definitions section that eliminates ambiguous terms before the build starts
- Explicit asset state machine with valid and invalid transitions
- User journey written step-by-step for both happy and unhappy paths
- Acceptance criteria written as testable outcomes, not prose descriptions
- Constraints and Non-Goals section that tells the agent what it must not decide autonomously
- Open Questions section that surfaces what is unknown rather than hiding it

**How to use it:**
Paste it into Claude Code, Cursor, or any LLM at the start of a session. The agent will have what it needs to build from without asking for clarification or inventing answers.

**How to adapt it:**
Replace the inventory system content with your own feature. Keep the structure. The seven sections — Problem, Users, Jobs to Be Done, Definitions, User Journey, Requirements, and Constraints and Non-Goals — apply to any feature.

---

### `CTO-PROMPT-LIBRARY.md` — 10 prompts for product and engineering leaders

A library of prompts designed for CTOs, CPOs, and heads of engineering who want to use AI to get better signal from their existing documents — specs, status updates, roadmaps, retrospectives — without the AI inventing information to fill gaps.

Each prompt includes the goal, instructions for use, what to look for in the response, and follow-up prompts for going deeper or pressure-testing the agent's own assumptions.

**The 10 prompts:**

| # | Prompt | What it does |
|---|---|---|
| 1 | Spec Audit | Counts how many decisions your spec leaves unanswered — your context debt score |
| 2 | Decision Surface Scan | Shows what the agent will decide for you before you start the build |
| 3 | PRD to Context File Conversion | Transforms existing docs into a structured context file |
| 4 | Status Update Digest | Cuts noise from updates and surfaces only what requires a leadership decision |
| 5 | Delivery Risk Scan | Identifies scope, dependency, timeline, and context risks in a project plan |
| 6 | Acceptance Criteria Generator | Produces machine-testable criteria with explicit gap markers |
| 7 | Human-AI Decision Boundary Mapper | Pre-sprint checklist: what the agent can decide vs. what requires a human |
| 8 | Parallel Build Readiness Check | Determines if a spec is tight enough to support independent parallel builds |
| 9 | Feature Bloat Detector | Stress-tests scope against the original problem statement |
| 10 | Retrospective Debrief | Learns from what went wrong and builds it into your standard process |

**Universal rule to include in every session:**
> *"Do not infer, assume, or invent information that is not explicitly present in what I share with you. If something is unclear or missing, flag it as an open question rather than filling the gap."*

---

### `repo_context_starter.md` — How to structure your repo for AI tools

A practical guide to keeping persistent project context in the repo itself — so engineering teams don't have to paste the same background into every Claude Code or Cursor session.

Covers the official Claude Code `CLAUDE.md` pattern, how to layer context across the repo hierarchy, how to use path-scoped rules for different subsystems, and what is confirmed vs. what is still unknown about how these tools work.

Includes a beginner version (one root file, one docs folder) and an advanced version (layered context, scoped rules, directory-specific instructions).

---

## The core idea behind all three files

The old PM-engineering contract assumed engineers would fill gaps through interpretation, tribal knowledge, and senior judgment. That contract worked when engineering was the bottleneck.

It doesn't work when agents are doing the building.

Agents don't fill gaps. They invent answers — confidently, silently, and without asking. Every ambiguous requirement becomes a product decision made by a model, not your team.

The three files in this repo represent the new contract:

- The **context file** is what the PM delivers instead of a PRD
- The **prompt library** gives leaders tools to find gaps before the build starts
- The **repo context guide** gives engineering teams a way to make the codebase itself a source of persistent, reliable context

---

## How to use this repo

**If you are a CTO or head of engineering:**
Start with `CTO-PROMPT-LIBRARY.md`. Pick one prompt, paste in a recent spec or status update, and run it. The gaps it surfaces are your baseline.

**If you are a PM or product leader:**
Start with `CONTEXT.md`. Read through the structure. Then ask: which of these sections am I currently leaving out of my specs? That's your improvement list.

**If you are setting up a team for agentic development:**
Start with `repo_context_starter.md`. Use the beginner version first. Get one `CLAUDE.md` in your repo root. Expand from there.

---

## About this repo

Built by **Buddy Triplett**, Principal PM at SpecBooks, as a companion resource to the LA CTO Forum virtual session on April 10, 2026.

The session: *Product and Engineering Working Together in the Era of Agentic Coding* — a practical conversation with product and engineering leaders on how the PM-engineering interface changes when agents are doing the building.

Questions or feedback: connect on [LinkedIn](https://www.linkedin.com/in/buddytriplett) or open an issue.
