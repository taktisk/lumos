# Credits & tredjepartslicenser

Lumos egen kod är MIT-licensierad (se [LICENSE](LICENSE)). Lumos använder dessutom
ett antal externa **datakällor, API:er och bibliotek** som har sina **egna**
licenser och attributionskrav. MIT-licensen för Lumos-koden ger dig **inga**
rättigheter till dessa tjänster eller data — du måste själv följa varje
leverantörs villkor när du driftar Lumos publikt.

> **Sammanfattning av dina skyldigheter:** behåll attributionerna som visas i
> kartans hörn och i hjälpavsnitten, respektera varje API:s användarvillkor och
> ratelimits, och registrera egna API-nycklar. Flera flöden (t.ex. OpenSky) är
> begränsade till **icke-kommersiell** användning — kontrollera innan du
> kommersialiserar en Lumos-instans.

---

## Datakällor & API:er

### Anthropic Claude (AI)
- **Används till:** sammanfattningar, SITREP, entitets-/ämnesutvinning, IO- och
  biasanalys, översättning (Haiku), vision, källförslag.
- **Villkor:** kommersiellt API. Lyder under Anthropic Commercial Terms of
  Service. Egen API-nyckel krävs. **Betaltjänst — kostar pengar per token.**
- **Valfritt:** Lumos fungerar utan AI (tom `CLAUDE_API_KEY`). Se budgettak
  `CLAUDE_MONTHLY_TOKEN_LIMIT` och AI-strömbrytaren i admin.
- https://www.anthropic.com/legal/commercial-terms

### DeepL (översättning)
- **Används till:** översättning av rubriker/texter.
- **Villkor:** DeepL API Terms. Gratis- eller betalkonto. Egen nyckel krävs.
- https://www.deepl.com/pro-license

### OpenWeatherMap (väder)
- **Används till:** väderlager och prognos på kartan.
- **Licens:** väderdata under CC BY-SA 4.0. **Attribution krävs:** "Weather data
  provided by OpenWeather".
- https://openweathermap.org/ · https://creativecommons.org/licenses/by-sa/4.0/

### OpenSky Network (flyg / ADS-B)
- **Används till:** flyglagret (positioner, callsign, ursprungsland, kategori).
- **Licens:** data tillgängliggörs för **icke-kommersiell forskning/bruk**.
  Attribution krävs: "Flight data: The OpenSky Network, https://opensky-network.org".
- https://opensky-network.org/about/terms-of-use

### AISStream.io (fartyg / AIS — global)
- **Används till:** globalt fartygslager via WebSocket.
- **Villkor:** gratis med registrering. Lyder under AISStream Terms. Egen nyckel
  krävs. **OBS:** nyckeln exponeras i klienten — kör endast IP-låst.
- https://aisstream.io/

### Digitraffic / Fintraffic (fartyg / AIS — Östersjön)
- **Används till:** AIS-fallback (Östersjön) utan nyckel.
- **Licens:** CC BY 4.0. **Attribution krävs:** "Traffic data: Fintraffic /
  digitraffic.fi, license CC BY 4.0".
- https://www.digitraffic.fi/en/terms-of-service/

### BarentsWatch (fartyg / AIS — norska/arktiska vatten)
- **Används till:** AIS-komplement för norska vatten.
- **Licens:** Norwegian Coastal Administration (Kystverket), öppna data (NLOD-typ).
  Attribution: "AIS data: BarentsWatch / Kystverket". Kräver registrerad
  AIS-klient (client_id/secret).
- https://www.barentswatch.no/en/articles/open-data-via-barentswatch/

### Copernicus / Sentinel Hub (satellitbild)
- **Används till:** Sentinel-1 (SAR) och Sentinel-2 (true color) bildlager.
- **Licens:** Copernicus Sentinel-data är fritt och öppet. **Attribution krävs:**
  "Contains modified Copernicus Sentinel data [år]". Tjänsten Sentinel Hub via
  Copernicus Data Space Ecosystem har egna villkor och mäts i Processing Units.
- https://dataspace.copernicus.eu/ · https://sentinel.esa.int/web/sentinel/terms-conditions

### NASA FIRMS (bränder / termiska anomalier)
- **Används till:** brandpunkter (VIIRS/MODIS) på kartan.
- **Licens:** NASA öppna data. Attribution: "Fire data: NASA FIRMS".
- https://firms.modaps.eosdis.nasa.gov/

### NASA GIBS (Earth at Night / VIIRS DNB)
- **Används till:** "Earth at Night"-bakgrundslager (gap-filled VIIRS DNB).
- **Licens:** NASA öppna data, fri användning. Attribution: "Imagery: NASA GIBS /
  EOSDIS".
- https://nasa-gibs.github.io/gibs-api-docs/

### Nominatim (geokodning, OpenStreetMap)
- **Används till:** platssökning ("sök plats").
- **Licens/policy:** ODbL-data. Lyder under Nominatim Usage Policy (max 1 req/s,
  giltig User-Agent, ingen massanvändning). **Driv egen instans vid hög last.**
  Attribution: "© OpenStreetMap contributors".
- https://operations.osmfoundation.org/policies/nominatim/

### OpenStreetMap-rutor / CARTO basemaps (kartbakgrund)
- **Används till:** ljus (OSM) och mörk (CARTO) baskarta.
- **Licens:** kartdata ODbL "© OpenStreetMap contributors"; CARTO-rutorna kräver
  CARTO-attribution. **Respektera OSM:s tile usage policy** — använd ett eget
  rutlager/CDN i produktion vid hög trafik.
- https://www.openstreetmap.org/copyright · https://carto.com/attribution/

---

## Medföljande bibliotek

| Bibliotek | Används till | Licens | Distribution |
|---|---|---|---|
| **Leaflet** | kartrendering | BSD-2-Clause | CDN (unpkg) |
| **mgrs** (proj4js) | MGRS ↔ lat/long | MIT | CDN (unpkg) |
| **FPDF** | PDF-export (SITREP) | FPDF License (fri, även kommersiellt) | medföljer (`fpdf.php`, `font/`) |
| **pptxgenjs** | presentationsbyggare (ej del av webbappen) | MIT | endast `presentation/`-verktyget |

CDN-resurser laddas från `unpkg.com` och tillåts uttryckligen i Content-Security-Policy.
Vill du undvika tredjepartshämtning i klienten kan du själv-hosta Leaflet och mgrs
och uppdatera `script-src`/`style-src` i `lib/lumos_csp.php`.

---

## Attribution som redan visas i Lumos

Lumos visar källattribution i kartans hörn (Leaflet attribution control) och i
hjälpavsnitten (`help.html`). **Ta inte bort dessa** — de uppfyller flera av
kraven ovan (OSM, CARTO, OpenSky, AIS, NASA, Copernicus). Om du lägger till eller
byter datakälla, uppdatera attributionen därefter.
