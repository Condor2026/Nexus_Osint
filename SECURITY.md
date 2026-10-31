# Política de Seguridad

**NEXUS OSINT X Command Center** · v3.1
Última revisión: 2026-10-30

---

## 1. Versiones soportadas

| Versión | Soporte de seguridad |
|---|---|
| 3.1.x   | ✅ Activo |
| < 3.1   | ❌ Sin soporte |

Solo se aceptan reportes sobre la rama `main` y la última release etiquetada. Las versiones antiguas no reciben parches.

---

## 2. Reportar una vulnerabilidad

**No abras un issue público con detalles explotables.** Usa uno de estos canales:

1. **GitHub Security Advisory** (preferido): pestaña *Security → Report a vulnerability* del repositorio.
   **SpectrumSecIt@proton.me**

Incluye en el reporte:

- Versión de NEXUS afectada.
- Pasos para reproducir (mínimos y deterministas).
- Impacto realista (qué se consigue, con qué privilegios).
- Prueba de concepto si la tienes (sin datos de terceros).
- Parche sugerido si puedes.

**Compromiso de respuesta:** acuse de recibo en ≤ 72 h laborables, evaluación inicial en ≤ 7 días, y parche o mitigación en ≤ 30 días para vulnerabilidades confirmadas de severidad alta/crítica. Se te acreditará en el advisory salvo que pidas anonimato.

**Fuera de alcance** (no se consideran vulnerabilidades de NEXUS):

- Fallos en dependencias de terceros sin impacto demostrable en NEXUS (repórtalos upstream).
- Ingeniería social contra el operador.
- Ataques de denegación de servicio volumétricos contra tu propia instancia.
- Ausencia de cabeceras de seguridad en un despliegue local sin proxy.
- Contenido de fuentes externas (RSS, APIs, Telegram) que NEXUS solo muestra.
- Configuraciones deliberadamente inseguras documentadas como "no usar en producción".

---

## 3. Alcance y naturaleza del proyecto

NEXUS es una **herramienta de visualización OSINT de solo lectura**. Su función es agregar, correlacionar y mostrar información publicada por terceros.

**Lo que NEXUS hace:**

- Lee fuentes públicas (RSS, APIs abiertas, canales públicos de Telegram vía `t.me/s/`).
- Almacena eventos en una base SQLite local.
- Los muestra en un mapa 3D y un panel de análisis.
- Exporta los datos en varios formatos.

**Lo que NEXUS NO hace:**

- ❌ No escanea puertos ni hosts.
- ❌ No explota vulnerabilidades.
- ❌ No contacta con infraestructura de Command & Control (C2).
- ❌ No envía tráfico a los indicadores que muestra.
- ❌ No realiza acciones ofensivas de ningún tipo.
- ❌ No automatiza ataques ni genera payloads.

Los indicadores (IPs, dominios, hashes, URLs) se muestran **defangeados** cuando corresponde (`hxxp://ejemplo[.]com`) para evitar clics accidentales. NEXUS no resuelve ni visita las URLs de malware que lista.

---

## 4. Fuentes de datos

Todos los indicadores provienen de **fuentes públicas y documentadas**:

| Fuente | Tipo | Documentación |
|---|---|---|
| GovCERT.ch (CERT Suiza) | IPs/CIDR de NoName057(16) | https://github.com/govcert-ch/CTI |
| CISA KEV | Vulnerabilidades explotadas | https://www.cisa.gov/known-exploited-vulnerabilities-catalog |
| abuse.ch (Feodo, URLhaus, ThreatFox, SSLBL, MalwareBazaar) | IoCs de malware/C2 | https://abuse.ch |
| Cloudflare Radar | Pares origen→objetivo de DDoS | https://radar.cloudflare.com |
| Spamhaus DROP | Redes secuestradas | https://www.spamhaus.org/drop/ |
| Blocklist.de, ET Compromised, CI Army, GreenSnow, Binary Defense | IPs atacantes | Sitios oficiales |
| Canales públicos de Telegram (`t.me/s/`) | Reivindicaciones de grupos | Lectura pública sin login |
| RSS de prensa y seguridad | Noticias | Feeds públicos |
| GDELT, USGS, NASA EONET, NOAA, WHO, ReliefWeb, ACLED | Eventos multi-dominio | APIs públicas documentadas |
| OFAC, ONU, INTERPOL, OpenSanctions | Sanciones y notificaciones | APIs públicas |

**Atribución:** la asignación de una IP o un evento a un grupo (Killnet, NoName057(16), Lazarus, APT28, etc.) refleja **lo que dice la fuente pública**, no una conclusión forense propia. NEXUS no verifica independientemente esas atribuciones y lo indica en la interfaz (`conf: feed` vs `conf: reportada`).

**Nombres de grupos:** los nombres y alias incluidos en `ACTOR_GROUPS` son públicos y están documentados por CERTs, gobiernos y prensa especializada. No son acusaciones privadas ni señalamientos no contrastados.

---

## 5. Privacidad y datos personales

- NEXUS **no recopila datos de usuarios** de la instancia (sin telemetría, sin analytics, sin cookies de terceros).
- La base de datos local contiene **eventos públicos**, no datos personales de usuarios del panel.
- Los indicadores pueden incluir IPs que, en algunos casos, correspondan a personas físicas (p. ej. IPs dinámicas de ISP). Se muestran **como dato técnico de una fuente pública**, no como identificación de una persona.
- Si eres titular de una IP listada y crees que es un error, abre un issue **sin datos personales** y se revisa la fuente original.

---

## 6. Buenas prácticas al desplegar

NEXUS está pensado para ejecutarse en **localhost** o tras un proxy inverso con autenticación. Antes de exponerlo:

- **Cambia `ADMIN_PASSWORD`** y genera un `JWT_SECRET` aleatorio (`python -c "import secrets; print(secrets.token_hex(32))"`).
- **No expongas el puerto 8000 directamente a Internet.** Usa un proxy inverso con TLS (Caddy, nginx, Traefik) y desactiva `NEXUS_LOCAL_NOAUTH=0`.
- **Revisa que `.env`, `nexus.db`, `*.db-wal`, `*.db-shm` y `.cache/` no estén en el repo** ni en backups públicos (el `.gitignore` los excluye por defecto).
- **Rota las API keys** (`OPENAI_API_KEY`, `ABUSECH_AUTH_KEY`, `CLOUDFLARE_API_TOKEN`, `TELEGRAM_BOT_TOKEN`, etc.) 
- **El bypass de loopback** (`NEXUS_LOCAL_NOAUTH=1`, por defecto) valida `Host`, `Origin`, `Sec-Fetch-Site` y rechaza peticiones con `X-Forwarded-For`, pero un proxy mal configurado puede saltárselo. **En producción ponlo a `0`.**
- **Limita el acceso** al panel por IP o VPN si lo despliegas en una red compartida.
- **Revisa los logs** (`nexus.log`) y la tabla `audit_log` para detectar accesos anómalos.
- **Mantén las dependencias actualizadas** (`pip list --outdated`, `pip-audit`).

---

## 7. Uso aceptable

**Permitido:**

- Investigación defensiva, threat intelligence, SOC, CERTs.
- Educación y formación.
- Periodismo de datos y verificación.
- Análisis académico.

**Prohibido:**

- Ataques activos, escaneos no autorizados, explotación.
- Acciones legales basadas únicamente en los datos mostrados (verifica la fuente original).
- Acoso, señalamiento o doxxing de personas.
- Reventa de los datos sin respetar las licencias de las fuentes originales.
- Uso que vulnere los Términos de Servicio de las fuentes (abuse.ch, Cloudflare, Telegram, etc.).

**Cualquier uso que vulnere los ToS de las fuentes es responsabilidad exclusiva del operador de la instancia.**

---

## 8. Dependencias y cadena de suministro

- Las dependencias se fijan en `requirements.txt` con versiones mínimas. Revisa `pip-audit` antes de desplegar.
- NEXUS carga **deck.gl** y **Chart.js** desde CDN pública (jsDelivr/unpkg). Si tu política lo exige, sirve estos assets localmente y ajusta `NEXUS_DECK_URL`.
- No se ejecuta código remoto: los datos se tratan como texto y se escapan antes de insertarse en HTML (`autoescape=True` en Jinja2, `esc()` en JS).
- Los enlaces externos se validan (`_safe_url()`: solo `http(s)://`) y se abren con `rel="noopener noreferrer"`.

---

## 9. Contacto

- **Seguridad:** `SpectrumSecIt@proton.me` (sustitúyelo antes de publicar).
- **Issues no sensibles:** pestaña *Issues* del repositorio.
- **Repositorio:** `https://github.com/Condor2026/Nexus_Osint_X`.
- **Condor2026**

---

## 10. Reconocimientos

Agradecemos los reportes responsables. Los investigadores que reporten vulnerabilidades confirmadas serán acreditados en el advisory correspondiente (salvo que pidan anonimato).
