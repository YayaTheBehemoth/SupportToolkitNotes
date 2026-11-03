# Project Overview

**Source:** https://github.com/YayaTheBehemoth/SupportToolkit

SupportToolkit is a modular .NET/C# console application for automating repetitive MSP support workflows, especially backup exception reviews and potential cross-tenant endpoint compliance investigations.

## Design goals

- Preserve both dotnet run development and a portable Windows executable for colleagues.
- Develop fixture-first with explicitly configured external instances.
- Keep monitoring read-only by default, and gate external writes separately.
- Use tenant-level aggregation, exception-first reporting and explicit unknown states.
- Separate providers for integrations, modules for workflows and Core for shared infrastructure.
- Keep public code and notes free of customer data and non-public access details.

## Implemented on `main` as of 2026-11-03 (release `v0.4.0`)

- **Local web UI** (`serve`, or `Start SupportToolkit.cmd`): Profile, Backup Aggregator and Zendesk bulk delete pages, Simple / Detailed view, loopback only with launch-token security. Same session rules as the console.
- **Zendesk bulk delete:** PKCE sign-in as the operator, search with completeness checks, exception-first preview, frozen ID list, typed confirmations, receipts. Console command `dev zendesk-bulk-delete` and a web page.
- BackupAggregator with Business Central monitoring scope (`v0.3.0`), in-progress wait, stray-tenant check and scoped tickets.
- Read-only diagnostics: `dev acronis-tenants`, `bc-subscriptions`, `customer-matching`, `acronis-usage`, `usage-compare`, plus fixture walkthroughs.
- Zendesk ticket creation with connection-scoped destinations, explicit submitter and requester, and destination-aware idempotency. Ticketing and deletion are separate connections with separate OAuth clients.
- Configuration: named profiles, provider connections, wizard, `configure import` for handed-over settings files, Windows Credential Manager; unrecognised settings are preserved on save.
- Acronis provider behind a restricted read-only transport; fail-closed discovery and explicit partial-review reporting.
- `LayeringTests` keep workflows free of console and UI code.
- Windows x64 CI and release workflow; zip launchers `Start SupportToolkit.cmd` and `Import config.cmd`.
- **BackupHealth was removed** in `v0.4.0`.

Validation: Acronis and Business Central production reads exercised; Zendesk ticketing and bulk delete only in personal trial/sandbox accounts; organizational Zendesk not validated. Operator-reported: 1,386 tests passing.

## On unmerged feature branches (not released)

- `feature/compliance-investigation`: synthetic Intune/Graph evidence, device investigation and a consolidated multi-tenant `compliance report`. No live Graph access. `main` merged in; 1,460 tests passing (operator-reported).

See [[Documentation Checkpoint - 2026-11-03]] for the evidence behind this list.

## Investigating / proposed

- Authorized live Graph/Intune access for compliance reporting.
- Lighthouse-independent aggregation.
- Stale-device monitoring.
- Defender triage and incident-response assistance.
- Onboarding, distribution and external-write safety improvements.
- Scheduling, persistence, hosting and multi-user operation: deliberately undecided. A local single-operator web UI exists since `v0.4.0` (November 3). See [[Strategy Checkpoint - 2026-10-20]] and [[Documentation Checkpoint - 2026-11-03]].

The source repository is authoritative for implementation and tests.

Related: [[Architecture and Decisions]], [[Ticketing and Configuration]], [[Roadmap]].
