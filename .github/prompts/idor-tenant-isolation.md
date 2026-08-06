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

This is read as a GitHub Actions job summary. Match this structure exactly —
it uses collapsible sections so the page stays short until someone expands a
finding:

1. **One verdict line**, first thing in the report: emoji + severity counts,
   e.g. `🔴 1 HIGH, 🟡 2 MEDIUM` — or `✅ No findings` if the audit is clean.
   Nothing above this line: no preamble, no restating this prompt.
2. **One collapsible block per finding**, most severe first, in exactly this
   shape (including the blank line after `<summary>`):

<details open>
<summary>🔴 <b>HIGH</b> — one-line title · <code>path/to/file.py:42</code></summary>

**What's wrong** — one to three sentences, concrete, no restating the code.

**Attack** — the exact request or scenario that exploits it. Omit this line
entirely for non-exploit findings (dead code, missing tests, versioning gaps).

**Fix**
```python
<the actual patch, not a description of one>
```
</details>

3. Only the single most severe finding gets `<details open>` (expanded by
   default). Every other finding uses `<details>` with no `open` attribute,
   so it renders collapsed and the reader clicks to expand.
4. Severity emoji: 🔴 HIGH, 🟡 MEDIUM, 🔵 LOW. Pick based on real exploitability
   and blast radius, not by inflating everything to HIGH.
5. Cap at 10 findings, most severe/valuable first. If you found more, add one
   line after the last block: "N more found, not shown."
6. If there is nothing to report, output only the verdict line and stop —
   no empty sections, no "I checked X, Y, Z and found nothing" narration.
