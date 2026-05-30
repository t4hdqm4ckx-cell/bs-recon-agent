# Privacy Policy

**System:** Balance Sheet Reconciliation Agent  
**Applies to:** Account Reconciliation · Exception Management · Rec Package  
**Effective:** 2026-05-30

---

## Scope

This policy governs how the Balance Sheet Reconciliation Agent system collects, processes, stores, and protects financial and personal data for Lumina Streaming Co. (LuminaUS, LuminaEMEA, LuminaAPAC) during month-end close operations.

---

## Data Handled

| Data Type | Examples | Classification |
|---|---|---|
| General ledger balances | Trial balance, account totals | Confidential |
| Sub-ledger detail | AR aging, AP invoices, payroll accruals | Restricted |
| Bank & lender statements | Operating account, money market, debt schedules | Restricted |
| Intercompany transactions | IC matrix, elimination entries | Confidential |
| Journal entries | Proposed and posted JEs | Confidential |
| Reconciling items | Exception log, aging buckets | Confidential |
| Reviewer identity | Approver name, sign-off date | Internal |

**Restricted** data is never included in executive-level outputs or the CFO sign-off matrix — only account-level summaries appear in those documents.

---

## Data Collection

- Data is ingested only from authorized source systems: ERP/GL export, sub-ledger feeds, bank statement uploads, and lender confirmations.
- The agent does not scrape, infer, or synthesize data from sources outside the designated input directories (`data/`).
- No personal data beyond reviewer identity (name, sign-off date) is collected or stored.

---

## Data Use

Data is used exclusively to:

1. Reconcile GL balances to supporting schedules and third-party confirmations.
2. Identify, age, and escalate reconciling items.
3. Assemble the monthly rec package for internal review and audit support.

Data is **never** used for model training, benchmarking, or any purpose outside the reconciliation workflow.

---

## Data Retention

| Artifact | Retention Period | Location |
|---|---|---|
| Workpapers | 7 years | `workpapers/YYYY-MM/` |
| Rec packages | 7 years | `outputs/YYYY-MM/` |
| Exception log | 7 years | `outputs/YYYY-MM/` |
| Raw source inputs | 90 days after close | `data/` (then purged) |
| Agent run logs | 1 year | System audit trail |

Retention periods align with standard audit and SOX documentation requirements. Purges are logged in the audit trail.

---

## Access Controls

- **Restricted workpapers** (sub-ledger detail): Controller and external auditors only.
- **Confidential outputs** (rec package, exception log): Controller, CFO, internal audit.
- **Internal artifacts** (run logs, config): System administrators only.
- Agent outputs are never distributed to external parties by the agent itself — distribution requires explicit human action (see [Human-in-the-Loop Policy](human-in-the-loop-policy.md)).

---

## Data Residency & Transmission

- All processing occurs within the designated environment for Lumina Streaming Co.
- Source data is not transmitted to external APIs or third-party services beyond the configured Anthropic Claude API endpoint used for agent inference.
- API calls contain only the data necessary for the specific reconciliation task in progress; full sub-ledger files are never sent wholesale.

---

## Incident Response

If a data exposure or unauthorized access is suspected:

1. Controller is notified immediately.
2. Affected outputs are quarantined pending review.
3. Incident is logged in the audit trail with timestamp, scope, and remediation steps.
4. CFO is notified if Restricted data is involved.

---

## Policy Ownership

| Role | Responsibility |
|---|---|
| Controller | Day-to-day compliance, access approvals |
| CFO | Policy owner, threshold overrides, escalations |
| System Administrator | Technical controls, retention purges, audit trail integrity |

This policy is reviewed annually or when material changes are made to the agent system or data sources.

---

*This policy should be read alongside the [Human-in-the-Loop Policy](human-in-the-loop-policy.md).*
