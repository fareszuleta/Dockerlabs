# Vacaciones — DockerLabs

![DockerLabs](https://img.shields.io/badge/Plataforma-DockerLabs-blue)
![Dificultad](https://img.shields.io/badge/Dificultad-Muy%20f%C3%A1cil-brightgreen)
![SO](https://img.shields.io/badge/SO-Ubuntu%20Linux-orange)
![Estado](https://img.shields.io/badge/Estado-Completada-success)

Writeup de la máquina **Vacaciones** de DockerLabs.

## Técnicas utilizadas

- Enumeración web (comentarios HTML)
- Fuerza bruta SSH con Hydra
- Pivote lateral entre usuarios
- Lectura de correo del sistema (`/var/mail`)
- Escalada de privilegios con `sudo ruby` (GTFOBins)

## Flujo resumido

```
curl → usuarios en HTML → Hydra (camilo) → SSH
→ /var/mail → contraseña de juan → sudo ruby → root
```

## Archivos

| Archivo         | Descripción              |
|-----------------|--------------------------|
| `Vacaciones.md` | Writeup completo         |
| `scan`          | Resultado del escaneo Nmap |

## Referencias

- [GTFOBins — ruby](https://gtfobins.github.io/gtfobins/ruby/)
- [DockerLabs](https://dockerlabs.es/)
