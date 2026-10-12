# Compliance Module — Discovery

**Status:** Discovery / requirements not finalized  
**Implementation:** Deferred pending operational and technical investigation  
**Last reviewed:** 2026-10-12

## Objective

Explore a provider-independent compliance monitoring capability for SupportToolkit. The intended outcome is to help operators identify, investigate, and track device compliance observations across authorized managed environments.

This document describes research questions, not an approved production design.

## Discovery tracks

### 1. Operational domain

Establish how compliance-related work should move from detection through investigation, follow-up, verification, and closure.

Questions to investigate:
- What is the agreed scope of monitoring?
- Which observations constitute actionable issues?
- How are transient, stale, unknown, and persistent observations distinguished?
- What are the responsibilities and handoffs at each stage?
- How are follow-ups, exceptions, and resolutions documented?
- Which decisions belong in existing ticketing systems rather than SupportToolkit?
- Which lifecycle rules are formally defined, and which still require stakeholder decisions?

Gather evidence from approved, sanitized process descriptions and representative case patterns. Distinguish observed behavior, documented policy, assumptions, and proposed changes.

### 2. Technical domain

Investigate available Microsoft Intune, Defender, and Microsoft Graph data and authorization models.

Questions to investigate:
- Which API resources expose device, policy, setting-level, and security observations?
- What identifiers remain stable across data sources and collection runs?
- How do evaluation timestamps, device check-in times, and reporting delays differ?
- What delegated or application permissions and cross-tenant authorization are required?
- What collection methods are supported in the approved execution environment?
- How can partial failures, inaccessible tenants, and stale data be represented without implying compliance?
- What are the API limits, pagination behavior, and operational constraints?

Use synthetic fixtures and public API schemas for development. Perform live authorization checks only in approved environments.

## Preliminary conceptual boundaries

These are research concepts, **not yet finalized domain entities**:

- **Observation:** A source-reported technical state at a point in time.
- **Issue:** A potentially actionable condition derived from one or more observations.
- **Case:** The operational handling of an issue, potentially represented by an existing ticket.
- **Resolution:** A verified outcome, not merely absence from a subsequent response.
- **Collection health:** Whether data is available and sufficiently current to support conclusions.

Do not conflate technical compliance state with operational case status.

## Candidate architecture (uncommitted)

- Pluggable data providers
- Normalized observations
- Comparison and reconciliation logic
- Read-only reporting
- Optional integration with existing ticketing workflows

Provider selection, persistence, identity rules, case lifecycle, and alerting behavior remain open decisions.

## Evidence and decision log

For each discovery finding, record:
1. Question being investigated
2. Sanitized evidence or publicly documented reference
3. What is confirmed
4. What remains uncertain
5. Consequences for requirements
6. Whether a stakeholder decision is needed

Do not publish customer records, real ticket excerpts, internal configuration, access tokens, tenant IDs, employee identities, or non-public security/process details.

## Exit criteria for discovery

Before implementation:
- Monitoring scope and intended users are understood.
- Representative end-to-end workflows have been mapped and reviewed.
- Technical data sources and their limitations have been validated.
- Entity identity and state-transition assumptions are documented.
- Boundaries between monitoring, investigation, and ticketing are agreed.
- An initial, narrow MVP can be stated in testable terms.

## Immediate next actions

- Map representative compliance lifecycles using sanitized findings.
- Identify unresolved ownership and follow-up decisions for stakeholders.
- Validate approved Graph access and available data shapes.
- Revisit the module architecture only after the evidence supports it.
