# Roadmap

Working backlog, not commitments.

Implementation status is grounded in the SupportToolkit source repository and tests. Discovery and architectural findings are tracked in these notes.

## Current

- [ ] Resolve authorized Graph/Intune access and tenant discovery.
- [ ] Validate one authorized tenant's compliance data and create synthetic fixtures.
- [ ] Investigate the historical Acronis 49-vs-44 count discrepancy.
- [ ] Verify a clean Windows ZIP release installation.
- [ ] Keep fixture schemas aligned with observed API behavior.
- [ ] Update the SupportToolkit README and local.config.example.json for Zendesk destinations.

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

- [ ] Clear fixture-mode onboarding documentation.
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

**Branch:** `feature/business-central-monitoring-scope` remains unmerged; existing production review/ticketing behavior is unchanged.

Related: [[Development Journal - 2026-10-15]], [[Modules/BackupAggregator]].
