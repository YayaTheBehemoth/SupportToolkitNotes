# SupportToolkit Notes

Public development knowledge base for [SupportToolkit](https://github.com/YayaTheBehemoth/SupportToolkit).

Notes on architecture, modules, API discovery, development history and roadmap.

## Latest decisions

Start with [[Documentation Checkpoint - 2026-11-03]] for the current state (`v0.4.0`): the local web UI, Zendesk bulk delete and the removal of BackupHealth. [[Documentation Checkpoint - 2026-11-02]] records the decisions that led there. [[Strategy Checkpoint - 2026-10-20]] still holds the priorities and the decision to defer a universal deployment architecture. Older notes may describe historical states.

## Where things are

| Topic | Page |
|---|---|
| What the project is | [[Project Overview]] |
| Design decisions and reasons | [[Architecture and Decisions]] |
| Zendesk routing and configuration | [[Ticketing and Configuration]] |
| Backup inventory review | [[Modules/BackupAggregator]] |
| Zendesk bulk ticket deletion | [[Modules/Zendesk Bulk Delete]] |
| Backup health evaluation (removed in `v0.4.0`) | [[Modules/BackupHealth]] |
| Compliance investigation (synthetic) | [[Modules/Compliance Monitor]] |
| Endpoint triage ideas | [[Modules/Endpoint Triage Ideas]] |
| Backlog | [[Roadmap]] |
| History | [[Development Timeline]] |
| What may be published here | [[Public Documentation Safety]] |

Implementation and tests in the code repository are authoritative. These notes record sanitized decisions, observations and validation boundaries.
