# 30. Configuración con nmcli

> **Módulo 6: Redes en Red Hat Enterprise Linux**  
> **Página 30 del Manual RHCSA**

---

# Objetivos de aprendizaje

Al finalizar esta lección serás capaz de:

- Comprender qué es `nmcli`.
- Administrar conexiones desde la línea de comandos.
- Crear conexiones Ethernet.
- Configurar direcciones IP estáticas y dinámicas.
- Configurar DNS y Gateway.
- Activar y desactivar conexiones.
- Verificar el estado de la red utilizando `nmcli`.

---

# ¿Qué es nmcli?

**nmcli** (Network Manager Command Line Interface) es la herramienta de línea de comandos que permite administrar **NetworkManager** sin necesidad de utilizar una interfaz gráfica.

Es la herramienta oficial utilizada en servidores Red Hat Enterprise Linux y una de las habilidades más importantes evaluadas en el examen **RHCSA**.

Con `nmcli` es posible:

- Crear conexiones.
- Eliminar conexiones.
- Modificar configuraciones.
- Activar y desactivar interfaces.
- Configurar IPv4 e IPv6.
- Configurar DNS.
- Administrar VLAN, Bonding y Bridges.
- Consultar el estado de la red.

---

# Sintaxis básica

```bash
nmcli [OBJETO] [COMANDO]
```

Ejemplos:

```bash
nmcli device
```

```bash
nmcli connection show
```

```bash
nmcli general status
```

---

# Ver el estado general

```bash
nmcli general status
```

Ejemplo:

```
STATE      CONNECTIVITY

connected  full
```

Estados posibles:

- connected
- disconnected
- connecting
- limited

---

# Ver todas las interfaces

```bash
nmcli device
```

Ejemplo:

```
DEVICE     TYPE      STATE         CONNECTION

ens160     ethernet  connected     LAN
ens192     ethernet  disconnected  --
lo         loopback  unmanaged     --
```

---

# Mostrar todas las conexiones

```bash
nmcli connection show
```

Salida:

```
NAME
LAN
Laboratorio
Wired connection 1
```

---

# Mostrar información detallada

```bash
nmcli connection show LAN
```

Muestra información como:

- Dirección IP
- Gateway
- DNS
- Método IPv4
- Método IPv6
- MTU
- Dirección MAC
- UUID

---

# Crear una nueva conexión

Crear una conexión Ethernet:

```bash
sudo nmcli connection add \
type ethernet \
ifname ens160 \
con-name Oficina
```

Verificar:

```bash
nmcli connection show
```

---

# Activar una conexión

```bash
sudo nmcli connection up Oficina
```

Salida esperada:

```
Connection successfully activated
```

---

# Desactivar una conexión

```bash
sudo nmcli connection down Oficina
```

---

# Eliminar una conexión

```bash
sudo nmcli connection delete Oficina
```

---

# Cambiar el nombre de una conexión

```bash
sudo nmcli connection modify Oficina connection.id LAN
```

---

# Configurar una dirección IP estática

Ejemplo:

IP:

```
192.168.1.100
```

Máscara:

```
24
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

# Aplicar la configuración

```bash
sudo nmcli connection down LAN
```

Luego:

```bash
sudo nmcli connection up LAN
```

---

# Configurar DHCP

Para volver a obtener la dirección IP automáticamente:

```bash
sudo nmcli connection modify LAN ipv4.method auto
```

Aplicar:

```bash
sudo nmcli connection up LAN
```

---

# Configurar varios DNS

```bash
sudo nmcli connection modify LAN \
ipv4.dns "1.1.1.1 8.8.8.8"
```

---

# Eliminar los DNS configurados

```bash
sudo nmcli connection modify LAN ipv4.dns ""
```

---

# Cambiar únicamente el Gateway

```bash
sudo nmcli connection modify LAN \
ipv4.gateway 192.168.1.254
```

---

# Ver la dirección IP

```bash
ip addr
```

O únicamente:

```bash
hostname -I
```

---

# Ver la puerta de enlace

```bash
ip route
```

Resultado:

```
default via 192.168.1.1 dev ens160
```

---

# Ver el DNS

```bash
cat /etc/resolv.conf
```

---

# Consultar los dispositivos

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

# Mostrar información de una interfaz

```bash
nmcli device show ens160
```

Información disponible:

- Dirección IP
- Gateway
- DNS
- MTU
- Dirección MAC
- Estado
- Velocidad

---

# Desconectar una interfaz

```bash
sudo nmcli device disconnect ens160
```

---

# Conectar una interfaz

```bash
sudo nmcli device connect ens160
```

---

# Reiniciar NetworkManager

```bash
sudo systemctl restart NetworkManager
```

---

# Verificar que NetworkManager está activo

```bash
systemctl status NetworkManager
```

Debe mostrarse:

```
Active: active (running)
```

---

# Consultar el hostname

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

# Comprobar conectividad

Ping al Gateway:

```bash
ping 192.168.1.1
```

Ping a Internet:

```bash
ping 8.8.8.8
```

Probar DNS:

```bash
ping google.com
```

---

# Configuración almacenada

Todas las conexiones administradas por NetworkManager se almacenan en:

```bash
/etc/NetworkManager/system-connections/
```

Ejemplo:

```
LAN.nmconnection

Servidor.nmconnection

Laboratorio.nmconnection
```

---

# Comandos más utilizados

| Comando | Descripción |
|----------|-------------|
| `nmcli general status` | Estado general de NetworkManager |
| `nmcli device` | Lista interfaces |
| `nmcli device status` | Estado de interfaces |
| `nmcli connection show` | Lista conexiones |
| `nmcli connection up` | Activa una conexión |
| `nmcli connection down` | Desactiva una conexión |
| `nmcli connection add` | Crea una conexión |
| `nmcli connection delete` | Elimina una conexión |
| `nmcli connection modify` | Modifica una conexión |
| `nmcli device show` | Información detallada del dispositivo |

---

# Buenas prácticas RHCSA

✔ Utilizar nombres descriptivos para las conexiones.

✔ Verificar siempre el estado de la conexión antes de modificarla.

✔ Probar la conectividad después de realizar cambios.

✔ Utilizar `nmcli` en lugar de editar archivos manualmente.

✔ Documentar las configuraciones de red implementadas.

---

# Errores comunes

## Error: Device not managed

Verificar:

```bash
nmcli device
```

Si la interfaz aparece como **unmanaged**, revisar la configuración de NetworkManager.

---

## Error: Sin dirección IP

Reactivar la conexión:

```bash
nmcli connection down LAN

nmcli connection up LAN
```

---

## Error: No resuelve nombres

Verificar:

```bash
cat /etc/resolv.conf
```

Comprobar los servidores DNS configurados.

---

## Error: Sin acceso a Internet

Revisar la ruta por defecto:

```bash
ip route
```

Comprobar la puerta de enlace y la conectividad con:

```bash
ping 8.8.8.8
```

---

# Resumen

En esta lección aprendiste a:

- Comprender el funcionamiento de `nmcli`.
- Administrar conexiones de red desde la terminal.
- Crear, modificar y eliminar conexiones.
- Configurar direcciones IP estáticas y dinámicas.
- Configurar DNS y Gateway.
- Activar y desactivar conexiones.
- Verificar el estado de la red y solucionar problemas básicos.

---

# Ejercicio práctico RHCSA

1. Verifica el estado general de NetworkManager.
2. Lista todas las interfaces de red disponibles.
3. Crea una nueva conexión llamada **LAB** para la interfaz Ethernet.
4. Configúrala inicialmente para utilizar DHCP.
5. Activa la conexión y verifica que reciba una dirección IP.
6. Modifica la conexión para utilizar una IP estática (`192.168.100.50/24`).
7. Configura un Gateway y dos servidores DNS.
8. Reactiva la conexión y verifica los cambios con `ip addr` y `ip route`.
9. Comprueba la conectividad mediante `ping` al Gateway y a `google.com`.
10. Elimina la conexión **LAB**.

> **Objetivo:** dominar la herramienta `nmcli`, utilizada diariamente por administradores de sistemas Linux y evaluada en el examen **RHCSA** para la administración de redes desde la línea de comandos.