# October 22, 2026 — Documentation checkpoint

**Status:** Documentation refresh. No code-repository Git action is implied; the SupportToolkit changes described here are local edits for the user to review and commit.

## Why this checkpoint exists

The README in the code repository was last substantially rewritten on October 17, before `v0.2.3` and `v0.2.4` were released and before the Compliance and Business Central work reached their current branch state. The Notes had the same problem: several pages still described October 12–15 as the present, and a few listed documentation work as outstanding that had already been done.

This checkpoint records what was out of date, what was changed, and where the evidence for each statement comes from.

## Evidence used

| Source | Used for |
|---|---|
| `main` at `83a7bf1` (October 21) | Implemented commands, configuration behavior, release workflow, tag history |
| Tags `v0.1.0` to `v0.2.4` | Release contents and dates |
| `feature/compliance-investigation` at `3953c80` | `compliance` command surface and file layout |
| `feature/business-central-monitoring-scope` at `6513b79` (October 21) | Business Central provider, `scoped-review` and `dev` commands |
| Existing Notes | Rationale, validation boundaries, operator-reported results |

This is a source read, not a test run. No `dotnet test` result was produced for this checkpoint, so the `251` figure still applies only to the `v0.2.3` stabilization checkpoint.

## What was stale, and what changed

### SupportToolkit README (`main`)

| Gap | Change |
|---|---|
| No release identity | Added the current release (`v0.2.4`) and a release history table with a reason for each release. |
| No command summary | Added a command overview table including external effect per command. |
| `v0.2.4` change undocumented | Added a Microsoft 365 workload accounting section describing the narrow Team/Group rule and its limits. |
| Retired `allowWrites` undocumented | Added a note: legacy files still load, the property is dropped on save, `SUPPORTTOOLKIT_ALLOW_WRITES` is ignored. |
| Project status was generic | Replaced with a status section that separates Acronis production reads, the personal Zendesk trial and the unvalidated organizational Zendesk path, and states that deployment, scheduling, persistence and operator UI are deliberately undecided. |
| Feature branches invisible | Added a development branches section that states each branch's evidence boundary. |

### `local.config.example.json`

The example still contained the retired `allowWrites` field and had no Zendesk destinations or `zendeskDestination`, so it did not demonstrate the routing model. It now shows two named destinations on one connection and a profile that selects one. All values remain fictional. This closes the long-standing Roadmap item about the README and example configuration.

### Notes

- **Home**: now lists the pages and points to the newest checkpoint.
- **Project Overview**: implemented list updated to `v0.2.4`, branch work separated from `main`.
- **Development Timeline**: added October 21–22 and removed the obsolete "README needs updating" limitation.
- **Roadmap**: closed the documentation items and marked earlier backlog entries that are now done.
- **Modules/BackupAggregator** and **Modules/Compliance Monitor**: recorded the current branch command surface, including the Business Central provider that now exists on its branch.
- **Architecture and Decisions**: added decisions made visible by this review.

## Findings worth keeping

1. **The Business Central branch is further along than the October 15 notes say.** Commit `6513b79` adds a real read-only Business Central provider (delegated sign-in, API and authority hosts pinned in code, read-only handler, OData paginator), a `dev bc-subscriptions --production` diagnostic that prints aggregate counts only, and an opt-in `backup-aggregator scoped-review`. This is **source-confirmed code on an unmerged branch**. Notes contain no record of a validated production read, so production access, entitlement semantics and identity mapping remain unverified.
2. **Production `scoped-review` fails closed today.** The default identity source is an explicit "unavailable" implementation, and the workflow checks it before any sign-in or API call, so no production scoped run can proceed until a validated identity source exists. That matches the existing guardrail: do not wire commercial filtering to fixture results.
3. **The Compliance branch has two different operating modes.** `compliance report` runs on in-process synthetic fixtures and needs nothing else. `compliance list` and `compliance investigate` talk to a local Dev Proxy on `127.0.0.1:8000` and only accept synthetic device names (`STK-SYNTHETIC-` prefix). Neither performs live Graph authentication.
4. **The configuration example still documented a retired setting.** `allowWrites` was retired in `v0.2.3` (October 17), but the example kept showing it, with `true` on the Zendesk profile. Stale examples get copied by operators, so they are worth treating as part of the safety surface.
5. **Group routing is not a security boundary.** The updated example uses two destinations to show the model; a Zendesk group routes and assigns tickets, it does not isolate them.

## CLI command catalogue

A reference catalogue of every command, option, exit code and environment variable now lives in the Claude project space as `CLI Command Catalogue`. It is derived from source, separates `main` from branch-only commands, and is intended to be refreshed whenever a command is added or changed. It is not duplicated here to avoid two competing versions.

## Decisions

- Keep the README as the operator-facing source for released behavior, and the catalogue as a quick-reference. Do not let either describe branch-only features as released.
- Describe feature branches in the README only with their evidence boundary. Fixture output must not read as production validation.
- Record reasons next to release notes, not just changes.

## Still open

- No fresh `v0.2.4` test run is recorded.
- Organizational Zendesk end-to-end validation remains pending.
- Business Central read access, entitlement rules and customer identity mapping remain unresolved.
- Compliance live Graph access and completeness remain unverified.
- The sanitized BackupAggregator case study has not been written.

Related: [[Strategy Checkpoint - 2026-10-20]], [[Roadmap]], [[Architecture and Decisions]], [[Public Documentation Safety]].
