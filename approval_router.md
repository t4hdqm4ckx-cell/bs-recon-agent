# Approval Router

**System:** Balance Sheet Reconciliation Agent  
**Applies to:** Account Reconciliation · Exception Management · Rec Package  
**Effective:** 2026-05-30

---

## Purpose

The Approval Router defines how agent-generated outputs, exceptions, and escalations are directed to the correct human reviewer — and what happens if a response is not received within the required window.

---

## Routing Logic

Every agent action that requires human approval is classified by **type** and **severity**, which together determine the approver, channel, and deadline.

### Approval Types

| Type | Description | Triggered By |
|---|---|---|
| `JE_APPROVAL` | Proposed journal entry awaiting Controller sign-off | Account Reconciliation agent |
| `ITEM_CLEAR` | Reconciling item proposed for closure | Exception Management agent |
| `ESCALATION` | Exception exceeds materiality or aging threshold | Exception Management agent |
| `PACKAGE_RELEASE` | Monthly rec package ready for CFO sign-off | Rec Package agent |
| `THRESHOLD_OVERRIDE` | Request to change a value in `config/thresholds.yaml` | Any agent or administrator |

---

### Severity Levels

| Level | Criteria |
|---|---|
| `LOW` | Item < $100,000 and < 30 days old |
| `MEDIUM` | Item $100,000–$250,000 or 30–60 days old |
| `HIGH` | Item > $250,000 or 61–90 days old |
| `CRITICAL` | Item 90+ days old or aggregate trivial > $450,000 |

---

### Routing Matrix

| Type | Severity | Primary Approver | Escalation Approver | SLA |
|---|---|---|---|---|
| `JE_APPROVAL` | LOW | Controller | — | 24 hours |
| `JE_APPROVAL` | MEDIUM | Controller | — | 12 hours |
| `JE_APPROVAL` | HIGH | Controller | CFO (notify only) | 4 hours |
| `ITEM_CLEAR` | LOW | Controller | — | 48 hours |
| `ITEM_CLEAR` | MEDIUM | Controller | — | 24 hours |
| `ITEM_CLEAR` | HIGH | Controller | CFO | 12 hours |
| `ESCALATION` | HIGH | Controller + CFO | — | 4 hours |
| `ESCALATION` | CRITICAL | CFO | External Auditor (notify) | 2 hours |
| `PACKAGE_RELEASE` | Any | Controller + CFO | — | 24 hours |
| `THRESHOLD_OVERRIDE` | Any | CFO | — | 48 hours |

---

## SLA Breach Behavior

If the primary approver does not respond within the SLA window:

1. **Hour 0** — Approval request sent to primary approver.
2. **50% of SLA elapsed** — Reminder sent to primary approver.
3. **SLA breach** — Escalation approver is notified; item severity is promoted one level.
4. **2× SLA elapsed** — Workflow is flagged as blocked; close calendar impact is logged.

The agent does **not** auto-approve or auto-close any item due to SLA breach. The workflow remains paused until human action is taken.

---

## Notification Channels

| Approver | Primary Channel | Fallback Channel |
|---|---|---|
| Controller | Email | Direct message |
| CFO | Email | Direct message |
| External Auditor | Email | — |
| System Administrator | Audit trail alert | Email |

Notification templates are defined in `config/notifications/`. The agent populates templates with item details, amounts, aging, and a direct link to the relevant workpaper.

---

## Approval Record

Every completed approval is written to the audit trail immediately and includes:

| Field | Value |
|---|---|
| `approval_type` | One of the types above |
| `severity` | Level at time of approval |
| `approver` | Name and role |
| `approved_at` | ISO 8601 timestamp |
| `action_taken` | `approved` / `rejected` / `returned_for_revision` |
| `notes` | Free-text field (optional) |

Rejected or returned items re-enter the agent queue with the approver's notes attached.

---

## Rejection & Revision Flow

If an approver rejects or returns an item:

```
Approver rejects / returns item
    → Agent receives rejection with notes
    → Item re-queued for agent revision
    → Agent revises output and resubmits for approval
    → Revision count logged in audit trail
    → If revision count > 3: item escalated to CFO for manual resolution
```

---

*Read alongside [GUARDRAILS.md](GUARDRAILS.md) and [docs/human-in-the-loop-policy.md](docs/human-in-the-loop-policy.md).*
