# Hedgehog

## 📋 Información de la Máquina

- **Plataforma:** DockerLabs
- **Dificultad:** ⭐ Muy fácil
- **SO:** Ubuntu 24.04.1 LTS
- **IP:** 172.17.0.2

---

## 🎯 Skills Practicadas

✅ **Enumeración Web** — Descubrimiento de información sensible en servicios HTTP  
✅ **Fuerza Bruta SSH** — Ataque de diccionario con Hydra  
✅ **Escalada Lateral** — Pivote entre usuarios con permisos sudoers  
✅ **Escalada Vertical** — Elevación de privilegios a root  

---

## 🔗 Cadena de Explotación

```
Enumeración Web (usuario: tails)
    ↓
Fuerza Bruta SSH (tails:3117548331)
    ↓
Escalada Lateral (tails → sonic)
    ↓
Escalada Vertical (sonic → root)
    ↓
Acceso Root Obtenido ✅
```

---
