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
