# Compliance Monitor

**Status: discovery / proposed, not implemented.**

## Problem
Manually inspect noncompliant devices across customer tenants and document tenant, device, reason and next action. Lighthouse CSV exports provide limited views.

## Desired output
Tenant-grouped exceptions with device identity, compliance reason, evidence timestamp, last check-in and triage category. Missing evidence must never be silently classified as compliant.

## October 2026 discovery
- Microsoft Graph application registration permissions and consent remain a bottleneck.
- Managed-tenants requests returned `400 Request not applicable to target tenant`; exact cause is unresolved.
- Presence of a delegated `scp` claim does not establish eligibility or application authorization.
- Lighthouse web UI was inaccessible for multiple operators on several occasions, including October 9 and 12. This is an observation, not proof of a Microsoft-wide outage.
- Direction: deprioritize Lighthouse as a required dependency, while investigating **authorized** direct Graph/Intune reads.

## Discovery plan
1. Establish identity and effective delegated/application permissions with an authorized administrator.
2. Test a minimal permitted read in an approved test tenant.
3. Validate tenant enumeration, device inventory and compliance details separately.
4. Design synthetic tenant/device/exception fixtures.
5. Build a read-only collector and tenant-grouped report.
6. Distinguish unknown, inaccessible and incomplete from healthy.

Do not bypass tenant authorization or transfer production customer data to personal/public repositories.

Related: [[Roadmap]], [[Development Journal - 2026-10-12]].
