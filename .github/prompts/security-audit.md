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


## Output format

Keep the report short and skimmable — it's read as a GitHub Actions job
summary, not a document. Follow this shape exactly:

- Line 1: one-sentence verdict — total finding count and worst severity
  (e.g. "3 findings, 1 critical").
- One bullet per finding: `file:line` — issue in one clause — fix in one
  clause. No sub-bullets, no code blocks, no restating file contents.
- Skip preamble, skip restating this prompt, skip narrating files you
  checked and found clean.
- Cap at 10 findings, ordered most severe/valuable first. If you found
  more, say "N more found, showing top 10" — don't list them all.
- If there is nothing to report, say so in one line and stop.
