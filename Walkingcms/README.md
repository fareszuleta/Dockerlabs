# WalkingCMS

Writeup de la máquina **WalkingCMS** de DockerLabs.

## Información

| Campo             | Valor      |
| ----------------- | ---------- |
| Máquina           | WalkingCMS |
| Dificultad        | Fácil      |
| Sistema Operativo | Linux      |
| IP                | 172.17.0.2 |

## Técnicas utilizadas

* Enumeración web con ffuf
* Enumeración de usuarios WordPress
* Fuerza bruta de credenciales
* Acceso al panel de administración
* Remote Code Execution (RCE)
* Reverse Shell
* Escalada de privilegios mediante SUID (`env`)

## Flujo de explotación

```text
Reconocimiento
    ↓
Enumeración web
    ↓
WordPress descubierto
    ↓
Enumeración de usuarios
    ↓
Bruteforce de credenciales
    ↓
Acceso al panel admin
    ↓
WP File Manager
    ↓
PHP Reverse Shell
    ↓
www-data
    ↓
SUID env
    ↓
root
```

## Credenciales obtenidas

| Usuario | Contraseña |
| ------- | ---------- |
| mario   | love       |

## Privilege Escalation

```bash
/usr/bin/env /bin/bash -p
```

## Resultado

```bash
whoami
root
```
