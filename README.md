# NEXUS OSINT · Command Center v3.1

> Dashboard OSINT autohospedado: 53 fuentes, globo 3D, correlación entre dominios, MITRE ATT&CK y bot de Telegram. Todo en un único `nexus.py` ejecutable con `uvicorn`.

![Python](https://img.shields.io/badge/Python-3.11+-3776AB?logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-0.115+-009688?logo=fastapi&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-blue)
![Deck.gl](https://img.shields.io/badge/deck.gl-9.0-FF6B6B)

---
## ⚠️ Descargo de responsabilidad

- Todos los indicadores (IPs, dominios, hashes) provienen de **fuentes públicas**: CERTs oficiales (GovCERT.ch, CISA), prensa, canales públicos de Telegram, APIs públicas (abuse.ch, Cloudflare Radar).
- **Sin verificación independiente.** La atribución a grupos es lo que dicen las fuentes públicas, no una conclusión forense.
- **Uso permitido:** investigación defensiva, threat intelligence, educación, SOC.
- **Uso prohibido:** ataques activos, escaneos no autorizados, acciones legales basadas en estos datos, acoso a personas.
- Los nombres de grupos son **públicos y documentados** por CERTs y prensa (Killnet, NoName057(16), Lazarus, APT28…). No son acusaciones privadas.
- Si eres titular de una IP listada y crees que es un error, abre un issue y se revisa.

## ✨ Qué hace

- **53 fuentes OSINT** integradas: ciberataques (DDoS, malware/botnets, fuerza bruta, exploits, ransomware, phishing), MITRE ATT&CK, cibercrimen, crimen mundial/urbano, conflictos, naturaleza, economía.
- **Globo 3D y plano** con `deck.gl` en la propia página (sin iframe) + red neuronal animada de fondo.
- **Modos temáticos** (ciber / crimen / conflicto / naturaleza / economía), ranking de riesgo por país y buscador de IOC.
- **SQLite** con histórico, playback temporal y clustering espacial.
- **Correlación** con scoring ponderado, decaimiento temporal y entidades compartidas.
- **Detección de anomalías** (z-score, entidades nuevas, ráfagas, sentimiento negativo).
- **Enriquecimiento**: dedupe semántico (SimHash), escalado de severidad, watchlist persistente, tendencias por dominio, búsqueda facetada.
- **Auth JWT + HTTP Basic**, rate limiting y circuit breaker por fuente.
- **WebSocket** para actualizaciones en vivo.
- **Export** CSV / JSON / GeoJSON / KML / Markdown.
- **Bot de Telegram** con 12 comandos e inline keyboards.
- **LLM** con múltiples prompts y caché (opcional).
- **Scheduler** con health monitoring por fuente y retención automática configurable.

---

## 🚀 Arranque rápido

### 1. Requisitos

- Python **3.11+**
- (Opcional) `GeoLite2-City.mmdb` de [MaxMind](https://dev.maxmind.com/geoip/geolite2-free-geolocation-data) para geolocalización local. Sin él se usa `ip-api.com/batch`.

### 2. Instalar y configurar

```bash
git clone https://github.com/Condor2026/nexus_osint
cd nexus_osint

python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt

cp .env.example .env
# Edita .env: como mínimo cambia ADMIN_PASSWORD y genera JWT_SECRET
```

Genera un `JWT_SECRET` fuerte:

```bash
python -c "import secrets; print(secrets.token_hex(32))"
```

### 3. Ejecutar

```bash
python nexus.py
# → http://127.0.0.1:8000
```

El primer barrido arranca solo en segundo plano. La BD (`nexus.db`) se crea en el primer arranque y se migra automáticamente si ya existía de una versión anterior.

---

## 🔑 Fuentes y claves

Todas las claves son **opcionales**: sin ellas, la fuente simplemente devuelve 0 eventos y el resto del sistema sigue funcionando.

| Fuente | Clave necesaria | Dónde obtenerla |
|---|---|---|
| URLhaus, ThreatFox, MalwareBazaar | `ABUSECH_AUTH_KEY` | https://auth.abuse.ch/ (gratis) |
| Cloudflare Radar (DDoS) | `CLOUDFLARE_API_TOKEN` | https://dash.cloudflare.com/profile/api-tokens (permiso Radar) |
| NASA FIRMS | `NASA_FIRMS_MAP_KEY` | https://firms.modaps.eosdis.nasa.gov/api/ |
| OpenSky | `OPENSKY_USER` / `OPENSKY_PASS` | https://opensky-network.org/ |
| FRED | `FRED_API_KEY` | https://fredaccount.stlouisfed.org/apikeys |
| EIA | `EIA_API_KEY` | https://www.eia.gov/opendata/ |
| ACLED | `ACLED_EMAIL` / `ACLED_PASSWORD` | https://developer.acleddata.com/ |
| AISStream | `AISSTREAM_API_KEY` | https://aisstream.io/ |
| ADS-B Exchange | `ADSB_API_KEY` | https://www.adsbexchange.com/ |
| ReliefWeb | `RELIEFWEB_APPNAME` | https://apidoc.reliefweb.int/ |
| OpenSanctions | `OPENSANCTIONS_API_KEY` | https://www.opensanctions.org/api/ |
| Ransomware.live | `RANSOMWARE_LIVE_KEY` | https://www.ransomware.live/ |
| Telegram | `TELEGRAM_BOT_TOKEN` / `TELEGRAM_CHAT_ID` | @BotFather · @userinfobot |
| LLM | `OPENAI_API_KEY` | https://platform.openai.com/api-keys |

Las fuentes sin clave (Feodo, CISA KEV, SANS ISC, Blocklist.de, ET Compromised, Spamhaus DROP, SSLBL, OpenPhish, RSS de seguridad, GDELT, USGS, EONET, NOAA, WHO, OFAC, portales de crimen urbano, etc.) funcionan **desde el primer minuto**.

---

## 🗺️ Uso del dashboard

| Atajo | Acción |
|---|---|
| `/` | Enfocar el buscador |
| `r` | Refrescar datos |
| `j` | Abrir/cerrar el panel JARVIS |
| `Esc` | Cerrar JARVIS |

- **Clic en un evento del feed** → centra el mapa en su ubicación.
- **Clic en una leyenda de dominio** → filtra por ese dominio.
- **Clic en un país (pestaña Países)** → filtra y centra.
- **Pestaña IOC** → pega IP/dominio/hash/CVE (acepta formato *defangeado* `hxxp://ejemplo[.]com`) y obtiene coincidencias en la BD + enlaces a VirusTotal, AbuseIPDB, Shodan, urlscan, NVD…

---

## 🧠 Panel JARVIS

Overlay de análisis global (`j` o botón **JARVIS**) con:

- **Índice de amenaza** 0-100 con comparación vs. ventana previa.
- **KPIs**: C2/botnets, malware, DDoS, fuerza bruta, exploits/0-day, ransomware, phishing, vulnerabilidades KEV, IPs únicas, crimen, conflicto, críticos.
- **Tácticas MITRE ATT&CK** en radar polar.
- **Línea temporal** por familia de dominios (24 cubos).
- **Top países** por riesgo ponderado.
- **Fuentes más activas** y **alertas recientes**.
- **Técnicas ATT&CK** con clic para filtrar el mapa.

---

## 🔌 API

| Método | Ruta | Descripción |
|---|---|---|
| `GET` | `/api/kpis` | Contadores de la ventana actual |
| `GET` | `/api/map` | GeoJSON compacto para deck.gl |
| `GET` | `/api/events` | Eventos filtrables |
| `GET` | `/api/correlations` | Correlaciones por score |
| `GET` | `/api/anomalies` | Anomalías |
| `GET` | `/api/stats` | Estadísticas globales |
| `GET` | `/api/search` | Búsqueda facetada |
| `GET` | `/api/entities` | Top entidades |
| `GET` | `/api/trends` | Tendencias por dominio |
| `GET` | `/api/countries` | Ranking de riesgo por país |
| `GET` | `/api/ioc` | Clasificar y buscar IOC |
| `GET` | `/api/attack` | Técnicas ATT&CK más vistas |
| `GET` | `/api/analytics` | Datos completos del panel JARVIS |
| `GET` | `/api/health` | Estado de fuentes y circuit breakers |
| `GET` | `/api/sweep/status` | Estado del barrido |
| `POST` | `/api/sweep/start` | Lanzar barrido en segundo plano |
| `POST` | `/api/circuit/reset` | Reiniciar circuit breakers |
| `GET` | `/api/watchlist` | Ver watchlist |
| `POST` | `/api/watchlist/{add,remove}` | Modificar watchlist |
| `POST` | `/api/token` | JWT (form: `username`, `password`) |
| `GET` | `/export/{csv,json,geojson,kml,markdown}` | Exportaciones |
| `WS` | `/ws` | Actualizaciones en vivo |

Auth: `Authorization: Bearer <jwt>` o HTTP Basic. Desde `127.0.0.1` sin proxy el dashboard no pide login (`NEXUS_LOCAL_NOAUTH=1`).

---

## 🤖 Bot de Telegram

Comandos: `/start`, `/brief`, `/sweep`, `/stats`, `/top`, `/health`, `/anomalies`, `/export`, `/search <q>`, `/domains`, `/trends`, `/watch`.

Requiere `TELEGRAM_BOT_TOKEN` y `TELEGRAM_CHAT_ID`. Las alertas se disparan por C2 nuevos, DDoS/malware ALTA/CRÍTICA, anomalías y top de correlaciones.

---

## 🧱 Arquitectura

```text
nexus.py
├── §0   Configuración + caché + circuit breaker
├── §1   Auth (JWT + Basic + bypass loopback)
├── §2   Caché TTL + breaker
├── §3   SQLite (schema, migración, queries, retención)
├── §4   Geolocalización multi-proveedor (GeoLite2 → ip-api batch)
├── §5   Texto: sentimiento, países, entidades, tags
├── §5b  Enriquecimiento: dedupe SimHash, severidad, watchlist, tendencias
├── §6   Colectores (53 fuentes, cada una degrada sola)
├── §7   Correlación (clustering espacial + decaimiento + entidades)
├── §8   Anomalías (z-score + avanzadas)
├── §9   LLM (3 prompts + caché)
├── §10  Telegram + alertas
├── §11  Exportación
├── §12  Orquestación de barrido
├── §13  Payload del mapa (JSON compacto)
├── §14  Frontend (HTML/CSS/JS + NeuralNet + deck.gl + JARVIS)
└── §15  FastAPI (rutas, parciales, API, WebSocket)
```

**Decisiones de diseño**

- **El mapa ya no se genera con pydeck en iframe.** El servidor entrega JSON compacto (`/api/map`) y el navegador mantiene *una sola* instancia de deck.gl; cambiar filtros no recrea WebGL.
- **Fuentes que degradan solas.** Cada colector puede lanzar excepciones: `run_source()` las captura, registra y alimenta el circuit breaker. Sin clave → `[]`.
- **Migración automática de la BD.** Si la tabla `events` no tiene `entities`/`tags`, se añaden con `ALTER TABLE` al arrancar.
- **Ubicación honesta.** Si un texto no menciona país, el evento se descarta (no se inventan coordenadas). Los centroides se *jitterizan* de forma determinista para que 40 noticias del mismo país no se apilen.

---

## 📁 Estructura del repo

```text
.
├── nexus.py                 # Aplicación completa
├── requirements.txt
├── .env.example
├── .gitignore
├── LICENSE
├── README.md
├── CONTRIBUTING.md
├── SECURITY.md
└── (opcional) GeoLite2-City.mmdb   # NO se sube: añádelo a mano
```

---

## ⚠️ Aviso legal y uso responsable

NEXUS agrega **fuentes públicas** (feeds de seguridad, agencias gubernamentales, prensa, APIs abiertas). No realiza escaneos activos, no visita URLs de phishing (solo resuelve DNS), no interactúa con infraestructura maliciosa.

Úsalo solo para **investigación defensiva, monitorización de amenazas o docencia**, cumpliendo las leyes de tu jurisdicción y los términos de cada API. El autor no se responsabiliza del uso indebido.

Los indicadores (IPs, hashes, URLs) se muestran *defangeados* (`hxxp://ejemplo[.]com`) para evitar clics accidentales.

---

## 🙏 Créditos

- [abuse.ch](https://abuse.ch/) (Feodo, URLhaus, ThreatFox, SSLBL, MalwareBazaar)
- [CISA KEV](https://www.cisa.gov/known-exploited-vulnerabilities-catalog), [SANS ISC](https://isc.sans.edu/), [Spamhaus](https://www.spamhaus.org/)
- [GDELT](https://www.gdeltproject.org/), [ACLED](https://acleddata.com/), [ReliefWeb](https://reliefweb.int/)
- [USGS](https://earthquake.usgs.gov/), [NASA EONET/FIRMS](https://eonet.gsfc.nasa.gov/), [NOAA](https://www.weather.gov/), [WHO](https://www.who.int/)
- [MITRE ATT&CK](https://attack.mitre.org/)
- [deck.gl](https://deck.gl/), [FastAPI](https://fastapi.tiangolo.com/), [Natural Earth](https://www.naturalearthdata.com/)

---

## 📄 Licencia

MIT — ver [LICENSE](LICENSE).
