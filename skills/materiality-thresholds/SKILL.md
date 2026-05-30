---
name: materiality-thresholds
description: "Use this skill any time a reconciliation, exception, journal entry, or close output requires a materiality judgment — including 'is this material', 'do I need to flag this', 'which approver routes this JE', 'is this item trivial', 'is this an auto-investigate trigger', or any task that needs to know the dollar / aging cutoffs in effect for the current client. Read this skill before applying any threshold so that all agents work from the same numbers and the audit trail reflects a single source of truth."
---

# Materiality Thresholds

## Purpose

Every reconciliation, exception, and journal entry in this system is judged against a defined materiality framework. This skill is the **single source of truth** for those thresholds. All agents — Account Reconciliation, Exception Management, Rec Package — read this skill first before flagging an item, routing an approval, or excluding something from executive output.

If a threshold appears anywhere else (a workpaper, a memo, a config snippet), it must match the values defined here. Discrepancies are bugs.

## Source of truth

Values live in `config/thresholds.yaml`. This document explains how to **apply** them; the YAML is the authoritative store. Any runtime override requires CFO approval and a manual edit to that file, logged in the audit trail per `GUARDRAILS.md`.

## Current values (Lumina Streaming Co.)

### Individual item thresholds

| Threshold | Value | Trigger condition |
|---|---|---|
| Material exception | **> $100,000** **OR** **> 30 days old** | Either condition flips the item to MATERIAL |
| Auto-investigate | **> $250,000** | Agent pauses and notifies Controller immediately; CFO notified |
| Trivial threshold | **$450,000 aggregate** | Items below this in aggregate need no commentary — recorded in workpaper, excluded from executive memo |

### Aging buckets

| Bucket | Days |
|---|---|
| Current  | 0–30 |
| Aged     | 31–60 |
| At Risk  | 61–90 |
| Critical | 90+ |

### Journal entry approval tiers

| Amount | Approver |
|---|---|
| < $10,000 | No approval — post with notation |
| $10,000 – $250,000 | Controller |
| $250,000 – $1,000,000 | CFO |
| > $1,000,000 | CFO + Audit Committee |

### Rec-type-specific triggers

| Rec type | Field | Threshold |
|---|---|---|
| Bank | Outstanding check flag | > 60 days |
| Bank | Deposit-in-transit flag | > 5 days |
| Intercompany | IC mismatch flag | > $50K |
| AR | 90+ aging flag | > 5% of total AR |
| Accruals | Stale accrual flag | > 45 days without reversal |
| Prepaids | Schedule vs GL variance | > $5K |

## Application rules

### Classification logic (apply in this order)

```
IF amount > $250,000:
    classification = AUTO-INVESTIGATE
    approver       = CFO
    SLA            = 2–4 hours
ELIF amount > $100,000  OR  age > 30 days:
    classification = MATERIAL
    approver       = Controller (CFO notify on high severity)
    SLA            = 4–12 hours
ELIF amount + aggregate of similar < trivial threshold:
    classification = TRIVIAL
    approver       = Note only — no escalation required
    SLA            = N/A
ELSE:
    classification = MATERIAL by aggregation
```

The first matching rule wins. "OR" between dollar and age means **either** condition independently triggers — a $50K item that has been open 35 days is still MATERIAL.

### What goes where

- **Executive memo / Rec Package summary** — material items only. Trivial items are noted in workpapers, omitted from CFO-facing outputs.
- **Exception Log** — every flagged item, regardless of size. Trivial items appear with classification = TRIVIAL.
- **Aging buckets** — applied to all open items, used by Exception Management to drive SLA escalations.

### When to escalate aging

| Bucket | Required action |
|---|---|
| Current | Monitor — included in standard log |
| Aged | Controller review at next close |
| At Risk | Escalate to CFO — appears in CFO sign-off matrix |
| Critical | CFO + external auditor notification — must be cleared or formally extended |

## Override process

Thresholds are **not changeable at runtime by the agent**. To change a value:

1. CFO documents approval (email or sign-off matrix entry).
2. System administrator edits `config/thresholds.yaml`.
3. Edit is logged in audit trail with old value, new value, approver name, timestamp.
4. New threshold takes effect on the **next** close run — never retroactively.

See `GUARDRAILS.md` for the full override workflow.

## Common pitfalls

- **Don't combine OR conditions inconsistently.** `amount > $100K OR age > 30d` means either one is enough. Don't accidentally code it as AND.
- **Don't apply the trivial threshold per-item.** It's aggregate — the sum of all individually-immaterial items.
- **Don't infer thresholds from precedent.** If a prior workpaper used $75K as the cutoff, that was wrong. Always read the YAML; never copy a number from another workpaper.
- **Don't combine dollar tiers across approvers.** A $200K JE goes to Controller, not CFO, even if it's part of a series of related entries totaling > $1M. Each JE is routed individually unless explicitly batched.
- **Always re-read this skill at the start of a close cycle.** Thresholds may have been changed since your last run.

## Cross-references

- `config/thresholds.yaml` — authoritative values
- `GUARDRAILS.md` — hard stops triggered by these thresholds
- `approval_router.md` — how thresholds drive routing to approvers
- `skills/bs-reconciliation/SKILL.md` — how reconciliation agent applies these
- `skills/je-review/SKILL.md` — how proposed JEs are routed
