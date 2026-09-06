# REPO.md

`Produkter` innehåller en Python/Flask-applikation, tre Cloudflare Workers under `cloudflare/` och scraper-verktyg i Python/Playwright under `scraper/`.

## Invarians

- Bevara `account_id`-isolering för kontobunden data.
- Credentials och provider-nycklar får inte hårdkodas eller committas.
- Produktionsdistribution från `main` hanteras av Cloudflare Workers Builds.
- Varje Workers `wrangler.jsonc` är källa till sanning för versionshanterade bindings, routes och konfiguration.
- Den delade D1-databasen ska ha en entydig migrationsägare och en idempotent migrationskedja innan produktionsmigrering kopplas till deployment. Flera Workers får inte oberoende applicera samma SQL.
- Scraper-credentials ska hållas utanför images och förrådsfiler. Felrapportering ska fortsätta redigera känslig information.
- Ändra inte shell error flags globalt för stil; använd flaggor som stöds av den deklarerade runtime-shellen.

## Validering

Kör relevanta Python-tester samt Worker type-check/tests och Wrangler dry-run för berörda paket. Validera Docker-image och säkerhetsskanning när containerbeteende ändras.
