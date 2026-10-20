# October 20, 2026 — Strategy checkpoint: reporting, orchestration, and deployment

**Status:** Design decisions and roadmap guidance; not authorization to change the SupportToolkit code repository.

## Why this checkpoint matters

A Power Automate approval proof of concept began as a narrow, valuable question: can an operator review and approve a generated BackupAggregator ticket draft? Discussion expanded into scheduling, hosted runtimes, nontechnical user interfaces, central storage, deployment and long-running operational infrastructure. This exposed a planning trap: a choice about one workflow became a hypothetical platform-wide deployment decision before the modules' operational requirements were understood.

**Decision: Defer a universal SupportToolkit deployment architecture.** Keep the approval POC as a small synthetic, independently validated learning exercise. It does not require committing to a production hosting model or implementing approval-gated Zendesk submission now.

The deployment requirements differ across modules:
- BackupAggregator could ultimately become an unattended scheduled process producing reviewable drafts.
- Compliance is primarily a reporting and investigation MVP; its actual operator interaction model is not settled.
- BC monitoring-scope is data acquisition/entitlement discovery rather than a separate workload deployment requirement.
- A contemplated interactive PTO/report calculation would have different data-entry and output requirements.
- The current CLI suits the development workflow but non-developer trial use exposed usability problems; this is evidence to revisit operator UX, not a mandate to build a web platform immediately.

Preserve fixture development, module/provider boundaries, explicit external-write safeguards, and the existing `dotnet run`/`dotnet test` workflow. Reconsider hosting, orchestration, persistence and UI only when a specific validated workflow needs them.

## Current engineering snapshot

- User-reported public Windows release: **v0.2.3**, from stabilized main checkpoint `5705aa5`, previously validated with **251** reported tests.
- Code repository main subsequently received an additional Microsoft 365 Teams/SharePoint backup accounting change (`fc56e15`). Do not conflate that change with the published tag.
- BackupAggregator: production Acronis collection tested; ticketing routed successfully in a personal Zendesk trial. Authorized organizational Zendesk end-to-end validation was **pending** at this checkpoint.
- Compliance development branch: fixture-driven exception list, setting-level evidence, diagnostics, investigation, and a consolidated synthetic multi-tenant `compliance report`. Live Graph access/completeness and production scope remain unverified.
- BC monitoring-scope branch: synthetic subscription header/line assessment, conservative tenant selection and scoped inventory execution. Read-only production exposure, entitlement semantics and BC-to-provider identity mapping remain unverified.
- Compliance and BC feature branches remain separate from main. Operator-reported merge-validation test totals: 307 (Compliance) and 305 (BC) before subsequent development.
- Power Automate: workflow approval is a POC/design track, **not** a production feature or selected hosting strategy.

## Refined priorities

1. Finish the narrow Compliance reporting MVP with explicit failed/incomplete tenant visibility.
2. Pursue authorized company Zendesk validation and minimal BC read-only access feasibility without blocking fixture development.
3. Time-box a small synthetic Power Automate approval POC. Do not design platform-wide deployment to enable the POC.
4. Develop sanitized **process-engineering case studies**, showing the operational problem, workflow redesign, technical decisions and validated outcomes. SupportToolkit is a portfolio for automation engineering, not a reason to add arbitrary modules.
5. Keep SupportToolkitNotes as the one curated knowledge base. ChatGPT and Microsoft 365 Copilot should consume derived context, not maintain competing versions. A generated Copilot knowledge export is possible; automatic ingestion and work-tenant governance remain to be verified.

## Evidence and governance

Source code and tests in [SupportToolkit](https://github.com/YayaTheBehemoth/SupportToolkit) remain authoritative for implementation. This Notes repository is authoritative for sanitized decisions, observations, requirements and validation. Synthetic providers do not prove live API contracts, authorization, or completeness.

**Git ownership:** The user performs or specifically authorizes each SupportToolkit code-repository Git action. Focused sanitized documentation maintenance in SupportToolkitNotes is permitted by the separate documentation workflow.

Related: [[Roadmap]], [[Architecture and Decisions]], [[Modules/Compliance Monitor]], [[Public Documentation Safety]].
