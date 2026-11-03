# Roadmap

Working backlog, not commitments.

Implementation status is grounded in the SupportToolkit source repository and tests. Discovery and architectural findings are tracked in these notes.

## Updated November 3, 2026 (`v0.4.0`)

- [x] Remove BackupHealth (`v0.4.0`).
- [x] Zendesk bulk deletion workflow, receipts and web page (`v0.4.0`). Sandbox-validated only.
- [ ] Organizational end-to-end validation of Zendesk bulk delete, starting with the supervised run on approved tickets (archived and recently closed).
- [ ] Record in Notes: the Zendesk audit-log entry for an API deletion, and the leftover-secret test result.
- [ ] Verify `--domain` search (`requester:*@domain`); replace it if Zendesk does not support it.
- [ ] Run the Backup Aggregator web page in production mode.
- [ ] Test the hidden-typing secret prompt of `configure import` interactively.
- [ ] Add `dev acronis-usage` and `dev usage-compare` to the `v0.4.0` README release row; reword the README project-status sentence about the operator interface.
- [ ] Remove leftover sandbox test organizations and people tagged `st_seed`.
- [ ] Refresh the CLI command catalogue (Claude project space) for `v0.4.0`.

## Added November 2, 2026

- [ ] Remove BackupHealth as a separate change: keep the Acronis resource-status provider and keep older configuration files loadable. See [[Modules/BackupHealth]].
- [ ] Zendesk bulk deletion: user checks before building (which user the existing OAuth client acts as and whether it can delete; whether agents may delete; audit-log entry for an API deletion in the trial; localhost redirect port; English wording of the 24-hour rule). See [[Modules/Zendesk Bulk Delete]].
- [ ] Zendesk bulk deletion: UI-independent workflow with fake-handler tests and a CLI path.
- [ ] Zendesk bulk deletion: WPF project for that module only, plus a second publish target in the release script.
- [ ] Zendesk bulk deletion: trial test, then one supervised production run on approved tickets that include both archived and recently closed tickets.
- [ ] Record a fresh `dotnet test` result for `v0.3.0`.
- [ ] Correct the source README introduction, which still says `Current release: v0.2.4` and lists BackupHealth as current (code-owner action).
- [ ] Review `feature/scope-visibility-wait-usage-probe` for merge, including the read-only allowlist addition.

## Priorities revised October 20, 2026

- [ ] Finish synthetic, multi-tenant, exception-first Compliance reporting with explicit collection failures.
- [ ] Seek authorized organizational Zendesk end-to-end validation; do not assume a personal test proves organizational behavior.
- [x] Confirm read-only BC subscription exposure and stable customer identity independently of feature development. (Released in `v0.3.0` with automatic name matching instead of a stable shared identifier; README states live read-only exercise.)
- [ ] Test synthetic Power Automate approval/rejection **as a proof of concept**; production deployment and scheduling are not prerequisites.
- [ ] Create a sanitized BackupAggregator process-engineering case study (before/after, technical decisions, measured evidence where available).
- [ ] Maintain one Obsidian/GitHub Notes source of truth; avoid manually duplicated Copilot documents.
- [x] Refresh the source README, example configuration and Notes to match `v0.2.4` and the feature branches (October 22). See [[Documentation Checkpoint - 2026-10-22]].
- [ ] Keep the CLI command catalogue (Claude project space) in step with command changes.
- [ ] ~~Record a fresh `dotnet test` result for `v0.2.4`~~ Superseded by the `v0.3.0` item above; the `251` figure belongs to `v0.2.3`.

**Deferred:** No universal deployment, hosting, operator UI or persistence architecture until concrete module needs and approved operating environments are understood. (November 2: one module-specific GUI decided, for Zendesk bulk deletion. This does not change the platform-level deferral.) Older items below are historical backlog entries and may be stale. See [[Strategy Checkpoint - 2026-10-20]].

## Earlier backlog

- [ ] Resolve authorized Graph/Intune access and tenant discovery.
- [ ] Validate one authorized tenant's compliance data and create synthetic fixtures.
- [ ] Investigate the historical Acronis 49-vs-44 count discrepancy.
- [ ] Verify a clean Windows ZIP release installation. (Release workflow already validates extraction and startup; a manual clean-machine check is still open.)
- [ ] Keep fixture schemas aligned with observed API behavior.
- [x] Update the SupportToolkit README and local.config.example.json for Zendesk destinations. (Done October 22, local edit; see [[Documentation Checkpoint - 2026-10-22]].)

## Ticketing

- [x] Keep ticketing a shared capability, not an operational module.
- [x] Add connection-scoped named Zendesk destinations.
- [x] Select exactly one Zendesk destination from each profile.
- [x] Reject missing or invalid group routing configuration.
- [x] Include instance/group identity in ticket idempotency.
- [x] Make existing-connection reuse explicit in the wizard.
- [x] Validate synthetic development-group routing in a personal trial.
- [ ] Add automated wizard interaction tests for connection reuse, destination selection and duplicate-instance warnings.
- [ ] Investigate durable duplicate-ticket prevention beyond the provider's idempotency window.
- [ ] Evaluate an additional safety confirmation for direct CLI ticket submission.
- [ ] Validate organizational Zendesk routing, triggers, automations and notifications before deployment.
- [ ] Document deployment permissions and operator handoff after organizational discovery.

## Compliance — discovery stage

- [ ] Tenant-grouped compliance exception normalization.
- [ ] Explicit unknown/incomplete reporting.
- [ ] Synthetic fixtures grounded in authorized discovery.
- [ ] Confirm minimum permissions and supported Graph/Intune access paths.

Compliance remains in discovery until operational and technical requirements are understood.

## Later candidates

- [ ] Clear fixture-mode onboarding documentation. (The README covers fixture mode; a walkthrough for non-developers is still open.)
- [ ] Stale endpoint monitoring following workflow discovery.
- [ ] Defender/Intune correlation, subject to access and operational validation.
- [ ] Incident-response assistance after procedure discovery.
- [ ] Reusable sanitized data-anonymization helpers when justified.

Related: [[Ticketing and Configuration]], [[Architecture and Decisions]], [[Compliance Monitor]].

## October 14 discovery milestones

### Compliance fixture development
- [x] Reported: exception-first device list using synthetic Graph responses.
- [x] Reported: setting-level evidence, collection diagnostics and readable investigation reports.
- [ ] Add missing-device scenarios.
- [ ] Add conflicting-evidence scenarios.
- [ ] Add incomplete-observation scenarios and assert explicit uncertainty.
- [ ] Independently establish authorized live Graph access and completeness.

### Backup monitoring entitlement — updated October 15
- [x] Reported: four production comparison runs for **unscoped** BackupAggregator inventory.
- [x] Identify commercial subscription evidence as the entitlement source; reject scheduled-report recipients as a proxy.
- [x] Inspect BC subscription header/line fields and verify one representative line-to-customer join.
- [x] Observe active and blocked subscription examples with blank end dates.
- [x] Implement a configurable monitoring-item evaluator and ordinary/exception synthetic fixtures.
- [x] Implement tenant selection using fictional resolved identities; keep scope uncertainty visible.
- [x] Add fixture-only scoped inventory execution with independent inventory/scope/overall completeness.
- [x] Reported: CLI demonstration of three selected tenant inventories and two unresolved cases, with zero external calls.
- [ ] Confirm final scoped-review automated test summary locally.
- [ ] Find an existing supported, authorized read-only API/service/export for the subscription records; the inspected table viewer did not offer export.
- [ ] Establish required authentication, permissions and explicit production access approval.
- [ ] Validate full entitlement semantics (status, dates, line duplication and contract transitions).
- [ ] Investigate stable external references/provisioning for BC-to-Acronis customer matching.
- [ ] Define operational policy for missing and conflicting monitoring scope.
- [ ] Integrate production BC acquisition and monitored-only execution **only after** identity and policy validation and explicit authorization.

**Branch (as of October 15):** `feature/business-central-monitoring-scope` remained unmerged at the time.

**Update November 2:** merged and released in `v0.3.0` (October 27). Production acquisition, conservative automatic name matching and the scoped review are now on `main`. Still open from the list above: full entitlement semantics across contract transitions, a stable external reference instead of name matching, and a source for which tenants are actually monitored. See [[Modules/BackupAggregator]].

Related: [[Development Journal - 2026-10-15]], [[Modules/BackupAggregator]].
