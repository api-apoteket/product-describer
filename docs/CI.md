# CI och deploy

## CI

`.github/workflows/ci.yml` producerar `CI / required` för pull requests, merge queue, push till `main` och manuella körningar.

Jobbet verifierar Python-koden genom dependency-installation, `compileall` och `pytest`. Därefter verifieras alla tre Cloudflare Workers (`app`, `engine`, `processor`) med `npm ci`, TypeScript-kontroll och Wrangler dry-run. CI kör också testerna för produktionsverifiering och D1-sessioner under `cloudflare/scripts/`.

`.github/workflows/dependency-review.yml` kör GitHubs dependency review på pull requests och merge groups och blockerar nya beroenden från severity `high` och uppåt.

## Docker och säkerhet

`.github/workflows/docker.yml` bygger images för huvudapplikationen och `scraper`, kör Trivy och laddar SARIF till GitHub Code Scanning. HIGH/CRITICAL-fynd gate:as separat med `exit-code: 1`.

Workflowen körs på pull requests, merge groups, push till `main`, veckoschema och manuellt. Images publiceras endast från `main` vid push eller manuell körning. Aggregatjobbet heter `docker`.

Code Scanning-kategorierna är `trivy-product-describer` och `trivy-scraper`.

## Deploy

Cloudflare Workers Builds äger normal produktionsdeploy från `main`; GitHub Actions deployar inte Workers-produktionen.

| Worker | Root directory | Deploy command |
| --- | --- | --- |
| `produkter` | `cloudflare/app` | `npm run deploy && npm run verify:production` |
| `produkter-motor` | `cloudflare/engine` | `npm run deploy && npm run verify:production` |
| `produkter-bearbetare` | `cloudflare/processor` | `npm run deploy` |

`deploy` är i samtliga tre paket direkt `wrangler deploy --strict`.

Efter deploy verifieras bara ytor som faktiskt finns:

- `produkter`: huvuddomänen måste svara HTTP 200 via `npm run verify:production`.
- `produkter-motor`: `https://motor.denied.se/health` måste svara HTTP 200 med `{ "ok": true }` via `npm run verify:production`.
- `produkter-bearbetare`: ingen HTTP-check finns eftersom Workern är en privat Queue-konsument utan publik route.

`cloudflare/scripts/verify-production.mjs` innehåller applikationsspecifik HTTP-verifiering.

`wrangler.jsonc` i respektive katalog är source of truth för Worker-namn, bindings, routes och cron-triggers.

D1-databasen `produkter` delas av flera Workers. Repositoryt har ingen Wrangler `migrations/`-kedja; versionshanterade SQL-filer ligger under `cloudflare/infra/`. Schemaändringar hanteras därför separat från de tre Workers Builds.
