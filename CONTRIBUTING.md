# Contribuir a NEXUS OSINT

## Antes de nada

- **No subas `.env`, `nexus.db`, `GeoLite2-*.mmdb` ni `.cache/`.** Están en `.gitignore`; si se te cuelan, bórralos del índice con `git rm --cached`.
- Para probar un colector nuevo, usa una rama: `git checkout -b feat/fuente-xyz`.

## Estructura

Todo vive en `nexus.py`, seccionado con `§ N.` en comentarios. Añade tu colector en `§ 6` y regístralo en el diccionario `SOURCES`. Si devuelve datos:

- Cada evento debe ser un `dict` con al menos `domain`, `source`, `lat`, `lon`, `severity`, `title`.
- Si necesitas geolocalizar IPs, usa `await geo_batch(ips)` y luego `await ip_to_coords(ip)`; **nunca** inventes `(0,0)`.
- Si tu fuente necesita API key, lee `os.getenv(...)` y devuelve `[]` si falta. Añádela a `.env.example`.
- Los textos maliciosos deben mostrarse con `_defang()` y los enlaces pasar por `_safe_url()`.

## Estilo

- 4 espacios, sin tabs.
- Las funciones nuevas de análisis, con `★ NUEVO` en el docstring si abren sección.
- No rompas la firma pública de `run_full_sweep()`, `build_map_payload()` ni de los endpoints existentes.

## Pull requests

- Un PR = un cambio. Mensajes en presente (`add: colector X`, `fix: dedupe en bordes`).
- Si tocas el frontend, comprueba que `j`, `r` y `/` siguen funcionando y que el mapa no se recrea al filtrar.
- Si tocas SQLite, actualiza `SCHEMA` **y** `_migrate_schema()` en el mismo commit.

Gracias por aportar.
