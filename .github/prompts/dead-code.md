Find dead code in this repository: functions, components, exports, and Python
views/serializers with no remaining callers or references.

Check both sides of the monorepo:

1. TypeScript/React under `apps/web`, `apps/admin`, `apps/space`, and `packages/*`
   — exported functions, components, and types with zero imports elsewhere in
   the workspace. Use `workspace:*` and `catalog:` dependency declarations (see
   `AGENTS.md`) to tell internal package boundaries apart from external ones.
2. Python under `apps/api/plane` — views, serializers, or utility functions with
   no URL mapping and no call site outside their own module or tests.

Exclude from consideration: anything exported from a package's public entry
point (`index.ts`/`__init__.py`) purely for external consumers, Storybook
stories, and test files themselves.

For each finding, give the file path, the symbol name, and how you confirmed no
references exist (e.g. "grepped for `ComponentName` repo-wide, only match is its
own definition and export"). Rank by file size removed if deleted — largest
first. Do not delete anything. Report only.


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
