# Project Overview

**Source:** https://github.com/YayaTheBehemoth/SupportToolkit

SupportToolkit is a modular .NET/C# console application to automate repetitive MSP support workflows, especially backup exception reviews and cross-tenant endpoint compliance investigations.

## Design goals
- Preserve both `dotnet run` development and a portable Windows executable for colleagues.
- Fixture-first development and explicit production connections.
- Read-only monitoring by default; separate gated external writes.
- Tenant-level aggregation, exception-first reporting and explicit unknown states.
- Providers for integrations, modules for workflows, Core for shared infrastructure.

## Implemented as of 2026-10-12
BackupHealth, BackupAggregator, Acronis provider, fixture and production modes, CLI and interactive shell, configuration wizard and named profiles, Windows Credential Manager secrets, Zendesk ticketing for BackupAggregator, Windows x64 release workflow, tests.

## Investigating / proposed
Graph/Intune Compliance Monitor, independent Lighthouse-like aggregation, stale-device monitoring, Defender triage and incident-response assistance.

The source repository is authoritative for implementation status. Related: [[Architecture and Decisions]], [[Roadmap]].
