Scan this repository for committed secrets and credentials.

Look for: API keys, access tokens, private keys (`BEGIN.*PRIVATE KEY`), database
connection strings with embedded passwords, JWT signing secrets, OAuth client
secrets, cloud provider credentials (AWS/GCP/Azure key patterns), and hardcoded
passwords in code, config files, fixtures, or test data — not just `.env` files.

Pay particular attention to:

1. `apps/api/plane/settings/` and any `.env*` files that may have been committed
   despite `.gitignore`.
2. Test fixtures and seed data under `apps/api/tests` that might use a real-looking
   credential instead of an obvious placeholder.
3. Frontend code under `apps/web`, `apps/admin`, `apps/space` for API keys or
   tokens baked into client-side bundles (anything shipped to the browser is
   public regardless of intent).
4. Docker and CI config (`docker-compose*.yml`, `.github/workflows/*.yml`) for
   credentials that should be secrets/variables instead of literal values.

For each finding, give the file path, line number, and the credential type. Do
not report values that are clearly placeholders (`your-secret-here`,
`changeme`, `xxx`) — flag only strings that look like real, usable credentials.
If you find a real secret, do not repeat it verbatim in your report; reference
its location and type only.


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
