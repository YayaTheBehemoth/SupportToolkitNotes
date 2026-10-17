# Ticketing and Configuration

**Status:** Implemented; Zendesk destination routing validated in a personal trial (2026-10-13).

## Shared ticketing

Ticketing is shared infrastructure, not a standalone operational module.

BackupAggregator generates provider-independent ticket drafts. They can be previewed without external writes or submitted through the Zendesk provider. Interactive submission requests operator confirmation; the direct CLI submit command does not.

~~~cmd
dotnet run --project src/SupportToolkit -- backup-aggregator ticket preview
dotnet run --project src/SupportToolkit -- backup-aggregator ticket submit
~~~

## Configuration architecture

SupportToolkit separates three responsibilities:

1. **Profile:** Per-module runtime modes, reusable provider references and one selected Zendesk destination.
2. **Connection:** Named external instance and non-secret connection metadata.
3. **Destination:** A named Zendesk group belonging to one connection.

Each profile selects exactly one destination. Multiple destinations may exist on the same connection, but one ticket creation request targets one group.

~~~text
Zendesk connection: trial
├── Subdomain and OAuth client metadata
├── Secret reference (stored separately)
└── Destinations
    ├── dev        -> Group A
    └── operations -> Group B

Profiles
├── zendesk-dev        -> trial / dev
└── zendesk-operations -> trial / operations
~~~

Credentials are stored through ISecretStore, using Windows Credential Manager on Windows.

### Illustrative configuration

All instance names, client values and IDs below are fictional. This is a fragment, not a complete configuration file.

~~~json
{
  "connections": {
    "zendesk": {
      "trial": {
        "subdomain": "example",
        "clientId": "example-client",
        "destinations": {
          "dev": { "groupId": 111111 },
          "operations": { "groupId": 222222 }
        }
      }
    }
  },
  "profiles": {
    "zendesk-dev": {
      "zendeskConnection": "trial",
      "zendeskDestination": "dev",
      "modules": { "backup-aggregator": "fixture" }
    }
  }
}
~~~

## Ticket routing

The resolver obtains the active profile's connection and destination before constructing a ticketing session. The Zendesk provider maps the selected numeric group ID to the ticket creation payload's group_id field. The shared ticket draft stays provider-independent.

### Fail-closed behavior

- A Zendesk connection and destination must be explicitly selected.
- Missing destinations and non-positive group IDs are rejected.
- Environment credentials alone cannot bypass profile/destination selection.
- Changing the Zendesk instance using ZENDESK_SUBDOMAIN is rejected when it conflicts with the configured connection.
- Changing a connection's subdomain through the wizard clears its existing group mappings.
- The Zendesk provider requires a positive destination group ID.
- Ticket creation requires an explicitly invoked submit operation, resolved connection and valid destination; interactive submission additionally requires confirmation.

These checks do not guarantee that Zendesk business rules preserve a submitted group assignment.

## Wizard interaction

Existing connections are selected from a numbered list; new connection creation is an explicit separate choice.

Selecting an existing connection reuses its metadata and previously stored secret unless the operator opts to update them. The wizard warns if a proposed new connection references a Zendesk subdomain already configured.

Connection selection and ticket destination selection are distinct actions.

~~~cmd
dotnet run --project src/SupportToolkit -- configure
dotnet run --project src/SupportToolkit -- configure profile zendesk-dev
dotnet run --project src/SupportToolkit -- configure status
dotnet run --project src/SupportToolkit -- configure use zendesk-dev
~~~

## Ticket safety and idempotency

- Zendesk tickets are created with private comments by the current provider.
- The narrow Zendesk transport supports ticket creation, not general-purpose mutation.
- There is no separate `AllowWrites` switch. The direct CLI submit command is an explicit write without an additional confirmation prompt; interactive submission asks for confirmation.
- The deterministic idempotency key includes ticket source, content and destination identity (Zendesk instance and group).
- Different groups or instances generate different keys for otherwise identical ticket contents.
- Idempotency only protects within the provider's supported retention window; it is not permanent duplicate-ticket prevention.

An ambiguous creation outcome must not be followed by uncontrolled repeat submissions.

## Validation — 2026-10-13

**Locally reported:** .NET build and automated tests passed. Resolver tests cover valid and invalid destination configuration, mocked HTTP tests cover group_id and private comments, and idempotency tests cover destination separation. The revised wizard was exercised successfully.

**Personal Zendesk trial:** A synthetic BackupAggregator submission through the CLI was confirmed in the intended development-group view. A second synthetic ticket was observed in the operations-group view. The observed ticket priority was Normal, consistent with the inspected Zendesk trigger. Zendesk assigned an individual agent in the observed trial.

**Not yet established:** organizational Zendesk deployment behavior, organizational triggers/automations/notification effects, durable duplicate prevention, and automated interactive-wizard tests. The operations view alone does not establish the exact submission path of that second ticket.

A Zendesk group routes/assigns tickets; a view filters tickets for operators. Neither is a true sandbox security boundary.

## Local configuration and distribution

- Debug configuration: repository-root local.config.json (do not commit).
- Portable release configuration: the user's LocalApplicationData/SupportToolkit/local.config.json.
- SUPPORTTOOLKIT_CONFIG overrides the default configuration path.
- Self-contained Windows x64 ZIP is produced by scripts/publish-windows.cmd.

Preserve the dotnet run workflow and fixture onboarding for non-developers.

Related: [[Architecture and Decisions]], [[Roadmap]], [[BackupAggregator]].

## Audit 2 closure — October 17, 2026

**Source:** SupportToolkit branch `audit/acronis-provider-contracts`; not asserted merged or released.

- Configurations now support keeping existing locally valid Acronis/Zendesk connections without repeating setup; Zendesk destinations support numbered selection with the existing choice as default. Saving another profile does not silently activate it; first-time onboarding still activates the first profile.
- Retired the redundant `AllowWrites` configuration and environment permission flag. Legacy configuration loading still tolerates the former field, and saving drops it. Submission remains an explicit operator operation with required Zendesk destination routing; the interactive UI additionally confirms the instance/group selection.
- Zendesk trial verification included a successfully routed synthetic ticket and an idempotent replay showing the existing ticket identifier and link. Provider idempotency is bounded by its retention behavior; unconfirmed write outcomes require reconciliation before retrying. Organizational deployment was not established.
- Added fail-closed behavior for zero-customer discovery, incomplete tenant reviews, inconsistent resource accounting and ambiguous Zendesk outcomes; operators receive clear partial-result reporting rather than false success.
- Operational logging redacts UUIDs and line breaks; tenant-failure debug diagnostics and top-level debug exception chains omit raw exception messages and stack traces. This is not blanket anonymization of production output.
- The README was refreshed to reflect these behaviors. The operator reported a successful build, **251 passing automated tests**, zero failures/skips, clean whitespace checks and a clean working tree after the final test-only rationalization. The test suite was reduced from 265 to 251 cases while retaining the key contract, pagination, security and submission failure scenarios. These counts are operator-reported local validation, not a live integration test.

**Decision:** Audit 2 implementation and test-rationalization work is ready for branch integration review; merge and release remain user-owned decisions. Longer-term work includes validating vendor pagination variants and organization-specific Zendesk routing and automation behavior.


### Integration status — October 17, 2026

The user completed a **fast-forward merge** of `audit/acronis-provider-contracts` into `main` and pushed `origin/main`. GitHub verification confirms `main` at commit `5705aa5` (`refactor(config): separate profile lifecycle and provider setup`). The local `main` working tree was clean and synchronized after the push. The final structural refactor reorganized `ConfigureCommand` partial files without intentionally changing runtime behavior; the operator reported the full **251-test suite green** after the small active-profile confirmation-message fix. **This is a merged development baseline, not a newly published release.** Branch deletion, if desired, remains user-owned.
