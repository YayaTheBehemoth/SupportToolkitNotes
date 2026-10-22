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

## Implemented on `main` as of 2026-10-22 (release `v0.2.4`)

- BackupHealth and BackupAggregator operational modules.
- Acronis provider behind a restricted read-only transport, fixture/production modes, resource normalization and tests.
- Provider contract and pagination hardening, including fail-closed zero-customer discovery and explicit partial-review reporting.
- Teams/SharePoint workload accounting rule (`v0.2.4`).
- Direct CLI, interactive shell and a read-only `dev acronis-tenants` diagnostic.
- Named profiles, provider connections and first-run wizard.
- Windows Credential Manager integration.
- Shared Zendesk ticket creation from BackupAggregator with connection-scoped destinations, profile-selected routing and destination-aware idempotency.
- The global `allowWrites` switch is retired. Writes are gated by an explicit submit operation, a resolved connection and destination, and interactive confirmation.
- Windows x64 CI and release workflow.

Zendesk routing was validated in a personal trial with synthetic data, not in an organizational instance.

## On unmerged feature branches (not released)

- `feature/compliance-investigation`: synthetic Intune/Graph evidence, device investigation and a consolidated multi-tenant `compliance report`. No live Graph access.
- `feature/business-central-monitoring-scope`: fixture-driven entitlement assessment and tenant selection, plus a read-only Business Central provider and an opt-in `scoped-review`. No validated production read, and production scoped review fails closed without an identity source.

See [[Documentation Checkpoint - 2026-10-22]] for the evidence behind this list.

## Investigating / proposed

- Authorized live Graph/Intune access for compliance reporting.
- Lighthouse-independent aggregation.
- Stale-device monitoring.
- Defender triage and incident-response assistance.
- Onboarding, distribution and external-write safety improvements.
- Operator interface, scheduling and persistence: deliberately undecided. See [[Strategy Checkpoint - 2026-10-20]].

The source repository is authoritative for implementation and tests.

Related: [[Architecture and Decisions]], [[Ticketing and Configuration]], [[Roadmap]].
