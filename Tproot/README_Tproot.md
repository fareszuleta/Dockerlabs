# Tproot — DockerLabs

![DockerLabs](https://img.shields.io/badge/Plataforma-DockerLabs-blue)
![Dificultad](https://img.shields.io/badge/Dificultad-Muy%20f%C3%A1cil-brightgreen)
![SO](https://img.shields.io/badge/SO-Unix-orange)
![CVE](https://img.shields.io/badge/CVE-2011--2523-red)
![Estado](https://img.shields.io/badge/Estado-Completada-success)

Writeup de la máquina **Tproot** de DockerLabs.

## Técnicas utilizadas

- Enumeración de versiones de servicios (Nmap)
- Explotación de CVE-2011-2523 (vsftpd 2.3.4 backdoor)
- Script propio en Python (`CVE-2011-2523.py`)
- Método manual con Telnet + Netcat

## Flujo resumido

```
Nmap → vsftpd 2.3.4 → CVE-2011-2523 → backdoor puerto 6200 → root
```

## Archivos

| Archivo              | Descripción                       |
|----------------------|-----------------------------------|
| `Tproot.md`          | Writeup completo                  |
| `CVE-2011-2523.py`   | Exploit adaptado para esta máquina|
| `scan`               | Resultado del escaneo Nmap        |

## Referencias

- [CVE-2011-2523 — NVD](https://nvd.nist.gov/vuln/detail/CVE-2011-2523)
- [GTFOBins](https://gtfobins.github.io/)
- [DockerLabs](https://dockerlabs.es/)
