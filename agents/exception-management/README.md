# Exception Management Agent
**Status:** Operational (v0.1.0)

Runs BD3–BD4. Aggregates all reconciling items from the Account Reconciliation agent, classifies them by materiality (AUTO-INVESTIGATE >$250K / MATERIAL >$100K or >30d / TRIVIAL), assigns aging buckets (Current/Aged/At Risk/Critical), and routes to approver (Controller, CFO) per `config/thresholds.yaml` and `approval_router.md`.

**First run:** Nov 2026 close — output at `outputs/2026-11/2026-11_Exception_Log.xlsx` (Exception Log, Aging Summary, Routing Matrix tabs).
