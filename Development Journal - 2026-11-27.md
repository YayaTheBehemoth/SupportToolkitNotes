# Development Journal — November 26–27, 2026

Continues [[Development Journal - 2026-11-26]] on branch `feature/zammad-provider` (not merged). Verify against the source repository before relying on any of this.

## Bulk delete: ticket number search and ticket links

Requested by the operator, for every ticketing system, with the page unaware of which one it shows.

- **Search by ticket number.** A new search term. Each system searches for it its own way and then keeps only the ticket with exactly that number: Zammad's `number:` field (exact, observed in the lab); in Zendesk the ID is searched as text, because the deletion connection may only call the search endpoint, and that boundary was kept. A number the search does not return finds nothing, so nothing unintended can be selected.
- **A link to each ticket**, in the ticket's row ("Open in Zendesk" / "Open in Zammad"), opening the ticket in its system in a new tab. Each system builds the link from the configured instance, never from an API answer.
- **Evidence.** Lab (Zammad) and production (Zendesk): the number search found exactly the ticket asked for, and each link opened the corresponding ticket. Nothing was deleted in these checks.

## The usage report follows the profile's backup and commercial systems

Applies the November 25 decision ([[Architecture and Decisions]]) to the usage report, which until now always read the first backup and billing systems directly, whatever the profile selected. The review and the usage report now read the same systems.

- **The seam.** The report's sources were split into a backup half (tenants and storage) and a billing half (subscriptions, billing lines, customer names). Each half comes from the profile's selection, through the same composition registries the Backup Aggregator uses. The report itself did not change.
- **The first systems are unchanged.** Their halves are the previous code split in two: the same reads in the same order with the same messages. A profile without `backup` or `commercial` settings reads them as before; the existing tests pass unchanged, and every locally installed profile was checked to resolve to the same calls.
- **Duplicati as a usage source.** A customer's storage is its jobs' backup size at their destinations, as each server last reported it. A job that has never completed a backup holds none and is not counted, with a notice giving the count (counts only, no names). A completed job without a size, or a server that cannot be read, makes the figure unknown. Duplicati backs up no Microsoft 365, so that figure is unknown with that reason, never zero. (Decided by the operator.)
- **Dolibarr as a billing source.** Contract lines become billing lines (quantity, unit price, discount, amount excluding tax), numbered as the monitored review numbers them. Dolibarr states no unit.
- **One identity.** A Duplicati customer's identity, used by both modules, is now defined once, in the provider's own mapping.
- **The page names the systems read** ("Duplicati GB", "billed in Dolibarr"); system-specific wording (the billing system's own word for quantity, the backup portal's GB convention, a browser sign-in hint) appears only where it applies. The layering tests caught one slip during the work: the first systems' names had been placed in the module's provider-neutral code; they were moved to its production adapter.
- **A selected system the report cannot read** is reported as unavailable, never replaced by another.

**Evidence.** Full suite green on Windows (1,761 tests). In the lab, a production-mode report matched the lab's two backup customers to their billing customers across differing names (a legal suffix), compared billed quantities with Duplicati's storage (the lab's backups are a few KB, so the differences are the billed quantities), showed Microsoft 365 as unknown with its reason, and left out the customer with no backup line.

## Open

- Merging `feature/zammad-provider` into `main`, and a release so other operators get the bulk delete features (owner's decision).
- How the lab stands in for the production backup system's usage reads is still undecided.
- The storage-conversion discrepancy seen during a demo stays parked.
