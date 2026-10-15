# Development Journal — 2026-10-15

> [!note] Public evidence boundary
> This record summarizes authorized user-interface findings, code on an unmerged feature branch and operator-reported fixture runs. Customer details, real service codes, proprietary table IDs, internal URLs, production payloads, credentials, screenshots and sensitive investigative steps are omitted. Fixture execution is not proof of live API access or commercial coverage.

## Business Central — subscription discovery

Goal: determine which customers purchase managed backup monitoring, independently of existing backup inventory and report-email delivery.

- UI inspection identified matching subscription header and line data. A header contains a subscription reference, customer reference, status and effective dates; a line identifies a purchased service and links back to its header.
- Multiple subscription records containing the monitored service were observed. One line reference was checked against the matching header and customer record.
- An active subscription had a populated start date and no end date. A blocked subscription retained the service line and also had no end date. Presence of the item or an empty end date alone therefore cannot establish active responsibility.
- The inspected table viewer offered no usable bulk Excel export. No relevant published subscription web service was found via the investigated listing. A different supported API/export mechanism may exist.

**Unknowns:** full status/effective-date and billing semantics, whether the extension publishes a supported read-only API, approved permissions, and stable cross-system customer references.

## Fixture-driven development — unmerged SupportToolkit branch

On `feature/business-central-monitoring-scope`:

1. Normalize subscription headers and lines and evaluate a configurable monitoring-item identifier (synthetic value `MONITORING-SERVICE`).
2. Keep active monitoring **candidates** separate from evidence requiring investigation. Ordinary fixture output favors three candidates, one review case and one lacking monitoring evidence; more unusual cases live in focused tests.
3. Consume **fictional resolved** BC-customer/Acronis-tenant relationships to select tenants. The selector does not guess from names, nor prove an actual shared identity.
4. Preserve unclassified tenants, conflicting assignments and unmatched commercial evidence rather than silently excluding them.
5. Add an optional selected-tenant inventory method using the existing concurrency and per-tenant failure handling, without changing the normal all-tenant production path.
6. Track selected-inventory, monitoring-scope and overall completeness independently. Production ticketing and operational scope selection are unchanged.

Fixture commands:

~~~cmd
dotnet run --project src/SupportToolkit -- dev monitoring-scope
dotnet run --project src/SupportToolkit -- dev scoped-backup-review
~~~

### Reported validation

The supplied local console output showed five fictional Acronis customer tenants: three selected and reviewed, one needs scope review and one unclassified. Three healthy synthetic backup device resources were processed, with zero collection failures and zero external requests.

Completeness: **selected inventory YES; monitoring scope NO; overall NO**. The operator described the execution as successful; the final scoped increment's full automated test result was not provided here.

A Business Central read-only transport handler also exists with HTTPS and allowlist restrictions; no production authentication/client or validated subscription endpoint is in use.

## Decisions and next dependency

The model is **commercial evidence → entitlement assessment → resolved identity → tenant selection → backup inventory**. Subscription state is not itself a complete operational contract decision; unresolved scope must remain visible and should not be represented as a successful overall run.

The next practical handoff is to a Business Central developer: ask whether the header and line records are exposed by an **existing supported authenticated read-only API/service/export**, and whether stable integration references exist to reconcile customers with Acronis. Do not assume extension table IDs imply API routes or replace that discovery with manual name matching.

Do not merge or turn on production scoped filtering until data access, business rules, identity reconciliation and authorization are resolved.

Related: [[Modules/BackupAggregator]], [[Architecture and Decisions]], [[Roadmap]].
