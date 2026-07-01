# Lumos

**OSINT-nyhetsaggregator med karta och valfri AI-analys.** PHP 8.1+, ingen
databas — allt drivs av cron och fil-baserade JSON-cacher utanför webroten.

Lumos samlar öppna källor (RSS + Telegram), kör en mestadels **deterministisk**
analyspipeline och presenterar resultatet som flöde, dashboard och en interaktiv
underrättelsekarta. AI (Anthropic Claude) används **selektivt och valfritt** för
sammanfattningar, SITREP, entitets-/ämnesutvinning, IO- och biasanalys.

> **Licens:** MIT för Lumos-koden. Tredjepartsdata/API:er har egna villkor —
> läs [CREDITS.md](CREDITS.md) **innan** du driftar publikt.

---

## Funktioner

**Aggregering & analys**
- RSS- och Telegram-skrapning (cron, var 15:e min)
- Dygnssammanfattningar, veckosammanfattning och SITREP (PDF-export)
- Entitetsutvinning, ämnesgruppering, aktörsspårning, tystnadsdetektion
- IO- (informationsoperationer) och biasanalys, motsägelsedetektion
- Källprimär-/trovärdighetspoäng
- Översättning (DeepL + Claude Haiku)

**Interaktiv karta** (Leaflet)
- Flyglager (OpenSky: position, callsign, ursprungsland, kategori)
- Fartygslager / AIS (AISStream globalt, fallback Digitraffic + BarentsWatch),
  färgkodat efter status
- Väderlager + prognos (OpenWeatherMap)
- Brandpunkter (NASA FIRMS) med zoom-medveten footprint
- Satellitbild: Sentinel-1 (SAR) & Sentinel-2 (true color) via Copernicus
- "Earth at Night" (NASA GIBS VIIRS) som bakgrund
- Ljus (OSM) och mörk (CARTO) baskarta
- Klick → MGRS + lat/long, platssökning som även tar koordinater

**Drift & säkerhet**
- Tvålagers admin-auth: htpasswd (perimeter) + `STATS_SECRET` (applager)
- IP-lockdown (`LUMOS_ALLOWED_IPS`) som kostnadsskydd
- Geo-blockering (binär IP-tabell, byggs veckovis av cron)
- Per-fil CSP med nonce, CSRF-skydd, rate-limit, brute-force-skydd
- Discord-notiser, daglig backup, log-rotation
- Token-/kostnadsmätning + **månadsbudgettak** för AI

---

## Krav

- **PHP 8.1+** med tilläggen: `curl`, `mbstring`, `simplexml`, `json`, `gd`, `phar`
  (valfria men rekommenderade: `apcu`, `openssl`, `zip`)
- Webbserver: **Apache eller LiteSpeed** med `.htaccess`-stöd (testat på cPanel/LiteSpeed)
- **Cron** (cPanel cron eller systemcron) för analyspipelinen
- HTTPS (Let's Encrypt / cPanel AutoSSL)
- Inga databaser krävs.

API-nycklar är **valfria** — Lumos fungerar utan dem (då döljs respektive
funktion). Se [CREDITS.md](CREDITS.md) för var nycklar hämtas.

---

## Snabbstart

```bash
# 1. Ladda upp Lumos till webroten (t.ex. <kontohem>/din-doman.se/aggregator)
# 2. Kör installationsguiden (skapar config.php med slumpade nycklar,
#    cache-/logg-kataloger och .htaccess):
php setup.php --apply          # CLI
#    …eller öppna setup.php i webbläsaren en gång.

# 3. RADERA setup.php från servern efteråt.
# 4. Fyll i API-nycklar i config.php (en nivå ovanför webroten).
# 5. Lägg in cron-schemat (se INSTALL.md).
```

Fullständig anvisning med cPanel-inställningar, behörigheter och cron-schema:
**[INSTALL.md](INSTALL.md)**.

---

## Konfiguration

All konfiguration ligger i **`config.php`**, som ska placeras **en nivå ovanför
webroten** (`<kontohem>/config.php`) så den aldrig kan laddas ner. Kopiera
[`config-example.php`](config-example.php) och fyll i värden — eller låt
`setup.php` generera den.

Lumos härleder själv sökvägar (`LUMOS_HOME`, `LUMOS_WEBROOT`, cache/logs) från
katalogstrukturen; du behöver normalt bara sätta `LUMOS_SITE_URL` och dina
nycklar. Allt centraliseras i [`lib/paths.php`](lib/paths.php).

---

## Säkerhetsmodell (kort)

| Lager | Mekanism |
|---|---|
| Perimeter | `.htaccess` htpasswd på `admin/`, IP-lockdown, geo-block |
| Applager | `STATS_SECRET` (admin-login), CSRF-token, rate-limit, brute-force |
| Transport | HTTPS + HSTS, per-fil CSP med nonce |
| Kostnad | IP-lockdown + `CLAUDE_MONTHLY_TOKEN_LIMIT` + AI-strömbrytare |

Kör betalda/mätta integrationer (Claude, Sentinel Hub)? **Håll IP-lockdownen på.**
Kontaktformulär och RSS omfattas av lockdownen — är sidan låst går de inte att nå.

---

## AI & kostnad

- AI är **avstängt som standard** (tom `CLAUDE_API_KEY`).
- Sätt `CLAUDE_MONTHLY_TOKEN_LIMIT` för att hårdstoppa alla AI-anrop (cron +
  on-demand) när månadens tokenförbrukning nås.
- Förbrukning loggas och visas i admin (System-fliken).
- Schemat är optimerat så att max **ett** AI-jobb körs per minutslot.

---

## Licens & erkännanden

- Lumos-kod: **MIT** — se [LICENSE](LICENSE).
- Tredjepartsdata, API:er och bibliotek: **se [CREDITS.md](CREDITS.md)** för
  licenser och obligatorisk attribution. Ta inte bort attributionen som visas på
  kartan och i hjälpavsnitten.
