# Public Documentation Safety

This vault is public. Treat committed content as permanently publishable.

## Appropriate
Generic architecture, sanitized API discoveries, synthetic fixtures, design decisions, testing strategy and links to public source code.

## Exclude
Credentials, API tokens, customer/tenant identifiers, device and user names, production payloads, tickets, confidential procedures and unredacted screenshots.

Review staged changes before pushing. `.gitignore` is not a security boundary. Deleting an exposed secret in a later commit does not erase Git history; rotate affected credentials and address history.

The application README documents restricted Acronis read transport, Windows Credential Manager and explicit Zendesk write gating. These application safeguards do not replace provider permissions or organizational authorization.
