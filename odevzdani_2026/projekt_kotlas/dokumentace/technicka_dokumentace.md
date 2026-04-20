# Technická dokumentace – Project Manager

**Projekt:** Systém na řízení firemních procesů a dokumentů  
**Autor:** Jiří Kotlas  
**Verze dokumentu:** 1.0  
**Datum:** Březen 2026  
**Repozitář:** [github.com/Jirka-Kotlas/maturitni-projekt](https://github.com/Jirka-Kotlas/maturitni-projekt)

---

## Obsah

1. [Přehled systému](#1-přehled-systému)
2. [Technologický stack](#2-technologický-stack)
3. [Struktura repozitáře](#3-struktura-repozitáře)
4. [Instalace a spuštění](#4-instalace-a-spuštění)
5. [Databázové schéma](#5-databázové-schéma)
6. [Workflow stavů](#6-workflow-stavů)
7. [Bezpečnost a role](#7-bezpečnost-a-role)
8. [Routing – přehled tras](#8-routing--přehled-tras)
9. [Hlavní moduly a controllery](#9-hlavní-moduly-a-controllery)
10. [Upload souborů](#10-upload-souborů)
11. [Reporty a dashboard](#11-reporty-a-dashboard)
12. [Deployment na produkci (VPS)](#12-deployment-na-produkci-vps)
13. [Testovací data (Fixtures)](#13-testovací-data-fixtures)
14. [Troubleshooting](#14-troubleshooting)

---

## 1. Přehled systému

Project Manager je webová aplikace určená pro správu firemních procesů a dokumentů. Pokrývá celý životní cyklus zakázky – od první nabídky přes objednávku a smlouvu až po akceptační protokol a fakturu. Systém je nasazen na technologiích PHP / Symfony a provozován v kontejnerizovaném prostředí Docker.

### Architektura

```
Prohlížeč
    │  HTTP/HTTPS
    ▼
Apache 2 (DocumentRoot: public/)
    │  Všechny požadavky → public/index.php
    ▼
Symfony Front Controller (PHP 8.3)
    ├── Router → Controller → Twig template
    ├── Doctrine ORM
    └── PostgreSQL 15
```

Aplikace je **serverem renderovaná** (SSR) webová aplikace s architekturou MVC. Neexistuje separátní REST API ani frontend SPA framework – vše se vykresluje pomocí Twig šablon na serveru. JavaScript (Stimulus + DataTables + Chart.js) se využívá výhradně pro UI vylepšení (tabulky, grafy, vyhledávání).

---

## 2. Technologický stack

| Vrstva | Technologie | Verze |
|---|---|---|
| Backend framework | Symfony | 7.4 |
| Jazyk | PHP | ≥ 8.2 (Docker: 8.3) |
| ORM | Doctrine ORM + Migrations | – |
| Databáze | PostgreSQL | 15 |
| Šablonovací engine | Twig | – |
| CSS framework | Bootstrap | 5 |
| Tabulky | DataTables | – |
| Grafy | Chart.js | – |
| JS (interaktivita) | Stimulus (Symfony UX) | – |
| Kontejnerizace | Docker + Docker Compose | – |
| Web server | Apache 2 (mod_rewrite) | – |
| Správce závislostí | Composer | 2 |
| Správce front-end assetů | Symfony AssetMapper / importmap | – |

---

## 3. Struktura repozitáře

```text
maturitni-projekt/
├── assets/                     # Frontend JS a CSS
│   ├── controllers/            # Stimulus controllery
│   │   ├── csrf_protection_controller.js
│   │   ├── datatables_controller.js
│   │   ├── search_controller.js
│   │   └── hello_controller.js
│   ├── styles/
│   │   ├── app.css
│   │   ├── dashboard.css
│   │   └── login.css
│   ├── app.js
│   └── datatables.js
├── config/                     # Symfony konfigurace
│   ├── packages/
│   │   ├── security.yaml       # Bezpečnost, role, firewall
│   │   └── ...
│   └── routes.yaml
├── dokumentace/
│   └── technicka_dokumentace.md  # Tento soubor
├── migrations/                 # Doctrine migrace (16 souborů)
├── public/
│   ├── index.php               # Front controller
│   └── .htaccess               # Apache rewrite pravidla
├── src/
│   ├── Controller/             # 14 controllerů
│   ├── DataFixtures/           # 18 fixture tříd (testovací data)
│   ├── Entity/                 # 22 Doctrine entit
│   ├── Form/                   # 13 Symfony Form tříd
│   ├── Model/                  # Rozhraní (ProjectRelatedInterface)
│   ├── Repository/             # 22 repository tříd
│   ├── Security/               # Auth handler + 3 Votery
│   └── Service/                # FileUploader service
├── templates/                  # Twig šablony (60 souborů)
│   ├── base.html.twig          # Základní layout
│   ├── dashboard/
│   ├── faktura/
│   ├── klient/
│   ├── nabidka/
│   ├── objednavka/
│   ├── projekt/
│   ├── projekt_uzivatel/
│   ├── report/
│   ├── reset_password/
│   ├── security/
│   ├── smlouva/
│   ├── akceptacni_protokol/
│   └── uzivatel/
├── tests/                      # PHPUnit testy
├── var/uploads/                # Nahraté soubory (runtime)
├── Dockerfile
├── docker-compose.yml
├── DEPLOYMENT.md               # Průvodce nasazením
├── README.md
└── PROGRESS_REPORT.md
```

---

## 4. Instalace a spuštění

### 4.1 Požadavky (lokální prostředí)

- Docker ≥ 24
- Docker Compose ≥ 2
- Git

### 4.2 Rychlé spuštění (Docker)

```bash
# 1. Klonování repozitáře
git clone https://github.com/Jirka-Kotlas/maturitni-projekt.git
cd maturitni-projekt

# 2. Příprava env souboru
cp .env .env.local
```

Upravte `.env.local` – nastavte přihlašovací údaje k databázi:

```env
DATABASE_URL="postgresql://<user>:<heslo>@db:5432/projekt_manager?serverVersion=15&charset=utf8"
POSTGRES_DB=projekt_manager
POSTGRES_USER=<user>
POSTGRES_PASSWORD=<heslo>
```

```bash
# 3. Build a spuštění kontejnerů
docker-compose up -d --build

# 4. Instalace PHP závislostí
docker-compose exec app composer install

# 5. Spuštění databázových migrací
docker-compose exec app php bin/console doctrine:migrations:migrate --no-interaction

# 6. Načtení testovacích dat (volitelné)
docker-compose exec app php bin/console doctrine:fixtures:load --no-interaction
```

Aplikace poběží na **http://localhost:8000**.

### 4.3 Docker Compose konfigurace

Soubor `docker-compose.yml` definuje dvě služby:

| Služba | Popis | Port |
|---|---|---|
| `app` | PHP 8.3 + Apache, zdrojový kód mountován jako volume | `8000:80` |
| `db` | PostgreSQL 15, data persistována v named volume `db_data` | `127.0.0.1:5433:5432` |

### 4.4 Dockerfile (app)

```dockerfile
FROM php:8.3-apache
RUN apt-get install -y libpq-dev git unzip
RUN docker-php-ext-install pdo pdo_pgsql
RUN a2enmod rewrite
# DocumentRoot nastaven na /var/www/html/public
```

### 4.5 Proměnné prostředí

| Proměnná | Popis | Příklad |
|---|---|---|
| `DATABASE_URL` | DSN připojení k PostgreSQL | `postgresql://user:pass@db:5432/projekt_manager?serverVersion=15` |
| `POSTGRES_DB` | Název databáze (pro PostgreSQL kontejner) | `projekt_manager` |
| `POSTGRES_USER` | Uživatel databáze | `app_user` |
| `POSTGRES_PASSWORD` | Heslo k databázi | `tajné_heslo` |
| `APP_ENV` | Prostředí Symfony | `prod` nebo `dev` |
| `APP_SECRET` | Symfony secret pro CSRF a sessions | Náhodný řetězec 32+ znaků |

> **Bezpečnost:** Soubor `.env.local` a `.env` s produkčními hodnotami **nikdy necommitujte** do Gitu. Soubor `.env` v repozitáři obsahuje pouze výchozí/ukázkové hodnoty.

### 4.6 Migrace databáze

Migrace jsou uloženy ve složce `migrations/` a spravovány přes Doctrine Migrations:

```bash
# Zobrazení stavu migrací
docker-compose exec app php bin/console doctrine:migrations:status

# Spuštění nových migrací
docker-compose exec app php bin/console doctrine:migrations:migrate --no-interaction

# Validace schématu
docker-compose exec app php bin/console doctrine:schema:validate
```

---

## 5. Databázové schéma

### 5.1 Přehled entit

Aplikace obsahuje 22 Doctrine entit rozdělených do čtyř skupin:

**Byznys entity** (hlavní obchodní objekty):

| Entita | Tabulka | Popis |
|---|---|---|
| `Uzivatel` | `uzivatel` | Uživatelský účet, autentizace, role |
| `Klient` | `klient` | Zákazník / firma |
| `Projekt` | `projekt` | Projekt vázaný na klienta |
| `ProjektUzivatel` | `projekt_uzivatel` | Přiřazení uživatele k projektu (M:N s metadaty) |
| `Nabidka` | `nabidka` | Cenová nabídka k projektu |
| `Objednavka` | `objednavka` | Objednávka (může navazovat na nabídku) |
| `Smlouva` | `smlouva` | Smlouva k projektu |
| `AkceptacniProtokol` | `akceptacni_protokol` | Protokol o převzetí díla |
| `Faktura` | `faktura` | Faktura navázaná na protokol a projekt |

**Dokumentové přílohy** (samostatné entity navázané na dokumenty):

| Entita | Tabulka | Popis |
|---|---|---|
| `NabidkaDokument` | `nabidka_dokument` | Přílohy nabídky |
| `ObjednavkaDokument` | `objednavka_dokument` | Přílohy objednávky |
| `SmlouvaDokument` | `smlouva_dokument` | Přílohy smlouvy |
| `AkceptacniProtokolDokument` | `akceptacni_protokol_dokument` | Přílohy akceptačního protokolu |
| `FakturaDokument` | `faktura_dokument` | Přílohy faktury |

**Číselníky (lookup entity)**:

| Entita | Tabulka | Možné hodnoty (kódy) |
|---|---|---|
| `ProjektStatus` | `projekt_status` | `planning`, `active`, `closed`, `archived` |
| `NabidkaStatus` | `nabidka_status` | `rozpracovana`, `odeslana`, `akceptovana`, `zamitnuta` |
| `ObjednavkaStatus` | `objednavka_status` | `nova`, `v_realizaci`, `splnena`, `stornovana` |
| `SmlouvaStatus` | `smlouva_status` | `draft`, `active`, `terminated`, `expired` |
| `SmlouvaPodpisStatus` | `smlouva_podpis_status` | `unsigned`, `signed_client`, `signed_us`, `fully_signed` |
| `FakturaStatus` | `faktura_status` | `ke_schvaleni`, `schvalena`, `vystavena`, `odeslana`, `zaplacena`, `po_splatnosti` |
| `ProtokolStatus` | `protokol_status` | `draft`, `signed_client`, `signed_us`, `completed` |

**Servisní entita**:

| Entita | Tabulka | Popis |
|---|---|---|
| `ResetPasswordRequest` | `reset_password_request` | Tokeny pro reset hesla (SymfonyCasts Reset Password Bundle) |

### 5.2 Klíčové atributy entit

#### `Uzivatel`
| Sloupec | Typ | Popis |
|---|---|---|
| `id` | int (PK) | Primární klíč |
| `email` | varchar(255), unique | Uživatelské jméno (přihlašovací identifikátor) |
| `password` | varchar(255) | Bcrypt hash hesla |
| `jmeno` | varchar(255) | Křestní jméno |
| `prijmeni` | varchar(255) | Příjmení |
| `active` | boolean | Aktivní / neaktivní účet |
| `roles` | json | Pole rolí (např. `["ROLE_ADMIN"]`) |

#### `Klient`
| Sloupec | Typ | Popis |
|---|---|---|
| `id` | int (PK) | – |
| `nazev` | varchar(255) | Název firmy / klienta |
| `ico` | varchar(20), nullable | IČO |
| `dic` | varchar(20), nullable | DIČ |
| `adresa` | text, nullable | Adresa |
| `kontaktniOsoba` | varchar(255), nullable | Jméno kontaktní osoby |
| `kontaktniOsobaEmail` | varchar(255), nullable | E-mail kontaktní osoby |
| `kontaktniOsobaTelefon` | varchar(50), nullable | Telefon kontaktní osoby |

#### `Projekt`
| Sloupec | Typ | Popis |
|---|---|---|
| `id` | int (PK) | – |
| `nazev` | varchar(255) | Název projektu |
| `popis` | text, nullable | Popis projektu |
| `klient_id` | FK → klient | Klient |
| `status_id` | FK → projekt_status | Stav projektu |

#### `ProjektUzivatel` (vazební tabulka M:N)
| Sloupec | Typ | Popis |
|---|---|---|
| `id` | int (PK) | – |
| `projekt_id` | FK → projekt | Projekt |
| `uzivatel_id` | FK → uzivatel | Uživatel |
| `roleVProjektu` | varchar(50) | Role v projektu (např. „Manažer", „Vývojář") |
| `prirazenDne` | datetime, nullable | Datum přiřazení |
| `odebranDne` | datetime, nullable | Datum odebrání |
| `aktivni` | boolean | Aktivní přiřazení |

#### Dokumentové entity (Nabidka, Objednavka, Smlouva, AkceptacniProtokol, Faktura)

Každá dokumentová entita sdílí společný vzor atributů:

| Sloupec | Typ | Popis |
|---|---|---|
| `projekt_id` | FK → projekt | Projekt, ke kterému dokument patří |
| `status_id` | FK → *_status | Aktuální stav dokumentu |
| `cislo*` | varchar | Jedinečné číslo dokladu (např. `cisloFaktury`) |
| `verze` | int | Číslo verze dokumentu |
| `filePath` | varchar(500), nullable | Relativní cesta k souboru (`uploads/...`) |
| `fileName` | varchar(255), nullable | Původní název souboru |
| `fileMimeType` | varchar(100), nullable | MIME typ souboru |
| `fileSize` | int, nullable | Velikost souboru v bajtech |
| `metadata` | json, nullable | Rozšířená metadata |
| `createdBy_id` | FK → uzivatel | Uživatel, který doklad vytvořil |

Specifické atributy `Faktura`:

| Sloupec | Typ | Popis |
|---|---|---|
| `cisloFaktury` | varchar(100), unique | Unikátní číslo faktury |
| `typ` | varchar(20) | Typ faktury (např. proforma, daňový doklad) |
| `datumVystaveni` | date | Datum vystavení |
| `datumSplatnosti` | date | Datum splatnosti |
| `castkaBezDph` | decimal(15,2) | Základ daně |
| `dphCastka` | decimal(15,2) | Výše DPH |
| `castkaCelkem` | decimal(15,2) | Celková částka včetně DPH |
| `mena` | varchar(3) | Měna (ISO 4217, např. `CZK`) |
| `datumUhrazeni` | date, nullable | Datum uhrazení faktury |
| `akceptacniProtokol_id` | FK → akceptacni_protokol | Navázaný protokol |

Specifické atributy `Smlouva`:

| Sloupec | Typ | Popis |
|---|---|---|
| `cislo` | varchar(50) | Číslo smlouvy |
| `datumVystaveni` | date | Datum vystavení |
| `datumPodpisu` | date, nullable | Datum podpisu |
| `podpisStatus_id` | FK → smlouva_podpis_status | Stav podpisu |
| `platnostOd` / `platnostDo` | date, nullable | Platnost smlouvy |
| `castka` | decimal(15,2), nullable | Hodnota smlouvy |
| `mena` | varchar(3), nullable | Měna |
| `predmetSmlouvy` | text, nullable | Předmět smlouvy |
| `poznamky` | text, nullable | Interní poznámky |

### 5.3 Vztahy mezi entitami

```
Klient (1) ──────────────── (N) Projekt
                                    │
              ┌─────────────────────┤
              │                     │
      (N) ProjektUzivatel (N)   (každý dokument)
              │                     │
      (N) Uzivatel              Nabidka
                                Objednavka ──── (nullable) Nabidka
                                Smlouva
                                AkceptacniProtokol
                                    │
                                    └──── (N) Faktura
```

- `Projekt` → `Klient` (N:1)
- `ProjektUzivatel` → `Projekt` + `Uzivatel` (M:N s metadaty)
- `Nabidka`, `Objednavka`, `Smlouva`, `AkceptacniProtokol`, `Faktura` → `Projekt` (N:1)
- `NabidkaDokument` → `Nabidka`, `ObjednavkaDokument` → `Objednavka`, `SmlouvaDokument` → `Smlouva`, `AkceptacniProtokolDokument` → `AkceptacniProtokol`, `FakturaDokument` → `Faktura` (N:1)
- `Objednavka` → `Nabidka` (N:1, nullable – objednávka může, ale nemusí vycházet z nabídky)
- `Faktura` → `AkceptacniProtokol` (N:1)
- Všechny dokumentové entity → `Uzivatel` přes `createdBy` (N:1)

---

## 6. Workflow stavů

### 6.1 Nabídka (`NabidkaStatus`)

```
Rozpracovaná → Odeslaná → Akceptovaná
                       └→ Zamítnutá
```

| Kód | Název | Barva |
|---|---|---|
| `rozpracovana` | Rozpracovaná | šedá |
| `odeslana` | Odeslaná | světle modrá |
| `akceptovana` | Akceptovaná | zelená |
| `zamitnuta` | Zamítnutá | červená |

### 6.2 Objednávka (`ObjednavkaStatus`)

```
Nová → V realizaci → Splněná
     └→ Stornovaná
```

| Kód | Název | Barva |
|---|---|---|
| `nova` | Nová | světle modrá |
| `v_realizaci` | V realizaci | oranžová |
| `splnena` | Splněná | zelená |
| `stornovana` | Stornovaná | červená |

> Objednávka může volitelně navazovat na existující nabídku (FK `nabidka_id` s `ON DELETE SET NULL`).

### 6.3 Smlouva (`SmlouvaStatus` + `SmlouvaPodpisStatus`)

Smlouva sleduje dva nezávislé stavy:

**Stav smlouvy:**

```
Koncept → Aktivní → Ukončená
                 └→ Vypršela
```

| Kód | Název |
|---|---|
| `draft` | Koncept |
| `active` | Aktivní |
| `terminated` | Ukončená |
| `expired` | Vypršela |

**Stav podpisu:**

```
Nepodepsáno → Podepsal klient → Plně podepsáno
            → Podepsáno námi  → Plně podepsáno
```

| Kód | Název |
|---|---|
| `unsigned` | Nepodepsáno |
| `signed_client` | Podepsal klient |
| `signed_us` | Podepsáno námi |
| `fully_signed` | Plně podepsáno |

### 6.4 Akceptační protokol (`ProtokolStatus`)

```
Koncept → Podepsal klient → Dokončeno
        → Podepsáno námi  → Dokončeno
```

| Kód | Název |
|---|---|
| `draft` | Koncept |
| `signed_client` | Podepsal klient |
| `signed_us` | Podepsáno námi |
| `completed` | Dokončeno |

Výchozí typ protokolu při vytvoření je `PROVADECI` (nastaveno v konstruktoru entity).

### 6.5 Faktura (`FakturaStatus`)

```
Ke schválení → Schválená → Vystavená → Odeslaná → Zaplacená
                                              └→ Po splatnosti
```

| Kód | Název | Barva |
|---|---|---|
| `ke_schvaleni` | Ke schválení | šedá |
| `schvalena` | Schválená | světle modrá |
| `vystavena` | Vystavená | oranžová |
| `odeslana` | Odeslaná | zlatá |
| `zaplacena` | Zaplacená | zelená |
| `po_splatnosti` | Po splatnosti | červená |

Faktura je vždy navázána na konkrétní akceptační protokol.

### 6.6 Projekt (`ProjektStatus`)

| Kód | Název | Barva |
|---|---|---|
| `planning` | Plánování | oranžová |
| `active` | Aktivní | zelená |
| `closed` | Ukončený | šedá |
| `archived` | Archivovaný | tmavě šedá |

### 6.7 Celkový procesní tok zakázky

```
[Klient]
    │
    ▼
[Projekt] (Plánování → Aktivní → Ukončený/Archivovaný)
    │
    ├── [Nabídka] (Rozpracovaná → Odeslaná → Akceptovaná/Zamítnutá)
    │         │
    │         ▼ (pokud akceptována)
    ├── [Objednávka] (Nová → V realizaci → Splněná/Stornovaná)
    │
    ├── [Smlouva] (Koncept → Aktivní → Ukončená)
    │
    ├── [Akceptační protokol] (Koncept → Podpisy → Dokončeno)
    │         │
    │         ▼
    └── [Faktura] (Ke schválení → ... → Zaplacená)
```

---

## 7. Bezpečnost a role

### 7.1 Hierarchie rolí

Aplikace definuje tři role s hierarchií dědičnosti:

```
ROLE_ADMIN
    └── ROLE_MANAGEMENT
            └── ROLE_USER
```

| Role | Oprávnění |
|---|---|
| `ROLE_USER` | Přístup k vlastním přiřazeným projektům a jejich dokumentům; zobrazení vlastního dashboardu |
| `ROLE_MANAGEMENT` | Rozšířený přístup k datům, editace entit, správa projektových přiřazení |
| `ROLE_ADMIN` | Plný přístup ke všem datům a funkcím systému bez omezení |

### 7.2 Konfigurace firewallu (security.yaml)

```yaml
firewalls:
  dev:
    pattern: ^/(_profiler|_wdt|assets|build)/
    security: false
  main:
    provider: app_user_provider
    form_login:
      login_path: app_login
      check_path: app_login
      enable_csrf: true
      default_target_path: app_dashboard
      success_handler: App\Security\LoginSuccessHandler
    logout:
      path: app_logout
      target: app_login

access_control:
  - { path: ^/login, roles: PUBLIC_ACCESS }
  - { path: ^/dashboard, roles: ROLE_USER }
```

Uživatel je identifikován podle e-mailové adresy (vlastnost `email` entity `Uzivatel`). Po úspěšném přihlášení je uživatel přesměrován na `/dashboard` prostřednictvím `LoginSuccessHandler`.

Hesla jsou hashována algoritmem `auto` (Symfony zvolí nejsilnější dostupný algoritmus – obvykle bcrypt nebo Argon2).

### 7.3 Votery (ACL)

Jemnozrnná kontrola přístupu je implementována třemi Voter třídami v `src/Security/Voter/`:

#### `ProjektVoter`
Řídí přístup k entitě `Projekt` s atributy `VIEW` a `EDIT`.

- **ROLE_ADMIN**: vždy povoleno
- **Ostatní**: přístup povolen, pokud je uživatel přiřazen k projektu přes `ProjektUzivatel`

#### `ProjectAccessVoter`
Řídí přístup k libovolné entitě implementující rozhraní `ProjectRelatedInterface` (tj. `Nabidka`, `Objednavka`, `Smlouva`, `AkceptacniProtokol`, `Faktura`) s atributy `VIEW`, `EDIT` a `DELETE`.

- **ROLE_ADMIN**: vždy povoleno
- **VIEW**: povoleno také autorovi dokumentu (`createdBy`)
- **Ostatní**: přístup povolen, pokud je uživatel přiřazen k projektu daného dokumentu

#### `KlientVoter`
Řídí přístup k entitě `Klient` s atributy `VIEW` a `EDIT`.

- **ROLE_ADMIN** nebo **ROLE_MANAGEMENT**: vždy povoleno
- **ROLE_USER**: přístup povolen, pokud je uživatel přiřazen k alespoň jednomu projektu daného klienta

### 7.4 CSRF ochrana

Formuláře i přihlašovací formulář mají aktivní CSRF ochranu (Symfony vestavěná podpora + `enable_csrf: true` pro form login). Stimulus controller `csrf_protection_controller.js` zajišťuje CSRF token pro AJAX požadavky.

---

## 8. Routing – přehled tras

### Veřejné trasy (bez přihlášení)

| URL | Název | Metoda | Popis |
|---|---|---|---|
| `/` | `app_home` | GET | Domovská stránka |
| `/login` | `app_login` | GET, POST | Přihlášení |
| `/logout` | `app_logout` | GET | Odhlášení |
| `/reset-password` | `app_forgot_password_request` | GET, POST | Žádost o reset hesla |
| `/reset-password/check-email` | – | GET | Potvrzení odeslání e-mailu |
| `/reset-password/reset/{token}` | `app_reset_password` | GET, POST | Reset hesla tokenem |

### Aplikační trasy (vyžadují ROLE_USER)

#### Dashboard

| URL | Název | Popis |
|---|---|---|
| `/dashboard` | `app_dashboard` | Hlavní přehled po přihlášení |

#### Uživatelé (`/uzivatele`)

| URL | Název | Metoda | Popis |
|---|---|---|---|
| `/uzivatele/` | `app_uzivatel_index` | GET | Seznam uživatelů |
| `/uzivatele/new` | `app_uzivatel_new` | GET, POST | Nový uživatel |
| `/uzivatele/{id}` | `app_uzivatel_show` | GET | Detail uživatele |
| `/uzivatele/{id}/edit` | `app_uzivatel_edit` | GET, POST | Editace uživatele |
| `/uzivatele/{id}/delete` | `app_uzivatel_delete` | POST | Smazání uživatele |
| `/uzivatele/profile` | `app_uzivatel_profile` | GET | Vlastní profil |
| `/uzivatele/edit-profile` | `app_uzivatel_edit_profile` | GET, POST | Editace profilu |
| `/uzivatele/change-password` | `app_uzivatel_change_password` | GET, POST | Změna hesla |

#### Klienti (`/klienti`)

| URL | Název | Metoda | Popis |
|---|---|---|---|
| `/klienti/` | `app_klient_index` | GET | Seznam klientů |
| `/klienti/new` | `app_klient_new` | GET, POST | Nový klient |
| `/klienti/{id}` | `app_klient_show` | GET | Detail klienta |
| `/klienti/{id}/edit` | `app_klient_edit` | GET, POST | Editace klienta |
| `/klienti/{id}/delete` | `app_klient_delete` | POST | Smazání klienta |

#### Projekty (`/projekty`)

| URL | Název | Metoda | Popis |
|---|---|---|---|
| `/projekty/` | `app_projekt_index` | GET | Seznam projektů |
| `/projekty/new` | `app_projekt_new` | GET, POST | Nový projekt |
| `/projekty/{id}` | `app_projekt_show` | GET | Detail projektu |
| `/projekty/{id}/edit` | `app_projekt_edit` | GET, POST | Editace projektu |
| `/projekty/{id}/delete` | `app_projekt_delete` | POST | Smazání projektu |

#### Přiřazení uživatelů k projektům (`/projekt-uzivatel`)

| URL | Název | Metoda | Popis |
|---|---|---|---|
| `/projekt-uzivatel/new` | `app_projekt_uzivatel_new` | GET, POST | Přiřazení uživatele k projektu |

#### Nabídky (`/nabidky`)

| URL | Název | Metoda | Popis |
|---|---|---|---|
| `/nabidky/` | `app_nabidka_index` | GET | Seznam nabídek |
| `/nabidky/new` | `app_nabidka_new` | GET, POST | Nová nabídka |
| `/nabidky/{id}` | `app_nabidka_show` | GET | Detail nabídky |
| `/nabidky/{id}/edit` | `app_nabidka_edit` | GET, POST | Editace nabídky |
| `/nabidky/{id}/delete` | `app_nabidka_delete` | POST | Smazání nabídky |

#### Objednávky (`/objednavky`)

| URL | Název | Metoda | Popis |
|---|---|---|---|
| `/objednavky/` | `app_objednavka_index` | GET | Seznam objednávek |
| `/objednavky/new` | `app_objednavka_new` | GET, POST | Nová objednávka |
| `/objednavky/{id}` | `app_objednavka_show` | GET | Detail objednávky |
| `/objednavky/{id}/edit` | `app_objednavka_edit` | GET, POST | Editace objednávky |
| `/objednavky/{id}/delete` | `app_objednavka_delete` | POST | Smazání objednávky |

#### Smlouvy (`/smlouvy`)

| URL | Název | Metoda | Popis |
|---|---|---|---|
| `/smlouvy/` | `app_smlouva_index` | GET | Seznam smluv |
| `/smlouvy/new` | `app_smlouva_new` | GET, POST | Nová smlouva |
| `/smlouvy/{id}` | `app_smlouva_show` | GET | Detail smlouvy |
| `/smlouvy/{id}/edit` | `app_smlouva_edit` | GET, POST | Editace smlouvy |
| `/smlouvy/{id}/delete` | `app_smlouva_delete` | POST | Smazání smlouvy |

#### Akceptační protokoly (`/akceptacni-protokoly`)

| URL | Název | Metoda | Popis |
|---|---|---|---|
| `/akceptacni-protokoly/` | `app_akceptacni_protokol_index` | GET | Seznam protokolů |
| `/akceptacni-protokoly/new` | `app_akceptacni_protokol_new` | GET, POST | Nový protokol |
| `/akceptacni-protokoly/{id}` | `app_akceptacni_protokol_show` | GET | Detail protokolu |
| `/akceptacni-protokoly/{id}/edit` | `app_akceptacni_protokol_edit` | GET, POST | Editace protokolu |
| `/akceptacni-protokoly/{id}/delete` | `app_akceptacni_protokol_delete` | POST | Smazání protokolu |

#### Faktury (`/faktury`)

| URL | Název | Metoda | Popis |
|---|---|---|---|
| `/faktury/` | `app_faktura_index` | GET | Seznam faktur |
| `/faktury/new` | `app_faktura_new` | GET, POST | Nová faktura |
| `/faktury/{id}` | `app_faktura_show` | GET | Detail faktury |
| `/faktury/{id}/edit` | `app_faktura_edit` | GET, POST | Editace faktury |
| `/faktury/{id}/delete` | `app_faktura_delete` | POST | Smazání faktury |

#### Reporty (`/reporty`)

| URL | Název | Metoda | Popis |
|---|---|---|---|
| `/reporty/` | `app_report_index` | GET | Přehled reportů s filtrem datumu |

---

## 9. Hlavní moduly a controllery

### 9.1 DashboardController

**Soubor:** `src/Controller/DashboardController.php`  
**Trasa:** `/dashboard`

Zobrazuje přehledný dashboard po přihlášení uživatele. Obsah se liší podle role:

- **Moje projekty:** všechny aktivní přiřazení uživatele z `ProjektUzivatel`, řazeno sestupně podle data přiřazení
- **Nedávné nabídky:** pro admina posledních 3 nabídky globálně; pro ostatní 3 nabídky filtrované na vlastní projekty
- **Nedávné faktury:** stejná logika jako nabídky

### 9.2 ReportController

**Soubor:** `src/Controller/ReportController.php`  
**Trasa:** `/reporty/`

Zobrazuje souhrnný přehled s časovým filtrem. Výchozí rozsah je aktuální rok (leden–prosinec). Uživatel může filtr vymazat (zobrazí se všechna data) nebo zadat libovolné datum.

Zobrazované metriky:
- Počet aktivních nabídek ve zvoleném období
- Počet aktivních objednávek ve zvoleném období
- Počet aktivních smluv ve zvoleném období
- Smlouvy vypršející do 30 dnů (globálně, bez filtru)
- Nezaplacené faktury (stavy `vystavena` a `odeslana`)
- Faktury po splatnosti
- Graf tržeb (Chart.js) – hodnoty `castkaCelkem` faktur agregované po měsících

### 9.3 UzivatelController

**Soubor:** `src/Controller/UzivatelController.php`  
**Trasy:** `/uzivatele/*`

Správa uživatelských účtů. Zahrnuje CRUD operace, správu profilu (editace jména, e-mailu), změnu hesla (bcrypt hash přes `UserPasswordHasherInterface`) a přehled vlastního profilu.

### 9.4 Dokumentové controllery (Nabidka, Objednavka, Smlouva, AkceptacniProtokol, Faktura)

Všechny dokumentové controllery sdílejí společný vzor:

1. **index** – seznam entit s DataTables (s ACL filtrem pro `ROLE_USER`)
2. **new** – formulář pro vytvoření, nahrání souboru přes `FileUploader`
3. **show** – detail záznamu + odkaz pro stažení souboru
4. **edit** – formulář pro editaci, nahrazení souboru
5. **delete** – POST formulář pro smazání záznamu + smazání souboru z disku

Přístup v každé akci je ověřen pomocí `$this->denyAccessUnlessGranted('VIEW'/'EDIT'/'DELETE', $entity)` nebo pomocí atributu `#[IsGranted(...)]`.

### 9.5 Stahování souborů

Soubory jsou ukládány mimo `public/` do `var/uploads/`. **Přímý přístup přes URL není možný.** Stažení probíhá výhradně přes controller akci, která:

1. Ověří oprávnění (ACL voter)
2. Sestaví absolutní cestu k souboru
3. Vrátí `BinaryFileResponse` se správným `Content-Disposition` hlavičkou

---

## 10. Upload souborů

### 10.1 Implementace

Upload je realizován přes třídu `src/Service/FileUploader.php`.

```
UploadedFile (formulář)
    │
    ▼
FileUploader::upload()
    ├── Sanitizace názvu souboru (Slugger)
    ├── Generování unikátního názvu: {slug}-{uniqid}.{ext}
    ├── Přesun souboru do var/uploads/
    └── Vrátí metadata: fileName, originalName, mimeType, size, filePath
```

### 10.2 Konfigurace cílového adresáře

Cílový adresář je nastaven v `config/services.yaml` jako parametr injektovaný do `FileUploader`:

```yaml
services:
  App\Service\FileUploader:
    arguments:
      $targetDirectory: '%kernel.project_dir%/var/uploads'
```

### 10.3 Bezpečnostní aspekty

- Název souboru je **vždy přegenerován** – původní název je uložen pouze jako metadata (sloupec `fileName`), nikdy se nepoužívá jako název na disku
- Soubory jsou uloženy mimo webroot (`var/uploads/`) – přímé URL stažení není dostupné
- Stažení je podmíněno ACL kontrolou v controlleru
- Smazání záznamu z databáze automaticky spustí `FileUploader::remove()` pro odstranění fyzického souboru

### 10.4 Mazání souborů

`FileUploader::remove(string $filepath)` odstraní fyzický soubor. Metoda automaticky ošetřuje prefix `uploads/` v cestě a ověřuje existenci souboru před smazáním.

---

## 11. Reporty a dashboard

### 11.1 Dashboard (`/dashboard`)

| Widget | Zdroj dat | Popis |
|---|---|---|
| Moje aktivní projekty | `ProjektUzivatelRepository::findBy(['uzivatel' => $user, 'aktivni' => true])` | Projekty přiřazené přihlášenému uživateli |
| Nedávné nabídky | `NabidkaRepository::findByUser($user)` (nebo globálně pro admina) | Posledních 3 nabídky |
| Nedávné faktury | `FakturaRepository::findByUser($user)` (nebo globálně pro admina) | Posledních 3 faktury |

### 11.2 Report (`/reporty/`)

| Widget | Zdroj dat | Popis |
|---|---|---|
| Aktivní nabídky | `NabidkaRepository::countActive($from, $to)` | Počet nabídek ve zvoleném období |
| Aktivní objednávky | `ObjednavkaRepository::countActive($from, $to)` | Počet objednávek ve zvoleném období |
| Aktivní smlouvy | `SmlouvaRepository::countActive($from, $to)` | Počet smluv ve zvoleném období |
| Vypršení smluv | `SmlouvaRepository::findExpiringInDays(30)` | Smlouvy s blížícím se koncem platnosti |
| Nezaplacené faktury | `FakturaRepository::findUnpaid()` | Faktury ve stavech `vystavena`, `odeslana` |
| Faktury po splatnosti | `FakturaRepository::findOverdue()` | Faktury ve stavu `po_splatnosti` |
| Graf tržeb | `FakturaRepository::getRevenueByPeriod($from, $to)` | Součet `castkaCelkem` po měsících |

Graf je renderován pomocí **Chart.js** a data jsou předávána z PHP jako JSON (`json_encode`) do JavaScriptu v Twig šabloně.

---

## 12. Deployment na produkci (VPS)

Kompletní průvodce nasazením je v souboru **[DEPLOYMENT.md](../DEPLOYMENT.md)**. Níže je stručný přehled.

### 12.1 Aktuální varianta – Docker Compose

**Požadavky na server:**

| Položka | Verze |
|---|---|
| Ubuntu | 22.04 LTS nebo 24.04 LTS |
| PostgreSQL | 15 |
| Docker Engine | 24+ |
| Docker Compose plugin | 2+ |
| Composer | 2 |

**Postup nasazení (shrnutí):**

```bash
# Klonování
cd /var/www
git clone https://github.com/Jirka-Kotlas/maturitni-projekt.git projekt-manager
cd projekt-manager

# Nastavení .env.local (produkční proměnné)
cp .env .env.local
# Editovat .env.local: APP_ENV=prod, APP_SECRET, DATABASE_URL

# Build + spuštění
docker compose up -d --build

# Instalace + migrace + cache
docker compose exec app composer install --no-dev --optimize-autoloader --no-interaction
docker compose exec app php bin/console doctrine:migrations:migrate --no-interaction --env=prod
docker compose exec app php bin/console cache:clear --env=prod
docker compose exec app php bin/console cache:warmup --env=prod
```

Výchozí provoz běží bez reverse proxy. Pokud bude v budoucnu vyžadován vyšší hardening, lze doplnit reverse proxy vrstvu (Nginx/Caddy/Traefik) a TLS.

### 12.2 Volitelný budoucí hardening

Pro veřejný HTTPS endpoint a další bezpečnostní prvky je možné doplnit reverse proxy vrstvu. Není to součást aktuálního provozního modelu.

### 12.3 Aktualizace aplikace

```bash
cd /var/www/projekt-manager
git pull origin main
composer install --no-dev --optimize-autoloader
docker compose up -d --build
docker compose exec app php bin/console doctrine:migrations:migrate --no-interaction --env=prod
docker compose exec app php bin/console cache:warmup --env=prod
```

---

## 13. Testovací data (Fixtures)

Po spuštění `php bin/console doctrine:fixtures:load` jsou do databáze načtena testovací data:

### 13.1 Testovací uživatelé

| E-mail | Heslo | Role |
|---|---|---|
| `admin@test.cz` | `admin123` | `ROLE_ADMIN` |
| `manager@test.cz` | `manager123` | `ROLE_MANAGEMENT` |
| `user@test.cz` | `user123` | `ROLE_USER` |
| `marie@test.cz` | `marie123` | `ROLE_USER` |
| + 15 generovaných uživatelů | `password123` | `ROLE_USER` |

Celkem: **19 uživatelů**.

### 13.2 Rozsah testovacích dat

| Entita | Počet |
|---|---|
| Klienti | 17 (2 statické firmy + 15 generovaných) |
| Projekty | 38 (3 statické + 35 generovaných) |
| Nabídky | 47 |
| Objednávky | 11 |
| Smlouvy | 10 |
| Faktury | 50 |
| Akceptační protokoly | dle fixture |

Fixtures pro statusy vytvářejí záznamy pro všechny číselníky (`ProjektStatus`, `NabidkaStatus`, `ObjednavkaStatus`, `SmlouvaStatus`, `SmlouvaPodpisStatus`, `FakturaStatus`, `ProtokolStatus`).

---

## 14. Troubleshooting

### 14.1 Aplikace neběží po spuštění Docker Compose

**Symptom:** stránka je nedostupná, vrací 500 nebo kontejner `app` restartuje.

**Diagnostika:**

```bash
# Stav kontejnerů
docker compose ps

# Logy aplikace a DB
docker compose logs -f app
docker compose logs -f db

# Ověření Symfony/Doctrine uvnitř app kontejneru
docker compose exec app php bin/console about
docker compose exec app php bin/console doctrine:schema:validate
```

**Typické příčiny:**
- neplatné nebo chybějící hodnoty v `.env.local` (`DATABASE_URL`, `POSTGRES_*`),
- nespárované proměnné mezi `app` a `db` službou,
- nespuštěné migrace po čistém startu prostředí.

### 14.2 Nelze se připojit k databázi

**Symptom:** chyba připojení (`connection refused`, `SQLSTATE[08006]`, apod.).

**Poznámka k portům:**
- uvnitř Docker sítě používá aplikace host `db:5432`,
- z hostitele je DB mapována na `127.0.0.1:5433`.

**Kontrola:**

```bash
# Aplikace -> DB přes interní host "db"
docker compose exec app printenv | grep DATABASE_URL

# Připojení přímo do DB kontejneru
docker compose exec db psql -U "$POSTGRES_USER" -d "$POSTGRES_DB"
```

### 14.3 Migrace selhávají nebo schéma je nekonzistentní

**Symptom:** `doctrine:migrations:migrate` skončí chybou nebo `schema:validate` hlásí chyby mapování.

**Diagnostika a náprava:**

```bash
# Stav migrací
docker compose exec app php bin/console doctrine:migrations:status

# Spuštění migrací
docker compose exec app php bin/console doctrine:migrations:migrate --no-interaction

# Validace mapování a schématu
docker compose exec app php bin/console doctrine:schema:validate
```

**Historicky řešený případ v projektu:**
- konflikt mapování mezi `Klient` a `Smlouva` (direct relation `Klient -> Smlouva` neodpovídala realitě modelu),
- aktuálně je vazba vedena přes `Projekt` (`Smlouva -> Projekt -> Klient`).

Pokud se podobná chyba objeví znovu, nejdřív ověřte obě strany asociace v entitách a následně spusťte validaci schématu.

### 14.4 Chyba 403 (Access Denied)

**Symptom:** uživatel je přihlášen, ale u detailu/akce dostane 403.

**Realita projektu:**
- přístup není řízen jen rolemi, ale i ACL logikou přes Voters (`ProjectAccessVoter`, `ProjektVoter`, `KlientVoter`),
- `ROLE_USER` musí být typicky přiřazen k projektu přes `projekt_uzivatel`.

**Kontrola:**

```bash
# Ověřte základní routy a zabezpečení
docker compose exec app php bin/console debug:router | grep -E "dashboard|projekt|faktur|smlouv"
```

Pak ověřte, že uživatel má odpovídající roli a aktivní přiřazení k projektu.

### 14.5 Po přihlášení 404 na dashboardu

**Symptom:** po loginu aplikace míří na neexistující route (často podle starší dokumentace).

**Vysvětlení:**
- aktuální aplikace používá jednotný dashboard `/dashboard`,
- historické cesty typu `/admin/dashboard` nebo `/management/dashboard` nejsou součástí současného routingu.

**Kontrola:**

```bash
docker compose exec app php bin/console debug:router | grep dashboard
```

### 14.6 Neplatný CSRF token při mazání / odeslání formuláře

**Symptom:** flash hláška „Neplatný token.“ nebo zamítnutý POST request.

**Nejčastější příčiny:**
- vypršela session,
- formulář byl otevřen dlouho ve starém tabu,
- cache prohlížeče drží starý formulář.

**Doporučený postup:**
- obnovit stránku a akci zopakovat,
- zkontrolovat, že formulář posílá správný `_token`,
- v prostředí serveru ověřit stabilní `APP_SECRET` (zejména v produkci).

### 14.7 Reset hesla neodesílá e-mail

**Symptom:** uživatel odešle formulář, ale e-mail nedorazí.

**Důležité chování aplikace:**
- flow schválně neprozrazuje, zda e-mail v systému existuje,
- uživatel je vždy přesměrován na „check email“ stránku.

**Kontrola konfigurace:**
- `MAILER_DSN` v `.env.local`,
- `MAILER_FROM_ADDRESS` a `MAILER_FROM_NAME`,
- logy aplikace (`var/log/prod.log` nebo `var/log/dev.log`).

Pokud je `MAILER_DSN=null://null`, e-maily se reálně neposílají (vhodné pro lokální vývoj).

### 14.8 Nelze nahrát nebo stáhnout soubor

**Symptom:** upload selže, nebo dokument nejde stáhnout.

**Kontrola práv a umístění:**

```bash
sudo chown -R www-data:www-data var/
sudo chmod -R 775 var/
sudo mkdir -p var/uploads
sudo chown -R www-data:www-data var/uploads
sudo chmod -R 775 var/uploads
```

**Poznámka:**
- soubory jsou ukládány do `var/uploads` (mimo webroot),
- download probíhá přes controller a ACL, ne přímým URL přístupem do souborového systému.

### 14.9 Produkce po deploy vrací 500

**Symptom:** po `git pull` aplikace padá na 500.

**Standardní recovery postup:**

```bash
composer install --no-dev --optimize-autoloader --no-interaction
php bin/console doctrine:migrations:migrate --no-interaction --env=prod
php bin/console cache:clear --env=prod
php bin/console cache:warmup --env=prod
sudo chown -R www-data:www-data var/
sudo chmod -R 775 var/
```

### 14.10 Užitečné diagnostické příkazy

```bash
# Router
docker compose exec app php bin/console debug:router

# Kontejnerové logy
docker compose logs -f app
docker compose logs -f db

# Symfony logy v projektu
tail -f var/log/dev.log
tail -f var/log/prod.log
```

U starších návodů může být použito `docker-compose` (s pomlčkou). V tomto projektu fungují obě varianty CLI podle verze Dockeru.

---

*Dokument odpovídá stavu repozitáře k březnu 2026.*
