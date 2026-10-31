# Cuaderno de trabajo · stripchat.com

Programa: https://hackerone.com/stripchat · alcance en `alcance.md` y `scope.txt`. Solo técnicas pasivas.

```
2026-10-29 18:41 · objetivo: stripchat.com (in scope)
$ whois stripchat.com
-> registrador Amazon Registrar; creado 2006-02-13, expira 2029-02-13; titular oculto (whoisproxy.com)
-> NS amy.ns.cloudflare.com y duke.ns.cloudflare.com; sin DNSSEC; clientTransferProhibited
-> el DNS lo sirve Cloudflare: la web estará detrás de su proxy y la IP pública no será la del servidor real
siguiente: registros DNS con dig
```

```
2026-10-29 19:22 · objetivo: stripchat.com (in scope)
$ dig +short A|NS|MX|TXT stripchat.com ; dig +short TXT _dmarc.stripchat.com
-> A 104.17.117.12 y 104.17.118.12 (rango de Cloudflare); NS Cloudflare; MX de Google
-> TXT: SPF -all con redirect a spf.stripchat.com; 3 google-site-verification; mailru-verification
-> DMARC p=quarantine, informes a postmaster@stripchat.com
-> el A confirma el proxy que anticipaba el whois; correo en Google Workspace; también verificados en Mail.ru
siguiente: ver el SPF completo en spf.stripchat.com
```

```
2026-10-29 19:25 · objetivo: spf.stripchat.com (in scope)
$ dig +short TXT spf.stripchat.com
-> 5 rangos propios (185.196.194.0/24, 88.208.19.33/28, 88.208.18.224/28, 185.56.234.240/28,
   88.208.1.221/26) + Google + mail.zendesk.com, con -all; además un registro spf2.0/pra antiguo
-> el SPF publica IP propias que Cloudflare oculta en la web; usan Zendesk para soporte
siguiente: whois de los rangos
```

```
2026-10-29 19:31 · objetivo: rangos del SPF de stripchat.com (in scope)
$ whois 88.208.19.33 ; whois 185.196.194.1
-> 88.208.19.33: Advanced Hosters B.V. (NL); 185.196.194.1: DataWeb Global LP (UK, IPs en NL)
-> los dos en AS39572: mismo operador con dos razones sociales; hosting alquilado en Países Bajos
siguiente: subdominios con subfinder y crt.sh
```

```
2026-10-29 19:43 · objetivo: *.stripchat.com (in scope)
$ subfinder -d stripchat.com -silent -o subfinder.txt
-> 601 subdominios; destacan test.id, deploy, beta.fr y websocket-v6-public-origin
-> aparece mta194-11.auth.stripchat.com, que está excluido (mta*)
siguiente: cruzar con crt.sh
```

```
2026-10-29 19:56 · objetivo: *.stripchat.com (in scope)
$ curl -s "https://crt.sh/?q=%25.stripchat.com&output=json"
-> error 502 intermitente, crt.sh caído
$ curl -s "https://api.certspotter.com/v1/issuances?domain=stripchat.com&include_subdomains=true&expand=dns_names" -o certspotter.json
$ jq -r '.[].dns_names[]' certspotter.json | sort -u > certspotter.txt
-> 68 subdominios. certspotter es la misma fuente que crt.sh (certificados públicos) y sustituye a crt.sh
siguiente: unir las dos listas y marcar el alcance
```

```
2026-10-29 20:00 · objetivo: *.stripchat.com (in scope)
$ cat subfinder.txt certspotter.txt | sort -u > subs.txt
$ grep -E '^(mta|wiki|support|pxl|ext|extensions)\.' subs.txt
-> 655 únicos: solo coinciden 14 entre las dos fuentes, cada una ve una parte distinta
-> fuera de alcance: ext, pxl, support, wiki, mta.staging
-> los certificados también incluyen xhamsterlive.com y supportlivecam.com: otras marcas del mismo grupo, fuera de alcance
siguiente: robots.txt y security.txt
```

```
2026-10-29 20:04 · objetivo: stripchat.com (in scope)
$ curl -s https://stripchat.com/robots.txt
-> bloqueado por Cloudflare; repetido con -A "Mozilla/5.0 ... Chrome/126.0" -> OK
-> Disallow de /user/, /messages/, borrado de cuenta y */filter/*; 9 sitemaps; ninguna ruta tipo /admin
$ curl -s -A "Mozilla/5.0 ... Chrome/126.0" https://stripchat.com/.well-known/security.txt
-> Contact (formulario de HackerOne), Policy, Acknowledgments, Expires 2026-12-31
-> Cloudflare tiene activa la protección anti-bots; robots.txt no revela rutas sensibles
siguiente: InternetDB de las IP encontradas
```

```
2026-10-29 20:09 · objetivo: IP de stripchat.com y del SPF (in scope)
$ curl -s https://internetdb.shodan.io/104.17.118.12 | jq
-> puertos 80, 443, 2082-2096, 8080, 8443, 8880; cpe cloudflare; sin vulns
$ curl -s https://internetdb.shodan.io/88.208.19.33 | jq ; curl -s https://internetdb.shodan.io/185.196.194.1 | jq
-> "No information available" en las dos
-> la IP pública solo enseña los puertos estándar de Cloudflare; las IP propias no están indexadas
siguiente: Netcraft
```

```
2026-10-29 20:16 · objetivo: stripchat.com (in scope)
$ https://sitereport.netcraft.com/?url=https://stripchat.com
-> Cloudflare y JavaScript; Netcraft tiene el dominio desde 2003 (whois dice creación en 2006)
-> coincide con whois y dig en que delante está Cloudflare; el dominio tuvo un dueño anterior a 2006
siguiente: Google dorks
```

```
2026-10-29 20:20 · objetivo: stripchat.com (in scope)
$ Google: site:stripchat.com filetype:pdf | inurl:admin | inurl:api
-> 0 resultados: no hay documentos, paneles ni rutas de API indexados
siguiente: correos con theHarvester
```

```
2026-10-29 20:22 · objetivo: stripchat.com (in scope)
$ theHarvester -d stripchat.com -b crtsh,duckduckgo -l 200
-> 0 correos (crt.sh seguía caído); 19 hosts, nuevos: staging, vr, trk.sender, link.inbox, emt-fallback
-> encuentra 5 hosts que no daban ni subfinder ni certspotter
siguiente: Shodan con cuenta y Hunter.io
```

```
2026-10-31 16:08 · objetivo: stripchat.com (in scope)
$ Shodan: hostname:stripchat.com
-> 84 resultados, todos de Cloudflare
$ Shodan: hostname:stripchat.com -asn:AS13335
-> servidores de correo en 86.62.36.x (Intrahost Solutions, NL), banner "220 mta.stripchat.com ESMTP Postal"
-> son mta*, fuera de alcance; el nombre lleva la IP (mta36-115 = 86.62.36.115)
-> Shodan con filtro encuentra servidores propios que InternetDB no tenía
siguiente: Hunter.io
```

```
2026-10-31 16:19 · objetivo: stripchat.com (in scope)
$ Hunter.io, Domain Search: stripchat.com
-> 0 correos, igual que theHarvester; no se puede deducir el patrón. Solo postmaster@ (del DMARC)
siguiente: revisar el alcance del programa
```

```
2026-10-31 16:30 · objetivo: programa de Stripchat en HackerOne (consulta a la plataforma, no al objetivo)
$ API pública de HackerOne (hackerone.com/graphql, equipo "stripchat"): política y alcance
-> bug bounty público; dentro *.stripchat.com, go.stripchat.com y my.club; exclusiones en alcance.md
-> exige la cabecera "HackerOne: <usuario>"; las peticiones del 2026-10-29 20:04 no la llevaban
siguiente: resolver los subdominios destacados
```

```
2026-10-31 16:39 · objetivo: subdominios de *.stripchat.com (in scope)
$ for s in staging test.id deploy beta.fr websocket-v6-public-origin vr trk.sender link.inbox emt-fallback go; do
    echo "$s.stripchat.com -> $(dig +short A $s.stripchat.com | tr '\n' ' ')"; done
-> 7 de 10 en Cloudflare (104.17.117.12 y 104.17.118.12); staging no resuelve
-> websocket-v6-public-origin -> 77.93.81.10, fuera de Cloudflare
-> link.inbox -> CNAME a mayflower.bapi-02.moengage.com -> CloudFront (18.154.48.x)
-> staging lo daba theHarvester pero ya no existe en DNS; websocket-v6-public-origin no pasa por el proxy
siguiente: whois e InternetDB de 77.93.81.10
```

```
2026-10-31 16:41 · objetivo: 77.93.81.10 (websocket-v6-public-origin.stripchat.com, in scope)
$ whois 77.93.81.10 | grep -iE 'netname|org-name|orgname|descr|country|origin'
-> netname NL-INTRAHOST-NETS-20251124-81-2, NL, AS214124
-> mismo proveedor que los servidores de correo de 86.62.36.x; otra red distinta de la del SPF (AS39572)
$ curl -s https://internetdb.shodan.io/77.93.81.10 | jq
-> "No information available"
siguiente: DNSDumpster
```

```
2026-10-31 16:46 · objetivo: stripchat.com (in scope)
$ https://dnsdumpster.com (stripchat.com)
-> 138 registros (la versión gratuita muestra 50), casi todos Cloudflare
-> mta194-11/-12/-72.auth.stripchat.com -> 185.196.194.11/.12/.72, AS39572; fuera de alcance (mta*)
-> coincide con el SPF: el rango 185.196.194.0/24 son servidores de envío de correo
siguiente: Google dorks que faltan
```

```
2026-10-31 16:50 · objetivo: stripchat.com (in scope)
$ Google (sin SafeSearch): site:stripchat.com -site:www.stripchat.com
-> ~6.710.000 resultados, páginas del sitio en muchos idiomas
$ Google (GHDB, Directories): site:stripchat.com intitle:"index of"
-> 0 resultados
-> mucho contenido indexado, pero ningún listado de directorios ni documento interno
siguiente: redactar mapa.md; antes, completar Netcraft y reintentar crt.sh
```

```
2026-10-31 17:27 · objetivo: stripchat.com (in scope)
$ https://sitereport.netcraft.com/?url=https://stripchat.com (informe completo)
-> site rank 566; IPv4 104.17.117.12 e IPv6 2606:4700::6811:760c, las dos de Cloudflare (AS13335)
-> tecnologías: Cloudflare (CDN), SSL, JavaScript, AJAX, Google Hosted Libraries, Local Storage, HTML5
-> cabeceras de seguridad: CSP, HSTS con subdominios, X-Frame-Options DENY; sin trackers de terceros
-> no hay histórico de hosting en el informe público
-> Netcraft marca DNSSEC "Enabled" y el whois del 10-29 decía "unsigned": no coinciden
siguiente: comprobar DNSSEC con dig DS
```

```
2026-10-31 17:27 · objetivo: *.stripchat.com (in scope)
$ curl -s "https://crt.sh/?q=%25.stripchat.com&output=json" -o crtsh.json && jq -r '.[].name_value' crtsh.json | sort -u > crtsh.txt; wc -l crtsh.txt
-> 106 nombres (crt.sh ya funciona)
$ comm -13 <(sort -u subs.txt) crtsh.txt | wc -l
-> 58 que no estaban en subfinder + certspotter: certspotter y crt.sh son la misma fuente pero no dan lo mismo
siguiente: revisar los 58 nuevos y marcar el alcance
```

```
2026-10-31 17:27 · objetivo: 77.93.81.10 (websocket-v6-public-origin.stripchat.com, in scope)
$ https://www.shodan.io/host/77.93.81.10
-> sin resultados, igual que InternetDB
-> ni Shodan ni InternetDB tienen datos del origen: no se puede saber qué expone sin pasar a la fase activa
siguiente: Wayback Machine
```

```
2026-10-31 17:30 · objetivo: stripchat.com (in scope)
$ dig +short DS stripchat.com
-> vacío: no hay registro DS, el dominio no tiene DNSSEC
-> el whois tenía razón ("unsigned") y Netcraft no
siguiente: revisar los 58 nombres nuevos de crt.sh
```

```
2026-10-31 17:30 · objetivo: *.stripchat.com (in scope)
$ comm -13 <(sort -u subs.txt) crtsh.txt
-> de los 58, solo 3 son subdominios de stripchat.com: *.stage, *.maxxcams.com y *.pinkflamelive.com
-> los otros 55 son dominios distintos que contienen "stripchat.com" en el nombre (eng--stripchat.com,
   en-en-eng-stripchat.com, eng-us-stripchat.com...), con subdominios por defecto de cPanel
   (cpanel, webmail, autodiscover) y cruces con en-eng-jerkmate.com
-> crt.sh con %25.stripchat.com busca por texto y no por dominio: hay que filtrar con '\.stripchat\.com$'
-> esos 55 están fuera de alcance; parecen dominios que imitan a la marca (no se investigan)
-> *.stage.stripchat.com: certificado comodín para un entorno de preproducción, dentro de alcance
siguiente: resolver los tres subdominios nuevos
```

```
2026-10-31 17:30 · objetivo: *.stripchat.com (in scope)
$ curl -s "https://web.archive.org/cdx/search/cdx?url=*.stripchat.com&fl=original&collapse=urlkey&limit=200000" -o wayback.txt; wc -l wayback.txt
-> 200000 líneas: llega al límite, el resultado está incompleto
$ grep -iE '\.(env|bak|sql|zip|log|config)$|/admin|/api/' wayback.txt | sort -u | head -30
-> /1.sql, /1111.zip, /2.zip, /4.zip, /admin.html, /admin.php y la API /api/embed/v1/models/username/<modelo>
-> Wayback guarda las URL que alguien pidió, aunque no existan: los .sql, .zip y admin pueden ser
   peticiones de escáneres de terceros. La API de embed lista perfiles públicos de modelos (no se copian los nombres)
siguiente: mirar en Wayback el código de estado de esas rutas, sin visitarlas en stripchat.com
```

```
2026-10-31 17:32 · objetivo: stripchat.com (in scope)
$ for p in 1.sql 1111.zip 2.zip 4.zip admin.html admin.php; do
    echo "$p: $(curl -s "https://web.archive.org/cdx/search/cdx?url=stripchat.com/$p&fl=statuscode" | sort | uniq -c | tr '\n' ' ')"; done
-> todas las capturas son 404 (1.sql 1, 1111.zip 1, 2.zip 1, 4.zip 1, admin.html 2, admin.php 4)
-> esos ficheros nunca existieron: son peticiones de escáneres que Wayback registró. Se descartan
siguiente: resolver los subdominios nuevos de crt.sh
```

```
2026-10-31 17:32 · objetivo: stage, maxxcams.com y pinkflamelive.com de *.stripchat.com (in scope)
$ dig +short A stage.stripchat.com ; dig +short A maxxcams.com.stripchat.com ; dig +short A pinkflamelive.com.stripchat.com
-> stage.stripchat.com -> 86.62.36.112, fuera de Cloudflare
-> maxxcams.com y pinkflamelive.com -> 104.17.117.12 / 104.17.118.12 (Cloudflare)
-> stage está en el mismo rango que los servidores de correo mta* (86.62.36.x, Intrahost). El host está en
   alcance (no es mta*); la IP es vecina de las excluidas
-> segundo servidor en alcance sin proxy, y además es preproducción
siguiente: actualizar mapa.md
```
