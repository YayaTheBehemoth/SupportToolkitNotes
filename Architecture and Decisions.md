# Architecture and Decisions

## Current architecture
```text
src/SupportToolkit/
  Program.cs
  Cli/Commands/
  Ui/Interactive/
  Core/{Configuration,Secrets,Ticketing,Logging,ErrorHandling,Modules}/
  Modules/{BackupAggregator,BackupHealth}/
  Providers/{Acronis,Zendesk}/
tests/SupportToolkit.Tests/
```

The host dispatches operational modules through `ISupportToolkitModule`. Configuration is host-level functionality, not a module. Ticketing is shared infrastructure attached to operational modules, not a standalone main-menu item.

## Decisions
| Decision | Reason | Status |
|---|---|---|
| Modular C#/.NET console | Extensibility and maintainability | Implemented |
| Fixture/production modes | Development without production dependency | Implemented |
| Restricted Acronis read transport | Reduce accidental mutations | Implemented |
| Windows Credential Manager | Keep secrets out of plaintext config | Implemented |
| Gated external ticket writes | Separate monitoring from side effects | Implemented |
| Shared ticketing | Avoid duplicate provider logic | Implemented |
| Independent compliance aggregation | Avoid Lighthouse as a required dependency | Investigating |

## Lessons
- Activity lifecycle state `completed` does not mean successful result.
- Deduplicate by stable resource IDs and meaningful timestamps.
- Resource inventory objects may not map 1:1 to physical devices or portal rows.
- Multi-tenant jobs need failure isolation and explicit incomplete states.
- A first-run wizard matters when handing tools to non-developers.

See [source README](https://github.com/YayaTheBehemoth/SupportToolkit/blob/main/README.md).
