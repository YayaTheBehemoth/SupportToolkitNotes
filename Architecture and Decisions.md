# Architecture and Decisions

## Current architecture

~~~text
src/SupportToolkit/
  Program.cs
  Cli/Commands/
  Ui/Interactive/
  Core/{Configuration,Secrets,Ticketing,Logging,ErrorHandling,Modules}/
  Modules/{BackupAggregator,BackupHealth}/
  Providers/{Acronis,Zendesk}/
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
