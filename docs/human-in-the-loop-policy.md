# Human-in-the-Loop Policy

**System:** Balance Sheet Reconciliation Agent  
**Applies to:** Account Reconciliation · Exception Management · Rec Package  
**Effective:** 2026-05-30

---

## Core Principle

> **Agents prepare. Humans approve.**

No journal entry is posted, no reconciling item is closed, and no rec package is released without an explicit human sign-off. The agents handle the analytical heavy lifting; the Controller, CFO, or designated reviewer retains full decision authority.

---

## Required Approvals

| Action | Approver | Required Before |
|---|---|---|
| Post or reverse a journal entry | Controller | GL booking |
| Close / clear a reconciling item | Controller | Item removed from exception log |
| Escalate an exception to the CFO | Controller | CFO notification sent |
| Release the monthly rec package | Controller + CFO (sign-off matrix) | Distribution to audit / management |
| Override a materiality threshold | CFO | Exception logged in audit trail |

---

## When the Agent Pauses and Waits

The agent halts and requests human review when:

1. **Auto-investigate trigger** — a reconciling item exceeds **$250,000** regardless of age.
2. **Escalation trigger** — an item exceeds **$100,000 or is 30+ days old** (either condition).
3. **Critical aging** — any open item reaches the **90-day** bucket.
4. **Workpaper discrepancy** — the GL balance cannot be fully tied to sub-ledger support.
5. **Missing support** — required backup (bank statement, lender confirm, IC confirmation) is absent at rec time.
6. **Threshold override requested** — any agent output would require changing a value in `config/thresholds.yaml`.

The agent logs the pause reason, timestamps it, and notifies the reviewer before stopping.

---

## Workpaper Sign-Off Requirements

Every workpaper must carry three fields before it is considered complete:

| Field | Populated by |
|---|---|
| **Preparer** | Agent (auto-stamped) |
| **Reviewer** | Human (manual entry) |
| **Sign-off date** | Human (manual entry) |

Workpapers with blank Reviewer or Sign-off date fields are treated as **Draft** and excluded from the rec package.

---

## Audit Trail

Every agent output envelope includes:

- Data sources and pull timestamps
- Agent version and run ID
- List of human approvals obtained (approver, date, action)
- Any thresholds that were overridden and by whom

This trail is immutable once the rec package is released.

---

## Out-of-Scope (Agent Never Does These)

- Post, reverse, or approve journal entries autonomously.
- Mark a reconciling item resolved without human confirmation.
- Send the rec package to external parties (auditors, lenders, board).
- Modify `config/thresholds.yaml` at runtime.
- Access restricted workpaper sub-ledger detail outside the reconciliation workflow.

---

*Thresholds referenced here are defined in `config/thresholds.yaml` and supersede this document if values differ.*
