# 32. IPv4, IPv6 y Subnetting

> **Módulo 6: Redes en Red Hat Enterprise Linux**  
> **Página 32 del Manual RHCSA**

---

# Objetivos de aprendizaje

Al finalizar esta lección serás capaz de:

- Comprender qué es una dirección IP.
- Diferenciar IPv4 e IPv6.
- Entender el concepto de subred (Subnetting).
- Interpretar la notación CIDR.
- Identificar direcciones de red, broadcast y hosts.
- Calcular el número de hosts disponibles en una subred.
- Aplicar estos conocimientos en la administración de servidores Linux.

---

# ¿Qué es una dirección IP?

Una **dirección IP (Internet Protocol Address)** es un identificador único asignado a un dispositivo dentro de una red.

Permite que los equipos puedan comunicarse entre sí.

Ejemplo:

```
PC -------- Router -------- Servidor Linux
       192.168.1.10
```

Cada equipo posee una dirección IP diferente.

---

# Tipos de direcciones IP

Existen dos versiones ampliamente utilizadas:

- IPv4
- IPv6

Actualmente ambos protocolos conviven en la mayoría de las redes.

---

# ¿Qué es IPv4?

IPv4 utiliza direcciones de **32 bits**.

Está dividido en cuatro octetos separados por puntos.

Ejemplo:

```
192.168.1.100
```

Cada octeto puede contener valores entre:

```
0 - 255
```

---

# Ejemplos de direcciones IPv4

```
10.0.0.10

172.16.10.20

192.168.1.100

8.8.8.8
```

---

# Estructura de una dirección IPv4

```
192 .168 .1 .100

|   |   |   |

Octetos
```

Cada octeto representa 8 bits.

Total:

```
8 + 8 + 8 + 8 = 32 bits
```

---

# ¿Qué es IPv6?

IPv6 fue creado para solucionar el agotamiento de direcciones IPv4.

Utiliza **128 bits**.

Ejemplo:

```
2001:db8:abcd:1000::15
```

Una dirección IPv6 posee ocho grupos hexadecimales separados por dos puntos.

---

# Ejemplos de IPv6

```
2001:db8::1

fe80::1234

2606:4700:4700::1111
```

---

# Ventajas de IPv6

- Muchísimas más direcciones disponibles.
- Mejor rendimiento en algunas redes.
- Mejor soporte para seguridad (IPsec).
- Autoconfiguración.
- Menor necesidad de NAT.

---

# Comparación IPv4 vs IPv6

| Característica | IPv4 | IPv6 |
|---------------|------|-------|
| Tamaño | 32 bits | 128 bits |
| Formato | Decimal | Hexadecimal |
| Separador | Punto (.) | Dos puntos (:) |
| NAT | Muy utilizado | Generalmente innecesario |
| Espacio de direcciones | Limitado | Prácticamente ilimitado |

---

# ¿Qué es una máscara de red?

La máscara indica qué parte de la dirección IP pertenece a la red y cuál identifica al host.

Ejemplo:

```
255.255.255.0
```

Equivale a:

```
/24
```

---

# Notación CIDR

Actualmente se utiliza la notación CIDR.

Ejemplos:

```
192.168.1.100/24

10.0.0.10/16

172.16.20.15/20
```

El número después de la barra representa la cantidad de bits destinados a la red.

---

# Máscaras más utilizadas

| CIDR | Máscara |
|------|----------------|
| /8 | 255.0.0.0 |
| /16 | 255.255.0.0 |
| /24 | 255.255.255.0 |
| /25 | 255.255.255.128 |
| /26 | 255.255.255.192 |
| /27 | 255.255.255.224 |
| /28 | 255.255.255.240 |
| /29 | 255.255.255.248 |
| /30 | 255.255.255.252 |

---

# ¿Qué es una subred?

Una subred divide una red grande en varias redes más pequeñas.

Beneficios:

- Mejor organización.
- Mayor seguridad.
- Menor tráfico de broadcast.
- Mejor administración.

---

# Ejemplo de una red /24

```
192.168.1.0/24
```

Contiene:

```
256 direcciones
```

Desde:

```
192.168.1.0
```

Hasta:

```
192.168.1.255
```

---

# Dirección de red

Es la primera dirección de la subred.

Ejemplo:

```
192.168.1.0
```

No puede asignarse a un equipo.

---

# Dirección Broadcast

Es la última dirección de la subred.

Ejemplo:

```
192.168.1.255
```

Se utiliza para enviar información a todos los equipos de la red.

---

# Direcciones utilizables

En una red:

```
192.168.1.0/24
```

Los hosts pueden utilizar:

```
192.168.1.1

...

192.168.1.254
```

---

# Número de hosts

Fórmula:

```
2^(bits de host) - 2
```

Ejemplo:

```
/24

32 - 24 = 8 bits

2⁸ = 256

256 - 2 = 254 hosts
```

---

# Ejemplo de una red /25

```
192.168.1.0/25
```

Rango:

```
192.168.1.0

↓

192.168.1.127
```

Hosts disponibles:

```
126
```

---

# Ejemplo de una red /26

```
192.168.1.0/26
```

Rango:

```
192.168.1.0

↓

192.168.1.63
```

Hosts:

```
62
```

---

# Ejemplo de una red /27

```
192.168.1.0/27
```

Hosts disponibles:

```
30
```

---

# Ejemplo de una red /28

```
192.168.1.0/28
```

Hosts:

```
14
```

---

# Ejemplo de una red /30

Muy utilizada para enlaces punto a punto.

```
192.168.1.0/30
```

Hosts:

```
2
```

---

# Ver la IP en Linux

```bash
ip addr
```

---

# Ver únicamente una interfaz

```bash
ip addr show ens160
```

---

# Ver las rutas

```bash
ip route
```

---

# Ver IPv6

```bash
ip -6 addr
```

---

# Ver rutas IPv6

```bash
ip -6 route
```

---

# Agregar temporalmente una IP

```bash
sudo ip addr add 192.168.1.100/24 dev ens160
```

---

# Eliminar una IP temporal

```bash
sudo ip addr del 192.168.1.100/24 dev ens160
```

---

# Configurar IPv4 permanente

Con NetworkManager:

```bash
sudo nmcli connection modify LAN \
ipv4.addresses 192.168.1.100/24 \
ipv4.gateway 192.168.1.1 \
ipv4.method manual
```

---

# Configurar IPv6

```bash
sudo nmcli connection modify LAN \
ipv6.addresses 2001:db8::10/64 \
ipv6.gateway 2001:db8::1 \
ipv6.method manual
```

---

# Verificar la configuración

```bash
ip addr
```

```bash
ip route
```

```bash
nmcli device show
```

---

# Buenas prácticas RHCSA

✔ Utilizar siempre la notación CIDR.

✔ Documentar las direcciones IP asignadas.

✔ Evitar duplicar direcciones IP.

✔ Verificar la máscara antes de configurar un servidor.

✔ Utilizar NetworkManager (`nmcli`) para realizar configuraciones permanentes.

✔ Comprobar siempre la conectividad después de realizar cambios.

---

# Errores comunes

## Máscara incorrecta

Una máscara mal configurada puede impedir la comunicación con otros equipos.

Verificar:

```bash
ip addr
```

---

## Gateway fuera de la red

Ejemplo incorrecto:

```
IP:
192.168.1.20/24

Gateway:
192.168.2.1
```

El gateway debe pertenecer a la misma red.

---

## IP duplicada

Dos equipos con la misma dirección IP provocan conflictos de red.

---

## Ruta por defecto ausente

Verificar:

```bash
ip route
```

Debe existir una línea similar a:

```
default via 192.168.1.1
```

---

# Comandos importantes RHCSA

| Comando | Descripción |
|----------|-------------|
| `ip addr` | Mostrar direcciones IP |
| `ip route` | Mostrar rutas IPv4 |
| `ip -6 addr` | Mostrar direcciones IPv6 |
| `ip -6 route` | Mostrar rutas IPv6 |
| `hostname -I` | Mostrar IP del host |
| `nmcli device show` | Información completa de la interfaz |
| `nmcli connection modify` | Configurar IP permanente |

---

# Resumen

En esta lección aprendiste a:

- Comprender el funcionamiento de IPv4 e IPv6.
- Diferenciar ambos protocolos.
- Interpretar la notación CIDR.
- Calcular hosts disponibles en una subred.
- Identificar direcciones de red y broadcast.
- Configurar direcciones IPv4 e IPv6 en Linux.
- Aplicar buenas prácticas para la administración de redes en Red Hat Enterprise Linux.

---

# Ejercicio práctico RHCSA

1. Identifica la dirección IPv4 y la dirección IPv6 de tu servidor utilizando `ip addr`.
2. Determina la máscara de red y expresa la configuración en notación CIDR.
3. Calcula cuántos hosts admite una red `/24`, `/26` y `/30`.
4. Identifica la dirección de red, la dirección de broadcast y el rango de hosts para la red `192.168.100.0/24`.
5. Agrega temporalmente la dirección `192.168.100.50/24` a una interfaz de red y verifica que se haya aplicado.
6. Elimina la dirección temporal.
7. Configura una dirección IPv4 permanente mediante `nmcli`.
8. Configura una dirección IPv6 estática con un prefijo `/64`.
9. Comprueba las rutas IPv4 e IPv6 utilizando `ip route` e `ip -6 route`.
10. Verifica la conectividad con otros equipos de la red.

> **Objetivo:** comprender la estructura de las direcciones IPv4 e IPv6, interpretar correctamente las máscaras de red y el subnetting, y configurar direcciones IP en Red Hat Enterprise Linux, competencias esenciales para el examen **RHCSA**.