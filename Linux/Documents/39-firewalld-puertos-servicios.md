# 39. Apertura de Puertos y Servicios con Firewalld

> **Módulo 7: Seguridad del Sistema**  
> **Página 39 del Manual RHCSA**

---

# Objetivos de aprendizaje

Al finalizar esta lección serás capaz de:

- Comprender la diferencia entre un puerto y un servicio.
- Abrir y cerrar puertos utilizando Firewalld.
- Permitir servicios predefinidos.
- Crear reglas permanentes y temporales.
- Administrar puertos TCP y UDP.
- Configurar reglas para aplicaciones comunes.
- Diagnosticar problemas relacionados con el Firewall.

---

# Introducción

Una de las tareas más frecuentes de un administrador Linux consiste en permitir el acceso a un servicio de red.

Por ejemplo:

- Un servidor Web necesita aceptar conexiones HTTP y HTTPS.
- Un servidor PostgreSQL debe permitir conexiones al puerto 5432.
- Un servidor SSH debe aceptar conexiones al puerto 22.

Todo esto se realiza mediante **Firewalld**.

---

# Servicio vs Puerto

Aunque muchas veces se utilizan como sinónimos, **no significan lo mismo**.

## Servicio

Es un nombre que representa uno o varios puertos.

Ejemplos:

| Servicio | Puerto |
|----------|---------|
| ssh | 22/TCP |
| http | 80/TCP |
| https | 443/TCP |
| dns | 53/TCP y UDP |
| postgresql | 5432/TCP |

---

## Puerto

Es el número por el cual una aplicación escucha conexiones.

Ejemplo:

```
PostgreSQL

↓

Puerto 5432

↓

TCP
```

---

# ¿Qué conviene abrir?

Siempre que exista un **servicio predefinido**, se recomienda abrir el servicio y no el puerto.

✔ Correcto

```bash
sudo firewall-cmd --add-service=http
```

Menos recomendable

```bash
sudo firewall-cmd --add-port=80/tcp
```

Los servicios son más fáciles de administrar y documentar.

---

# Ver los servicios disponibles

```bash
firewall-cmd --get-services
```

Ejemplo:

```
ssh

http

https

postgresql

mysql

smtp

imap

dns
```

---

# Consultar un servicio

Ejemplo:

```bash
firewall-cmd --info-service=http
```

Resultado:

```
ports:
80/tcp
```

---

# Abrir el servicio SSH

Temporal:

```bash
sudo firewall-cmd --add-service=ssh
```

Permanente:

```bash
sudo firewall-cmd \
--permanent \
--add-service=ssh
```

Aplicar cambios:

```bash
sudo firewall-cmd --reload
```

---

# Abrir un servidor Web

HTTP

```bash
sudo firewall-cmd \
--permanent \
--add-service=http
```

HTTPS

```bash
sudo firewall-cmd \
--permanent \
--add-service=https
```

Aplicar:

```bash
sudo firewall-cmd --reload
```

---

# Abrir PostgreSQL

```bash
sudo firewall-cmd \
--permanent \
--add-service=postgresql
```

Si la distribución no incluye el servicio:

```bash
sudo firewall-cmd \
--permanent \
--add-port=5432/tcp
```

---

# Abrir SQL Server

SQL Server utiliza:

```
1433/TCP
```

Como normalmente no existe un servicio predefinido:

```bash
sudo firewall-cmd \
--permanent \
--add-port=1433/tcp
```

Aplicar:

```bash
sudo firewall-cmd --reload
```

---

# Abrir MySQL

```bash
sudo firewall-cmd \
--permanent \
--add-service=mysql
```

---

# Abrir DNS

```bash
sudo firewall-cmd \
--permanent \
--add-service=dns
```

---

# Abrir un puerto personalizado

Ejemplo:

Aplicación interna.

Puerto 9000.

```bash
sudo firewall-cmd \
--permanent \
--add-port=9000/tcp
```

---

# Abrir un puerto UDP

Ejemplo:

```bash
sudo firewall-cmd \
--permanent \
--add-port=514/udp
```

---

# Abrir un rango de puertos

```bash
sudo firewall-cmd \
--permanent \
--add-port=8000-8010/tcp
```

---

# Eliminar un servicio

```bash
sudo firewall-cmd \
--permanent \
--remove-service=http
```

Aplicar:

```bash
sudo firewall-cmd --reload
```

---

# Eliminar un puerto

```bash
sudo firewall-cmd \
--permanent \
--remove-port=9000/tcp
```

---

# Verificar los servicios permitidos

```bash
firewall-cmd --list-services
```

---

# Verificar los puertos abiertos

```bash
firewall-cmd --list-ports
```

---

# Ver la configuración completa

```bash
firewall-cmd --list-all
```

---

# Comprobar que la aplicación escucha

Abrir un puerto en Firewalld **no inicia el servicio**.

Debe verificarse con:

```bash
ss -tuln
```

Ejemplo:

```
LISTEN

0.0.0.0:80
```

---

# Comprobar el proceso

```bash
sudo ss -tulpn
```

Ejemplo:

```
httpd

PID 1824
```

---

# Comprobar desde otro equipo

Usando Netcat:

```bash
nc -zv servidor 80
```

o

```bash
nc -zv 192.168.1.20 80
```

---

# Verificar el Firewall

```bash
firewall-cmd --state
```

Resultado:

```
running
```

---

# Configuración temporal

Ejemplo:

```bash
sudo firewall-cmd \
--add-port=5000/tcp
```

Se pierde al reiniciar.

---

# Configuración permanente

```bash
sudo firewall-cmd \
--permanent \
--add-port=5000/tcp
```

Después:

```bash
sudo firewall-cmd --reload
```

---

# Ejemplo práctico

Servidor Web Apache

Servicios necesarios:

```
SSH

HTTP

HTTPS
```

Configuración:

```bash
sudo firewall-cmd \
--permanent \
--add-service=ssh

sudo firewall-cmd \
--permanent \
--add-service=http

sudo firewall-cmd \
--permanent \
--add-service=https

sudo firewall-cmd --reload
```

---

# Ejemplo práctico

Servidor PostgreSQL

```bash
sudo firewall-cmd \
--permanent \
--add-service=postgresql

sudo firewall-cmd --reload
```

---

# Ejemplo práctico

Servidor SQL Server

```bash
sudo firewall-cmd \
--permanent \
--add-port=1433/tcp

sudo firewall-cmd --reload
```

---

# Orden recomendado para abrir un servicio

```
1. Instalar la aplicación

↓

2. Iniciar el servicio

↓

3. Verificar que escucha (ss)

↓

4. Abrir el Firewall

↓

5. Probar desde otro equipo
```

---

# Buenas prácticas RHCSA

✔ Abrir únicamente los servicios necesarios.

✔ Preferir servicios antes que puertos.

✔ Verificar siempre con `ss -tuln`.

✔ Utilizar reglas permanentes solo cuando sean definitivas.

✔ Documentar todos los cambios.

✔ Cerrar puertos que ya no sean utilizados.

✔ Comprobar el acceso desde otro equipo.

---

# Errores comunes

## Abrir el Firewall antes de iniciar el servicio

El puerto seguirá apareciendo cerrado.

Comprobar:

```bash
systemctl status servicio
```

---

## Abrir el puerto equivocado

Verificar qué puerto utiliza realmente la aplicación.

---

## No ejecutar reload

Después de usar:

```bash
--permanent
```

Debe ejecutarse:

```bash
firewall-cmd --reload
```

---

## El puerto está abierto pero no responde

Verificar:

- El servicio está iniciado.
- El proceso escucha en el puerto.
- SELinux permite el acceso.
- No existe otro Firewall intermedio.

---

# Comandos importantes RHCSA

| Comando | Descripción |
|----------|-------------|
| `firewall-cmd --get-services` | Servicios disponibles |
| `firewall-cmd --info-service=http` | Información de un servicio |
| `firewall-cmd --list-services` | Servicios permitidos |
| `firewall-cmd --list-ports` | Puertos abiertos |
| `firewall-cmd --add-service=` | Permitir un servicio |
| `firewall-cmd --remove-service=` | Eliminar un servicio |
| `firewall-cmd --add-port=` | Abrir un puerto |
| `firewall-cmd --remove-port=` | Cerrar un puerto |
| `ss -tuln` | Ver puertos en escucha |
| `ss -tulpn` | Ver procesos asociados a los puertos |

---

# Resumen

En esta lección aprendiste a:

- Diferenciar entre servicios y puertos.
- Abrir y cerrar servicios con Firewalld.
- Abrir y cerrar puertos TCP y UDP.
- Configurar reglas permanentes y temporales.
- Verificar que una aplicación está escuchando.
- Diagnosticar problemas comunes relacionados con el Firewall.

---

# Laboratorio práctico RHCSA

## Escenario 1

Permite permanentemente el servicio HTTP y verifica la configuración.

```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --reload
firewall-cmd --list-services
```

---

## Escenario 2

Abre el puerto TCP **8080** de forma permanente y verifica que aparece en la lista de puertos.

```bash
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
firewall-cmd --list-ports
```

---

## Escenario 3

Permite el puerto **1433/TCP** para un servidor Microsoft SQL Server y comprueba que el proceso está escuchando.

```bash
sudo firewall-cmd --permanent --add-port=1433/tcp
sudo firewall-cmd --reload

sudo ss -tulpn | grep 1433
```

---

## Escenario 4

Elimina la regla del puerto **8080/TCP** y verifica que ya no aparece en Firewalld.

```bash
sudo firewall-cmd --permanent --remove-port=8080/tcp
sudo firewall-cmd --reload

firewall-cmd --list-ports
```

---

## Escenario 5

Desde otro equipo de la red, verifica que el puerto **80** está accesible utilizando `nc`.

```bash
nc -zv <IP_DEL_SERVIDOR> 80
```

Si la conexión falla, revisa:

- El estado de Firewalld.
- Los servicios permitidos.
- El proceso escuchando con `ss -tulpn`.
- El estado del servicio web (`systemctl status httpd`).

> **Objetivo general:** aprender a administrar correctamente los puertos y servicios con **Firewalld**, verificando siempre que el Firewall, la aplicación y la configuración trabajen conjuntamente para ofrecer un acceso seguro a los servicios del servidor Linux.