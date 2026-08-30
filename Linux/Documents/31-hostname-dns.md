# 31. Hostname y Configuración DNS

> **Módulo 6: Redes en Red Hat Enterprise Linux**  
> **Página 31 del Manual RHCSA**

---

# Objetivos de aprendizaje

Al finalizar esta lección serás capaz de:

- Comprender qué es el hostname.
- Configurar el nombre del equipo correctamente.
- Comprender el funcionamiento del DNS.
- Configurar servidores DNS utilizando NetworkManager.
- Verificar la resolución de nombres.
- Solucionar problemas relacionados con hostname y DNS.
- Aplicar buenas prácticas para servidores Linux.

---

# ¿Qué es el Hostname?

El **hostname** es el nombre que identifica un equipo dentro de una red.

Por ejemplo:

```
servidor01
```

```
dbserver
```

```
web01
```

```
rhel9
```

Cuando un administrador se conecta mediante SSH normalmente visualiza algo similar a:

```bash
[root@servidor01 ~]#
```

En este caso:

```
Hostname = servidor01
```

---

# Consultar el hostname actual

```bash
hostname
```

Ejemplo:

```
server01
```

También:

```bash
hostnamectl
```

Salida:

```
Static hostname: server01
Operating System: Red Hat Enterprise Linux 9
Kernel: 5.x
Architecture: x86_64
```

---

# Cambiar el hostname

La forma recomendada en RHEL es:

```bash
sudo hostnamectl set-hostname servidor01
```

No requiere editar archivos manualmente.

---

# Verificar el cambio

```bash
hostname
```

o

```bash
hostnamectl
```

---

# Aplicar el cambio inmediatamente

Abrir una nueva sesión SSH o ejecutar:

```bash
exec bash
```

---

# Archivo donde se almacena el hostname

```bash
/etc/hostname
```

Ver contenido:

```bash
cat /etc/hostname
```

Ejemplo:

```
servidor01
```

---

# ¿Qué es DNS?

DNS significa:

**Domain Name System**

Es el servicio encargado de traducir nombres de dominio en direcciones IP.

Por ejemplo:

```
google.com
```

↓

```
142.250.xxx.xxx
```

Sin DNS tendríamos que recordar únicamente direcciones IP.

---

# Ejemplo de resolución DNS

Cuando ejecutamos:

```bash
ping google.com
```

Linux realiza:

```
google.com

↓

Consulta DNS

↓

Obtiene IP

↓

Realiza el Ping
```

---

# Ver los servidores DNS configurados

```bash
cat /etc/resolv.conf
```

Ejemplo:

```
nameserver 8.8.8.8
nameserver 1.1.1.1
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

Ejemplo:

```
IP4.DNS[1]: 8.8.8.8
IP4.DNS[2]: 1.1.1.1
```

---

# Configurar DNS utilizando nmcli

```bash
sudo nmcli connection modify LAN \
ipv4.dns "8.8.8.8 1.1.1.1"
```

Aplicar:

```bash
sudo nmcli connection up LAN
```

---

# Configurar DNS manualmente (no recomendado)

Editar:

```bash
sudo nano /etc/resolv.conf
```

Agregar:

```
nameserver 8.8.8.8
nameserver 1.1.1.1
```

> **Nota:** En sistemas administrados por **NetworkManager**, este archivo suele regenerarse automáticamente. Lo recomendable es utilizar `nmcli`.

---

# Consultar la IP de un dominio

Utilizando `getent`:

```bash
getent hosts google.com
```

Resultado:

```
142.250.xxx.xxx google.com
```

---

# Usar host

Si está instalado:

```bash
host google.com
```

---

# Usar dig

Si está instalado:

```bash
dig google.com
```

Información obtenida:

- Dirección IP
- Tiempo de respuesta
- Servidor DNS utilizado
- TTL
- Registros DNS

---

# Verificar conectividad

Primero probar por IP:

```bash
ping 8.8.8.8
```

Después por nombre:

```bash
ping google.com
```

Si responde únicamente por IP, el problema probablemente sea el DNS.

---

# Archivo hosts

Linux también utiliza:

```bash
/etc/hosts
```

Ver contenido:

```bash
cat /etc/hosts
```

Ejemplo:

```
127.0.0.1 localhost

192.168.1.20 servidor01
```

Este archivo permite resolver nombres localmente sin consultar un servidor DNS.

---

# Agregar una entrada al archivo hosts

Editar:

```bash
sudo nano /etc/hosts
```

Agregar:

```
192.168.100.50 laboratorio
```

Ahora:

```bash
ping laboratorio
```

funcionará sin necesidad de un servidor DNS.

---

# Diferencia entre /etc/hosts y DNS

| Archivo hosts | DNS |
|---------------|-----|
| Resolución local | Resolución en red |
| Manual | Automática |
| No requiere servidor | Requiere servidor DNS |
| Solo afecta al equipo local | Afecta a toda la red |

---

# Verificar el FQDN

FQDN significa:

**Fully Qualified Domain Name**

Consultar:

```bash
hostname -f
```

Ejemplo:

```
server01.midominio.local
```

---

# Ver información completa del hostname

```bash
hostnamectl status
```

---

# Configurar búsqueda de dominios

```bash
sudo nmcli connection modify LAN \
ipv4.dns-search empresa.local
```

Aplicar:

```bash
sudo nmcli connection up LAN
```

---

# Comprobar resolución de nombres

```bash
getent hosts redhat.com
```

---

# Reiniciar NetworkManager

Si es necesario:

```bash
sudo systemctl restart NetworkManager
```

---

# Buenas prácticas RHCSA

✔ Utilizar nombres descriptivos para los servidores.

✔ Configurar el hostname mediante `hostnamectl`.

✔ Configurar DNS utilizando `nmcli`.

✔ Verificar siempre la resolución de nombres después de modificar la configuración.

✔ Evitar editar manualmente `/etc/resolv.conf` en sistemas administrados por NetworkManager.

✔ Mantener actualizado el archivo `/etc/hosts` cuando se utilicen resoluciones locales.

---

# Errores comunes

## El hostname no cambia

Verificar:

```bash
hostnamectl
```

Abrir una nueva sesión SSH.

---

## No resuelve nombres

Verificar:

```bash
cat /etc/resolv.conf
```

---

## Resuelve por IP pero no por nombre

Comprobar:

```bash
nmcli device show
```

Revisar la configuración de DNS.

---

## No existe conexión a Internet

Comprobar:

```bash
ip route
```

Luego:

```bash
ping 8.8.8.8
```

---

## El archivo resolv.conf vuelve a cambiar

Esto ocurre porque NetworkManager administra automáticamente dicho archivo.

Utilizar:

```bash
nmcli connection modify
```

en lugar de editarlo manualmente.

---

# Comandos importantes para RHCSA

| Comando | Descripción |
|----------|-------------|
| `hostname` | Mostrar hostname |
| `hostnamectl` | Administrar hostname |
| `hostname -f` | Mostrar FQDN |
| `cat /etc/hostname` | Ver hostname almacenado |
| `cat /etc/resolv.conf` | Ver DNS |
| `cat /etc/hosts` | Ver resolución local |
| `getent hosts` | Resolver nombres |
| `host` | Consultar DNS |
| `dig` | Consulta avanzada de DNS |
| `nmcli device show` | Ver DNS configurados |

---

# Resumen

En esta lección aprendiste a:

- Configurar correctamente el hostname.
- Comprender el funcionamiento del DNS.
- Configurar servidores DNS mediante `nmcli`.
- Utilizar `/etc/hosts` para resoluciones locales.
- Diagnosticar problemas de resolución de nombres.
- Aplicar buenas prácticas de administración de red en Red Hat Enterprise Linux.

---

# Ejercicio práctico RHCSA

1. Consulta el hostname actual del servidor.
2. Cambia el hostname a **rhel-lab01** utilizando `hostnamectl`.
3. Verifica el cambio con `hostname` y `hostnamectl`.
4. Configura dos servidores DNS (`8.8.8.8` y `1.1.1.1`) mediante `nmcli`.
5. Reactiva la conexión de red para aplicar los cambios.
6. Comprueba el contenido de `/etc/resolv.conf`.
7. Verifica la resolución de `google.com` con `getent hosts`.
8. Agrega una entrada en `/etc/hosts` para el nombre **laboratorio** con la IP `192.168.100.50`.
9. Comprueba que el nombre **laboratorio** resuelva correctamente mediante `ping`.
10. Reinicia NetworkManager y verifica que toda la configuración continúe funcionando.

> **Objetivo:** dominar la configuración del hostname y del DNS en Red Hat Enterprise Linux utilizando las herramientas recomendadas para el examen **RHCSA**, garantizando una correcta resolución de nombres y administración de la identidad del servidor.