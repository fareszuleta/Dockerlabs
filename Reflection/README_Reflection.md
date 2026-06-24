# Reflection — DockerLabs

![DockerLabs](https://img.shields.io/badge/Plataforma-DockerLabs-blue)
![Dificultad](https://img.shields.io/badge/Dificultad-F%C3%A1cil-yellow)
![SO](https://img.shields.io/badge/SO-Debian%20Linux-orange)
![Tipo](https://img.shields.io/badge/Tipo-Bug%20Bounty-red)
![Estado](https://img.shields.io/badge/Estado-Completada-success)

Writeup de la máquina **Reflection** de DockerLabs.

## Técnicas utilizadas

- **XSS Reflected** — payload reflejado en página
- **XSS Stored** — payload almacenado en BD
- **XSS con URL encoding** — payload codificado en dropdowns
- **XSS en GET params** — payload inyectado vía parámetros
- **SUID env exploitation** — escalada a root

## Flujo resumido

```
4 laboratorios XSS (progresivos: Reflected → Stored → URL-encoded → GET params)
→ SSH (balu:balulero) → SUID env /bin/sh -p → root
```

## Archivos

| Archivo        | Descripción              |
|----------------|--------------------------|
| `Reflection.md`| Writeup completo         |
| `scan`         | Resultado del escaneo Nmap |

## Payload base

```html
<img src='1' onerror='alert(0)' >
```

Usado en todos los 4 laboratorios con variaciones de encoding y context.

## Referencias

- [OWASP — Cross Site Scripting (XSS)](https://owasp.org/www-community/attacks/xss/)
- [GTFOBins — env](https://gtfobins.github.io/gtfobins/env/)
- [PortSwigger — XSS](https://portswigger.net/web-security/cross-site-scripting)
- [DockerLabs](https://dockerlabs.es/)
