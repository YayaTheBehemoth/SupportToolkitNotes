# Development Journal — 2026-10-13

> [!note] Evidence boundary
> This is a **sanitized cross-workstream consolidation**, not a claim that every investigation below happened on October 13. It records only findings available from existing project discussions and repository notes. Additional October 13 investigation details require reconciliation with their original conversations.

## Documentation policy

Preserve transferable technical findings and decisions; exclude customer identities, tenant/device identifiers, raw production evidence, sensitive investigative procedures, access paths, internal case notes and organization-specific security operations. Use synthetic examples only. Keep personal/career discussion outside project documentation.

## Ticketing / Zendesk — confirmed in existing notes

- Named destinations belong to a provider connection; each active profile selects one destination.
- A ticket creation request routes to one group, not multiple groups.
- Explicit connection reuse in the configuration wizard reduces accidental creation of duplicate connections and confusion about stored credentials.
- Local build/tests and a synthetic trial established routing to an intended group.
- A second synthetic ticket was observed in another group; the observation alone does **not** prove its exact submission path.
- Organizational trigger behavior, deployment effects and long-term duplicate prevention remain unverified.
- Shared ticketing is infrastructure used by modules, not a standalone operational module.

See [[Ticketing and Configuration]] for the detailed technical and validation record.

## BackupAggregator — existing validation principles

- Inventory objects, backup activities and portal reports can represent different units of observation; apparent count mismatches require reconciliation before changing normalization.
- Tenant-level exception summaries must preserve unknown/incomplete results.
- Fixture success does not prove production parity. Repeated comparisons should record per-tenant discrepancies without publishing customer data.
- No new October 13 backup-specific root cause is asserted here.

See [[BackupAggregator]].

## BackupHealth — existing implementation boundary

- Health evaluation and stale-activity rules are implemented, but evidence freshness and API-versus-portal semantics must be considered separately.
- No new October 13 health-specific finding is asserted here.

See [[BackupHealth]].

## Compliance — process discovery, not lifecycle automation

### Observed investigation work
- The known workflow includes reviewing why a device is noncompliant, checking relevant device activity/check-in evidence and recording findings.
- An alert may require direct technical action, customer follow-up or further investigation; the next action depends on the trigger and evidence.
- Inactive devices can remain unresolved when verification depends on a customer response or a device coming online.

### Lifecycle distinctions
Investigation, follow-up, remediation, verification and closure are **different stages**. Only the investigation stage has been described in enough first-hand detail to begin a focused design. Later stages must be validated against actual operational practice.

A pending/on-hold case can depend on an external event and currently requires a deadline; **no universal deadline duration has been established**. Cases may be closed, deferred or remain in a queue, but the criteria are not yet sufficiently established to automate these decisions.

### Architectural consequence
Begin with a fixture-driven, read-only investigation assistant that produces evidence and suggested next steps. Do not automate putting cases on hold, choosing deadlines, closing tickets or remediation decisions until their rules are observed and approved.

Graph/Intune access and data coverage are separate technical discovery questions; simulated responses do not establish live API permissions or completeness.

See [[Compliance Monitor]].

## Cross-cutting next actions
- [ ] Reconcile this entry with the complete October 13 conversation findings when accessible.
- [ ] Capture confirmed compliance investigation inputs, outputs and decision points as synthetic fixtures.
- [ ] Observe follow-up, remediation, verification and closure before defining lifecycle automation.
- [ ] Maintain explicit evidence boundaries for Zendesk trial versus organizational deployment.
- [ ] Preserve per-tenant backup validation results privately and publish only sanitized patterns.

Related: [[Roadmap]], [[Development Timeline]], [[Architecture and Decisions]].
