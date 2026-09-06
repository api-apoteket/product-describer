# REPO.md

`Produkter` contains a Python/Flask application, three Cloudflare Workers under `cloudflare/`, and Python/Playwright scraper tooling under `scraper/`.

## Invariants

- Preserve `account_id` isolation for account-scoped data. Never hardcode or commit credentials/provider keys.
- Cloudflare Workers Builds owns normal production deployment from `main`; GitHub Actions validates but does not duplicate the production control plane.
- Each Worker's `wrangler.jsonc` is the source of truth for its versioned bindings/routes/configuration.
- The shared D1 database needs one unambiguous migration owner and an idempotent migration chain before production schema migration is added to deploys. Separate Workers must not independently apply the same SQL.
- Scraper credentials remain external to images/repository files, startup keeps restrictive credential-directory permissions, and error reporting preserves redaction.
- Do not globally change shell error flags for style; use flags supported by the declared runtime shell.

## Validation

Run relevant Python tests plus Worker type-check/tests and Wrangler dry-run validation for affected packages. Validate Docker images/security scanning when container behavior changes.

The live repository rules currently require `CI / required`, `docker` and `dependency-review`. Do not rename/remove a required check without updating and verifying the live ruleset in the same migration.

Pin third-party GitHub Actions to full commit SHAs.
