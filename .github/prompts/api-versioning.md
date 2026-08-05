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
