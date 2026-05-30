---
name: je-review
description: "Use this skill any time a proposed journal entry needs to be validated, classified, routed for approval, or queued for posting — including 'review this JE', 'is this entry correct', 'who approves this', 'can I post this', 'check the proposed adjustments', or any task that takes an agent-generated JE and prepares it for human disposition. Triggers especially during balance sheet reconciliations when the Account Reconciliation agent proposes adjusting entries, and during exception management when stale items require clearing entries. Use this skill before routing any JE to a human reviewer so the agent does not propose entries that violate format, account mapping, or approval-tier rules."
---

# Journal Entry Review

## Purpose

Every journal entry proposed by an agent must pass a structured review before it reaches a human approver. This skill defines:

1. **What a valid proposed JE looks like** (format requirements).
2. **What checks the agent must run** before surfacing it.
3. **How the JE is routed** to the right approver.
4. **What happens after** — approval, rejection, or revision.

The agent **never posts JEs directly**. The agent prepares; a human disposes. This skill ensures what reaches the human is complete, defensible, and routed correctly.

## JE format requirements

Every proposed journal entry must include all of the following fields. Missing any field = rejected by this skill before reaching a reviewer.

| Field | Description | Example |
|---|---|---|
| `description` | One sentence, plain English | "Record Nov wire transfer fee" |
| `debit` | Six-digit account code + name | "720100 Bank Fees" |
| `credit` | Six-digit account code + name | "100100 Cash - Operating" |
| `amount` | In actual dollars (not thousands) | 118000.00 |
| `period` | Close period the JE belongs to | "2026-11" |
| `reason` | Why the entry is needed | "Fee on bank statement, not in GL" |
| `source` | Document, transaction, or workpaper supporting it | "Bank statement line 4" |
| `confidence` | high / medium / low | "high" |
| `approval_tier` | Routing per materiality | "Controller" |

JEs with debit ≠ credit are invalid and never proposed. Multi-line JEs are allowed; total Dr must equal total Cr.

## Pre-routing checks

Before any JE is surfaced to a reviewer, the agent runs these checks in order. A failure halts the JE — it is logged in the workpaper Exceptions tab but never enters the approval queue.

### 1. Format check
- All required fields populated.
- Account codes match `config/account-map.yaml`.
- Debit total = Credit total.
- Amount > $0.

### 2. Account mapping check
- Debit and credit accounts both exist in the chart of accounts.
- Account categories make sense together (e.g., never P&L on both sides of a BS rec JE).
- Cross-entity entries (`LuminaUS` → `LuminaEMEA`) use the correct IC accounts.

### 3. Period check
- JE period matches the active close period.
- No backdated entries to closed periods — those require a separate Re-open Workflow.

### 4. Duplicate check
- No other proposed JE in the current run has the same description + debit + credit + amount within $1.
- If a duplicate is detected, the agent merges them into a single entry with a combined reason.

### 5. Plug-entry guard
- **Plug entries are forbidden.** Every JE must have a clear "reason" that ties to a source document or workpaper line.
- Reasons containing the words "plug", "balance", "to balance", "force", or "to clear difference" are auto-rejected.
- If the agent cannot identify the underlying transaction, the item enters the exception log instead of becoming a JE.

### 6. Round-number suspicion check
- JEs at exactly $100,000 / $500,000 / $1,000,000 / similar round figures are flagged with `confidence: low` and require manual reviewer scrutiny, even if other checks pass.
- A reconciling item of exactly $1,500,000 is more likely an error than $1,487,234. Flag, don't auto-classify as high confidence.

### 7. Sign-convention check
- Confirms debits/credits match the entity's GL convention (per `skills/finance-conventions/SKILL.md`).
- Common error: booking a credit to a normal-debit account without explicit reason.

## Approval-tier routing

Pulled from `skills/materiality-thresholds/SKILL.md` and `config/thresholds.yaml`. The agent assigns the tier; the Exception Management agent routes to the named approver.

| Amount | Approval tier | Approver | SLA |
|---|---|---|---|
| < $10,000 | No approval — post with notation | Auto | — |
| $10,000 – $250,000 | Controller | Controller | 12 hours |
| $250,000 – $1,000,000 | CFO | CFO | 4 hours |
| > $1,000,000 | CFO + Audit Committee | CFO + AC | 24 hours |

**Override conditions** (any one promotes the tier by one level):
- Confidence = low
- Source = inferred / derived (no direct supporting document)
- Account = IC, Goodwill, Long-Term Debt, or Equity (always at least Controller)
- Period = prior close

## Confidence assignment

| Confidence | When to assign |
|---|---|
| `high` | Direct supporting document (bank stmt line, vendor invoice, contract). Math is exact. Reason is unambiguous. |
| `medium` | Supporting schedule but interpretation required (e.g., stale check void — assumes payee won't cash). |
| `low` | Inferred from indirect evidence; round-number suspicion; no single source document. Always escalates one approver tier. |

## Output format

Each proposed JE appears in two places:

1. **Workpaper `Proposed JEs` tab** (Excel) — full structured row with all required fields.
2. **Memo `## Proposed Adjusting Entries` section** (Markdown) — human-readable narrative with reason, source, confidence, and approval tier called out.

Both must be byte-equivalent in content. The Excel row is what the approval router consumes; the memo is what the reviewer reads.

### Memo format example

```markdown
**JE-1** — Record Nov wire transfer fee
- Dr. 720100 Bank Fees / Cr. 100100 Cash - Operating — **$118,000.00**
- Reason: Fee on bank statement but not recorded in GL
- Source: Bank statement line 4  |  Confidence: high  |  Approval: Controller
```

## Post-routing flow

Once routed, the JE enters one of three states:

| State | Trigger | Next step |
|---|---|---|
| Approved | Reviewer signs off | JE moves to posting queue. Audit trail records approver, role, timestamp. |
| Rejected | Reviewer rejects with notes | JE re-queued for agent revision. Revision count incremented. After 3 failed revisions, automatic CFO escalation. |
| Returned | Reviewer requests changes | Same as rejected, but with specific revision guidance attached. |

See `approval_router.md` for full state machine details.

## Common pitfalls

- **Don't propose JEs to clear unreconciled variances without explanation.** That's a plug entry. Surface the variance as an exception instead and let the human decide.
- **Don't auto-merge JEs across accounts.** Multiple JEs hitting one account are routed individually; don't combine them to fit under a lower approval tier.
- **Don't propose JEs that hit P&L during a BS rec without flagging it.** P&L impact during balance-sheet work needs explicit reviewer awareness — call it out in the reason field.
- **Don't bypass the duplicate check.** If the same JE was proposed in a previous run and rejected, re-proposing it without addressing the rejection notes is grounds for auto-rejection.
- **Don't set confidence = high for inferred entries.** If you derived the amount from a calculation rather than reading it from a source, the highest confidence is medium.

## Cross-references

- `skills/materiality-thresholds/SKILL.md` — dollar thresholds that drive approval tier
- `skills/finance-conventions/SKILL.md` — entity-specific sign conventions
- `skills/bs-reconciliation/SKILL.md` — when reconciliation work surfaces proposed JEs
- `approval_router.md` — full state machine for approvals
- `GUARDRAILS.md` — hard stops; agent never posts JEs autonomously
- `config/account-map.yaml` — valid debit/credit accounts
