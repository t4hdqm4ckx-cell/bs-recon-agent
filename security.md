# Security Policy

**System:** Balance Sheet Reconciliation Agent  
**Applies to:** Account Reconciliation · Exception Management · Rec Package  
**Effective:** 2026-05-30

---

## Scope

This policy defines the security controls, access standards, and incident procedures for the Balance Sheet Reconciliation Agent system and all data it processes for Lumina Streaming Co. (LuminaUS, LuminaEMEA, LuminaAPAC).

---

## Access Control

### Role-Based Access

| Role | Permitted Actions |
|---|---|
| Controller | Approve JEs, clear reconciling items, review all workpapers and outputs |
| CFO | Release rec package, approve threshold overrides, receive critical escalations |
| Internal Auditor | Read-only access to finalized workpapers and rec packages |
| External Auditor | Read-only access to released rec packages (provided by Controller) |
| System Administrator | Config management, audit trail access, retention purges |
| Agent (automated) | Read source data, write draft outputs, log to audit trail — no approval authority |

### Principle of Least Privilege

- Each role is granted only the access required to perform its function.
- The agent itself holds no approval authority and cannot elevate its own permissions.
- Sub-ledger detail is accessible only to the Controller and external auditors during active audit engagements.
- Access rights are reviewed at the start of each close cycle.

---

## Authentication & Authorization

- All human access to the system requires authenticated login; shared or anonymous credentials are prohibited.
- Agent API calls to the Anthropic Claude endpoint are authenticated via environment-scoped API keys stored in the secrets manager — never hardcoded in config files or committed to the repository.
- API keys are rotated quarterly and immediately upon any suspected exposure.
- All access events (human and agent) are logged in the audit trail with timestamp, actor, and action.

---

## Data Security

| Data state | Control |
|---|---|
| At rest | Encrypted at the storage layer (AES-256 or equivalent) |
| In transit | TLS 1.2 minimum for all API calls and file transfers |
| In use | Processed in memory only; no plaintext writes to unprotected temp storage |
| Sub-ledger detail | Stored in restricted workpapers; excluded from all executive-level outputs |

Source input files in `data/` are purged 90 days after close per the [Privacy Policy](docs/privacy-policy.md).

---

## Secrets Management

- API keys, database credentials, and service tokens are stored in the designated secrets manager.
- No secrets appear in `config/`, source files, or commit history.
- `.gitignore` is configured to block common secret file patterns (`.env`, `*.key`, `*.pem`, credentials files).
- If a secret is accidentally committed, it is treated as compromised: rotate immediately, audit access logs, and log the incident.

---

## Dependency & Supply Chain

- Third-party dependencies are pinned to specific versions in the project lockfile.
- Dependencies are reviewed for known vulnerabilities before each close cycle run.
- Updates to the Anthropic SDK or other core dependencies require administrator approval and a regression test before deployment.

---

## Audit Trail Integrity

- The audit trail is append-only; no record may be modified or deleted after it is written.
- Audit logs are stored separately from agent outputs and are accessible only to the Controller, CFO, and System Administrator.
- Log integrity is verified via checksums on each close cycle.
- Audit trail retention: 7 years (aligned with SOX requirements).

---

## Vulnerability Reporting

If you discover a security vulnerability in this system:

1. **Do not** open a public GitHub issue.
2. Report privately to the system owner via direct message or email.
3. Include: description, reproduction steps, affected components, and potential impact.
4. A response will be provided within **48 hours**; a remediation plan within **7 days** for critical findings.

---

## Incident Response

| Severity | Definition | Response SLA |
|---|---|---|
| Critical | Unauthorized access to Restricted data; secret exposed | Immediate — within 1 hour |
| High | Unauthorized access to Confidential data; audit trail tampered | Within 4 hours |
| Medium | Failed login attempts; config change without authorization | Within 24 hours |
| Low | Anomalous agent behavior; unexpected output format | Within 48 hours |

**Steps for any incident:**

1. Contain — isolate affected components; suspend agent runs if data integrity is in doubt.
2. Notify — Controller for High/Medium; CFO and Legal for Critical.
3. Investigate — review audit trail, access logs, and agent run history.
4. Remediate — patch, rotate credentials, restore from clean state as needed.
5. Document — record timeline, root cause, and corrective actions in the audit trail.
6. Review — update guardrails or controls if the incident reveals a gap.

---

## Policy Review

This policy is reviewed annually and after any security incident or material change to the agent system, infrastructure, or data sources.

| Role | Responsibility |
|---|---|
| System Administrator | Technical controls, secrets rotation, dependency review |
| Controller | Access approvals, audit trail oversight |
| CFO | Policy owner, critical incident notification, sign-off on material changes |

---

*Read alongside [GUARDRAILS.md](GUARDRAILS.md), [docs/privacy-policy.md](docs/privacy-policy.md), and [approval_router.md](approval_router.md).*
