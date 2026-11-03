# November 3, 2026 — Documentation checkpoint: v0.4.0 and the local web UI

**Status:** Documentation refresh. No code-repository Git action is implied.

## Evidence used

| Source | Used for |
|---|---|
| `main` at `e788796` (tag `v0.4.0` at `73b786b`, November 3) | Released behavior, README (Web UI, Zendesk bulk delete, release history, project status), source layout, `LayeringTests` |
| Commits `v0.3.0..main` | What was merged and when |
| Session handoff, November 3 | Operator-reported test counts and validation state |

This is a source read, not a test run. Test counts below are **operator-reported**: 1,386 passing on `main`, 1,460 on `feature/compliance-investigation` (which has `main` merged in).

## What changed since November 2

- **`v0.4.0` released (November 3).** All feature branches except `feature/compliance-investigation` were merged and deleted, including `feature/scope-visibility-wait-usage-probe` (in-progress wait, stray-tenant check, `dev acronis-usage`, `dev usage-compare`).
- **BackupHealth removed** in the same change that added Zendesk bulk delete (`53a0d24`).
- **Zendesk bulk delete implemented** (`dev zendesk-bulk-delete` and a web page). See [[Modules/Zendesk Bulk Delete]].
- **UI-agnostic ticketing and a layering test** (`9100fd3`): ticket preparation returns typed results, `BackupReviewSession` owns the review rules, and `LayeringTests` forbids console, `Ui`/`Cli` references and UI frameworks below the UI layer.
- **Local web UI** (`SupportToolkit serve`, Blazor Server inside the same executable) with Profile, Backup Aggregator and Zendesk bulk delete pages, and a Simple / Detailed view.
- **`configure import`** moves a handed-over settings file into place, keeps existing settings as a dated backup, and asks for the secrets the connections need. Release zip gains `Start SupportToolkit.cmd` and `Import config.cmd`.
- **Saving keeps unrecognised settings** instead of dropping them; only deliberately retired settings are removed.

## Decision — November 3: a local web UI inside the CLI, loopback only

**Supersedes** the November 2 decision to build a separate WPF project hosting only the deletion module.

| | November 2 plan | November 3 implementation |
|---|---|---|
| Technology | WPF, separate project | Blazor Server inside the existing executable (`serve`) |
| Scope | Zendesk deletion only | Profile, Backup Aggregator, Zendesk bulk delete |
| Platform split | Deferred | Still deferred; layering enforced by `LayeringTests` instead of project boundaries |
| Distribution | Second publish target | Same zip; `wwwroot` and a manifest ship beside the executable |

**Reason recorded in the release notes:** the console was hard to hand to colleagues, and both interfaces now sit on the same session rules so they cannot diverge. A browser UI also avoids the WPF trade-off of losing the console, and needs no second executable.

**Security model (README, source-confirmed):**
- Listens on `127.0.0.1` on a free port; no option to listen elsewhere.
- A random launch token in the printed link is exchanged for an `HttpOnly`, `SameSite=Strict` session cookie; requests without it, including the live connection, are refused.
- `Host` must be the loopback address and port (DNS rebinding); state-changing requests and WebSockets must come from the UI's own origin.
- Nothing is loaded from other sites; secrets never reach the browser; request URLs are not logged because they carry the token.
- One operator; production use stays Windows-only because secrets come from Windows Credential Manager.

**Relation to the October 20 deferral:** a common UI now exists for two modules, which goes further than the November 2 "one module" limit. Hosting, scheduling, persistence and multi-user operation are still not decided; the web UI is a local, single-operator presentation layer over the same sessions as the console, not a hosted platform. The README project status still describes the operator interface as "intentionally undecided"; that sentence now understates the state and should be reworded by the code owner.

## Validation state

| Area | State |
|---|---|
| Zendesk bulk delete, including real deletion through the web UI | Personal sandbox only, with test tickets. No organizational use. |
| Domain search (`requester:*@domain`) | Unverified; Zendesk documents that wildcards do not work for email searches, so it may return nothing. |
| Backup Aggregator web page | Not run in production mode. |
| Hidden-typing secret prompt in `configure import` | Tested only non-interactively. |
| Web UI security | Source-confirmed and covered by tests; no independent review. |

## Still open

- Organizational end-to-end validation of Zendesk ticket creation and bulk delete.
- Whether the Zendesk audit log attributes PKCE (API) deletions to the signed-in operator: not recorded in Notes.
- Whether a public client's leftover secret can obtain a token through the client-credentials grant: not recorded in Notes.
- The `v0.4.0` README release row omits the `dev acronis-usage` and `dev usage-compare` diagnostics.
- Leftover sandbox test organizations and people (tag `st_seed`) can be removed.
- The CLI command catalogue in the Claude project space predates `v0.4.0`.

Related: [[Home]], [[Architecture and Decisions]], [[Modules/Zendesk Bulk Delete]], [[Roadmap]], [[Development Timeline]].
