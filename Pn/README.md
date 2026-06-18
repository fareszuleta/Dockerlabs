# Pn — DockerLabs

![DockerLabs](https://img.shields.io/badge/Plataforma-DockerLabs-blue)
![Dificultad](https://img.shields.io/badge/Dificultad-F%C3%A1cil-yellow)
![SO](https://img.shields.io/badge/SO-Unix-orange)
![Estado](https://img.shields.io/badge/Estado-Completada-success)

Writeup de la máquina **Pn** de DockerLabs.

## Técnicas utilizadas

- FTP anónimo — descarga de archivo con pista
- Apache Tomcat Manager con credenciales por defecto (`tomcat:s3cr3t`)
- RCE via WAR upload — dos métodos:
  - Metasploit (`tomcat_mgr_upload`)
  - Manual con `msfvenom` + `netcat`

## Flujo resumido

```
FTP anónimo (tomcat.txt) → Tomcat Manager (tomcat:s3cr3t)
→ WAR upload → RCE → root directo
```

## Archivos

| Archivo  | Descripción                |
|----------|----------------------------|
| `Pn.md`  | Writeup completo           |
| `scan`   | Resultado del escaneo Nmap |
| `tomcat.txt`   | Contenido del archivo dentro del protocolo |
| `shell.war`   | Archivo WAR malicioso |

## Referencias

- [HackTricks — Apache Tomcat](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/tomcat)
- [Metasploit — tomcat_mgr_upload](https://www.rapid7.com/db/modules/exploit/multi/http/tomcat_mgr_upload/)
- [revshells.com](https://www.revshells.com/)
- [DockerLabs](https://dockerlabs.es/)
