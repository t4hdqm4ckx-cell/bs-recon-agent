---
name: finance-conventions
description: "Use this skill whenever a reconciliation, journal entry, or financial statement task needs to apply entity-specific or system-specific accounting conventions — including 'what's the sign convention for this account', 'how does this entity book FX', 'is this a debit-normal or credit-normal account', 'what's the period cutoff for LuminaEMEA', or any task that requires knowing how Lumina Streaming Co.'s books are kept. Use before proposing a JE or interpreting a GL extract so that the agent applies the correct sign, period, and entity conventions. Triggers especially when working across entities (LuminaUS, LuminaEMEA, LuminaAPAC) where conventions differ."
---

# Finance Conventions

## Purpose

Lumina Streaming Co. operates three entities on different reporting standards, currencies, and GL systems. Conventions — sign, period cutoff, FX translation, IC bookkeeping — differ between them. This skill defines what those conventions are so that reconciliations, JEs, and financial statements apply the right rules.

If an account or balance looks wrong, the cause is often a misapplied convention, not an error in the underlying data. Read this skill before assuming a variance is real.

## Reporting framework summary

| Entity | Standard | Functional currency | GL system | Fiscal year-end |
|---|---|---|---|---|
| LuminaUS   | US GAAP  | USD | NetSuite (consolidated) | December 31 |
| LuminaEMEA | IFRS (parent reports under US GAAP) | EUR | NetSuite (consolidated subsidiary) | December 31 |
| LuminaAPAC | IFRS (parent reports under US GAAP) | SGD | NetSuite (consolidated subsidiary) | December 31 |

Consolidated reporting is US GAAP in USD. EMEA and APAC translate to USD monthly at month-end spot for B/S accounts and average-rate for P&L per ASC 830.

## Sign convention

NetSuite GL uses the standard **debit-positive / credit-negative** convention. The "Net Balance" column in the trial balance follows:

| Account type | Normal balance | Sign in TB |
|---|---|---|
| Assets        | Debit  | Positive |
| Liabilities   | Credit | Negative |
| Equity        | Credit | Negative |
| Revenue       | Credit | Negative |
| Expense       | Debit  | Positive |
| Contra-asset (e.g. accumulated depr) | Credit | Negative |

**Rule of thumb:** If a balance has an unexpected sign (e.g., a positive AP balance, a negative AR balance), it is either a real exception (credit balance in AR → reclassify to deferred revenue) or a posting error. Never silently flip the sign — always flag.

### Common normal-balance accounts

| Account family | Normal | Code range |
|---|---|---|
| Cash               | Dr | 100xxx |
| AR                 | Dr | 120xxx |
| Prepaids           | Dr | 130xxx |
| Fixed assets (net) | Dr | 150xxx |
| AP                 | Cr | 200xxx |
| Accrued liabilities| Cr | 210xxx |
| Deferred revenue   | Cr | 220xxx |
| IC Receivable      | Dr | 230xxx (US), 240xxx (EMEA/APAC) |
| IC Payable         | Cr | 230xxx (US), 240xxx (EMEA/APAC) |
| Long-term debt     | Cr | 300xxx |

## Period cutoff

Standard cutoff: **last business day of the calendar month** at local close-of-business in each entity's home time zone. Late-arriving items follow a 3-business-day cutoff extension policy:

| Item type | Cutoff |
|---|---|
| Cash transactions (bank-cleared) | Last calendar day, bank statement date |
| Vendor invoices                  | Receipt by BD+3 to accrue in period |
| Customer billings (auto)         | Last calendar day |
| Customer billings (manual)       | Submitted by BD+1 |
| Intercompany activity            | Last calendar day (both sides) |
| Payroll accruals                 | Last calendar day (semi-monthly cycle) |

**Cross-entity IC cutoff is critical.** All three entities must record both sides of an IC transaction in the same period. If LuminaUS books in November and LuminaEMEA books in December, that creates a timing mismatch that surfaces as an IC mismatch in the IC reconciliation. Default disposition: the entity that recorded in the later period adjusts to match the earlier one.

## FX translation (B/S accounts)

EMEA and APAC trial balances are translated to USD monthly using month-end spot rates per ASC 830. Translation gains/losses go to OCI (account 350100 — Cumulative Translation Adjustment).

| Account class | Rate used | Notes |
|---|---|---|
| Monetary B/S items (cash, AR, AP) | Month-end spot | Re-translated each period |
| Non-monetary B/S items (fixed assets, prepaids) | Historical rate at acquisition | Not re-translated |
| Equity | Historical rate at contribution | Not re-translated |
| P&L items | Monthly average rate | Per ASC 830 |

**FX-driven IC mismatches** are the most common cause of large IC variances. Before flagging an IC item as a "real" exception, check whether translating both sides at the same period rate eliminates the variance.

### Standard FX rates (Nov 2026 demo)

| Pair | Month-end spot | Monthly average |
|---|---|---|
| EUR/USD | 1.0840 | 1.0795 |
| SGD/USD | 0.7390 | 0.7415 |
| GBP/USD | 1.2680 | 1.2645 |

(Source: Bloomberg WMR fix, last business day of month. Used for both consolidation and IC reconciliation.)

## Intercompany conventions

| Rule | Detail |
|---|---|
| Account symmetry | A receivable on one entity = a payable on the counterparty. Never one-sided. |
| Account numbering | LuminaUS uses 230xxx; LuminaEMEA and LuminaAPAC use 240xxx. The numbering identifies the *recording* entity, not the counterparty. |
| FX treatment | Both sides translate at the **same** period-end rate. Differences arising from intra-month rate movements net to zero on consolidation. |
| Settlement | IC balances net-settled quarterly via cash sweep. Open IC balances are not interest-bearing. |
| Profit elimination | Inter-entity sales eliminated on consolidation. Margin held in inventory is reversed via a top-side JE. |

**IC mismatch troubleshooting order:**

1. Same period both sides? If not → timing.
2. Same FX rate both sides? If not → FX (re-translate).
3. Same classification both sides (IC vs third-party)? If not → reclassify.
4. Real economic difference? → escalate to both Controllers; possible billing dispute.

Steps 1–3 cover ~90% of mismatches in practice.

## Materiality interaction

All conventions in this skill are applied **before** materiality is judged. A $267K IC mismatch is not material until you've eliminated FX, timing, and classification as causes. A $50K rounding difference from FX is real but not actionable.

See `skills/materiality-thresholds/SKILL.md` for thresholds.

## Common pitfalls

- **Don't compare unconverted EMEA / APAC numbers to USD.** Always translate first.
- **Don't assume a credit balance is wrong.** Some accounts (refundable deposits, customer advances) legitimately carry credit balances on the asset side — they signal a reclassification need, not an error.
- **Don't backdate to fix a timing issue.** A November-booked item that should have been October is reported in November with an explanatory note. The October books are closed.
- **Don't apply average rate to balance sheet items.** ASC 830 is explicit: spot rate for monetary B/S, historical for non-monetary, average only for P&L.
- **Don't book to a contra-asset on the debit side without explanation.** Reducing accumulated depreciation (debiting it) is unusual outside of asset disposals — flag if you see it.
- **Don't ignore the entity prefix on account codes.** 240100 in LuminaEMEA is a different account than 240100 elsewhere — code + entity is the full key.

## Cross-references

- `config/account-map.yaml` — chart of accounts with entity tags
- `skills/materiality-thresholds/SKILL.md` — thresholds applied after conventions are validated
- `skills/bs-reconciliation/SKILL.md` — calls out FX as a common IC mismatch cause
- `skills/je-review/SKILL.md` — sign-convention check in JE review
- ASC 830 — Foreign Currency Matters (US GAAP source)
