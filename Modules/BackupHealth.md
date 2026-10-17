# BackupHealth

**Status: implemented.**

Acronis health workflow: retrieve resource and diagnostic information, normalize a snapshot, evaluate exceptions and present results in the console. The source README documents a **48-hour stale-backup threshold**.

```cmd
dotnet run --project src\SupportToolkit -- backup-health
```

Supports fixture and production modes.

## History
The first major integration module established API access, OAuth handling, read-only transport, fixtures, logging and module boundaries.

## Provider contract audit — October 17, 2026

During the separate `audit/acronis-provider-contracts` working branch, the shared Acronis cursor paginator gained bounded traversal (500 pages) and repeated-cursor/cycle detection. Local build and focused/full test suites passed; this is **not** a release/merge claim.

The legacy numeric-ID-to-UUID mapping traversal was changed to **fail**, rather than warn-and-skip, on child records with an unusable numeric ID or UUID. A failed traversal must not be cached as complete. The focused and full local tests passed. This safeguard follows from the mapping's requirements; malformed live responses were **not** observed.

An opt-in, read-only pre-deserialization diagnostic then observed one authorized production hierarchy run, reporting only allowlisted field types and counts (no tenant names or identifiers):

- One root response: `id` numeric, `uuid` string.
- One children response: `items` array with **39 object records** and no non-object rows.
- All 39 children had numeric `id`, string `uuid`, and numeric `has_children`. No missing/null/other types were detected for these inspected fields.
- All 39 `has_children` values were **0**. The provider followed no nested branches in this run; this does **not** validate nested traversal against live nested hierarchies.
- No JSON decoding or inspection failures; the mapping traversal reported completion.
- The BackupHealth evaluation finished with **37 resources**, **19 exceptions** and **43 diagnostics**. Diagnostic causes were not examined here and should not be attributed to hierarchy defects without further evidence.

**Evidence-based decision:** Observed structural types support the current DTO declarations; no type changes indicated. Subsequent local implementation added `[JsonRequired]` for `has_children`, rejected explicit null `items` collections, null child elements, and negative `has_children` values, while retaining valid empty collections and positive-branch traversal. The temporary diagnostic and its tests were removed. The user confirmed that local build, focused tests, and full test suite were green. These safeguards have **not** been independently exercised against malformed production responses. Nested production traversal also remains unverified. The user subsequently pushed these permanent safeguards with the shared pagination changes to the SupportToolkit audit branch in code commit `bacda31` (not merged or released).

### Account Management v2 tenant discovery — October 17, 2026

An explicitly enabled, read-only metadata diagnostic observed the response from `/api/2/tenants` during an authorized tenant inventory review. It inspected pre-deserialization JSON without outputting tenant/customer names, UUIDs, cursor tokens, or payload values. Local build and focused/full tests of the **diagnostic implementation** passed, but those synthetic tests did not establish the vendor contract.

- The API returned **one page with 81 tenant objects**, zero non-object entries; the collection completed.
- All **81 identifiers were UUID strings**, with no invalid or duplicate IDs. Every name was a nonblank string, and every `kind` was present as a string.
- **39** records had `kind="customer"`; **42** had another nonblank string kind. The breakdown of the other 42 kinds was not collected.
- Page `items` was an array; `paging` and `paging.cursors` were objects. Terminal `paging.cursors.after` was **explicitly null**, not absent. The timestamp field was a string; the existing DTO decoded the page successfully.
- `parent_id`, `created_at`, and `updated_at` were strings on all 81; `version` numeric, `enabled` Boolean, `contacts` and `offering_items` arrays; `deleted_at` null on all 81; `customer_id` **absent** on all 81.
- The independently observed legacy hierarchy returned **39 direct children** in a separate workflow. The matching numerical count of 39 v2 `customer` records is suggestive but **not an identity join or proof of inventory completeness**.

**Membership-validation follow-up:** Using the permanent, explicitly production-enabled `dev acronis-tenants` command (customer names visible only with `--names`), the operator manually compared the full customer-name listing against the Acronis Management Portal and confirmed **all 39 customers present in the web view were identified by SupportToolkit**. A separate portal Network inspection showed `/api/2/tenants/{uuid}/children?order=name&allow_deleted=true` returned **39 direct `items`** for the inspected hierarchy, with an empty terminal `after` cursor. This independently corroborates the v2 `subtree_root_id` result, though identity-by-UUID comparison was not performed and the portal uses the same vendor backend family.

**Decision:** Customer discovery is validated for the observed production hierarchy by count and manual name-level membership. No observed JSON shape requires a DTO type or customer-kind selection change. One-page behavior does **not** establish live multi-page continuation semantics; the shared paginator's safeguards have synthetic coverage. The reusable developer command is explicitly read-only and opt-in; the temporary raw-page shape probe and associated changes in the provider/paginator are to be removed after comparison, retaining the permanent command and prior hardening.

## Questions
- Which resource types and filters are comparable to portal views?
- How should missing, stale, failed and unknown activity be distinguished?
- What explains historical API-versus-portal count discrepancies?

Related: [[BackupAggregator]], [[Architecture and Decisions]].
