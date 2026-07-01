# Installation av Lumos

Den här guiden tar dig från en tom webbhotell-katalog till en körande, härdad
Lumos-instans. Skriven för **cPanel + LiteSpeed/Apache** men fungerar på vilken
PHP-8.1-värd som helst med `.htaccess`-stöd och cron.

- [1. Förkrav](#1-förkrav)
- [2. Katalogstruktur](#2-katalogstruktur)
- [3. Ladda upp filerna](#3-ladda-upp-filerna)
- [4. Kör installationsguiden](#4-kör-installationsguiden)
- [5. Fyll i config.php](#5-fyll-i-configphp)
- [6. cPanel-serverinställningar](#6-cpanel-serverinställningar)
- [7. Behörigheter](#7-behörigheter)
- [8. Cron-schema](#8-cron-schema)
- [9. Första körning & verifiering](#9-första-körning--verifiering)
- [10. Härdning (rekommenderas)](#10-härdning-rekommenderas)
- [11. Felsökning](#11-felsökning)

---

## 1. Förkrav

- **PHP 8.1 eller senare** med tilläggen:
  - Krävs: `curl`, `mbstring`, `simplexml`, `json`, `gd`, `phar`
  - Rekommenderas: `apcu`, `openssl`, `zip`
- Webbserver med `.htaccess`: **Apache eller LiteSpeed** (cPanel default)
- **Cron** (cPanel → Cron Jobs, eller systemcrontab)
- HTTPS (cPanel AutoSSL / Let's Encrypt)
- Shell-/SSH-åtkomst är en fördel men inte ett krav (guiden har webbläge)

Kontrollera PHP-version och tillägg i cPanel → **Select PHP Version** (sätt 8.1+
och bocka i tilläggen ovan). `setup.php` verifierar också detta åt dig.

---

## 2. Katalogstruktur

Lumos förväntar sig att **config och datakataloger ligger utanför webroten**:

```
/home/<konto>/                 ← LUMOS_HOME (kontohem)
├── config.php                 ← nycklar (UTANFÖR webroten — laddas aldrig ner)
├── cache/                     ← LUMOS_CACHE (JSON-cacher, skapas av setup)
├── logs/                      ← LUMOS_LOGS  (loggar, skapas av setup)
├── backups/                   ← dagliga tar.gz (skapas av setup)
└── <din-doman.se>/            ← LUMOS_WEBROOT (det webb-serverade trädet)
    └── aggregator/            ← Lumos-koden (denna mapp)
        ├── lib/paths.php       ← central bootstrap (härleder sökvägar)
        ├── admin/  cron/  pages/  proxies/  api/  font/
        ├── config-example.php
        └── setup.php           ← RADERA efter installation
```

Sökvägarna **härleds automatiskt** från denna struktur. Avviker din uppsättning
kan du överskrida `LUMOS_HOME` / `LUMOS_WEBROOT` i config.php.

---

## 3. Ladda upp filerna

Ladda upp `aggregator/`-mappen till din webroot (t.ex. via cPanel File Manager
eller SFTP). Ladda **inte** upp:

- `setup.php` till en redan installerad/publik server (kör den lokalt/engångs och
  radera den — se nästa steg)
- eventuell lokal `config.php` (skapas på servern)
- `presentation/`-mappen (separat verktyg, inte del av webbappen)

> Du styr själv vilka mappar som ska till GitHub/servern — det finns ingen
> `.gitignore` som tvingar något.

---

## 4. Kör installationsguiden

`setup.php` skapar `config.php` (med slumpade nycklar), datakatalogerna och
`.htaccess`. Den **skriver aldrig över** en befintlig `config.php`.

**CLI (rekommenderas):**
```bash
cd /home/<konto>/<din-doman.se>/aggregator
php setup.php            # torrkörning: visar vad som skulle hända
php setup.php --apply    # utför ändringarna
```

**Webbläge:** av säkerhetsskäl är webbinstallationen **låst mot fjärråtkomst**.
För att låsa upp den tillfälligt, skapa en tom fil `setup.allow` bredvid
`setup.php` (via cPanel File Manager), öppna sedan
`https://din-doman.se/aggregator/setup.php` och följ formuläret. (Lokala/SSH-anrop
och CLI behöver ingen markör.) Setup vägrar dessutom köra om `config.php` redan
finns och läcker då inga serversökvägar.

När guiden är klar:

```bash
rm setup.php setup.allow   # ⚠️ OBLIGATORISKT — lämna dem aldrig kvar publikt
```

---

## 5. Fyll i config.php

`config.php` ligger i `<kontohem>/config.php`. Öppna den och fyll i:

- **`LUMOS_SITE_URL`** — krävs, t.ex. `https://din-doman.se` (utan avslutande `/`)
- **`STATS_SECRET`, `REFRESH_SECRET`** — genererades av setup; lämna som de är
- API-nycklar **efter behov** (alla valfria, se [CREDITS.md](CREDITS.md)):
  - `CLAUDE_API_KEY` (AI — kostar pengar; lämna tom för att stänga av)
  - `CLAUDE_MONTHLY_TOKEN_LIMIT` (budgettak, 0 = inget tak)
  - `DEEPL_API_KEY`, `OWM_API_KEY`
  - `AISSTREAM_API_KEY` / `BARENTSWATCH_*` (fartyg)
  - `SENTINEL_CLIENT_ID/SECRET` (satellit)
  - `DISCORD_WEBHOOK` (notiser)

Mall med kommentarer: [`config-example.php`](config-example.php).

> Redigera config.php **manuellt på servern**. Den ska aldrig checkas in eller
> ligga i webroten.

---

## 6. cPanel-serverinställningar

Sätt dessa i cPanel (eller via `php.ini` / `.user.ini` om du har åtkomst):

**PHP (Select PHP Version → Options / MultiPHP INI Editor):**

| Direktiv | Värde | Varför |
|---|---|---|
| `display_errors` | `Off` | läck aldrig fel/sökvägar publikt |
| `expose_php` | `Off` | dölj PHP-version i HTTP-headers |
| `log_errors` | `On` | fel hamnar i loggfil istället |
| `error_reporting` | `E_ALL & ~E_DEPRECATED & ~E_NOTICE` | rimlig loggnivå |
| `allow_url_fopen` | `On` | krävs för stream-baserade API-anrop |
| `max_execution_time` | `60` | räcker för proxy-/AI-anrop |
| `open_basedir` | `/home/<konto>` | begränsa filåtkomst till ditt konto |

**HTTPS:**
- Aktivera **AutoSSL** (cPanel → SSL/TLS Status) så certifikatet förnyas automatiskt.
- Tvinga HTTPS + HSTS — `.htaccess` som setup genererar gör redan detta. Verifiera
  att `Strict-Transport-Security` skickas.

**Directory Privacy (htpasswd) på admin:**
- cPanel → **Directory Privacy** → välj `.../aggregator/admin/` → sätt
  användarnamn + lösenord. Detta är **perimeterskyddet** för admin.
- `setup.php` lägger htpasswd-sökvägen i `admin/.htaccess`. Bekräfta att
  `AuthUserFile` pekar på rätt `.htpasswd` (cPanel skapar den oftast i
  `<kontohem>/.htpasswds/...`).

---

## 7. Behörigheter

Sätt restriktiva behörigheter på allt utanför webroten (setup gör detta, men
verifiera):

```bash
chmod 700 /home/<konto>/cache /home/<konto>/logs /home/<konto>/backups
chmod 600 /home/<konto>/config.php
# kod: 644 för filer, 755 för kataloger (standard räcker)
```

`config.php` **600** gör att bara ditt konto kan läsa filen.

---

## 8. Cron-schema

Lägg in jobben i cPanel → **Cron Jobs**. Ersätt `PHP` och `BASE` nedan:

```
PHP=/usr/local/bin/php          # din PHP 8.1-binär (cPanel: kan vara /opt/cpanel/ea-php81/root/usr/bin/php)
BASE=/home/<konto>/<din-doman.se>/aggregator/cron
```

Samma schema finns färdigt att kopiera i [`crontab.example`](crontab.example).
Komplett schema (matchar `lumos_architecture.md`; AI-jobb är spridda så max ett
körs per minut):

```cron
# ── Datainsamling (icke-AI) ───────────────────────────────────────────────
*/15 * * * *      $PHP $BASE/rebuild_cache_cron.php rss
*/15 * * * *      $PHP $BASE/rebuild_cache_cron.php telegram
*/15 * * * *      $PHP $BASE/discord_push_cron.php
5,20,35,50 * * * * $PHP $BASE/velocity_snapshot_cron.php
5,20,35,50 * * * * $PHP $BASE/watchlist_cron.php
8,23,38,53 * * * * $PHP $BASE/dedup_cron.php
0,30 * * * *      $PHP $BASE/silence_detector_cron.php
0,30 * * * *      $PHP $BASE/article_geo_cron.php
0,30 * * * *      $PHP $BASE/firms_fires_cron.php
0 * * * *         $PHP $BASE/source_primacy_cron.php
0,45 * * * *      $PHP $BASE/source_confidence.php

# ── AI-jobb (Haiku) — utspridda ───────────────────────────────────────────
7,22,37,52 * * * * $PHP $BASE/image_analysis_cron.php
9,39 * * * *      $PHP $BASE/image_geo_cron.php
10,40 * * * *     $PHP $BASE/contradiction_cron.php
12,42 * * * *     $PHP $BASE/actor_track_cron.php
15,45 * * * *     $PHP $BASE/entity_cron.php
18,48 * * * *     $PHP $BASE/topic_cluster_cron.php
20,50 * * * *     $PHP $BASE/coordination_cron.php
21,51 * * * *     $PHP $BASE/translate_rss_cron.php
25,55 * * * *     $PHP $BASE/io_detect_cron.php
4,19,34,49 * * * * $PHP $BASE/translate_tg_cron.php

# ── AI-jobb (Sonnet) — schemalagda ────────────────────────────────────────
5 6,12,18 * * *   $PHP $BASE/summary_cron.php
1 0 * * *         $PHP $BASE/summary_cron.php
0 7,13,19 * * *   $PHP $BASE/telegram_summary_cron.php
25 14 * * 0       $PHP $BASE/weekly_summary_cron.php
30 6 * * *        $PHP $BASE/forecast_calibration_cron.php
0 4 * * *         $PHP $BASE/map_location_cron.php
0 8 * * 1         $PHP $BASE/source_suggestions_cron.php

# ── Underhåll ─────────────────────────────────────────────────────────────
55 23 * * *       $PHP $BASE/source_confidence_snapshot.php
0 2 * * *         $PHP $BASE/backup_cron.php
0 3 * * *         $PHP $BASE/log_rotate.php
0 2 1 * *         $PHP $BASE/trim_translation_cache.php
0 4 * * 0         $PHP $BASE/geoblock_update_cron.php
0 5 * * *         $PHP $BASE/sentinel_cache_cron.php
```

> Kör du **utan AI** (tom `CLAUDE_API_KEY`)? Hoppa över AI-jobben — eller låt dem
> ligga; de avslutar tidigt om nyckeln saknas. Använder du inte Sentinel-lagret
> kan `sentinel_cache_cron.php` utelämnas.

---

## 9. Första körning & verifiering

1. Besök `https://din-doman.se/aggregator/` — flödet ska ladda (tomt tills första
   `rebuild_cache_cron` kört).
2. Kör datainsamlingen en gång manuellt för att fylla cachen:
   ```bash
   php $BASE/rebuild_cache_cron.php rss
   php $BASE/rebuild_cache_cron.php telegram
   ```
3. Logga in i admin (`/aggregator/admin/`): först htpasswd-dialogen, sedan
   `STATS_SECRET`.
4. I admin → **System**: kontrollera cache-åldrar och (om AI används)
   token-/budgetmätaren.
5. Öppna kartan och bekräfta att de lager du konfigurerat dyker upp.

---

## 10. Härdning (rekommenderas)

- **IP-lockdown:** sätt `LUMOS_ALLOWED_IPS` i config.php om instansen är privat
  eller kör betalda integrationer (Claude/Sentinel). Skyddar mot extern
  kostnadsabuse. Bakom Cloudflare: sätt `LUMOS_TRUSTED_PROXIES` **först**.
- **Geo-blockering:** `LUMOS_GEO_BLOCK_ENABLED = true` + lista i `LUMOS_GEO_BLOCKED`.
  Kör `geoblock_update_cron.php` en gång för att bygga IP-tabellen. Lägg ditt eget
  IP i `LUMOS_GEO_ALLOWED_IPS` så du inte låser ut dig.
- **fail2ban** (om du har root/VPS): skapa en jail som matchar upprepade 403/401
  mot `admin/` i webbserverloggen och bannar IP:t. På delad cPanel utan root är
  IP-lockdown + htpasswd det viktiga skyddet.
- **Backup-kryptering:** sätt `BACKUP_KEY` i config.php så `backup_cron.php`
  krypterar de dagliga arkiven (AES-256, OpenSSL-format). Förvara nyckeln separat —
  utan den går backuperna inte att återställa. Återställning:
  `openssl enc -d -aes-256-cbc -pbkdf2 -md sha256 -iter 10000 -in fil.tar.gz.enc -out fil.tar.gz -pass pass:DIN_NYCKEL`.
- **AI-budget:** sätt `CLAUDE_MONTHLY_TOKEN_LIMIT` även om du litar på lockdownen —
  bälte och hängslen mot skenande kostnad.

---

## 11. Felsökning

| Symptom | Trolig orsak | Åtgärd |
|---|---|---|
| Vit sida / "headers already sent" | blank rad/utskrift före `<?php` i config | kontrollera att config.php börjar exakt med `<?php` |
| 500 på alla sidor | config.php hittas inte / saknar `LUMOS_SITE_URL` | verifiera placering (`<kontohem>/config.php`) och innehåll |
| 403 överallt | IP-lockdown utestänger dig | lägg ditt IP i `LUMOS_ALLOWED_IPS`; bakom CF: sätt `LUMOS_TRUSTED_PROXIES` |
| Admin ber inte om lösenord | htpasswd ej satt på `admin/` | sätt Directory Privacy i cPanel |
| Tomt flöde | cron har inte kört | kör `rebuild_cache_cron.php` manuellt; kontrollera cron-loggen |
| AI-svar: "budget förbrukad" | `CLAUDE_MONTHLY_TOKEN_LIMIT` nådd | höj taket eller vänta till nästa månad |
| Kartlager saknas | nyckel saknas/ogiltig | fyll i rätt nyckel i config; se [CREDITS.md](CREDITS.md) |
| Karta laddar inte Leaflet/mgrs | CSP blockerar CDN | kontrollera `script-src` i `lib/lumos_csp.php` |

Loggar hittas i `<kontohem>/logs/`. PHP-fel hamnar i webbserverns error-log
(`display_errors=Off`, `log_errors=On`).

---

Mer detaljer: [README.md](README.md) · [lumos_architecture.md](lumos_architecture.md) ·
[incident_response.md](incident_response.md) · [CREDITS.md](CREDITS.md)
