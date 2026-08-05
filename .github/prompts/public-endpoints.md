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
