# GHB Merch – Technická dokumentace

> Oficiální e-shop pro školní merch Gymnázia Havlíčkův Brod. Webová aplikace pro správu produktů, hromadných objednávek a jejich výdeje s QR kódy.

---

## Obsah

1. [Přehled projektu](#1-přehled-projektu)
2. [Architektura systému](#2-architektura-systému)
3. [Technologický stack](#3-technologický-stack)
4. [Instalace a spuštění](#4-instalace-a-spuštění)
5. [Konfigurace prostředí](#5-konfigurace-prostředí)
6. [Databázové schéma](#6-databázové-schéma)
7. [Autentizace a autorizace](#7-autentizace-a-autorizace)
8. [API dokumentace](#8-api-dokumentace)
9. [Struktura routování](#9-struktura-routování)
10. [Uživatelská příručka – koncový uživatel](#10-uživatelská-příručka--koncový-uživatel)
11. [Uživatelská příručka – administrátor](#11-uživatelská-příručka--administrátor)
12. [Emailový systém](#12-emailový-systém)
13. [Správa souborů a obrázků](#13-správa-souborů-a-obrázků)
14. [Bezpečnost](#14-bezpečnost)
15. [Deployment a produkční nasazení](#15-deployment-a-produkční-nasazení)
16. [Monitoring a logování](#16-monitoring-a-logování)

---

## 1. Přehled projektu

**GHB Merch** je webová aplikace sloužící jako e-shop pro školní merch Gymnázia Havlíčkův Brod. Systém umožňuje studentům a zaměstnancům školy objednávat školní oblečení a doplňky prostřednictvím systému hromadných objednávek.

### Klíčové funkce

- **Katalog produktů** – procházení produktů s filtry, barevnými variantami a velikostmi
- **Hromadné objednávky (batch)** – sběr objednávek do hromadné objednávky s minimálním počtem kusů
- **Nákupní košík** – localStorage košík se synchronizací mezi taby
- **Správa objednávek** – lifecycle objednávky od vytvoření po vyzvednutí
- **QR kódy** – generování a skenování QR kódů pro ověření při výdeji
- **Admin panel** – kompletní správa produktů, objednávek, uživatelů a statistik
- **Emailové notifikace** – automatické emaily pro potvrzení, změny stavů a admin upozornění
- **Microsoft SSO** – přihlášení přes školní Microsoft účet (@ghb.cz)
- **Optimalizace obrázků** – automatická konverze do AVIF formátu

### Cílová skupina

| Role | Popis |
|------|-------|
| **Student** | Prohlíží produkty, vytváří objednávky, sleduje jejich stav |
| **Zaměstnanec** | Stejné jako student |
| **Admin** | Spravuje produkty, objednávky, hromadné objednávky, uživatele |
| **Superadmin** | Plný přístup včetně správy administrátorů |

---

## 2. Architektura systému

### Celkový přehled

```
┌─────────────┐     ┌──────────┐     ┌──────────────┐     ┌────────────┐
│  Prohlížeč  │────▶│  Caddy   │────▶│  SvelteKit   │────▶│ PostgreSQL │
│  (klient)   │◀────│  (proxy) │◀────│  (Node.js)   │◀────│    (DB)    │
└─────────────┘     └──────────┘     └──────────────┘     └────────────┘
                         │
                    ┌────┴────┐
                    │ Uploads │
                    │ (AVIF)  │
                    └─────────┘
```

### Architektonické vrstvy

```
src/
├── routes/                    # Prezentační vrstva (stránky + API endpointy)
│   ├── +page.svelte           # Hlavní stránka
│   ├── admin/                 # Admin panel (chráněno rolí)
│   ├── api/                   # REST API endpointy
│   ├── prihlaseni/            # Přihlášení
│   ├── registrace/            # Registrace
│   ├── produkty/              # Katalog produktů
│   ├── produkt/               # Detail produktu
│   ├── kosik/                 # Nákupní košík
│   ├── objednavka/            # Proces objednávky
│   └── moje-objednavky/       # Objednávky uživatele
├── lib/
│   ├── server/                # Serverová vrstva
│   │   ├── db/                # Datová vrstva (Drizzle ORM)
│   │   │   ├── schemas/       # Databázová schémata
│   │   │   └── index.ts       # Připojení k DB
│   │   ├── auth.ts            # Autentizace (better-auth)
│   │   ├── nodemailer.ts      # Emailový systém
│   │   ├── storage.ts         # Správa souborů (sharp)
│   │   ├── security.ts        # QR bezpečnost (HMAC)
│   │   └── http-logger.ts     # HTTP logování
│   ├── client/                # Klientská vrstva
│   │   └── auth-client.ts     # Auth klient pro prohlížeč
│   ├── components/            # Svelte komponenty
│   ├── stores/                # Svelte stores (košík)
│   ├── utils/                 # Utility funkce
│   └── shared/                # Sdílený kód (barvy)
└── hooks.server.ts            # Server hooks (middleware)
```

---

## 3. Technologický stack

| Kategorie | Technologie | Verze | Účel |
|-----------|-------------|-------|------|
| **Framework** | SvelteKit | 2.x | Fullstack framework |
| **Jazyk** | TypeScript | 5.x | Typový systém |
| **Runtime** | Node.js | 24 | Server runtime |
| **Databáze** | PostgreSQL | 16 | Relační databáze |
| **ORM** | Drizzle ORM | 0.44+ | Databázová vrstva |
| **Auth** | better-auth | 1.3+ | Autentizace |
| **Email** | Nodemailer | 7.x | SMTP emaily (Gmail) |
| **Obrázky** | Sharp | 0.34+ | Optimalizace obrázků |
| **QR** | qrcode + html5-qrcode | — | Generování a skenování QR |
| **Proxy** | Caddy | 2.x | Reverse proxy, HTTPS |
| **Kontejnery** | Docker + Docker Compose | — | Kontejnerizace |
| **Balíčkovač** | pnpm | latest | Správa závislostí |

---

## 4. Instalace a spuštění

### Předpoklady

- **Node.js** 18+ (doporučeno 24)
- **pnpm** (`corepack enable && corepack prepare pnpm@latest --activate`)
- **Docker** a **Docker Compose** (pro databázi)
- **Git**

### Lokální vývoj

```bash
# 1. Klonování repozitáře
git clone <url-repozitáře>
cd school-merch

# 2. Instalace závislostí
pnpm install

# 3. Konfigurace prostředí
cp .env.example .env
# Upravte .env soubor podle sekce 5

# 4. Spuštění databáze
pnpm db:start
# Nebo: docker compose up -d

# 5. Push schématu do databáze
pnpm db:push

# 6. (Volitelně) Seed produktů
# psql < seed-products.sql

# 7. Spuštění dev serveru
pnpm dev
```

Aplikace bude dostupná na `http://localhost:5173`.

### Dostupné skripty

| Příkaz | Popis |
|--------|-------|
| `pnpm dev` | Spustí vývojový server |
| `pnpm build` | Sestaví produkční build |
| `pnpm preview` | Náhled produkčního buildu |
| `pnpm check` | Kontrola TypeScript typů |
| `pnpm db:start` | Spustí PostgreSQL v Dockeru |
| `pnpm db:push` | Pushne schéma do DB |
| `pnpm db:generate` | Vygeneruje SQL migrace |
| `pnpm db:migrate` | Aplikuje migrace |
| `pnpm db:studio` | Spustí Drizzle Studio (GUI) |

---

## 5. Konfigurace prostředí

### Proměnné prostředí (.env)

| Proměnná | Povinná | Popis | Příklad |
|----------|---------|-------|---------|
| `DATABASE_URL` | ✅ | PostgreSQL connection string | `postgres://admin:admin@localhost:5432/school_merch` |
| `BETTER_AUTH_SECRET` | ✅ | Secret klíč pro autentizaci a HMAC podepisování QR | `náhodný-bezpečný-řetězec` |
| `BETTER_AUTH_URL` | ✅ | Veřejná URL aplikace | `https://merch.jakubjakubek.cz` |
| `APP_USER` | ✅ | Gmail adresa pro SMTP | `email@gmail.com` |
| `APP_PASSWORD` | ✅ | Gmail App Password | `xxxx xxxx xxxx xxxx` |
| `MICROSOFT_CLIENT_ID` | ✅ | Microsoft OAuth Client ID | `uuid` |
| `MICROSOFT_CLIENT_SECRET` | ✅ | Microsoft OAuth Secret | `secret` |

### Docker-specifické proměnné

| Proměnná | Výchozí | Popis |
|----------|---------|-------|
| `POSTGRES_USER` | `admin` | Uživatel PostgreSQL |
| `POSTGRES_PASSWORD` | `admin` | Heslo PostgreSQL |
| `POSTGRES_DB` | `school_merch` | Název databáze |
| `DRIZZLE_STUDIO_PORT` | `4983` | Port pro Drizzle Studio |

---

## 6. Databázové schéma

### ER diagram

```
┌──────────┐    ┌──────────────┐    ┌──────────┐
│   user   │    │   orders     │    │ batches  │
├──────────┤    ├──────────────┤    ├──────────┤
│ id (PK)  │◀──│ user_id (FK) │    │ id (PK)  │
│ name     │    │ batch_id(FK) │──▶│ name     │
│ email    │    │ status       │    │ status   │
│ role     │    │ total_amount │    │ closed_at│
│ phone    │    │ payment_meth.│    └──────────┘
│ notify*  │    └──────┬───────┘
└──────────┘           │
                       │ 1:N
                ┌──────┴───────┐    ┌──────────┐
                │ order_items  │    │ products │
                ├──────────────┤    ├──────────┤
                │ order_id(FK) │    │ id (PK)  │
                │ product_id   │──▶│ name     │
                │ size, color  │    │ price_min│
                │ quantity     │    │ images[] │
                └──────────────┘    │ sizes[]  │
                                    │ colors[] │
┌──────────────────────┐            └────┬─────┘
│ order_change_requests│                 │ M:N
├──────────────────────┤          ┌──────┴──────┐
│ order_id (FK)        │          │ products_to │
│ type (cancel/modify) │          │ _labels     │
│ reason               │          └──────┬──────┘
│ status               │                 │
└──────────────────────┘          ┌──────┴──────┐
                                  │   labels    │
                                  ├─────────────┤
                                  │ name        │
                                  │ text_color  │
                                  │ bg_color    │
                                  └─────────────┘
```

### Tabulka: `user`

| Sloupec | Typ | Popis |
|---------|-----|-------|
| `id` | `text` PK | Unikátní ID uživatele |
| `name` | `text` NOT NULL | Jméno |
| `email` | `text` UNIQUE | Email |
| `email_verified` | `boolean` | Ověřený email |
| `image` | `text` | URL avataru |
| `phone` | `text` | Telefonní číslo |
| `notification_email` | `text` | Alternativní email pro notifikace |
| `role` | `text` | Role: `user`, `admin`, `superadmin` |
| `notify_order_status` | `boolean` | Notifikace o změnách objednávek |
| `notify_new_order` | `boolean` | Notifikace o nových objednávkách (admin) |
| `notify_new_request` | `boolean` | Notifikace o nových žádostech (admin) |
| `created_at` | `timestamp` | Datum vytvoření |
| `updated_at` | `timestamp` | Datum poslední úpravy |

### Tabulka: `products`

| Sloupec | Typ | Popis |
|---------|-----|-------|
| `id` | `uuid` PK | UUID produktu |
| `name` | `text` NOT NULL | Název produktu |
| `price_min` | `integer` | Minimální orientační cena (Kč) |
| `price_max` | `integer` | Maximální orientační cena (Kč) |
| `description` | `text` NOT NULL | Popis produktu |
| `images` | `jsonb` | Pole URL obrázků – `string[]` |
| `sizes` | `jsonb` | Pole velikostí – `string[]` (S, M, L…) |
| `colors` | `jsonb` | Pole barevných variant – `ProductColor[]` |
| `genders` | `jsonb` | Pohlaví – `("muži" \| "ženy" \| "unisex")[]` |
| `off_sale` | `boolean` | Staženo z prodeje |
| `featured` | `boolean` | Zobrazit na hlavní stránce |
| `minimum_to_order` | `integer` | Minimum kusů pro hromadnou objednávku |
| `created_at` | `timestamp` | Datum vytvoření |
| `updated_at` | `timestamp` | Datum poslední úpravy |

**Typ `ProductColor`:**
```typescript
{
  name: string;    // "Červená", "Černá"
  hexCode: string; // "#FF0000"
  images: string[]; // Obrázky specifické pro barvu
}
```

### Tabulka: `orders`

| Sloupec | Typ | Popis |
|---------|-----|-------|
| `id` | `uuid` PK | UUID objednávky |
| `user_id` | `text` FK → `user.id` | Vlastník (SET NULL on delete) |
| `customer_name` | `text` NOT NULL | Snapshot jména |
| `customer_email` | `text` NOT NULL | Snapshot emailu |
| `customer_phone` | `text` | Snapshot telefonu |
| `payment_method` | `enum` | `cash`, `card`, `bank_transfer` |
| `pickup_location` | `text` NOT NULL | Místo vyzvednutí |
| `batch_id` | `uuid` FK → `batches.id` | Hromadná objednávka |
| `total_amount` | `integer` | Celková cena (NULL dokud není naceněno) |
| `status` | `enum` | Stav objednávky (viz níže) |
| `customer_note` | `text` | Poznámka zákazníka |
| `admin_note` | `text` | Interní poznámka |
| `created_at` | `timestamp` | Datum vytvoření |
| `completed_at` | `timestamp` | Datum vyzvednutí |

**Stavy objednávky (`order_status`):**

| Stav | Popis |
|------|-------|
| `pending_batch` | Zapsáno, čeká na hromadnou objednávku |
| `processing` | Vydáno do výroby |
| `ready` | Připraveno k vyzvednutí |
| `completed` | Vyzvednuto a zaplaceno |
| `cancelled` | Zrušeno |

### Tabulka: `order_items`

| Sloupec | Typ | Popis |
|---------|-----|-------|
| `id` | `uuid` PK | UUID položky |
| `order_id` | `uuid` FK | Objednávka (CASCADE delete) |
| `product_id` | `uuid` FK | Produkt (RESTRICT delete) |
| `product_name` | `text` | Snapshot názvu |
| `product_price` | `integer` | Cena za kus (nastaveno později) |
| `product_image` | `text` | Snapshot obrázku |
| `size` | `text` | Velikost |
| `color` | `text` | Barva |
| `quantity` | `integer` | Počet kusů |
| `subtotal` | `integer` | Mezisoučet |

### Tabulka: `batches`

| Sloupec | Typ | Popis |
|---------|-----|-------|
| `id` | `uuid` PK | UUID hromadné objednávky |
| `name` | `text` NOT NULL | Název (např. „Zima 2024") |
| `status` | `enum` | `open`, `closed`, `received`, `completed` |
| `closed_at` | `timestamp` | Datum uzavření |
| `completed_at` | `timestamp` | Datum dokončení |

### Tabulka: `order_change_requests`

| Sloupec | Typ | Popis |
|---------|-----|-------|
| `id` | `uuid` PK | UUID žádosti |
| `order_id` | `uuid` FK | Objednávka |
| `type` | `enum` | `cancel` nebo `modify` |
| `reason` | `text` NOT NULL | Důvod žádosti |
| `status` | `enum` | `pending`, `approved`, `rejected` |
| `admin_note` | `text` | Poznámka admina |
| `resolved_at` | `timestamp` | Datum vyřízení |

### Tabulka: `labels` + `products_to_labels`

Štítky produktů (M:N relace) s vlastním textem a barvou pozadí/textu.

### Tabulky: `hero_images` + `hero_settings`

Obrázky pro hero slider na hlavní stránce s konfigurovatelným intervalem přepínání.

### Auth tabulky: `session`, `account`, `verification`

Spravovány knihovnou better-auth (sessions, OAuth účty, verifikační tokeny).

---

## 7. Autentizace a autorizace

### Metody přihlášení

1. **Email + heslo** – klasická registrace a přihlášení
2. **Microsoft SSO** – pro školní účty @ghb.cz (tenant `217f80c7-...`)

> **Pravidlo:** Uživatelé s emailem `@ghb.cz` se nemohou registrovat přes email/heslo – musí použít Microsoft SSO.

### Role

| Role | Oprávnění |
|------|-----------|
| `user` | Prohlížení produktů, vytváření objednávek, správa vlastního účtu |
| `admin` | Vše z `user` + admin panel (produkty, objednávky, uživatelé, statistiky) |
| `superadmin` | Vše z `admin` + správa rolí dalších uživatelů |

### Ochrana admin routy

Admin layout (`/admin/+layout.server.ts`) kontroluje:
1. Přihlášení – nepřihlášení uživatelé jsou přesměrováni na `/prihlaseni`
2. Roli – uživatelé bez role `admin`/`superadmin` jsou přesměrováni na `/`

### Server hooks (middleware)

Soubor `hooks.server.ts` při každém požadavku:
1. Načítá session z better-auth
2. Ukládá `user` a `session` do `event.locals`
3. Obaluje vše HTTP logováním

---

## 8. API dokumentace

### Veřejné API (v1)

#### `GET /api/v1/products`
Vrací seznam všech produktů.

**Response:**
```json
{
  "success": true,
  "data": [{ "id": "uuid", "name": "...", "images": [...], ... }]
}
```

#### `GET /api/v1/products/:id`
Vrací detail produktu.

#### `POST /api/v1/products`
Vytvoří nový produkt. Vyžaduje `name`, `price`, `description`.

#### `PUT /api/v1/products/:id`
Aktualizuje produkt – parciální update.

#### `DELETE /api/v1/products/:id`
Smaže produkt.

---

### Objednávky

#### `POST /api/orders/create`
Vytvoří novou objednávku. Vyžaduje přihlášení.

**Request body:**
```json
{
  "items": [{ "productId": "uuid", "name": "...", "price": 500, "size": "M", "color": "Černá", "quantity": 1, "image": "/uploads/..." }],
  "total": 500,
  "pickupLocation": "Kabinet zeměpisu",
  "paymentMethod": "cash",
  "customerNote": "Volitelná poznámka"
}
```

**Logika:** Automaticky najde otevřenou hromadnou objednávku (batch), nebo vytvoří novou. Odešle potvrzovací email zákazníkovi a notifikaci adminům.

#### `GET /api/orders`
Vrací objednávky přihlášeného uživatele s položkami.

---

### Upload

#### `POST /api/upload`
Nahraje obrázek. Přijímá `multipart/form-data` s polem `image`.

**Response:** `{ "url": "/uploads/hash.avif" }`

---

### Uživatel

#### `POST /api/user/update`
Aktualizuje profil (telefon, notifikační email, nastavení notifikací).

#### `POST /api/user/change-password`
Změna hesla.

#### `POST /api/user/delete`
Smazání účtu.

---

### Admin API

| Endpoint | Metoda | Popis |
|----------|--------|-------|
| `/api/admin/products` | POST | Vytvoření produktu (admin) |
| `/api/admin/products/:id` | PUT/DELETE | Úprava/smazání produktu |
| `/api/admin/orders/:id/status` | POST | Změna stavu objednávky |
| `/api/admin/orders/:id/note` | POST | Přidání admin poznámky |
| `/api/admin/orders/bulk-update` | POST | Hromadná změna stavů |
| `/api/admin/hero` | POST | Správa hero obrázků |
| `/api/admin/images` | GET/DELETE | Správa nahraných obrázků |
| `/api/admin/cleanup-images` | POST | Vyčištění nepoužívaných obrázků |

### Health check

#### `GET /api/health`
Kontrola dostupnosti aplikace a databáze.

**Response:**
```json
{ "status": "ok", "uptime": 12345.67, "database": "connected" }
```

---

## 9. Struktura routování

### Veřejné stránky

| Cesta | Popis |
|-------|-------|
| `/` | Hlavní stránka (hero slider, featured produkty, batch progress) |
| `/produkty` | Katalog všech produktů |
| `/produkt/[id]` | Detail produktu |
| `/kosik` | Nákupní košík |
| `/objednavka` | Proces vytvoření objednávky |
| `/moje-objednavky` | Seznam objednávek uživatele |
| `/muj-ucet` | Správa profilu |
| `/prihlaseni` | Přihlášení |
| `/registrace` | Registrace |
| `/zapomenute-heslo` | Žádost o reset hesla |
| `/reset-hesla` | Nastavení nového hesla |
| `/jak-to-funguje` | Informační stránka |
| `/podminky-uzivani` | Podmínky užívání |
| `/ochrana-osobnich-udaju` | GDPR |

### Admin stránky

| Cesta | Popis |
|-------|-------|
| `/admin` | Dashboard |
| `/admin/produkty` | Správa produktů |
| `/admin/produkty/novy` | Nový produkt |
| `/admin/produkty/[id]` | Editace produktu |
| `/admin/objednavky` | Seznam objednávek |
| `/admin/objednavky/[id]` | Detail objednávky |
| `/admin/hromadne-objednavky` | Správa batchů |
| `/admin/hromadne-objednavky/[id]` | Detail batche |
| `/admin/uzivatele` | Správa uživatelů |
| `/admin/zadosti` | Žádosti o změnu objednávek |
| `/admin/statistiky` | Statistiky prodeje |
| `/admin/stitky` | Správa štítků |
| `/admin/galerie` | Správa obrázků |
| `/admin/hero` | Nastavení hero slideru |
| `/admin/scanner` | QR skener pro výdej |
| `/admin/qr/[id]` | QR kód objednávky |

---

## 10. Uživatelská příručka – koncový uživatel

### Registrace a přihlášení

1. Přejděte na `/registrace` nebo `/prihlaseni`
2. **Studenti a zaměstnanci GHB:** Použijte tlačítko „Přihlásit se přes Microsoft" se školním účtem @ghb.cz
3. **Ostatní:** Vytvořte účet pomocí emailu a hesla

### Prohlížení produktů

- Na hlavní stránce vidíte doporučené produkty a progress bar hromadné objednávky
- V sekci `/produkty` najdete kompletní katalog
- U každého produktu vidíte cenové rozmezí, velikosti a barevné varianty

### Vytvoření objednávky

1. Vyberte produkt, velikost, barvu a přidejte do košíku
2. V košíku (`/kosik`) zkontrolujte položky a upravte množství
3. Pokračujte k objednávce – vyplňte místo vyzvednutí a způsob platby
4. Obdržíte potvrzovací email se shrnutím

### Sledování objednávky

- V sekci `/moje-objednavky` vidíte všechny své objednávky
- Stavy: **Zapsáno** → **Ve výrobě** → **Připraveno** → **Vyzvednuto**
- Můžete podat žádost o zrušení nebo změnu objednávky

### Vyzvednutí

- Když je objednávka **Připravena k vyzvednutí**, obdržíte email
- V detailu objednávky najdete QR kód pro ověření
- Při vyzvednutí ukažte QR kód administrátorovi

---

## 11. Uživatelská příručka – administrátor

### Přístup do admin panelu

Přejděte na `/admin`. Musíte mít roli `admin` nebo `superadmin`.

### Správa produktů

- **Nový produkt:** `/admin/produkty/novy` – vyplňte název, popis, ceny, velikosti, barvy, obrázky
- **Editace:** Klikněte na produkt v seznamu → upravte libovolná pole
- **Obrázky:** Nahrávejte přes drag & drop, automaticky se optimalizují do AVIF
- **Štítky:** Přiřaďte produktům štítky (Novinka, Skladem apod.) přes `/admin/stitky`

### Správa objednávek

- **Seznam:** `/admin/objednavky` – filtrujte podle stavu, batche, uživatele
- **Změna stavu:** V detailu objednávky změňte stav – zákazník obdrží email
- **Hromadná aktualizace:** Vyberte více objednávek a změňte stav najednou
- **Admin poznámky:** Přidejte interní poznámky k objednávkám

### Hromadné objednávky (batch)

1. Při první objednávce se automaticky vytvoří otevřený batch
2. V `/admin/hromadne-objednavky` můžete batch pojmenovat
3. **Uzavření batche:** Změní stav všech objednávek na `processing` (ve výrobě)
4. **Dokončení:** Po přijetí zboží a výdeji všech objednávek

### Výdej přes QR

1. Přejděte na `/admin/scanner`
2. Naskenujte QR kód z telefonu zákazníka
3. Systém ověří podpis HMAC – pokud je QR platný, zobrazí detail objednávky
4. Potvrďte výdej – stav se změní na `completed`

### Správa uživatelů

- `/admin/uzivatele` – seznam všech uživatelů s informacemi o třídě (parsováno z emailu)
- Změna rolí (pouze superadmin)

### Žádosti o změnu

- `/admin/zadosti` – přehled žádostí o zrušení/změnu objednávek
- Schvalte nebo zamítněte s poznámkou – zákazník obdrží email

---

## 12. Emailový systém

### Konfigurace

Emaily se odesílají přes **Gmail SMTP** (port 465, SSL) pomocí Nodemailer. Vyžaduje Gmail App Password.

### Typy emailů

| Typ | Příjemce | Kdy se odesílá |
|-----|----------|----------------|
| `orderConfirmation` | Zákazník | Po vytvoření objednávky |
| `orderStatusChange` | Zákazník | Při změně stavu objednávky |
| `newOrder` | Admini | Nová objednávka (pokud mají zapnuté notifikace) |
| `newRequest` | Admini | Nová žádost o změnu |
| `requestStatusChange` | Zákazník | Vyřízení žádosti (schválení/zamítnutí) |
| `minOrderReached` | Admini | Dosažení minimálního počtu kusů |
| `resetPassword` | Uživatel | Žádost o obnovení hesla |

Všechny emaily používají jednotný HTML šablonový systém s branding „GHB Merch" a oranžovou (#fd8549) color paletou.

---

## 13. Správa souborů a obrázků

### Proces uploadu

1. Klient odešle obrázek přes `POST /api/upload` jako `multipart/form-data`
2. Server vypočítá **SHA-256 hash** z původního souboru (pro deduplikaci)
3. Zkontroluje, zda soubor s tímto hashem již existuje (early exit)
4. Optimalizuje obrázek pomocí **Sharp**: resize na max 1200px šířky, konverze do **AVIF** (quality 65, effort 1)
5. Uloží do složky `uploads/` s názvem `{hash}.avif`
6. Vrátí URL `/uploads/{hash}.avif`

### Konfigurace optimalizace

| Parametr | Slabý hosting | Výkonný hosting |
|----------|---------------|-----------------|
| `IMAGE_MAX_WIDTH` | 1200 | 1600 |
| `IMAGE_QUALITY` | 60–70 | 80 |
| `IMAGE_EFFORT` | 0–1 | 4 |

### Servírování

V produkci servíruje obrázky přímo **Caddy** (statické soubory) s `Cache-Control: public, max-age=31536000` – obejde Node.js server.

---

## 14. Bezpečnost

### QR kódy

QR kódy pro výdej objednávek jsou podepsány pomocí **HMAC SHA-256**:

```
signature = HMAC-SHA256(BETTER_AUTH_SECRET, "qr:" + orderId).hex().slice(0, 16)
```

Při skenování admin scanner ověří podpis – falešné QR kódy jsou odmítnuty bez vysvětlení.

### HTTP bezpečnostní hlavičky (Caddy)

| Hlavička | Hodnota |
|----------|---------|
| `X-Content-Type-Options` | `nosniff` |
| `X-Frame-Options` | `DENY` |
| `Referrer-Policy` | `strict-origin-when-cross-origin` |
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains; preload` |
| `Permissions-Policy` | `interest-cohort=()` |

### Další bezpečnostní opatření

- HTTPS automaticky přes Caddy (Let's Encrypt)
- Session management přes better-auth (httpOnly cookies)
- CSRF ochrana (SvelteKit built-in)
- Rate limiting na Caddy úrovni
- Deduplikace obrázků přes hash (prevence DoS)
- `BODY_SIZE_LIMIT=50MB` nastaveno v Dockerfile

---

## 15. Deployment a produkční nasazení

### Architektura deploymentu

```
┌────────────────────────── Docker Compose ──────────────────────────┐
│                                                                     │
│  ┌─────────┐   ┌──────────┐   ┌──────────┐   ┌────────────────┐  │
│  │ Caddy   │──▶│ SvelteKit│──▶│PostgreSQL│   │ Drizzle Studio │  │
│  │ :80/:443│   │  :3000   │   │  :5432   │   │  :4983 (opt.)  │  │
│  └─────────┘   └──────────┘   └──────────┘   └────────────────┘  │
│       │              │                                              │
│       └──── /uploads ┘  (shared volume: app_uploads)               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Spuštění produkce

```bash
# Spuštění
docker compose -f docker-compose.prod.yml up -d --build

# Zastavení
docker compose -f docker-compose.prod.yml down

# Logy
docker compose -f docker-compose.prod.yml logs -f

# Drizzle Studio (volitelné)
docker compose -f docker-compose.prod.yml --profile tools up -d --build

# Migrace databáze
docker compose -f docker-compose.prod.yml exec app pnpm db:push
```

### Docker images

- **app:** Multi-stage build – Node 24 Alpine, build + produkční runtime
- **db:** PostgreSQL 16 Alpine s healthcheckem
- **caddy:** Caddy 2 s vlastním Caddyfile

### Volumes

| Volume | Účel |
|--------|------|
| `postgres_data` | Persistentní data PostgreSQL |
| `app_uploads` | Nahrané obrázky (sdíleno mezi app a caddy) |
| `caddy_data` | TLS certifikáty |
| `caddy_config` | Caddy konfigurace |

### Health check

Aplikace poskytuje endpoint `GET /api/health` – Docker ho kontroluje každých 5s s 10s start periodem.

### DNS konfigurace

Caddy automaticky obsluhuje doménu `merch.jakubjakubek.cz` – stačí nasměrovat A záznam na IP serveru.

---

## 16. Monitoring a logování

### HTTPLogger

Každý HTTP request je logován do konzole s barevným formátováním:

```
[14:32:15] GET     200 45ms  /produkty [user: Jan Novák]
[14:32:16] POST    201 123ms /api/orders/create [user: Marie K.]
[14:32:17] GET     404 12ms  /neexistuje
```

**Barvy:**
- **Metody:** GET=zelená, POST=modrá, PUT=žlutá, DELETE=červená
- **Status:** 2xx=zelená, 3xx=cyan, 4xx=žlutá, 5xx=červená
- **Rychlost:** <100ms=zelená, <500ms=žlutá, >500ms=červená

### Caddy logy

Caddy loguje ve formátu JSON na stdout – zachytitelné přes `docker logs`.

### Doporučení pro monitoring

- Sledujte endpoint `/api/health` externím monitoringem (UptimeRobot apod.)
- Sledujte Docker logy pro chybové stavy
- Nastavte alerting na 5xx chyby v Caddy logu

---

*Poslední aktualizace: březen 2026*
*Verze dokumentace: 1.0*
