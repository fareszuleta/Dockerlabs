# Redirection — DockerLabs

![DockerLabs](https://img.shields.io/badge/Plataforma-DockerLabs-blue)
![Dificultad](https://img.shields.io/badge/Dificultad-F%C3%A1cil-yellow)
![SO](https://img.shields.io/badge/SO-Debian%20Linux-orange)
![Tipo](https://img.shields.io/badge/Tipo-Bug%20Bounty-red)
![Estado](https://img.shields.io/badge/Estado-Completada-success)

Writeup de la máquina **Redirection** de DockerLabs.

## Técnicas utilizadas

- **Laboratorio 1:** Open Redirect sin validación
- **Laboratorio 2:** Bypass mediante userinfo en URL (`user@host`)
- **Laboratorio 3:** URL encoding bypass (`%5c%40`)
- Credential hunting (`/secret.bak`)
- Escalada con `sudo cp` (sobrescritua de `/etc/sudoers`)

## Flujo resumido

```
3 laboratorios de Open Redirect (progresivo)
→ Credenciales SSH en popup → secret.bak (balulito)
→ sudo cp /etc/sudoers → root
```

## Archivos

| Archivo         | Descripción              |
|-----------------|--------------------------|
| `Redirection.md`| Writeup completo         |
| `scan`          | Resultado del escaneo Nmap |

## Referencias

- [OWASP — Open Redirect](https://owasp.org/www-community/attacks/open_redirect)
- [GTFOBins — cp](https://gtfobins.github.io/gtfobins/cp/)
- [URI RFC 3986](https://tools.ietf.org/html/rfc3986#section-3.2.1)
- [DockerLabs](https://dockerlabs.es/)
