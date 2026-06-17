# Gotham — DockerLabs

![DockerLabs](https://img.shields.io/badge/Plataforma-DockerLabs-blue)
![Dificultad](https://img.shields.io/badge/Dificultad-F%C3%A1cil-yellow)
![SO](https://img.shields.io/badge/SO-Ubuntu%2022.04-orange)
![Estado](https://img.shields.io/badge/Estado-Completada-success)

Writeup de la máquina **Gotham** de DockerLabs.

## Técnicas utilizadas

- Enumeración web (comentarios HTML, robots.txt)
- JWT cracking con `jwt_tool` + manipulación de token
- Command Injection
- Fuerza bruta SSH (contraseña conocida, usuario desconocido)
- Escalada de privilegios con `sudo find` (GTFOBins)

## Flujo resumido

```
guest:guest (HTML) → JWT cracking (batman) → token admin
→ Command Injection → config.php (Arkh4m_Kn1ght!)
→ Hydra (bruce) → SSH → sudo find → root
```

## Archivos

| Archivo      | Descripción              |
|--------------|--------------------------|
| `Gotham.md`  | Writeup completo         |
| `scan`       | Resultado del escaneo Nmap |

## Referencias

- [jwt_tool](https://github.com/ticarpi/jwt_tool)
- [jwt.io](https://jwt.io/)
- [GTFOBins — find](https://gtfobins.github.io/gtfobins/find/)
- [OWASP — Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
- [DockerLabs](https://dockerlabs.es/)
