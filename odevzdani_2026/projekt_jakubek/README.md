# School Merch

E-shop a administrační systém pro správu a prodej školního merche. Projekt je postaven na moderním webovém stacku s důrazem na rychlost, bezpečnost a čistou architekturu.

## 🚀 Technologický stack
- **Frontend / Framework:** SvelteKit + Svelte 5 (využívá Runes: `$state`, `$derived`, atd.)
- **Styling:** Čisté CSS (CSS proměnné, bez Tailwindu)
- **Databáze:** PostgreSQL
- **ORM:** Drizzle ORM
- **Autentizace:** Better Auth
- **Package manager:** pnpm

## 📁 Popis obsahu (Struktura repozitáře)
- `src/routes/` - Hlavní routy aplikace, rozdělené na klientskou část, API (`/api`) a administraci (`/admin`).
- `src/lib/components/` - Znovupoužitelné Svelte komponenty.
- `src/lib/server/db/schemas/` - Definice databázových tabulek (Drizzle schémata modularizovaná do jednotlivých souborů).
- `src/lib/server/` - Serverová logika, připojení k databázi (`db`) a konfigurace autentizace (`auth.ts`).
- `src/lib/client/` - Klientské utility (např. klient pro Better Auth: `auth-client.ts`).
- `src/lib/stores/` - Klientské Svelte stores pro stavovou logiku.
- `AGENTS.md` - Důležitá pravidla architektury a instrukce pro vývoj (přečíst jako úplně první před úpravami kódu!).

## 🎯 Jak začít (Co otevřít jako první)

Pokud se chcete rychle zorientovat v projektu, doporučujeme tento postup:

1. **`AGENTS.md`** - Otevřete a přečtěte si jako první. Obsahuje klíčová pravidla pro psaní kódu (Svelte 5, CSS bez Tailwindu, Drizzle pravidla) a konvence projektu.
2. **`src/lib/server/db/schemas/`** - Prohlédněte si databázový model (soubory uvnitř složky), abyste pochopili, s jakými daty aplikace pracuje.
3. **`src/routes/`** - Zde najdete vstupní body aplikace (zobrazení frontendu, administrace a API endpoints).

## 💻 Lokální vývoj

```bash
# 1. Instalace závislostí
pnpm install

# 2. Spuštění lokální PostgreSQL databáze (vyžaduje Docker)
pnpm db:start

# 3. Nahrání Drizzle schématu do databáze (synchronizace)
pnpm db:push

# 4. Spuštění vývojového serveru
pnpm dev
```

K nahlédnutí přímo do databáze v prohlížeči můžete využít příkaz `pnpm db:studio`.

## 🐳 Produkční nasazení (Docker)

Projekt je připraven pro nasazení pomocí Dockeru (`@sveltejs/adapter-node`).

**Spuštění:**
```sh
docker compose -f docker-compose.prod.yml up -d --build
```

**Zastavení:**
```sh
docker compose -f docker-compose.prod.yml down
```

**Logy:**
```sh
docker compose -f docker-compose.prod.yml logs -f
```

**Volitelně Drizzle Studio (profil tools):**
```sh
docker compose -f docker-compose.prod.yml --profile tools up -d --build
```

### Spouštění příkazů uvnitř běžícího kontejneru

Příklady pro spouštění Drizzle příkazů v produkčním stacku:

```sh
# drizzle push (rychlá migrace do DB)
docker compose -f docker-compose.prod.yml exec app pnpm db:push

# generování standardních migrací
docker compose -f docker-compose.prod.yml exec app pnpm db:generate

# aplikace standardních migrací
docker compose -f docker-compose.prod.yml exec app pnpm db:migrate
```
