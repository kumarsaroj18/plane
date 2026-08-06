Enumerate and assess this API's unauthenticated (public) endpoints.

`apps/api/plane/urls.py` mounts `plane.app.urls` at `api/` and `plane.space.urls`
at `api/public/` — the `space` app is the intended public surface, used for
things like shared views and public project pages. Anything reachable without
authentication should live there deliberately, not by omission elsewhere.

1. List every view whose permission class is `AllowAny` (or has no permission
   class, which DRF treats as open) across `apps/api/plane/app/views` and
   `apps/api/plane/space/views`. Note the file, the URL it's mounted at, and
   which app package it's in.
2. For each one, does it belong in `api/public/`, or is it an authenticated
   endpoint that lost its permission class by accident?
3. For endpoints correctly public, do they leak more than the public feature
   requires — internal IDs, other users' data, workspace details not meant for
   anonymous visitors?
4. Check `apps/api/plane/web/urls.py` (`robots.txt`, health check) separately —
   these are expected to be public; don't flag them as findings, just note them
   as already-reviewed.

Present the results as a table: endpoint path, view class, file, and whether it
appears intentionally or accidentally public. Do not modify any permission
classes — report only.


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
