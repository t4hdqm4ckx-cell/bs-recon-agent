# Client Onboarding Guide

How to configure and deploy the Balance Sheet Reconciliation Agent System for a new client engagement.

---

## Overview

The system is driven by two config files and a data folder. Onboarding a new client means updating those three things — no code changes required.

| What | File | Purpose |
|---|---|---|
| Thresholds | `config/thresholds.yaml` | Materiality, aging buckets, JE approval tiers, rec-type flags |
| Account map | `config/account-map.yaml` | Which accounts to reconcile and what type each is |
| Source data | `data/client/` | Trial balance, sub-ledgers, bank statements, schedules |

---

## Step 1 — Calibrate thresholds

Copy the Lumina defaults and adjust for client size and risk tolerance.

```yaml
# config/thresholds.yaml
client: <client-slug>          # lowercase, hyphenated, used in output filenames

materiality:
  exception_amount_usd:        # flag individual items above this amount
  exception_age_days:          # flag items older than this (either condition triggers)
  auto_investigate_usd:        # investigate immediately regardless of age
  trivial_threshold_usd:       # aggregate below which items need no commentary

  current_max_days:   30       # aging bucket boundaries — adjust if client uses different
  aged_max_days:      60       # buckets (some use 30/60/90/120; adjust accordingly)
  at_risk_max_days:   90

  je_no_approval_usd:          # < this: post with notation
  je_controller_usd:           # this to next tier: Controller approval
  je_cfo_usd:                  # this to next tier: CFO approval
                               # > je_cfo_usd: CFO + Audit Committee

rec_types:
  bank:
    outstanding_check_flag_days:   # flag outstanding checks older than this
    deposit_in_transit_flag_days:  # flag deposits in transit older than this
  intercompany:
    mismatch_flag_usd:             # flag IC mismatches above this
  ar:
    aging_90_plus_flag_pct:        # flag if 90+ day AR exceeds this % of total AR
  accruals:
    stale_accrual_flag_days:       # flag accruals without reversal older than this
  prepaids:
    unamortized_variance_usd:      # flag if prepaid schedule doesn't tie within this
```

**Sizing guidance by company revenue:**

| Revenue | `exception_amount_usd` | `trivial_threshold_usd` | `auto_investigate_usd` |
|---|---|---|---|
| < $50M | $25,000 | $100,000 | $75,000 |
| $50M–$250M | $50,000 | $200,000 | $150,000 |
| $250M–$1B | $100,000 | $450,000 | $250,000 (Lumina default) |
| > $1B | $250,000+ | $1,000,000+ | $500,000+ |

Always confirm materiality thresholds with the engagement partner or Controller before the first close run.

---

## Step 2 — Build the account map

Map the client's chart of accounts to the rec types the system understands. Only balance sheet accounts go here — P&L accounts belong to the Flux & Variance Agent.

```yaml
# config/account-map.yaml
# Six-digit COA is the default; adjust `code` format to match client GL

assets:
  cash:
    - {code: "XXXXXX", name: "Cash - Operating", entity: EntityName}
  accounts_receivable:
    - {code: "XXXXXX", name: "AR - Trade", entity: EntityName}
  prepaids:
    - {code: "XXXXXX", name: "Prepaid Insurance"}
  fixed_assets:
    - {code: "XXXXXX", name: "PP&E - Net"}

liabilities:
  accounts_payable:
    - {code: "XXXXXX", name: "AP - Trade"}
  accrued_liabilities:
    - {code: "XXXXXX", name: "Accrued Compensation"}
  deferred_revenue:
    - {code: "XXXXXX", name: "Deferred Revenue - Subscription"}
  intercompany:
    - {code: "XXXXXX", name: "IC Receivable - Entity A"}
    - {code: "XXXXXX", name: "IC Payable - Entity A"}
  long_term_debt:
    - {code: "XXXXXX", name: "Long-Term Debt"}
```

**Supported rec types** (value in the `category` column of your trial balance export must match these keys):

| Key | Rec type | Required supporting document |
|---|---|---|
| `cash` | Bank tie-out | Bank statement |
| `accounts_receivable` | Subledger aging tie-out | AR aging report |
| `prepaids` | Amortization schedule | Prepaid schedule |
| `fixed_assets` | Asset register | Fixed asset register |
| `accounts_payable` | Subledger aging tie-out | AP aging report |
| `accrued_liabilities` | Accrual schedule review | Accrual schedule |
| `deferred_revenue` | Billing schedule | Deferred revenue schedule |
| `intercompany` | IC matrix matching | IC matrix |
| `long_term_debt` | Lender statement | LTD statement / amortization schedule |

If a client account type has no match above, add a row to this table and raise a request to extend the system — do not force-fit.

---

## Step 3 — Load source data

Drop client files into `data/client/`. This folder is gitignored and never committed.

**Required file naming convention:**

```
YYYY-MM_<EntityCode>_<DataType>.<ext>
```

Examples:
```
data/client/
  2026-11_ClientUS_TrialBalance.csv
  2026-11_ClientUS_BankStatement_100100.csv
  2026-11_ClientUS_AR_Aging.csv
  2026-11_ClientUS_AP_Aging.csv
  2026-11_ClientUS_Prepaid_Schedule.csv
  2026-11_ClientUS_Accrual_Schedule.csv
  2026-11_ClientUS_FixedAsset_Register.csv
  2026-11_ClientUS_DeferredRevenue_Schedule.csv
  2026-11_ClientUS_LTD_Statement.csv
  2026-11_IC_Matrix.csv
```

**Trial balance minimum columns:** `account_code`, `account_name`, `entity`, `category`, `ending_balance`, `normal_side`

**Bank statement minimum columns:** `date`, `description`, `amount`, `running_balance`

**Aging reports minimum columns:** `customer_or_vendor`, `invoice_number`, `invoice_date`, `due_date`, `amount`, `days_outstanding`

If the client export uses different column names, map them in a brief note at the top of the session — the agent will use that mapping for the run.

---

## Step 4 — Set entity names

Entity codes in `account-map.yaml` and data filenames must match. Use short codes (e.g., `ClientUS`, `ClientEMEA`) that are unique and contain no spaces. These codes appear in all output filenames and workpaper headers.

For multi-entity engagements, confirm the consolidation currency and FX translation method with the client before running intercompany recs — translation differences are the most common source of IC mismatches that are not actually errors.

---

## Step 5 — First run checklist

Before running the first close period:

- [ ] `config/thresholds.yaml` — `client:` field updated, all thresholds set, reviewed with Controller
- [ ] `config/account-map.yaml` — all BS accounts listed, categories match supported rec types
- [ ] `data/client/` — trial balance and all supporting schedules loaded for the period
- [ ] Entity codes consistent across config and data filenames
- [ ] JE approval tiers confirmed with Controller (who approves what, and at what dollar threshold)
- [ ] Escalation contacts identified: Controller name, CFO name, Audit Committee contact if applicable
- [ ] Output folders created: `outputs/YYYY-MM/`, `workpapers/YYYY-MM/`

---

## Output folders

Create period folders before the first run. They are not auto-created.

```bash
mkdir -p outputs/YYYY-MM workpapers/YYYY-MM
```

Workpapers (detailed, may contain subledger data — restrict distribution):
```
workpapers/YYYY-MM/
```

Outputs (executive-level memos — safe for Controller / CFO distribution):
```
outputs/YYYY-MM/
```

---

## Sensitive data handling

- `data/client/` is gitignored. Never commit client files.
- Workpapers containing subledger detail (customer names, vendor names, transaction-level data) should not be emailed in plain text — share via the client's secure portal or SharePoint.
- Executive output memos show account-level summaries only. They are safe to share with the CFO and Audit Committee.
- If a client requires data to stay on-premise, run the agent locally and do not sync `data/client/` or `workpapers/` to any cloud storage.

---

## Updating thresholds mid-engagement

If the Controller requests a threshold change after the first close run, update `config/thresholds.yaml`, note the change in the commit message, and re-run any recs affected by the change. Do not update thresholds retroactively to clear exceptions — the original exception stands and should be resolved on its own terms.
