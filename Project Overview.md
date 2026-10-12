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

## Implemented as of 2026-10-13

- BackupHealth and BackupAggregator operational modules.
- Acronis provider, fixture/production modes, resource normalization and tests.
- Direct CLI and interactive shell.
- Named profiles, provider connections and first-run wizard.
- Windows Credential Manager integration.
- Shared Zendesk ticket creation from BackupAggregator.
- Connection-scoped Zendesk destinations with profile-selected group routing.
- Destination-aware ticket idempotency and wizard connection reuse.
- Windows x64 release workflow.

Zendesk development-group routing was validated in a personal trial with synthetic data, not in an organizational instance.

## Investigating / proposed

- Direct Graph/Intune Compliance Monitor.
- Lighthouse-independent aggregation.
- Stale-device monitoring.
- Defender triage and incident-response assistance.
- Onboarding, distribution and external-write safety improvements.

The source repository is authoritative for implementation and tests.

Related: [[Architecture and Decisions]], [[Ticketing and Configuration]], [[Roadmap]].
