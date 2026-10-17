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

**Evidence-based decision:** Observed structural types support the current DTO declarations; no type changes indicated. Subsequent local implementation added `[JsonRequired]` for `has_children`, rejected explicit null `items` collections, null child elements, and negative `has_children` values, while retaining valid empty collections and positive-branch traversal. The temporary diagnostic and its tests were removed. The user confirmed that local build, focused tests, and full test suite were green. These safeguards have **not** been independently exercised against malformed production responses. Nested production traversal also remains unverified. Changes remain in the local SupportToolkit working tree until the user chooses a code-repository Git operation.

## Questions
- Which resource types and filters are comparable to portal views?
- How should missing, stale, failed and unknown activity be distinguished?
- What explains historical API-versus-portal count discrepancies?

Related: [[BackupAggregator]], [[Architecture and Decisions]].
