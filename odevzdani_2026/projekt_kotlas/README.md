# Project Manager

Informační systém pro malé firmy, který sjednocuje práci s klienty, projekty, dokumentovým tokem a reporty do jednoho bezpečného prostředí.

Projekt vznikl jako maturitní práce a je postavený na Symfony. Cílem je nahradit roztříštěnou agendu (e-mail, tabulky, sdílené disky) jedním konzistentním systémem s jasnými rolemi a kontrolou přístupu.

## Co projekt řeší

- Centrální evidence klientů, projektů a souvisejících dokumentů
- End-to-end tok zakázky: nabídka -> objednávka -> smlouva -> akceptační protokol -> faktura
- Role-based přístupový model s objektovou autorizací (ACL přes Voters)
- Bezpečný upload/download dokumentů bez přímého veřejného přístupu k souborům
- Dashboard a reporty s časovým filtrem a grafem tržeb

## Aktuální stav

Implementováno a funkční:

- autentizace, role hierarchy a jednotný dashboard po loginu
- CRUD moduly pro všechny hlavní business entity
- ACL na úrovni route, role i konkrétní entity
- reset hesla (SymfonyCasts Reset Password Bundle)
- dokumentové přílohy a bezpečné stahování přes controller
- reportovací modul s filtrováním dat

## Hlavní funkce a moduly

- Dashboard: `/dashboard`
- Uživatelé: `/uzivatele/*`
- Klienti: `/klienti/*`
- Projekty: `/projekty/*`
- Přiřazení uživatelů k projektům: `/projekt-uzivatel/*`
- Nabídky: `/nabidky/*`
- Objednávky: `/objednavky/*`
- Smlouvy: `/smlouvy/*`
- Akceptační protokoly: `/akceptacni-protokoly/*`
- Faktury: `/faktury/*`
- Reporty: `/reporty/*`
- Reset hesla: `/reset-password/*`

## Technologie

- PHP >= 8.2 (runtime v Dockeru: PHP 8.3 + Apache)
- Symfony 7.4
- PostgreSQL 15
- Doctrine ORM + Doctrine Migrations
- Twig + Bootstrap 5
- DataTables + Chart.js
- AssetMapper / Importmap + Stimulus
- Docker + Docker Compose

## Architektura (stručně)

- SSR monolit (Symfony)
- vrstvy: Controller -> Service/Repository -> Doctrine ORM -> PostgreSQL
- soubory ukládány mimo veřejný webroot, metadata v DB
- doménové jádro: `Projekt` + navázané dokumentové entity
- ACL kombinací `access_control`, `#[IsGranted(...)]`, `denyAccessUnlessGranted(...)` a Voterů

## Role a bezpečnost

- `ROLE_USER`
- `ROLE_MANAGEMENT` (dědí `ROLE_USER`)
- `ROLE_ADMIN` (dědí `ROLE_MANAGEMENT` + `ROLE_USER`)

Použité Votery:

- `ProjectAccessVoter` (dokumenty navázané na projekt)
- `ProjektVoter` (operace nad projektem)
- `KlientVoter` (operace nad klientem)

## Datový model (výběr)

Projekt aktuálně obsahuje 22 Doctrine entit, mimo jiné:

- `Uzivatel`, `Klient`, `Projekt`, `ProjektUzivatel`
- dokumentové entity: `Nabidka`, `Objednavka`, `Smlouva`, `AkceptacniProtokol`, `Faktura`
- statusové číselníky pro jednotlivé moduly
- přílohové entity: `NabidkaDokument`, `ObjednavkaDokument`, `SmlouvaDokument`, `AkceptacniProtokolDokument`, `FakturaDokument`

## Rychlý start (Docker)

### 1) Klonování repozitáře

```bash
git clone https://github.com/Jirka-Kotlas/maturitni-projekt.git
cd maturitni-projekt
```

### 2) Lokální konfigurace

```bash
cp .env .env.local
```

Minimálně nastav databázové proměnné v `.env.local`:

```env
DATABASE_URL="postgresql://<user>:<password>@db:5432/projekt_manager?serverVersion=15&charset=utf8"
POSTGRES_DB=projekt_manager
POSTGRES_USER=<user>
POSTGRES_PASSWORD=<password>
```

### 3) Build a start kontejnerů

```bash
docker compose up -d --build
```

### 4) Instalace závislostí

```bash
docker compose exec app composer install
```

### 5) Migrace databáze

```bash
docker compose exec app php bin/console doctrine:migrations:migrate --no-interaction
```

### 6) Naplnění testovacích dat (volitelné, doporučeno pro lokální vývoj)

```bash
docker compose exec app php bin/console doctrine:fixtures:load --no-interaction
```

Aplikace poběží na `http://localhost:8000`.

## Testovací účty (fixtures)

- `admin@test.cz` / `admin123`
- `manager@test.cz` / `manager123`
- `user@test.cz` / `user123`
- `marie@test.cz` / `marie123`

Dalších 15 uživatelů je generovaných fixtures a používá heslo `password123`.

## Užitečné příkazy

```bash
# Přehled rout
docker compose exec app php bin/console debug:router

# Stav migrací
docker compose exec app php bin/console doctrine:migrations:status

# Validace DB schématu
docker compose exec app php bin/console doctrine:schema:validate

# Spuštění testů
docker compose exec app php bin/phpunit

# Cache
docker compose exec app php bin/console cache:clear

# Logy
docker compose logs -f app
docker compose logs -f db
```

## Testování

V projektu jsou funkční a servisní testy (adresář `tests`).

Lokálně:

```bash
docker compose exec app php bin/phpunit
```

Poznámka: pokrytí testy je průběžně rozšiřováno; část scénářů je zatím validována manuálně.

## Nasazení

Aktuální provozní model:

- Docker Compose (`app` + `db`)
- aplikace: PHP 8.3 + Apache
- databáze: PostgreSQL 15

Kompletní deployment postup je v [DEPLOYMENT.md](DEPLOYMENT.md).

Alternativně je dostupný skript [deploy.sh](deploy.sh) pro automatizaci běžného postupu (update, dependencies, migrace, cache, assets).

## Struktura projektu

```text
src/
  Controller/
  DataFixtures/
  Entity/
  Form/
  Repository/
  Security/
  Service/
templates/
assets/
config/
migrations/
tests/
```

## Stav projektu a roadmapa

Stav: aktivní vývoj a stabilizace pro produkční provoz.

Plán dalšího rozvoje:

- vyšší pokrytí unit/integration testy
- CI/CD hardening + monitoring + backup workflow
- další UX/UI sjednocení napříč moduly
- formalizace API vrstvy pro externí integrace
- automatizace dokumentových toků (PDF)

## Dokumentace

- Technický průběžný stav: [PROGRESS_REPORT.md](PROGRESS_REPORT.md)
- Nasazení: [DEPLOYMENT.md](DEPLOYMENT.md)
- Technická dokumentace: [dokumentace/technicka_dokumentace.md](dokumentace/technicka_dokumentace.md)

## Autor

Jiří Kotlas

GitHub: [@Jirka-Kotlas](https://github.com/Jirka-Kotlas)

## Licence

`proprietary` (viz `composer.json`).

Projekt je vytvořen primárně pro účely maturitní práce.
