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
