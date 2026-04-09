# Order Status Tracker — Context File
**Feature:** Real-Time Order Status and Exception Management
**PM:** [Your Name]
**Last Updated:** April 2026
**Status:** Ready for parallel builds

> **Note:** This is a fictional but realistic example built to demonstrate the context file format.
> The company, product, users, and data are illustrative. Use this as a template — replace
> the content with your own feature, keep the structure.

---

## Problem

Operations teams at Acme Fulfillment Co. currently track order status across three disconnected systems: a legacy ERP, a spreadsheet maintained by the ops lead, and a Slack channel where exceptions get flagged manually. There is no single source of truth for where an order is in the fulfillment lifecycle. When an order goes wrong, the investigation requires pulling data from all three places and cross-referencing timestamps by hand.

This creates resolution delays averaging 4+ hours per exception, increases the risk of SLA violations, and puts the accountability burden on the ops lead rather than the system.

---

## Users

**Primary: Ops Coordinator**
Monitors order queues throughout the shift. Works from a laptop or large tablet. Manages 50–200 active orders at any given time. Currently splits attention across three tabs and a Slack channel. Does not write code. Needs to act on exceptions within minutes, not hours.

**Secondary: Ops Manager**
Reviews exception history and SLA compliance at end of day and end of week. Needs exportable summaries for reporting to leadership. Does not want to dig through raw logs — needs a filtered, readable view.

**Not in scope:** End customers, external partners, finance teams, executive dashboards. These are a future phase.

---

## Jobs to Be Done

When an order changes status, I want the system to reflect it immediately, so I can trust what I'm looking at without manually refreshing or cross-referencing another tool.

When an exception occurs, I want to know about it before the SLA window closes, so I can intervene in time rather than discovering the failure after the fact.

When I need to investigate a past exception, I want to pull the full order history in one place, so I don't have to reconstruct it from Slack threads and spreadsheets.

---

## Definitions

**Order:** A confirmed customer purchase that has entered the fulfillment lifecycle. An order is not a cart, a quote, or a draft. Orders have a unique `order_id` assigned at the point of confirmation.

**Exception:** Any order event that falls outside the normal status progression — including: status regression, stalled orders (no status change within the expected window), orders assigned to an inactive operator, and any manual override of a system-generated status.

**Status window:** The maximum time allowed between status transitions for a given order type before the system treats the order as stalled. Window durations are defined per order type in `/config/sla-windows.json`. Do not hardcode these values.

**Audit log:** An append-only record of every status change for every order. Each entry must include: order ID, previous status, new status, timestamp (UTC), operator ID, and change source (system or manual). The audit log cannot be edited or deleted. It is not the same as the activity feed shown to coordinators.

**HITL checkpoint:** A point in the workflow where the system must pause and route to a human operator before proceeding. The system must not auto-resolve HITL checkpoints or make product decisions on behalf of the operator.

**Manual override:** A status change initiated by a human operator rather than the system. Manual overrides must be flagged in the audit log with the operator's ID and a mandatory reason code.

---

## Order Statuses

Orders exist in exactly one status at any time:

| Status | Meaning |
|---|---|
| `received` | Order confirmed, not yet picked up by fulfillment |
| `in_progress` | Actively being processed |
| `pending_review` | Flagged for human review — cannot advance automatically |
| `shipped` | Dispatched to carrier |
| `delivered` | Confirmed delivered |
| `cancelled` | Permanently closed — no further transitions permitted |

**Valid transitions:**
- `received` → `in_progress`
- `in_progress` → `shipped`
- `in_progress` → `pending_review`
- `pending_review` → `in_progress` (operator approval required)
- `pending_review` → `cancelled` (operator approval required)
- `shipped` → `delivered`
- Any status → `cancelled` requires operator approval and a reason code

**Invalid transitions:** Any transition not listed above must be rejected with a `422` error. The attempt must be logged. The order status must not change.

**Status regression** (e.g. `shipped` → `in_progress`) is always invalid and must never be permitted, even via manual override.

---

## User Journey

### Happy path: coordinator reviews and advances a stalled order

1. Coordinator opens the order queue dashboard
2. System displays orders sorted by urgency — stalled orders and approaching SLA windows appear at the top
3. Coordinator selects a stalled order
4. System displays full order detail: current status, status history, time in current status, SLA window remaining
5. Coordinator selects "Advance to In Progress"
6. System writes audit log entry, transitions order status, and removes order from stalled queue
7. Coordinator returns to queue — updated order no longer appears as stalled
8. Total interaction: under 60 seconds

### Unhappy path: order flagged as exception during processing

1. System detects order has exceeded the SLA window for its current status
2. System transitions order to `pending_review` automatically
3. HITL checkpoint triggered: ops coordinator receives an in-app alert and an email notification
4. Order status does not advance until coordinator reviews and acts
5. Coordinator sees: "This order has exceeded its processing window. Review required before it can advance."
6. Coordinator selects action: advance or cancel
7. If advance: coordinator enters reason code, system logs and transitions to `in_progress`
8. If cancel: coordinator enters reason code, system logs and transitions to `cancelled`

### Unhappy path: manual override attempted on a cancelled order

1. Coordinator attempts to change the status of a cancelled order
2. System rejects the action with an error: "Cancelled orders cannot be modified."
3. System logs the attempt with operator ID and timestamp
4. No status change occurs

---

## Requirements

All requirements are outcome-focused. Implementation patterns are the engineer's decision.

- Order status must update in the UI within 3 seconds of a system or manual status change (P95)
- The audit log must be append-only — no update or delete operations permitted on log entries
- SLA window durations must be read from `/config/sla-windows.json` — not hardcoded
- The dashboard must be usable on a 1280px wide screen at standard zoom — no horizontal scrolling
- Exception alerts must be delivered both in-app and via email — not one or the other
- Manual overrides must require a reason code from the approved taxonomy before the system accepts the transition
- HITL checkpoints must not be auto-resolved by the system under any circumstances
- Invalid status transitions must return a `422` with a reason code — never silently fail
- Operator approval actions must be recorded in the audit log with the operator's user ID

---

## Constraints and Non-Goals

**The system must not:**
- Auto-resolve exceptions or HITL checkpoints
- Permit status regression under any circumstances, including manual override
- Hardcode SLA window values — always read from config
- Modify or delete audit log entries
- Create or modify user accounts or operator permissions
- Send customer-facing notifications (handled by a separate notification service)

**Not in scope for this build:**
- Reporting dashboards or SLA compliance exports (future phase)
- Customer-facing order tracking portal
- Integration with carrier APIs for delivery confirmation (manual entry only for now)
- Bulk status updates
- Mobile app — this is a web UI

**Known technical constraints:**
- Must integrate with the existing identity service for operator session validation — do not build a parallel auth layer
- The notification system for email alerts is already built — use the existing `/api/notifications/ops` endpoint
- Backend is Node.js/Express. Frontend is React. Both live in the monorepo under `/apps`.

---

## Acceptance Criteria

The following must be true before this feature ships:

- [ ] Status update appears in the UI within 3 seconds of a change — verified with a stopwatch across 10 manual tests
- [ ] Audit log captures all required fields on every transition — verified by log inspection after each test scenario
- [ ] Invalid status transitions return a `422` and do not change order status — verified for each invalid transition in the state map
- [ ] HITL checkpoint fires for all stalled order scenarios — coordinator receives both in-app alert and email
- [ ] Manual override requires a reason code — confirmed by attempting a transition without one
- [ ] SLA windows update correctly when `/config/sla-windows.json` values change — no deploy required
- [ ] Audit log entries cannot be modified or deleted — verified by attempting a direct `PUT` and `DELETE` via API
- [ ] Status regression is rejected for all tested scenarios — confirmed that `shipped` → `in_progress` and equivalent regressions return `422`

---

## Open Questions

These must be resolved before the agent starts building, not during:

1. **Session validation:** Does the identity service expose an endpoint to confirm an operator has an active session, or do we read from a shared DB table? — *Owner: engineering lead, needed before sprint start*

2. **Reason code taxonomy:** Where does the approved list of reason codes live, and who owns it? Is it hardcoded, config-driven, or managed in a CMS? — *Owner: ops team, needed before sprint start*

3. **SLA window format:** Are SLA windows defined in minutes or hours in the config file? Are they the same across all order types, or does each type have its own window? — *Owner: ops team, needed before sprint start*

4. **Email notification format:** Does the existing `/api/notifications/ops` endpoint accept a custom message body, or does it use a fixed template? — *Owner: engineering lead, can be resolved in sprint*

5. **Stale order threshold:** At what point does an order appear as "stalled" in the queue — when it hits 50% of its SLA window, 80%, or exactly at the limit? — *Owner: ops manager, needed before sprint start*

---

## How to Use This File

**For engineers running parallel builds:**
This file is your source of truth. Build from it independently — do not coordinate on implementation. That is the point. When the design review happens, we will compare your solutions and pick the best attributes from each.

If something in this file is ambiguous, flag it as an open question rather than making an assumption. Every assumption is a product decision that was not reviewed.

**For the agent (Cursor, Claude Code, or similar):**
Load this file as context at the start of your session. Do not infer requirements that are not listed here. If you encounter a decision point not covered by this document, surface it rather than resolving it autonomously.

**For everyone:**
The Open Questions section is not a placeholder. Those questions must be answered before the build starts. If they are not answered, the build should not start.
