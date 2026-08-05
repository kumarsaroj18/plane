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
