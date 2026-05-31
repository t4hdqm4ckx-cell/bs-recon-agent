# Bug Fix Log

Known issues, fixes applied, and workarounds for the Balance Sheet Reconciliation Agent System.

---

## Open issues

| ID | Severity | Component | Description | Workaround |
|---|---|---|---|---|
| BUG-004 | Medium | Account Reconciliation | IC matrix mismatch classifier treats FX translation differences as errors rather than expected timing items when entities operate in different functional currencies | Manually mark FX-driven IC mismatches as "Deferred – timing" in the exception log; agent will be updated to detect FX deltas from the exchange rate table |
| BUG-005 | Low | Rec Package Agent | docx output does not preserve table column widths; wide tables (e.g., account summary) wrap awkwardly in Word | Open the docx and auto-fit columns before distributing (`Table → AutoFit → AutoFit to Window`) |
| BUG-006 | Low | Exception Management | Recurring exception detection counts periods by calendar month, not fiscal period; clients with non-calendar FYE may see incorrect recurrence counts | No workaround needed for Lumina (calendar FYE); non-calendar clients should verify recurrence flags manually |

---

## Resolved issues

### BUG-001 — Bank rec sign error on credit-balance accounts
**Severity:** High | **Component:** Account Reconciliation — bank rec
**Reported:** 2026-11-28 | **Fixed:** 2026-11-29 | **Fixed in:** v0.1.1

**Symptom:** When a bank account carried a book overdraft (credit balance), the adjusted book balance was calculated with the wrong sign, producing a spurious unreconciled variance equal to 2× the overdraft amount.

**Root cause:** The reconciliation formula assumed all cash accounts have a debit normal side. The `normal_side` column in the trial balance export was not being read.

**Fix:** Agent now reads `normal_side` from the trial balance before computing the adjusted book balance. Credit-normal accounts have their reconciling items sign-flipped before the difference is calculated.

**Test case:** Money market account (100200) with a hypothetical $50K overdraft now reconciles cleanly.

---

### BUG-002 — Prepaid schedule tie-out double-counting additions
**Severity:** Medium | **Component:** Account Reconciliation — prepaid amortization
**Reported:** 2026-11-30 | **Fixed:** 2026-12-01 | **Fixed in:** v0.1.2

**Symptom:** When a prepaid asset had both an addition and amortization in the same period, the schedule total exceeded the GL balance by the addition amount, generating a false variance.

**Root cause:** The period-end formula was computing `prior_balance - amortization + additions` correctly but then adding `additions` a second time when summing the schedule to the GL tie-out row.

**Fix:** Schedule total now sums only the `ending_balance` column per asset line, not the additions column separately.

**Test case:** Prepaid Software (130200) with a $480K November addition and $880K amortization now ties to the $5,800,000 GL balance without variance.

---

### BUG-003 — Exception age calculated from run date, not origination date
**Severity:** Medium | **Component:** Exception Management
**Reported:** 2026-12-02 | **Fixed:** 2026-12-03 | **Fixed in:** v0.1.3

**Symptom:** Exception ages reset to zero each time the agent ran, causing At Risk and Critical promotions to never trigger. An item first identified in September 2026 appeared as 0 days old on the December run.

**Root cause:** The age field was being populated with `today - run_date` instead of `today - origination_date`. The origination date was present in the output envelope but not being carried forward into the exception log.

**Fix:** Exception Management Agent now reads `origination_date` from the source envelope and persists it across runs. Age is always computed as `run_date - origination_date`.

**Verification:** Check #4468 (issued 2026-09-19) correctly shows 72 days outstanding on the November 30 run.

---

## Reporting a new issue

Open an issue on the GitHub repository with:
1. The agent and component affected
2. The period and entity where the issue occurred
3. The expected output vs. the actual output
4. The source files involved (do not attach client data — describe the shape of the data instead)
5. Whether a workaround exists
