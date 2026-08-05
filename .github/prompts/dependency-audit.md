Audit dependency health for this monorepo.

The workspace is pnpm with a catalog in `pnpm-workspace.yaml`. External dependencies
use `catalog:` and internal packages use `workspace:*`, per `AGENTS.md`.

Report:

1. Dependencies declared directly with a version range where a `catalog:` entry
   already exists, since these bypass the catalog and cause version skew.
2. The same external dependency pinned to different versions across packages.
3. Packages listed in a `package.json` but not imported anywhere in that package.
4. Imports of packages that are not declared as a dependency of the importing
   package, which work by hoisting accident and break under stricter resolution.

For each finding give the package path and the specific dependency. Suggest the
concrete `package.json` change. Do not run any install or modify any lockfile.
