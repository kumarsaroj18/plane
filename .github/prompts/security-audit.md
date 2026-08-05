Audit this repository for security problems.

Scope your attention to application code under `apps/` and `packages/`. This is a
fork of Plane, an open-source project management tool with a Django API in
`apps/api` and React apps in `apps/web`, `apps/admin`, and `apps/space`.

Look for:

1. Secrets or credentials committed to the repository.
2. Injection risks — SQL, command, or template injection, especially in `apps/api`.
3. Authentication and authorization gaps in Django views and DRF permissions.
4. Unsafe deserialization or unvalidated user input reaching a sink.
5. Cross-site scripting risks in React code, especially `dangerouslySetInnerHTML`.
6. Overly permissive CORS or cookie settings.

For each finding, give the file path, line number, the attack scenario in concrete
terms, and a fix. Do not report theoretical issues with no reachable path. Rank
findings most severe first.
