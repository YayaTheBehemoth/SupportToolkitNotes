# BackupAggregator

**Status:** Backup inventory and exception reporting implemented. Monitoring-scope integration is fixture-only on an unmerged feature branch (2026-10-15).

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

## Monitoring entitlement discovery and fixture development — October 14–15, 2026

**Context:** Four successful production comparisons were reported for the existing all-tenant backup inventory and exception report. These do **not** validate entitlement filtering or Business Central connectivity.

### Business Central discovery

Commercial subscription information was identified as the authority to investigate for purchased backup monitoring. Scheduled backup-report delivery settings were rejected as an entitlement proxy: they describe the process being replaced, not the contract.

Authorized Business Central UI inspection located related subscription **header** and **line** records. Headers expose a subscription reference, customer reference, status and effective dates; lines expose their subscription reference and service item. A monitoring service appeared across multiple subscriptions, and one subscription reference was manually traced to the expected customer through the header. Real service codes, table IDs, customer identities and investigation access paths are omitted here.

One active example had a populated start date with no end date. One blocked example still contained the monitoring-service line and likewise had no end date. This demonstrates that service-line presence or an empty end date alone is insufficient to establish current monitoring responsibility. Full status/date rules remain unverified.

The inspected table viewer provided no practical bulk export. No applicable published service was identified via the investigated web-services list; a supported API/export could still exist.

### Fixture-driven implementation — unmerged code branch

On `feature/business-central-monitoring-scope`, SupportToolkit now has:

- Normalized subscription header/line records, a configurable monitoring-item identifier and a conservative entitlement evaluator.
- Synthetic fixtures favoring ordinary outcomes, with adverse cases retained as focused tests.
- A selector consuming synthetic resolved Business Central customer-to-Acronis tenant relationships, with no display-name matching or claimed production mapping mechanism.
- Explicit selected, needs-review, unclassified and unmatched commercial-evidence outcomes.
- An optional selected-tenant inventory service method that reuses existing bounded-concurrency, failure-isolated inventory execution.
- Separate completeness results for **selected inventory**, **monitoring scope** and **overall execution**.
- A Business Central read-only HTTP boundary with no real authorized SMARTsubscription API endpoint configured.

Fixture developer commands:

~~~cmd
dotnet run --project src/SupportToolkit -- dev monitoring-scope
dotnet run --project src/SupportToolkit -- dev scoped-backup-review
~~~

**Reported local CLI validation (October 15):** Five fictional customers yielded three selected inventory candidates, one needs-review case and one unclassified case. The scoped inventory reviewed three synthetic healthy device resources with zero external requests. Inventory completion was **YES**, scope completion **NO**, overall completion **NO**. The final increment's full automated test result was not supplied in the conversation.

**Production boundary:** Existing unscoped inventory collection and ticketing are unchanged. Fixture results do not authorize filtering any real customer out of monitoring.

### Remaining dependencies

- Supported, approved read-only access to the relevant BC subscription header/line records and appropriate authentication.
- Validation of status, dates, duplicate service lines and commercial responsibility rules.
- Stable customer identity reconciliation across BC and Acronis; no shared identifier has been confirmed.
- Agreed operational behavior for missing/ambiguous scope and for incomplete runs before enabling scoped production execution.

**Next handoff:** Ask BC development whether these records are already exposed through a supported authenticated read-only API, service or export, and whether a stable external customer reference is already available.

Related: [[Development Journal - 2026-10-15]], [[Architecture and Decisions]], [[Roadmap]].
