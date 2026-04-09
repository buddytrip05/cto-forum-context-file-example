# Fulfillment Center Asset Inventory — Context File
**Feature:** High-Value Asset Tracking System
**PM:** Buddy Triplett
**Last Updated:** April 8, 2026
**Status:** Ready for parallel builds

---

## Problem

Fulfillment center operators currently track high-value asset movement using a combination of spreadsheets, paper logs, and verbal handoffs. There is no audit trail, no single source of truth, and no way to reconstruct asset history after the fact. When assets go missing or are misrouted, the investigation starts from zero.

This creates compliance risk, slows fulfillment throughput, and erodes ops team confidence in the system.

---

## Users

**Primary: Fulfillment Center Operator**
Responsible for checking assets in and out throughout the shift. Works on a warehouse floor. Uses a tablet or shared terminal. Does not have time for multi-step workflows. Currently uses a paper log or shouts across the floor.

**Secondary: Ops Supervisor**
Reviews asset movement at end of shift or when exceptions occur. Needs a clean audit trail they can export or screenshot. Does not write code. Will use this for compliance reporting.

**Not in scope:** External auditors, finance, executive dashboards. These are a future phase.

---

## Jobs to Be Done

When I check an asset out of inventory, I want to log it in under 10 seconds, so I can stay in flow without creating a paper trail I'll have to reconcile later.

When an asset goes missing, I want to pull up its full state history immediately, so I can identify where the handoff broke down without interviewing the whole shift.

When a high-value asset is flagged as an exception, I want the system to escalate it automatically, so I don't have to decide whether it's my call to make.

---

## Definitions

**High-value asset:** Any asset with a replacement cost of $500 or more per unit, as defined in `/ops/policy/asset-thresholds.json`. Do not hardcode this value. Read it from the policy document.

**Audit log:** A tamper-resistant, append-only record of every state change. Each entry must include: asset ID, previous state, new state, timestamp (UTC), operator user ID, and reason code. The audit log is not the same as the activity feed. The audit log cannot be edited or deleted.

**State change:** Any transition between the four defined asset states. A state change must be logged even if the operator reverses it immediately.

**Exception:** Any event that falls outside normal state transitions — including: asset not found at expected location, asset flagged as damaged, asset checked out to an operator who no longer has active shift status.

**HITL checkpoint:** A decision point where the system must pause and route to the ops supervisor before continuing. The agent or system must not resolve HITL checkpoints autonomously.

---

## Asset States

Assets exist in exactly one of four states at any time:

| State | Meaning |
|---|---|
| `available` | In inventory, ready for use |
| `checked_out` | Assigned to an active operator for the current shift |
| `quarantine` | Flagged for inspection — cannot be checked out |
| `disposed` | Permanently removed from inventory |

**Valid transitions:**
- `available` → `checked_out`
- `checked_out` → `available`
- `available` → `quarantine`
- `quarantine` → `available` (supervisor approval required)
- `quarantine` → `disposed` (supervisor approval required)
- Any state → `disposed` is irreversible

**Invalid transitions:** Any transition not listed above must be rejected with an error, not silently ignored. Log the attempt.

---

## User Journey

### Happy path: operator checks out an asset

1. Operator scans asset QR code or enters asset ID manually
2. System confirms asset is in `available` state and displays asset name, current location, and replacement cost
3. Operator taps "Check Out" — confirms their operator ID (pre-populated from session)
4. System writes audit log entry and transitions asset to `checked_out`
5. Confirmation screen appears for 2 seconds, then returns to home state
6. Total interaction time: under 10 seconds

### Unhappy path: asset is in quarantine

1. Operator scans asset
2. System displays quarantine status and the reason it was flagged
3. System shows "Contact your supervisor — this asset cannot be checked out"
4. No further action is available to the operator
5. System logs the attempted checkout with operator ID and timestamp

### Unhappy path: exception during check-in

1. Operator attempts to check in an asset that is not assigned to them
2. System flags as exception
3. HITL checkpoint triggered: ops supervisor notified via existing shift alert system
4. Asset state does not change until supervisor resolves
5. Operator sees: "This asset is logged to a different operator. Your supervisor has been notified."

---

## Requirements

All requirements are outcome-focused. Implementation patterns are the engineer's decision.

- Every state change must be written to the audit log within 2 seconds of the operator action (P95)
- The audit log must be append-only — no update or delete operations permitted on log entries
- High-value threshold must be read from the ops policy document, not hardcoded
- The UI must work on a tablet in portrait mode at 768px width minimum
- Operators must be able to complete a check-out in under 10 seconds from scan to confirmation
- Exception handling must trigger a HITL checkpoint — the system must not auto-resolve exceptions
- Asset state transitions outside the valid transition map must be rejected and logged
- Supervisor approval for `quarantine` → `available` or `quarantine` → `disposed` must be recorded in the audit log with the supervisor's user ID

---

## Constraints and Non-Goals

**The agent must not:**
- Modify audit log entries under any circumstances
- Auto-resolve HITL checkpoints
- Infer high-value thresholds — always read from the policy document
- Create, delete, or modify user accounts or operator permissions
- Make decisions about exception handling without supervisor input

**Not in scope for this build:**
- Barcode or RFID hardware integration (we're using manual entry and QR scan via device camera)
- Reporting dashboards or data exports (future phase)
- External audit access
- Mobile app — this is a web UI optimized for tablet
- Bulk import or batch operations

**Known constraints:**
- Must integrate with the existing shift management system for operator session validation
- The alert system for supervisor notifications is already built — use the existing `/api/alerts/shift` endpoint
- Backend is Python/FastAPI. Frontend is React. Both are in the monorepo.

---

## Acceptance Criteria

The following must be true before this feature ships:

- [ ] Check-out flow completes in under 10 seconds on a tablet at 768px (manual test, 5 iterations)
- [ ] Audit log captures all required fields on every state change — verified by log inspection
- [ ] Invalid state transitions are rejected with an error response, not silently dropped
- [ ] HITL checkpoint fires on all defined exception types — supervisor receives alert via existing system
- [ ] High-value threshold is read from the policy document — confirmed by changing the value and verifying behavior updates without a code deploy
- [ ] Audit log entries cannot be modified or deleted — verified by attempting a direct update via API
- [ ] All state transitions outside the valid map return a 422 with a reason code from the approved error taxonomy

---

## Open Questions

These must be resolved before the agent starts, not during the build:

1. **Operator session validation:** Does the existing shift management system expose an API endpoint we can call to confirm an operator has an active session? Or do we read from a shared DB table? — *Owner: engineering lead, needed before sprint start*

2. **Supervisor alert timing:** The HITL checkpoint triggers a supervisor alert. What's the expected response window before we escalate further? Is there an escalation path? — *Owner: ops team, needed before sprint start*

3. **Asset ID format:** Are asset IDs always numeric, or can they contain alphanumeric characters? Does the format differ by asset category? — *Owner: ops team, needed before sprint start*

4. **Quarantine reason codes:** Does the quarantine state require a reason code from an approved list, or is it free text? If a list, where does it live? — *Owner: compliance, can be resolved in sprint*

---

## How to Use This File

**For engineers running parallel builds:**
This file is your source of truth. Build from it independently. Do not coordinate on implementation — that's the point. When the design review happens, we'll compare your solutions and pick the best attributes from each.

If something in this file is ambiguous, flag it as an open question rather than making an assumption. Every assumption you make is a product decision that wasn't reviewed.

**For the agent (Cursor, Claude Code, or similar):**
Load this file as your context at the start of your session. Do not infer requirements not listed here. If you encounter a decision point not covered by this document, stop and surface it rather than resolving it autonomously.
