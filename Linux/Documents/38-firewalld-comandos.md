# 38. Administración de Firewalld con firewall-cmd

> **Módulo 7: Seguridad del Sistema**  
> **Página 38 del Manual RHCSA**

---

# Objetivos de aprendizaje

Al finalizar esta lección serás capaz de:

- Administrar Firewalld desde la línea de comandos.
- Consultar la configuración actual del Firewall.
- Agregar y eliminar servicios.
- Abrir y cerrar puertos.
- Comprender la diferencia entre configuraciones temporales y permanentes.
- Recargar correctamente la configuración del Firewall.
- Verificar que las reglas se encuentren aplicadas.

---

# Introducción

La herramienta principal para administrar **Firewalld** es:

```bash
firewall-cmd
```

Con ella es posible:

- Abrir servicios.
- Abrir puertos.
- Cambiar zonas.
- Agregar reglas permanentes.
- Consultar la configuración.
- Eliminar reglas.
- Recargar el Firewall.

Es una de las herramientas más utilizadas por los administradores Linux y es evaluada en el examen **RHCSA**.

---

# Verificar el estado del Firewall

```bash
firewall-cmd --state
```

Resultado esperado:

```
running
```

Si responde:

```
not running
```

Iniciar el servicio:

```bash
sudo systemctl start firewalld
```

---

# Consultar la configuración actual

```bash
firewall-cmd --list-all
```

Ejemplo:

```
public

interfaces:
ens160

services:
ssh dhcpv6-client

ports:

masquerade: no
```

---

# Ver únicamente los servicios permitidos

```bash
firewall-cmd --list-services
```

---

# Ver únicamente los puertos abiertos

```bash
firewall-cmd --list-ports
```

---

# Ver las interfaces asociadas

```bash
firewall-cmd --get-active-zones
```

---

# Agregar un servicio (temporal)

Ejemplo:

Permitir HTTP.

```bash
sudo firewall-cmd --add-service=http
```

Verificar:

```bash
firewall-cmd --list-services
```

---

# Eliminar un servicio (temporal)

```bash
sudo firewall-cmd --remove-service=http
```

---

# Agregar un servicio (permanente)

```bash
sudo firewall-cmd \
--permanent \
--add-service=http
```

Aplicar cambios:

```bash
sudo firewall-cmd --reload
```

---

# Eliminar un servicio (permanente)

```bash
sudo firewall-cmd \
--permanent \
--remove-service=http
```

Luego:

```bash
sudo firewall-cmd --reload
```

---

# Abrir un puerto (temporal)

Ejemplo:

Puerto 8080/TCP

```bash
sudo firewall-cmd \
--add-port=8080/tcp
```

---

# Abrir un puerto UDP

```bash
sudo firewall-cmd \
--add-port=514/udp
```

---

# Verificar puertos abiertos

```bash
firewall-cmd --list-ports
```

---

# Abrir un puerto permanentemente

```bash
sudo firewall-cmd \
--permanent \
--add-port=8080/tcp
```

Aplicar:

```bash
sudo firewall-cmd --reload
```

---

# Cerrar un puerto

```bash
sudo firewall-cmd \
--remove-port=8080/tcp
```

---

# Cerrar un puerto permanentemente

```bash
sudo firewall-cmd \
--permanent \
--remove-port=8080/tcp
```

Luego:

```bash
sudo firewall-cmd --reload
```

---

# Abrir múltiples puertos

```bash
sudo firewall-cmd \
--add-port=8080/tcp

sudo firewall-cmd \
--add-port=8443/tcp
```

---

# Abrir un rango de puertos

```bash
sudo firewall-cmd \
--add-port=8000-8010/tcp
```

---

# Ver todos los servicios disponibles

```bash
firewall-cmd --get-services
```

Ejemplo:

```
ssh

http

https

dns

smtp

imap

postgresql

mysql
```

---

# Consultar información de un servicio

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

# Consultar la configuración permanente

```bash
firewall-cmd \
--permanent \
--list-all
```

---

# Diferencia entre temporal y permanente

## Temporal

```bash
firewall-cmd --add-service=http
```

- No requiere recarga.
- Se pierde al reiniciar.

---

## Permanente

```bash
firewall-cmd \
--permanent \
--add-service=http
```

Después:

```bash
firewall-cmd --reload
```

Permanece después del reinicio.

---

# Recargar la configuración

```bash
sudo firewall-cmd --reload
```

Firewalld aplica los cambios permanentes sin detener el servicio.

---

# Restaurar configuración

Volver a cargar todas las reglas:

```bash
sudo firewall-cmd --reload
```

---

# Restablecer Firewalld (precaución)

Eliminar toda la configuración personalizada:

```bash
sudo rm -rf /etc/firewalld
```

Luego:

```bash
sudo systemctl restart firewalld
```

> **Advertencia:** Este procedimiento elimina todas las reglas personalizadas y debe utilizarse únicamente en laboratorios o cuando se tenga un respaldo de la configuración.

---

# Verificar que el puerto realmente esté abierto

Consultar Firewalld:

```bash
firewall-cmd --list-ports
```

Consultar el servicio escuchando:

```bash
ss -tuln
```

Debe existir un proceso escuchando en dicho puerto.

---

# Ejemplo práctico

Servidor Web.

Permitir HTTP:

```bash
sudo firewall-cmd \
--permanent \
--add-service=http
```

Permitir HTTPS:

```bash
sudo firewall-cmd \
--permanent \
--add-service=https
```

Aplicar:

```bash
sudo firewall-cmd --reload
```

Verificar:

```bash
firewall-cmd --list-services
```

Resultado:

```
ssh

http

https
```

---

# Consultar la zona activa

```bash
firewall-cmd --get-active-zones
```

---

# Mostrar toda la configuración de una zona

```bash
firewall-cmd \
--zone=public \
--list-all
```

---

# Comandos más utilizados

| Comando | Descripción |
|----------|-------------|
| `firewall-cmd --state` | Estado del Firewall |
| `firewall-cmd --list-all` | Configuración actual |
| `firewall-cmd --list-services` | Servicios permitidos |
| `firewall-cmd --list-ports` | Puertos abiertos |
| `firewall-cmd --add-service=` | Agregar servicio |
| `firewall-cmd --remove-service=` | Eliminar servicio |
| `firewall-cmd --add-port=` | Abrir puerto |
| `firewall-cmd --remove-port=` | Cerrar puerto |
| `firewall-cmd --reload` | Recargar configuración |
| `firewall-cmd --get-services` | Servicios disponibles |
| `firewall-cmd --info-service=` | Información de un servicio |

---

# Buenas prácticas RHCSA

✔ Preferir **servicios** en lugar de puertos cuando exista un servicio predefinido.

✔ Abrir únicamente los puertos necesarios.

✔ Utilizar configuraciones permanentes únicamente después de probar las reglas.

✔ Verificar siempre el resultado con `--list-all`.

✔ Confirmar que el servicio correspondiente esté escuchando mediante `ss -tuln`.

✔ Documentar todas las reglas implementadas.

---

# Errores comunes

## Abrir el puerto pero no el servicio

El Firewall puede permitir el tráfico, pero si la aplicación no está ejecutándose, seguirá siendo inaccesible.

Verificar:

```bash
ss -tuln
```

---

## Los cambios desaparecen

Se utilizó:

```bash
--add-service
```

en lugar de:

```bash
--permanent
```

---

## Se agregó una regla permanente pero no funciona

Falta ejecutar:

```bash
firewall-cmd --reload
```

---

## El puerto sigue cerrado

Comprobar:

- Que la aplicación esté iniciada.
- Que SELinux no esté bloqueando el acceso.
- Que el puerto esté realmente escuchando.

---

# Resumen

En esta lección aprendiste a:

- Administrar Firewalld mediante `firewall-cmd`.
- Abrir y cerrar servicios.
- Abrir y cerrar puertos.
- Comprender la diferencia entre reglas temporales y permanentes.
- Recargar correctamente la configuración.
- Verificar las reglas configuradas.
- Diagnosticar problemas básicos relacionados con el Firewall.

---

# Laboratorio práctico RHCSA

## Escenario 1

Comprueba que Firewalld se encuentra activo.

```bash
firewall-cmd --state
```

---

## Escenario 2

Permite temporalmente el servicio HTTP.

```bash
sudo firewall-cmd --add-service=http
```

Comprueba que aparece en la lista de servicios.

---

## Escenario 3

Elimina el servicio HTTP temporal.

```bash
sudo firewall-cmd --remove-service=http
```

---

## Escenario 4

Abre permanentemente los servicios HTTP y HTTPS.

```bash
sudo firewall-cmd --permanent --add-service=http

sudo firewall-cmd --permanent --add-service=https

sudo firewall-cmd --reload
```

Verifica la configuración.

---

## Escenario 5

Abre el puerto TCP **8080**, verifica que aparece en Firewalld y confirma con `ss -tuln` que una aplicación está escuchando en dicho puerto. Finalmente, elimina la regla y vuelve a comprobar la configuración.

> **Objetivo general:** dominar la administración de **Firewalld** mediante `firewall-cmd`, aprendiendo a gestionar servicios, puertos y configuraciones permanentes, competencias fundamentales para el examen **RHCSA** y la administración diaria de servidores Linux.