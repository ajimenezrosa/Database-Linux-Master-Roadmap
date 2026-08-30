# 45. Diagnóstico y Solución de Problemas con SELinux

> **Módulo 7: Seguridad del Sistema**  
> **Página 45 del Manual RHCSA**

---

# Objetivos de aprendizaje

Al finalizar esta lección serás capaz de:

- Diagnosticar problemas relacionados con SELinux.
- Identificar cuándo un error proviene de SELinux y cuándo no.
- Interpretar mensajes de denegación (AVC).
- Utilizar las principales herramientas de diagnóstico.
- Resolver problemas de contextos y booleanos.
- Aplicar un procedimiento sistemático para solucionar incidencias.

---

# Introducción

Uno de los errores más comunes entre administradores principiantes consiste en **deshabilitar SELinux** cuando una aplicación deja de funcionar.

Sin embargo, la mayoría de los problemas pueden resolverse sin desactivar SELinux.

El objetivo del administrador debe ser:

- Identificar la causa.
- Corregir la configuración.
- Mantener SELinux en modo **Enforcing**.

---

# ¿Cómo saber si el problema es SELinux?

No todos los errores están relacionados con SELinux.

Antes de asumirlo, verifica:

- ¿El servicio está iniciado?
- ¿El Firewall permite el acceso?
- ¿La aplicación escucha en el puerto correcto?
- ¿Los permisos Linux son correctos?
- ¿Existen errores en los registros?

Solo después conviene investigar SELinux.

---

# Flujo recomendado de diagnóstico

```
Aplicación falla

        │

        ▼

¿Servicio iniciado?

        │

        ▼

¿Firewall correcto?

        │

        ▼

¿Puerto abierto?

        │

        ▼

¿Permisos Linux correctos?

        │

        ▼

¿SELinux?

        │

        ▼

Revisar Contextos

↓

Revisar Booleanos

↓

Revisar Logs
```

---

# Paso 1: Verificar el estado de SELinux

```bash
getenforce
```

Resultado esperado:

```
Enforcing
```

Obtener información completa:

```bash
sestatus
```

---

# Paso 2: Comprobar si el problema desaparece en modo Permissive

Cambiar temporalmente:

```bash
sudo setenforce 0
```

Probar nuevamente la aplicación.

Si ahora funciona correctamente:

```
Probablemente el problema esté relacionado con SELinux.
```

Después de la prueba:

```bash
sudo setenforce 1
```

Nunca dejes un servidor de producción en modo **Permissive** sin una razón justificada.

---

# Paso 3: Revisar los contextos

Consultar el contexto del archivo:

```bash
ls -lZ /var/www/html
```

Consultar el contexto del proceso:

```bash
ps -eZ | grep httpd
```

Comparar ambos contextos.

---

# Paso 4: Restaurar contextos

Muchos problemas se solucionan con:

```bash
sudo restorecon -Rv /var/www/html
```

Verificar nuevamente:

```bash
ls -lZ /var/www/html
```

---

# Paso 5: Revisar los booleanos

Consultar:

```bash
getsebool -a | grep httpd
```

Si Apache necesita conectarse a otro servicio:

```bash
getsebool httpd_can_network_connect
```

Activar permanentemente:

```bash
sudo setsebool -P httpd_can_network_connect on
```

---

# Paso 6: Revisar los registros

## Utilizando journalctl

Buscar eventos relacionados con SELinux:

```bash
sudo journalctl | grep AVC
```

También:

```bash
sudo journalctl -xe
```

---

# Utilizando ausearch

Buscar eventos AVC:

```bash
sudo ausearch -m AVC
```

Buscar denegaciones recientes:

```bash
sudo ausearch -m AVC -ts recent
```

---

# ¿Qué es un mensaje AVC?

AVC significa:

```
Access Vector Cache
```

Cuando SELinux bloquea un acceso registra un evento similar a:

```
AVC denied
```

Este mensaje indica:

- Qué proceso realizó la acción.
- Qué recurso intentó utilizar.
- Qué operación fue bloqueada.

---

# Ejemplo de un mensaje AVC

```
type=AVC

denied

comm="httpd"

name="index.html"
```

Información importante:

- `comm` → proceso que intentó acceder.
- `name` → recurso afectado.
- `denied` → acción bloqueada.

---

# Analizar una denegación

Supongamos:

```
Apache

↓

Intenta acceder

↓

/web/index.html

↓

Denegado
```

Comprobar el contexto:

```bash
ls -lZ /web
```

Resultado:

```
user_home_t
```

Solución:

```bash
sudo semanage fcontext \
-a \
-t httpd_sys_content_t \
"/web(/.*)?"

sudo restorecon -Rv /web
```

---

# Herramienta audit2why

`audit2why` interpreta los mensajes de auditoría y explica por qué ocurrió la denegación.

Ejemplo:

```bash
sudo ausearch -m AVC | audit2why
```

Salida aproximada:

```
SELinux denied access because...
```

Esta herramienta resulta muy útil para comprender la causa del problema.

---

# Herramienta audit2allow

`audit2allow` puede generar políticas SELinux a partir de los eventos registrados.

Ejemplo:

```bash
sudo ausearch -m AVC | audit2allow
```

> **Importante:** En un entorno RHCSA normalmente **no se espera crear políticas personalizadas**. Antes de utilizar `audit2allow`, verifica si el problema puede resolverse corrigiendo un contexto o activando un booleano.

---

# Escenario 1: Apache devuelve "403 Forbidden"

Comprobar:

```bash
ls -lZ /var/www/html
```

Si el contexto es incorrecto:

```bash
restorecon -Rv /var/www/html
```

---

# Escenario 2: Apache no conecta con PostgreSQL

Verificar:

```bash
getsebool httpd_can_network_connect
```

Si está desactivado:

```bash
sudo setsebool -P httpd_can_network_connect on
```

---

# Escenario 3: Archivo copiado desde el directorio personal

Consultar:

```bash
ls -Z archivo
```

Resultado:

```
user_home_t
```

Corregir:

```bash
restorecon archivo
```

o, si el directorio es nuevo:

```bash
semanage fcontext
```

---

# Escenario 4: Servicio aparentemente correcto

Comprobar:

```bash
systemctl status servicio
```

Después:

```bash
ss -tulpn
```

Finalmente:

```bash
journalctl -xe
```

No todos los problemas son provocados por SELinux.

---

# Orden recomendado de diagnóstico

```
1. Servicio

↓

2. Firewall

↓

3. Puerto

↓

4. Permisos Linux

↓

5. Contextos

↓

6. Booleanos

↓

7. Logs

↓

8. ausearch

↓

9. audit2why
```

---

# Herramientas de diagnóstico

| Herramienta | Función |
|-------------|---------|
| `getenforce` | Ver modo actual |
| `sestatus` | Estado completo |
| `ls -Z` | Contextos de archivos |
| `ps -eZ` | Contextos de procesos |
| `restorecon` | Restaurar contextos |
| `getsebool` | Consultar booleanos |
| `setsebool` | Modificar booleanos |
| `journalctl` | Consultar registros |
| `ausearch` | Buscar eventos de auditoría |
| `audit2why` | Explicar denegaciones |
| `audit2allow` | Generar políticas (uso avanzado) |

---

# Buenas prácticas RHCSA

✔ Mantener SELinux en **Enforcing**.

✔ Revisar primero los registros antes de modificar configuraciones.

✔ Corregir los contextos utilizando `restorecon`.

✔ Utilizar `semanage` para cambios permanentes.

✔ Activar únicamente los booleanos necesarios.

✔ Utilizar **Permissive** solo durante el diagnóstico.

✔ Documentar cualquier modificación realizada sobre SELinux.

---

# Errores comunes

## Deshabilitar SELinux

No es la solución adecuada.

Debe identificarse la causa del bloqueo.

---

## Utilizar chmod 777

Los permisos Linux no corrigen un contexto SELinux incorrecto.

---

## Olvidar restorecon

Después de crear una regla con `semanage` siempre debe ejecutarse:

```bash
restorecon
```

---

## Ignorar los mensajes AVC

Los registros contienen información valiosa para resolver el problema.

---

## Crear políticas innecesarias

Antes de utilizar `audit2allow`, verifica si el problema se resuelve con:

- Contextos.
- Booleanos.
- Configuración de la aplicación.

---

# Comandos importantes RHCSA

| Comando | Descripción |
|----------|-------------|
| `getenforce` | Ver modo actual |
| `sestatus` | Estado completo |
| `setenforce 0` | Cambiar a Permissive |
| `setenforce 1` | Volver a Enforcing |
| `ls -Z` | Consultar contextos |
| `restorecon -Rv` | Restaurar contextos |
| `getsebool -a` | Listar booleanos |
| `journalctl \| grep AVC` | Buscar eventos SELinux |
| `ausearch -m AVC` | Consultar denegaciones |
| `audit2why` | Explicar denegaciones |
| `audit2allow` | Generar políticas (uso avanzado) |

---

# Resumen

En esta lección aprendiste a:

- Diagnosticar problemas relacionados con SELinux.
- Determinar si una falla proviene realmente de SELinux.
- Interpretar eventos AVC.
- Utilizar `journalctl`, `ausearch` y `audit2why`.
- Resolver problemas mediante contextos, booleanos y políticas.
- Aplicar un procedimiento ordenado de diagnóstico sin deshabilitar SELinux.

---

# Laboratorio práctico RHCSA

## Escenario 1

Comprueba el estado de SELinux.

```bash
getenforce
```

Obtén información completa.

```bash
sestatus
```

---

## Escenario 2

Cambia temporalmente a modo **Permissive**.

```bash
sudo setenforce 0
```

Prueba una aplicación y vuelve al modo **Enforcing**.

```bash
sudo setenforce 1
```

---

## Escenario 3

Consulta el contexto del directorio del servidor web.

```bash
ls -lZ /var/www/html
```

Restaura los contextos.

```bash
sudo restorecon -Rv /var/www/html
```

---

## Escenario 4

Busca eventos recientes relacionados con SELinux.

```bash
sudo ausearch -m AVC -ts recent
```

Interpreta los resultados utilizando:

```bash
sudo ausearch -m AVC | audit2why
```

---

## Escenario 5

Verifica si Apache puede realizar conexiones de red.

```bash
getsebool httpd_can_network_connect
```

Si es necesario, actívalo permanentemente.

```bash
sudo setsebool -P httpd_can_network_connect on
```

Comprueba nuevamente el estado del booleano.

> **Objetivo general:** desarrollar una metodología de diagnóstico para resolver problemas relacionados con **SELinux** sin comprometer la seguridad del sistema. Dominar estas herramientas permitirá identificar rápidamente errores de contexto, booleanos y políticas, una habilidad esencial para el examen **RHCSA** y para la administración profesional de servidores Red Hat Enterprise Linux.