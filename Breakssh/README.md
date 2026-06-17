# BreakMySSH — DockerLabs

![DockerLabs](https://img.shields.io/badge/Plataforma-DockerLabs-blue)
![Dificultad](https://img.shields.io/badge/Dificultad-Muy%20f%C3%A1cil-brightgreen)
![SO](https://img.shields.io/badge/SO-Debian%20Linux-orange)
![Estado](https://img.shields.io/badge/Estado-Completada-success)

Writeup de la máquina **BreakMySSH** de DockerLabs.

## Técnicas utilizadas

- Fuerza bruta SSH con diccionario de usuarios y contraseñas (Hydra)
- Explotación de `PermitRootLogin yes` en configuración SSH insegura

## Flujo resumido

```
Nmap (solo SSH) → Hydra (root:estrella) → SSH → root directo
```

## Archivos

| Archivo          | Descripción                |
|------------------|----------------------------|
| `BreakMySSH.md`  | Writeup completo           |
| `scan`           | Resultado del escaneo Nmap |
| `hydra.txt`      | Resultado del escaneo Hydra|

## Referencias

- [Hydra — THC](https://github.com/vanhauser-thc/thc-hydra)
- [SecLists](https://github.com/danielmiessler/SecLists)
- [OpenSSH hardening guide](https://www.ssh.com/academy/ssh/sshd_config)
- [DockerLabs](https://dockerlabs.es/)
