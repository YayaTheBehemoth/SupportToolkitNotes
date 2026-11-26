# Development Journal — November 25–26, 2026

Goal: a lab where bulk delete can be used against a self-hosted ticketing system. **Zammad** replaces osTicket in the lab, because osTicket's API cannot delete tickets. Source branch `feature/zammad-provider`, built on `main` after `v0.5.0`; not merged. Verify against the source repository before relying on any of this.

## Zammad in the lab

Zammad 7.3.0 runs in the lab with its own Elasticsearch, PostgreSQL, Redis and memcached (source repository, `lab/zammad/`), on loopback only, about 3 GB of memory. `lab/up.sh` provisions it without the setup wizard: an admin, its group access, and two API tokens whose values come from the operator's own variables. It seeds synthetic organizations, customers and tickets that match the lab's other systems, plus a bulk-delete customer (below).

The observed API contract is recorded in the source repository (`lab/zammad/README.md`). The findings that shaped the code:

| Finding | Consequence |
|---|---|
| Ticket creation (`POST /api/v1/tickets`) has no idempotency; an `Idempotency-Key` header is ignored | Nothing is retried; any unclear answer is "not confirmed, check before retrying" |
| Validation errors (`422`) happen before anything is created | Only `400`/`401`/`403`/`422` count as "no ticket was created" |
| `DELETE /api/v1/tickets/{id}` deletes at once and permanently; no trash | The plan warns that deletion cannot be undone |
| Deleting requires the `admin` permission; `ticket.agent` gets `403` | Creating and deleting can use separate tokens |
| A ticket's ID and its number differ (ID 13 is `Ticket#44013`) | Operators are shown the number; deletion uses the ID |
| Search (`full=true`) returns IDs, a total, and the tickets' customers and organizations, but state IDs only and no ticket text | One call for state names; no ticket text in the preview |
| Search matches loosely: a quoted organization is a phrase within a longer name; a domain is an email suffix | Only exact matches are kept before anything can be selected |

## Zammad as a ticketing system

The Backup Aggregator's review tickets can go to Zammad: a provider like osTicket's, with the same fail-closed outcomes. The lab profile sends its tickets there; a live review ticket arrived as plain text in the configured group.

**Measure: 3 source files changed outside the provider's folder, no layering rules added** — one entry in each of the two composition registries, as the November 20 finding predicted, and the lab profile.

## Bulk delete follows the profile's ticketing system

Bulk delete was built for Zendesk. Two changes made it provider-neutral, with Zendesk's behaviour unchanged:

- **Renamed** from a Zendesk module to `BulkDelete`, as a separate commit with names and folders only.
- **Each system declares its deletion rules** (tickets per request, waiting time after closing, archive age, how long deleted tickets can be restored, browser or saved sign-in, background jobs or direct answers). The module, the console flow and the web page read them instead of Zendesk literals. Zendesk's rules give exactly the texts and values it had before; the existing tests pass with no expected value changed.
- **Selection.** Bulk delete works on the system the profile's `ticketing` setting selects (see [[Architecture and Decisions]], November 25). A deletion connection for a different instance is refused; a system that cannot delete is shown as unavailable and never falls back to another; a profile with no ticketing system keeps using its Zendesk deletion connection.

The deletion engine did not change: its job model (start a job, follow it to its end) also describes a system that answers each deletion directly, as a job that has already finished.

## Bulk delete on Zammad

- **Separate access.** A Zammad connection allows bulk delete only when it opts in, and then needs a second secret, the deletion token (with `admin`). The token that creates tickets has `ticket.agent` only. Using one token for both is refused. A connection used only for tickets needs nothing new.
- **Search.** Paged 100 at a time until Zammad's own count is reached; any shortfall is reported as incomplete. Then only exact matches on organization, requester, domain and sender name are kept: narrowing can only remove tickets, never add one.
- **Deletion.** One ticket per request. `200` is deleted, `404` is not deleted; a refused token or missing permission stops the run before another ticket is sent; anything else, or no answer, is unknown, never deleted.
- **Receipts** record both the ID and the number. A Zendesk receipt is unchanged: the number is left out when there is none.

**Measure:** the provider's folder, the module's Zammad adapter (where the layering rules place provider-specific module code), one registry entry and the lab profile — plus one change inside the module: an optional ticket number shown to operators, because Zammad's screens do not show the ID the API deletes by. No layering rules added.

## Lab changes

- Two Zammad tokens, kept apart by purpose as in production.
- A bulk-delete customer, "Tailspin Toys": 110 tickets (104 closed, 4 open, 2 new) from three requesters on two domains, more than one search page. `lab/up.sh --more-tickets` adds another set after a bulk delete.
- **WSL finding:** on a Windows PC, WSL stops a few seconds after no Windows program uses it, and the lab's containers stop with it. The start script now keeps a minimized window open that holds WSL running; the stop script closes it.
- The lab runs from the repository clone inside WSL. Its data lives beside that clone, so a copy elsewhere must never be started against it.

## Evidence

- Full automated suite green on Windows: 1,740 tests, including recorded-response tests for the Zammad ticket and deletion contracts and a test that pins Zendesk's deletion texts.
- In the lab: a review ticket created in Zammad by a token without `admin`; a bulk-delete search for the lab customer found exactly 110 tickets across two pages, 104 selected by default; one closed ticket was deleted through the web UI, and its receipt recorded the ID, the number and the outcome "Deleted", with no ticket content.
- This is lab evidence for a self-hosted vendor, not validation of any organizational deployment.

## Open

- The usage report still reads its systems directly instead of through the profile's `backup` and `commercial` settings, so it does not follow the lab's systems. Next milestone (see [[Roadmap]]).
- How the lab should stand in for the usage-side backup reads (a second backup system only, or also a stand-in that answers like the production backup system) is undecided.
- A discrepancy in the usage report's storage conversion, seen during a demo, is parked as a separate investigation.
- The Zendesk console command stays Zendesk-only and refuses a profile whose ticketing system is another.
