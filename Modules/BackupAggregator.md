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
