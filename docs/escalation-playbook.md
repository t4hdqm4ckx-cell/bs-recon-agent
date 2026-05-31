# Escalation Playbook

How the Exception Management Agent routes open reconciling items to human reviewers, and what those reviewers are expected to do.

---

## Escalation hierarchy

| Level | Who | Triggers |
|---|---|---|
| 1 — Controller | Controller / Accounting Manager | Items $10K–$250K; any material exception (>$100K or >30d); items needing JE approval |
| 2 — CFO | CFO | Items $250K–$1M; any At Risk item (61–90d) unresolved after Controller review; aggregate exceptions > $1M |
| 3 — Audit Committee | CFO + Audit Committee Chair | Items > $1M; Critical items (90d+) unresolved; suspected fraud or error requiring restatement |

All thresholds are in `config/thresholds.yaml`. Controllers and CFOs should be named in the workpaper header for the period — update the escalation contacts section of that file before each close.

---

## Escalation triggers

### Automatic (Exception Management Agent escalates without human prompting)

| Condition | Action |
|---|---|
| Item amount > $250,000 | Flag AUTO-INVESTIGATE; surface immediately to Controller regardless of age |
| Item amount > $100,000 OR item age > 30 days | Flag MATERIAL; include in Controller daily exception summary |
| Item reaches 61 days unresolved | Promote to At Risk; notify CFO |
| Item reaches 90 days unresolved | Promote to Critical; escalate to CFO + Audit Committee |
| Aggregate unresolved exceptions > $1,000,000 | Include CFO in exception summary |
| IC mismatch > $50,000 | Flag immediately; route to consolidation team and both entity Controllers |
| Outstanding check > 60 days | Flag for void/reissue decision; route to AP and Treasury |
| Deposit in transit > 5 days | Flag for treasury investigation; route to Treasury |
| 90+ day AR > 5% of total AR | Flag for collections review; route to AR Manager |
| Stale accrual > 45 days without reversal | Flag for reversal decision; route to Controller |

### Manual (Controller or CFO judgment call)

- Exception the agent classifies as Low/Trivial but has characteristics that warrant review (round numbers, recurring counterparty, unusual account combination)
- New account type with no established rec pattern
- Suspected duplicate entry or contra-entry that offsets a real difference
- Any item the Controller wants escalated regardless of dollar amount

---

## BD sequence

The Exception Management Agent runs BD3–BD4. Escalations should be resolved before BD5 so the Rec Package Agent can assemble a clean package.

| Business Day | Activity |
|---|---|
| BD3 morning | Exception Management Agent aggregates items from all BD1–BD3 recs; generates Exception Log v1 |
| BD3 COB | Controller reviews Exception Log; approves or challenges classifications; assigns owners to open items |
| BD4 morning | Exception Management Agent re-ages items; generates Exception Log v2 with updated dispositions |
| BD4 COB | All material items dispositioned (resolved, JE approved, or formally deferred with CFO sign-off) |
| BD5 | Rec Package Agent assembles final package; CFO Sign-off Matrix reflects BD4 COB state |

Items still open at BD5 do not block package assembly — they appear in the open items section of the CFO Sign-off Matrix with the assigned owner and expected resolution date.

---

## Controller daily exception summary

The Exception Management Agent produces this summary each morning BD3–BD4. It is the primary communication artifact between the agent and the Controller.

**Format:**

```
Subject: [ENTITY] BS Rec Exception Summary — YYYY-MM — BD[N]

Period: November 2026 | Entity: LuminaUS | Run: BD3 | As of: 2026-12-03 08:00 UTC

OPEN EXCEPTIONS: 4 items | $487,500 net unresolved

AUTO-INVESTIGATE (>$250K)
  ─ 230100 IC Receivable — LuminaUS vs LuminaEMEA mismatch | $320,000 | Age: 2d
    → Action required: Confirm with LuminaEMEA Controller whether Nov IC entry posted

MATERIAL (>$100K or >30d)
  ─ 120100 AR Trade — Customer X balance in dispute | $145,000 | Age: 47d
    → Action required: Collections status update; consider allowance JE if unresolved by BD5

LOW / MONITORING
  ─ 100100 Cash Operating — Check #4468 outstanding | $45,000 | Age: 72d
    → Action required: Void and reissue or confirm receipt with vendor
  ─ 100200 Cash MM — Unbooked dividends | $100,000 | Age: 0d
    → JE-002 proposed; Controller approval required (<$250K tier)

RESOLVED SINCE LAST RUN: 2 items | $62,300 cleared

Next run: BD4 morning. Items unresolved by BD4 COB will appear in CFO Sign-off Matrix.
```

---

## CFO escalation brief

Used when an item reaches Level 2 (CFO) or when the aggregate exception balance crosses $1M. The brief is concise — one page maximum.

**Format:**

```
Subject: [ENTITY] BS Rec Escalation — [Account] — [Amount] — Action Required by [Date]

Period: November 2026 | Entity: LuminaUS | Escalated: BD4 | CFO Decision Required by: BD5 COB

ITEM
  Account: 230100 IC Receivable — LuminaUS / LuminaEMEA
  Amount: $320,000
  Age: 3 days
  Classification: AUTO-INVESTIGATE (>$250K)

BACKGROUND
  LuminaUS books a $320,000 IC receivable from LuminaEMEA for Q4 content licensing fees
  invoiced 2026-11-28. LuminaEMEA has not posted the corresponding payable as of BD4.
  This is a timing difference, not an error — EMEA close runs one day behind US close.

PROPOSED RESOLUTION
  Option A (Recommended): Post a $320,000 IC payable in LuminaEMEA in the November period.
  LuminaEMEA Controller has confirmed the invoice is valid and will post today.
  Option B: Defer to December period with CFO notation. No financial statement impact
  if consolidated elimination runs correctly — risk is disclosure footnote timing.

ACTION REQUIRED
  [ ] Approve Option A — LuminaEMEA Controller posts today
  [ ] Approve Option B — defer with notation
  [ ] Other: _______________________________________________

  CFO sign-off: ___________________________ Date: _______________
```

---

## Audit Committee escalation

Triggered by items > $1M, Critical items (90d+) unresolved, or suspected misstatement.

The Audit Committee escalation is not templated here — it is a formal communication prepared by the CFO and reviewed by legal/external auditors before transmission. The agent surfaces the item to the CFO with the same brief format above, and flags it as requiring Audit Committee notification. The CFO determines timing and form of that communication.

What the agent provides:
- Full exception history with all prior dispositions
- All supporting documents referenced in the reconciliation
- Audit trail from the output envelope (sources, hashes, timestamps)
- Prior-period comparatives if the item has recurred

---

## Disposition options

Every open item must be closed with one of these dispositions before the period is locked:

| Disposition | When to use | Who approves |
|---|---|---|
| **Resolved — JE posted** | Book-side entry clears the item | Controller (or CFO if > $250K) |
| **Resolved — bank/vendor confirmed** | Third party confirms item is correct | Controller notation |
| **Resolved — reclassified** | Item belongs in a different account or period | Controller |
| **Deferred — management decision** | Item is real but resolution requires business decision | CFO sign-off required |
| **Deferred — timing** | Item will clear in the next period naturally | Controller notation with expected clear date |
| **Deferred — in dispute** | Item is disputed with a counterparty | Controller + CFO aware; legal loop-in if > $100K |
| **Written off** | Item is not collectable or correctable | CFO approval; document write-off rationale |
| **Escalated — Audit Committee** | Item > $1M or 90d+ unresolved | CFO + Audit Committee sign-off |

No item may be left with disposition "Open" in the final Rec Package. If an item cannot be resolved by BD5, it must be assigned one of the Deferred dispositions with an owner and expected resolution date.

---

## Recurring exception policy

If the same reconciling item (same account, same counterparty, same approximate amount) appears in three or more consecutive periods:

1. The Exception Management Agent flags it as a **recurring exception**.
2. The Controller must document the root cause and a remediation plan.
3. If the item is not resolved within two additional periods, it escalates to the CFO.
4. Recurring exceptions are highlighted in the CFO Sign-off Matrix with the recurrence count.

Recurring exceptions are frequently a process problem (missing automation, manual step that keeps getting skipped) rather than a one-time error. The remediation plan should address the process, not just the current-period balance.

---

## If the agent cannot classify an item

Occasionally a reconciling item will not fit any standard pattern — novel account combination, unusual transaction type, or data the agent cannot parse. In that case:

1. The agent flags the item as **UNCLASSIFIED** in the Exception Log.
2. The item is routed to the Controller for manual classification.
3. The Controller classifies it, enters the disposition, and notes the classification rationale.
4. If the item recurs, the Controller should request a system update to handle the new pattern.

Never mark an UNCLASSIFIED item as trivial without human review. Unclassified items frequently represent novel errors.
