# Política de seguridad

## Versiones soportadas

| Versión | Soporte |
|---|---|
| 3.1.x | ✅ |
| < 3.1 | ❌ |

## Reportar una vulnerabilidad

Abre un aviso privado en GitHub (*Security → Report a vulnerability*) o escribe a `<tu-email>`. **No** abras un issue público con detalles explotables.

Incluye: versión, cómo reproducir, impacto y, si puedes, un parche.

## Buenas prácticas al desplegar

- Cambia `ADMIN_PASSWORD` y genera un `JWT_SECRET` aleatorio.
- **No** expongas el puerto 8000 directamente a Internet: pon un proxy inverso con TLS (Caddy, nginx) y desactiva `NEXUS_LOCAL_NOAUTH`.
- Revisa que `.env` y `nexus.db` no estén en el repo ni en backups públicos.
- Las API keys dan acceso a tus cuotas: rótalas si sospechas filtración.
- El bypass de loopback (`NEXUS_LOCAL_NOAUTH=1`) valida `Host`, `Origin` y `Sec-Fetch-Site`, pero un proxy mal configurado puede saltárselo. En producción ponlo a `0`.

## Alcance

Este proyecto **no** realiza acciones ofensivas: no escanea, no explota, no contacta con C2. Los indicadores se muestran *defangeados*. Cualquier uso que vulnere los ToS de las fuentes es responsabilidad del operador.
