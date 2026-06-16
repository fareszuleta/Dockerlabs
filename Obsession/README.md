# Obsession — DockerLabs

![DockerLabs](https://img.shields.io/badge/Plataforma-DockerLabs-blue)
![Dificultad](https://img.shields.io/badge/Dificultad-Muy%20f%C3%A1cil-brightgreen)
![SO](https://img.shields.io/badge/SO-Ubuntu%20Linux-orange)
![Estado](https://img.shields.io/badge/Estado-Completada-success)

Writeup de la máquina **Obsession** de DockerLabs.

## Técnicas utilizadas

- Enumeración FTP con login anónimo
- Análisis de comentarios en código fuente HTML
- Fuzzing de directorios con ffuf
- Fuerza bruta SSH con Hydra
- Escalada de privilegios con `sudo vim` (GTFOBins)

## Flujo resumido

```
Nmap → FTP anón (chat-gonza.txt + pendientes.txt) → curl HTML (comentario usuario)
→ ffuf (/backup/backup.txt) → Hydra (russoski) → SSH → sudo vim → :shell → root
```

## Archivos

| Archivo | Descripción |
|---------|-------------|
| `Obsession_github.md` | Writeup completo |
| `scan` | Resultado del escaneo Nmap |

## Referencias

- [GTFOBins — vim](https://gtfobins.github.io/gtfobins/vim/)
- [DockerLabs](https://dockerlabs.es/)
