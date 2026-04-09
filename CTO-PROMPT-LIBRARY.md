# PM + Engineering AI Prompt Library
**For CTOs, CPOs, and Product Leaders**
*Companion resource to "Product and Engineering Working Together in the Agentic Coding Era" — LA CTO Forum, April 10, 2026*

---

## How to Use This File

Each prompt is designed to help you extract signal from documents, specs, status updates, team conversations, or data — without the AI making things up to fill gaps.

**Before you use any prompt, paste the relevant document, ticket, spec, or data into your conversation first, then use the prompt below it.**

**Universal rules to include in any session:**
> At the start of any Claude, ChatGPT, or Cursor session, paste this:
> *"Do not infer, assume, or invent information that is not explicitly present in what I share with you. If something is unclear or missing, flag it as an open question rather than filling the gap. I would rather have an honest unknown than a confident wrong answer."*

---

## TABLE OF CONTENTS

1. [Spec Audit — Find Your Context Debt](#1-spec-audit--find-your-context-debt)
2. [Decision Surface Scan — What is the Agent Deciding For You?](#2-decision-surface-scan--what-is-the-agent-deciding-for-you)
3. [PRD to Context File Conversion](#3-prd-to-context-file-conversion)
4. [Status Update Digest — Cut Through the Noise](#4-status-update-digest--cut-through-the-noise)
5. [Delivery Risk Scan — What Should You Be Worried About?](#5-delivery-risk-scan--what-should-you-be-worried-about)
6. [Acceptance Criteria Generator](#6-acceptance-criteria-generator)
7. [Human-AI Decision Boundary Mapper](#7-human-ai-decision-boundary-mapper)
8. [Parallel Build Readiness Check](#8-parallel-build-readiness-check)
9. [Feature Bloat Detector](#9-feature-bloat-detector)
10. [Retrospective Debrief — What Would the Context File Have Prevented?](#10-retrospective-debrief--what-would-the-context-file-have-prevented)

---

## 1. Spec Audit — Find Your Context Debt

**Goal:** Identify every place in a spec, ticket, or PRD where an engineer or AI agent would have to make an assumption because the document doesn't answer the question. Your "context debt" is the count of those gaps.

**How to use:** Paste your spec, Jira ticket, Confluence page, or PRD into the conversation, then send the prompt.

---

### Prompt

```
Review the spec I've shared and do the following:

1. List every decision an engineer or AI coding agent would have to make that this document does not answer. Be specific — quote the section that creates the ambiguity and state exactly what decision it leaves open.

2. Count the total number of unanswered decisions. That number is my context debt score.

3. Rank the top 5 gaps by risk — which ones would most likely result in the wrong thing being built if left unresolved?

4. For each of the top 5, write one question I should answer before this goes to engineering.

Rules:
- Do not suggest answers to the gaps — only surface them.
- Do not assume any standard industry practice fills a gap unless it is explicitly stated in the document.
- If the document is too vague to even identify specific gaps, say so directly.
```

**What to look for in the response:**
The gaps that surprise you are the important ones. If the agent surfaces 10+ decision points on a spec you thought was complete, that's your baseline. A well-formed context file should produce 0–2 genuine gaps.

**Follow-up prompt if you want to go deeper:**
```
Take the top 3 gaps you identified and write the exact question I should ask my ops team, engineering lead, or stakeholder to resolve each one. Format as: Gap → Owner → Question to ask.
```

**Follow-up prompt to pressure-test the agent:**
```
Now check your own analysis. Are there any gaps you may have missed because they are easy to overlook — like error states, edge cases, or rollback behavior? Add any additional gaps you find.
```

---

## 2. Decision Surface Scan — What is the Agent Deciding For You?

**Goal:** When an AI coding agent runs against a spec, it makes product decisions wherever the spec is silent. This prompt makes those decisions visible before the build starts — not after.

**How to use:** Paste your spec or ticket, then send the prompt.

---

### Prompt

```
I am about to run an AI coding agent against the spec I've shared. Before I do, I need to understand what product decisions the agent will make on my behalf because my spec doesn't answer them.

1. List every place where the agent will have to make a product decision because the spec is silent or ambiguous. Examples include: what happens on error, what the fallback behavior is, what counts as a valid input, what a vague term like "fast" or "user-friendly" actually means in code.

2. For each decision, write what the agent will most likely assume and why.

3. Flag which of these decisions have user-facing consequences — things a real customer would experience differently depending on which way the agent decides.

Rules:
- Do not make the decisions for me. Surface them so I can make them.
- Do not assume best practices resolve the ambiguity. Flag it anyway.
- If a decision is genuinely low-stakes and any reasonable implementation would work, you may note that — but still list it.
```

**What to look for in the response:**
Any decision with a user-facing consequence is one you should resolve in the spec before the agent runs. The goal is zero silent product decisions.

**Follow-up prompt:**
```
For each user-facing decision you identified, write one acceptance criterion I can add to the spec to resolve it. Format as a testable statement: "Given [condition], when [action], then [outcome]."
```

---

## 3. PRD to Context File Conversion

**Goal:** Transform an existing PRD, Confluence page, or set of requirements into a structured context file that both engineers and AI agents can execute from without interpretation.

**How to use:** Paste your existing PRD or requirements document, then send the prompt.

---

### Prompt

```
Convert the document I've shared into a structured context file using the following seven sections. For each section, use only information that is explicitly present in the document. If a section cannot be filled because the information isn't there, write "[MISSING — needs input from PM/stakeholder]" rather than inventing content.

Sections:

1. Problem — 2–3 sentences. What is broken today and for whom?
2. Users — Who is affected? What do they do instead of using this feature today?
3. Jobs to Be Done — For each user type, one sentence in the format: "When [situation], I want to [motivation], so I can [outcome]."
4. Definitions — Any term in this document that an engineer or AI agent could misinterpret. Define it precisely.
5. User Journey — Step-by-step: happy path, then unhappy path(s). Flag any step that is not described in the source document.
6. Requirements — Outcome-focused statements only. Remove any requirement that describes implementation rather than outcome.
7. Constraints and Non-Goals — What must the agent NOT do or decide? What is explicitly out of scope?

After completing all seven sections, list any information that was missing from the source document that I will need to resolve before this context file is ready for engineering.

Rules:
- Do not invent or infer anything not stated in the source.
- Do not paraphrase in ways that change meaning.
- Flag every gap explicitly. Do not hide incomplete sections.
```

**What to look for in the response:**
The `[MISSING]` markers are your action list. Every one of those is a conversation you need to have before the build starts. Count them — that's your PRD quality score.

**Follow-up prompt:**
```
For each [MISSING] section, write the specific question I need to answer to complete it, and identify who in my organization is most likely the right person to answer it.
```

---

## 4. Status Update Digest — Cut Through the Noise

**Goal:** Transform a lengthy status update, Slack thread, standup summary, or project report into a structured executive digest that surfaces only what requires a decision or action.

**How to use:** Paste the status update, thread, or report, then send the prompt.

---

### Prompt

```
Read the status update I've shared and produce a structured digest with four sections:

1. What is on track — summarize in 2–3 sentences. No detail needed.
2. What is at risk — list each risk with: what it is, why it's a risk, and what decision or action is needed to resolve it. Include a suggested owner.
3. What is blocked — list each blocker, who owns it, and how long it has been unresolved if that information is present.
4. What requires my attention — list only items that require a decision, approval, or escalation from leadership. Be specific about what the decision is.

Rules:
- Do not summarize things that are progressing normally unless they are relevant to a risk or blocker.
- Do not recommend a course of action unless asked. Surface the situation; I will decide.
- If something is unclear from the update, say so — do not interpret charitably or assume it is fine.
- If a timeline or milestone is mentioned without context, flag it rather than accepting it at face value.
```

**What to look for in the response:**
Section 4 is the most important. If it's empty, your status update is genuinely healthy. If it has more than 2–3 items, you have a prioritization conversation to have.

**Follow-up prompt:**
```
For each item in "What requires my attention," write the specific question I should bring to the team to resolve it — including what information I would need to make a good decision.
```

**Follow-up prompt if the update is vague:**
```
This update uses vague language in several places. Identify every statement that uses imprecise terms like "almost done," "on track," "soon," "minor issue," or "working on it" — and flag them as unverified claims that need a concrete date, metric, or owner attached before I can act on them.
```

---

## 5. Delivery Risk Scan — What Should You Be Worried About?

**Goal:** Identify the real delivery risks in a project, sprint plan, or roadmap — including risks the document doesn't name but that are implied by what's missing or vague.

**How to use:** Paste your sprint plan, roadmap, or project brief, then send the prompt.

---

### Prompt

```
Analyze the project plan or roadmap I've shared and identify delivery risks across four categories:

1. Scope risks — things that are likely to expand, get contested, or be interpreted differently by different people
2. Dependency risks — things this project relies on that aren't controlled by the team, including external teams, APIs, data, or decisions not yet made
3. Timeline risks — milestones that look optimistic, have no buffer, or depend on assumptions that aren't stated
4. Context risks — things that are missing from this document that an engineer or agent would have to invent, which could cause the wrong thing to be built

For each risk, write: what the risk is, why it's a risk, and what would need to be true for it not to be a risk.

Rules:
- Do not sugarcoat risks to seem balanced.
- Do not assume a risk is mitigated just because it isn't mentioned.
- If you don't have enough information to assess a risk category, say so rather than speculating.
- Do not rank risks unless I ask you to — list them all.
```

**What to look for in the response:**
Context risks (category 4) are the ones most teams miss. If the agent surfaces several, you have a spec quality problem, not a planning problem.

**Follow-up prompt:**
```
Rank the risks you identified by likelihood of impacting the delivery date. For the top 3, write one mitigation action the team could take this week.
```

---

## 6. Acceptance Criteria Generator

**Goal:** Convert a feature description, user story, or requirement into machine-testable acceptance criteria that an agent can execute against — not prose that an engineer has to interpret.

**How to use:** Paste the feature description or user story, then send the prompt.

---

### Prompt

```
Convert the feature description I've shared into acceptance criteria that are specific enough for an AI coding agent to test against without interpretation.

For each acceptance criterion:
- Write it in "Given [condition] / When [action] / Then [outcome]" format
- Include the specific value, threshold, or behavior — not vague descriptors like "fast" or "user-friendly"
- Write a separate criterion for the happy path and each unhappy path you can identify from the description

After writing the criteria, do two things:
1. Flag any criterion where you had to make an assumption because the description didn't specify the expected behavior. Mark these clearly as [ASSUMPTION — needs PM confirmation].
2. List any scenarios that are not covered by the description but that a real user would likely encounter — these are gaps I should address before the build starts.

Rules:
- Do not invent specific values (e.g., timeout durations, error codes, thresholds) unless they are stated in the description. Use [VALUE NEEDED] as a placeholder.
- Do not assume error handling behavior. Flag every error state as needing explicit specification.
- A criterion like "the system should be fast" is not acceptable — push it to a specific measurable outcome or flag it as incomplete.
```

**What to look for in the response:**
Every `[ASSUMPTION]` and `[VALUE NEEDED]` marker is a conversation you need to have. These are the silent product decisions your agent would otherwise make for you.

**Follow-up prompt:**
```
Take the [ASSUMPTION] items and write the question I need to answer for each one, along with a suggested default behavior I could accept or reject.
```

---

## 7. Human-AI Decision Boundary Mapper

**Goal:** Before an AI agent starts working on a feature, explicitly define which decisions the agent can make autonomously and which ones require a human. This is one of the highest-leverage things a team can do before a sprint starts.

**How to use:** Paste your context file, spec, or feature brief, then send the prompt.

---

### Prompt

```
Review the feature description I've shared and map the human-AI decision boundary for this build.

Produce two lists:

1. Decisions the agent can make autonomously — things where any reasonable implementation choice is acceptable and the outcome doesn't require product judgment. These are safe to delegate.

2. Decisions that require a human — things where the choice has user-facing consequences, compliance implications, or depends on business context the agent cannot know. These are not safe to delegate.

For each item in list 2, write:
- What the decision is
- Why it requires a human
- Who the right human is to make it (PM, engineering lead, legal, ops, etc.)
- Whether it needs to be resolved before the agent starts or can be resolved during the build

Rules:
- If you are uncertain whether a decision is safe to delegate, put it in list 2 by default. When in doubt, escalate.
- Do not assume business context. If a decision requires knowledge of the business that isn't in the document, flag it explicitly.
- Do not suggest that "best practice" resolves a decision that requires product judgment.
```

**What to look for in the response:**
List 2 is your pre-sprint checklist. Every item on it should be resolved — or explicitly assigned to someone — before the agent starts. If list 2 has more than 5–6 items, your spec isn't ready.

**Follow-up prompt:**
```
For each item in list 2 that needs to be resolved before the agent starts, write it as a standing rule I can add to my context file under "Constraints and Non-Goals" — so the agent knows it must not make this decision autonomously.
```

---

## 8. Parallel Build Readiness Check

**Goal:** Determine whether a spec is precise enough to support parallel development — where multiple engineers each build an independent version of the same feature from the same document without coordinating.

**How to use:** Paste your spec or context file, then send the prompt.

---

### Prompt

```
I want to run a parallel build: two or more engineers will each independently build a complete version of this feature from the spec I've shared, without coordinating with each other. I'll then review all versions and pick the best one.

This only works if the spec is precise enough that each engineer builds the same thing — not two different interpretations of a vague document.

Review the spec and tell me:

1. Is this spec ready for a parallel build? Answer yes, no, or not yet.

2. If not yet: list every section or statement that is ambiguous enough that two engineers would likely implement it differently. For each one, explain what the two most likely different implementations would be.

3. If yes: list the 3 areas where implementations are still most likely to diverge, so I know what to compare during the design review.

4. Write three specific questions I should ask during the design review to evaluate which parallel build is the strongest.

Rules:
- Do not assume engineers would make the same reasonable default choices. Flag every place where a reasonable developer could go a different direction.
- Do not say the spec is ready if there are open questions that haven't been resolved.
```

**What to look for in the response:**
If the answer is "not yet," fix the spec before you run the build — not after. A parallel build on a vague spec produces five different interpretations, not five creative solutions.

**Follow-up prompt:**
```
Take the ambiguities you identified and write the minimum additions to the spec that would resolve them — without over-specifying the implementation. I want to give engineers creative latitude on the how, while being precise about the what and why.
```

---

## 9. Feature Bloat Detector

**Goal:** In the agentic era, when the cost to build drops dramatically, teams tend to build too much. This prompt stress-tests a feature or roadmap against the original problem statement to identify scope that doesn't belong.

**How to use:** Paste your feature list, sprint scope, or roadmap alongside the original problem statement, then send the prompt.

---

### Prompt

```
I'm going to share two things: a problem statement and a feature list (or sprint scope or roadmap). Your job is to identify feature bloat — things we're building that don't directly solve the stated problem.

Problem statement: [paste here]

Feature list / scope: [paste here]

Do the following:

1. For each feature or scope item, write one sentence explaining how it directly solves the stated problem. If you can't write that sentence without stretching, flag the item as potential bloat.

2. Identify any features that solve a different problem than the one stated — a problem that may be real but wasn't the one we set out to solve.

3. Identify any features that are solving for a future user or use case that isn't described in the problem statement.

4. Recommend which items could be cut or deferred without compromising the core solution.

Rules:
- Do not recommend keeping something just because it sounds useful.
- Do not assume a feature is justified because it's already been built or announced.
- Be direct. If something doesn't belong in this release, say so.
```

**What to look for in the response:**
Any feature that survives only because it "sounds good" or "we had the capacity" is a candidate for the cut list. Tighter scope means faster validation of the core problem.

**Follow-up prompt:**
```
For the items you flagged as potential bloat, write a one-sentence argument for why each one should be deferred — framed in terms of what we learn faster by not building it yet.
```

---

## 10. Retrospective Debrief — What Would the Context File Have Prevented?

**Goal:** After a feature ships, use this prompt to learn from what went wrong — specifically by identifying which issues were caused by missing or vague context at the start of the build.

**How to use:** Paste your original spec alongside a description of what went wrong (bug reports, rework notes, missed requirements, engineer feedback), then send the prompt.

---

### Prompt

```
I'm going to share two things: the original spec for a feature we shipped, and a summary of issues we encountered during or after the build — including rework, missed requirements, bugs, or misaligned implementations.

Original spec: [paste here]

Issues encountered: [paste here]

Do the following:

1. For each issue, identify whether it was caused by:
   a) Missing context — something that wasn't in the spec that an engineer or agent had to guess
   b) Ambiguous context — something that was in the spec but could be interpreted more than one way
   c) Correct context, incorrect execution — the spec was clear but it wasn't followed
   d) External factor — something outside the spec that caused the issue

2. For each issue in category (a) or (b), write the specific addition to the original spec that would have prevented it.

3. Produce a "context file gap list" — the five things missing from this spec that I should make standard in every context file going forward.

Rules:
- Do not assign blame to individuals.
- Do not speculate about causes that aren't supported by the issues I've described.
- If you can't determine the root cause from the information provided, say "more information needed" rather than guessing.
```

**What to look for in the response:**
The gap list at the end is your institutional learning. After two or three of these retrospectives, you'll start to see patterns — the same categories of missing context causing the same categories of problems. That's what you put in your standard context file template.

**Follow-up prompt:**
```
Based on the gap list you produced, write a context file template section I can add to our standard PRD process — so these gaps are caught before every build, not discovered after.
```

---

## Quick Reference: Universal Follow-Up Prompts

Use these at any point in any session when you want the agent to check its own work.

**To surface hidden assumptions:**
```
Review your last response and list every assumption you made that wasn't explicitly stated in the document I shared. For each one, tell me what you assumed and what the alternative interpretation could have been.
```

**To check for overconfidence:**
```
In your last response, which statements are you most uncertain about? What additional information would change your answer?
```

**To find what's missing:**
```
What information would you need that I haven't provided to give you a more complete or accurate answer?
```

**To pressure-test a decision:**
```
Argue the other side. What's the strongest case against the recommendation you just made?
```

**To tighten a spec:**
```
Read this spec as if you were an AI coding agent with no context about our business. What would you build? Where would you get stuck? What would you invent?
```

---

*Built for the LA CTO Forum — April 10, 2026*
*Companion to: "Product and Engineering Working Together in the Agentic Coding Era"*
*Speaker: Buddy Triplett, Principal PM, SpecBooks*
