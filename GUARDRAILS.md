# Guardrails

**System:** Balance Sheet Reconciliation Agent  
**Applies to:** Account Reconciliation · Exception Management · Rec Package  
**Effective:** 2026-05-30

---

## Purpose

Guardrails define the hard limits and behavioral boundaries the agent system enforces automatically — actions it will never take, outputs it will never produce, and conditions that force an immediate stop regardless of instruction.

---

## Hard Stops (Agent Halts Unconditionally)

The agent stops processing and requires human intervention when any of the following conditions are met:

| Condition | Threshold | Agent Action |
|---|---|---|
| Reconciling item exceeds auto-investigate limit | > $250,000 | Pause, flag, notify Controller |
| GL balance cannot be tied to any supporting schedule | Any amount | Pause, log gap, request support |
| Required third-party confirmation is missing at rec time | Any account | Pause, request document |
| Open item reaches critical aging bucket | 90+ days | Pause, escalate to CFO |
| Aggregate trivial items exceed threshold | > $450,000 | Pause, notify Controller |
| Source data checksum or record count fails validation | Any file | Abort run, log error |

---

## Actions the Agent Will Never Take

Regardless of prompt, instruction, or configuration:

- **Post or reverse a journal entry** without an approved Controller sign-off on record.
- **Mark a reconciling item as cleared** without explicit human confirmation.
- **Release or distribute the rec package** to any external party (auditors, lenders, board).
- **Modify `config/thresholds.yaml`** at runtime or during a reconciliation run.
- **Access sub-ledger detail** outside the active reconciliation workflow.
- **Suppress or omit a reconciling item** from the exception log, regardless of size.
- **Backdate a workpaper** or alter a previously signed-off sign-off date.
- **Infer missing support** — if a document is absent, the agent flags it rather than substituting estimated or synthetic data.

---

## Output Boundaries

| Output type | Agent may produce | Agent may NOT produce |
|---|---|---|
| Workpapers | Draft with preparer stamp | Finalized (requires human Reviewer + sign-off date) |
| Journal entries | Proposed JE with support | Posted or approved JE |
| Exception log | Open items, aging, root-cause notes | Closed items without human confirmation |
| Rec package | Assembled draft | Released package without CFO sign-off matrix |
| Executive summary | Account-level summaries | Sub-ledger line-item detail |

---

## Input Validation

Before any reconciliation run begins, the agent validates:

1. **Source file integrity** — record counts and checksums match expected values.
2. **Period consistency** — all source files reference the same close period.
3. **Entity coverage** — GL export includes all three entities (LuminaUS, LuminaEMEA, LuminaAPAC).
4. **Config version** — `config/thresholds.yaml` version matches the agent's expected schema.

A failed validation aborts the run and logs the specific failure before any data is processed.

---

## Escalation Chain

When a guardrail is triggered, the agent follows this sequence:

```
Agent detects condition
    → Logs event in audit trail (timestamp, condition, data context)
    → Pauses workflow
    → Notifies Controller
        → If Restricted data or CFO-level threshold: also notifies CFO
    → Awaits explicit human instruction to resume or abort
```

The agent does not resume automatically after a hard stop.

---

## Guardrail Override

No guardrail may be overridden at runtime by the agent itself. Threshold changes require:

1. CFO approval (documented).
2. Manual edit of `config/thresholds.yaml` by a system administrator.
3. Entry in the audit trail recording who changed what and why.

Overrides applied mid-close-cycle take effect on the next run, not retroactively.

---

*Read alongside the [Human-in-the-Loop Policy](docs/human-in-the-loop-policy.md) and [Privacy Policy](docs/privacy-policy.md).*
