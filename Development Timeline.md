# Development Timeline

Retrospective chronology; dates approximate where no exact commit date was established.

| Period | Milestone |
|---|---|
| Late September 2026 | Compliance-monitoring concept; Lighthouse CSV limitations |
| Late September | Early Python automation ideas evolved into modular C#/.NET |
| Sep 30–Oct 6 | BackupHealth, Acronis API client, fixtures, provider/module separation |
| Oct 6–7 | BackupAggregator activity grouping, pagination, result-vs-state discovery |
| Oct 7–8 | Tenant accounting, classification and exception reporting |
| Oct 8–11 | CLI, configuration profiles, onboarding, portable EXE and Zendesk ticketing |
| Oct 10–12 | Graph/Lighthouse permission and availability roadblocks |
| Oct 11 | Endpoint monitoring and incident-response automation brainstorming |
| Oct 12 | Independent compliance aggregation idea and public Obsidian knowledge base |
| Oct 12–13 | Zendesk connection-scoped destinations, explicit group routing, resolver and provider tests |
| Oct 13 | Synthetic trial routing validation; wizard connection-reuse usability correction |
| Oct 13 | Zendesk destination routing merged into SupportToolkit main |
| Oct 15 | Business Central subscription discovery and unmerged, fixture-driven entitlement evaluation, selection and scoped backup inventory execution |

## Zendesk routing observations

One Zendesk connection can hold multiple named group destinations. Profiles select exactly one; the Zendesk provider sets group_id during ticket creation.

Local tests and a controlled synthetic trial submission established that the development queue receives correctly routed tickets.

A second synthetic ticket was observed in the operations group; the view alone does not establish how it was submitted.

A confusing wizard flow exposed the difference between a connection name and a destination name. Connection reuse is now an explicit, selectable path.

## Remaining limitations

Idempotency protection depends on the Zendesk retention window. Organizational Zendesk behavior has not been validated.

The public application README and local.config.example.json need to be updated for the new routing configuration.

## General theme

Production and trial observations repeatedly expose assumptions that local fixtures and initial implementations cannot prove.

Related: [[Ticketing and Configuration]], [[Development Journal - 2026-10-12]], [[Roadmap]].

## October 14, 2026

- Compliance: reported working fixture-driven exception list, setting-level evidence, collection diagnostics and investigation reports. Next: missing, conflicting and incomplete evidence scenarios; live authorization remains separate.
- BackupAggregator: reported fourth successful production comparison run. Monitoring entitlement emerged as a separate requirement from backup inventory.
- Scheduler configuration was investigated and rejected as an entitlement source; exploratory integration was not merged.
- Commercial subscription records identified as operational entitlement source; authorized read-only export and cross-system identity discovery planned. No production monitoring-scope integration implemented.

Related: [[Development Journal - 2026-10-14]].

## October 15, 2026

- Business Central UI discovery verified the subscription header/line relationship and a representative customer join. Active and blocked examples showed line presence and a blank end date are insufficient to establish entitlement.
- No supported API/export or cross-provider customer identity mechanism was confirmed.
- The unmerged monitoring-scope code branch now contains a fixture evaluator, a tenant selector and selected-tenant backup inventory execution.
- Local CLI output reported three selected healthy synthetic inventory reviews and two unresolved-scope cases; scope completeness correctly remained false.
- Existing production all-tenant inventory and ticketing remain unchanged. Next dependency: a read-only data-access and stable-reference discussion with BC development.

Related: [[Development Journal - 2026-10-15]].

## October 17–20, 2026 — Stabilization and release progression

- October 17: Core audits, provider hardening and configuration refactoring were integrated into `main` at `5705aa5`; **251 automated tests passing** was reported for this stabilized checkpoint.
- October 18–19: That baseline was released as Windows prerelease `v0.2.3`. Compliance and BC feature work remained on separate branches.
- October 19: `v0.2.4` was published with a focused Teams/SharePoint workload accounting correction and additional regression tests (`fc56e15`). GitHub comparison on October 20 confirms `v0.2.4` matches `main`.
- Organizational Zendesk integration and end-to-end production ticketing were still pending confirmation; Acronis production-data validation and personal Zendesk trial validation are separate evidence.

Related: [[Modules/BackupAggregator]], [[Strategy Checkpoint - 2026-10-20]].
