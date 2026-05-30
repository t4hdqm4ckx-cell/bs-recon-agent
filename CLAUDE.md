# Balance Sheet Reconciliation Agent System

> Multi-agent system for month-end balance sheet account reconciliations.
> Covers bank, intercompany, prepaids, accruals, fixed assets, AR/AP, and
> deferred revenue. Built as a consulting portfolio piece for Controllers,
> CFOs, and Audit teams.

## Architecture

Three agents on a shared layer of Skills, config, and data:

| Agent | Role | Runs |
|---|---|---|
| **Account Reconciliation** | Reconciles all BS accounts: ties GL to support, identifies reconciling items, proposes JEs | BD1–BD3 |
| **Exception Management** | Tracks, ages, and resolves open reconciling items across periods; escalates based on materiality | BD3–BD4 |
| **Rec Package** | Assembles the monthly rec package: summary dashboard, workpapers, exception log, CFO sign-off matrix | BD5 |

## Demo company

`Lumina Streaming Co.` — same synthetic company as close-system and headcount-agent.

- Three entities: LuminaUS, LuminaEMEA, LuminaAPAC
- FYE December, USD / US GAAP
- Close period: November 2026, BD5 target

## Account coverage

| Category | Accounts | Rec type |
|---|---|---|
| Cash | 100100 Operating, 100200 Money Market | Bank statement tie-out |
| Accounts Receivable | 120100 Trade AR, 120200 Advertising AR | AR subledger aging tie-out |
| Prepaids | 130100–130400 | Amortization schedule |
| Fixed Assets | 150100 PP&E, 150200 Intangibles | Fixed asset register |
| Accounts Payable | 200100 AP Trade, 200200 AP Accrued | AP subledger tie-out |
| Accrued Liabilities | 210100–210500 | Accrual schedule review |
| Deferred Revenue | 220100–220200 | Subscriber billing schedule |
| Intercompany | 230100–230200 (US), 240100–240200 (EMEA/APAC) | IC matrix matching |
| Long-Term Debt | 300100 | Lender statement / amortization |

## Materiality thresholds

- Material exception: > $100,000 OR > 30 days old (either condition)
- Auto-investigate: > $250,000 regardless of age
- Trivial threshold: $450,000 aggregate
- Aging buckets: Current (0–30d), Aged (31–60d), At Risk (61–90d), Critical (90d+)

All thresholds in `config/thresholds.yaml`.

## Operating rules

- Agent proposes. Human disposes. No JE is posted without Controller approval.
- Every workpaper includes the preparer (agent), reviewer (human), and sign-off date.
- Reconciling items must be cleared within 90 days or escalated to the CFO.
- Individual sub-ledger detail stays in restricted workpapers — executive outputs show account-level summaries only.
- Audit trail: every output envelope includes sources, timestamps, and agent version.

## Output conventions

File naming: `YYYY-MM_<entity>_<account>_Rec_v<n>.<ext>`
Example: `2026-11_LuminaUS_Cash_Rec_v1.xlsx`

Workpapers: `workpapers/YYYY-MM/`
Outputs: `outputs/YYYY-MM/`

## Related systems

- [close-system](https://github.com/t4hdqm4ckx-cell/close-system) — month-end close automation; Reconciliation Agent in that system is a lighter version of this dedicated system
- [headcount-agent](https://github.com/t4hdqm4ckx-cell/headcount-agent) — people cost analysis
