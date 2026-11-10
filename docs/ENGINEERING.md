# Enterprise Knowledge: engineering notes

Retrieval permissions must still hold after an answer has been cached or a link has been saved. Enterprise Knowledge applies authorization across retrieval, history, citations and exports.

## System path

~~~mermaid
flowchart LR
  Session --> TenantTransaction
  TenantTransaction --> PermittedSources
  PermittedSources --> Generation
  Generation --> RevisionRecheck
  RevisionRecheck --> Answer
  Answer --> ProtectedCitationAndExport
~~~

This diagram describes the demonstrated system boundary, not a production deployment topology.

## Decisions and tradeoffs

### Enforce tenant scope inside the database

A non-owner application role, forced row-level security and transaction-local identity keep pooled requests from inheriting another tenant's context.

### Recheck after the slow model step

Generation happens outside the database transaction. Authorization revision and readable sources are checked again before the answer is committed.

### Revoke every route to the content

The same permission change affects cached answers, direct citations, conversation history and exports—not only the initial search.

## Review questions

- What evidence demonstrates **enforce tenant scope inside the database**, and what would fail without it?
- What evidence demonstrates **recheck after the slow model step**, and what would fail without it?
- What evidence demonstrates **revoke every route to the content**, and what would fail without it?

## Verification boundary

11 database regressions and six browser checks passed in the current verification pass. Recorded identity switching and source-backed answer inspection. Documented denial behavior uses neutral not-found responses for inaccessible objects.

Revocation prevents later protected access; it cannot erase information someone already saw. Synthetic tests are not an external security audit.
