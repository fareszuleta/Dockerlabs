# 🖥️ Borazuwarah

## 📋 Información de la Máquina

- **Plataforma:** DockerLabs
- **Dificultad:** ⭐ Muy fácil
- **SO:** Debian Linux
- **IP:** 172.17.0.2

---

## 🎯 Skills Practicadas

✅ **Análisis de Metadatos** — Extracción de información sensible con exiftool  
✅ **Fuerza Bruta SSH** — Ataque de diccionario con Hydra  
✅ **Escalada de Privilegios** — Explotación de sudo NOPASSWD  

---

## 🔗 Cadena de Explotación

```
Enumeración Web (imagen con metadatos)
    ↓
Análisis de Metadatos (usuario: borazuwarah)
    ↓
Fuerza Bruta SSH (borazuwarah:123456)
    ↓
Escalada de Privilegios (sudo /bin/bash)
    ↓
Acceso Root Obtenido ✅
```

---

**Estado:** ✅ Completada | **Fecha:** 2026-06-12