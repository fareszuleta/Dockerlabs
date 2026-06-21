# Upload — DockerLabs

![DockerLabs](https://img.shields.io/badge/Plataforma-DockerLabs-blue)
![Dificultad](https://img.shields.io/badge/Dificultad-Muy%20f%C3%A1cil-brightgreen)
![SO](https://img.shields.io/badge/SO-Ubuntu%20Linux-orange)
![Estado](https://img.shields.io/badge/Estado-Completada-success)

Writeup de la máquina **Upload** de DockerLabs.

## Técnicas utilizadas

- Enumeración web con ffuf
- File upload sin validación de tipo
- PHP reverse shell (revshells.com)
- RCE via PHP upload
- Escalada con `sudo env` (GTFOBins)

## Flujo resumido

```
Upload page → ffuf (/uploads) → PHP reverse shell upload
→ RCE (www-data) → sudo env /bin/sh → root
```

## Archivos

| Archivo        | Descripción              |
|----------------|--------------------------|
| `Upload.md`    | Writeup completo         |
| `scan`         | Resultado del escaneo Nmap |

## Referencias

- [revshells.com](https://www.revshells.com/)
- [GTFOBins — env](https://gtfobins.org/gtfobins/env/)
- [OWASP — File Upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
- [DockerLabs](https://dockerlabs.es/)
