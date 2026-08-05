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
