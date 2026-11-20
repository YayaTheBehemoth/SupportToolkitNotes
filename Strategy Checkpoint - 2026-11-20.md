# November 20, 2026 — Strategy checkpoint: provider-agnostic proof and a Linux lab

**Status:** Design decisions and roadmap guidance; not authorization to change the SupportToolkit code repository.

This checkpoint replaces the priorities in [[Strategy Checkpoint - 2026-10-20]]. The decision to defer a universal deployment architecture still stands.

## Direction (decided November 19)

The point of SupportToolkit is its **provider-agnostic architecture**, not any particular vendor. The existing providers (Acronis, Zendesk, Business Central) stay as they are. The architecture is tested by adding **second providers** behind the existing seams, using free, self-hosted vendors in a private lab.

**Planned order:**
1. **osTicket** (ticketing), behind the ticket sink that Zendesk fills today.
2. **Duplicati** (backup), behind the Backup Aggregator's sources.
3. **Dolibarr** (commercial entitlement), behind the monitoring scope and the usage report's billing source.
4. **Fleet + osquery** (compliance).

**Measure for each new provider:** the number of files changed outside the new provider's folder, plus any layering rules added. The goal is for that number to trend toward zero from one provider to the next. A high number is a finding about the seams, not a failure.

**Lab rules:**
- The lab runs in GitHub Codespaces (Linux), one vendor container at a time, using `docker compose` inside the dev container.
- Real vendor instances are used only to discover their API contracts. Daily development runs against recorded fixtures, which are synthetic and sanitized like every other fixture.
- Nothing from the lab touches work systems, and lab credentials stay in Codespaces secrets.

## Known coupling to address in later milestones

Found in a source read on November 19 and left alone on purpose. Each provider milestone decides which of these it must break:

- Profile configuration holds `ZendeskConnection` and `ZendeskDestination`, and `Core/Configuration` names Zendesk in about eight files.
- The composition always uses the Zendesk backup ticket sink.
- The backup review's tenant result counts resource kinds shaped like Acronis's (Microsoft 365, devices).
- `Domain/MonitoringScope` uses Business Central's vocabulary (smart subscriptions, item numbers).

## Prerequisite done: Linux support (November 19–20)

Branch `feature/linux-dev`, not yet merged into `main`. Verify against the source repository before relying on any of this.

- SupportToolkit builds and tests on Linux as well as Windows: the same full test suite passed in a GitHub Codespace and on Windows.
- **Secrets:** the secret store is chosen per platform in `Composition/`.
  - On Windows, secrets stay in the Windows Credential Manager.
  - Elsewhere, they are read from environment variables named `SUPPORTTOOLKIT_SECRET__<PROVIDER>__<CONNECTION>__<SECRET>` and never stored, because a plaintext secret on disk is not acceptable.
  - Connection names that would map to the same variable are refused, rather than risk sending one connection's secret to another.
  - The configure wizard names the variable instead of asking for a secret it can't store.
- **Dev container:** `.devcontainer/` holds the .NET 10 SDK on Ubuntu, Docker inside the container, and the C# Dev Kit.
- **CI:** builds and tests on Ubuntu as well as Windows. The release stays Windows-only.
- **Web UI:** stays loopback-only. On a codespace or a remote Linux machine it is reached through a localhost tunnel that keeps the same port (`ssh -L`, or VS Code or `gh codespace ports forward`). Reaching it directly over a network would be a separate hosting decision, which hasn't been made.
- **Zendesk deletion sign-in:** where no browser can be opened, it shows the sign-in link instead of failing.

## Working agreements

- The source repository's `CLAUDE.md` now holds rules only. Current state comes from Git and the code, and priorities come from the newest strategy checkpoint in this vault.
- **Git in the code repository:** each operation is asked for and approved one at a time. Merges, tags, releases, pushes to `main` and CI changes happen only on the owner's request.
- Plans and handoffs may be challenged with reasons. Agreed refinements replace the original plan and are recorded.

## Still open from earlier checkpoints

The organizational validation items in [[Roadmap]] remain open: Zendesk bulk delete end-to-end, organizational Zendesk routing, and Business Central entitlement semantics. They run alongside the lab and are not replaced by it.

Related: [[Roadmap]], [[Architecture and Decisions]], [[Public Documentation Safety]].
