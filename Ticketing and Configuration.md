# Ticketing and Configuration

## Shared ticketing
Ticketing is not its own operational module. BackupAggregator generates ticket drafts, previews and submits through Zendesk. Submission requires explicitly enabled external-write permission; interactive submission also asks for confirmation.

```cmd
dotnet run --project src\SupportToolkit -- backup-aggregator ticket preview
dotnet run --project src\SupportToolkit -- backup-aggregator ticket submit
```

## Configuration
Named profiles, fixture/production modes, first-run wizard, provider connections and Windows Credential Manager for secrets.

```cmd
dotnet run --project src\SupportToolkit -- configure
dotnet run --project src\SupportToolkit -- configure status
dotnet run --project src\SupportToolkit -- configure use local-dev
```

Debug configuration: repo-root `local.config.json`. Portable release: `%LOCALAPPDATA%\SupportToolkit\local.config.json`. `SUPPORTTOOLKIT_CONFIG` overrides the path.

## Distribution
Self-contained Windows x64 ZIP via `scripts\publish-windows.cmd`. Preserve developer `dotnet run` workflow and make fixture onboarding approachable for non-developers. Verify extraction on a clean Windows installation.

Related: [[Architecture and Decisions]].
