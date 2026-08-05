Audit for insecure direct object reference (IDOR) and multi-tenancy isolation
failures — cases where one authenticated user can act on another user's or
another workspace's data by supplying a different ID in the request.

This is a multi-tenant Django REST Framework API under `apps/api/plane`. The
core scoping hierarchy is Workspace -> Project -> Issue (and related models).
Permission classes live under `apps/api/plane/app/permissions` and
`apps/api/plane/utils/permissions`; endpoint scoping happens both through URL
path segments (workspace slug, project ID) and through request payload fields.

For every view under `apps/api/plane/app/views` and `apps/api/plane/space/views`
that accepts an ID in the URL, query string, or request body (workspace ID,
project ID, issue ID, member ID, or similar), check:

1. Does `get_queryset()` filter by the workspace/project the authenticated user
   actually belongs to, or does it trust an ID from the request unfiltered?
2. Can a payload field (not the URL) override which workspace or project a write
   applies to, bypassing the permission class that gated the URL-level object?
3. Do nested resources (e.g. an issue comment, an issue activity) re-verify
   workspace/project membership, or only check that the parent issue exists?
4. Are `pk`/`id` lookups scoped with `.filter(workspace=..., project=...)` before
   `.get()`, or does an unscoped `Model.objects.get(pk=...)` let a valid ID from
   any tenant resolve?

For each finding, give the file path, line number, the exact request a malicious
authenticated user (member of a different workspace) could send, and what they
would gain access to. Do not report a finding where the permission class already
enforces workspace membership at the queryset level — verify by reading the
permission class, not just the view.


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
