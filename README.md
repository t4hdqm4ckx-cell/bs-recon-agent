# Balance Sheet Reconciliation Agent System

Three-agent AI system for month-end balance sheet account reconciliations.
Built on Anthropic Claude as a consulting portfolio piece for Controllers,
CFOs, and Audit teams.

Every balance sheet account on the trial balance must be supported by an
underlying schedule, document, or third-party confirmation. This system
automates the analytical work — tying GL balances to support, identifying
reconciling items, aging exceptions, and assembling the monthly rec package
— while keeping humans in the loop for every approval.

## Agents

| Agent | Status |
|---|---|
| Account Reconciliation | Pending |
| Exception Management | Pending |
| Rec Package | Pending |

## Account coverage

Bank · AR · Prepaids · Fixed Assets · AP · Accruals · Deferred Revenue · Intercompany · Long-Term Debt

## Quick start

```bash
cd bs-recon-agent
claude
```

## Related

- [close-system](https://github.com/t4hdqm4ckx-cell/close-system) — month-end close automation
- [headcount-agent](https://github.com/t4hdqm4ckx-cell/headcount-agent) — people cost analysis
