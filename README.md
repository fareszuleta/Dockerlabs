# 🐳 Dockerlabs — Writeups

Colección de writeups de máquinas resueltas en [DockerLabs](https://dockerlabs.es/) y [BunkerLabs](https://bunkerlabs.es/).  
Cada carpeta contiene el writeup detallado paso a paso con los comandos utilizados.

---

## 📂 Máquinas

| Máquina | Dificultad | Técnicas principales |
|---|---|---|
| [Bigwear](./Bigwear) | ⭐⭐ Media | WordPress · CVE-2025-34077 · RCE · File Manager |
| [Borazuwarah](./Borazuwarah) | ⭐ Muy fácil | Metadatos (exiftool) · Fuerza bruta SSH · sudo NOPASSWD bash |
| [BreakMFA](./BreakMFA) | ⭐⭐ Fácil | IDOR GraphQL · Account Takeover · Rate Limit Bypass · OTP brute force |
| [ClientBypass](./ClientBypass) | ⭐ Muy fácil | MFA bypass client-side · Manipulación de respuesta (Burp Suite) |
| [Firsthacking](./Firsthacking) | ⭐ Muy fácil | CVE-2011-2523 · vsftpd 2.3.4 backdoor · Netcat |
| [Hedgehog](./Hedgehog) | ⭐ Muy fácil | Fuerza bruta SSH · Pivote lateral (sudo) · sudo NOPASSWD bash |
| [Vacaciones](./Vacaciones) | ⭐ Muy fácil | Enumeración web · Fuerza bruta SSH · /var/mail · sudo ruby |

---

## 🛠️ Herramientas utilizadas

`nmap` · `ffuf` · `hydra` · `curl` · `exiftool` · `netcat` · `burpsuite` · `telnet` · `python3`



---

## ⚠️ Aviso legal

Todos los writeups están realizados en entornos controlados con fines educativos.  
Nunca apliques estas técnicas en sistemas sin autorización explícita.
