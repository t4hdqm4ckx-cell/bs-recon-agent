# Client Instructions

Welcome. This guide walks you through what to expect from the **Balance Sheet Reconciliation Agent** and what we need from your team each month to run a successful close.

---

## What the system does

The agent reconciles your balance sheet accounts each month:

- Ties your GL balances to supporting schedules (bank statements, sub-ledgers, lender confirms, etc.)
- Identifies reconciling items and ages them
- Proposes adjusting journal entries
- Assembles a sign-off-ready rec package by Business Day 5 (BD5)

**You stay in control.** The agent prepares. Your Controller and CFO approve. Nothing posts and nothing releases without explicit human sign-off.

---

## What we need from you each month

Before BD1 of each close, please drop the following files into `data/client/`:

| File | Source | Cutoff |
|---|---|---|
| Trial balance (all entities) | GL system export | BD1 morning |
| Bank statements | Bank portal / treasury | BD1 morning |
| AR aging report | Sub-ledger | BD1 morning |
| AP aging report | Sub-ledger | BD1 morning |
| Prepaid schedule | Maintained externally | BD1 morning |
| Accrual schedule | Maintained externally | BD1 morning |
| Fixed asset register | FA module | BD1 morning |
| Deferred revenue schedule | Billing system | BD1 morning |
| Intercompany matrix (all entities) | Consolidation tool | BD1 morning |
| Lender statement | Lender portal | BD1 morning |

**File naming:** `YYYY-MM_<entity>_<schedule>.xlsx` (e.g. `2026-11_LuminaUS_BankStatement.xlsx`)

Missing files don't block the run — the agent will pause that account and flag it as a missing-support exception for follow-up.

---

## Timeline

| Day | What happens |
|---|---|
| **BD1** | Data ingested · Validation runs · Account Reconciliation agent starts |
| **BD2** | Reconciliations completed for all accounts · Initial workpapers ready |
| **BD3** | Exceptions identified · Routed to Controller queue |
| **BD4** | Exception resolution · Controller approves proposed JEs |
| **BD5** | Rec package assembled · CFO sign-off · Package released |

---

## What you'll receive

At the end of each close, you get:

1. **Workpapers** (`workpapers/YYYY-MM/`) — one Excel file per account with 6 tabs: Summary, Detail, Supporting, Exceptions, Proposed JEs, Audit Trail. Each has a markdown memo alongside it.

2. **Exception Log** (`outputs/YYYY-MM/`) — every reconciling item with severity, aging bucket, suggested action, and assigned approver.

3. **Rec Package** (`outputs/YYYY-MM/`) — executive dashboard, account summary, CFO sign-off matrix.

4. **Audit envelope** (`outputs/YYYY-MM/`) — machine-readable JSON for handoff to audit tools.

---

## What we need from your Controller

The Controller is the primary approver for the close. Expected actions:

- **Review proposed JEs** in the exception log and sign off in the package matrix.
- **Resolve flagged exceptions** (e.g. confirm a stale check can be voided, decide on a reclass).
- **Sign off each workpaper** by entering name + date in the Audit Trail tab.

The Controller has a 12-hour SLA on most items. Critical items escalate to the CFO at the 24-hour mark.

---

## What we need from your CFO

The CFO signs off on:

- **The final rec package** (always)
- **JEs over $250,000** (per materiality threshold)
- **AUTO-INVESTIGATE exceptions** (any item over $250K)
- **Critical aging items** (anything 90+ days old)
- **Any threshold changes** to `config/thresholds.yaml`

---

## Materiality at a glance

| Threshold | What it means |
|---|---|
| **> $250,000** | Auto-investigate — agent pauses and notifies Controller immediately |
| **> $100,000 OR > 30 days old** | Material — Controller review required |
| **> 90 days old** | Critical — CFO escalation |
| **$450,000 aggregate** | Trivial threshold — items below are noted but excluded from executive memo |

Full thresholds in [`config/thresholds.yaml`](config/thresholds.yaml).

---

## Frequently asked questions

**Will the agent post journal entries automatically?**  
No. Every JE requires Controller sign-off. Nothing posts without human approval.

**Can the agent see my entire general ledger?**  
The agent only reads the source files you drop in `data/client/`. It does not connect directly to your GL system.

**What happens if a reconciliation has a variance?**  
The agent flags the variance as an exception, proposes a JE if there is clear supporting documentation, and routes it to the Controller. The agent never plugs entries to force a reconciliation.

**Can I change the materiality thresholds?**  
Yes — your CFO can approve a change to `config/thresholds.yaml`. The change is logged in the audit trail and takes effect on the next close.

**Where does my data go?**  
Source files stay in your environment for 90 days then are purged. Workpapers and rec packages are retained for 7 years per SOX §802. See [`docs/privacy-policy.md`](docs/privacy-policy.md).

**What if I find a bug or have a question mid-close?**  
Contact your engagement lead. For security issues, follow the private disclosure process in [`security.md`](security.md).

---

## Reference documents

- [`README.md`](README.md) — system overview
- [`CLAUDE.md`](CLAUDE.md) — full architecture and operating rules
- [`GUARDRAILS.md`](GUARDRAILS.md) — hard limits and never-do list
- [`approval_router.md`](approval_router.md) — how items route to approvers
- [`AUDIT.md`](AUDIT.md) — audit trail and SOX alignment
- [`security.md`](security.md) — security policy
- [`docs/human-in-the-loop-policy.md`](docs/human-in-the-loop-policy.md) — approval workflow
- [`docs/privacy-policy.md`](docs/privacy-policy.md) — data handling

---

*Questions? Reach out to your engagement lead. We're here to help you close faster, with a cleaner audit trail.*
