# Alcance

**Programa:** Stripchat Bug Bounty Program (HackerOne) · https://hackerone.com/stripchat
**Consultado:** 2026-10-31
**Tipo:** bug bounty público, con recompensa

## Dentro del alcance

- `*.stripchat.com`
- `go.stripchat.com` (solo cuentan vulnerabilidades de severidad alta o crítica)
- `my.club`

El dominio raíz `stripchat.com` no aparece como activo propio, pero el programa excluye rutas suyas (`/page*` y `/wiki`). Por eso lo considero dentro y lo añado aparte en `scope.txt`, porque `*.stripchat.com` no lo incluye.

## Fuera del alcance

- `mta*.stripchat.com`
- `pxl.stripchat.com`
- `support.stripchat.com`
- `wiki.stripchat.com` y `https://stripchat.com/wiki`
- `ext.stripchat.com` y `extensions.stripchat.com`
- `https://stripchat.com/page*`
- `websocket-apps.sc-apps.com`

## Reglas que afectan a este proyecto

- **¿Permite herramientas automáticas?** Sí, hasta 100 peticiones por segundo y nunca durante el mantenimiento. En `my.club`, solo de 7:00 a 15:00 UTC.
- **¿Límite de peticiones?** 100 por segundo.
- **¿Prohíbe contactar con empleados?** Sí. Toda comunicación va por HackerOne y está prohibida la ingeniería social contra empleados, soporte y contratas.
- **¿Requisito de identificación?** Sí. Toda petición HTTP debe llevar la cabecera `HackerOne: <usuario>` (en `my.club`, `X-HackerOne-Research: <usuario>`). Las dos peticiones a `robots.txt` y `security.txt` del 2026-10-29 se hicieron sin ella (ver el cuaderno). A partir de ahí, cualquier petición la lleva.
- **Otras:** solo se puede probar con cuentas propias, está prohibido acceder a datos de usuarios y no se pueden publicar vulnerabilidades fuera de HackerOne. En este proyecto no se crean cuentas y solo se usan técnicas pasivas.
