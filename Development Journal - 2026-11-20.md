# Development Journal — November 20, 2026

The first second provider from [[Strategy Checkpoint - 2026-11-20]]: **osTicket** behind the ticket destination that Zendesk fills. Source branch `feature/osticket-provider`, built on `feature/linux-dev`; neither is merged into `main`. Verify against the source repository before relying on any of this.

## osTicket lab and contract discovery

A throwaway osTicket 1.17.5 instance with MariaDB ran in the Codespace through `docker compose` (source repository, `lab/osticket/`). It listens on loopback only, its passwords come from the environment with no defaults, and its mail goes to a host that does not exist. Every request used synthetic data.

Observed behaviour of `POST /api/tickets.json` (header `X-API-Key`):

| Case | Result |
|---|---|
| Valid create | `201`; the body is the **ticket number** as plain text, not the internal ID |
| The same request again, or with an `Idempotency-Key` header | `201` and **another ticket**: there is no idempotency |
| Wrong key, or the right key from another source IP address | `401` (a key is bound to one address) |
| Malformed JSON | `400` |
| Missing subject or message | `500`, body starting `Unable to create new ticket`, naming the field |
| Email address osTicket considers invalid (the reserved `.invalid` domain included) | `500`, `Incomplete client information` |
| **Unknown help topic** | **`201`**: filed without a topic in the default department; routing errors are not reported |
| Message sent as a `data:text/plain` URL | Stored as text; HTML escaped; commas, Unicode and `%` kept |

The API cannot read tickets or list help topics, so a configured topic cannot be validated through it.

## What was built

- **A provider-neutral ticketing target.** Core gained `ITicketingTarget`: a resolved, locally validated destination with a preview, status details, a retry identity and a session that creates tickets. The Backup Aggregator now has one sink for every system, and `Composition/` is the only place that maps a system name to its provider. Zendesk moved behind the same contract.
- **Profile selection.** A profile names its system, connection and destination in one `ticketing` setting. Profiles that still use the older Zendesk connection and destination settings load and behave as before; when both exist, `ticketing` wins. The wizard does not configure osTicket yet: it is set in the configuration file, and the wizard shows it and leaves it alone.
- **Destination preview.** The confirmation screens name the system, its routing (Zendesk group, osTicket help topic) and, for osTicket, a standing warning: the help topic is not confirmed, and there is no duplicate protection.
- **Fail-closed outcomes.** Only an answer osTicket gives before creating anything (refused key, malformed request, validation error) is reported as "no ticket was created". Any other failure is "not confirmed", with advice to check the helpdesk before retrying, because a retry creates another ticket. Nothing is retried automatically.
- **Transport.** HTTPS is required because the key travels in a header; plain HTTP only for a loopback lab. Redirects are not followed.
- **Secrets.** The API key is an ordinary connection secret: Windows Credential Manager on Windows, `SUPPORTTOOLKIT_SECRET__OSTICKET__<CONNECTION>__API_KEY` elsewhere. It is included in required-secret and clash checks.
- **Status.** Console and web status report whichever system the profile selects, with no secret.

## Evidence

- Full automated suite green in the Codespace, including new fixture tests built from the recorded lab responses.
- Against the lab: status named the missing key's variable; a fixture-mode review submitted a ticket that landed under the configured help topic as plain text, with a working staff link; a wrong key produced "no ticket was created".
- This is lab evidence for a self-hosted vendor, not validation of any organizational deployment.

## The measure

**29 source files changed outside the provider's folder; no layering rules added.**

Most of these are a one-time correction of the ticketing seam (the shared target, the single sink, provider-neutral status and confirmation screens). A third ticketing system would still change about eight shared files: seven in `Core/` for its connection settings, resolver and secret entries, and one line in the composition. That remaining coupling is a finding about the configuration seam: each provider's connection settings live in Core, because Core may not reference providers.

## Follow-up: provider-owned connection settings

The finding above was addressed the same day, on the same branch. A provider can now own its connection settings: they live in its own `connections.<section>` of the configuration file, which Core keeps without knowing their shape, and the provider reads and writes them itself. Core knows only what every connection has, a name and one secret, through a small contract each such provider implements. `Composition/` registers these providers, and the required-secret checks, secret-clash checks and status cover every registered one; a test fails if a provider exists without being registered, because an unregistered one would be skipped silently.

osTicket moved onto this. Core no longer names it, saved configurations and secret keys are unchanged, and the lab check was repeated successfully. **A further ticketing system now changes its provider folder plus one entry in each of two composition registries.** Acronis, Zendesk and Business Central stay as they are, by decision.

## Duplicati behind the Backup Aggregator

Second provider of the planned order: **Duplicati** (backup), on branch `feature/duplicati-provider`, built on the osTicket branch. Two Duplicati 2.4.1.0 servers ran in the Codespace lab (source repository, `lab/duplicati/`), standing in for two customer machines.

**Contract findings.** Each server has its own password and signs in to a short-lived bearer token. No single field states a job's latest outcome: a warning appears only in the run log; a run that fails before the job's local database exists appears only in the job's metadata; and the metadata keeps the last error's date after later successes, with whole-second precision. The job list returns each job's destination URL, which in real configurations can carry destination credentials, and error messages contain paths.

**Decisions (agreed November 20).** One Duplicati connection is the MSP's fleet: it lists customers, and each customer lists its servers. A customer is a tenant; each backup job on its servers is a resource. Every server has its own password, so a provider connection may now declare several secrets. A profile selects its backup system in a `backup` setting, like `ticketing`; profiles that only name an Acronis connection keep reviewing Acronis.

**What was built.**
- A backup-system registry in `Composition/`, mirroring ticketing; the all-tenant and monitored reviews take their inventory from it.
- The provider decides each job's latest outcome from the log and the metadata together; when they cannot decide it (an error and a success in the same second, an unreadable time), the outcome is unknown, not a guess. It never reads a destination URL or keeps error text.
- The transport is read-only by construction: only reads and the sign-in can leave the computer.
- Classification follows the Acronis rules: success is healthy; warning, failure and never-run need review; a running job is healthy only after an earlier success; anything undecided stays visible as unclassified.
- One-time corrections to the backup seam: the tenant review counts resources per kind instead of Acronis's Microsoft 365 and device counts, and a report entry may state its own display status, because the shared presentation spoke Acronis's state names.
- Status shows the selected fleet and no longer asks for an Acronis connection for a review that does not read Acronis.

**Evidence.** Full automated suite green with recorded-response tests; against the lab, a full review read both customers' jobs and classified them as expected, a wrong password failed only that customer, reported as incomplete; a synthetic monitored review matched Duplicati customers to commercial evidence by name.

**The measure.** 22 source files changed outside the provider's folder; no layering rules added. Most are one-time corrections of the backup seam and the several-secrets contract. **A third backup system now needs its provider folder, its module adapter (required by the layering rules) and one entry in each of two composition registries.** Still Acronis-shaped: the monitored review's wording ("Acronis tenant"), and the usage report, which compares Acronis storage by design.

## Dolibarr behind the monitoring scope

Third provider of the planned order: **Dolibarr** (commercial entitlement), on branch `feature/dolibarr-provider`, built on the Duplicati branch. A Dolibarr 23.0.4 with MariaDB ran in the Codespace lab (source repository, `lab/dolibarr/`) with synthetic customers, services and contracts.

**Contract findings.** Activity is per contract line, not per contract: each service line is never activated, running or closed, with its own activation and closing dates. Contracts name their customer by internal ID; the customer number operators know is the third party's customer code. Types are loose: IDs, statuses and quantities arrive as strings, and a date as Unix seconds, an empty string or null for the same field. A refused key returns a body with a debug block, which is never logged or shown. Dolibarr has no "blocked" concept.

**Decisions (agreed November 20).**
- **One subscription per contract line**, numbered "contract reference/line ID", with that line's status and dates. A running line of a validated contract is active; a never-activated or closed line, or a draft or closed contract, is suspended; any other status is passed on unrecognized and reviewed. A line's actual dates win over its planned ones, so a running line past its planned end ends then.
- **Monitoring scope first.** The usage report's billing source is a follow-up, because the usage report compares billing with Acronis storage.

**What was built.**
- A commercial-system registry in `Composition/`, mirroring backup and ticketing; a profile selects its system in a `commercial` setting, and profiles that only name a Business Central connection keep using it. The monitored review is available with any commercial system.
- The provider implements the Domain's subscription-evidence and customer-name contracts itself, so no module adapter is needed. Evidence is complete or the read fails: an unreadable date fails it rather than reading as "no date". A failed name read degrades matching visibly, as with Business Central.
- The transport is read-only by construction (only REST reads leave the computer), pages to the end, and refuses a server that repeats records.
- Status shows the selected system and no longer asks for Business Central for the monitoring scope.
- The ticket text no longer claims "Business Central entitlement"; it says "subscription entitlement".

**Evidence.** Full automated suite green with recorded-response tests. Across the labs: Dolibarr entitlement selected the one customer with a running monitoring line, Duplicati supplied its backups, the review was complete, and a ticket from it was created in osTicket, all three second providers behind their seams in one run. A wrong key stopped the review with a clear message.

**The measure.** 14 source files changed outside the provider's folder; no layering rules added. Twelve are one-time corrections of the commercial seam. **A further commercial system now needs its provider folder and one entry in each of two composition registries.** Still shaped by the first providers: the monitoring-scope report's wording, and the Domain's subscription vocabulary, which Dolibarr maps onto rather than changes.

## Integration and the lab as a working environment

The three provider branches were merged into `feature/linux-dev` by fast-forward and pushed; `main` is unchanged, and the merge into it is the owner's decision. Fleet + osquery is deferred: compliance exists only on the unfinished `feature/compliance-investigation` branch, built directly on Microsoft Graph interfaces, and would need reworking to the shape of the other seams first.

To make switching between production and the lab simple:
- `lab/up.sh` starts the three lab systems, seeds their synthetic records and sets the keys SupportToolkit uses from variables the operator owns. The four keys the toolkit reads use its own `SUPPORTTOOLKIT_SECRET__…` names, so the lab and the toolkit share one value.
- `configure lab` adds or resets a `lab` profile beside the operator's own profiles, never instead of them, and asks only for the lab's secrets. Switching is `configure use lab` and back.
- Status, the profile page and the Backup Aggregator page name the systems the active profile uses; the Backup Aggregator's production banner had always said "Acronis and Business Central".
- The installed exe reaches the lab while the codespace runs, through ports forwarded to the same numbers on the operator's computer.
- The wizard no longer asks for an Acronis connection for a profile that reviews another backup system.
- *Later:* on November 23, `feature/linux-dev` was merged into `main` and released as `v0.5.0`.

**Finding from a fresh lab run:** Dolibarr answers an empty customer list with `404 "Not Found: No third parties found"`, not `[]` as for contracts. The provider treated it as a failure; only that exact answer now means "empty", and any other 404 still fails.

## Related fix

On Linux, a missing Acronis or Zendesk secret was reported with "run the configure wizard", which cannot store a secret there. The message now names the environment variable to set.

Related: [[Strategy Checkpoint - 2026-11-20]], [[Ticketing and Configuration]], [[Roadmap]].
