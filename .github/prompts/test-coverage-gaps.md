Identify the most valuable missing tests in this repository.

Backend tests live under `apps/api/tests` and run with pytest inside the stack
defined by `docker-compose-test.yml`. See `apps/api/tests/TESTING_GUIDE.md` for
conventions and fixtures.

Rank by risk rather than by raw coverage percentage. Prioritize:

1. Code paths handling authentication, permissions, or access control.
2. Data mutation paths where a bug corrupts or destroys user data.
3. Complex conditional logic with many branches and no corresponding test.
4. Recently changed code with no accompanying test.

For each gap, name the file and function, explain what breaks if it regresses, and
sketch the specific test case to add including the inputs that matter. Propose at
most ten, ordered by risk. Do not write the tests.


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
