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

### Backup monitoring entitlement
- [x] Reported: four production validation comparisons for existing BackupAggregator reporting.
- [x] Identify commercial subscription management as operational entitlement source.
- [x] Reject scheduled-report configuration as monitoring entitlement authority.
- [ ] Inspect authorized, read-only subscription export and sanitize findings.
- [ ] Determine service records, status/effective-date rules and stable customer references.
- [ ] Investigate existing cross-system identity/provisioning references.
- [ ] Decide whether export-based integration is adequate or authorized API access is needed.
- [ ] Implement a narrowly scoped fixture-backed entitlement provider **only after** requirements discovery.
- [ ] Ensure unknown/unmatched scope cannot silently exclude customers.

Related: [[Development Journal - 2026-10-14]].
