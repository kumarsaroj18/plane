Review the current state of this repository for code quality.

Follow the conventions in `AGENTS.md`: camelCase for variables and functions,
PascalCase for components and types, strict TypeScript, oxlint via `.oxlintrc.json`,
formatting via oxfmt.

Focus on:

1. Correctness — logic errors, unhandled edge cases, incorrect async handling.
2. Error handling — try/catch with proper error types, per `AGENTS.md`.
3. Readability — naming, function length, unclear control flow.
4. Consistency — code that diverges from surrounding patterns in the same package.

Report findings grouped by severity. For each finding give the file path, the line
number, what is wrong, and a concrete suggested fix. If you find nothing
significant, say so plainly rather than inventing minor nits.
