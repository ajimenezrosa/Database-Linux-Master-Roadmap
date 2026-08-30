# 35. Puertos, Sockets y Servicios de Red

> **Módulo 6: Redes en Red Hat Enterprise Linux**  
> **Página 35 del Manual RHCSA**

---

# Objetivos de aprendizaje

Al finalizar esta lección serás capaz de:

- Comprender qué son los puertos de red.
- Diferenciar entre TCP y UDP.
- Entender el concepto de Socket.
- Identificar qué servicios están escuchando en el servidor.
- Utilizar la herramienta `ss`.
- Verificar puertos abiertos.
- Diagnosticar problemas relacionados con servicios de red.

---

# Introducción

Cuando una aplicación necesita comunicarse a través de la red utiliza un **puerto**.

Un servidor Linux puede tener cientos de aplicaciones ejecutándose al mismo tiempo, y cada una utiliza uno o varios puertos para enviar y recibir información.

Por ejemplo:

```
Internet
      │
      ▼
+----------------------+
|     Servidor Linux   |
|                      |
| Puerto 22  → SSH     |
| Puerto 80  → HTTP    |
| Puerto 443 → HTTPS   |
| Puerto 5432 → PostgreSQL |
| Puerto 1433 → SQL Server |
+----------------------+
```

---

# ¿Qué es un puerto?

Un **puerto** es un número lógico asociado a un servicio de red.

Permite que múltiples aplicaciones utilicen la misma dirección IP sin interferirse entre sí.

Cada puerto identifica un servicio diferente.

Ejemplos:

```
Servidor

IP:
192.168.1.10

Puerto:
22

↓

SSH
```

---

# Rango de puertos

Los puertos tienen valores entre:

```
0

↓

65535
```

---

# Clasificación de puertos

## Puertos bien conocidos (Well-Known)

```
0 - 1023
```

Ejemplos:

| Puerto | Servicio |
|---------|----------|
| 20 | FTP Data |
| 21 | FTP |
| 22 | SSH |
| 23 | Telnet |
| 25 | SMTP |
| 53 | DNS |
| 67 | DHCP |
| 68 | DHCP Cliente |
| 80 | HTTP |
| 110 | POP3 |
| 143 | IMAP |
| 443 | HTTPS |

---

## Puertos registrados

```
1024 - 49151
```

Utilizados por aplicaciones.

Ejemplos:

```
1433 SQL Server

1521 Oracle

3306 MySQL

5432 PostgreSQL

6379 Redis
```

---

## Puertos dinámicos

```
49152 - 65535
```

Normalmente utilizados por clientes.

---

# ¿Qué es TCP?

TCP (Transmission Control Protocol) proporciona:

- Confiabilidad.
- Confirmación de entrega.
- Reenvío de paquetes perdidos.
- Control de errores.

Utilizado por:

- SSH
- HTTP
- HTTPS
- PostgreSQL
- SQL Server

---

# ¿Qué es UDP?

UDP (User Datagram Protocol):

- Más rápido.
- Menor sobrecarga.
- No garantiza la entrega.

Utilizado por:

- DNS
- Streaming
- VoIP
- Juegos en línea

---

# TCP vs UDP

| TCP | UDP |
|------|------|
| Confiable | No garantiza entrega |
| Más lento | Más rápido |
| Confirma recepción | Sin confirmación |
| Orientado a conexión | Sin conexión |

---

# ¿Qué es un Socket?

Un **Socket** representa un punto de comunicación entre dos procesos.

Está formado por:

```
Dirección IP

+

Puerto

+

Protocolo
```

Ejemplo:

```
192.168.1.100

Puerto 22

TCP
```

---

# Mostrar todos los sockets

```bash
ss
```

---

# Mostrar únicamente sockets TCP

```bash
ss -t
```

---

# Mostrar sockets UDP

```bash
ss -u
```

---

# Mostrar sockets en escucha

```bash
ss -l
```

---

# Mostrar TCP escuchando

```bash
ss -lt
```

---

# Mostrar UDP escuchando

```bash
ss -lu
```

---

# Mostrar todos los puertos abiertos

```bash
ss -tuln
```

Parámetros:

| Opción | Descripción |
|---------|-------------|
| `-t` | TCP |
| `-u` | UDP |
| `-l` | Listening |
| `-n` | Mostrar números |

---

# Ejemplo

```
LISTEN

0.0.0.0:22

0.0.0.0:80

0.0.0.0:443
```

---

# Mostrar el proceso propietario

```bash
sudo ss -tulpn
```

Ejemplo:

```
users:

("sshd",pid=842)
```

---

# Buscar un puerto específico

Ejemplo SSH:

```bash
ss -tuln | grep 22
```

---

Ejemplo PostgreSQL:

```bash
ss -tuln | grep 5432
```

---

Ejemplo SQL Server:

```bash
ss -tuln | grep 1433
```

---

# Ver conexiones activas

```bash
ss -tun
```

---

# Mostrar conexiones establecidas

```bash
ss -ant
```

Buscar:

```
ESTAB
```

---

# Mostrar únicamente conexiones SSH

```bash
ss -ant | grep :22
```

---

# Ver estadísticas

```bash
ss -s
```

Ejemplo:

```
TCP:

established

closed

timewait
```

---

# Ver servicios del sistema

```bash
systemctl list-units --type=service
```

---

# Consultar un servicio

Ejemplo:

```bash
systemctl status sshd
```

---

# Iniciar un servicio

```bash
sudo systemctl start sshd
```

---

# Detener un servicio

```bash
sudo systemctl stop sshd
```

---

# Reiniciar un servicio

```bash
sudo systemctl restart sshd
```

---

# Habilitar inicio automático

```bash
sudo systemctl enable sshd
```

---

# Deshabilitar inicio automático

```bash
sudo systemctl disable sshd
```

---

# Verificar si un puerto remoto está abierto

Con Netcat:

```bash
nc -zv 192.168.1.100 22
```

---

Con Bash:

```bash
timeout 5 bash -c "</dev/tcp/192.168.1.100/22"
```

---

# Verificar procesos

```bash
ps aux
```

---

Buscar SSH:

```bash
ps aux | grep sshd
```

---

# Consultar el PID

```bash
pidof sshd
```

---

# Relación Servicio → Puerto

| Servicio | Puerto |
|-----------|---------|
| SSH | 22 |
| HTTP | 80 |
| HTTPS | 443 |
| FTP | 21 |
| DNS | 53 |
| PostgreSQL | 5432 |
| SQL Server | 1433 |
| MySQL | 3306 |
| Oracle | 1521 |
| Redis | 6379 |

---

# Buenas prácticas RHCSA

✔ Mantener abiertos únicamente los puertos necesarios.

✔ Verificar periódicamente los servicios en escucha.

✔ Utilizar `ss` en lugar de `netstat` (obsoleto).

✔ Deshabilitar servicios innecesarios.

✔ Comprobar siempre qué proceso utiliza un puerto antes de detener un servicio.

✔ Documentar los puertos utilizados por las aplicaciones.

---

# Errores comunes

## El servicio no responde

Verificar:

```bash
systemctl status servicio
```

---

## El puerto no aparece

Consultar:

```bash
ss -tuln
```

Si no aparece, probablemente el servicio no está iniciado.

---

## El puerto está bloqueado

Comprobar el Firewall:

```bash
sudo firewall-cmd --list-ports
```

---

## Puerto ocupado

Identificar el proceso:

```bash
sudo ss -tulpn
```

---

## Muchas conexiones TIME_WAIT

Consultar:

```bash
ss -s
```

Generalmente es un comportamiento normal en servidores con alta actividad.

---

# Comandos importantes RHCSA

| Comando | Descripción |
|----------|-------------|
| `ss` | Mostrar sockets |
| `ss -tuln` | Puertos abiertos |
| `ss -tulpn` | Puertos con procesos |
| `ss -ant` | Conexiones TCP |
| `ss -s` | Estadísticas |
| `systemctl status` | Estado de un servicio |
| `systemctl restart` | Reiniciar servicio |
| `pidof` | Obtener PID |
| `ps aux` | Procesos activos |
| `nc -zv` | Probar un puerto remoto |

---

# Resumen

En esta lección aprendiste a:

- Comprender el funcionamiento de los puertos de red.
- Diferenciar TCP y UDP.
- Entender el concepto de Socket.
- Utilizar la herramienta `ss`.
- Verificar qué servicios están escuchando.
- Identificar qué proceso utiliza un puerto.
- Diagnosticar problemas relacionados con puertos y servicios de red.

---

# Laboratorio práctico RHCSA

## Escenario 1

Verifica todos los puertos abiertos en el servidor.

```bash
ss -tuln
```

Identifica cuáles pertenecen a SSH, HTTP y HTTPS.

---

## Escenario 2

Consulta qué proceso utiliza el puerto **22**.

```bash
sudo ss -tulpn | grep :22
```

Obtén el nombre del proceso y su PID.

---

## Escenario 3

Comprueba el estado del servicio SSH.

```bash
systemctl status sshd
```

Reinícialo y verifica que el puerto 22 permanezca en estado **LISTEN**.

---

## Escenario 4

Verifica si el puerto **5432** (PostgreSQL) o **1433** (SQL Server) está escuchando.

```bash
ss -tuln | grep 5432
```

o

```bash
ss -tuln | grep 1433
```

---

## Escenario 5

Desde otro equipo o desde la misma máquina, comprueba que el puerto SSH está accesible utilizando `nc`.

```bash
nc -zv 127.0.0.1 22
```

> **Objetivo general:** aprender a identificar servicios, puertos y sockets en Red Hat Enterprise Linux, diagnosticar problemas de conectividad y administrar servicios de red, habilidades fundamentales para el examen **RHCSA** y para la administración de servidores Linux en producción.