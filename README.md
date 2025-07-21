
# Laboratorio: Fortificación de Metasploitable2 con Nmap, UFW e IPTables

Este laboratorio simula el proceso de detección de servicios vulnerables y aplicación de medidas de seguridad para reducir la superficie de ataque en un sistema comprometido.

---

## 1. Escaneo Inicial con Nmap

Se realizó un escaneo SYN con detección de servicios y sistema operativo:

```bash
nmap -sS -sV -O 192.168.20.129
```

### Resultado:
Se detectaron **27 puertos abiertos**, incluyendo servicios críticos inseguros como:
- `21` (vsftpd con backdoor)
- `23` (Telnet)
- `512-514` (rlogin, rexec, shell)
- `1524` (Shell remota)
- `5432` (PostgreSQL)
- `5900`, `6667`, `8009`, `8180` (servicios potencialmente explotables)

---

## 2. Cierre manual de servicios vulnerables

Se desactivaron servicios mediante scripts en `/etc/init.d/`:

```bash
sudo /etc/init.d/openbsd-inetd stop
sudo /etc/init.d/xinetd stop
sudo /etc/init.d/proftpd stop
sudo /etc/init.d/samba stop
sudo /etc/init.d/postgresql-8.3 stop
sudo killall postgres
```

---

## 3. Escaneo posterior al cierre de servicios

Se redujo la exposición, pero aún se mantenían muchos puertos abiertos:

```bash
nmap -sS -sV -O 192.168.20.129
```

- Servicios como FTP, IRC, Java-RMI, Tomcat aún estaban disponibles.

---

## 4. Intento de Fortificación con UFW

Se aplicaron las siguientes reglas:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 53/tcp
sudo ufw allow 3306/tcp
sudo ufw enable
```

Sin embargo, el escaneo mostró que `ufw` no logró bloquear todos los servicios.

---

## 5. Implementación de IPTables

Se decidió aplicar reglas manuales de `iptables`:

```bash
sudo iptables -F
sudo iptables -X
sudo iptables -Z
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
```

---

## 6. Escaneo final con IPTables activo

```bash
nmap -sS -sV -O 192.168.20.129
```

### Resultado:
Solo visibles:
- `22/tcp` (SSH)
- `80/tcp` (HTTP)

Todos los demás puertos aparecieron como **filtrados**. El sistema fue fortificado exitosamente.

---

## Evidencias

1. Escaneo inicial 
2. Cierre de servicios
3. Reglas UFW aplicadas
4. Reglas IPTables aplicadas
5. Estado final con Nmap
ARCHIVO: ESCANEO_MAP y CAPTURAS DE PANTALLA DE METASPLOTAIBLE2
---

## Conclusiones

- El escaneo inicial evidenció una máquina vulnerable con múltiples vectores de ataque.
- Se aplicaron medidas de hardening progresivas: desactivación de servicios, UFW y finalmente IPTables.
- Se logró una **defensa en profundidad** efectiva, reduciendo la superficie de ataque a solo dos servicios necesarios.

---

## Lecciones aprendidas

- Verificación continua con Nmap es clave.
- UFW no siempre es suficiente, especialmente con servicios no controlados.
- IPTables permite control granular y fue decisivo para lograr el objetivo de protección.
