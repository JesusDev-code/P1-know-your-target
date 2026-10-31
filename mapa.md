# Mapa de superficie de ataque · stripchat.com

Reconocimiento pasivo del 2026-10-29 al 2026-10-31. La columna "Cuaderno" indica la entrada de la
que sale cada dato.

## 1. Resumen

Stripchat es una plataforma de *webcams* en directo con un programa de bug bounty público en
HackerOne. El dominio existe desde 2006 y el titular está oculto. Toda la web pasa por Cloudflare,
el correo corporativo está en Google Workspace y el soporte en Zendesk. Comparte certificados e
infraestructura con otras marcas del mismo grupo.

Lo más relevante: dos subdominios dentro del alcance resuelven fuera de Cloudflare.
`stage.stripchat.com` (preproducción) apunta a 86.62.36.112 y
`websocket-v6-public-origin.stripchat.com` a 77.93.81.10, ambos en Intrahost (Países Bajos). El SPF
y Shodan muestran además la infraestructura propia de correo. La exposición pública es baja: no hay
correos, documentos ni paneles indexados.

## 2. Dominios y subdominios

Total: 658 subdominios únicos (601 de subfinder, 68 de certspotter y 3 más de crt.sh). DNSDumpster
cuenta 138 registros. crt.sh devolvió además 55 dominios que no son de stripchat.com pero contienen
el texto en su nombre (`eng--stripchat.com`, `en-en-eng-stripchat.com`…). Están fuera de alcance y
parecen dominios que imitan a la marca (10-31 17:30).

| Subdominio | IP | En alcance | Para qué parece servir | Cuaderno |
|---|---|---|---|---|
| stripchat.com | 104.17.117.12, 104.17.118.12 (Cloudflare) | Sí | Web principal | 10-29 19:22 |
| stage.stripchat.com | 86.62.36.112 (Intrahost, NL) | Sí | Preproducción, sin Cloudflare | 10-31 17:30, 17:32 |
| websocket-v6-public-origin.stripchat.com | 77.93.81.10 (Intrahost, NL) | Sí | Origen de *websockets*, sin Cloudflare | 10-31 16:39, 16:41 |
| test.id.stripchat.com | Cloudflare | Sí | Pruebas del servicio de identidad | 10-31 16:39 |
| deploy.stripchat.com | Cloudflare | Sí | Despliegue | 10-31 16:39 |
| beta.fr.stripchat.com | Cloudflare | Sí | Versión beta en francés | 10-31 16:39 |
| vr.stripchat.com | Cloudflare | Sí | Realidad virtual | 10-31 16:39 |
| go.stripchat.com | Cloudflare | Sí (solo severidad alta o crítica) | Desconocido | 10-31 16:39 |
| maxxcams.com.stripchat.com | Cloudflare | Sí | Marca blanca (comodín en el certificado) | 10-31 17:30, 17:32 |
| pinkflamelive.com.stripchat.com | Cloudflare | Sí | Marca blanca (comodín en el certificado) | 10-31 17:30, 17:32 |
| trk.sender.stripchat.com | Cloudflare | Sí | Seguimiento de correos de marketing | 10-31 16:39 |
| emt-fallback.stripchat.com | Cloudflare | Sí | Respaldo de envío de correo | 10-31 16:39 |
| link.inbox.stripchat.com | CNAME a MoEngage (CloudFront, 18.154.48.x) | Sí | Enlaces de marketing, servicio externo | 10-31 16:39 |
| staging.stripchat.com | No resuelve | Sí | Preproducción (nombre antiguo) | 10-29 20:22; 10-31 16:39 |
| mta36-115 y otros mta* | 86.62.36.x (Intrahost, NL) | No | Servidores de correo (Postal) | 10-31 16:08 |
| mta194-11/12/72.auth.stripchat.com | 185.196.194.11/12/72 (AS39572) | No | Servidores de correo | 10-31 16:46 |
| support, wiki, ext, pxl | — | No | Soporte, wiki, extensiones, píxel | 10-29 20:00 |

## 3. Infraestructura y servicios

| Elemento | Dato | Cuaderno |
|---|---|---|
| Registro | Amazon Registrar; 2006-02-13 a 2029-02-13; titular oculto; sin DNSSEC (sin registro DS, aunque Netcraft lo marca como activo) | 10-29 18:41; 10-31 17:27, 17:30 |
| Histórico | Netcraft lo registra desde 2003, antes de la creación actual; site rank 566 | 10-29 20:16; 10-31 17:27 |
| CDN/WAF | Cloudflare (NS, IPv4 e IPv6, AS13335) con protección anti-bots | 10-29 19:22, 20:04; 10-31 17:27 |
| Tecnologías | Cloudflare, SSL, JavaScript, AJAX, Google Hosted Libraries, HTML5. Cabeceras CSP, HSTS con subdominios y X-Frame-Options DENY | 10-31 17:27 |
| Correo | Google Workspace; SPF -all; DMARC p=quarantine | 10-29 19:22 |
| Envío propio de correo | 5 rangos en el SPF (AS39572, NL) y servidores Postal en 86.62.36.x (Intrahost) | 10-29 19:25, 19:31; 10-31 16:08, 16:46 |
| Servidores sin proxy | stage (86.62.36.112) y websocket-v6-public-origin (77.93.81.10, AS214124), los dos en Intrahost | 10-31 16:41, 17:32 |
| Servicios externos | Google Workspace, Mail.ru (verificación), Zendesk, MoEngage | 10-29 19:22, 19:25; 10-31 16:39 |
| Shodan | 84 resultados de Cloudflare con los puertos estándar; las IP propias consultadas no están indexadas | 10-29 20:09; 10-31 16:08, 16:41, 17:27 |
| Rutas antiguas (Wayback) | API pública `/api/embed/v1/models/...`. Las rutas `.sql`, `.zip` y `admin` archivadas son todas 404: peticiones de escáneres, no ficheros reales | 10-31 17:30, 17:32 |

## 4. Correos y personas

- No se ha encontrado ningún correo: theHarvester y Hunter.io devuelven 0 (10-29 20:22, 10-31 16:19).
- Sin correos no se puede deducir el patrón. La única cuenta conocida es `postmaster@stripchat.com`,
  que figura en el DMARC (10-29 19:22).
- El contacto de seguridad es el formulario de HackerOne, según `security.txt` (10-29 20:04).
- No hay empleados visibles en las fuentes consultadas y el whois oculta al titular.

## 5. Puntos de interés

1. **stage.stripchat.com (86.62.36.112).** Es un entorno de preproducción y no pasa por Cloudflare.
   Los entornos de preproducción suelen tener menos controles que producción, y aquí además no hay
   WAF delante. Lo primero sería ver qué servicios expone y si sirve una versión de la aplicación.
2. **websocket-v6-public-origin.stripchat.com (77.93.81.10).** Es otro servidor de origen sin
   Cloudflare. Ni Shodan ni InternetDB tienen datos de él, así que habría que ver qué servicios
   expone además de los *websockets*.
3. **test.id.stripchat.com.** Es un entorno de pruebas del servicio de identidad. Aunque está detrás
   de Cloudflare, un fallo de autenticación ahí permitiría tomar cuentas, que es el impacto más alto
   en la plataforma.
