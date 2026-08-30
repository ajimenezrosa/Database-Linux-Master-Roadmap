# 34. Diagnóstico y Solución de Problemas de Red

> **Módulo 6: Redes en Red Hat Enterprise Linux**  
> **Página 34 del Manual RHCSA**

---

# Objetivos de aprendizaje

Al finalizar esta lección serás capaz de:

- Diagnosticar problemas de conectividad en Linux.
- Verificar el estado de las interfaces de red.
- Comprobar la configuración IP.
- Analizar la tabla de rutas.
- Validar la resolución DNS.
- Utilizar las principales herramientas de diagnóstico de red.
- Aplicar una metodología para solucionar problemas de red en servidores Red Hat.

---

# Introducción

Uno de los trabajos más importantes de un administrador Linux consiste en **diagnosticar problemas de red**.

Cuando un servidor pierde conectividad, el problema puede encontrarse en diferentes niveles:

- Interfaz de red
- Dirección IP
- Gateway
- DNS
- Firewall
- Rutas
- Cableado o Switch
- Servicios de red

La clave consiste en seguir un proceso ordenado de diagnóstico.

---

# Paso 1: Verificar la interfaz de red

Mostrar todas las interfaces:

```bash
ip link
```

Ejemplo:

```
1: lo
2: ens160
3: ens192
```

---

# Verificar si la interfaz está activa

```bash
ip addr
```

Ejemplo:

```
ens160:
state UP
```

Si aparece:

```
DOWN
```

La interfaz está deshabilitada.

---

# Activar una interfaz

Temporalmente:

```bash
sudo ip link set ens160 up
```

Con NetworkManager:

```bash
sudo nmcli device connect ens160
```

---

# Paso 2: Verificar la dirección IP

```bash
ip addr
```

O solamente una interfaz:

```bash
ip addr show ens160
```

Verificar:

- Dirección IPv4
- Dirección IPv6
- Máscara
- Estado

---

# Mostrar únicamente la IP

```bash
hostname -I
```

---

# Paso 3: Verificar NetworkManager

```bash
systemctl status NetworkManager
```

Debe indicar:

```
Active: active (running)
```

Si no está iniciado:

```bash
sudo systemctl restart NetworkManager
```

---

# Paso 4: Verificar la configuración

```bash
nmcli device show
```

Información disponible:

- Dirección IP
- Gateway
- DNS
- MTU
- Dirección MAC

---

# Paso 5: Verificar la puerta de enlace

```bash
ip route
```

Debe existir una ruta similar a:

```
default via 192.168.1.1
```

---

# Probar el Gateway

```bash
ping 192.168.1.1
```

Si responde:

```
64 bytes from...
```

La comunicación local funciona.

---

# Paso 6: Verificar conectividad externa

Probar una IP pública:

```bash
ping 8.8.8.8
```

Si responde correctamente:

```
Internet disponible
```

---

# Paso 7: Verificar DNS

Intentar resolver un nombre:

```bash
ping google.com
```

Si responde:

```
DNS correcto
```

Si responde únicamente por IP pero no por nombre, el problema es el DNS.

---

# Consultar DNS configurados

```bash
cat /etc/resolv.conf
```

---

# Consultar DNS mediante nmcli

```bash
nmcli device show
```

Buscar:

```
IP4.DNS
```

---

# Resolver nombres manualmente

```bash
getent hosts google.com
```

---

# Utilizar host

Si está instalado:

```bash
host google.com
```

---

# Utilizar dig

Si está instalado:

```bash
dig google.com
```

---

# Paso 8: Verificar las rutas

```bash
ip route
```

Comprobar:

- Ruta por defecto
- Redes configuradas
- Gateway

---

# Paso 9: Verificar la interfaz física

Mostrar información:

```bash
ip link
```

También:

```bash
nmcli device status
```

---

# Paso 10: Verificar conectividad por saltos

Si está instalado:

```bash
traceroute google.com
```

Permite conocer dónde se pierde la comunicación.

---

# Verificar el hostname

```bash
hostnamectl
```

---

# Verificar resolución local

```bash
cat /etc/hosts
```

---

# Mostrar conexiones activas

```bash
nmcli connection show
```

---

# Reiniciar una conexión

```bash
sudo nmcli connection down LAN

sudo nmcli connection up LAN
```

---

# Reiniciar NetworkManager

```bash
sudo systemctl restart NetworkManager
```

---

# Comprobar puertos abiertos

```bash
ss -tuln
```

Ejemplo:

```
tcp LISTEN
```

---

# Ver conexiones establecidas

```bash
ss -tun
```

---

# Ver estadísticas de red

```bash
ip -s link
```

Información:

- Paquetes enviados
- Paquetes recibidos
- Errores
- Paquetes descartados

---

# Ver la tabla ARP

```bash
ip neigh
```

Ejemplo:

```
192.168.1.1 dev ens160 lladdr ...
```

---

# Probar un puerto remoto

Con Netcat (si está instalado):

```bash
nc -zv google.com 443
```

O mediante Bash:

```bash
timeout 5 bash -c "</dev/tcp/google.com/443" && echo "Puerto abierto"
```

---

# Comprobar el Firewall

En RHEL:

```bash
sudo firewall-cmd --state
```

Debe responder:

```
running
```

---

# Ver zonas del Firewall

```bash
sudo firewall-cmd --get-active-zones
```

---

# Ver servicios permitidos

```bash
sudo firewall-cmd --list-services
```

---

# Metodología de diagnóstico

```
1. ¿La interfaz está UP?

↓

2. ¿Tiene dirección IP?

↓

3. ¿Existe Gateway?

↓

4. ¿Responde el Gateway?

↓

5. ¿Responde Internet por IP?

↓

6. ¿Resuelve nombres DNS?

↓

7. ¿Existen rutas?

↓

8. ¿Existe algún bloqueo por Firewall?
```

---

# Herramientas principales

| Herramienta | Uso |
|-------------|-----|
| `ip addr` | Ver direcciones IP |
| `ip link` | Ver interfaces |
| `ip route` | Ver rutas |
| `ip neigh` | Tabla ARP |
| `nmcli` | Administrar red |
| `ping` | Verificar conectividad |
| `traceroute` | Analizar el recorrido de paquetes |
| `host` | Consultar DNS |
| `dig` | Diagnóstico DNS |
| `getent hosts` | Resolver nombres |
| `ss` | Ver conexiones y puertos |
| `hostnamectl` | Ver hostname |
| `firewall-cmd` | Administrar Firewall |

---

# Buenas prácticas RHCSA

✔ Diagnosticar de lo más simple a lo más complejo.

✔ Verificar primero la interfaz de red.

✔ Confirmar la dirección IP antes de revisar DNS.

✔ Comprobar la puerta de enlace antes de probar Internet.

✔ Verificar primero por IP y luego por nombre.

✔ No modificar varias configuraciones al mismo tiempo.

✔ Documentar los cambios realizados.

---

# Errores comunes

## La interfaz aparece DOWN

Activarla:

```bash
sudo ip link set ens160 up
```

---

## No tiene dirección IP

Comprobar DHCP o la configuración estática.

```bash
ip addr
```

---

## No existe Gateway

Verificar:

```bash
ip route
```

---

## DNS incorrecto

Consultar:

```bash
cat /etc/resolv.conf
```

---

## Firewall bloqueando tráfico

Verificar:

```bash
sudo firewall-cmd --list-all
```

---

## No existe conexión a Internet

Realizar las pruebas en este orden:

```bash
ping Gateway
```

↓

```bash
ping 8.8.8.8
```

↓

```bash
ping google.com
```

---

# Resumen

En esta lección aprendiste a:

- Diagnosticar problemas de red de forma estructurada.
- Verificar interfaces, direcciones IP y rutas.
- Analizar la resolución DNS.
- Comprobar puertos y conexiones.
- Validar el estado del Firewall.
- Aplicar una metodología profesional para solucionar problemas de conectividad en Linux.

---

# Laboratorio práctico RHCSA

## Escenario 1

La interfaz de red aparece como **DOWN**.

**Objetivo:**

- Identificar el problema.
- Activar la interfaz.
- Verificar la conectividad.

---

## Escenario 2

El servidor tiene dirección IP, pero no puede acceder a Internet.

**Objetivo:**

- Verificar la ruta por defecto.
- Probar el Gateway.
- Confirmar la conectividad hacia `8.8.8.8`.

---

## Escenario 3

El servidor responde a `8.8.8.8`, pero no a `google.com`.

**Objetivo:**

- Revisar `/etc/resolv.conf`.
- Verificar los servidores DNS configurados.
- Resolver nombres utilizando `getent`, `host` o `dig`.

---

## Escenario 4

Una aplicación no responde desde otro equipo.

**Objetivo:**

- Verificar que el servicio esté escuchando con `ss -tuln`.
- Confirmar que el Firewall permita el puerto correspondiente.
- Probar la conexión remota con `nc` o `telnet` (si está disponible).

---

## Escenario 5

Un servidor pierde comunicación con una red remota.

**Objetivo:**

- Revisar la tabla de rutas.
- Verificar las rutas estáticas configuradas.
- Utilizar `traceroute` para identificar el punto donde se interrumpe la comunicación.

> **Objetivo general:** desarrollar una metodología sistemática para identificar y resolver problemas de red en Red Hat Enterprise Linux, una habilidad esencial para el examen **RHCSA** y para la administración de servidores en entornos de producción.