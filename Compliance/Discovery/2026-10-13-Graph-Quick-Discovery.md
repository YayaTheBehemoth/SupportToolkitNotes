# Compliance — Graph quick discovery

**Date:** 2026-10-13  
**Status:** Approved discovery plan — testing pending  
**Goal:** Determine whether direct Microsoft Graph access to Intune can replace Lighthouse as the source for the recurring device-compliance investigation.

> **Scope:** Documentation and approved read-only tests only. Use a permitted test/customer tenant; do not request expanded permissions or register applications without authorization. Do not commit tokens, tenant/device identifiers, responses or screenshots to public repositories.

## Before trying endpoints

- Identify an approved way to acquire a **Microsoft Graph token for a specific customer tenant**. Confirm tenant context, delegated vs application authentication, granted scopes/roles, and whether existing MSP delegation (e.g. GDAP) actually permits this API call. **Lighthouse portal access does not prove Graph Intune access.**
- If blocked by tooling, sign-in, consent, Intune licensing or permissions, **record the blocker and stop** rather than treating it as an endpoint failure. Basic Copilot access is not an authentication method.

## Endpoints to try — in order

| Priority           | Endpoint (Graph v1.0, tenant-scoped)                                             | Why                                                                                   | Follow-up                                                                                                                                                                                                                                        |
| ------------------ | -------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1                  | `GET /deviceManagement/managedDevices?$top=5`                                    | Verify that approved Graph access returns Intune managed devices for one tenant.      | Confirm tenant context, required `DeviceManagementManagedDevices.Read.All` permission, Intune license, pagination, and which device/compliance fields are available. If successful, examine `complianceState` and `lastSyncDateTime`.            |
| 2                  | `GET /deviceManagement/managedDevices?$filter=complianceState eq 'noncompliant'` | Check whether the existing noncompliant-device queue can be retrieved directly.       | Verify that the filter is supported in this environment; compare a small, authorized sample against the Intune portal, noting differences, stale device check-ins and paging. If filtering fails, record that separately from access failure.    |
| 3                  | `GET /deviceManagement/deviceCompliancePolicies`                                 | Determine whether Graph exposes the compliance policies behind device state.          | Requires `DeviceManagementConfiguration.Read.All`. If available, inspect a policy using `GET /deviceManagement/deviceCompliancePolicies/{policyId}/deviceStatuses` and check whether the results can be linked reliably to the observed devices. |
| 4 (only if needed) | `GET /deviceManagement/deviceCompliancePolicySettingStateSummaries`              | Investigate whether policy-setting summaries expose the **reason** for noncompliance. | If supported, inspect the nested `deviceComplianceSettingStates` relationship for setting name, state and error details. Establish whether this actually matches the per-device information shown in Intune; do not assume it does.              |

**Base URL:** `https://graph.microsoft.com/v1.0`  
**Testing rule:** Run against one authorized tenant first; only investigate a second tenant after confirming the first. Never assume a partner token can traverse all customers automatically.

## What to record (sanitized)

For each endpoint: **accessible / forbidden / unsupported / not attempted**; authentication type; permission requirement; useful fields returned (field names only); whether it answers *tenant → device → compliance state → cause*; and next blocker. Keep identifiers, device names, user information and raw response bodies out of public notes.

## End-of-day decision

- **Graph viable:** authorized read-only retrieval works and provides useful compliance data → plan deeper mapping and cross-tenant validation.
- **Partially viable:** device statuses work but causes or MSP-scale access remain unresolved → investigate that specific gap.
- **Blocked:** no approved token/consent/permissions/tooling → document the exact dependency and do not begin a Compliance provider implementation.

**Not tomorrow:** production ingestion, credential handling implementation, Compliance domain models, ticket automation or a Lighthouse replacement decision based solely on endpoint documentation.

## Microsoft documentation

- [List managed devices](https://learn.microsoft.com/en-us/graph/api/intune-devices-manageddevice-list?view=graph-rest-1.0)
- [Managed device properties](https://learn.microsoft.com/en-us/graph/api/resources/intune-devices-manageddevice?view=graph-rest-1.0)
- [List compliance policies](https://learn.microsoft.com/en-us/graph/api/intune-deviceconfig-devicecompliancepolicy-list?view=graph-rest-1.0)
- [List per-policy device statuses](https://learn.microsoft.com/en-us/graph/api/intune-deviceconfig-devicecompliancedevicestatus-list?view=graph-rest-1.0)
- [List compliance setting states](https://learn.microsoft.com/en-us/graph/api/intune-deviceconfig-devicecompliancesettingstate-list?view=graph-rest-1.0)
