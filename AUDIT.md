# Audit Policy

**System:** Balance Sheet Reconciliation Agent  
**Applies to:** Account Reconciliation · Exception Management · Rec Package  
**Effective:** 2026-05-30

---

## Purpose

This document defines what the Balance Sheet Reconciliation Agent system logs, how those logs are protected, who can access them, and how they are used to support internal review, external audit, and SOX compliance for Lumina Streaming Co.

---

## What Is Logged

Every agent action and human approval produces an immutable audit record. The following events are always captured:

### Agent Events

| Event | Logged Fields |
|---|---|
| Run initiated | Agent name, run ID, close period, entity, timestamp |
| Source file ingested | File name, record count, checksum, timestamp |
| Input validation result | Pass / fail, specific failure detail if applicable |
| Account reconciliation completed | Account, GL balance, support balance, variance, status |
| Reconciling item identified | Item ID, account, amount, description, aging bucket, timestamp |
| Exception escalated | Item ID, severity level, escalation reason, notified approver, timestamp |
| Journal entry proposed | JE ID, account, debit/credit, amount, supporting reference, timestamp |
| Output file written | File name, file type, destination path, checksum, timestamp |
| Run completed | Run ID, accounts processed, exceptions raised, duration, timestamp |
| Run aborted | Run ID, abort reason, last successful step, timestamp |

### Human Events

| Event | Logged Fields |
|---|---|
| Approval granted | Approval type, approver name, role, item ID or JE ID, timestamp |
| Approval rejected | Approval type, approver name, role, item ID or JE ID, rejection notes, timestamp |
| Item returned for revision | Item ID, approver name, revision notes, timestamp |
| Workpaper signed off | Workpaper ID, reviewer name, sign-off date |
| Rec package released | Package ID, Controller name, CFO name, release timestamp |
| Threshold override applied | Field changed, old value, new value, approver name, timestamp |
| User login | User, role, timestamp, success / failure |

---

## Log Format

Each audit record is written as a structured entry with the following standard fields:

```
{
  "event_id":    "<uuid>",
  "event_type":  "<category.action>",
  "actor":       "<agent_name | human_name>",
  "actor_type":  "agent | human",
  "run_id":      "<uuid>",
  "entity":      "LuminaUS | LuminaEMEA | LuminaAPAC",
  "close_period":"YYYY-MM",
  "timestamp":   "<ISO 8601>",
  "payload":     { <event-specific fields> },
  "checksum":    "<sha256 of record content>"
}
```

---

## Log Integrity

- Audit records are **append-only** — no record may be modified or deleted after it is written.
- Each record includes a SHA-256 checksum of its content; any tampering is detectable on verification.
- Log integrity is verified automatically at the start of each close cycle run.
- A failed integrity check aborts the run and notifies the System Administrator immediately.

---

## Retention

| Log type | Retention period | Location |
|---|---|---|
| Agent run logs | 7 years | Audit trail store |
| Human approval records | 7 years | Audit trail store |
| User login events | 3 years | Access log store |
| Input validation failures | 7 years | Audit trail store |
| Incident records | 7 years | Audit trail store |

Retention periods meet SOX Section 802 documentation requirements. Purges are logged as audit events before execution.

---

## Access

| Role | Access level |
|---|---|
| Controller | Read — all agent and approval events for their close period |
| CFO | Read — all events; required for rec package release and threshold override records |
| Internal Auditor | Read — all events within the audit engagement scope |
| External Auditor | Read — provided by Controller for the engagement period only |
| System Administrator | Read/Write — log storage management, integrity verification, purge execution |
| Agent | Write — append new records only; cannot read or modify existing records |

No role may delete audit records. Purges at end of retention period are executed by the System Administrator and are themselves logged.

---

## SOX Alignment

The audit trail is designed to satisfy the following SOX requirements:

| SOX Control | How It Is Met |
|---|---|
| Evidence of review and approval (ITGC) | Every JE and reconciling item records the approver, role, and timestamp |
| Segregation of duties | Agent (preparer) and human (reviewer/approver) are distinct actors in every record |
| Change management | All config changes (including threshold overrides) are logged with approver identity |
| Access controls | All login events and role assignments are logged |
| Data integrity | Append-only log with per-record checksums; integrity verified each cycle |
| Retention | 7-year retention for all financial records per SOX Section 802 |

---

## Audit Support Package

At the request of the Controller or external auditors, the system can produce an **Audit Support Package** for a specified close period containing:

- Full agent run log for the period
- All workpapers with preparer, reviewer, and sign-off date
- Complete approval record (JEs, item clearances, rec package release)
- Exception log with aging history and resolution trail
- Threshold configuration at time of close (point-in-time snapshot)

The package is assembled by the Rec Package agent and released by the Controller. It is never distributed directly by the agent.

---

## Policy Review

This policy is reviewed annually and after any audit finding, security incident, or material change to the agent system.

| Role | Responsibility |
|---|---|
| System Administrator | Log storage, integrity checks, retention purges |
| Controller | Audit trail oversight, audit support package requests |
| CFO | Policy owner, SOX sign-off, external auditor coordination |

---

*Read alongside [GUARDRAILS.md](GUARDRAILS.md), [security.md](security.md), and [docs/human-in-the-loop-policy.md](docs/human-in-the-loop-policy.md).*
