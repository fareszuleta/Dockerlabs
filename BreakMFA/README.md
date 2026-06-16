# 🖥️ BreakMFA

## 📋 Información de la Máquina

- **Plataforma:** BunkerLabs
- **Dificultad:** ⭐⭐ Fácil
- **Stack:** Werkzeug / Python 3.12 / GraphQL
- **IP:** 172.17.0.2

---

## 🎯 Skills Practicadas

✅ **IDOR en GraphQL** — Explotación de autorización débil en mutaciones GraphQL  
✅ **Account Takeover** — Suplantación de cuentas mediante vulnerabilidades de autenticación  
✅ **Bypass de Rate Limit** — Evasion de límite de intentos con X-Forwarded-For  
✅ **Fuerza Bruta de OTP** — Ataque de diccionario contra códigos MFA con ffuf  

---

## 🔗 Cadena de Explotación

```
Login como user@user.es (password123)
    ↓
MFA solicitado
    ↓
IDOR en GraphQL: Cambiar email a admin@admin.es
    ↓
Código MFA generado para admin
    ↓
Rate limit bloqueado (429)
    ↓
Bypass con X-Forwarded-For
    ↓
Fuerza bruta OTP con ffuf (pitchfork)
    ↓
Código correcto encontrado
    ↓
Acceso al Dashboard como admin ✅
```

---

**Estado:** ✅ Completada | **Fecha:** 2026-06-15