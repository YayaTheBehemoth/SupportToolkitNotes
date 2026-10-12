# BackupHealth

**Status: implemented.**

Acronis health workflow: retrieve resource and diagnostic information, normalize a snapshot, evaluate exceptions and present results in the console. The source README documents a **48-hour stale-backup threshold**.

```cmd
dotnet run --project src\SupportToolkit -- backup-health
```

Supports fixture and production modes.

## History
The first major integration module established API access, OAuth handling, read-only transport, fixtures, logging and module boundaries.

## Questions
- Which resource types and filters are comparable to portal views?
- How should missing, stale, failed and unknown activity be distinguished?
- What explains historical API-versus-portal count discrepancies?

Related: [[BackupAggregator]], [[Architecture and Decisions]].
