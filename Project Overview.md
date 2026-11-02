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

## Implemented on `main` as of 2026-11-02 (release `v0.3.0`, October 27)

- BackupAggregator operational module. BackupHealth is still in the code but **retirement was decided on November 2** (see [[Modules/BackupHealth]]).
- Acronis provider behind a restricted read-only transport, fixture/production modes, resource normalization and tests.
- Provider contract and pagination hardening, including fail-closed zero-customer discovery and explicit partial-review reporting.
- Teams/SharePoint workload accounting rule (`v0.2.4`).
- **Business Central monitoring scope (`v0.3.0`):** read-only Business Central provider with delegated sign-in, entitlement from subscription records, automatic name-based customer-to-tenant matching (nothing stored), the operator's own tenant always included, a scope completeness report and scoped tickets. The interactive console offers only the scoped review; the all-tenant review remains a command.
- Explicit Zendesk ticket submitter and requester per connection (`v0.3.0`).
- Direct CLI, interactive shell and read-only `dev` diagnostics (`acronis-tenants`, `bc-subscriptions`, `customer-matching`, plus fixture walkthroughs).
- Named profiles, provider connections and first-run wizard.
- Windows Credential Manager integration.
- Shared Zendesk ticket creation from BackupAggregator with connection-scoped destinations, profile-selected routing and destination-aware idempotency.
- The global `allowWrites` switch is retired. Writes are gated by an explicit submit operation, a resolved connection and destination, and interactive confirmation.
- Windows x64 CI and release workflow.

Zendesk routing was validated in a personal trial with synthetic data, not in an organizational instance. The source README states that Business Central reads and matching were exercised read-only against a live environment and that the scoped review has had limited end-to-end validation; Notes hold no detailed record of those runs.

## On unmerged feature branches (not released)

- `feature/compliance-investigation`: synthetic Intune/Graph evidence, device investigation and a consolidated multi-tenant `compliance report`. No live Graph access. No functional change since October 18.
- `feature/scope-visibility-wait-usage-probe` (October 29–30): waiting for in-progress backups before a review is final, a stray-tenant check in scoped tickets, an Acronis usage probe (`dev acronis-usage`) and a Business Central vs Acronis usage comparison (`dev usage-compare`). Read-only. See [[Modules/BackupAggregator]].

## Designed, not built

- Zendesk bulk ticket deletion with a WPF GUI for that module only. See [[Modules/Zendesk Bulk Delete]].

See [[Documentation Checkpoint - 2026-11-02]] for the evidence behind this list.

## Investigating / proposed

- Authorized live Graph/Intune access for compliance reporting.
- Lighthouse-independent aggregation.
- Stale-device monitoring.
- Defender triage and incident-response assistance.
- Onboarding, distribution and external-write safety improvements.
- Operator interface, scheduling and persistence: deliberately undecided at platform level. One exception is decided: a GUI for the Zendesk deletion module only (November 2). See [[Strategy Checkpoint - 2026-10-20]].

The source repository is authoritative for implementation and tests.

Related: [[Architecture and Decisions]], [[Ticketing and Configuration]], [[Roadmap]].
