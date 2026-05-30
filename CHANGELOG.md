# Changelog

All notable changes to the Balance Sheet Reconciliation Agent system are documented here.  
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

---

## [Unreleased]

### Planned
- Account Reconciliation agent implementation
- Exception Management agent implementation
- Rec Package agent implementation
- Synthetic demo data for Lumina Streaming Co. (November 2026 close)
- End-to-end close cycle test run

---

## [0.3.0] — 2026-05-30

### Added
- `GUARDRAILS.md` — hard stops, never-do list, output boundaries, input validation, and escalation chain
- `docs/privacy-policy.md` — data classification, retention schedule, access controls, and incident response
- `docs/human-in-the-loop-policy.md` — approval requirements, agent pause triggers, workpaper sign-off rules, and audit trail spec

---

## [0.2.0] — 2026-05-01

### Added
- `CLAUDE.md` — full system architecture, agent roles, account coverage, materiality thresholds, and operating rules
- `README.md` — project overview and quick-start guide
- Repo structure scaffolded: `agents/`, `config/`, `data/`, `docs/`, `outputs/`, `scripts/`, `skills/`, `workpapers/`
- Three agent stubs: `agents/account-reconciliation`, `agents/exception-management`, `agents/rec-package`
- `docs/BS_Recon_Agent_Documentation.docx` — detailed system documentation

---

## [0.1.0] — 2026-04-15

### Added
- Initial repository setup
- Project concept: three-agent AI system for month-end balance sheet reconciliations
- Demo company defined: Lumina Streaming Co. (LuminaUS, LuminaEMEA, LuminaAPAC)
- Account coverage defined: Bank, AR, Prepaids, Fixed Assets, AP, Accruals, Deferred Revenue, Intercompany, Long-Term Debt

---

*Dates reflect approximate milestones. Versions follow [Semantic Versioning](https://semver.org/).*
