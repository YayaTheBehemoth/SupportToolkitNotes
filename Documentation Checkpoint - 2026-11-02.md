# November 2, 2026 — Documentation checkpoint

**Status:** Documentation refresh, plus decisions made on November 2. No code-repository Git action is implied.

## Why this checkpoint exists

Since the [[Documentation Checkpoint - 2026-10-22]], the Business Central monitoring-scope branch was merged and released as `v0.3.0`, a new feature branch appeared, and the Notes still described `v0.2.4` as current and monitoring scope as unmerged and failing closed in production. Separately, a new module (Zendesk bulk deletion) was scoped, a GUI decision was made for it, and BackupHealth was marked for retirement.

## Evidence used

| Source | Used for |
|---|---|
| `main` at `06f9327` (tag `v0.3.0`, October 27) | Released behavior, README release history and command list, source layout |
| `feature/scope-visibility-wait-usage-probe` at `2fea460` (October 30) | Branch contents (README diff and changed files) |
| `feature/compliance-investigation` at `8b9e51f` (October 26) | Branch state; no functional change since October 18, only merges from `main` |
| Design discussion, November 2 | Zendesk bulk deletion decisions, GUI decision, BackupHealth retirement |

This is a source read, not a test run. **No fresh `dotnet test` result is recorded for `v0.3.0` or `v0.2.4`.** The `251` figure still belongs to the `v0.2.3` checkpoint.

## What was stale

| Note | Was | Now |
|---|---|---|
| Home, Project Overview | `v0.2.4` current; monitoring scope on an unmerged branch | `v0.3.0` current; monitoring scope released |
| Modules/BackupAggregator | Production `scoped-review` fails closed for lack of an identity source | Automatic name-based customer-to-tenant matching runs on every run; interactive console offers the scoped review only |
| Architecture and Decisions | Tree without `Providers/BusinessCentral`, `Core/CustomerMapping`, `Core/MonitoringScope`, `Core/Reconciliation`; BC decisions "on unmerged branch" | Tree and statuses updated; name matching recorded as an adopted decision that supersedes the October 14 guidance to avoid it |
| Ticketing and Configuration | No submitter or requester settings | Explicit `submitterId`/`requesterId` per Zendesk connection (released in `v0.3.0`) |
| Roadmap | BC items open; no deletion or UI work | BC items updated; new items for deletion, GUI and BackupHealth removal |
| Development Timeline | Ends October 22 | October 26 – November 2 added |

## Findings worth keeping

1. **The October 14 identity guidance was overtaken by implementation.** Notes said to investigate shared references before adopting name matching. `v0.3.0` adopted conservative automatic name matching instead: unique strong matches only, ambiguous and conflicting cases left unmatched and visible, nothing stored. Accepting this is a recorded decision, not a silent drift.
2. **Production validation claims for `v0.3.0` come from the README, not from Notes.** The README states that Business Central reads and the automatic matching were exercised read-only against a live environment, and that the scoped review has had limited end-to-end validation. No detailed observation was recorded in Notes.
3. **The source README still says `Current release: v0.2.4`** in its introduction, although its release history lists `v0.3.0`. It also lists BackupHealth as a current module. Both are for the code owner to correct.
4. **The new branch adds a write-free Acronis usage endpoint to the read-only allowlist as an exact path.** That keeps the transport restriction intact, but it widens the allowlist and should be reviewed as such at merge.

## Decisions — November 2, 2026

- **Zendesk bulk deletion** becomes a SupportToolkit module, owned by the project author, interactive only, with per-user PKCE sign-in. Details in [[Modules/Zendesk Bulk Delete]].
- **GUI for one module, not for the platform.** A separate WPF project references the existing project and hosts only the deletion module. No library/host split for now; revisit if presentation code leaks into modules or a second module needs a GUI. This is consistent with the October 20 deferral: the need comes from one concrete, validated workflow.
- **BackupHealth is retired.** BackupAggregator covers the needed visibility. Removal is planned as a separate change before the deletion module. Keep the Acronis resource-status provider, which BackupAggregator uses, and keep existing configuration files with a `backup-health` profile entry loadable.

## Still open

- Fresh test run for `v0.3.0`.
- Organizational Zendesk end-to-end validation (ticket creation) remains pending.
- Zendesk deletion: permanence of API deletion for archived tickets, audit-log recording, localhost redirect port, agent delete permissions.
- Compliance live Graph access and completeness remain unverified.
- The sanitized BackupAggregator case study has not been written.

Full decision log: [[Development Journal - 2026-11-02]].

Related: [[Home]], [[Roadmap]], [[Architecture and Decisions]], [[Modules/Zendesk Bulk Delete]], [[Public Documentation Safety]].
