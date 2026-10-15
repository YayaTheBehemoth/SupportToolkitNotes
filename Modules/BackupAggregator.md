# BackupAggregator

**Status: implemented.**

## Motivation and early discovery
Replace manual review of numerous Acronis backup PDFs with tenant-grouped exceptions. Early experiments grouped multiple activities per resource and selected the latest relevant outcome. Activity lifecycle state `completed` is not equivalent to success. Pagination, stable identity and timestamps are essential.

An early production probe considered 102 activities across 48 resource groups. This was an **earlier activity-based experiment**, not necessarily the current collection path.

## Current implementation (source README, 2026-10-12)
- Discover customer tenants; review one or all.
- Microsoft 365 inventory via Acronis Resource Manager; device inventory via `AllMachines`.
- Pagination, normalization, deduplication and tenant aggregation.
- Suppress healthy detail, show exceptions and incomplete tenant reviews.

Classification: **Healthy**, **NeedsReview**, **Unknown**.

Invariant: `Resources checked = Healthy + NeedsReview + Unknown`.

Acronis resource objects do not necessarily correspond one-to-one with physical devices.

```cmd
dotnet run --project src\SupportToolkit -- backup-aggregator
dotnet run --project src\SupportToolkit -- backup-aggregator inventory-review --all
```

## Unresolved discrepancy
One historical tool result reported 49 healthy resources while a portal view reported 44. Root cause **not established**. Check filters, inventory scope, resource types, pagination, deduplication and time semantics.

Related: [[BackupHealth]], [[Ticketing and Configuration]].

## Monitoring entitlement discovery — 2026-10-14

**Reported validation:** four successful production comparison runs against existing tenant-level backup reports. This supports confidence in the tested inventory/reporting behavior, not in entitlement filtering.

### Separate responsibilities

1. **Commercial entitlement:** Which customers purchase managed monitoring?
2. **Backup inventory:** Which resources exist and what is their health?
3. **Operational reporting:** How should eligible results be presented and acted upon?

The existing subscription-management system has been identified as the operational source used to check purchased services. Its relevant record structure, status semantics, effective dates and integration access remain unverified.

### Rejected entitlement proxy

Exploration of scheduled backup-report configuration showed that a scheduled report is not sufficient evidence of a monitoring entitlement. Report delivery settings must not determine which customers are monitored; such a dependency would recreate the reporting process that this module aims to replace. The experimental integration was not merged.

### Identity and integration unknowns

- No stable common customer identifier across the commercial and backup systems has been confirmed.
- Investigate existing references and provisioning relationships before designing reconciliation.
- Do not introduce a permanent manually maintained mapping as the default solution.
- A read-only spreadsheet export is planned for initial discovery; neither export suitability nor API access is established.
- A fixture-backed monitoring-scope provider is a potential later implementation, not an implemented feature.
- Unknown or unmatched scope must not silently exclude a customer.

**Next step:** Inspect authorized, sanitized field structure and entitlement rules; preserve existing BackupAggregator functionality.

Related: [[Architecture and Decisions]], [[Development Journal - 2026-10-14]].
