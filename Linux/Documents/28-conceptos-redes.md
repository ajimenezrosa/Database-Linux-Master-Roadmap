# Capítulo 28 - Conceptos Fundamentales de Redes
## Parte 1

# Introducción

Las redes de computadoras constituyen uno de los pilares de la administración de sistemas Linux. En Red Hat Enterprise Linux (RHEL) y Fedora, comprender cómo funcionan las redes es esencial para administrar servidores, bases de datos, servicios web y aplicaciones empresariales.

Esta primera parte introduce los conceptos fundamentales que servirán de base para los siguientes capítulos.

---

# Objetivos

Al finalizar esta sección podrás:

- Comprender qué es una red.
- Diferenciar LAN, WAN, MAN y WLAN.
- Entender el modelo OSI.
- Comprender el modelo TCP/IP.
- Identificar dispositivos de red.
- Entender el proceso básico de comunicación entre equipos.

---

# ¿Qué es una red?

Una red es un conjunto de dispositivos capaces de intercambiar información mediante protocolos de comunicación.

Ejemplos:

- Dos computadoras conectadas mediante un cable Ethernet.
- Una oficina conectada mediante un switch.
- Internet.

---

# Componentes principales

- Host
- Cliente
- Servidor
- Switch
- Router
- Firewall
- Punto de acceso inalámbrico
- Medio físico o inalámbrico

---

# Tipos de redes

## LAN
Red de área local.

Ejemplo:

```text
PC ---- Switch ---- Servidor Linux
```

## MAN

Interconecta varias LAN dentro de una ciudad.

## WAN

Conecta redes a grandes distancias.

Ejemplo:

```text
Sucursal A -------- Internet -------- Sucursal B
```

## WLAN

Red inalámbrica basada en Wi‑Fi.

---

# Modelo OSI

| Capa | Nombre | Función |
|------:|---------|---------|
|7|Aplicación|Servicios al usuario|
|6|Presentación|Formato y cifrado|
|5|Sesión|Control de sesiones|
|4|Transporte|TCP y UDP|
|3|Red|IP y enrutamiento|
|2|Enlace|MAC y switches|
|1|Física|Cableado y señales|

---

# Modelo TCP/IP

| Capa | Protocolos |
|-------|------------|
|Aplicación|HTTP, HTTPS, SSH, DNS|
|Transporte|TCP, UDP|
|Internet|IPv4, IPv6, ICMP|
|Acceso a red|Ethernet, Wi‑Fi|

---

# Encapsulamiento

```text
Datos
 ↓
TCP
 ↓
IP
 ↓
Ethernet
 ↓
Bits
```

Cada capa agrega su propia información antes de transmitir el paquete.

---

# Protocolos importantes

- TCP
- UDP
- IP
- ICMP
- ARP
- DNS
- DHCP
- SSH
- HTTP
- HTTPS

---

# Primeros comandos en Linux

```bash
hostname
hostnamectl
ip address
ip link
ip route
ping 127.0.0.1
```

---

# Laboratorio 1

1. Ejecuta:

```bash
hostnamectl
```

2. Lista las interfaces:

```bash
ip link
```

3. Muestra las direcciones IP:

```bash
ip address
```

4. Comprueba conectividad local:

```bash
ping -c 4 127.0.0.1
```

---

# Buenas prácticas

- Documenta la topología de red.
- Utiliza nombres descriptivos para los servidores.
- Comprende el modelo OSI antes de configurar servicios.
- Familiarízate con `ip` en lugar de herramientas obsoletas.

---

# Resumen

En esta primera parte aprendiste los conceptos básicos de redes, los modelos OSI y TCP/IP, los componentes principales y los primeros comandos que utilizarás durante todo el módulo.

**En la Parte 2** profundizaremos en direcciones MAC, IPv4, IPv6, ARP, switches, routers y el proceso completo de comunicación entre equipos.
