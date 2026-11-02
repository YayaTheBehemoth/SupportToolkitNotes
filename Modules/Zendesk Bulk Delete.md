# Zendesk Bulk Delete

**Status (2026-11-02):** Scoped and designed. **Nothing is implemented.** No Zendesk deletion code exists in SupportToolkit, and no deletion has been tested against any Zendesk instance.

## Problem

An internal team that administers Zendesk needs to remove all tickets belonging to a former customer after the customer relationship has ended, as required by an internal retention rule. Today this is slow and partly impossible from the Zendesk agent interface:

- The agent interface can bulk delete open, pending and solved tickets from a view, 100 at a time, but **not closed tickets**. ([Managing tickets in bulk](https://support.zendesk.com/hc/en-us/articles/4408886890906))
- Closed tickets can only be bulk deleted through the API or with deletion schedules. ([How can I bulk delete closed tickets?](https://support.zendesk.com/hc/en-us/articles/4408830952474))
- Deletion schedules only delete **archived** tickets and are age-based, so they cannot cover a customer whose last tickets closed recently, or be triggered by the end of a customer relationship. ([Creating ticket deletion schedules](https://support.zendesk.com/hc/en-us/articles/6388012977306))

The request: search tickets by a key (organization, email domain, requester, free text), review the result, select tickets, confirm, delete.

## "Archived" is not a status

The basic Zendesk status flow is Open (optionally Pending or On-hold) → Solved → Closed; an automation closes solved tickets after a configurable period. **Archived is not a status and has no localized status name.** About 120 days after a ticket is closed, Zendesk archives it automatically; it keeps status Closed and shows an "archived ticket" banner. There is no view or search filter for archived tickets, though they remain reachable through search, direct links and user/organization profiles. ([About ticket archiving](https://support.zendesk.com/hc/en-us/articles/4408887617050))

Consequence for this use case: a former customer's tickets will typically be a **mix** of archived tickets (closed long ago) and recently closed, non-archived ones. Archived tickets are the main case, not an edge case.

## Verified API facts (official Zendesk documentation)

- `DELETE /api/v2/tickets/destroy_many`: at most 100 IDs per request, tickets closed for 24 hours or more, standard limit of 400 requests per minute (English article confirmed 2026-11-02). ([How can I bulk delete closed tickets?](https://support.zendesk.com/hc/en-us/articles/4408830952474))
- Deleting a ticket moves it to the Deleted tickets view for 30 days, after which it is permanently deleted. **An archived ticket that is deleted does not appear in the Deleted tickets view and cannot be restored from the agent interface, but it can still be restored through the API by ticket ID during the 30-day window.** ([Viewing and recovering deleted tickets](https://support.zendesk.com/hc/en-us/articles/4408835089050), [Deleting tickets](https://support.zendesk.com/hc/en-us/articles/4408883872538))
- The audit log records manual permanent deletions; automatic permanent deletion after 30 days is not tracked there. Deletion events can be reported in Explore (Updates history, Activity > Deletions). After permanent deletion the ticket ID remains as a record with content scrubbed.
- Search API: at most 1,000 results per query, 100 per page, offset pagination that can return duplicates. `count` reports the true total. The export endpoint is cursor-based and returns archived tickets.
- OAuth scope `tickets:write` includes permanent deletion; `tickets:read` is the only separate ticket scope.
- PKCE is supported for public clients (S256, no secret); http redirects are allowed only for `localhost`/`127.0.0.1`.

## Unverified

- What the Zendesk audit log records for a (soft) API deletion, and under which user.
- That the API restore path works for tickets deleted through `destroy_many` (documented, not tested; the trial has no archived tickets).
- Whether a localhost redirect accepts any port.
- Which agent roles may delete tickets in the target instance (a Zendesk setting).

## Decisions — November 2, 2026

| Decision | Reason |
|---|---|
| Lives in SupportToolkit as its own module, owned by the project author. | Built on own initiative; the requester supplies the problem, not a mandate or hours. |
| Interactive only; never scheduled or unattended. | Bulk deletion of customer data must always have a person reviewing it. The signed-in operator is accountable for each run. |
| Per-user OAuth with PKCE; the acting person is the identity. | Each operator's own Zendesk permissions apply. The existing ticket-creation credential is not widened. |
| Deletion sits behind its own narrow interface, separate from ticket creation. | Keeps the destructive capability isolated and reviewable. |
| Flow: search → review → select → confirm → delete → back to search. | Requested flow. Nothing is deleted without an explicit selection and confirmation. |
| Two-step confirmation: count first; a second, explicit confirmation listing any non-closed tickets. | Requested. For a former customer, a non-closed ticket is an anomaly. |
| Non-closed tickets are unselected by default. | They must be chosen deliberately. |
| A truncated search is never shown as complete. Compare the reported count with unique IDs received; refuse or fall back to the export endpoint. | The 1,000-result search cap is realistic for long-standing customers. |
| Delete only the exact confirmed ID list; never re-run the search at delete time. | What was reviewed is what is deleted. |
| Preview shows breakdowns by organization and requester domain. | Email-domain and free-text keys can catch other customers' tickets; strays must surface as exceptions. |
| The tool does not check the retention policy itself. | It cannot know the relationship end date; the operator confirms it. |
| A deletion receipt per run (search, operator, time, counts, ticket IDs; no ticket content). **Decided 2026-11-02.** | Archived tickets can only be restored through the API by ID within 30 days, so the receipt's ID list is the recovery path. It also gives the operator evidence afterwards. |
| GUI: a separate WPF project referencing the existing project, containing only this module. Workflow stays UI-independent with a CLI path for development. | A GUI is required for this module. One module's need does not justify a platform-wide UI. See [[Architecture and Decisions]]. |

## Testing plan

1. Fake HTTP handler tests for search completeness, ID handling, closed-only rules and confirmation logic.
2. A personal Zendesk trial with synthetic tickets for the real API contract and the audit-log observation.
3. One supervised production run on tickets the stakeholder has approved for deletion. The set should include both tickets closed more than 120 days ago and recently closed ones, so both deletion behaviours are observed. No sandbox is available.

Related: [[Ticketing and Configuration]], [[Architecture and Decisions]], [[Roadmap]], [[Documentation Checkpoint - 2026-11-02]].
