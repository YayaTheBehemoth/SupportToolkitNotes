# Architecture and Decisions

## Current architecture (`main` at `v0.3.0`)

~~~text
src/SupportToolkit/
  Program.cs
  Cli/Commands/
  Ui/Interactive/
  Core/{Configuration,CustomerMapping,ErrorHandling,Logging,Modules,
       MonitoringScope,Reconciliation,Secrets,Ticketing}/
  Modules/{BackupAggregator,BackupHealth}/      # BackupHealth: retirement decided 2026-11-02
  Providers/{Acronis,BusinessCentral,Zendesk}/
tests/SupportToolkit.Tests/
~~~

The host dispatches operational modules through ISupportToolkitModule. Configuration is a host-level concern. Ticketing is a shared capability attached to operational modules, not a separate main-menu module.

Providers own external API details; modules own operational workflows; Core owns reusable infrastructure and provider-independent contracts.

## Decisions

| Decision | Reason | Status |
|---|---|---|
| Modular C#/.NET console | Extensibility and maintainability | Implemented |
| Fixture/production module modes | Development independent of production data | Implemented |
| Restricted Acronis read transport | Reduce accidental mutations | Implemented |
| Windows Credential Manager secrets | Keep secrets out of plaintext config | Implemented |
| Gated external writes | Separate monitoring from side effects | Implemented |
| Shared ticketing | Avoid duplicated provider logic | Implemented |
| Named Zendesk destinations scoped to connections | Centralize mappings per Zendesk instance | Implemented |
| One selected destination per profile | Predictable routing without implicit defaults | Implemented |
| Fail-closed Zendesk destination resolution | Prevent submissions with invalid or missing routing | Implemented |
| Destination-aware idempotency | Distinguish retry identity across instances/groups | Implemented |
| Explicit existing-connection selection in wizard | Prevent accidental credential duplication | Implemented |
| Independent compliance aggregation | Avoid mandatory Lighthouse dependency | Investigating |
| Retire the global `AllowWrites` switch | Duplicated the real write gates (explicit submit, resolved destination, interactive confirmation) and could give false assurance | Implemented (`v0.2.3`) |
| Pin Business Central API/authority hosts and scope in code | A tampered configuration file must not redirect a bearer token | Implemented (`v0.3.0`) |
| Scoped review is separate and opt-in | Default review and ticketing must not depend on unvalidated entitlement logic | Superseded in `v0.3.0`: the interactive console offers only the scoped review; the all-tenant review remains a command |
| Automatic name-based customer-to-tenant matching, nothing stored | No shared identifier exists; unique strong matches only, ambiguous and conflicting cases stay visible | Implemented (`v0.3.0`); supersedes the October 14 guidance below |
| Explicit Zendesk submitter and requester per connection | With client credentials, tickets would otherwise be attributed to the user who created the OAuth client | Implemented (`v0.3.0`) |
| Retire BackupHealth | BackupAggregator covers the needed visibility | Decided 2026-11-02; code not yet removed |
| GUI for the Zendesk deletion module only (separate WPF project) | One concrete workflow requires a GUI; no platform-wide UI | Decided 2026-11-02; not built |
| Zendesk deletion uses per-user PKCE, its own narrow interface | The acting operator is the identity; the ticket-creation credential is not widened | Decided 2026-11-02; not built |

## Zendesk destination architecture — 2026-10-13

A named Zendesk connection describes an external instance and its available ticket group destinations. A SupportToolkit profile references one connection and selects exactly one named destination. The resolver produces the group ID, and the Zendesk provider includes it in ticket creation.

This supports multiple profiles reusing credentials and group mappings without duplicating those IDs inside every profile.

### Boundaries

- The provider-independent TicketDraft model has no Zendesk group fields.
- ZendeskTicketProvider owns Zendesk request mapping, including group_id.
- Group IDs are scoped to the instance-defining connection.
- Missing or invalid routing configuration fails before submission.
- A Zendesk view is operator-facing filtering, not an API routing mechanism.
- The active profile selects one Zendesk destination for ticket submissions. Per-module routing within a single active profile has not been designed or requested.

## Wizard usability finding

During trial testing, a second profile was inadvertently configured with a new connection when the operator intended to reuse a connection and select another ticket destination.

The wizard previously treated entering a connection name as the normal path and made connection creation difficult to distinguish from destination selection.

The revised wizard lists existing connections, makes creation explicit, reuses stored credentials and warns on duplicate-instance creation. This is an operator-interface correction, not a change to the underlying routing model.

## Validation limits

Mock HTTP tests cover group_id routing and private comments; local tests were reported green. A personal Zendesk trial demonstrated synthetic development-group routing. Organizational instances may have different triggers, automations and permissions.

Idempotency is bounded retry protection, not permanent deduplication. Direct CLI write confirmation and wizard interaction tests remain possible hardening work.

## Other lessons

- Activity lifecycle state completed does not imply successful result.
- Deduplicate by stable resource identity and meaningful timestamps.
- Resource inventory objects may not map one-to-one to devices or portal rows.
- Multi-tenant workflows need failure isolation and explicit incomplete states.
- Trial and production observations should be distinguished from model assumptions.
- Configuration abstractions must remain understandable to non-developer operators.

See [source README](https://github.com/YayaTheBehemoth/SupportToolkit/blob/main/README.md).

Related: [[Ticketing and Configuration]], [[Roadmap]].

## Data authority and workflow separation — 2026-10-14

**Decision:** Monitoring entitlement must be independent of backup resource inventory and scheduled-report delivery configuration. Commercial records determine eligibility; backup data determines resource state; operational reporting decides presentation and follow-up. Exact entitlement rules remain to be discovered.

**As of October 14:** a fixture-backed monitoring-scope provider was proposed. On October 15, a disconnected fixture evaluator, tenant selector and scoped inventory execution path were implemented on an unmerged feature branch. An authorized production Business Central provider remains unimplemented.

**Cross-system identity:** no shared identifier is confirmed. Investigate existing references and integrations before adopting name matching or maintaining a manual registry. Unmatched identities are explicit exceptions, not silently excluded records.

*Superseded October 27 (`v0.3.0`):* conservative automatic name matching was adopted. The principle that unmatched identities stay explicit exceptions still holds. See [[Documentation Checkpoint - 2026-11-02]].

**Compliance boundary:** synthetic Graph-backed exception lists, setting evidence, diagnostics and investigation reports are a development milestone, not proof of production Graph authorization or completeness. Preserve separation between collection, interpretation and case lifecycle action.

Related: [[Modules/BackupAggregator]], [[Modules/Compliance Monitor]], [[Development Journal - 2026-10-14]].

## Monitoring-scope boundaries — 2026-10-15

The implemented **fixture-only** pipeline is:
`subscription evidence → conservative entitlement assessment → independently resolved customer identity → Acronis tenant selection → inventory review`.

- Entitlement and backup resource health are distinct. Scope selection precedes resource collection rather than hiding completed report rows.
- Absence, ambiguity or conflicting evidence cannot silently establish non-entitlement. The selector preserves unresolved tenants and unmatched commercial observations.
- `Inventory complete`, `scope complete` and `overall complete` are separate assertions. Successful selected inventory does not mean all monitoring responsibility has been resolved.
- Identity links in fixtures are fictional. There is no validated automatic BC-to-Acronis reconciliation or permanent production mapping registry.
- The scoped service method reuses existing inventory concurrency and failure handling. The normal all-tenant production workflow and ticketing were not changed.
- A read-only BC transport guard exists, but approved production endpoints, authentication/consent and exact API payloads remain unknown. No production BC access has been demonstrated.

Design guardrail: do not wire commercial filtering or external ticket side effects to these fixture results until authoritative access, entitlement rules, identity completeness and unresolved-case behavior are validated.

Related: [[Development Journal - 2026-10-15]], [[Modules/BackupAggregator]].

## October 20 — Deployment architecture intentionally deferred

The approval POC discussion expanded from one reviewable ticket draft into hypothetical decisions about scheduling, multi-user deployment, storage and replacing the CLI. This is premature given differing module requirements. **Decision:** keep a small synthetic approval experiment, preserve the current modular CLI and security boundaries, and defer a common hosting/interface architecture until concrete operational requirements justify it. See [[Strategy Checkpoint - 2026-10-20]].


## October 22 — Documentation reflects released behavior, branches stay labelled

The README, example configuration and Notes were refreshed against `main` at `83a7bf1` and the two feature branches. Reasons and findings are in [[Documentation Checkpoint - 2026-10-22]]. Guardrail: README text may describe released behavior; branch-only work appears only under a development branches section with its evidence boundary stated, so fixture output cannot be read as production validation.

## November 2 — A GUI for one module, not a UI platform

The Zendesk deletion module needs a GUI. The interactive shell is already a presentation layer over module workflows, so a GUI is a second presentation layer, not a new architecture.

**Decision:** add a separate WPF project that references the existing project (as the test project already does) and hosts only the deletion module. The deletion workflow contains no UI code and keeps a CLI path for development and testing. The existing CLI and the portable release stay as they are; the release script gains a second publish target.

**Not done now:** splitting the code into a class library with separate console and GUI hosts. That pattern enforces that presentation cannot leak into modules, but it is not needed to add one GUI. Revisit it if presentation code starts leaking into modules or a second module needs the GUI. Converting the existing executable to WPF was rejected because a Windows GUI executable has no console, which would break the CLI.

**Also decided:** retire BackupHealth as a separate change before the deletion module. Keep the Acronis resource-status provider (BackupAggregator uses it) and keep older configuration files loadable.

See [[Modules/Zendesk Bulk Delete]] and [[Documentation Checkpoint - 2026-11-02]].
