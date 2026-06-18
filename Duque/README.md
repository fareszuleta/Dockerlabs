# Duque — DockerLabs

![DockerLabs](https://img.shields.io/badge/Plataforma-DockerLabs-blue)
![Dificultad](https://img.shields.io/badge/Dificultad-F%C3%A1cil-yellow)
![SO](https://img.shields.io/badge/SO-Ubuntu%2022.04-orange)
![Estado](https://img.shields.io/badge/Estado-Completada-success)

Writeup de la máquina **Duque** de DockerLabs.

## Técnicas utilizadas

- Enumeración web recursiva con ffuf
- SQL Injection + SQLMap (dump de base de datos)
- IDOR en parámetro GET con fuerza bruta de IDs
- Lectura de código fuente del servidor con `LOAD_FILE` via SQLMap
- Escalada de privilegios con SUID en `/usr/bin/env`

## Flujo resumido

```
ffuf → /bills/ → SQLi → SQLMap (admin:admin123) → login admin
→ IDOR (xyc724) → duque:duquelaje81029557! → SSH → SUID env → root
```

## Archivos

| Archivo    | Descripción               |
|------------|---------------------------|
| `Duque.md` | Writeup completo          |
| `scan`     | Resultado del escaneo Nmap|


## Referencias

- [SQLMap](https://sqlmap.org/)
- [GTFOBins — env](https://gtfobins.github.io/gtfobins/env/#suid)
- [OWASP — SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
- [OWASP — IDOR](https://owasp.org/www-chapter-ghana/assets/slides/IDOR.pdf)
- [DockerLabs](https://dockerlabs.es/)
