---
name: bs-reconciliation
description: "Use this skill when performing balance sheet account reconciliations during a month-end close — including bank reconciliations, intercompany matching, prepaid amortization verification, accrual schedule reviews, and AR/AP tie-outs to sub-ledger. Triggers include phrases like 'reconcile this account', 'do a bank rec', 'tie out the prepaid schedule', 'check intercompany balances', 'review accruals', 'recon workpaper', or any request to produce a reconciliation deliverable. Use whenever a trial balance, sub-ledger, bank statement, or accrual schedule is provided and the task is to verify the balance is correct, identify reconciling items, and propose any necessary adjusting journal entries."
---

Balance sheet account reconciliation
Purpose
Every balance sheet account on the trial balance must be supported by an underlying schedule, document, or third-party confirmation. A reconciliation proves the GL balance is correct by tying it to that supporting evidence and explaining any difference.

Output of a reconciliation has three parts:

A workpaper showing the rec mechanics with live formulas
An exceptions list flagging items needing investigation or action
Proposed adjusting journal entries to clear known book-side items

The agent proposes. The human disposes. No JE is posted automatically.
General reconciliation pattern
Every reconciliation follows the same structure:

GL balance per source (TB or sub-ledger detail)

+/- Reconciling items (timing, classification, errors not yet booked)

= Independent support balance (bank statement, schedule, third-party confirmation)

If the two sides don't tie after accounting for known reconciling items, there is an unreconciled variance. The size and age of that variance determine whether it requires action this period.
Materiality application
Always read the materiality-thresholds skill first. For balance sheet recs, the relevant defaults are:

Material exception: any unreconciled item > $100K OR > 30 days old
Auto-investigate: any item > $250K regardless of age
Trivial threshold: $450K aggregate; items below this can be noted but require no commentary

Items below the trivial threshold are recorded in the workpaper but excluded from the executive memo.
Reconciliation types
Bank reconciliation
Source of truth: the bank statement. Compare to GL cash account.

Workpaper structure:

Bank statement balance (per the bank)
Plus deposits in transit (recorded in book, not yet on bank statement)
Less outstanding checks (recorded in book, not yet cleared by bank)
Equals adjusted bank balance
Book balance (per GL cash account)
Plus or minus reconciling items the book hasn't yet recorded (wire fees, interest credits, bank service charges)
Equals adjusted book balance
Difference (should be zero or below trivial threshold)

Items to flag:

Outstanding checks aged greater than 60 days: recommend voiding via JE after vendor confirmation
Outstanding deposits greater than 5 days old: contact treasury, possible lost deposit
Bank-side items appearing in book reconciliation or vice-versa: classification error
Unreconciled variance exceeding the trivial threshold: escalate to Controller
Intercompany reconciliation
Source of truth: there is none. Each entity records its own side, and the two sides must mirror.

Workpaper structure: a matrix of every entity pair.

Entity A receivable from Entity B  =  Entity B payable to Entity A   (must mirror)

Items to flag:

Any mismatch where |A receivable − B payable| exceeds the trivial threshold
Stale balances where neither side has moved in more than 60 days
One-sided entries: a balance booked in one entity with no counterparty entry in the other

Most common causes of mismatches: timing (one entity recorded in current month, the other in prior), FX translation differences, or classification disagreement (one party treats as IC, the other as third-party).
Prepaid amortization
Source of truth: the prepaid schedule maintained outside the GL.

Workpaper structure: per asset, prior balance minus current month amortization equals current balance. Sum the asset balances and tie the total to the GL.

Items to flag:

Asset fully amortized but balance remains on GL: propose JE to write off
Amortization period inconsistent with the contract term: schedule error
New additions in period without supporting invoice or contract reference
Schedule total does not tie to GL: investigate before proposing any other action
Accrual review
Source of truth: the accrual schedule maintained outside the GL.

Workpaper structure: per accrual, prior balance plus current period activity minus reversal equals current balance.

Items to flag:

Stale accruals: prior-period accruals that should have reversed but remain on the books. This is the highest-frequency finding in accrual reviews and the most material in dollar terms.
Accruals more than 90 days old without a "recurring" or "active" reversal status
Accrued amount significantly different from the related invoice when received
Missing accrual for a known obligation (validate against the AP "received not invoiced" list)
AR / AP tie-out to sub-ledger
Source of truth: the sub-ledger aging report.

The tie-out is mechanical: GL balance must equal the sum of all aging buckets. Differences are typically timing (entries posted to GL but not sub-ledger or vice versa) or unposted journal entries.

Items to flag:

90+ day AR exceeding 5% of total AR balance
Negative AR balances (credit balances should be reclassified to deferred revenue)
AP older than 60 days when standard terms are net 30 — possible dispute or process issue
Output format
Workpaper file (xlsx)
Filename: YYYY-MM_<entity>_<account>Rec_v<n>.xlsx

Tabs:

Summary — top-of-the-house: account, period, GL balance, support balance, difference, exception count, status, preparer / reviewer block
Detail — the actual reconciliation mechanics with live formulas
Supporting — outstanding items detail, aging analysis, supporting calculations
Exceptions — items needing action with proposed disposition
Proposed JEs — adjusting entries to clear book-side reconciling items
Audit Trail — sources with hashes, agent metadata, sign-off fields
Memo file (md)
Filename: YYYY-MM_<entity>_<scope>Recon_v<n>_memo.md

Sections:

Executive summary (3–5 lines)
Findings — exceptions prioritized by severity
Proposed adjusting entries with debit, credit, amount, account, description
Open items requiring human decision
Metadata — agent version, source files with hashes, timestamp, reviewer field
JE proposal format
Every proposed JE includes:

Description — one sentence, plain English
Debit account — six-digit code plus name
Credit account — six-digit code plus name
Amount — in actual dollars, not thousands
Period — close period the JE belongs to
Reason — why the entry is needed
Source — the document, transaction, or rec workpaper that supports it
Confidence — high, medium, or low. Drives whether the entry can be auto-approved or must be reviewed by a human.

JEs are never posted directly. They are surfaced to a human reviewer who decides whether to approve.
Output envelope
Every reconciliation returns a structured envelope that the Close Orchestrator can route:

{

  "result": {

    "account": "100100",

    "account_name": "Cash - Operating",

    "entity": "LuminaUS",

    "gl_balance": 142500000,

    "support_balance": 142500000,

    "difference": 0,

    "workpaper_path": "...",

    "memo_path": "..."

  },

  "exceptions": [

    {

      "severity": "high | medium | low",

      "category": "...",

      "description": "...",

      "proposed_action": "..."

    }

  ],

  "_metadata": {

    "agent": "reconciliation",

    "version": "0.1.0",

    "run_timestamp": "ISO-8601",

    "sources": [{"path": "...", "sheet": "...", "sha256": "..."}],

    "human_reviewer": null

  }

}
Common pitfalls
Don't reconcile to the wrong source. If the bank statement is in a different cutoff than the GL (common around quarter-end), pause and request the correct one.
Don't auto-clear items without explanation. Every reconciling item has a reason. Plug entries are forbidden.
Round numbers are suspicious. A reconciling item of exactly $1,500,000 is more likely an error than $1,487,234.
Watch for sign errors. GL convention varies by system; read finance-conventions for the entity in question.
Aging matters as much as amount. A $25K item that's been outstanding 90 days is more interesting than a $250K item that just appeared.
Don't drift into P&L. P&L variance work belongs to the Flux & Variance Agent. If a P&L issue surfaces during BS rec, note it for handoff but don't analyze.

