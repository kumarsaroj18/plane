Assess this API's versioning strategy and whether it is applied correctly.

`apps/api/plane/urls.py` currently mounts endpoints at `api/` and `api/public/`
with no version segment (no `api/v1/`) visible at the root. The project uses
`drf-spectacular` for schema generation (see the `Spectacular*View` imports in
`apps/api/plane/urls.py`).

1. Confirm whether any version prefix or header-based versioning exists anywhere
   in `apps/api/plane` (check `urls.py` files under each app, and DRF settings
   for a `DEFAULT_VERSIONING_CLASS`). State plainly if there is none.
2. If there is no versioning scheme, identify what happens today when a
   breaking change is needed — is there any convention (deprecation headers,
   parallel endpoints, a changelog) that substitutes for real versioning?
3. Check `drf-spectacular`'s generated schema for any version metadata, and
   whether the OpenAPI spec exposed at the `Spectacular*View` routes reflects
   a single unversioned API surface.
4. If mobile or third-party API clients exist in this monorepo or are referenced
   in docs, note whether they pin against a specific API shape that an
   unversioned API could silently break.

Report your findings plainly: what exists today, the concrete risk of shipping
breaking changes without versioning, and — only if asked for a recommendation —
a minimal versioning approach that would fit this codebase's existing URL
structure. Do not implement anything.


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
