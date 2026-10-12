# Development Journal — 2026-10-12

## Focus
Compliance API feasibility, Lighthouse reliability and documentation.

## Observations
- Graph managed-tenant requests returned `400 Request not applicable to target tenant`.
- Delegated scope presence did not establish valid access to the target tenant.
- Lighthouse web UI failed for multiple operators, with similar trouble on October 9.
- Approval of Graph application permissions remains a dependency.

## Interpretation (tentative)
Lighthouse is not dependable enough to be a required dependency right now. An independent aggregation layer may be valuable, but still requires authorized access to underlying Graph/Intune data.

## Decisions
- Deprioritize Lighthouse-specific integration.
- Investigate direct, authorized Graph/Intune reads.
- Keep discovery findings distinct from implementation claims.
- Maintain public, sanitized Obsidian notes via GitHub.

## Next actions
- Validate minimum permissions and tenant relationship.
- Test one authorized read.
- Build synthetic fixtures and normalized compliance models.
- Continue validating backup modules independently.

Related: [[Compliance Monitor]], [[Roadmap]].
