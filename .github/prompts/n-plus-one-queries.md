Find N+1 query problems in the Django backend under `apps/api/plane`.

Look at views, serializers, and viewsets — particularly list endpoints in
`apps/api/plane/app/views` that serialize collections (issues, projects, workspace
members, comments). For each serializer field that traverses a foreign key,
many-to-many, or reverse relation:

1. Does the queryset use `select_related` for each forward foreign key the
   serializer touches, and `prefetch_related` for each reverse relation or
   many-to-many?
2. Does a serializer method field (`SerializerMethodField`) run a query per
   object rather than using an annotated or prefetched value?
3. Are there loops in view code that call `.save()`, `.get()`, or iterate a
   related manager once per item in an outer queryset?
4. Do paginated list views apply the same optimizations as their unpaginated
   counterparts, or did an optimization get missed on one but not the other?

For each finding, give the file path, line number, the query count you'd expect
under load (e.g. "1 + N where N is the number of issues returned"), and the
concrete `select_related`/`prefetch_related`/annotation fix. Prefer findings you
can support by reading the serializer's declared fields against the view's
`get_queryset()` — don't speculate about code you have not read.


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
