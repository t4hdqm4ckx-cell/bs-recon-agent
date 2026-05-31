# Requirements

Functional and non-functional requirements for the Balance Sheet Reconciliation Agent System.

---

## Functional requirements

### Account Reconciliation Agent

| ID | Requirement |
|---|---|
| FR-001 | Reconcile all balance sheet accounts listed in `config/account-map.yaml` for the specified entity and period |
| FR-002 | Tie GL ending balance to independent supporting document (bank statement, sub-ledger, schedule, or third-party confirmation) |
| FR-003 | Identify and document all reconciling items with amount, age, description, and proposed disposition |
| FR-004 | Classify each exception as AUTO-INVESTIGATE, MATERIAL, or TRIVIAL per thresholds in `config/thresholds.yaml` |
| FR-005 | Propose adjusting journal entries for all book-side reconciling items; include account, amount, period, and confidence level |
| FR-006 | Produce a 6-tab workpaper (Summary, Detail, Supporting, Exceptions, Proposed JEs, Audit Trail) per account |
| FR-007 | Produce a markdown memo per account following the format in `skills/bs-reconciliation/SKILL.md` |
| FR-008 | Emit a structured JSON output envelope per account for downstream agent consumption |
| FR-009 | Support all nine reconciliation types: cash, AR, prepaids, fixed assets, AP, accrued liabilities, deferred revenue, intercompany, long-term debt |
| FR-010 | Never post a journal entry directly; all JEs require human approval before posting |

### Exception Management Agent

| ID | Requirement |
|---|---|
| FR-011 | Aggregate reconciling items from all Account Reconciliation Agent output envelopes for the period |
| FR-012 | Age each item from its origination date; promote to At Risk (61d) and Critical (90d+) automatically |
| FR-013 | Route material items to the correct approver (Controller or CFO) based on amount and age thresholds |
| FR-014 | Generate a Controller daily exception summary each morning BD3–BD4 |
| FR-015 | Generate a CFO escalation brief for any item reaching Level 2 escalation |
| FR-016 | Flag recurring exceptions (same account, counterparty, approximate amount across 3+ consecutive periods) |
| FR-017 | Track disposition for every open item; no item may remain undispositioned in the final package |
| FR-018 | Produce the Exception Log with tabs: Exception Log, Aging Summary, Routing Matrix |

### Rec Package Agent

| ID | Requirement |
|---|---|
| FR-019 | Assemble the monthly rec package on BD5 from all Account Reconciliation and Exception Management outputs |
| FR-020 | Produce an Executive Summary showing status, exception count, and net unbooked amount for every account |
| FR-021 | Produce a CFO Sign-off Matrix listing every item requiring approval with owner and due date |
| FR-022 | Produce an Audit Trail linking every output to its source files with SHA-256 hashes and timestamps |
| FR-023 | Output the package as both markdown (for version control) and docx (for distribution) |
| FR-024 | Open items at BD5 must appear in the package with disposition, owner, and expected resolution date — they do not block package assembly |

### Data and configuration

| ID | Requirement |
|---|---|
| FR-025 | All materiality thresholds, aging buckets, and JE approval tiers must be configurable in `config/thresholds.yaml` without code changes |
| FR-026 | All balance sheet accounts must be configurable in `config/account-map.yaml` without code changes |
| FR-027 | Client source data must be loadable from `data/client/` using the naming convention `YYYY-MM_<EntityCode>_<DataType>.<ext>` |
| FR-028 | Synthetic demo data for Lumina Streaming Co. (LuminaUS, November 2026) must be maintained in `data/synthetic/` |

---

## Non-functional requirements

| ID | Requirement |
|---|---|
| NFR-001 | **Auditability** — every output must include source file paths, SHA-256 hashes, agent version, and run timestamp |
| NFR-002 | **Human-in-the-loop** — no journal entry, disposition, or escalation is posted or sent without explicit human approval |
| NFR-003 | **Data confidentiality** — `data/client/` is gitignored and must never be committed; workpapers with sub-ledger detail must not be distributed outside the client's secure environment |
| NFR-004 | **Idempotency** — re-running the agent for the same period and entity must produce the same output given the same source data; version the output file (`_v1`, `_v2`) when inputs change |
| NFR-005 | **Configurability** — a new client must be onboardable by editing config files only, with no changes to agent prompts or skill files |
| NFR-006 | **Traceability** — every reconciling item must be traceable from the final rec package back to a specific line in a source file |
| NFR-007 | **Separation of concerns** — P&L variance work is out of scope; if a P&L issue surfaces during a BS rec, it is noted for handoff to the Flux & Variance Agent and not analyzed |
| NFR-008 | **Output formats** — all primary outputs are produced in both markdown (version-controlled) and docx (distributable) |

---

## Out of scope

- Posting journal entries to any ERP or GL system
- P&L account reconciliation or flux analysis
- Tax provision or deferred tax account recs (requires specialist agent)
- Consolidation eliminations (handled by the consolidation layer of close-system)
- Real-time or intra-month reconciliation; system is designed for month-end close cadence
