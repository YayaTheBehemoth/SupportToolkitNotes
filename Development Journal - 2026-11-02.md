# Development Journal — November 2, 2026

Decision log for the Zendesk bulk deletion scoping and the related architecture decisions. Details and reasons live in [[Modules/Zendesk Bulk Delete]], [[Architecture and Decisions]] and [[Documentation Checkpoint - 2026-11-02]]. No code was written.

## Stakeholder answers (summarized)

- Users: the internal team that administers Zendesk. Assume an operator who knows what they are doing.
- Main use case: remove all tickets of a former customer, found by organization, email domain, requester or free text.
- Flow: search, review the results, select, confirm, delete. Nothing may be deleted straight from a search.
- A simple yes/no confirmation stating the number of tickets is enough. Non-closed tickets deserve an extra warning.
- Signing in as the acting person is accepted. Which agents may delete is still to be checked.

## Decisions

1. **Ownership and location:** the feature is a SupportToolkit module, built and owned by the project author. The stakeholder supplies the problem; no hours are mandated.
2. **"Archived" clarified:** not a status in any language. A closed ticket is archived automatically about 120 days after closing and keeps status Closed. A former customer's tickets will usually mix archived and recently closed tickets, so archived tickets are in scope by default, not an opt-in.
3. **Interactive only:** never scheduled or unattended. The signed-in operator is accountable for each deletion; the tool does not check the retention rule, which the operator confirms.
4. **Per-user PKCE sign-in** and a narrow deletion interface, separate from ticket creation. The existing ticket-creation credential is not widened.
5. **Confirmation:** count confirmation, plus a second explicit confirmation listing non-closed tickets, which are unselected by default.
6. **Safety rules:** truncated searches are never shown as complete; only the exact confirmed ID list is deleted; the search is not re-run at delete time; the preview breaks results down by organization and requester domain.
7. **Deletion receipt** per run (search, operator, time, counts, IDs; no content): pending the audit-log check in the trial. *(Made required later the same day; see the update below.)*
8. **Testing agreed with the stakeholder:** fake-handler tests, a personal trial with synthetic tickets, then a supervised production run on tickets approved for deletion. That set should contain both archived and recently closed tickets. No sandbox exists.
9. **GUI:** required for this module. A separate WPF project references the existing project and hosts only this module. No platform-wide UI; the library/host split is deferred, not rejected. Converting the existing executable to WPF was rejected because it would break the CLI.
10. **Deployment:** unchanged portable release; the release script gets a second publish target.
11. **BackupHealth retired:** BackupAggregator covers the needed visibility. Remove it as a separate change first, keeping the shared Acronis resource-status provider and older configuration files loadable.

## Planned order

1. Remove BackupHealth.
2. Deletion workflow with tests and a CLI path.
3. WPF project for the deletion module.
4. Trial test, then a supervised production run.

## Still open before building

- Which user the existing ticket-creation OAuth client acts as, and whether that role can delete.
- Whether all intended operators have delete permission.
- What the Zendesk audit log records for an API deletion, and under whom.
- Whether the localhost redirect accepts any port.

Related: [[Development Timeline]], [[Roadmap]].

## Update later the same day

Two open points were answered from Zendesk's official documentation:

- The English article confirms: at most 100 tickets per request, closed for 24 hours or more, 400 requests per minute.
- Deleted archived tickets skip the Deleted tickets view and cannot be restored from the agent interface, but can be restored through the API by ID within the 30-day window. The earlier third-party claim that they become permanent at once was wrong.

Consequence: decision 7 is no longer pending. The deletion receipt with ticket IDs is required, because it is the recovery path for archived tickets.
