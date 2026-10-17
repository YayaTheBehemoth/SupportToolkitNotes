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

## Teams/Groups workload accounting — October 16, 2026

**Status:** Bugfix and regression tests observed on SupportToolkit `main` (2026-10-16), with local and production comparison reported passing. Release status not inferred.

### Observed discrepancy

Two anonymized customer comparisons exposed undercounting after Microsoft 365 normalization:

| Anonymized case | Acronis protected workloads | Previous SupportToolkit count | Team-associated Groups in Acronis | Corrected SupportToolkit count |
| --- | ---: | ---: | ---: | ---: |
| Small mixed-workload tenant | 16 | 15 | 1 | 16 |
| Larger Microsoft 365 tenant | 136 | 131 | 5 | 136 |

- The larger tenant returned **136 unique Microsoft 365 resource records** and no device resources; the old normalizer reduced the report to 131.
- Acronis's **All Groups → Team** column showed exactly five linked Groups. An independent second tenant showed exactly one, matching its discrepancy.
- Read-only, debug-only diagnostics confirmed five `GROUP` resources whose logical identity matched a `TEAM`, and five unmapped `teamName` string fields. Matching Groups had top-level `hasProtections=true` in the observed diagnostic sample; protection flags were **not** adopted as the consolidation criterion because they do not establish workload identity.
- The `All devices` inventory need not list Microsoft 365 cloud workloads. In the mixed-workload case, the single discovered device was consistent with an existing, successfully protected server-backed NAS-file plan; no separate NAS-specific bug was established.

### Root cause and targeted fix

The previous normalizer collapsed a `GROUP` and `TEAM` whenever their `GROUP.<key>` / `TEAM.<key>` identities matched. This conflated *Team association* with *a duplicate protected workload*.

Changes reported locally:

1. Map the observed `teamName` response property to nullable `AcronisMicrosoft365ResourceDto.TeamName`.
2. Preserve a matching `GROUP` as an independent monitored workload when `teamName` is populated.
3. Retain the prior consolidation and health-evidence merge for matching Groups with missing/blank `teamName`, along with upstream provider deduplication of genuinely repeated resource identities.
4. Add synthetic regression tests for the one-Group and five-Group count patterns, independently failing linked Groups, absent protection evidence, and legacy consolidation.
5. Remove the temporary relationship-discovery instrumentation after the contract was established.

**Validation reported:** `dotnet build` and `dotnet test` green after the change and cleanup. Authorized read-only production reviews returned the expected **16** and **136** counts. The operator reported **no newly unexpected `Unclassified` or `Require attention` outcomes**. No broader estate-wide equivalence is claimed.

### Follow-up kept separate

Conduct a focused audit of Acronis DTOs and vendor semantics *after* this fix is finalized. Candidate topics: `basicKinds` mixed protection flags, `hasCompositeProtections`, identifier/deduplication semantics, nullable paging and malformed responses, and whether undocumented v1 endpoints expose richer association metadata. Do not transfer fields from the public v4 Resource Management API into the console-facing v1 DTOs without payload verification.

Keep real tenant identifiers, resource names, credentials, raw responses, and internal access details out of public documentation.

## Acronis provider contract audit — October 16, 2026

**Working branch:** `audit/acronis-provider-contracts` (local; not presumed pushed). The operator reports `dotnet test` passing after each of the following incremental changes.

### Contract and completeness hardening

- Synthetic Microsoft 365 JSON contract tests cover explicitly empty/null collections, `teamName` mapping, protection evidence, and opaque paging cursor deserialization.
- A provider safeguard distinguishes an *absent* `data`/`items` collection property from a present but explicit `null` or empty array. Former now fails a review; latter retains observed valid-empty behavior.
- Repeated continuation cursors now fail promptly rather than reaching the existing 500-page cap; no cursor values are placed into diagnostics.
- These are locally tested changes on the audit branch, **not yet claimed merged or released**.

### Temporary pagination probe — production observations

An authorized, explicitly enabled, read-only probe of one anonymized M365 tenant inspected **six leaf groups**, **10 page responses**, and **three multi-page groups**. All six terminal pages had `paging.cursors` present and no `after` token. Aggregate M365 collection: **183 raw rows, 136 unique resources**.

Within one sampled three-page leaf group (30 + 30 + 12 = **72** rows), three continuation styles were compared:

| Continuation style | Observation |
| --- | --- |
| Existing `?after=<cursor>` | Three pages, 72 rows |
| `?limit=30&after=<cursor>` | Three pages, 72 rows; same identity multiset and order as existing |
| `?limit=30&order=asc(last_task_status)&after=<cursor>` | HTTP 400; shadow comparison failure did not affect authoritative report |

**Decision:** Preserve the existing `?after=` continuation behavior. General Acronis pagination guidance should not be assumed to describe this console-facing v1 resource-manager endpoint. Preserve permissive final-page `paging` handling for now: all six observed terminal responses contained it, but that is insufficient evidence to require it for every tenant. The alternative-query comparison was performed on **one** group, not the whole tenant.

**Pagination cleanup:** The operator reported all tests green after removing the temporary probe and probe-only tests; permanent completeness checks and cursor-cycle guards remain.

### Duplicate-record diagnostic (October 17)

A read-only, observation-only diagnostic on the already fetched anonymized M365 inventory reported 183 raw rows and 136 unique identities. There were 47 duplicate-identity groups, 47 surplus rows, and no rows without a usable identity. All compared fields matched across duplicate instances: zero groups with differing health evidence, Team-association fields, timestamps, or names. The diagnostic compared selected DTO fields, not entire raw JSON responses.

**Decision:** Keep last-representation-wins deduplication for now; no disagreement was observed to justify a merge policy. This is evidence for this sample only, not a guarantee across customers or API revisions. The operator reported green tests after removing the temporary probe and its tests.

**Next:** Investigate the meanings of `hasProtections`, `basicKinds`, and `hasCompositeProtections` before changing classification. Also review leaf metadata and other provider DTOs.



### Protection-state probe — October 17, 2026

An explicitly enabled, read-only diagnostic examined two anonymized production tenant inventories using data already fetched by the M365 provider; it did not change classification or collection requests.

| Observed case | M365 resources | `hasProtections=true` | All `basicKinds` flags true | `hasCompositeProtections=false` | `hasCompositeProtections` absent | Mixed or contradictory protection signals |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| Larger all-healthy M365 tenant | 136 | 136 | 136 | 72 | 64 | 0 |
| Mixed protected/unprotected inventory tenant | 49 | 49 | 49 | 37 | 12 | 0 |

Both samples exhibited successful M365 task evidence for every observed M365 resource. The `hasCompositeProtections=false` values appeared on USER resources even when other protection and task indicators were positive. This supports **not** treating the composite flag as a general protected/unprotected indicator. It does **not** establish its full vendor semantics or validate existing `protection_conflict` handling for partially protected M365 objects.

The second tenant's portal displayed **52 protected workloads**. SupportToolkit reported **54 resources checked: 52 healthy, 2 requiring attention (displayed as Not protected), 0 unclassified**. The M365 probe found 49 M365 resources with successful and positive protection evidence. The extra two unprotected entries plausibly originate from device inventory, but the diagnostic did not confirm their source. The operator noted that this tenant uses an unusual backup setup; the exact setup is not established.

**Decision:** Leave the M365 normalizer unchanged pending genuine evidence of mixed or negative M365 protection signals. Keep the distinction between portal *protected workloads* and SupportToolkit *all inventoried resources*. Remove the temporary protection probe and its probe-only tests after local validation. The two unusual unprotected resources are **parked for operator follow-up with a colleague**, not a product defect or a trigger to change classification. Their configuration and ownership have not been confirmed. The desired operational behavior is to surface potentially uncommunicated backup changes for investigation; current CLI reporting alone does not create a lasting shared support record, which will require the separate Zendesk workflow.

### Device inventory contract audit — October 17, 2026

The device endpoint (`/bc/api/resource_manager/v1/epm/resources`) has its own DTO and pagination contract; it must not be assumed equivalent to the console-facing Microsoft 365 endpoint.

- The operator reported green local build and test results after adding a **missing/explicit-null `items` collection guard** (an explicit empty array stays valid). Unlike M365, `items:null` has not been established as a valid device response.
- Added focused, locally green tests for immediate repeated continuation cursors, longer cursor cycles, and successful synthetic three-page traversal. Existing `after`-only continuation remains unchanged.
- An opt-in read-only diagnostic examined one production tenant: **one device page, zero device rows, `paging.cursors` present and no `after` token**. The same combined tenant report checked 103 resources, sourced from other backup inventory. This run **did not validate multi-page device behavior**.
- Operational discovery indicates that no known customer has anything approaching 30 Acronis device resources, so a useful real multi-page device case is not currently available. No artificial production test or customer hunting is justified.

**Decision:** Keep the working device continuation query, collection guard, and cycle protection; document multi-page device pagination as **fixture-tested only, not production-verified**. Remove the temporary device pagination probe and probe-specific tests after cleanup. Do not restore the older provider from a commit that predates the permanent device hardening.

**Open:** Verify actual device status/agent DTO field types and operational meanings, investigate nullable paging only when evidence requires it, and review the remaining tenant/group contract assumptions. The unusual unprotected resources from an earlier tenant review remain a separate operator follow-up.

### Device status evidence — October 17, 2026

An authorized, opt-in read-only diagnostic reused the normal device inventory of one anonymized tenant; no extra API calls or classification changes.

- **5 device records**: 3 with `status.state=idle`, 2 with `status.state=notProtected`, none missing status.
- All five had both `lastBackup` and `lastSuccessBackup`, with **equal values** in every record. The two `notProtected` devices nonetheless had prior successful-backup timestamps. The normalizer correctly prioritizes current `notProtected` state over backup history.
- `appliedPolicyNames`, `nextBackupPolicy`, and `nextBackup` were each populated on 3 of the 5 devices; DTO deserialization succeeded for this sample.
- The combined inventory from the earlier review can now be accounted for as **49 Microsoft 365 resources + 5 device resources = 54 checked**, including 52 Healthy and 2 Requires attention. The two unprotected device states align with the two exceptions; individual resource identities were not matched by the diagnostic.
- No `idle`/`ok` device with `lastBackup` newer than `lastSuccessBackup` was observed. Whether an old success plus a more recent unsuccessful attempt may be misclassified remains **unverified**.

**Decision:** Preserve device health classification for the observed cases; do not infer that historical success means current protection. The two unusual device entries remain an operational follow-up with a colleague, not a confirmed application defect. The operator restored the clean hardened device provider, removed the temporary status probe and probe-only tests, and reported **209 tests passed, zero failures, a successful build, and a clean working tree synchronized with the audit branch**. The visible report presents *Not protected* plus *Last activity*, derived from `lastBackup` with fallback to `lastActionTime`; activity is an investigation clue, not proof of current protection or a separate backup system. This is useful evidence for an operational handoff, especially when backup setup changes may not have been communicated.

**Open:** Establish vendor field-type variation and semantic meaning beyond the sampled five device records (including nullable/alternate policy fields and time ordering); continue tenant and group DTO completeness work without introducing speculative production rules.

### Device JSON shape evidence — October 17, 2026

An explicitly enabled, read-only JSON-token diagnostic inspected **one page and five device resource objects** in an anonymized production tenant before typed DTO deserialization. The review completed, with **zero non-object items and zero decode failures**. The diagnostic made no extra API requests and emitted only predefined field names and token-type counts.

- Page `items` was an array, `paging` an object, `paging.cursors` an object, and `after` absent on the final page.
- `id`, `name`, `displayName`, and `type` were strings on all five records; `status` and `agent` were objects. Both `agent.id` and `agent.name` were strings.
- `status.state`, `status.appliedPolicyNames`, and `status.lastAction` were strings on all five. Despite its plural name, `appliedPolicyNames` appeared as a **string**, not an array, in this sample.
- `status.nextBackupPolicy` was a string in three records, missing in two; `nextBackup` was a parseable date string in three, missing in two.
- `lastActionTime`, `lastBackup`, `lastSuccessActionTime`, and `lastSuccessBackup` appeared as parseable date strings in all five; no invalid date strings were observed.

**Decision:** The inspected production shapes support the existing device DTO field types. No DTO or normalizer change is justified by this sample. **Limitations:** Five records from one tenant do not establish a complete vendor contract, and the allowlisted probe does not verify all unmapped JSON fields or vendor semantics. Remove the temporary response-shape probe and probe-only tests, retaining the permanent device collection and pagination safeguards. Continue with tenant/M365-group response contracts.

### Microsoft 365 group-discovery response shape — October 17, 2026

An authorized, opt-in, read-only JSON-shape diagnostic examined the M365 group-list response of the anonymized tenant previously validated with **136 M365 resource records**. The diagnostic did not add API calls or change group selection.

- Response root was an object with `data` as an array; `items` was absent. **13 group objects**, no non-object rows.
- **6 groups had `leaf=true`; 7 had `leaf=false`**; no missing, null, or non-Boolean leaf flags.
- All **13 group IDs were valid UUID strings**; none missing or invalid, including among leaf groups. Therefore the existing filter (`leaf==true` and valid UUID) selected six queryable groups without silently excluding a malformed candidate in this sample.
- `accountId`, `parentId`, `groupKind`, `kind`, `type` were strings in all 13; `groupType` was numeric; `custom` Boolean; `resource_types` an array, matching current DTO types.
- The seven non-leaf groups were not queried by design. The observed response and independently validated 136-resource count support the current group-discovery behavior for this tenant, but cannot establish all vendor semantics or that non-leaf groups never contain relevant resources in other setups.

**Decision:** Do not change M365 group DTOs or leaf selection based on this observation. Remove the temporary group-shape probe and its probe-only tests after successful local cleanup; preserve permanent provider completeness and pagination guards. Continue with tenant-discovery and legacy hierarchy response-contract validation. Sanitized aggregate field counts only—no customer/group names or IDs retained.

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
