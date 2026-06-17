# Trust — DockerLabs

![DockerLabs](https://img.shields.io/badge/Plataforma-DockerLabs-blue)
![Dificultad](https://img.shields.io/badge/Dificultad-Muy%20f%C3%A1cil-brightgreen)
![SO](https://img.shields.io/badge/SO-Debian%20Linux-orange)
![Estado](https://img.shields.io/badge/Estado-Completada-success)

Writeup de la máquina **Trust** de DockerLabs.

## Técnicas utilizadas

- Enumeración web (Apache2 por defecto)
- Fuzzing de directorios con ffuf (`secret.php`)
- Fuerza bruta SSH con Hydra
- Escalada de privilegios con `sudo vim` (GTFOBins)

## Flujo resumido

```
Nmap → ffuf (secret.php → usuario mario) → Hydra (mario:chocolate)
→ SSH → sudo vim → :shell → root
```

## Archivos

| Archivo | Descripción |
|---------|-------------|
| `Trust_github.md` | Writeup completo |
| `scan` | Resultado del escaneo Nmap |

## Referencias

- [GTFOBins — vim](https://gtfobins.github.io/gtfobins/vim/)
- [SecLists](https://github.com/danielmiessler/SecLists)
- [THC Hydra](https://github.com/vanhauser-thc/thc-hydra)
- [DockerLabs](https://dockerlabs.es/)
