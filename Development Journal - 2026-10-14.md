# Development Journal — 2026-10-14

> [!note] Evidence and publication boundaries
> Consolidated from the user's Compliance Graph research handoff and BackupAggregator monitoring-scope discovery handoff. These notes are intentionally sanitized: no customer identities, contract data, internal endpoints, authentication experiments, tenant identifiers, or operational investigation procedures. Reported implementation/validation is distinguished from independently verified source behavior.

## Compliance — fixture-driven investigation

**Reported implemented with synthetic Graph responses:**
- Exception-first device list.
- Setting-level evidence.
- Collection diagnostics.
- Readable investigation reports.

**Next milestone:** realistic fixture scenarios involving missing devices, contradictory evidence and incomplete observations. These can proceed without waiting for production Graph access.

**Unresolved:** live authorization, production data completeness and operational case-lifecycle requirements. Synthetic responses cannot establish any of these.

## BackupAggregator — monitoring scope

**Reported:** four successful production comparison runs between BackupAggregator output and existing tenant-level reports. The major remaining business requirement is determining which customers are entitled to managed monitoring.

### Decisions

1. Separate **commercial entitlement**, **backup inventory** and **operational reporting**.
2. Scheduled backup reports and their delivery configuration are not authoritative evidence of monitoring entitlement.
3. Use the established commercial subscription system as the operational entitlement source, subject to discovery of exact rules.
4. Do not design permanent manual customer-identity mapping before checking existing identifiers and integrations.
5. Keep unmatched or unknown scope visible; never silently classify it as ineligible.
6. Preserve the existing BackupAggregator implementation. No production entitlement provider has been approved or implemented.

### Investigation result

An exploratory scheduled-report integration was evaluated and abandoned because the data did not reliably represent commercial entitlement. No scheduler feature was merged. This note intentionally excludes the technical access path and investigation mechanics.

### Open questions

- Which subscription records and statuses establish purchased monitoring?
- How do effective dates affect eligibility?
- Can authorized read-only exports provide sufficient data?
- Is a suitable supported API available?
- Are stable customer references shared between commercial and backup systems?

**Next planned discovery (October 15):** inspect an authorized, read-only subscription export, using only sanitized field structures and synthetic examples for public documentation. The export is a discovery artifact, not yet an approved production source.

## Architectural synthesis

Both workstreams benefit from explicit separation of **data acquisition**, **interpretation** and **operational decisions**. Unavailable, contradictory or unmatched data must remain visible as uncertainty, not be transformed into a confident negative conclusion.

Related: [[Modules/Compliance Monitor]], [[Modules/BackupAggregator]], [[Architecture and Decisions]], [[Roadmap]].
