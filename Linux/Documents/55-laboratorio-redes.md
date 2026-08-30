# 55. Laboratorio de Administración de Redes en Linux — Fase 1

> **Módulo 6 — Redes en Red Hat**
>
> **Archivo:** `55-laboratorio-redes.md`
>
> **Nivel:** RHCSA
>
> **Objetivo general:** Configurar, verificar y solucionar problemas básicos de red en sistemas Red Hat Enterprise Linux, Rocky Linux, AlmaLinux y Fedora utilizando NetworkManager, `nmcli`, `ip`, `ss`, `ping`, `hostnamectl`, DNS y rutas estáticas.

---

# 1. Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Comprender los fundamentos de las redes TCP/IP.
- Identificar interfaces de red.
- Consultar direcciones IPv4 e IPv6.
- Interpretar máscaras de red y prefijos CIDR.
- Identificar la puerta de enlace predeterminada.
- Consultar y configurar servidores DNS.
- Utilizar NetworkManager.
- Administrar conexiones mediante `nmcli`.
- Crear conexiones Ethernet estáticas.
- Configurar conexiones mediante DHCP.
- Activar y desactivar conexiones.
- Cambiar el nombre del sistema.
- Configurar rutas estáticas.
- Verificar conectividad local y remota.
- Identificar puertos abiertos.
- Consultar sockets y servicios.
- Resolver problemas frecuentes de conectividad.
- Configurar conexiones persistentes.
- Aplicar procedimientos similares a los evaluados en RHCSA.

---

# 2. Introducción

La red permite que los sistemas Linux puedan comunicarse con:

- Otros servidores.
- Estaciones de trabajo.
- Bases de datos.
- Aplicaciones.
- Servicios en la nube.
- Repositorios de paquetes.
- Sistemas de monitoreo.
- Dispositivos de almacenamiento.
- Internet.

Un administrador Linux debe poder determinar rápidamente:

```text
¿La interfaz está activa?

¿El sistema tiene una dirección IP?

¿La máscara es correcta?

¿Existe una puerta de enlace?

¿El DNS funciona?

¿El destino responde?

¿El puerto está abierto?

¿El servicio está escuchando?

¿El firewall permite la conexión?
```

La administración de redes en sistemas modernos de la familia Red Hat se realiza principalmente mediante:

```text
NetworkManager
```

y su herramienta de línea de comandos:

```text
nmcli
```

---

# 3. Escenario empresarial

La empresa **TechData Solutions** ha instalado un nuevo servidor Linux.

El servidor debe configurarse con los siguientes parámetros:

| Parámetro | Valor |
|---|---|
| Nombre del servidor | `server01.techdata.local` |
| Interfaz | `ens192` |
| Dirección IPv4 | `192.168.50.20` |
| Prefijo | `/24` |
| Puerta de enlace | `192.168.50.1` |
| DNS primario | `192.168.50.10` |
| DNS secundario | `8.8.8.8` |
| Dominio de búsqueda | `techdata.local` |
| Ruta adicional | `10.20.0.0/16 vía 192.168.50.254` |

El administrador debe configurar la red de forma persistente y validar la conectividad.

---

# 4. Arquitectura de red del laboratorio

```text
                         Internet

                            │

                            ▼

                    Router / Firewall

                       192.168.50.1

                            │

           ┌────────────────┼────────────────┐

           │                │                │

           ▼                ▼                ▼

       DNS Server        Server01         Server02

     192.168.50.10    192.168.50.20    192.168.50.21

                            │

                            ▼

                   Red 192.168.50.0/24

                            │

                            ▼

                      Router interno

                     192.168.50.254

                            │

                            ▼

                     Red 10.20.0.0/16
```

---

# 5. Conceptos fundamentales

## 5.1 Dirección IP

Una dirección IP identifica un dispositivo dentro de una red.

Ejemplo IPv4:

```text
192.168.50.20
```

Ejemplo IPv6:

```text
2001:db8:50::20
```

---

## 5.2 Máscara de red

La máscara permite diferenciar:

- Parte de red.
- Parte de host.

Ejemplo:

```text
255.255.255.0
```

Su representación CIDR es:

```text
/24
```

Por tanto:

```text
192.168.50.20/24
```

significa que los primeros 24 bits identifican la red.

---

## 5.3 Puerta de enlace

La puerta de enlace predeterminada permite comunicarse con redes externas.

Ejemplo:

```text
192.168.50.1
```

Sin una puerta de enlace, el sistema normalmente podrá comunicarse únicamente con dispositivos dentro de su misma red local.

---

## 5.4 DNS

DNS traduce nombres a direcciones IP.

Ejemplo:

```text
server01.techdata.local
```

puede resolverse como:

```text
192.168.50.20
```

---

## 5.5 Dirección de red

Para:

```text
192.168.50.20/24
```

la dirección de red es:

```text
192.168.50.0
```

---

## 5.6 Dirección de broadcast

Para:

```text
192.168.50.20/24
```

la dirección de broadcast es:

```text
192.168.50.255
```

---

## 5.7 Rango de hosts

```text
192.168.50.1
```

hasta:

```text
192.168.50.254
```

---

# 6. Tabla CIDR frecuente

| Prefijo | Máscara | Hosts utilizables aproximados |
|---:|---|---:|
| `/8` | `255.0.0.0` | 16,777,214 |
| `/16` | `255.255.0.0` | 65,534 |
| `/24` | `255.255.255.0` | 254 |
| `/25` | `255.255.255.128` | 126 |
| `/26` | `255.255.255.192` | 62 |
| `/27` | `255.255.255.224` | 30 |
| `/28` | `255.255.255.240` | 14 |
| `/29` | `255.255.255.248` | 6 |
| `/30` | `255.255.255.252` | 2 |
| `/32` | `255.255.255.255` | 1 dirección |

---

# 7. Componentes de red en Red Hat

```text
Aplicaciones

↓

Resolución DNS

↓

TCP / UDP

↓

IPv4 / IPv6

↓

NetworkManager

↓

Interfaz de red

↓

Controlador

↓

Tarjeta física o virtual
```

---

# 8. NetworkManager

NetworkManager es el servicio encargado de administrar conexiones de red.

Puede controlar:

- Ethernet.
- Wi-Fi.
- VLAN.
- Bonding.
- Teaming.
- Bridges.
- VPN.
- Direcciones IPv4.
- Direcciones IPv6.
- DNS.
- Rutas.
- Puertas de enlace.

---

# 9. Verificar el servicio NetworkManager

```bash
systemctl status NetworkManager
```

Verificar si está habilitado:

```bash
systemctl is-enabled NetworkManager
```

Verificar si está activo:

```bash
systemctl is-active NetworkManager
```

Resultado esperado:

```text
active
```

---

# 10. Iniciar y habilitar NetworkManager

```bash
sudo systemctl enable --now NetworkManager
```

Verificar:

```bash
systemctl status NetworkManager
```

---

# 11. Diferencia entre dispositivo y conexión

Este concepto es fundamental.

## Dispositivo

Es la interfaz real o virtual.

Ejemplos:

```text
ens192
enp1s0
eth0
wlp3s0
```

## Conexión

Es un perfil de configuración administrado por NetworkManager.

Ejemplos:

```text
Wired connection 1
conexion-servidor
red-produccion
```

Diagrama:

```text
Perfil de conexión

        │

        ▼

    NetworkManager

        │

        ▼

Dispositivo ens192
```

Una misma interfaz puede tener varios perfiles, aunque normalmente sólo uno estará activo a la vez.

---

# 12. Identificar interfaces

```bash
ip link show
```

Versión corta:

```bash
ip link
```

Ejemplo:

```text
1: lo: <LOOPBACK,UP,LOWER_UP>
2: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP>
```

---

# 13. Interpretar estados de interfaz

| Estado | Significado |
|---|---|
| `UP` | La interfaz está habilitada |
| `DOWN` | La interfaz está deshabilitada |
| `LOWER_UP` | Existe enlace físico |
| `NO-CARRIER` | No se detecta enlace |
| `LOOPBACK` | Interfaz local |
| `BROADCAST` | Soporta broadcast |
| `MULTICAST` | Soporta multicast |

---

# 14. Consultar direcciones IP

```bash
ip address show
```

Versión corta:

```bash
ip addr
```

Aún más corta:

```bash
ip a
```

Consultar una interfaz específica:

```bash
ip addr show ens192
```

---

# 15. Ejemplo de salida

```text
2: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500
    inet 192.168.50.20/24 brd 192.168.50.255 scope global ens192
    inet6 fe80::20c:29ff:fe12:3456/64 scope link
```

Interpretación:

| Campo | Significado |
|---|---|
| `inet` | Dirección IPv4 |
| `192.168.50.20/24` | Dirección y prefijo |
| `brd` | Broadcast |
| `scope global` | Dirección válida globalmente en la red |
| `inet6` | Dirección IPv6 |
| `fe80::/64` | Dirección IPv6 link-local |
| `mtu 1500` | Tamaño máximo de trama |

---

# 16. Mostrar únicamente IPv4

```bash
ip -4 addr show
```

Para una interfaz:

```bash
ip -4 addr show dev ens192
```

---

# 17. Mostrar únicamente IPv6

```bash
ip -6 addr show
```

---

# 18. Identificar nombres de interfaces con nmcli

```bash
nmcli device status
```

Ejemplo:

```text
DEVICE   TYPE      STATE                   CONNECTION
ens192   ethernet  connected               red-produccion
lo       loopback  connected (externally)  lo
ens224   ethernet  disconnected            --
```

---

# 19. Consultar información detallada de dispositivos

```bash
nmcli device show
```

Para una interfaz específica:

```bash
nmcli device show ens192
```

Campos importantes:

```text
GENERAL.DEVICE
GENERAL.TYPE
GENERAL.STATE
GENERAL.CONNECTION
IP4.ADDRESS
IP4.GATEWAY
IP4.DNS
IP4.ROUTE
IP6.ADDRESS
```

---

# 20. Mostrar conexiones existentes

```bash
nmcli connection show
```

Versión corta:

```bash
nmcli con show
```

Ejemplo:

```text
NAME             UUID                                  TYPE      DEVICE
red-produccion   3fdb9ad5-....                         ethernet  ens192
```

---

# 21. Mostrar conexiones activas

```bash
nmcli connection show --active
```

Versión corta:

```bash
nmcli con show --active
```

---

# 22. Consultar un perfil específico

```bash
nmcli connection show red-produccion
```

Versión corta:

```bash
nmcli con show red-produccion
```

---

# 23. Identificar el nombre real de la interfaz

En sistemas modernos, los nombres predecibles pueden ser:

```text
ens160
ens192
enp0s3
enp1s0
eno1
wlp3s0
```

Nunca asumas que la interfaz se llama:

```text
eth0
```

Verifica siempre:

```bash
nmcli device status
```

o:

```bash
ip link
```

---

# 24. Nombres predecibles de interfaces

| Prefijo | Significado frecuente |
|---|---|
| `eno` | Ethernet integrada |
| `ens` | Ethernet basada en slot |
| `enp` | Ethernet basada en bus PCI |
| `enx` | Ethernet basada en MAC |
| `wlo` | Wi-Fi integrada |
| `wlp` | Wi-Fi basada en PCI |

---

# 25. Consultar dirección MAC

```bash
ip link show ens192
```

Ejemplo:

```text
link/ether 00:0c:29:12:34:56
```

También:

```bash
cat /sys/class/net/ens192/address
```

---

# 26. Consultar velocidad y enlace

Instalar `ethtool`:

```bash
sudo dnf install -y ethtool
```

Consultar:

```bash
sudo ethtool ens192
```

Campos importantes:

```text
Speed
Duplex
Auto-negotiation
Link detected
```

Ejemplo:

```text
Speed: 1000Mb/s
Duplex: Full
Link detected: yes
```

---

# 27. Configuración DHCP

DHCP asigna automáticamente:

- Dirección IP.
- Máscara.
- Puerta de enlace.
- DNS.
- Tiempo de concesión.

Flujo:

```text
Cliente

↓

DHCP Discover

↓

DHCP Offer

↓

DHCP Request

↓

DHCP Acknowledge
```

---

# 28. Crear conexión DHCP con nmcli

Supongamos que la interfaz es:

```text
ens192
```

Crear conexión:

```bash
sudo nmcli connection add \
  type ethernet \
  ifname ens192 \
  con-name red-dhcp \
  ipv4.method auto
```

Activar:

```bash
sudo nmcli connection up red-dhcp
```

Verificar:

```bash
nmcli device show ens192
```

---

# 29. Modificar una conexión existente para DHCP

```bash
sudo nmcli connection modify red-produccion \
  ipv4.method auto
```

Eliminar dirección estática anterior:

```bash
sudo nmcli connection modify red-produccion \
  ipv4.addresses ""
```

Eliminar gateway anterior:

```bash
sudo nmcli connection modify red-produccion \
  ipv4.gateway ""
```

Reactivar:

```bash
sudo nmcli connection down red-produccion
sudo nmcli connection up red-produccion
```

---

# 30. Riesgo de perder conexión remota

Si estás conectado por SSH y ejecutas:

```bash
nmcli connection down red-produccion
```

puedes perder inmediatamente el acceso.

Antes de modificar una conexión remota:

```text
□ Confirmar acceso por consola

□ Verificar dirección correcta

□ Confirmar gateway

□ Confirmar DNS

□ Mantener una segunda sesión

□ Programar reversión si es necesario

□ Probar primero en laboratorio
```

---

# 31. Crear conexión IPv4 estática

Parámetros:

```text
Interfaz: ens192
Conexión: red-produccion
IP: 192.168.50.20/24
Gateway: 192.168.50.1
DNS: 192.168.50.10 y 8.8.8.8
```

Comando:

```bash
sudo nmcli connection add \
  type ethernet \
  ifname ens192 \
  con-name red-produccion \
  ipv4.method manual \
  ipv4.addresses 192.168.50.20/24 \
  ipv4.gateway 192.168.50.1 \
  ipv4.dns "192.168.50.10 8.8.8.8"
```

---

# 32. Configurar dominio de búsqueda

```bash
sudo nmcli connection modify red-produccion \
  ipv4.dns-search "techdata.local"
```

---

# 33. Deshabilitar IPv6 de un perfil

Cuando el entorno no utiliza IPv6:

```bash
sudo nmcli connection modify red-produccion \
  ipv6.method disabled
```

En muchos entornos es mejor dejar IPv6 habilitado, salvo que exista una política específica que indique lo contrario.

---

# 34. Activar la conexión estática

```bash
sudo nmcli connection up red-produccion
```

---

# 35. Verificar parámetros aplicados

```bash
nmcli connection show red-produccion
```

Consultar únicamente propiedades relevantes:

```bash
nmcli -f \
connection.id,connection.interface-name,ipv4.method,ipv4.addresses,ipv4.gateway,ipv4.dns \
connection show red-produccion
```

---

# 36. Verificar dirección IP

```bash
ip addr show ens192
```

Resultado esperado:

```text
inet 192.168.50.20/24
```

---

# 37. Verificar la puerta de enlace

```bash
ip route
```

Resultado esperado:

```text
default via 192.168.50.1 dev ens192
192.168.50.0/24 dev ens192 proto kernel scope link src 192.168.50.20
```

---

# 38. Interpretar tabla de rutas

```text
default via 192.168.50.1 dev ens192
```

Significa:

```text
Para cualquier red no conocida

↓

Enviar a 192.168.50.1

↓

Utilizando ens192
```

---

# 39. Consultar ruta hacia un destino

```bash
ip route get 8.8.8.8
```

Ejemplo:

```text
8.8.8.8 via 192.168.50.1 dev ens192 src 192.168.50.20
```

---

# 40. Modificar una conexión existente

Supongamos que ya existe:

```text
Wired connection 1
```

Cambiarla a estática:

```bash
sudo nmcli connection modify "Wired connection 1" \
  ipv4.method manual \
  ipv4.addresses 192.168.50.20/24 \
  ipv4.gateway 192.168.50.1 \
  ipv4.dns "192.168.50.10 8.8.8.8" \
  ipv4.dns-search "techdata.local"
```

Reactivar:

```bash
sudo nmcli connection up "Wired connection 1"
```

---

# 41. Cambiar nombre de una conexión

```bash
sudo nmcli connection modify "Wired connection 1" \
  connection.id red-produccion
```

Verificar:

```bash
nmcli con show
```

---

# 42. Configurar autoconexión

```bash
sudo nmcli connection modify red-produccion \
  connection.autoconnect yes
```

Verificar:

```bash
nmcli -f connection.id,connection.autoconnect \
connection show red-produccion
```

---

# 43. Configurar prioridad de autoconexión

Cuando existen varios perfiles:

```bash
sudo nmcli connection modify red-produccion \
  connection.autoconnect-priority 100
```

Una prioridad mayor hace que NetworkManager prefiera esa conexión.

---

# 44. Desactivar una conexión

```bash
sudo nmcli connection down red-produccion
```

---

# 45. Activar una conexión

```bash
sudo nmcli connection up red-produccion
```

---

# 46. Desconectar un dispositivo

```bash
sudo nmcli device disconnect ens192
```

---

# 47. Conectar un dispositivo

```bash
sudo nmcli device connect ens192
```

---

# 48. Eliminar una conexión

```bash
sudo nmcli connection delete red-dhcp
```

Verificar:

```bash
nmcli connection show
```

---

# 49. Recargar perfiles de NetworkManager

```bash
sudo nmcli connection reload
```

También puede utilizarse:

```bash
sudo systemctl reload NetworkManager
```

No siempre es necesario reiniciar el servicio.

---

# 50. Reiniciar NetworkManager

```bash
sudo systemctl restart NetworkManager
```

Advertencia:

```text
Reiniciar NetworkManager puede interrumpir conexiones activas.
```

En servidores remotos debe realizarse con precaución.

---

# 51. Archivos de perfiles de NetworkManager

En sistemas modernos, los perfiles suelen almacenarse en:

```text
/etc/NetworkManager/system-connections/
```

Consultar:

```bash
sudo ls -l /etc/NetworkManager/system-connections/
```

Los archivos normalmente tienen permisos:

```text
600
```

---

# 52. Ejemplo de perfil keyfile

```ini
[connection]
id=red-produccion
type=ethernet
interface-name=ens192
autoconnect=true

[ipv4]
method=manual
address1=192.168.50.20/24,192.168.50.1
dns=192.168.50.10;8.8.8.8;
dns-search=techdata.local;

[ipv6]
method=disabled
```

Aunque pueden editarse manualmente, se recomienda utilizar:

```text
nmcli
```

para reducir errores.

---

# 53. Permisos de perfiles

Verificar:

```bash
sudo stat /etc/NetworkManager/system-connections/*
```

Los perfiles pueden contener información sensible, como credenciales de Wi-Fi o VPN.

Por eso deben estar protegidos.

---

# 54. Configurar hostname

Consultar el nombre actual:

```bash
hostname
```

Consultar información completa:

```bash
hostnamectl
```

---

# 55. Establecer hostname

```bash
sudo hostnamectl set-hostname server01.techdata.local
```

Verificar:

```bash
hostnamectl
```

También:

```bash
hostname
```

---

# 56. Nombre corto y FQDN

Nombre corto:

```text
server01
```

FQDN:

```text
server01.techdata.local
```

Descomposición:

```text
server01         techdata.local
   │                    │
   ▼                    ▼
 hostname              dominio
```

---

# 57. Archivo `/etc/hostname`

Consultar:

```bash
cat /etc/hostname
```

Contenido esperado:

```text
server01.techdata.local
```

---

# 58. Archivo `/etc/hosts`

Ejemplo:

```text
127.0.0.1   localhost localhost.localdomain
::1         localhost localhost.localdomain

192.168.50.20 server01.techdata.local server01
192.168.50.21 server02.techdata.local server02
```

Editar:

```bash
sudo vi /etc/hosts
```

---

# 59. Cuándo usar `/etc/hosts`

Puede utilizarse para:

- Laboratorios.
- Resolución temporal.
- Servidores sin DNS.
- Entradas críticas.
- Pruebas.

No debe sustituir una infraestructura DNS adecuada en entornos grandes.

---

# 60. Orden de resolución de nombres

Consultar:

```bash
grep '^hosts:' /etc/nsswitch.conf
```

Ejemplo:

```text
hosts: files dns myhostname
```

Esto significa:

```text
1. Revisar /etc/hosts

2. Consultar DNS

3. Utilizar resolución local del hostname
```

---

# 61. Consultar configuración DNS

```bash
cat /etc/resolv.conf
```

Ejemplo:

```text
search techdata.local
nameserver 192.168.50.10
nameserver 8.8.8.8
```

En sistemas con NetworkManager, este archivo suele generarse automáticamente.

No es recomendable editarlo directamente.

---

# 62. Consultar DNS mediante nmcli

```bash
nmcli device show ens192 | grep DNS
```

También:

```bash
nmcli -f IP4.DNS device show ens192
```

---

# 63. Ignorar DNS recibido por DHCP

Cuando una conexión utiliza DHCP, pero deseas DNS manual:

```bash
sudo nmcli connection modify red-dhcp \
  ipv4.ignore-auto-dns yes \
  ipv4.dns "192.168.50.10 8.8.8.8"
```

Reactivar:

```bash
sudo nmcli connection up red-dhcp
```

---

# 64. Restablecer DNS automático

```bash
sudo nmcli connection modify red-dhcp \
  ipv4.ignore-auto-dns no \
  ipv4.dns ""
```

---

# 65. Consultar resolución de nombres

Con `getent`:

```bash
getent hosts server01.techdata.local
```

Consultar un dominio público:

```bash
getent hosts example.com
```

---

# 66. Utilizar host

Instalar herramientas DNS:

```bash
sudo dnf install -y bind-utils
```

Consultar:

```bash
host example.com
```

---

# 67. Utilizar dig

```bash
dig example.com
```

Consulta corta:

```bash
dig +short example.com
```

Consultar servidor específico:

```bash
dig @8.8.8.8 example.com
```

Consultar un registro A:

```bash
dig example.com A
```

Consultar un registro MX:

```bash
dig example.com MX
```

---

# 68. Utilizar nslookup

```bash
nslookup example.com
```

Aunque continúa disponible, `dig` suele ofrecer información más detallada.

---

# 69. Diferencia entre conectividad IP y DNS

Prueba IP:

```bash
ping -c 4 8.8.8.8
```

Prueba DNS:

```bash
ping -c 4 example.com
```

Interpretación:

| Resultado | Diagnóstico probable |
|---|---|
| IP funciona y nombre falla | Problema DNS |
| IP falla y nombre falla | Problema de red, ruta o firewall |
| Gateway responde, Internet no | Problema de salida o router |
| Localhost responde, gateway no | Problema local o de segmento |

---

# 70. Probar interfaz loopback

```bash
ping -c 4 127.0.0.1
```

También:

```bash
ping -c 4 localhost
```

Si falla, existe un problema grave en la pila TCP/IP local.

---

# 71. Probar dirección propia

```bash
ping -c 4 192.168.50.20
```

---

# 72. Probar puerta de enlace

```bash
ping -c 4 192.168.50.1
```

---

# 73. Probar destino externo

```bash
ping -c 4 8.8.8.8
```

---

# 74. Probar resolución DNS

```bash
ping -c 4 example.com
```

---

# 75. Metodología de pruebas

```text
1. Loopback

↓

2. Dirección propia

↓

3. Otro host local

↓

4. Gateway

↓

5. IP externa

↓

6. Nombre DNS

↓

7. Puerto de aplicación
```

---

# 76. Ping puede estar bloqueado

Que un servidor no responda a `ping` no significa necesariamente que esté caído.

ICMP puede estar bloqueado por:

- Firewall.
- Router.
- Política de seguridad.
- Proveedor.
- Configuración del destino.

Por eso deben probarse también puertos específicos.

---

# 77. Probar un puerto con nc

Instalar:

```bash
sudo dnf install -y nmap-ncat
```

Probar SSH:

```bash
nc -vz 192.168.50.21 22
```

Probar HTTP:

```bash
nc -vz 192.168.50.21 80
```

Probar PostgreSQL:

```bash
nc -vz 192.168.50.31 5432
```

---

# 78. Probar un servicio con curl

```bash
curl http://192.168.50.21
```

Ver encabezados:

```bash
curl -I http://192.168.50.21
```

Modo detallado:

```bash
curl -v http://192.168.50.21
```

---

# 79. Consultar sockets con ss

Mostrar sockets escuchando:

```bash
ss -tuln
```

Significado:

| Opción | Función |
|---|---|
| `-t` | TCP |
| `-u` | UDP |
| `-l` | Listening |
| `-n` | No resolver nombres |

---

# 80. Mostrar procesos asociados

```bash
sudo ss -tulpn
```

Ejemplo:

```text
LISTEN 0 128 0.0.0.0:22 0.0.0.0:* users:(("sshd",pid=1020,fd=3))
```

---

# 81. Consultar conexiones TCP activas

```bash
ss -tn
```

---

# 82. Consultar estadísticas

```bash
ss -s
```

Ejemplo:

```text
Total: 150
TCP: 20
UDP: 8
```

---

# 83. Identificar proceso que usa un puerto

```bash
sudo ss -tulpn | grep ':22'
```

También:

```bash
sudo lsof -i :22
```

Instalar `lsof` si es necesario:

```bash
sudo dnf install -y lsof
```

---

# 84. Rutas estáticas

Una ruta estática indica cómo llegar a una red específica.

Ejemplo:

```text
Destino: 10.20.0.0/16
Gateway: 192.168.50.254
Interfaz: ens192
```

Diagrama:

```text
Server01

192.168.50.20

     │

     ▼

192.168.50.254

     │

     ▼

10.20.0.0/16
```

---

# 85. Agregar ruta temporal

```bash
sudo ip route add \
  10.20.0.0/16 \
  via 192.168.50.254 \
  dev ens192
```

Verificar:

```bash
ip route
```

Esta ruta se pierde después de reiniciar.

---

# 86. Eliminar ruta temporal

```bash
sudo ip route del \
  10.20.0.0/16 \
  via 192.168.50.254 \
  dev ens192
```

---

# 87. Agregar ruta persistente con nmcli

```bash
sudo nmcli connection modify red-produccion \
  +ipv4.routes "10.20.0.0/16 192.168.50.254"
```

Reactivar:

```bash
sudo nmcli connection up red-produccion
```

Verificar:

```bash
ip route
```

---

# 88. Agregar varias rutas

```bash
sudo nmcli connection modify red-produccion \
  +ipv4.routes "10.30.0.0/16 192.168.50.254"
```

Cada uso de:

```text
+ipv4.routes
```

agrega una ruta sin borrar las existentes.

---

# 89. Sustituir todas las rutas

```bash
sudo nmcli connection modify red-produccion \
  ipv4.routes "10.20.0.0/16 192.168.50.254,10.30.0.0/16 192.168.50.254"
```

Sin el signo `+`, la lista puede reemplazarse completamente.

---

# 90. Eliminar una ruta persistente

```bash
sudo nmcli connection modify red-produccion \
  -ipv4.routes "10.20.0.0/16 192.168.50.254"
```

Reactivar:

```bash
sudo nmcli connection up red-produccion
```

---

# 91. Consultar rutas del perfil

```bash
nmcli -f ipv4.routes connection show red-produccion
```

---

# 92. Métrica de ruta

La métrica determina prioridad.

```text
Métrica menor

↓

Mayor prioridad
```

Ejemplo:

```bash
sudo nmcli connection modify red-produccion \
  ipv4.route-metric 100
```

Otra conexión:

```bash
sudo nmcli connection modify red-respaldo \
  ipv4.route-metric 200
```

La conexión con métrica 100 será preferida.

---

# 93. Varias puertas de enlace

No es recomendable configurar múltiples gateways predeterminados sin comprender:

- Métricas.
- Enrutamiento asimétrico.
- Policy routing.
- Failover.
- Interfaces múltiples.
- Reglas de origen.

Un error frecuente es crear dos rutas por defecto con la misma prioridad.

---

# 94. Comandos de consulta rápida

| Objetivo | Comando |
|---|---|
| Ver interfaces | `ip link` |
| Ver IP | `ip addr` |
| Ver rutas | `ip route` |
| Ver dispositivos NetworkManager | `nmcli device status` |
| Ver conexiones | `nmcli connection show` |
| Ver DNS | `nmcli device show` |
| Ver hostname | `hostnamectl` |
| Probar conectividad | `ping` |
| Probar puerto | `nc -vz` |
| Ver sockets | `ss -tulpn` |
| Consultar DNS | `dig` |
| Ver enlace físico | `ethtool` |

---

# 95. Crear configuración completa del escenario

## Paso 1: verificar interfaz

```bash
nmcli device status
```

Supongamos:

```text
ens192
```

---

## Paso 2: crear perfil

```bash
sudo nmcli connection add \
  type ethernet \
  ifname ens192 \
  con-name red-produccion
```

---

## Paso 3: configurar IPv4

```bash
sudo nmcli connection modify red-produccion \
  ipv4.method manual \
  ipv4.addresses 192.168.50.20/24 \
  ipv4.gateway 192.168.50.1 \
  ipv4.dns "192.168.50.10 8.8.8.8" \
  ipv4.dns-search "techdata.local"
```

---

## Paso 4: configurar ruta

```bash
sudo nmcli connection modify red-produccion \
  +ipv4.routes "10.20.0.0/16 192.168.50.254"
```

---

## Paso 5: configurar autoconexión

```bash
sudo nmcli connection modify red-produccion \
  connection.autoconnect yes
```

---

## Paso 6: configurar IPv6

Si no se utiliza:

```bash
sudo nmcli connection modify red-produccion \
  ipv6.method disabled
```

---

## Paso 7: activar

```bash
sudo nmcli connection up red-produccion
```

---

## Paso 8: configurar hostname

```bash
sudo hostnamectl set-hostname server01.techdata.local
```

---

## Paso 9: validar

```bash
ip addr show ens192
ip route
nmcli device show ens192
hostnamectl
```

---

# 96. Validación automatizada básica

```bash
#!/bin/bash

INTERFACE="ens192"
EXPECTED_IP="192.168.50.20/24"
EXPECTED_GATEWAY="192.168.50.1"
EXPECTED_HOSTNAME="server01.techdata.local"

echo "=== VALIDACIÓN DE RED ==="

echo
echo "1. Interfaz:"
ip link show "$INTERFACE"

echo
echo "2. Dirección IPv4:"
ip -4 addr show "$INTERFACE" | grep "$EXPECTED_IP"

echo
echo "3. Gateway:"
ip route | grep "default via $EXPECTED_GATEWAY"

echo
echo "4. Hostname:"
hostnamectl --static

echo
echo "5. NetworkManager:"
systemctl is-active NetworkManager

echo
echo "6. Conectividad local:"
ping -c 2 127.0.0.1

echo
echo "7. Conectividad con gateway:"
ping -c 2 "$EXPECTED_GATEWAY"

echo
echo "8. Resolución DNS:"
getent hosts example.com
```

Guardar como:

```text
validar-red.sh
```

Dar permisos:

```bash
chmod +x validar-red.sh
```

Ejecutar:

```bash
sudo ./validar-red.sh
```

---

# 97. Diagnóstico por capas

```text
Aplicación

↓

Puerto

↓

DNS

↓

Ruta

↓

Dirección IP

↓

Interfaz

↓

Enlace físico

↓

Hardware
```

---

# 98. Metodología de Troubleshooting

Cuando un servidor no tiene conectividad:

```text
1. Verificar NetworkManager

2. Verificar dispositivo

3. Verificar enlace

4. Verificar dirección IP

5. Verificar máscara

6. Verificar ruta local

7. Verificar gateway

8. Verificar DNS

9. Verificar firewall

10. Verificar servicio remoto
```

---

# 99. Problema: interfaz desconectada

Diagnóstico:

```bash
nmcli device status
```

Salida:

```text
ens192 ethernet disconnected
```

Intentar:

```bash
sudo nmcli device connect ens192
```

Verificar perfil:

```bash
nmcli connection show
```

---

# 100. Problema: interfaz no administrada

Salida:

```text
ens192 ethernet unmanaged
```

Revisar:

```bash
nmcli device show ens192
```

Consultar configuración:

```bash
grep -R "managed" /etc/NetworkManager/
```

Revisar si otra herramienta administra la interfaz.

---

# 101. Problema: no existe dirección IP

Consultar:

```bash
ip addr show ens192
```

Verificar perfil:

```bash
nmcli connection show red-produccion
```

Intentar activar:

```bash
sudo nmcli connection up red-produccion
```

Revisar logs:

```bash
journalctl -u NetworkManager -n 100
```

---

# 102. Problema: dirección IP duplicada

Síntomas:

- Conectividad intermitente.
- Advertencias ARP.
- Sesiones que se desconectan.
- Respuestas desde MAC distintas.

Consultar vecinos:

```bash
ip neigh
```

Probar con `arping`:

```bash
sudo arping -D -I ens192 192.168.50.20
```

Instalar si es necesario:

```bash
sudo dnf install -y iputils
```

---

# 103. Problema: gateway incorrecto

Consultar:

```bash
ip route
```

Verificar perfil:

```bash
nmcli -f ipv4.gateway connection show red-produccion
```

Corregir:

```bash
sudo nmcli connection modify red-produccion \
  ipv4.gateway 192.168.50.1
```

Reactivar:

```bash
sudo nmcli connection up red-produccion
```

---

# 104. Problema: DNS incorrecto

Prueba:

```bash
ping -c 2 8.8.8.8
```

Luego:

```bash
ping -c 2 example.com
```

Si el primero funciona y el segundo falla:

```text
Problema probable de DNS
```

Verificar:

```bash
cat /etc/resolv.conf
nmcli device show ens192 | grep DNS
```

---

# 105. Problema: servicio no escucha

Consultar:

```bash
sudo ss -tulpn
```

Ejemplo para SSH:

```bash
sudo ss -tulpn | grep ':22'
```

Verificar servicio:

```bash
systemctl status sshd
```

---

# 106. Problema: puerto bloqueado

Revisar servicio:

```bash
sudo ss -tulpn
```

Revisar firewall:

```bash
sudo firewall-cmd --list-all
```

Probar desde cliente:

```bash
nc -vz servidor 22
```

---

# 107. Problema: ruta ausente

Consultar:

```bash
ip route
```

Probar ruta:

```bash
ip route get 10.20.10.50
```

Si no existe una ruta válida, agregarla temporalmente o mediante NetworkManager.

---

# 108. Problema: perfil no se activa al iniciar

Verificar:

```bash
nmcli -f connection.id,connection.autoconnect \
connection show red-produccion
```

Corregir:

```bash
sudo nmcli connection modify red-produccion \
  connection.autoconnect yes
```

---

# 109. Problema: cambio perdido después del reinicio

Causa frecuente:

```text
Se utilizó ip addr add o ip route add
```

Estos comandos normalmente crean configuraciones temporales.

Para persistencia utilizar:

```text
nmcli
```

---

# 110. Configuración temporal con ip

Agregar IP temporal:

```bash
sudo ip addr add 192.168.50.25/24 dev ens192
```

Activar interfaz:

```bash
sudo ip link set ens192 up
```

Agregar gateway temporal:

```bash
sudo ip route add default via 192.168.50.1
```

Estas configuraciones se pierden al reiniciar.

---

# 111. Eliminar IP temporal

```bash
sudo ip addr del 192.168.50.25/24 dev ens192
```

---

# 112. Limpiar direcciones de una interfaz

```bash
sudo ip addr flush dev ens192
```

Advertencia:

```text
Este comando elimina las direcciones activas de la interfaz y puede desconectarte.
```

---

# 113. Consultar caché ARP o vecinos

```bash
ip neigh
```

Ejemplo:

```text
192.168.50.1 dev ens192 lladdr 00:11:22:33:44:55 REACHABLE
```

Estados frecuentes:

| Estado | Significado |
|---|---|
| `REACHABLE` | Vecino confirmado |
| `STALE` | Entrada conocida pero no reciente |
| `DELAY` | Esperando confirmación |
| `FAILED` | No pudo resolverse |
| `INCOMPLETE` | Resolución en progreso |

---

# 114. Limpiar caché de vecinos

```bash
sudo ip neigh flush all
```

Utilizar sólo durante diagnóstico.

---

# 115. Trazar ruta

Instalar:

```bash
sudo dnf install -y traceroute
```

Ejecutar:

```bash
traceroute 8.8.8.8
```

Para TCP:

```bash
sudo traceroute -T -p 443 example.com
```

---

# 116. Usar tracepath

```bash
tracepath 8.8.8.8
```

`tracepath` normalmente no requiere privilegios de root.

---

# 117. Ver estadísticas de interfaz

```bash
ip -s link show ens192
```

Muestra:

- Paquetes recibidos.
- Paquetes enviados.
- Errores.
- Paquetes descartados.
- Colisiones.
- Overruns.

---

# 118. Interpretar errores de interfaz

```text
RX errors
TX errors
dropped
overruns
carrier
collisions
```

Valores crecientes pueden indicar:

- Cable defectuoso.
- Problemas de switch.
- Driver.
- Saturación.
- MTU incorrecta.
- Negociación deficiente.
- Hardware defectuoso.

---

# 119. MTU

Consultar:

```bash
ip link show ens192
```

Valor común:

```text
1500
```

Modificar temporalmente:

```bash
sudo ip link set dev ens192 mtu 1400
```

Configurar persistentemente:

```bash
sudo nmcli connection modify red-produccion \
  802-3-ethernet.mtu 1400
```

Reactivar:

```bash
sudo nmcli connection up red-produccion
```

---

# 120. No cambiar MTU sin justificación

Una MTU incorrecta puede provocar:

- Fragmentación.
- Paquetes descartados.
- VPN inestable.
- Aplicaciones lentas.
- Conexiones que se establecen pero no transfieren datos.
- Problemas difíciles de diagnosticar.

---

# 121. Ver logs de NetworkManager

Últimas líneas:

```bash
journalctl -u NetworkManager -n 100
```

Seguir en tiempo real:

```bash
journalctl -u NetworkManager -f
```

Desde el arranque actual:

```bash
journalctl -u NetworkManager -b
```

---

# 122. Aumentar nivel de logging temporalmente

```bash
sudo nmcli general logging level DEBUG domains ALL
```

Consultar:

```bash
nmcli general logging
```

Restaurar:

```bash
sudo nmcli general logging level INFO domains DEFAULT
```

---

# 123. Estado general de NetworkManager

```bash
nmcli general status
```

Ejemplo:

```text
STATE      CONNECTIVITY  WIFI-HW  WIFI
connected  full          enabled  enabled
```

---

# 124. Ver conectividad detectada

```bash
nmcli networking connectivity
```

Resultados posibles:

| Resultado | Significado |
|---|---|
| `full` | Acceso completo |
| `limited` | Acceso limitado |
| `portal` | Portal cautivo |
| `none` | Sin conectividad |
| `unknown` | No determinado |

---

# 125. Habilitar networking

```bash
sudo nmcli networking on
```

Deshabilitar:

```bash
sudo nmcli networking off
```

Advertencia:

```text
Deshabilitar networking desconecta todas las interfaces administradas.
```

---

# 126. Buenas prácticas

- Documentar cada dirección IP.
- Utilizar nombres descriptivos para perfiles.
- Verificar el nombre real de la interfaz.
- Utilizar `nmcli` para cambios persistentes.
- Mantener NetworkManager habilitado.
- Evitar editar `/etc/resolv.conf` directamente.
- Configurar `connection.autoconnect yes`.
- Validar gateway y DNS antes de activar.
- Probar primero en un servidor de laboratorio.
- Mantener acceso por consola para cambios remotos.
- No reiniciar NetworkManager innecesariamente.
- Utilizar prefijos CIDR correctamente.
- Evitar direcciones duplicadas.
- Mantener inventario de IP.
- Utilizar DNS interno para servidores empresariales.
- Agregar rutas con `+ipv4.routes`.
- Verificar la tabla de rutas.
- Probar conectividad por capas.
- Confirmar que el servicio esté escuchando.
- Verificar firewall antes de culpar a la red.
- Revisar logs.
- Guardar evidencias de cambios.
- Utilizar perfiles separados para Producción y respaldo.
- No deshabilitar IPv6 sin una política.
- Validar persistencia después de reiniciar.

---

# 127. Errores comunes

## Error 1: configurar la interfaz equivocada

Verificar siempre:

```bash
nmcli device status
```

---

## Error 2: usar dirección sin prefijo

Incorrecto:

```text
192.168.50.20
```

Correcto:

```text
192.168.50.20/24
```

---

## Error 3: configurar gateway fuera de la red

Para:

```text
192.168.50.20/24
```

un gateway como:

```text
192.168.60.1
```

normalmente no es alcanzable directamente.

---

## Error 4: olvidar ipv4.method manual

Una conexión estática debe utilizar:

```bash
ipv4.method manual
```

---

## Error 5: editar resolv.conf directamente

NetworkManager puede sobrescribirlo.

Configurar DNS mediante:

```bash
nmcli connection modify
```

---

## Error 6: usar ip addr para cambios persistentes

`ip addr` modifica el estado actual, pero no necesariamente el perfil permanente.

---

## Error 7: bajar una conexión SSH activa

Ejecutar:

```bash
nmcli connection down
```

puede cortar la sesión remota.

---

## Error 8: reemplazar rutas por accidente

Usar:

```bash
+ipv4.routes
```

para agregar sin sobrescribir.

---

## Error 9: confundir nombre de conexión con interfaz

Ejemplo:

```text
Conexión: red-produccion

Interfaz: ens192
```

No son lo mismo.

---

## Error 10: asumir que ping confirma un servicio

`ping` prueba ICMP, no confirma que:

- SSH funcione.
- HTTP funcione.
- PostgreSQL escuche.
- El puerto esté permitido.

---

# 128. Checklist de configuración

```text
□ NetworkManager activo

□ Interfaz correcta identificada

□ Perfil correcto identificado

□ Dirección IP configurada

□ Prefijo correcto

□ Gateway correcto

□ DNS configurado

□ Dominio de búsqueda configurado

□ Ruta adicional configurada

□ Autoconexión habilitada

□ Hostname configurado

□ Dirección visible con ip addr

□ Ruta visible con ip route

□ DNS visible con nmcli

□ Gateway responde

□ Resolución DNS funciona

□ Servicios escuchan

□ Firewall validado

□ Persistencia comprobada

□ Cambios documentados
```

---

# 129. Laboratorios RHCSA

## Laboratorio 1: identificar interfaces

Ejecuta:

```bash
ip link
nmcli device status
```

Documenta:

- Nombre.
- Tipo.
- Estado.
- Dirección MAC.
- Perfil activo.

---

## Laboratorio 2: consultar configuración actual

Obtén:

- IPv4.
- IPv6.
- Gateway.
- DNS.
- Rutas.
- Hostname.

Utiliza:

```bash
ip addr
ip route
nmcli device show
hostnamectl
```

---

## Laboratorio 3: crear conexión DHCP

Crea un perfil llamado:

```text
laboratorio-dhcp
```

para una interfaz Ethernet disponible.

Configura:

```text
ipv4.method auto
connection.autoconnect yes
```

Actívalo y valida la dirección recibida.

---

## Laboratorio 4: crear conexión estática

Configura:

| Parámetro | Valor |
|---|---|
| Conexión | `laboratorio-static` |
| Dirección | `192.168.100.50/24` |
| Gateway | `192.168.100.1` |
| DNS | `8.8.8.8` |
| Autoconnect | `yes` |

Utiliza una red que corresponda a tu laboratorio.

---

## Laboratorio 5: modificar DNS

Configura:

```text
DNS primario: 1.1.1.1
DNS secundario: 8.8.8.8
```

Valida mediante:

```bash
nmcli device show
cat /etc/resolv.conf
dig example.com
```

---

## Laboratorio 6: configurar hostname

Establece:

```text
node01.laboratorio.local
```

Verifica:

```bash
hostnamectl
hostname
cat /etc/hostname
```

---

## Laboratorio 7: ruta estática

Agrega una ruta persistente:

```text
Destino: 172.16.0.0/16
Gateway: 192.168.100.254
```

Valida:

```bash
ip route
nmcli connection show laboratorio-static
```

---

## Laboratorio 8: identificar puertos

Instala y arranca SSH.

Verifica:

```bash
sudo ss -tulpn | grep ':22'
```

Prueba desde otro host:

```bash
nc -vz direccion_ip 22
```

---

## Laboratorio 9: diagnóstico DNS

Simula un DNS incorrecto.

Comprueba que:

```bash
ping -c 2 8.8.8.8
```

funciona, pero:

```bash
ping -c 2 example.com
```

falla.

Corrige el DNS y documenta el proceso.

---

## Laboratorio 10: persistencia

Reinicia el servidor.

Después verifica:

```bash
nmcli con show --active
ip addr
ip route
hostnamectl
```

La configuración debe mantenerse.

---

# 130. Laboratorio empresarial completo

## Escenario

Debes configurar tres servidores.

| Servidor | IP | Gateway | DNS | Función |
|---|---|---|---|---|
| `web01` | `192.168.80.21/24` | `192.168.80.1` | `192.168.80.10` | Apache |
| `db01` | `192.168.80.31/24` | `192.168.80.1` | `192.168.80.10` | PostgreSQL |
| `backup01` | `192.168.80.41/24` | `192.168.80.1` | `192.168.80.10` | Respaldos |

Dominio:

```text
empresa.local
```

Ruta adicional:

```text
10.100.0.0/16 vía 192.168.80.254
```

---

## Requisitos

```text
□ Configurar hostname

□ Crear perfil con nombre descriptivo

□ Configurar IPv4 estática

□ Configurar gateway

□ Configurar DNS

□ Configurar dominio de búsqueda

□ Habilitar autoconexión

□ Agregar ruta estática

□ Validar dirección IP

□ Validar tabla de rutas

□ Validar resolución DNS

□ Verificar conectividad entre servidores

□ Verificar persistencia tras reinicio

□ Generar reporte de evidencias
```

---

# 131. Ejemplo para web01

```bash
sudo hostnamectl set-hostname web01.empresa.local
```

Crear perfil:

```bash
sudo nmcli connection add \
  type ethernet \
  ifname ens192 \
  con-name red-web01 \
  ipv4.method manual \
  ipv4.addresses 192.168.80.21/24 \
  ipv4.gateway 192.168.80.1 \
  ipv4.dns 192.168.80.10 \
  ipv4.dns-search empresa.local \
  connection.autoconnect yes
```

Agregar ruta:

```bash
sudo nmcli connection modify red-web01 \
  +ipv4.routes "10.100.0.0/16 192.168.80.254"
```

Activar:

```bash
sudo nmcli connection up red-web01
```

---

# 132. Reporte de validación

```bash
{
  echo "=== FECHA ==="
  date

  echo
  echo "=== HOSTNAME ==="
  hostnamectl

  echo
  echo "=== DISPOSITIVOS ==="
  nmcli device status

  echo
  echo "=== CONEXIONES ACTIVAS ==="
  nmcli connection show --active

  echo
  echo "=== DIRECCIONES ==="
  ip addr

  echo
  echo "=== RUTAS ==="
  ip route

  echo
  echo "=== DNS ==="
  nmcli device show | grep -E 'GENERAL.DEVICE|IP4.DNS|IP4.DOMAIN'

  echo
  echo "=== SOCKETS ==="
  ss -tuln

  echo
  echo "=== NETWORKMANAGER ==="
  systemctl is-active NetworkManager
} | sudo tee /root/reporte-redes.txt
```

---

# 133. Desafío final de la Fase 1

Configura un servidor con los siguientes datos:

| Parámetro | Valor |
|---|---|
| Hostname | `rhserver01.lab.local` |
| Interfaz | La disponible en tu sistema |
| Perfil | `red-laboratorio` |
| Dirección | `192.168.90.50/24` |
| Gateway | `192.168.90.1` |
| DNS primario | `192.168.90.10` |
| DNS secundario | `1.1.1.1` |
| Search domain | `lab.local` |
| Ruta | `172.20.0.0/16 vía 192.168.90.254` |
| IPv6 | Automático o según política |
| Autoconnect | Habilitado |

Debes demostrar:

```text
□ NetworkManager activo

□ Perfil creado

□ Dirección correcta

□ Gateway correcto

□ DNS correcto

□ Ruta persistente

□ Hostname correcto

□ Conectividad local

□ Conectividad con gateway

□ Resolución DNS

□ Puerto SSH escuchando

□ Configuración persistente tras reinicio

□ Reporte guardado
```

---

# 134. Criterios de evaluación

| Criterio | Puntos |
|---|---:|
| Identificación correcta de interfaz | 5 |
| Creación del perfil | 10 |
| Dirección IPv4 correcta | 15 |
| Gateway correcto | 10 |
| DNS correcto | 10 |
| Hostname correcto | 10 |
| Ruta estática persistente | 10 |
| Autoconexión | 5 |
| Validaciones de conectividad | 10 |
| Verificación de puertos | 5 |
| Persistencia después de reinicio | 5 |
| Evidencias y documentación | 5 |
| **Total** | **100** |

---

# 135. Preguntas de repaso

1. ¿Qué función cumple NetworkManager?
2. ¿Cuál es la diferencia entre dispositivo y conexión?
3. ¿Qué comando muestra interfaces de red?
4. ¿Qué comando muestra direcciones IP?
5. ¿Cómo se muestran las rutas?
6. ¿Qué representa `/24`?
7. ¿Cuál es la máscara equivalente a `/24`?
8. ¿Qué función cumple la puerta de enlace?
9. ¿Qué función cumple DNS?
10. ¿Qué comando muestra los perfiles de NetworkManager?
11. ¿Cómo se crea una conexión DHCP?
12. ¿Cómo se crea una conexión estática?
13. ¿Qué propiedad define una dirección manual?
14. ¿Cómo se configura un DNS mediante `nmcli`?
15. ¿Cómo se habilita autoconexión?
16. ¿Qué diferencia existe entre `nmcli con down` y `nmcli device disconnect`?
17. ¿Cómo se agrega una ruta sin reemplazar las anteriores?
18. ¿Qué comando muestra la ruta usada hacia un destino?
19. ¿Por qué no debe editarse `/etc/resolv.conf` directamente?
20. ¿Qué archivo contiene el hostname persistente?
21. ¿Qué función cumple `/etc/hosts`?
22. ¿Cómo se consulta DNS con `dig`?
23. ¿Cómo se prueba un puerto TCP?
24. ¿Qué comando muestra servicios escuchando?
25. ¿Por qué `ping` no confirma que una aplicación funcione?
26. ¿Qué diferencia existe entre una ruta temporal y persistente?
27. ¿Cómo se revisan los logs de NetworkManager?
28. ¿Qué puede indicar un estado `NO-CARRIER`?
29. ¿Qué riesgos existen al bajar una conexión remota?
30. ¿Cuál es el orden recomendado para diagnosticar conectividad?

---

# 136. Resumen de la Fase 1

En esta fase desarrollamos un laboratorio completo de administración básica de redes en Linux.

Aprendimos a:

- Comprender IPv4, prefijos y máscaras.
- Identificar interfaces.
- Consultar direcciones IP.
- Diferenciar dispositivos y conexiones.
- Administrar NetworkManager.
- Utilizar `nmcli`.
- Crear conexiones DHCP.
- Crear conexiones estáticas.
- Configurar gateway.
- Configurar DNS.
- Configurar dominios de búsqueda.
- Configurar hostname.
- Utilizar `/etc/hosts`.
- Consultar rutas.
- Agregar rutas estáticas.
- Configurar autoconexión.
- Verificar conectividad.
- Probar puertos.
- Consultar sockets.
- Revisar estadísticas.
- Consultar logs.
- Diagnosticar problemas de red.
- Validar persistencia.
- Generar reportes de evidencias.

Estas competencias son fundamentales para el examen RHCSA y para la administración diaria de servidores Linux.

---

# Próxima fase

## Fase 2 — Administración avanzada de redes

En la siguiente fase se desarrollarán:

- Configuración avanzada de IPv6.
- Múltiples direcciones IP.
- Múltiples interfaces.
- Métricas y rutas avanzadas.
- NetworkManager TUI con `nmtui`.
- Bonding.
- Bridging.
- VLAN.
- Reenvío IP.
- Resolución avanzada de nombres.
- Diagnóstico con `tcpdump`.
- Análisis de paquetes.
- Firewalld.
- Rich Rules.
- NAT y masquerading.
- Troubleshooting avanzado.
- Laboratorio empresarial final de redes.

----

# 55. Laboratorio de Administración Avanzada de Redes en Linux — Fase 2

> **Módulo 6 — Redes en Red Hat**
>
> **Archivo:** `55-laboratorio-redes.md`
>
> **Fase 2:** Administración avanzada de redes
>
> **Nivel:** RHCSA
>
> **Sistemas compatibles:** Red Hat Enterprise Linux, Rocky Linux, AlmaLinux y Fedora
>
> **Objetivo general:** Configurar y diagnosticar redes Linux avanzadas mediante IPv6, múltiples interfaces, múltiples direcciones IP, rutas y métricas, `nmtui`, bonding, bridging, VLAN, reenvío IP, `tcpdump`, `firewalld`, Rich Rules, NAT y masquerading.

---

# 1. Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Administrar configuraciones IPv6.
- Interpretar direcciones IPv6 link-local, globales y loopback.
- Configurar IPv6 automático y estático con NetworkManager.
- Asignar varias direcciones IP a una misma conexión.
- Administrar servidores con múltiples interfaces.
- Configurar métricas para seleccionar rutas preferidas.
- Evitar múltiples rutas predeterminadas conflictivas.
- Consultar rutas avanzadas con `ip route`.
- Administrar conexiones mediante `nmtui`.
- Comprender y configurar bonding.
- Crear interfaces bridge.
- Crear y administrar VLAN.
- Configurar reenvío IPv4 e IPv6.
- Analizar tráfico mediante `tcpdump`.
- Identificar problemas de ARP, ICMP, DNS, TCP y UDP.
- Configurar `firewalld`.
- Administrar zonas, servicios y puertos.
- Crear Rich Rules.
- Configurar masquerading y NAT.
- Realizar port forwarding.
- Aplicar procedimientos avanzados de troubleshooting.
- Resolver laboratorios similares a los evaluados en RHCSA.

---

# 2. Escenario empresarial de la Fase 2

La empresa **TechData Solutions** ampliará su infraestructura.

El nuevo servidor Linux tendrá varias funciones:

- Servidor de aplicaciones.
- Puerta de enlace de una red interna.
- Servidor con conexión redundante.
- Host para máquinas virtuales.
- Servidor con redes VLAN.
- Punto de acceso controlado mediante Firewall.

La infraestructura utilizará los siguientes segmentos:

| Red | Uso |
|---|---|
| `192.168.50.0/24` | Red administrativa |
| `192.168.60.0/24` | Red de aplicaciones |
| `192.168.70.0/24` | Red de bases de datos |
| `192.168.80.0/24` | Red de respaldo |
| `10.20.0.0/16` | Red interna remota |
| `2001:db8:50::/64` | Red IPv6 de laboratorio |

El servidor tendrá las siguientes interfaces:

| Interfaz | Función |
|---|---|
| `ens192` | Administración y salida principal |
| `ens224` | Red interna |
| `ens256` | Interfaz secundaria o redundante |

---

# 3. Arquitectura general

```text
                             Internet

                                │

                                ▼

                       Router principal

                         192.168.50.1

                                │

                                ▼

                     ens192 — Servidor Linux

                         192.168.50.20

                     2001:db8:50::20/64

                                │

              ┌─────────────────┼───────────────────┐

              │                 │                   │

              ▼                 ▼                   ▼

        ens224 interna      VLAN 60            Bridge br0

       192.168.60.1       192.168.60.20       Máquinas virtuales

              │

              ▼

        Red interna

      192.168.60.0/24
```

---

# 4. IPv6

IPv6 fue diseñado para resolver limitaciones de IPv4, principalmente la escasez de direcciones.

Una dirección IPv6 contiene:

```text
128 bits
```

Una dirección IPv4 contiene:

```text
32 bits
```

Ejemplo IPv4:

```text
192.168.50.20
```

Ejemplo IPv6:

```text
2001:db8:50::20
```

---

# 5. Estructura de una dirección IPv6

Ejemplo completo:

```text
2001:0db8:0050:0000:0000:0000:0000:0020
```

Puede abreviarse como:

```text
2001:db8:50::20
```

Reglas de abreviación:

- Se pueden eliminar ceros a la izquierda dentro de un bloque.
- Una secuencia continua de bloques en cero puede representarse mediante `::`.
- `::` sólo debe aparecer una vez en la dirección.

---

# 6. Tipos de direcciones IPv6

| Tipo | Rango | Uso |
|---|---|---|
| Loopback | `::1/128` | Equivalente a `127.0.0.1` |
| Link-local | `fe80::/10` | Comunicación dentro del enlace local |
| Global unicast | `2000::/3` | Direcciones globales |
| Unique local | `fc00::/7` | Redes privadas IPv6 |
| Multicast | `ff00::/8` | Comunicación multicast |
| Unspecified | `::/128` | Dirección no definida |

---

# 7. Dirección link-local

Cada interfaz IPv6 normalmente recibe una dirección que comienza con:

```text
fe80::
```

Ejemplo:

```text
fe80::20c:29ff:fe12:3456/64
```

Esta dirección:

- Sólo es válida dentro del enlace local.
- No debe enrutarse por Internet.
- Puede necesitar que se especifique la interfaz.

Ejemplo de `ping`:

```bash
ping -6 -c 4 fe80::1%ens192
```

El símbolo:

```text
%ens192
```

indica la interfaz o zona de alcance.

---

# 8. Consultar direcciones IPv6

```bash
ip -6 address show
```

Versión corta:

```bash
ip -6 a
```

Para una interfaz:

```bash
ip -6 addr show dev ens192
```

---

# 9. Consultar rutas IPv6

```bash
ip -6 route
```

Ruta predeterminada conceptual:

```text
default via fe80::1 dev ens192
```

---

# 10. Probar conectividad IPv6

Loopback:

```bash
ping -6 -c 4 ::1
```

Puerta de enlace link-local:

```bash
ping -6 -c 4 fe80::1%ens192
```

Destino global:

```bash
ping -6 -c 4 2001:4860:4860::8888
```

La conectividad dependerá de que la red disponga de IPv6 funcional.

---

# 11. Consultar configuración IPv6 con nmcli

```bash
nmcli device show ens192
```

Filtrar:

```bash
nmcli device show ens192 | grep -E 'IP6|GENERAL'
```

Consultar perfil:

```bash
nmcli -f ipv6.method,ipv6.addresses,ipv6.gateway,ipv6.dns \
connection show red-produccion
```

---

# 12. Métodos IPv6 en NetworkManager

| Método | Uso |
|---|---|
| `auto` | SLAAC y posiblemente DHCPv6 |
| `dhcp` | DHCPv6 |
| `manual` | Dirección estática |
| `link-local` | Sólo dirección link-local |
| `disabled` | IPv6 deshabilitado |
| `ignore` | NetworkManager no administra IPv6 |

---

# 13. Configurar IPv6 automático

```bash
sudo nmcli connection modify red-produccion \
  ipv6.method auto
```

Activar:

```bash
sudo nmcli connection up red-produccion
```

Verificar:

```bash
ip -6 addr show ens192
```

---

# 14. Configurar IPv6 estático

Datos:

| Parámetro | Valor |
|---|---|
| Dirección | `2001:db8:50::20/64` |
| Gateway | `2001:db8:50::1` |
| DNS | `2001:4860:4860::8888` |

Configurar:

```bash
sudo nmcli connection modify red-produccion \
  ipv6.method manual \
  ipv6.addresses "2001:db8:50::20/64" \
  ipv6.gateway "2001:db8:50::1" \
  ipv6.dns "2001:4860:4860::8888"
```

Reactivar:

```bash
sudo nmcli connection up red-produccion
```

---

# 15. Configurar varias direcciones IPv6

Primera dirección:

```bash
sudo nmcli connection modify red-produccion \
  ipv6.addresses "2001:db8:50::20/64"
```

Agregar otra:

```bash
sudo nmcli connection modify red-produccion \
  +ipv6.addresses "2001:db8:50::21/64"
```

Verificar:

```bash
nmcli -f ipv6.addresses connection show red-produccion
```

---

# 16. Eliminar una dirección IPv6

```bash
sudo nmcli connection modify red-produccion \
  -ipv6.addresses "2001:db8:50::21/64"
```

Reactivar:

```bash
sudo nmcli connection up red-produccion
```

---

# 17. Configurar DNS IPv6

```bash
sudo nmcli connection modify red-produccion \
  +ipv6.dns "2001:4860:4860::8888"
```

Agregar segundo DNS:

```bash
sudo nmcli connection modify red-produccion \
  +ipv6.dns "2606:4700:4700::1111"
```

---

# 18. Deshabilitar IPv6 en un perfil

```bash
sudo nmcli connection modify red-produccion \
  ipv6.method disabled
```

Reactivar:

```bash
sudo nmcli connection up red-produccion
```

Esto deshabilita IPv6 para el perfil administrado por NetworkManager.

No necesariamente modifica todos los parámetros globales del Kernel.

---

# 19. Consideraciones antes de deshabilitar IPv6

Deshabilitar IPv6 sin análisis puede afectar:

- Servicios que escuchan en `::`.
- Resolución DNS.
- Aplicaciones modernas.
- Kubernetes.
- Contenedores.
- Clústeres.
- Monitoreo.
- Servicios en la nube.
- Dependencias de sistema.

Debe existir una política formal antes de deshabilitarlo.

---

# 20. Múltiples direcciones IPv4

Una interfaz puede tener varias direcciones IPv4.

Ejemplo:

```text
ens192

├── 192.168.50.20/24

└── 192.168.50.21/24
```

Esto puede utilizarse para:

- Migraciones.
- Servicios virtuales.
- Compatibilidad temporal.
- Múltiples aplicaciones.
- Alias de red.
- Direcciones flotantes.

---

# 21. Agregar una dirección IPv4 adicional

```bash
sudo nmcli connection modify red-produccion \
  +ipv4.addresses "192.168.50.21/24"
```

Reactivar:

```bash
sudo nmcli connection up red-produccion
```

Verificar:

```bash
ip -4 addr show ens192
```

---

# 22. Agregar varias direcciones en una sola operación

```bash
sudo nmcli connection modify red-produccion \
  ipv4.addresses \
  "192.168.50.20/24,192.168.50.21/24,192.168.50.22/24"
```

Advertencia:

Sin el signo `+`, se reemplaza la lista actual.

---

# 23. Eliminar una dirección secundaria

```bash
sudo nmcli connection modify red-produccion \
  -ipv4.addresses "192.168.50.21/24"
```

Reactivar:

```bash
sudo nmcli connection up red-produccion
```

---

# 24. Dirección temporal con ip

Agregar:

```bash
sudo ip addr add 192.168.50.25/24 dev ens192
```

Verificar:

```bash
ip addr show ens192
```

Eliminar:

```bash
sudo ip addr del 192.168.50.25/24 dev ens192
```

Estos cambios no son persistentes.

---

# 25. Múltiples interfaces

Un servidor puede tener varias interfaces para separar tráfico.

Ejemplo:

| Interfaz | Dirección | Uso |
|---|---|---|
| `ens192` | `192.168.50.20/24` | Administración |
| `ens224` | `192.168.60.20/24` | Aplicaciones |
| `ens256` | `192.168.80.20/24` | Respaldo |

Diagrama:

```text
                       Servidor

         ┌────────────────┼────────────────┐

         │                │                │

         ▼                ▼                ▼

       ens192           ens224           ens256

   Administración     Aplicación        Backups

  192.168.50.20    192.168.60.20    192.168.80.20
```

---

# 26. Crear conexión para una segunda interfaz

```bash
sudo nmcli connection add \
  type ethernet \
  ifname ens224 \
  con-name red-aplicaciones \
  ipv4.method manual \
  ipv4.addresses 192.168.60.20/24 \
  connection.autoconnect yes
```

No se configura gateway si esa interfaz sólo accede a su red local.

Activar:

```bash
sudo nmcli connection up red-aplicaciones
```

---

# 27. Evitar varios gateways predeterminados

Un servidor con varias interfaces no debe recibir automáticamente múltiples rutas predeterminadas sin una planificación adecuada.

Ejemplo problemático:

```text
default via 192.168.50.1 dev ens192

default via 192.168.60.1 dev ens224
```

Esto puede provocar:

- Respuestas por la interfaz incorrecta.
- Enrutamiento asimétrico.
- Sesiones intermitentes.
- Problemas con firewalls.
- Pérdida de conectividad.

---

# 28. Evitar que una conexión sea ruta predeterminada

```bash
sudo nmcli connection modify red-aplicaciones \
  ipv4.never-default yes
```

Para IPv6:

```bash
sudo nmcli connection modify red-aplicaciones \
  ipv6.never-default yes
```

---

# 29. Métricas de ruta

Cuando existen varias rutas hacia el mismo destino, la métrica ayuda a definir la preferida.

```text
Métrica menor = mayor prioridad
```

Ejemplo:

```text
Ruta principal: métrica 100

Ruta secundaria: métrica 200
```

---

# 30. Configurar métrica de conexión principal

```bash
sudo nmcli connection modify red-produccion \
  ipv4.route-metric 100
```

---

# 31. Configurar métrica de respaldo

```bash
sudo nmcli connection modify red-respaldo \
  ipv4.route-metric 200
```

---

# 32. Verificar métricas

```bash
ip route
```

Ejemplo:

```text
default via 192.168.50.1 dev ens192 metric 100

default via 192.168.80.1 dev ens256 metric 200
```

---

# 33. Consultar ruta seleccionada

```bash
ip route get 8.8.8.8
```

Resultado conceptual:

```text
8.8.8.8 via 192.168.50.1 dev ens192 src 192.168.50.20
```

---

# 34. Rutas estáticas con métricas

Agregar ruta:

```bash
sudo nmcli connection modify red-produccion \
  +ipv4.routes "10.20.0.0/16 192.168.50.254,100"
```

Según la versión de NetworkManager, también puede utilizarse sintaxis de atributos de ruta.

Verificar siempre:

```bash
nmcli -f ipv4.routes connection show red-produccion
```

---

# 35. Rutas host

Una ruta hacia una única dirección utiliza `/32`.

Ejemplo:

```bash
sudo nmcli connection modify red-produccion \
  +ipv4.routes "10.20.30.40/32 192.168.50.254"
```

---

# 36. Ruta sin gateway

Cuando el destino está directamente conectado:

```bash
sudo ip route add 10.10.10.0/24 dev ens224
```

Persistente:

```bash
sudo nmcli connection modify red-aplicaciones \
  +ipv4.routes "10.10.10.0/24"
```

La sintaxis exacta puede variar según la versión de NetworkManager.

---

# 37. Consultar todas las tablas

```bash
ip route show table all
```

Para IPv6:

```bash
ip -6 route show table all
```

---

# 38. Policy routing

En servidores con múltiples interfaces puede ser necesario seleccionar rutas según:

- Dirección de origen.
- Interfaz.
- Marca de paquetes.
- Aplicación.
- Tabla de rutas.

Consultar reglas:

```bash
ip rule
```

Ejemplo:

```text
0:      from all lookup local

32766:  from all lookup main

32767:  from all lookup default
```

La configuración avanzada de policy routing supera el alcance básico de RHCSA, pero es importante reconocer su existencia.

---

# 39. NetworkManager TUI

`nmtui` ofrece una interfaz gráfica de texto.

Ejecutar:

```bash
sudo nmtui
```

Opciones principales:

```text
Edit a connection

Activate a connection

Set system hostname
```

---

# 40. Instalar nmtui

En sistemas donde no está disponible:

```bash
sudo dnf install -y NetworkManager-tui
```

---

# 41. Editar una conexión con nmtui

Procedimiento:

```text
1. Ejecutar sudo nmtui

2. Seleccionar Edit a connection

3. Seleccionar el perfil

4. Elegir Edit

5. Configurar IPv4 o IPv6

6. Configurar DNS

7. Configurar rutas

8. Marcar Automatically connect

9. Guardar

10. Activar la conexión
```

---

# 42. Activar una conexión con nmtui

```text
nmtui

↓

Activate a connection

↓

Seleccionar perfil

↓

Activate
```

---

# 43. Cambiar hostname con nmtui

```text
nmtui

↓

Set system hostname

↓

Escribir FQDN

↓

OK
```

También puede utilizarse:

```bash
sudo nmtui-hostname
```

---

# 44. Comandos específicos de nmtui

```bash
sudo nmtui-edit
```

```bash
sudo nmtui-connect
```

```bash
sudo nmtui-hostname
```

---

# 45. Bonding

Bonding combina varias interfaces físicas en una interfaz lógica.

Objetivos:

- Redundancia.
- Alta disponibilidad.
- Balanceo.
- Mayor ancho de banda, según el modo.
- Tolerancia a fallos.

Diagrama:

```text
          ens192 ─────┐

                      ├──── bond0 ─── Red

          ens224 ─────┘
```

---

# 46. Modos comunes de bonding

| Modo | Nombre | Función |
|---:|---|---|
| 0 | `balance-rr` | Balanceo round-robin |
| 1 | `active-backup` | Una interfaz activa y otra en espera |
| 2 | `balance-xor` | Selección mediante hash |
| 3 | `broadcast` | Envía por todas las interfaces |
| 4 | `802.3ad` | LACP |
| 5 | `balance-tlb` | Balanceo adaptativo de transmisión |
| 6 | `balance-alb` | Balanceo adaptativo completo |

Para laboratorios y redundancia simple, suele utilizarse:

```text
active-backup
```

---

# 47. Requisitos para bonding

- Dos o más interfaces.
- Módulo de bonding disponible.
- Interfaces sin direcciones IP independientes.
- Configuración compatible en el switch si se utiliza LACP.
- Acceso por consola durante cambios.
- Pruebas de failover.

---

# 48. Crear un bond active-backup

Crear conexión bond:

```bash
sudo nmcli connection add \
  type bond \
  ifname bond0 \
  con-name bond0 \
  bond.options "mode=active-backup,miimon=100"
```

---

# 49. Agregar primera interfaz al bond

```bash
sudo nmcli connection add \
  type ethernet \
  ifname ens192 \
  con-name bond0-port1 \
  master bond0
```

---

# 50. Agregar segunda interfaz al bond

```bash
sudo nmcli connection add \
  type ethernet \
  ifname ens224 \
  con-name bond0-port2 \
  master bond0
```

En versiones recientes también puede aparecer terminología:

```text
controller
port-type
```

NetworkManager mantiene compatibilidad con la terminología tradicional `master` y `slave` en muchas versiones.

---

# 51. Configurar IPv4 en bond0

```bash
sudo nmcli connection modify bond0 \
  ipv4.method manual \
  ipv4.addresses 192.168.50.20/24 \
  ipv4.gateway 192.168.50.1 \
  ipv4.dns "192.168.50.10 8.8.8.8" \
  connection.autoconnect yes
```

---

# 52. Activar bond

```bash
sudo nmcli connection up bond0
```

Activar puertos si fuera necesario:

```bash
sudo nmcli connection up bond0-port1
sudo nmcli connection up bond0-port2
```

---

# 53. Verificar bond

```bash
nmcli device status
```

Consultar:

```bash
cat /proc/net/bonding/bond0
```

Información importante:

```text
Bonding Mode
Currently Active Slave
MII Status
Slave Interface
Link Failure Count
```

---

# 54. Probar failover

Consultar interfaz activa:

```bash
cat /proc/net/bonding/bond0
```

Desconectar temporalmente un puerto de laboratorio:

```bash
sudo nmcli device disconnect ens192
```

Verificar:

```bash
cat /proc/net/bonding/bond0
```

El segundo puerto debe asumir el tráfico.

Advertencia:

No realices esta prueba en Producción sin una ventana autorizada.

---

# 55. Configurar interfaz primaria del bond

```bash
sudo nmcli connection modify bond0 \
  bond.options "mode=active-backup,miimon=100,primary=ens192"
```

---

# 56. Bonding 802.3ad

Ejemplo:

```bash
sudo nmcli connection add \
  type bond \
  ifname bond0 \
  con-name bond0 \
  bond.options "mode=802.3ad,miimon=100,lacp_rate=fast"
```

Este modo requiere configuración compatible en el switch.

Sin configuración del switch, el enlace puede fallar o funcionar incorrectamente.

---

# 57. Eliminar un bond

Primero desactivar:

```bash
sudo nmcli connection down bond0
```

Eliminar perfiles:

```bash
sudo nmcli connection delete bond0-port1
sudo nmcli connection delete bond0-port2
sudo nmcli connection delete bond0
```

Verificar:

```bash
nmcli connection show
```

---

# 58. Bridge de red

Un bridge funciona de manera similar a un switch de software.

Puede conectar:

- Interfaces físicas.
- Máquinas virtuales.
- Contenedores.
- Redes virtuales.
- Interfaces TAP.

Diagrama:

```text
                      br0

              ┌────────┼────────┐

              │        │        │

              ▼        ▼        ▼

           ens192     VM1      VM2
```

---

# 59. Cuándo utilizar un bridge

- Hosts KVM.
- Máquinas virtuales que deben aparecer en la red física.
- Contenedores.
- Laboratorios de virtualización.
- Integración de interfaces virtuales.

---

# 60. Crear bridge br0

```bash
sudo nmcli connection add \
  type bridge \
  ifname br0 \
  con-name br0
```

---

# 61. Configurar IPv4 en el bridge

```bash
sudo nmcli connection modify br0 \
  ipv4.method manual \
  ipv4.addresses 192.168.50.20/24 \
  ipv4.gateway 192.168.50.1 \
  ipv4.dns "192.168.50.10 8.8.8.8" \
  connection.autoconnect yes
```

---

# 62. Agregar interfaz física al bridge

```bash
sudo nmcli connection add \
  type ethernet \
  ifname ens192 \
  con-name br0-port1 \
  master br0
```

En configuraciones de bridge, la dirección IP debe asignarse a:

```text
br0
```

No a:

```text
ens192
```

---

# 63. Activar bridge

```bash
sudo nmcli connection up br0
```

Activar puerto:

```bash
sudo nmcli connection up br0-port1
```

---

# 64. Verificar bridge

```bash
nmcli device status
```

```bash
ip addr show br0
```

```bash
bridge link
```

```bash
bridge vlan show
```

---

# 65. Instalar herramientas de bridge

Si el comando `bridge` no está disponible:

```bash
sudo dnf install -y iproute
```

En sistemas modernos suele formar parte de `iproute`.

---

# 66. STP

Spanning Tree Protocol ayuda a evitar bucles de red.

Consultar:

```bash
nmcli -f bridge.stp connection show br0
```

Habilitar:

```bash
sudo nmcli connection modify br0 \
  bridge.stp yes
```

Deshabilitar en un laboratorio controlado:

```bash
sudo nmcli connection modify br0 \
  bridge.stp no
```

Debe comprenderse la topología antes de deshabilitarlo.

---

# 67. Eliminar bridge

```bash
sudo nmcli connection down br0
```

```bash
sudo nmcli connection delete br0-port1
```

```bash
sudo nmcli connection delete br0
```

---

# 68. VLAN

Una VLAN permite dividir lógicamente una red física.

Ejemplo:

| VLAN | Red | Uso |
|---:|---|---|
| 10 | `192.168.10.0/24` | Administración |
| 20 | `192.168.20.0/24` | Aplicaciones |
| 30 | `192.168.30.0/24` | Bases de datos |
| 40 | `192.168.40.0/24` | Respaldo |

---

# 69. Arquitectura VLAN

```text
                  Switch configurado como trunk

                              │

                              ▼

                           ens192

                              │

           ┌──────────────────┼──────────────────┐

           │                  │                  │

           ▼                  ▼                  ▼

       ens192.10          ens192.20          ens192.30

        VLAN 10            VLAN 20            VLAN 30
```

---

# 70. Requisitos de VLAN

- Switch configurado correctamente.
- Puerto en modo trunk o etiquetado.
- ID de VLAN correcto.
- Interfaz física funcional.
- NetworkManager activo.
- Direcciones IP acordes con cada red.

---

# 71. Crear VLAN 60

Datos:

```text
Interfaz principal: ens192

VLAN ID: 60

Interfaz VLAN: ens192.60

Dirección: 192.168.60.20/24
```

Comando:

```bash
sudo nmcli connection add \
  type vlan \
  con-name vlan60 \
  ifname ens192.60 \
  dev ens192 \
  id 60 \
  ipv4.method manual \
  ipv4.addresses 192.168.60.20/24
```

---

# 72. Configurar autoconexión de VLAN

```bash
sudo nmcli connection modify vlan60 \
  connection.autoconnect yes
```

---

# 73. Evitar gateway en VLAN interna

```bash
sudo nmcli connection modify vlan60 \
  ipv4.never-default yes
```

---

# 74. Activar VLAN

```bash
sudo nmcli connection up vlan60
```

---

# 75. Verificar VLAN

```bash
nmcli device status
```

```bash
ip -d link show ens192.60
```

La opción:

```text
-d
```

muestra detalles adicionales, incluido el ID de VLAN.

---

# 76. Consultar VLAN con nmcli

```bash
nmcli -f connection.id,vlan.id,vlan.parent,ipv4.addresses \
connection show vlan60
```

---

# 77. Crear varias VLAN

VLAN 70:

```bash
sudo nmcli connection add \
  type vlan \
  con-name vlan70 \
  ifname ens192.70 \
  dev ens192 \
  id 70 \
  ipv4.method manual \
  ipv4.addresses 192.168.70.20/24 \
  ipv4.never-default yes
```

VLAN 80:

```bash
sudo nmcli connection add \
  type vlan \
  con-name vlan80 \
  ifname ens192.80 \
  dev ens192 \
  id 80 \
  ipv4.method manual \
  ipv4.addresses 192.168.80.20/24 \
  ipv4.never-default yes
```

---

# 78. Eliminar VLAN

```bash
sudo nmcli connection down vlan60
```

```bash
sudo nmcli connection delete vlan60
```

---

# 79. Errores comunes de VLAN

- VLAN no configurada en el switch.
- Puerto no está en modo trunk.
- ID incorrecto.
- Interfaz padre incorrecta.
- IP perteneciente a otra red.
- Firewall bloqueando tráfico.
- MTU incompatible.
- Gateway configurado innecesariamente.
- VLAN nativa confundida con VLAN etiquetada.

---

# 80. Reenvío IP

Un sistema Linux puede reenviar tráfico entre interfaces.

Ejemplo:

```text
Cliente interno

192.168.60.50

       │

       ▼

Servidor Linux

ens224: 192.168.60.1

ens192: 192.168.50.20

       │

       ▼

Internet
```

Para funcionar como router, debe habilitarse:

```text
IP forwarding
```

---

# 81. Consultar reenvío IPv4

```bash
sysctl net.ipv4.ip_forward
```

Resultado deshabilitado:

```text
net.ipv4.ip_forward = 0
```

Resultado habilitado:

```text
net.ipv4.ip_forward = 1
```

---

# 82. Habilitar reenvío temporalmente

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

Este cambio puede perderse tras reiniciar.

---

# 83. Habilitar reenvío persistentemente

Crear archivo:

```bash
sudo tee /etc/sysctl.d/90-ip-forwarding.conf > /dev/null <<'EOF'
net.ipv4.ip_forward = 1
EOF
```

Aplicar:

```bash
sudo sysctl --system
```

Verificar:

```bash
sysctl net.ipv4.ip_forward
```

---

# 84. Reenvío IPv6

Consultar:

```bash
sysctl net.ipv6.conf.all.forwarding
```

Habilitar persistentemente:

```bash
sudo tee -a /etc/sysctl.d/90-ip-forwarding.conf > /dev/null <<'EOF'
net.ipv6.conf.all.forwarding = 1
EOF
```

Aplicar:

```bash
sudo sysctl --system
```

---

# 85. Seguridad del reenvío

Habilitar IP forwarding convierte el servidor en potencial router.

Debe combinarse con:

- Políticas de Firewall.
- Rutas correctas.
- NAT cuando corresponda.
- Segmentación.
- Monitoreo.
- Registro.
- Control de acceso.

No debe habilitarse sin una necesidad real.

---

# 86. Introducción a tcpdump

`tcpdump` captura y muestra paquetes de red.

Permite analizar:

- ARP.
- ICMP.
- TCP.
- UDP.
- DNS.
- DHCP.
- SSH.
- HTTP.
- PostgreSQL.
- Tráfico entre hosts.
- Handshake TCP.
- Retransmisiones.
- Resets.

---

# 87. Instalar tcpdump

```bash
sudo dnf install -y tcpdump
```

Verificar:

```bash
tcpdump --version
```

---

# 88. Listar interfaces capturables

```bash
sudo tcpdump -D
```

---

# 89. Capturar tráfico en una interfaz

```bash
sudo tcpdump -i ens192
```

Detener con:

```text
Ctrl + C
```

---

# 90. Evitar resolución de nombres

```bash
sudo tcpdump -n -i ens192
```

La opción `-n` evita resolución DNS.

Para evitar nombres de puertos:

```bash
sudo tcpdump -nn -i ens192
```

---

# 91. Capturar una cantidad limitada

```bash
sudo tcpdump -nn -i ens192 -c 20
```

Captura 20 paquetes y finaliza.

---

# 92. Capturar ICMP

```bash
sudo tcpdump -nn -i ens192 icmp
```

Mientras la captura está activa, desde otra terminal:

```bash
ping -c 4 192.168.50.1
```

---

# 93. Capturar ARP

```bash
sudo tcpdump -nn -i ens192 arp
```

ARP permite asociar:

```text
Dirección IPv4

↓

Dirección MAC
```

---

# 94. Capturar tráfico de un host

```bash
sudo tcpdump -nn -i ens192 host 192.168.50.10
```

Sólo origen:

```bash
sudo tcpdump -nn -i ens192 src host 192.168.50.10
```

Sólo destino:

```bash
sudo tcpdump -nn -i ens192 dst host 192.168.50.10
```

---

# 95. Capturar tráfico de una red

```bash
sudo tcpdump -nn -i ens192 net 192.168.50.0/24
```

---

# 96. Capturar un puerto

SSH:

```bash
sudo tcpdump -nn -i ens192 port 22
```

DNS:

```bash
sudo tcpdump -nn -i ens192 port 53
```

PostgreSQL:

```bash
sudo tcpdump -nn -i ens192 port 5432
```

---

# 97. Capturar TCP o UDP

TCP:

```bash
sudo tcpdump -nn -i ens192 tcp
```

UDP:

```bash
sudo tcpdump -nn -i ens192 udp
```

---

# 98. Combinar filtros

Tráfico SSH con un servidor:

```bash
sudo tcpdump -nn -i ens192 \
  host 192.168.50.21 and port 22
```

DNS UDP:

```bash
sudo tcpdump -nn -i ens192 \
  udp and port 53
```

HTTP o HTTPS:

```bash
sudo tcpdump -nn -i ens192 \
  'port 80 or port 443'
```

---

# 99. Excluir tráfico

Excluir SSH para evitar capturar la propia sesión:

```bash
sudo tcpdump -nn -i ens192 \
  'not port 22'
```

---

# 100. Guardar captura

```bash
sudo tcpdump -nn -i ens192 \
  -w /tmp/captura-red.pcap
```

El archivo puede analizarse posteriormente con:

- `tcpdump`.
- Wireshark.
- TShark.

---

# 101. Leer una captura guardada

```bash
sudo tcpdump -nn -r /tmp/captura-red.pcap
```

Filtrar al leer:

```bash
sudo tcpdump -nn -r /tmp/captura-red.pcap port 53
```

---

# 102. Captura con rotación

Crear archivos de 10 MB:

```bash
sudo tcpdump -nn -i ens192 \
  -C 10 \
  -W 5 \
  -w /tmp/captura-%02d.pcap
```

Esto mantiene hasta cinco archivos.

---

# 103. Handshake TCP

El establecimiento normal de una conexión TCP utiliza:

```text
Cliente                    Servidor

  │                           │

  │ -------- SYN -----------> │

  │ <----- SYN, ACK ----------│

  │ -------- ACK -----------> │

  │                           │

  │     Conexión establecida  │
```

---

# 104. Identificar SYN con tcpdump

```bash
sudo tcpdump -nn -i ens192 \
  'tcp[tcpflags] & tcp-syn != 0'
```

---

# 105. Capturar paquetes RST

```bash
sudo tcpdump -nn -i ens192 \
  'tcp[tcpflags] & tcp-rst != 0'
```

Un RST puede indicar:

- Puerto cerrado.
- Aplicación rechazando conexión.
- Firewall enviando rechazo.
- Sesión terminada abruptamente.

---

# 106. Diagnóstico mediante captura

| Observación | Posible causa |
|---|---|
| Se envía SYN y no hay respuesta | Firewall, ruta o host caído |
| Se recibe RST | Puerto cerrado o rechazo |
| Handshake completo y luego falla | Problema de aplicación |
| Consulta DNS sin respuesta | DNS inaccesible |
| ARP repetido sin respuesta | Host local no disponible |
| ICMP unreachable | Ruta o puerto inaccesible |
| Muchas retransmisiones | Pérdida, saturación o MTU |

---

# 107. Introducción a firewalld

`firewalld` es el servicio de Firewall dinámico utilizado en sistemas de la familia Red Hat.

Permite administrar reglas mediante:

- Zonas.
- Servicios.
- Puertos.
- Interfaces.
- Fuentes.
- Rich Rules.
- NAT.
- Masquerading.
- Port forwarding.

---

# 108. Arquitectura de firewalld

```text
Tráfico entrante

       │

       ▼

Interfaz o dirección de origen

       │

       ▼

Zona de firewalld

       │

       ▼

Servicios, puertos y reglas

       │

       ▼

Aceptar, rechazar o descartar
```

---

# 109. Verificar firewalld

```bash
systemctl status firewalld
```

Activar:

```bash
sudo systemctl enable --now firewalld
```

Verificar:

```bash
firewall-cmd --state
```

Resultado:

```text
running
```

---

# 110. Zonas predeterminadas

Consultar:

```bash
firewall-cmd --get-zones
```

Zonas frecuentes:

| Zona | Uso general |
|---|---|
| `drop` | Descarta tráfico no permitido |
| `block` | Rechaza tráfico no permitido |
| `public` | Redes públicas |
| `external` | Redes externas con NAT |
| `dmz` | Zona desmilitarizada |
| `work` | Redes de trabajo |
| `home` | Redes domésticas |
| `internal` | Redes internas |
| `trusted` | Confía en todo el tráfico |

---

# 111. Consultar zona predeterminada

```bash
firewall-cmd --get-default-zone
```

---

# 112. Cambiar zona predeterminada

```bash
sudo firewall-cmd --set-default-zone=public
```

---

# 113. Consultar zonas activas

```bash
firewall-cmd --get-active-zones
```

Ejemplo:

```text
public
  interfaces: ens192

internal
  interfaces: ens224
```

---

# 114. Consultar configuración completa

```bash
sudo firewall-cmd --list-all
```

Para una zona específica:

```bash
sudo firewall-cmd \
  --zone=public \
  --list-all
```

---

# 115. Runtime y permanent

Firewalld maneja dos configuraciones:

```text
Runtime

Permanent
```

## Runtime

- Se aplica inmediatamente.
- Se pierde al reiniciar o recargar.

## Permanent

- Se guarda persistentemente.
- No siempre se aplica de inmediato.

---

# 116. Agregar servicio temporalmente

```bash
sudo firewall-cmd \
  --zone=public \
  --add-service=http
```

Verificar:

```bash
firewall-cmd \
  --zone=public \
  --list-services
```

---

# 117. Agregar servicio permanentemente

```bash
sudo firewall-cmd \
  --zone=public \
  --add-service=http \
  --permanent
```

Aplicar configuración permanente:

```bash
sudo firewall-cmd --reload
```

---

# 118. Aplicar simultáneamente runtime y permanent

Método explícito:

```bash
sudo firewall-cmd \
  --zone=public \
  --add-service=http

sudo firewall-cmd \
  --zone=public \
  --add-service=http \
  --permanent
```

Esto evita esperar al próximo reload.

---

# 119. Copiar runtime a permanent

Después de probar reglas temporales:

```bash
sudo firewall-cmd --runtime-to-permanent
```

Advertencia:

Este comando guarda toda la configuración runtime actual.

Debe revisarse antes de utilizarlo.

---

# 120. Listar servicios disponibles

```bash
firewall-cmd --get-services
```

Buscar servicios relacionados con PostgreSQL:

```bash
firewall-cmd --get-services | tr ' ' '\n' | grep postgres
```

---

# 121. Consultar definición de un servicio

```bash
firewall-cmd \
  --info-service=http
```

Resultado conceptual:

```text
http
  ports: 80/tcp
```

---

# 122. Abrir SSH

```bash
sudo firewall-cmd \
  --zone=public \
  --add-service=ssh \
  --permanent
```

Recargar:

```bash
sudo firewall-cmd --reload
```

---

# 123. Abrir HTTP y HTTPS

```bash
sudo firewall-cmd \
  --zone=public \
  --add-service=http \
  --permanent
```

```bash
sudo firewall-cmd \
  --zone=public \
  --add-service=https \
  --permanent
```

```bash
sudo firewall-cmd --reload
```

---

# 124. Eliminar un servicio

```bash
sudo firewall-cmd \
  --zone=public \
  --remove-service=http \
  --permanent
```

Recargar:

```bash
sudo firewall-cmd --reload
```

---

# 125. Abrir un puerto

PostgreSQL:

```bash
sudo firewall-cmd \
  --zone=public \
  --add-port=5432/tcp \
  --permanent
```

Recargar:

```bash
sudo firewall-cmd --reload
```

---

# 126. Abrir un rango de puertos

```bash
sudo firewall-cmd \
  --zone=public \
  --add-port=8000-8010/tcp \
  --permanent
```

---

# 127. Eliminar un puerto

```bash
sudo firewall-cmd \
  --zone=public \
  --remove-port=5432/tcp \
  --permanent
```

Recargar:

```bash
sudo firewall-cmd --reload
```

---

# 128. Consultar servicios y puertos

```bash
firewall-cmd \
  --zone=public \
  --list-services
```

```bash
firewall-cmd \
  --zone=public \
  --list-ports
```

---

# 129. Asociar interfaz a zona

```bash
sudo firewall-cmd \
  --zone=internal \
  --change-interface=ens224 \
  --permanent
```

Recargar:

```bash
sudo firewall-cmd --reload
```

---

# 130. Verificar la zona de una interfaz

```bash
firewall-cmd --get-zone-of-interface=ens224
```

---

# 131. Asociar zona mediante NetworkManager

La asociación persistente de zona puede administrarse desde el perfil:

```bash
sudo nmcli connection modify red-aplicaciones \
  connection.zone internal
```

Reactivar:

```bash
sudo nmcli connection up red-aplicaciones
```

Verificar:

```bash
firewall-cmd --get-active-zones
```

Este método suele ser preferible cuando NetworkManager administra la interfaz.

---

# 132. Asociar una red de origen a una zona

```bash
sudo firewall-cmd \
  --zone=internal \
  --add-source=192.168.60.0/24 \
  --permanent
```

Recargar:

```bash
sudo firewall-cmd --reload
```

---

# 133. Eliminar una fuente

```bash
sudo firewall-cmd \
  --zone=internal \
  --remove-source=192.168.60.0/24 \
  --permanent
```

---

# 134. Rich Rules

Las Rich Rules permiten controles más detallados.

Pueden considerar:

- Familia IPv4 o IPv6.
- Dirección de origen.
- Dirección de destino.
- Servicio.
- Puerto.
- Protocolo.
- Registro.
- Límite.
- Acción.

---

# 135. Permitir SSH desde una red específica

```bash
sudo firewall-cmd \
  --zone=public \
  --add-rich-rule='
rule family="ipv4"
source address="192.168.50.0/24"
service name="ssh"
accept' \
  --permanent
```

Recargar:

```bash
sudo firewall-cmd --reload
```

---

# 136. Permitir PostgreSQL sólo desde un servidor

```bash
sudo firewall-cmd \
  --zone=public \
  --add-rich-rule='
rule family="ipv4"
source address="192.168.50.25/32"
port protocol="tcp" port="5432"
accept' \
  --permanent
```

---

# 137. Rechazar una dirección

```bash
sudo firewall-cmd \
  --zone=public \
  --add-rich-rule='
rule family="ipv4"
source address="192.168.50.200/32"
reject' \
  --permanent
```

---

# 138. Descartar tráfico

```bash
sudo firewall-cmd \
  --zone=public \
  --add-rich-rule='
rule family="ipv4"
source address="192.168.50.201/32"
drop' \
  --permanent
```

Diferencia:

| Acción | Comportamiento |
|---|---|
| `accept` | Permite |
| `reject` | Rechaza y responde |
| `drop` | Descarta silenciosamente |

---

# 139. Registrar intentos SSH

```bash
sudo firewall-cmd \
  --zone=public \
  --add-rich-rule='
rule family="ipv4"
service name="ssh"
log prefix="SSH_ATTEMPT " level="info" limit value="5/m"
accept' \
  --permanent
```

Recargar:

```bash
sudo firewall-cmd --reload
```

Consultar logs:

```bash
journalctl -k | grep SSH_ATTEMPT
```

---

# 140. Listar Rich Rules

```bash
firewall-cmd \
  --zone=public \
  --list-rich-rules
```

---

# 141. Eliminar Rich Rule

Debe suministrarse exactamente la misma regla:

```bash
sudo firewall-cmd \
  --zone=public \
  --remove-rich-rule='
rule family="ipv4"
source address="192.168.50.200/32"
reject' \
  --permanent
```

---

# 142. ICMP blocks

Consultar tipos ICMP:

```bash
firewall-cmd --get-icmptypes
```

Bloquear echo-request:

```bash
sudo firewall-cmd \
  --zone=public \
  --add-icmp-block=echo-request \
  --permanent
```

Recargar:

```bash
sudo firewall-cmd --reload
```

---

# 143. Eliminar bloqueo ICMP

```bash
sudo firewall-cmd \
  --zone=public \
  --remove-icmp-block=echo-request \
  --permanent
```

---

# 144. No bloquear ICMP indiscriminadamente

ICMP es utilizado para:

- Diagnóstico.
- Descubrimiento de MTU.
- Mensajes de destino inaccesible.
- Control de errores.
- IPv6 Neighbor Discovery.
- Operación normal de IPv6.

Bloquearlo completamente puede provocar problemas difíciles de diagnosticar.

---

# 145. Masquerading

Masquerading realiza una forma de NAT de origen.

Se utiliza cuando los clientes internos deben salir utilizando la dirección externa del servidor Linux.

Diagrama:

```text
Cliente interno

192.168.60.50

       │

       ▼

Servidor Linux

ens224: 192.168.60.1

ens192: 192.168.50.20

       │

       ▼

Internet

Origen visible: 192.168.50.20
```

---

# 146. Habilitar masquerading

Normalmente en la zona externa:

```bash
sudo firewall-cmd \
  --zone=external \
  --add-masquerade \
  --permanent
```

Recargar:

```bash
sudo firewall-cmd --reload
```

---

# 147. Verificar masquerading

```bash
firewall-cmd \
  --zone=external \
  --query-masquerade
```

Resultado esperado:

```text
yes
```

---

# 148. Deshabilitar masquerading

```bash
sudo firewall-cmd \
  --zone=external \
  --remove-masquerade \
  --permanent
```

---

# 149. Configuración de gateway básica

Interfaz externa:

```text
ens192
```

Interfaz interna:

```text
ens224
```

Asociar zonas:

```bash
sudo nmcli connection modify red-produccion \
  connection.zone external
```

```bash
sudo nmcli connection modify red-aplicaciones \
  connection.zone internal
```

Reactivar:

```bash
sudo nmcli connection up red-produccion
sudo nmcli connection up red-aplicaciones
```

Habilitar reenvío:

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

Habilitar masquerading:

```bash
sudo firewall-cmd \
  --zone=external \
  --add-masquerade \
  --permanent
```

Recargar:

```bash
sudo firewall-cmd --reload
```

---

# 150. Port forwarding

Port forwarding redirige tráfico recibido en un puerto hacia otro puerto o servidor.

Ejemplo:

```text
Cliente

↓

192.168.50.20:8080

↓

192.168.60.30:80
```

---

# 151. Redirigir puerto local

Redirigir `8080/tcp` hacia `80/tcp` en el mismo servidor:

```bash
sudo firewall-cmd \
  --zone=public \
  --add-forward-port=port=8080:proto=tcp:toport=80 \
  --permanent
```

Recargar:

```bash
sudo firewall-cmd --reload
```

---

# 152. Redirigir a otro servidor

```bash
sudo firewall-cmd \
  --zone=external \
  --add-forward-port=port=8080:proto=tcp:toport=80:toaddr=192.168.60.30 \
  --permanent
```

Recargar:

```bash
sudo firewall-cmd --reload
```

Para reenviar a otro servidor debe estar habilitado:

```text
net.ipv4.ip_forward = 1
```

---

# 153. Consultar port forwarding

```bash
firewall-cmd \
  --zone=external \
  --list-forward-ports
```

---

# 154. Eliminar port forwarding

```bash
sudo firewall-cmd \
  --zone=external \
  --remove-forward-port=port=8080:proto=tcp:toport=80:toaddr=192.168.60.30 \
  --permanent
```

Recargar:

```bash
sudo firewall-cmd --reload
```

---

# 155. Servicios personalizados de firewalld

Cuando una aplicación utiliza varios puertos, puede crearse un servicio personalizado.

Directorio:

```text
/etc/firewalld/services/
```

Ejemplo:

```bash
sudo tee /etc/firewalld/services/aplicacion.xml > /dev/null <<'EOF'
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>Aplicacion</short>
  <description>Servicio de aplicación empresarial</description>
  <port protocol="tcp" port="8080"/>
  <port protocol="tcp" port="8443"/>
</service>
EOF
```

Recargar completamente:

```bash
sudo firewall-cmd --reload
```

Habilitar servicio:

```bash
sudo firewall-cmd \
  --zone=public \
  --add-service=aplicacion \
  --permanent
```

---

# 156. Verificar servicio personalizado

```bash
firewall-cmd --info-service=aplicacion
```

---

# 157. Panic mode

Firewalld dispone de modo pánico.

Activar:

```bash
sudo firewall-cmd --panic-on
```

Esto bloquea el tráfico administrado.

Verificar:

```bash
firewall-cmd --query-panic
```

Desactivar:

```bash
sudo firewall-cmd --panic-off
```

Advertencia:

No actives Panic Mode en una sesión SSH remota, ya que probablemente perderás la conexión.

---

# 158. Firewall y servicios escuchando

Abrir un puerto en el Firewall no inicia la aplicación.

Ejemplo:

```bash
sudo firewall-cmd --add-service=http
```

no inicia Apache.

Debe verificarse:

```bash
systemctl status httpd
```

```bash
ss -tulpn | grep ':80'
```

Flujo correcto:

```text
Servicio instalado

↓

Servicio iniciado

↓

Puerto escuchando

↓

Firewall permite

↓

Cliente puede conectar
```

---

# 159. Firewall y SELinux

Aunque el Firewall permita un puerto, SELinux puede impedir que una aplicación lo utilice.

Ejemplo:

Apache en puerto 8080.

Consultar puertos permitidos:

```bash
sudo semanage port -l | grep http_port_t
```

Agregar:

```bash
sudo semanage port \
  -a \
  -t http_port_t \
  -p tcp \
  8080
```

En caso de existir:

```bash
sudo semanage port \
  -m \
  -t http_port_t \
  -p tcp \
  8080
```

También abrir en firewalld:

```bash
sudo firewall-cmd \
  --zone=public \
  --add-port=8080/tcp \
  --permanent
```

---

# 160. Troubleshooting avanzado por capas

```text
1. Hardware o enlace

↓

2. Interfaz

↓

3. Perfil NetworkManager

↓

4. Dirección IP

↓

5. Máscara

↓

6. ARP o Neighbor Discovery

↓

7. Tabla de rutas

↓

8. Gateway

↓

9. DNS

↓

10. Firewall local

↓

11. SELinux

↓

12. Servicio escuchando

↓

13. Firewall remoto

↓

14. Aplicación
```

---

# 161. Diagnóstico de enlace físico

```bash
ip link show ens192
```

```bash
sudo ethtool ens192
```

Buscar:

```text
Link detected: yes
```

Si aparece:

```text
Link detected: no
```

revisar:

- Cable.
- Switch.
- Puerto.
- NIC virtual.
- Driver.
- Configuración de hipervisor.

---

# 162. Diagnóstico de NetworkManager

```bash
nmcli general status
```

```bash
nmcli device status
```

```bash
nmcli connection show --active
```

```bash
journalctl -u NetworkManager -b
```

---

# 163. Diagnóstico de IP

```bash
ip addr show ens192
```

Verificar:

- Dirección.
- Prefijo.
- Estado.
- Dirección duplicada.
- Dirección secundaria inesperada.

---

# 164. Diagnóstico de ARP

```bash
ip neigh show
```

Probar gateway:

```bash
arping -I ens192 192.168.50.1
```

Detectar duplicado:

```bash
sudo arping -D -I ens192 192.168.50.20
```

---

# 165. Diagnóstico de rutas

```bash
ip route
```

```bash
ip route get 8.8.8.8
```

```bash
ip route get 10.20.10.50
```

Verificar:

- Gateway.
- Interfaz.
- Métrica.
- Dirección de origen.
- Ruta específica.
- Rutas duplicadas.

---

# 166. Diagnóstico de DNS

```bash
getent hosts example.com
```

```bash
dig example.com
```

```bash
dig @192.168.50.10 example.com
```

```bash
cat /etc/resolv.conf
```

```bash
nmcli device show ens192 | grep DNS
```

---

# 167. Diagnóstico de puerto local

```bash
sudo ss -tulpn
```

```bash
sudo lsof -i :5432
```

---

# 168. Diagnóstico de puerto remoto

```bash
nc -vz 192.168.50.31 5432
```

```bash
curl -v http://192.168.50.21
```

---

# 169. Diagnóstico con tcpdump

En el servidor:

```bash
sudo tcpdump -nn -i ens192 port 5432
```

Desde el cliente:

```bash
nc -vz 192.168.50.31 5432
```

Interpretación:

```text
No llegan paquetes

↓

Problema antes del servidor
```

```text
Llega SYN y servidor responde RST

↓

Puerto cerrado o servicio no escucha
```

```text
Llega SYN y no sale respuesta

↓

Firewall local o problema de pila
```

---

# 170. Diagnóstico de Firewall

```bash
firewall-cmd --state
```

```bash
firewall-cmd --get-active-zones
```

```bash
firewall-cmd --list-all
```

```bash
firewall-cmd --list-all-zones
```

```bash
firewall-cmd --list-rich-rules
```

---

# 171. Diagnóstico con nftables

Firewalld utiliza normalmente `nftables` como backend moderno.

Consultar reglas:

```bash
sudo nft list ruleset
```

No se recomienda modificar manualmente las reglas generadas por firewalld, porque pueden sobrescribirse.

---

# 172. Logs del Kernel

```bash
journalctl -k
```

Filtrar interfaces:

```bash
journalctl -k | grep -Ei 'ens192|link|network'
```

---

# 173. Estadísticas de interfaz

```bash
ip -s link show ens192
```

Consultar repetidamente:

```bash
watch -n 2 'ip -s link show ens192'
```

Revisar crecimiento de:

```text
errors
dropped
overruns
carrier
collisions
```

---

# 174. Diagnóstico de MTU

Probar paquetes sin fragmentación:

```bash
ping -M do -s 1472 -c 4 192.168.50.1
```

Para Ethernet con MTU 1500:

```text
1472 bytes de datos

+

28 bytes de encabezados IPv4 e ICMP

=

1500 bytes
```

Reducir tamaño:

```bash
ping -M do -s 1400 -c 4 destino
```

---

# 175. Problema de ruta asimétrica

Ejemplo:

```text
Solicitud entra por ens192

↓

Respuesta sale por ens224
```

Esto puede provocar bloqueo por:

- Firewalls.
- Reverse path filtering.
- Routers.
- Sesiones con estado.

Consultar rutas:

```bash
ip route get direccion_cliente
```

---

# 176. Reverse path filtering

Consultar:

```bash
sysctl net.ipv4.conf.all.rp_filter
```

Valores frecuentes:

| Valor | Significado |
|---:|---|
| `0` | Deshabilitado |
| `1` | Modo estricto |
| `2` | Modo flexible |

No debe modificarse sin comprender la topología y las políticas de seguridad.

---

# 177. Problema de conexión lenta

Revisar:

- DNS inverso.
- Pérdida de paquetes.
- MTU.
- Saturación.
- Retransmisiones.
- Negociación de enlace.
- Firewall.
- Aplicación.
- Discos o CPU.

Comandos:

```bash
ss -s
```

```bash
ip -s link
```

```bash
tcpdump
```

```bash
ethtool
```

```bash
dig
```

---

# 178. Error: conexión no compatible con el dispositivo

Mensaje conceptual:

```text
Connection activation failed:
No suitable device found
```

Revisar:

```bash
nmcli connection show perfil
```

```bash
nmcli device status
```

Verificar:

```text
connection.interface-name
```

Corregir:

```bash
sudo nmcli connection modify perfil \
  connection.interface-name ens192
```

---

# 179. Error: interfaz ya pertenece a otro controlador

Una interfaz no puede pertenecer simultáneamente a:

- Bond.
- Bridge.
- Conexión Ethernet independiente.

Revisar:

```bash
nmcli connection show
```

Eliminar o desactivar perfiles conflictivos.

---

# 180. Error: VLAN no transmite

Verificar:

```bash
ip -d link show ens192.60
```

```bash
nmcli connection show vlan60
```

```bash
tcpdump -e -nn -i ens192 vlan 60
```

También revisar el switch.

---

# 181. Error: bond no realiza failover

Revisar:

```bash
cat /proc/net/bonding/bond0
```

Validar:

- `MII Status`.
- `Currently Active Slave`.
- `miimon`.
- Estado físico.
- Modo.
- Perfil de puertos.

---

# 182. Error: bridge sin conectividad

Verificar:

```bash
ip addr show br0
```

La IP debe estar en el bridge, no en el puerto físico.

Consultar:

```bash
bridge link
```

```bash
nmcli connection show
```

---

# 183. Error: masquerading no funciona

Revisar:

```bash
sysctl net.ipv4.ip_forward
```

Debe ser:

```text
1
```

Verificar:

```bash
firewall-cmd \
  --zone=external \
  --query-masquerade
```

También revisar:

- Zona de interfaz externa.
- Zona interna.
- Gateway de clientes.
- Rutas.
- DNS.
- Reglas adicionales.

---

# 184. Error: regla permanente no funciona

Posible causa:

```text
Se agregó con --permanent, pero no se ejecutó --reload
```

Aplicar:

```bash
sudo firewall-cmd --reload
```

Verificar:

```bash
firewall-cmd --list-all
```

---

# 185. Error: cambio runtime se perdió

Los cambios sin:

```text
--permanent
```

se pierden después de recargar o reiniciar.

Guardar:

```bash
sudo firewall-cmd --runtime-to-permanent
```

Sólo después de revisar toda la configuración runtime.

---

# 186. Laboratorio 1 — IPv6 estático

Configura:

| Parámetro | Valor |
|---|---|
| Perfil | `red-ipv6` |
| Interfaz | La disponible |
| Dirección | `2001:db8:100::50/64` |
| Gateway | `2001:db8:100::1` |
| DNS | `2001:4860:4860::8888` |

Valida:

```bash
ip -6 addr
ip -6 route
nmcli connection show red-ipv6
```

---

# 187. Laboratorio 2 — Dirección secundaria

Agrega una segunda dirección a una conexión existente:

```text
192.168.100.51/24
```

Verifica:

```bash
ip -4 addr show
```

Luego elimínala sin afectar la dirección principal.

---

# 188. Laboratorio 3 — Dos interfaces

Configura:

```text
ens192: 192.168.100.50/24 con gateway

ens224: 192.168.200.50/24 sin gateway
```

Configura en la segunda conexión:

```text
ipv4.never-default yes
```

Verifica:

```bash
ip route
```

Sólo debe existir una ruta predeterminada principal.

---

# 189. Laboratorio 4 — Métricas

Configura dos rutas predeterminadas:

```text
Principal: métrica 100

Respaldo: métrica 200
```

Valida la ruta usada:

```bash
ip route get 8.8.8.8
```

---

# 190. Laboratorio 5 — nmtui

Utiliza `nmtui` para:

- Crear una conexión.
- Asignar IPv4 estática.
- Configurar DNS.
- Configurar autoconexión.
- Cambiar hostname.
- Activar el perfil.

Documenta las pantallas y resultados.

---

# 191. Laboratorio 6 — Bond active-backup

Crea:

```text
bond0
```

Utilizando dos interfaces.

Configura:

```text
mode=active-backup

miimon=100
```

Asigna una dirección IPv4.

Valida:

```bash
cat /proc/net/bonding/bond0
```

Simula la caída de una interfaz y verifica failover.

---

# 192. Laboratorio 7 — Bridge

Crea:

```text
br0
```

Agrega una interfaz física.

Asigna la dirección IP al bridge.

Valida:

```bash
ip addr show br0
bridge link
nmcli device status
```

---

# 193. Laboratorio 8 — VLAN

Crea:

```text
VLAN ID: 200

Interfaz: ens192.200

Dirección: 192.168.200.20/24
```

Valida:

```bash
ip -d link show ens192.200
```

---

# 194. Laboratorio 9 — Reenvío IP

Configura un servidor con:

```text
Interfaz externa: ens192

Interfaz interna: ens224
```

Habilita:

```text
net.ipv4.ip_forward = 1
```

Verifica persistencia después de reiniciar.

---

# 195. Laboratorio 10 — Captura ICMP

Ejecuta:

```bash
sudo tcpdump -nn -i ens192 icmp
```

Desde otra terminal realiza un `ping`.

Identifica:

- Echo request.
- Echo reply.
- Dirección origen.
- Dirección destino.

---

# 196. Laboratorio 11 — Captura DNS

Ejecuta:

```bash
sudo tcpdump -nn -i ens192 port 53
```

Luego:

```bash
dig example.com
```

Identifica:

- Consulta.
- Respuesta.
- DNS utilizado.
- Protocolo UDP o TCP.

---

# 197. Laboratorio 12 — Captura TCP

Captura tráfico SSH:

```bash
sudo tcpdump -nn -i ens192 port 22
```

Inicia una nueva conexión SSH.

Identifica:

```text
SYN

SYN-ACK

ACK
```

---

# 198. Laboratorio 13 — Firewalld básico

Configura zona `public`.

Permite:

```text
ssh

http

https
```

Valida:

```bash
firewall-cmd --list-all
```

---

# 199. Laboratorio 14 — Rich Rule

Permite PostgreSQL únicamente desde:

```text
192.168.100.25
```

Puerto:

```text
5432/tcp
```

Verifica desde:

- Host permitido.
- Host no permitido.

---

# 200. Laboratorio 15 — NAT

Configura:

```text
ens192 = zona external

ens224 = zona internal
```

Habilita:

- IP forwarding.
- Masquerading.

Configura un cliente interno para usar el servidor Linux como gateway.

Valida acceso externo.

---

# 201. Laboratorio 16 — Port forwarding

Redirige:

```text
Servidor Linux:8080
```

hacia:

```text
Servidor Web interno:80
```

Valida:

```bash
curl http://direccion-linux:8080
```

---

# 202. Laboratorio empresarial final

## Escenario

La empresa necesita convertir un servidor Linux en gateway seguro y host de virtualización.

Interfaces:

| Interfaz | Función | Dirección |
|---|---|---|
| `ens192` | Red externa | `192.168.50.20/24` |
| `ens224` | Red interna | `192.168.60.1/24` |
| `ens256` | Red de respaldo | `192.168.80.20/24` |

Requisitos:

```text
□ Configurar hostname gateway01.empresa.local

□ Configurar ens192 con gateway 192.168.50.1

□ Configurar ens224 sin gateway

□ Configurar ens256 sin gateway

□ Evitar rutas predeterminadas en interfaces internas

□ Configurar DNS en la interfaz externa

□ Habilitar IPv4 forwarding

□ Asociar ens192 a la zona external

□ Asociar ens224 a la zona internal

□ Asociar ens256 a la zona work

□ Habilitar masquerading en external

□ Permitir SSH únicamente desde 192.168.50.0/24

□ Permitir DNS desde la red interna

□ Crear port forwarding de 8080 hacia 192.168.60.30:80

□ Crear VLAN 70 sobre ens224

□ Configurar 192.168.70.1/24 en VLAN 70

□ Crear bridge br0 para virtualización

□ Capturar tráfico de validación con tcpdump

□ Comprobar persistencia después de reiniciar

□ Generar reporte completo
```

---

# 203. Configuración propuesta de interfaz externa

```bash
sudo nmcli connection add \
  type ethernet \
  ifname ens192 \
  con-name red-externa \
  ipv4.method manual \
  ipv4.addresses 192.168.50.20/24 \
  ipv4.gateway 192.168.50.1 \
  ipv4.dns "192.168.50.10 8.8.8.8" \
  ipv4.dns-search empresa.local \
  connection.autoconnect yes
```

Asignar zona:

```bash
sudo nmcli connection modify red-externa \
  connection.zone external
```

---

# 204. Configuración propuesta de interfaz interna

```bash
sudo nmcli connection add \
  type ethernet \
  ifname ens224 \
  con-name red-interna \
  ipv4.method manual \
  ipv4.addresses 192.168.60.1/24 \
  ipv4.never-default yes \
  connection.autoconnect yes
```

Asignar zona:

```bash
sudo nmcli connection modify red-interna \
  connection.zone internal
```

---

# 205. Configuración propuesta de respaldo

```bash
sudo nmcli connection add \
  type ethernet \
  ifname ens256 \
  con-name red-respaldo \
  ipv4.method manual \
  ipv4.addresses 192.168.80.20/24 \
  ipv4.never-default yes \
  connection.autoconnect yes
```

Asignar zona:

```bash
sudo nmcli connection modify red-respaldo \
  connection.zone work
```

---

# 206. Configurar forwarding persistente

```bash
sudo tee /etc/sysctl.d/90-gateway.conf > /dev/null <<'EOF'
net.ipv4.ip_forward = 1
EOF
```

Aplicar:

```bash
sudo sysctl --system
```

---

# 207. Habilitar masquerading

```bash
sudo firewall-cmd \
  --zone=external \
  --add-masquerade \
  --permanent
```

---

# 208. Restringir SSH

Primero verificar que SSH está permitido durante la configuración.

Después agregar Rich Rule:

```bash
sudo firewall-cmd \
  --zone=external \
  --add-rich-rule='
rule family="ipv4"
source address="192.168.50.0/24"
service name="ssh"
accept' \
  --permanent
```

Debe evaluarse cuidadosamente la existencia de reglas generales que también permitan SSH.

---

# 209. Port forwarding del escenario

```bash
sudo firewall-cmd \
  --zone=external \
  --add-forward-port=port=8080:proto=tcp:toport=80:toaddr=192.168.60.30 \
  --permanent
```

Recargar:

```bash
sudo firewall-cmd --reload
```

---

# 210. Crear VLAN 70

```bash
sudo nmcli connection add \
  type vlan \
  con-name vlan70 \
  ifname ens224.70 \
  dev ens224 \
  id 70 \
  ipv4.method manual \
  ipv4.addresses 192.168.70.1/24 \
  ipv4.never-default yes \
  connection.autoconnect yes
```

---

# 211. Reporte empresarial

```bash
#!/bin/bash

REPORT="/root/reporte-red-avanzada-$(date +%Y%m%d-%H%M%S).txt"

{
  echo "============================================================"
  echo "REPORTE DE RED AVANZADA"
  echo "============================================================"

  echo
  echo "Fecha:"
  date

  echo
  echo "Hostname:"
  hostnamectl

  echo
  echo "NetworkManager:"
  systemctl is-active NetworkManager

  echo
  echo "Dispositivos:"
  nmcli device status

  echo
  echo "Conexiones activas:"
  nmcli connection show --active

  echo
  echo "Direcciones IPv4:"
  ip -4 addr

  echo
  echo "Direcciones IPv6:"
  ip -6 addr

  echo
  echo "Rutas IPv4:"
  ip route

  echo
  echo "Rutas IPv6:"
  ip -6 route

  echo
  echo "Reglas de enrutamiento:"
  ip rule

  echo
  echo "Reenvío IPv4:"
  sysctl net.ipv4.ip_forward

  echo
  echo "Reenvío IPv6:"
  sysctl net.ipv6.conf.all.forwarding

  echo
  echo "Zonas activas:"
  firewall-cmd --get-active-zones

  echo
  echo "Configuración completa de firewalld:"
  firewall-cmd --list-all-zones

  echo
  echo "Sockets escuchando:"
  ss -tulpn

  echo
  echo "Estadísticas de interfaces:"
  ip -s link

  echo
  echo "Tabla de vecinos:"
  ip neigh

} | tee "$REPORT"

echo "Reporte guardado en: $REPORT"
```

Guardar como:

```text
/usr/local/sbin/reporte-red-avanzada.sh
```

Asignar permisos:

```bash
sudo chmod 750 /usr/local/sbin/reporte-red-avanzada.sh
```

Ejecutar:

```bash
sudo /usr/local/sbin/reporte-red-avanzada.sh
```

---

# 212. Script de validación avanzada

```bash
#!/bin/bash

set -u

EXTERNAL_IF="ens192"
INTERNAL_IF="ens224"
EXTERNAL_IP="192.168.50.20/24"
INTERNAL_IP="192.168.60.1/24"
GATEWAY="192.168.50.1"

errors=0

check_ok() {
  echo "OK: $1"
}

check_error() {
  echo "ERROR: $1"
  errors=$((errors + 1))
}

echo "=== VALIDACIÓN AVANZADA DE RED ==="

if systemctl is-active --quiet NetworkManager; then
  check_ok "NetworkManager está activo"
else
  check_error "NetworkManager no está activo"
fi

if ip -4 addr show "$EXTERNAL_IF" | grep -q "$EXTERNAL_IP"; then
  check_ok "$EXTERNAL_IF tiene $EXTERNAL_IP"
else
  check_error "$EXTERNAL_IF no tiene $EXTERNAL_IP"
fi

if ip -4 addr show "$INTERNAL_IF" | grep -q "$INTERNAL_IP"; then
  check_ok "$INTERNAL_IF tiene $INTERNAL_IP"
else
  check_error "$INTERNAL_IF no tiene $INTERNAL_IP"
fi

if ip route | grep -q "default via $GATEWAY"; then
  check_ok "Gateway predeterminado correcto"
else
  check_error "Gateway predeterminado incorrecto"
fi

if [ "$(sysctl -n net.ipv4.ip_forward)" = "1" ]; then
  check_ok "IPv4 forwarding habilitado"
else
  check_error "IPv4 forwarding deshabilitado"
fi

if firewall-cmd --state 2>/dev/null | grep -q running; then
  check_ok "firewalld está activo"
else
  check_error "firewalld no está activo"
fi

if firewall-cmd \
  --zone=external \
  --query-masquerade 2>/dev/null | grep -q yes; then
  check_ok "Masquerading habilitado"
else
  check_error "Masquerading no está habilitado"
fi

if ping -c 2 -W 2 "$GATEWAY" >/dev/null 2>&1; then
  check_ok "Gateway responde"
else
  check_error "Gateway no responde"
fi

if getent hosts example.com >/dev/null 2>&1; then
  check_ok "Resolución DNS funcional"
else
  check_error "Resolución DNS falló"
fi

echo
echo "Errores encontrados: $errors"

if [ "$errors" -eq 0 ]; then
  echo "VALIDACIÓN COMPLETADA CORRECTAMENTE"
  exit 0
else
  echo "VALIDACIÓN FINALIZADA CON ERRORES"
  exit 1
fi
```

---

# 213. Buenas prácticas avanzadas

- Utilizar nombres descriptivos para conexiones.
- Documentar cada interfaz y propósito.
- Evitar varios gateways sin métricas.
- Configurar `never-default` en redes internas.
- Mantener una única ruta principal cuando sea posible.
- Probar configuraciones desde consola.
- No modificar interfaces remotas sin plan de reversión.
- Utilizar bonding para redundancia.
- Coordinar LACP con administradores de switches.
- Colocar la dirección IP sobre el bridge, no sobre el puerto físico.
- Confirmar configuración del trunk antes de crear VLAN.
- No habilitar forwarding sin Firewall.
- Utilizar zonas según el nivel de confianza.
- Aplicar mínimo privilegio a servicios y puertos.
- Preferir servicios de firewalld sobre puertos cuando existan.
- Restringir puertos sensibles por dirección de origen.
- Probar reglas runtime antes de hacerlas permanentes.
- Revisar antes de ejecutar `runtime-to-permanent`.
- No usar zona `trusted` por comodidad.
- Evitar bloquear ICMP completamente.
- Capturar sólo el tráfico necesario.
- Limitar tamaño y duración de capturas.
- Proteger archivos `.pcap`.
- Eliminar capturas con información sensible.
- Validar servicio, Firewall y SELinux.
- Revisar métricas y rutas tras reinicios.
- Guardar evidencia de la configuración.
- Automatizar validaciones.
- Mantener una copia de perfiles de NetworkManager.
- Probar failover periódicamente.
- Documentar NAT y port forwarding.
- Auditar Rich Rules.
- Revisar logs del Kernel y NetworkManager.

---

# 214. Errores comunes

## Error 1: configurar IP en un puerto de bridge

Incorrecto:

```text
ens192 tiene la IP

br0 no tiene IP
```

Correcto:

```text
br0 tiene la IP

ens192 es puerto del bridge
```

---

## Error 2: interfaces del bond con IP propia

Las direcciones deben configurarse sobre:

```text
bond0
```

No sobre los puertos individuales.

---

## Error 3: usar LACP sin configurar el switch

El modo:

```text
802.3ad
```

requiere configuración del switch.

---

## Error 4: varias rutas predeterminadas iguales

Puede provocar enrutamiento asimétrico.

Configurar:

- Métricas.
- `never-default`.
- Policy routing cuando sea necesario.

---

## Error 5: crear VLAN sin trunk

La interfaz VLAN puede aparecer activa, pero no habrá comunicación real.

---

## Error 6: habilitar forwarding sin Firewall

El servidor puede reenviar tráfico no autorizado.

---

## Error 7: permitir puerto sin servicio escuchando

Firewalld no inicia aplicaciones.

Verificar:

```bash
ss -tulpn
```

---

## Error 8: configurar sólo permanent

La regla no estará activa hasta ejecutar:

```bash
firewall-cmd --reload
```

---

## Error 9: configurar sólo runtime

La regla se perderá tras recargar o reiniciar.

---

## Error 10: cerrar SSH antes de validar

Mantén una sesión activa hasta confirmar:

- Nueva dirección.
- Ruta.
- Firewall.
- SSH.
- Sudo.
- Persistencia.

---

## Error 11: capturar sin filtro

Puede generar:

- Archivos enormes.
- Alto uso de disco.
- Información sensible.
- Dificultad de análisis.

---

## Error 12: tcpdump dentro de la misma sesión SSH

La captura puede llenarse con tráfico de administración.

Excluir:

```bash
not port 22
```

---

## Error 13: deshabilitar IPv6 sin análisis

Puede afectar servicios y aplicaciones.

---

## Error 14: asignar zona únicamente con firewall-cmd

Si NetworkManager administra la interfaz, la zona puede reasignarse según el perfil.

Configurar también:

```bash
nmcli connection modify perfil connection.zone zona
```

---

## Error 15: utilizar trusted para resolver problemas

La zona `trusted` permite todo el tráfico.

No debe utilizarse como solución permanente.

---

# 215. Checklist de IPv6

```text
□ IPv6 requerido por la organización

□ Método configurado

□ Dirección correcta

□ Prefijo correcto

□ Gateway correcto

□ DNS IPv6 configurado

□ Ruta IPv6 visible

□ Link-local disponible

□ Conectividad probada

□ Firewall IPv6 validado
```

---

# 216. Checklist de bonding

```text
□ Interfaces disponibles

□ Sin IP en puertos individuales

□ Bond creado

□ Modo definido

□ miimon configurado

□ Puertos agregados

□ IP asignada a bond0

□ Gateway correcto

□ Failover probado

□ Configuración persistente
```

---

# 217. Checklist de bridge

```text
□ Bridge creado

□ Interfaz física agregada

□ IP asignada al bridge

□ Puerto físico sin IP independiente

□ STP revisado

□ Conectividad validada

□ Máquinas virtuales probadas

□ Persistencia verificada
```

---

# 218. Checklist de VLAN

```text
□ VLAN aprobada

□ ID correcto

□ Interfaz padre correcta

□ Puerto del switch configurado

□ Perfil VLAN creado

□ IP correcta

□ Gateway evaluado

□ never-default configurado cuando corresponde

□ Tráfico etiquetado verificado

□ Persistencia validada
```

---

# 219. Checklist de Firewall

```text
□ firewalld activo

□ Zona predeterminada correcta

□ Interfaces en zonas correctas

□ Servicios mínimos habilitados

□ Puertos mínimos habilitados

□ Rich Rules revisadas

□ Runtime validado

□ Permanent validado

□ Configuración recargada

□ Servicios escuchando

□ SELinux validado

□ Acceso remoto comprobado

□ Reglas documentadas
```

---

# 220. Desafío final de la Fase 2

Configura un servidor Linux con los siguientes requisitos:

```text
Hostname:
router01.lab.local

Interfaz externa:
ens192
192.168.90.50/24
Gateway 192.168.90.1
DNS 192.168.90.10 y 1.1.1.1
Zona external

Interfaz interna:
ens224
192.168.100.1/24
Sin gateway
Zona internal

Interfaz de respaldo:
ens256
192.168.110.50/24
Sin gateway
Zona work

VLAN:
ID 120
Interfaz ens224.120
Dirección 192.168.120.1/24

IPv6:
2001:db8:90::50/64
Gateway 2001:db8:90::1

Forwarding:
IPv4 habilitado persistentemente

Firewall:
Masquerading en external
SSH sólo desde 192.168.90.0/24
HTTP desde cualquier origen
PostgreSQL sólo desde 192.168.100.25
Port forwarding 8080 hacia 192.168.100.30:80

Captura:
Guardar evidencia de DNS, ICMP y HTTP
```

Debes demostrar:

```text
□ NetworkManager activo

□ Todas las conexiones persistentes

□ Una única ruta predeterminada principal

□ Métricas correctas

□ IPv4 funcional

□ IPv6 funcional

□ VLAN activa

□ Forwarding habilitado

□ Zonas correctas

□ Masquerading activo

□ Rich Rules correctas

□ Port forwarding funcional

□ Servicios escuchando

□ SELinux revisado

□ Capturas generadas

□ Configuración persistente después de reiniciar

□ Reporte completo guardado
```

---

# 221. Criterios de evaluación

| Criterio | Puntos |
|---|---:|
| Configuración de múltiples interfaces | 10 |
| Configuración IPv6 | 8 |
| Rutas y métricas | 8 |
| Bonding o redundancia | 8 |
| Bridge | 8 |
| VLAN | 10 |
| Reenvío IP | 6 |
| Zonas de firewalld | 8 |
| Servicios y puertos | 8 |
| Rich Rules | 8 |
| Masquerading | 6 |
| Port forwarding | 6 |
| Captura con tcpdump | 6 |
| Validación y persistencia | 5 |
| Documentación | 3 |
| **Total** | **100** |

---

# 222. Preguntas de repaso

1. ¿Cuántos bits contiene una dirección IPv6?
2. ¿Qué función cumple una dirección link-local?
3. ¿Qué prefijo identifica normalmente una dirección link-local?
4. ¿Cómo se muestra la tabla de rutas IPv6?
5. ¿Cómo se configura IPv6 estático con `nmcli`?
6. ¿Cómo se agrega una segunda dirección IP sin reemplazar la primera?
7. ¿Para qué sirve `ipv4.never-default`?
8. ¿Qué significa una métrica de ruta menor?
9. ¿Qué comando muestra la ruta exacta hacia un destino?
10. ¿Para qué sirve `nmtui`?
11. ¿Qué diferencia existe entre bonding y bridge?
12. ¿Qué modo de bonding ofrece failover simple?
13. ¿Qué requiere el modo `802.3ad`?
14. ¿Dónde debe configurarse la IP de un bond?
15. ¿Dónde debe configurarse la IP de un bridge?
16. ¿Qué función cumple STP?
17. ¿Qué es una VLAN?
18. ¿Qué debe configurarse en el switch para utilizar VLAN etiquetada?
19. ¿Cómo se verifica el ID de una VLAN?
20. ¿Qué parámetro del Kernel habilita reenvío IPv4?
21. ¿Por qué forwarding debe combinarse con Firewall?
22. ¿Para qué sirve `tcpdump`?
23. ¿Qué opción evita resolución de nombres?
24. ¿Cómo se filtra tráfico por puerto?
25. ¿Cómo se guarda una captura en formato PCAP?
26. ¿Qué representa el handshake SYN, SYN-ACK y ACK?
27. ¿Qué puede indicar un paquete RST?
28. ¿Qué diferencia existe entre configuración runtime y permanent?
29. ¿Qué función cumple una zona de firewalld?
30. ¿Cómo se asigna una interfaz a una zona mediante NetworkManager?
31. ¿Qué es una Rich Rule?
32. ¿Qué diferencia existe entre `reject` y `drop`?
33. ¿Qué función cumple masquerading?
34. ¿Qué parámetro debe habilitarse para reenviar tráfico hacia otro servidor?
35. ¿Qué es port forwarding?
36. ¿Por qué abrir un puerto no inicia una aplicación?
37. ¿Cómo interactúan Firewall y SELinux?
38. ¿Qué comando muestra reglas nftables?
39. ¿Por qué no debe utilizarse `trusted` como solución rápida?
40. ¿Qué procedimiento seguirías para diagnosticar un puerto inaccesible?

---

# 223. Resumen de la Fase 2

En esta fase desarrollamos la administración avanzada de redes Linux.

Aprendimos a:

- Comprender IPv6.
- Identificar direcciones link-local y globales.
- Configurar IPv6 automático y estático.
- Administrar varias direcciones IP.
- Configurar múltiples interfaces.
- Evitar rutas predeterminadas conflictivas.
- Administrar métricas.
- Consultar policy routing.
- Utilizar `nmtui`.
- Crear bonds.
- Configurar failover.
- Crear bridges.
- Configurar VLAN.
- Habilitar reenvío IP.
- Convertir Linux en un gateway.
- Capturar tráfico con `tcpdump`.
- Analizar ARP, ICMP, DNS, TCP y UDP.
- Interpretar el handshake TCP.
- Administrar `firewalld`.
- Configurar zonas.
- Abrir servicios y puertos.
- Crear Rich Rules.
- Restringir tráfico por origen.
- Configurar masquerading.
- Implementar port forwarding.
- Integrar Firewall con NetworkManager.
- Revisar interacción con SELinux.
- Diagnosticar problemas avanzados.
- Automatizar validaciones.
- Generar evidencias de configuración.

Estas competencias fortalecen la preparación para RHCSA y permiten administrar redes Linux empresariales de forma segura, persistente y estructurada.

---

# Próxima fase

## Fase 3 — Laboratorio final integral de redes

En la siguiente fase se desarrollará un escenario completo que integrará:

- Configuración desde cero.
- IPv4 e IPv6.
- Varias interfaces.
- Bonding.
- Bridge.
- VLAN.
- Rutas y métricas.
- DNS.
- Reenvío.
- Firewalld.
- Rich Rules.
- NAT.
- Port forwarding.
- SELinux.
- Captura y análisis de paquetes.
- Scripts de auditoría.
- Respaldo y restauración de perfiles.
- Plan de reversión.
- Troubleshooting empresarial.
- Simulación práctica final tipo RHCSA.


----

# 56. Laboratorio Final Integral de Redes — Fase 3

> **Módulo 6 — Redes en Red Hat Enterprise Linux**
>
> **Archivo:** `56-laboratorio-final-redes.md`
>
> **Nivel:** RHCSA
>
> **Fase 3:** Proyecto Final Integrador
>
> **Duración estimada:** 8 a 12 horas
>
> **Objetivo:** Integrar absolutamente todos los conocimientos adquiridos en Redes durante el módulo RHCSA mediante un escenario empresarial completamente funcional.

---

# Objetivos

Al finalizar este laboratorio el estudiante será capaz de:

- Implementar una infraestructura completa desde cero.
- Configurar múltiples interfaces.
- Configurar IPv4 e IPv6.
- Configurar NetworkManager.
- Utilizar nmcli y nmtui.
- Crear VLAN.
- Crear Bridge.
- Crear Bond.
- Configurar métricas.
- Administrar múltiples rutas.
- Configurar DNS.
- Implementar Firewalld.
- Crear Rich Rules.
- Configurar NAT.
- Implementar Masquerading.
- Configurar Port Forwarding.
- Habilitar IP Forwarding.
- Analizar tráfico con tcpdump.
- Diagnosticar fallos de red.
- Automatizar verificaciones.
- Documentar toda la configuración.

---

# Escenario Empresarial

La empresa **TechCloud International** está migrando toda su infraestructura hacia Linux.

El nuevo servidor será el encargado de funcionar como:

- Gateway
- Firewall
- Router
- Servidor de Virtualización
- Servidor de Aplicaciones
- Concentrador de Redes

Toda la infraestructura deberá configurarse únicamente utilizando herramientas nativas de Linux.

---

# Topología

```text
                         INTERNET

                             |

                     Router ISP

                   192.168.10.1

                             |

                    ----------------

                    |              |

                  ens192        Gateway

                    |

            Linux Enterprise Server

    --------------------------------------------------

       ens192

       ens224

       ens256

       bond0

       br0

       VLAN100

       VLAN200

       VLAN300

-----------------------------------------------------

          |           |            |

      Red Apps     Base Datos     Backups
```

---

# Información de direccionamiento

| Interfaz | Dirección |
|-----------|-----------|
| ens192 | 192.168.10.20/24 |
| ens224 | 172.16.10.1/24 |
| ens256 | 172.16.20.1/24 |
| bond0 | 10.10.10.20/24 |
| br0 | 192.168.100.1/24 |
| VLAN100 | 10.100.0.1/24 |
| VLAN200 | 10.200.0.1/24 |
| VLAN300 | 10.300.0.1/24 |

IPv6

| Interfaz | Dirección |
|-----------|-----------|
| ens192 | 2001:db8:10::20/64 |
| ens224 | 2001:db8:20::1/64 |

---

# Requerimientos

El servidor deberá cumplir con todos los siguientes puntos.

## Parte 1

Hostname

```text
gateway01.techcloud.local
```

Dominio

```text
techcloud.local
```

DNS

```text
8.8.8.8

1.1.1.1
```

---

## Parte 2

Configurar IPv4 estático.

Configurar IPv6.

Configurar DNS.

Configurar búsqueda DNS.

Configurar gateway.

---

## Parte 3

Crear las siguientes VLAN.

```text
100

200

300
```

---

## Parte 4

Crear un Bridge.

```text
br0
```

---

## Parte 5

Crear Bond.

Modo

```text
active-backup
```

---

## Parte 6

Configurar Firewall.

Crear zonas.

External

Internal

Work

DMZ

---

## Parte 7

Permitir únicamente

SSH

HTTPS

DNS

ICMP

---

## Parte 8

Bloquear completamente

FTP

TELNET

SMTP

---

## Parte 9

Crear Rich Rules.

SSH únicamente

```text
192.168.10.0/24
```

PostgreSQL

```text
172.16.10.25
```

HTTP

Toda la red.

---

## Parte 10

Configurar NAT.

Toda la red interna deberá navegar por Internet.

---

## Parte 11

Port Forward

```text
8080

↓

Servidor Web

↓

172.16.10.100

↓

80
```

---

## Parte 12

Capturar tráfico.

Guardar

```text
DNS

SSH

HTTP

ICMP
```

---

## Parte 13

Generar documentación.

---

# Restricciones

No utilizar herramientas gráficas.

Todo debe realizarse mediante:

```text
nmcli

firewall-cmd

ip

bridge

tcpdump

sysctl
```

---

# Fase A

Configuración inicial.

El estudiante deberá:

✔ Cambiar hostname.

✔ Configurar DNS.

✔ Configurar IPv4.

✔ Configurar IPv6.

✔ Reiniciar NetworkManager.

✔ Validar.

---

# Fase B

Configurar múltiples interfaces.

Todas deberán quedar persistentes.

No deberá existir más de un Gateway.

---

# Fase C

Crear Bond.

Validar:

```text
cat /proc/net/bonding/bond0
```

Desconectar un cable.

Verificar Failover.

---

# Fase D

Crear Bridge.

Conectar una VM.

Validar comunicación.

---

# Fase E

Crear VLAN.

100

200

300

Validar.

---

# Fase F

Firewall.

Crear zonas.

Asignar interfaces.

Crear reglas.

Crear Rich Rules.

---

# Fase G

Configurar NAT.

Verificar navegación.

---

# Fase H

Port Forward.

Probar con curl.

---

# Fase I

Captura de tráfico.

Guardar

```text
dns.pcap

icmp.pcap

ssh.pcap

http.pcap
```

---

# Fase J

Documentación.

Crear un documento con:

Direcciones.

Interfaces.

Firewall.

VLAN.

Bridge.

Bond.

Rutas.

DNS.

Gateway.

---

# Troubleshooting

El instructor provocará errores aleatorios.

Ejemplos:

- Gateway incorrecto.
- DNS incorrecto.
- Firewall bloqueando SSH.
- VLAN mal creada.
- Bond caído.
- Bridge sin IP.
- NAT deshabilitado.
- IPv6 incorrecto.
- Ruta incorrecta.
- DNS sin resolver.

El estudiante deberá encontrar el problema.

---

# Evidencias

Deberán entregarse capturas de:

```text
hostnamectl

ip addr

ip route

ip -6 route

nmcli

firewall-cmd --list-all

firewall-cmd --list-rich-rules

bridge link

bridge vlan

cat /proc/net/bonding/bond0

tcpdump

ss -tulpn
```

---

# Script Final de Validación

El estudiante deberá desarrollar un script que valide automáticamente:

- Hostname
- Interfaces
- IPv4
- IPv6
- DNS
- Gateway
- Bond
- Bridge
- VLAN
- Firewalld
- NAT
- Rich Rules
- Forwarding
- Servicios
- TCP
- UDP

El script deberá finalizar indicando:

```text
VALIDACIÓN EXITOSA
```

o

```text
ERROR EN CONFIGURACIÓN
```

---

# Desafío RHCSA

El instructor eliminará completamente la configuración de red.

El estudiante dispondrá únicamente de:

- El escenario.
- El direccionamiento.
- Las políticas.

Tiempo máximo:

**90 minutos**

Durante ese tiempo deberá reconstruir completamente la infraestructura sin utilizar documentación externa.

---

# Checklist Final

```text
□ Hostname

□ IPv4

□ IPv6

□ DNS

□ Gateway

□ Interfaces

□ Bond

□ Bridge

□ VLAN100

□ VLAN200

□ VLAN300

□ NAT

□ Masquerading

□ Firewall

□ Rich Rules

□ SSH

□ HTTPS

□ DNS

□ ICMP

□ Port Forward

□ tcpdump

□ Reporte

□ Persistencia

□ Reinicio exitoso
```

---

# Criterios de Evaluación

| Tema | Puntos |
|--------|---------|
| IPv4 | 5 |
| IPv6 | 5 |
| NetworkManager | 5 |
| VLAN | 10 |
| Bond | 10 |
| Bridge | 10 |
| Routing | 10 |
| Firewalld | 15 |
| NAT | 10 |
| Port Forward | 5 |
| Rich Rules | 10 |
| tcpdump | 5 |
| Troubleshooting | 5 |
| Documentación | 5 |
| Automatización | 5 |

**Total: 100 puntos**

---

# Resultado esperado

Al finalizar este laboratorio el estudiante habrá implementado una infraestructura Linux empresarial completamente funcional, integrando todas las tecnologías de red estudiadas durante el módulo RHCSA. Además, habrá adquirido la capacidad de diagnosticar y corregir fallos de conectividad, automatizar validaciones, asegurar el tráfico mediante Firewalld y documentar una configuración lista para un entorno de producción.

Con esta fase concluye el **Módulo 6 - Redes**, quedando preparado para afrontar ejercicios prácticos del examen RHCSA y escenarios reales de administración de sistemas Linux.

----









