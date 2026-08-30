# 29. Interfaces de Red y NetworkManager (RHCSA)

> **Módulo 6: Redes en Red Hat Enterprise Linux**  
> **Página 29 del Manual RHCSA**

---

# Objetivos de aprendizaje

Al finalizar esta lección serás capaz de:

- Comprender qué es NetworkManager.
- Visualizar las interfaces de red disponibles.
- Administrar conexiones utilizando `nmcli`.
- Crear, modificar y eliminar conexiones.
- Configurar direcciones IP estáticas y dinámicas.
- Reiniciar conexiones sin reiniciar el servidor.
- Verificar el estado de la conectividad.

---

# ¿Qué es NetworkManager?

**NetworkManager** es el servicio encargado de administrar todas las conexiones de red en Red Hat Enterprise Linux.

Su función principal consiste en:

- Detectar interfaces de red.
- Administrar conexiones Ethernet.
- Administrar WiFi.
- Configurar direcciones IP.
- Configurar DNS.
- Administrar gateways.
- Configutar VLAN, Bonding y Bridges.
- Mantener las conexiones activas automáticamente.

En RHEL 8 y RHEL 9 prácticamente toda la administración de red se realiza mediante NetworkManager.

---

# Verificar que NetworkManager está ejecutándose

```bash
systemctl status NetworkManager
```

Salida esperada:

```
● NetworkManager.service
Loaded: loaded
Active: active (running)
```

Si estuviera detenido:

```bash
sudo systemctl start NetworkManager
```

Para iniciarlo automáticamente:

```bash
sudo systemctl enable NetworkManager
```

---

# Ver todas las interfaces

```bash
ip link
```

Ejemplo:

```
1: lo
2: ens160
3: ens192
```

Otra forma:

```bash
nmcli device
```

Resultado:

```
DEVICE     TYPE      STATE         CONNECTION

ens160     ethernet  connected     Wired connection 1
ens192     ethernet  disconnected  --
lo         loopback  unmanaged     --
```

---

# Mostrar únicamente las conexiones

```bash
nmcli connection show
```

Ejemplo:

```
NAME                  UUID
Wired connection 1    xxxxx
Lab                   xxxxx
```

---

# Mostrar una conexión específica

```bash
nmcli connection show "Wired connection 1"
```

Se visualizarán:

- IP
- Gateway
- DNS
- MTU
- Método IPv4
- Método IPv6
- Nombre de la interfaz

---

# Ver el estado de las interfaces

```bash
nmcli device status
```

Ejemplo:

```
DEVICE     TYPE       STATE

ens160     ethernet   connected
ens192     ethernet   disconnected
```

---

# Obtener información detallada

```bash
nmcli device show ens160
```

Información:

- Dirección IP
- Gateway
- DNS
- Dirección MAC
- MTU
- Estado
- Velocidad

---

# Ver la dirección IP

Con el comando tradicional:

```bash
ip addr
```

O únicamente una interfaz:

```bash
ip addr show ens160
```

---

# Crear una nueva conexión DHCP

```bash
sudo nmcli connection add \
type ethernet \
ifname ens160 \
con-name oficina
```

Esta conexión utilizará DHCP por defecto.

---

# Activar una conexión

```bash
sudo nmcli connection up oficina
```

---

# Desactivar una conexión

```bash
sudo nmcli connection down oficina
```

---

# Eliminar una conexión

```bash
sudo nmcli connection delete oficina
```

---

# Cambiar el nombre de una conexión

```bash
sudo nmcli connection modify "Wired connection 1" connection.id LAN
```

---

# Configurar IP estática

Ejemplo:

IP:

```
192.168.1.100
```

Gateway:

```
192.168.1.1
```

DNS:

```
8.8.8.8
```

Comando:

```bash
sudo nmcli connection modify LAN \
ipv4.addresses 192.168.1.100/24 \
ipv4.gateway 192.168.1.1 \
ipv4.dns "8.8.8.8" \
ipv4.method manual
```

---

# Volver a DHCP

```bash
sudo nmcli connection modify LAN ipv4.method auto
```

Reactivar:

```bash
sudo nmcli connection down LAN
sudo nmcli connection up LAN
```

---

# Cambiar únicamente el DNS

```bash
sudo nmcli connection modify LAN \
ipv4.dns "1.1.1.1 8.8.8.8"
```

Aplicar cambios:

```bash
sudo nmcli connection up LAN
```

---

# Cambiar el Gateway

```bash
sudo nmcli connection modify LAN \
ipv4.gateway 192.168.1.254
```

---

# Ver el Gateway actual

```bash
ip route
```

Resultado:

```
default via 192.168.1.1 dev ens160
```

---

# Reiniciar una conexión

```bash
nmcli connection down LAN
nmcli connection up LAN
```

---

# Reiniciar completamente NetworkManager

```bash
sudo systemctl restart NetworkManager
```

---

# Ver los perfiles almacenados

```bash
ls /etc/NetworkManager/system-connections/
```

Ejemplo:

```
LAN.nmconnection
Servidor.nmconnection
Laboratorio.nmconnection
```

Estos archivos contienen toda la configuración de la red.

---

# Mostrar la configuración IPv4

```bash
nmcli connection show LAN | grep ipv4
```

---

# Mostrar únicamente la IP

```bash
hostname -I
```

Ejemplo:

```
192.168.1.100
```

---

# Mostrar el hostname

```bash
hostnamectl
```

---

# Cambiar el hostname

```bash
sudo hostnamectl set-hostname servidor01
```

Verificar:

```bash
hostname
```

---

# Probar conectividad

Ping al gateway:

```bash
ping 192.168.1.1
```

Ping a Google:

```bash
ping 8.8.8.8
```

Probar resolución DNS:

```bash
ping google.com
```

Si responde por IP pero no por nombre, el problema es DNS.

---

# Ver las rutas

```bash
ip route
```

Ejemplo:

```
default via 192.168.1.1
192.168.1.0/24 dev ens160
```

---

# Reiniciar la interfaz sin reiniciar Linux

```bash
nmcli device disconnect ens160
```

Luego:

```bash
nmcli device connect ens160
```

---

# Buenas prácticas RHCSA

✔ Utilizar `nmcli` en lugar de editar archivos manualmente.

✔ Verificar el estado antes de realizar cambios.

✔ Comprobar la conectividad después de modificar una configuración.

✔ Documentar las IP estáticas utilizadas.

✔ Mantener nombres descriptivos para las conexiones.

---

# Errores comunes

### "Device not managed"

NetworkManager no administra esa interfaz.

Verificar:

```bash
nmcli device
```

---

### No obtiene IP

Renovar la conexión:

```bash
nmcli connection down LAN
nmcli connection up LAN
```

---

### No resuelve nombres

Comprobar DNS:

```bash
cat /etc/resolv.conf
```

---

### Sin acceso a Internet

Revisar:

```bash
ip route
```

Verificar gateway y conectividad:

```bash
ping 8.8.8.8
```

---

# Resumen

En esta lección aprendiste a:

- Comprender el funcionamiento de NetworkManager.
- Visualizar interfaces y conexiones de red.
- Administrar conexiones mediante `nmcli`.
- Configurar direcciones IP estáticas y dinámicas.
- Modificar DNS y gateway.
- Reiniciar conexiones sin reiniciar el servidor.
- Diagnosticar problemas básicos de conectividad.

---

# Ejercicio práctico RHCSA

En una máquina virtual:

1. Identifica el nombre de tu interfaz de red con `nmcli device`.
2. Crea una nueva conexión llamada **LAB**.
3. Configúrala para obtener IP por DHCP.
4. Activa la conexión y verifica que reciba una dirección IP.
5. Cambia la configuración a una IP estática (por ejemplo, `192.168.100.50/24`).
6. Configura el gateway y un servidor DNS.
7. Reactiva la conexión y verifica los cambios con `ip addr` y `ip route`.
8. Haz `ping` al gateway y luego a `google.com`.
9. Restaura la conexión para utilizar DHCP nuevamente.
10. Elimina la conexión **LAB**.

> **Objetivo:** dominar la administración de interfaces y conexiones con `nmcli`, una habilidad fundamental para el examen RHCSA.