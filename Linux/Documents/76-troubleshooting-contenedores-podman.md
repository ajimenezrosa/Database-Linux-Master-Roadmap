# 76. Troubleshooting de Contenedores con Podman (Fase 1)

> **Módulo 10 – Contenedores con Podman**
>
> **Manual RHCSA**
>
> **Archivo:** `76-troubleshooting-contenedores-podman.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase podrás:

- Comprender la metodología profesional para diagnosticar problemas en Podman.
- Identificar rápidamente el origen de una falla.
- Utilizar correctamente las herramientas de diagnóstico.
- Diferenciar problemas del Host de problemas del contenedor.
- Comprender el flujo de resolución utilizado por administradores Linux.
- Prepararte para escenarios de troubleshooting del examen RHCSA.

---

# Introducción

En un ambiente de producción los contenedores fallan por muchas razones.

Algunos ejemplos son:

- Un puerto ya está ocupado.
- Un volumen perdió permisos.
- SELinux bloqueó un acceso.
- La imagen está dañada.
- El almacenamiento se llenó.
- El contenedor consume demasiada memoria.
- La aplicación interna dejó de funcionar.
- El servicio systemd no pudo iniciarlo.

El objetivo de un administrador Linux no consiste únicamente en reiniciar el contenedor.

Debe descubrir la causa raíz.

---

# ¿Qué es Troubleshooting?

Troubleshooting es el proceso sistemático de identificar, analizar y resolver problemas.

Nunca debe hacerse al azar.

Siempre debe existir una metodología.

---

# Metodología RHCSA

```text
Problema

↓

Identificar

↓

Recolectar evidencia

↓

Analizar

↓

Resolver

↓

Validar

↓

Documentar
```

---

# Principio Fundamental

Nunca comiences modificando configuraciones.

Primero recopila información.

```text
NO

↓

Modificar archivos

↓

Reiniciar

↓

Esperar que funcione
```

Debe hacerse:

```text
Consultar

↓

Analizar

↓

Comprender

↓

Corregir
```

---

# Flujo General

```text
Usuario reporta problema

↓

¿El contenedor existe?

↓

¿Está ejecutándose?

↓

¿Genera errores?

↓

¿El Host está saludable?

↓

¿La aplicación funciona?

↓

Resolver
```

---

# Primera pregunta

¿Existe el contenedor?

Consultar

```bash
podman ps -a
```

---

# Resultado

```text
CONTAINER ID

IMAGE

STATUS

NAMES
```

---

# Posibles estados

| Estado | Significado |
|----------|-------------|
| Up | Ejecutándose |
| Exited | Finalizado |
| Created | Creado |
| Configured | Configurado |
| Removing | Eliminándose |
| Stopping | Deteniéndose |

---

# Segunda pregunta

¿Está ejecutándose?

Consultar

```bash
podman ps
```

Si no aparece:

Consultar

```bash
podman ps -a
```

---

# Arquitectura

```text
Contenedor

↓

¿Existe?

↓

SI

↓

¿Está ejecutándose?

↓

SI

↓

Analizar aplicación
```

---

# Tercera pregunta

¿Qué ocurrió?

Consultar

```bash
podman logs web
```

---

# Consultar últimas líneas

```bash
podman logs \
--tail 50 web
```

---

# Seguir en tiempo real

```bash
podman logs \
-f web
```

---

# Arquitectura

```text
Container

↓

Logs

↓

Errores

↓

Diagnóstico
```

---

# Cuarta pregunta

¿Cómo fue creado?

Consultar

```bash
podman inspect web
```

---

# Información disponible

- Imagen
- Red
- Volúmenes
- Variables
- PID
- UID
- Capabilities
- Montajes
- Labels
- Recursos

---

# Consultar formato JSON

```bash
podman inspect web
```

---

# Consultar únicamente IP

```bash
podman inspect \
--format '{{.NetworkSettings.IPAddress}}' \
web
```

---

# Consultar Imagen

```bash
podman inspect \
--format '{{.ImageName}}' \
web
```

---

# Consultar Estado

```bash
podman inspect \
--format '{{.State.Status}}' \
web
```

---

# Consultar PID

```bash
podman inspect \
--format '{{.State.Pid}}' \
web
```

---

# Quinta pregunta

¿Consume demasiados recursos?

Consultar

```bash
podman stats
```

---

Resultado

```text
CPU

MEM

NET

BLOCK IO
```

---

# Consultar una sola vez

```bash
podman stats \
--no-stream
```

---

# Sexta pregunta

¿Qué procesos ejecuta?

Consultar

```bash
podman top web
```

---

Resultado

```text
PID

USER

TIME

COMMAND
```

---

# Séptima pregunta

¿Está respondiendo?

Entrar al contenedor.

```bash
podman exec \
-it web bash
```

---

Consultar

```bash
ps aux
```

---

Consultar

```bash
ss -tln
```

---

Consultar

```bash
curl localhost
```

---

# Arquitectura

```text
Host

↓

podman exec

↓

Container

↓

Aplicación
```

---

# Octava pregunta

¿Existe la imagen?

Consultar

```bash
podman images
```

---

Consultar información

```bash
podman image inspect
```

---

# Novena pregunta

¿Hay suficiente espacio?

Consultar

```bash
df -h
```

---

Consultar inodos

```bash
df -i
```

---

Consultar almacenamiento Podman

```bash
podman system df
```

---

# Décima pregunta

¿El problema es del Host?

Consultar

```bash
uptime
```

---

Consultar memoria

```bash
free -h
```

---

Consultar CPU

```bash
top
```

---

Consultar carga

```bash
uptime
```

---

# Flujo Completo

```text
Contenedor

↓

Estado

↓

Logs

↓

Inspect

↓

Procesos

↓

Recursos

↓

Host

↓

Aplicación
```

---

# Herramientas Principales

| Herramienta | Función |
|-------------|----------|
| podman ps | Contenedores |
| podman logs | Logs |
| podman inspect | Configuración |
| podman exec | Acceso |
| podman stats | Recursos |
| podman top | Procesos |
| podman images | Imágenes |
| podman volume ls | Volúmenes |
| podman network ls | Redes |

---

# Herramientas Linux

| Herramienta | Función |
|-------------|----------|
| top | CPU |
| htop | CPU interactivo |
| free | Memoria |
| df | Disco |
| lsblk | Discos |
| mount | Sistemas montados |
| ip | Red |
| ss | Puertos |
| journalctl | Logs |
| systemctl | Servicios |

---

# Escenario 1

## El contenedor no existe

Consultar

```bash
podman ps -a
```

Resultado

```text
No container found
```

Posibles causas

- Eliminado.
- Nombre incorrecto.
- Usuario diferente.

---

# Escenario 2

## El contenedor está detenido

Consultar

```bash
podman ps -a
```

Estado

```text
Exited
```

Consultar

```bash
podman logs
```

---

# Escenario 3

## El contenedor inicia y termina inmediatamente

Consultar

```bash
podman logs
```

Generalmente ocurre cuando:

- El proceso principal finaliza.
- Error de configuración.
- Variables faltantes.
- Error interno de la aplicación.

---

# Escenario 4

## El contenedor consume demasiada memoria

Consultar

```bash
podman stats
```

↓

```bash
free -h
```

↓

```bash
top
```

---

# Escenario 5

## El contenedor responde lentamente

Consultar

```bash
top
```

↓

```bash
iostat
```

↓

```bash
vmstat
```

↓

```bash
podman stats
```

---

# Laboratorio RHCSA

## Laboratorio 1

Listar todos los contenedores.

```bash
podman ps -a
```

---

## Laboratorio 2

Consultar únicamente los activos.

```bash
podman ps
```

---

## Laboratorio 3

Consultar logs.

```bash
podman logs web
```

---

## Laboratorio 4

Consultar últimas líneas.

```bash
podman logs \
--tail 20 web
```

---

## Laboratorio 5

Entrar al contenedor.

```bash
podman exec \
-it web bash
```

---

## Laboratorio 6

Consultar procesos.

```bash
ps aux
```

---

## Laboratorio 7

Consultar puertos.

```bash
ss -tln
```

---

## Laboratorio 8

Consultar estadísticas.

```bash
podman stats
```

---

## Laboratorio 9

Consultar imágenes.

```bash
podman images
```

---

## Laboratorio 10

Consultar inspect.

```bash
podman inspect web
```

---

## Laboratorio 11

Consultar espacio.

```bash
df -h
```

---

## Laboratorio 12

Consultar memoria.

```bash
free -h
```

---

## Laboratorio 13

Consultar CPU.

```bash
top
```

---

## Laboratorio 14

Consultar Journal.

```bash
journalctl
```

---

## Laboratorio 15

Realizar un diagnóstico completo utilizando el flujo aprendido.

---

# Buenas prácticas

- Comenzar siempre con `podman ps -a`.
- Consultar los logs antes de reiniciar un contenedor.
- Analizar el resultado de `podman inspect`.
- Verificar el estado general del Host antes de asumir que el problema pertenece al contenedor.
- Documentar la causa raíz y la solución aplicada.

---

# Errores comunes

## Error 1

Reiniciar el contenedor sin revisar los registros.

---

## Error 2

Modificar múltiples parámetros simultáneamente, dificultando identificar la causa del problema.

---

## Error 3

Ignorar el consumo de recursos del Host.

---

## Error 4

No distinguir entre un problema de la aplicación y un problema del contenedor.

---

## Error 5

Trabajar directamente sobre producción sin reproducir previamente el problema en un laboratorio.

---

# Resumen

En esta primera fase aprendimos:

- La metodología profesional para realizar troubleshooting.
- Cómo identificar rápidamente el estado de un contenedor.
- Cómo utilizar `podman ps`, `logs`, `inspect`, `exec`, `top` y `stats`.
- Cómo determinar si el problema proviene del Host o del contenedor.
- Cómo recopilar evidencia antes de realizar cualquier cambio.

En la **Fase 2** aprenderemos a resolver problemas relacionados con redes, almacenamiento, volúmenes, permisos, SELinux, Rootless, Rootful y systemd, utilizando escenarios reales encontrados en servidores Fedora y Red Hat Enterprise Linux.

----

# 76. Troubleshooting de Contenedores con Podman (Fase 2)

> **Módulo 10 – Contenedores con Podman**
>
> **Manual RHCSA**
>
> **Archivo:** `76-troubleshooting-contenedores-podman.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase podrás:

- Diagnosticar problemas de red en Podman.
- Resolver incidencias relacionadas con almacenamiento y volúmenes.
- Identificar errores de permisos Linux.
- Diagnosticar problemas causados por SELinux.
- Resolver incidencias en contenedores Rootless y Rootful.
- Diagnosticar problemas con systemd y Quadlets.
- Aplicar una metodología profesional de resolución de problemas.

---

# Introducción

La mayoría de los problemas que ocurren en producción pertenecen a uno de estos grupos:

- Red
- Almacenamiento
- Permisos
- SELinux
- Recursos
- Systemd
- Rootless
- Rootful

Un administrador RHCSA debe ser capaz de determinar rápidamente cuál de estas áreas está provocando la falla.

---

# Árbol de Diagnóstico

```text
Contenedor falla

        │

        ▼

¿Existe?

        │

        ▼

¿Está ejecutándose?

        │

        ▼

¿Tiene red?

        │

        ▼

¿Tiene almacenamiento?

        │

        ▼

¿Tiene permisos?

        │

        ▼

¿SELinux lo permite?

        │

        ▼

¿Systemd funciona?

        │

        ▼

Resolver
```

---

# Problemas de Red

Los síntomas más comunes son:

- No responde al ping.
- No responde por HTTP.
- No puede acceder a Internet.
- No resuelve DNS.
- No puede comunicarse con otro contenedor.

---

# Verificar Redes

Consultar:

```bash
podman network ls
```

Resultado:

```text
NAME

podman

backend

frontend
```

---

# Inspeccionar una Red

```bash
podman network inspect podman
```

Información disponible:

- Subred
- Gateway
- Driver
- DNS
- Contenedores conectados

---

# Verificar IP

```bash
podman inspect \
--format '{{.NetworkSettings.IPAddress}}' \
web
```

---

# Entrar al Contenedor

```bash
podman exec \
-it web bash
```

Consultar:

```bash
ip addr
```

---

# Probar Conectividad

```bash
ping 8.8.8.8
```

---

# Probar DNS

```bash
ping google.com
```

Si:

```text
8.8.8.8 funciona

↓

google.com falla
```

El problema normalmente es:

```text
DNS
```

---

# Consultar DNS

Dentro del contenedor

```bash
cat /etc/resolv.conf
```

---

# Verificar Puertos

```bash
ss -tln
```

---

# Verificar Publicación

```bash
podman port web
```

Ejemplo

```text
80/tcp -> 0.0.0.0:8080
```

---

# Puerto Ocupado

Consultar

```bash
ss -tulpn
```

Buscar

```text
LISTEN
```

---

# Arquitectura

```text
Cliente

↓

Host

↓

Puerto

↓

Contenedor

↓

Aplicación
```

---

# Problemas de Almacenamiento

Los síntomas más comunes:

- Archivos desaparecen.
- La aplicación no guarda datos.
- No puede escribir.
- El disco está lleno.

---

# Consultar Volúmenes

```bash
podman volume ls
```

---

# Inspeccionar Volumen

```bash
podman volume inspect datos
```

---

# Consultar Montajes

```bash
podman inspect web
```

Buscar:

```text
Mounts
```

---

# Consultar Espacio

```bash
df -h
```

---

# Consultar Inodos

```bash
df -i
```

---

# Consultar Uso de Podman

```bash
podman system df
```

---

# Limpiar Recursos

Eliminar contenedores detenidos

```bash
podman container prune
```

---

Eliminar imágenes sin uso

```bash
podman image prune
```

---

Eliminar volúmenes no utilizados

```bash
podman volume prune
```

---

Eliminar redes sin uso

```bash
podman network prune
```

---

Limpiar todo

```bash
podman system prune
```

---

# Advertencia

Antes de ejecutar:

```bash
podman system prune
```

Verifica cuidadosamente qué recursos serán eliminados.

En producción podría eliminar:

- imágenes
- volúmenes
- redes
- caché

---

# Problemas de Permisos

Ejemplo

```text
Permission denied
```

Consultar

```bash
ls -l
```

Consultar propietario

```bash
stat archivo
```

---

# Consultar Usuario

```bash
id
```

---

# Comparar UID

Host

```bash
id
```

Contenedor

```bash
podman exec \
-it web id
```

---

# Rootless

Solo puede escribir donde el usuario tenga permisos.

Ejemplo correcto

```text
/home/usuario
```

---

Ejemplo incorrecto

```text
/root
```

---

# Problemas con SELinux

Uno de los errores más comunes.

Consultar

```bash
getenforce
```

Resultado

```text
Enforcing
```

---

Consultar Estado

```bash
sestatus
```

---

Consultar Contextos

```bash
ls -lZ
```

---

Ejemplo

```text
system_u

object_r

container_file_t
```

---

# Bind Mount

Incorrecto

```bash
-v /datos:/datos
```

---

Correcto

```bash
-v /datos:/datos:Z
```

---

Compartido

```bash
-v /datos:/datos:z
```

---

# Consultar AVC

```bash
journalctl | grep AVC
```

---

También

```bash
ausearch -m avc
```

---

# Arquitectura

```text
Host

↓

SELinux

↓

Volumen

↓

Contenedor
```

---

# Problemas Rootless

Consultar

```bash
podman info \
--format '{{.Host.Security.Rootless}}'
```

---

Consultar Storage

```bash
podman info \
--format '{{.Store.GraphRoot}}'
```

---

Consultar Runtime

```bash
echo $XDG_RUNTIME_DIR
```

---

Consultar Linger

```bash
loginctl show-user $USER
```

---

# Problemas Rootful

Consultar

```bash
sudo podman info
```

Comparar con

```bash
podman info
```

Verificar:

- imágenes
- volúmenes
- redes
- configuración

---

# Problemas con Systemd

Consultar

```bash
systemctl status \
container-web.service
```

---

Consultar Logs

```bash
journalctl \
-u container-web.service
```

---

Consultar Propiedades

```bash
systemctl show \
container-web.service
```

---

Consultar Dependencias

```bash
systemctl list-dependencies \
container-web.service
```

---

# Problemas con Quadlets

Consultar directorio

```bash
ls ~/.config/containers/systemd
```

---

Recargar

```bash
systemctl --user daemon-reload
```

---

Verificar

```bash
systemctl --user status web.service
```

---

Validar

```bash
systemd-analyze verify \
~/.config/containers/systemd/web.container
```

---

# Diagnóstico por Categorías

| Síntoma | Posible causa | Herramienta |
|---------|---------------|-------------|
| No inicia | Imagen | podman images |
| No responde | Red | podman network |
| Permission denied | Linux | ls -l |
| Permission denied | SELinux | ls -lZ |
| Sin espacio | Disco | df -h |
| Sin memoria | RAM | free -h |
| Lento | CPU | top |
| Puerto ocupado | Red | ss |
| No guarda datos | Volúmenes | podman volume inspect |
| Error systemd | Servicio | systemctl status |

---

# Escenario 1

## Error de Puerto

```text
listen tcp :8080

address already in use
```

Consultar

```bash
ss -tulpn
```

---

# Escenario 2

## Volumen vacío

Consultar

```bash
podman inspect
```

↓

```bash
podman volume inspect
```

↓

Verificar montaje.

---

# Escenario 3

## No guarda información

Consultar

```bash
mount
```

↓

Verificar volumen.

↓

Verificar permisos.

---

# Escenario 4

## Rootless no publica puerto

Consultar

```bash
sysctl \
net.ipv4.ip_unprivileged_port_start
```

---

# Escenario 5

## Imagen dañada

Eliminar

```bash
podman rmi imagen
```

Volver a descargar

```bash
podman pull imagen
```

---

# Escenario 6

## Problema DNS

Entrar

```bash
podman exec \
-it web bash
```

↓

Consultar

```bash
cat /etc/resolv.conf
```

↓

Realizar

```bash
ping google.com
```

---

# Escenario 7

## SELinux bloquea volumen

Consultar

```bash
ls -lZ
```

↓

Agregar

```text
:Z
```

↓

Recrear contenedor.

---

# Escenario 8

## Disco lleno

Consultar

```bash
df -h
```

↓

```bash
podman system df
```

↓

```bash
podman system prune
```

---

# Escenario 9

## Servicio no inicia automáticamente

Consultar

```bash
systemctl is-enabled
```

↓

```bash
systemctl enable
```

---

# Escenario 10

## Quadlet ignorado

Consultar

```bash
daemon-reload
```

↓

Verificar sintaxis.

↓

Consultar Journal.

---

# Laboratorio RHCSA

## Laboratorio 1

Consultar redes.

```bash
podman network ls
```

---

## Laboratorio 2

Consultar IP.

```bash
podman inspect \
--format '{{.NetworkSettings.IPAddress}}' \
web
```

---

## Laboratorio 3

Consultar puertos.

```bash
podman port web
```

---

## Laboratorio 4

Consultar DNS.

```bash
cat /etc/resolv.conf
```

---

## Laboratorio 5

Consultar volúmenes.

```bash
podman volume ls
```

---

## Laboratorio 6

Consultar almacenamiento.

```bash
podman system df
```

---

## Laboratorio 7

Consultar permisos.

```bash
ls -l
```

---

## Laboratorio 8

Consultar SELinux.

```bash
ls -lZ
```

---

## Laboratorio 9

Consultar estado.

```bash
getenforce
```

---

## Laboratorio 10

Consultar AVC.

```bash
ausearch -m avc
```

---

## Laboratorio 11

Consultar Rootless.

```bash
podman info \
--format '{{.Host.Security.Rootless}}'
```

---

## Laboratorio 12

Consultar GraphRoot.

```bash
podman info \
--format '{{.Store.GraphRoot}}'
```

---

## Laboratorio 13

Consultar servicio.

```bash
systemctl status \
container-web.service
```

---

## Laboratorio 14

Consultar Journal.

```bash
journalctl \
-u container-web.service
```

---

## Laboratorio 15

Simular un problema de permisos en un volumen, identificar la causa utilizando las herramientas aprendidas y corregirlo sin deshabilitar SELinux.

---

# Buenas prácticas

- Diagnosticar un único problema a la vez.
- No desactivar SELinux para resolver errores de permisos.
- Revisar siempre el contexto SELinux antes de modificar permisos.
- Limpiar recursos de Podman únicamente cuando se conozca el impacto.
- Mantener separados los recursos Rootless y Rootful.
- Verificar la red antes de modificar la aplicación.

---

# Errores comunes

## Error 1

Ejecutar `chmod 777` para resolver un problema de permisos sin investigar la causa real.

---

## Error 2

Desactivar SELinux (`setenforce 0`) como solución permanente.

---

## Error 3

Eliminar todos los recursos con `podman system prune` en un servidor de producción.

---

## Error 4

Confundir un problema de DNS con un problema de conectividad IP.

---

## Error 5

Olvidar ejecutar `systemctl --user daemon-reload` después de modificar un Quadlet.

---

# Resumen

En esta segunda fase aprendimos a:

- Diagnosticar problemas de red, DNS y puertos.
- Resolver incidencias relacionadas con almacenamiento y volúmenes.
- Identificar errores de permisos Linux y de SELinux.
- Diferenciar problemas entre Rootless y Rootful.
- Diagnosticar servicios administrados por systemd y Quadlets.
- Aplicar una metodología estructurada para resolver problemas reales en entornos Fedora y Red Hat Enterprise Linux.

En la **Fase 3** aprenderemos técnicas avanzadas de troubleshooting, incluyendo análisis de recursos, depuración de procesos, problemas de rendimiento, monitoreo, recuperación de contenedores, análisis forense, herramientas avanzadas del kernel y escenarios empresariales complejos orientados al examen RHCSA.

----

# 76. Troubleshooting de Contenedores con Podman (Fase 3)

> **Módulo 10 – Contenedores con Podman**
>
> **Manual RHCSA**
>
> **Archivo:** `76-troubleshooting-contenedores-podman.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase podrás:

- Diagnosticar problemas de rendimiento.
- Analizar consumo de CPU, memoria y almacenamiento.
- Identificar cuellos de botella.
- Recuperar contenedores dañados.
- Utilizar herramientas avanzadas de Linux para diagnóstico.
- Analizar problemas relacionados con cgroups y namespaces.
- Aplicar procedimientos utilizados por administradores Linux Senior.

---

# Introducción

No todos los problemas producen mensajes de error.

En muchos casos el contenedor:

- responde lentamente,
- consume demasiados recursos,
- deja de responder ocasionalmente,
- se reinicia sin explicación,
- provoca lentitud en todo el servidor.

Estos problemas requieren un análisis mucho más profundo que simplemente revisar los logs.

---

# Metodología de Diagnóstico Avanzado

```text
Problema

↓

Recursos

↓

Procesos

↓

Kernel

↓

Storage

↓

Network

↓

Application

↓

Root Cause
```

---

# Diagnóstico de Rendimiento

Siempre responder estas preguntas:

- ¿CPU?
- ¿RAM?
- ¿Swap?
- ¿Disco?
- ¿I/O?
- ¿Red?
- ¿Kernel?
- ¿Aplicación?

Nunca asumir una respuesta.

---

# Verificar utilización del sistema

```bash
top
```

---

## Información importante

```text
Load Average

CPU

Memory

Swap

Running Tasks
```

---

# htop

Si está instalado:

```bash
htop
```

Ventajas:

- Vista interactiva.
- Ordenar procesos.
- Buscar procesos.
- Filtrar usuarios.
- Visualización de CPU por núcleo.

---

# Arquitectura

```text
Host

↓

top

↓

CPU

↓

Memory

↓

Swap

↓

Procesos
```

---

# Analizar CPU

Consultar

```bash
mpstat -P ALL
```

---

Consultar

```bash
sar -u
```

---

Consultar

```bash
vmstat 2
```

---

# Interpretación

CPU elevada no significa necesariamente un problema.

Preguntas:

- ¿Qué proceso consume CPU?
- ¿Es esperado?
- ¿Es temporal?
- ¿Es permanente?

---

# Analizar Memoria

Consultar

```bash
free -h
```

---

Consultar

```bash
cat /proc/meminfo
```

---

Consultar

```bash
vmstat
```

---

Campos importantes

```text
MemAvailable

Buffers

Cached

SwapFree
```

---

# Analizar Swap

Consultar

```bash
swapon --show
```

---

Consultar

```bash
free -h
```

---

Si Swap aumenta continuamente:

Puede existir presión de memoria.

---

# Analizar Disco

Consultar

```bash
df -h
```

---

Consultar inodos

```bash
df -i
```

---

Consultar bloques

```bash
lsblk
```

---

Arquitectura

```text
Filesystem

↓

Espacio

↓

Inodos

↓

Montajes
```

---

# Analizar I/O

Consultar

```bash
iostat
```

---

Consultar

```bash
iotop
```

(si está instalado)

---

Consultar

```bash
pidstat -d
```

---

# Analizar Red

Consultar

```bash
ip addr
```

---

Consultar rutas

```bash
ip route
```

---

Consultar conexiones

```bash
ss -tulpn
```

---

Consultar tráfico

```bash
ip -s link
```

---

# Diagnóstico de Podman

Consultar estadísticas

```bash
podman stats
```

---

Consultar una vez

```bash
podman stats --no-stream
```

---

Campos

```text
CPU

MEM

MEM LIMIT

NET I/O

BLOCK I/O
```

---

# Interpretación

CPU alta

↓

Aplicación

Memoria alta

↓

Fuga de memoria

I/O alto

↓

Disco lento

---

# Analizar Procesos

Consultar

```bash
podman top web
```

---

Consultar dentro del contenedor

```bash
podman exec -it web bash
```

Después

```bash
ps aux
```

---

Consultar árbol

```bash
pstree
```

---

# Identificar PID

```bash
podman inspect \
--format '{{.State.Pid}}' web
```

---

Consultar proceso

```bash
ps -fp PID
```

---

# Analizar Namespaces

Consultar

```bash
lsns
```

---

Resultado

```text
PID

TYPE

NS

COMMAND
```

---

Tipos

```text
PID

NET

IPC

UTS

USER

MNT

CGROUP
```

---

# Analizar Cgroups

Consultar

```bash
systemd-cgls
```

---

Consultar

```bash
systemd-cgtop
```

---

Arquitectura

```text
Kernel

↓

Cgroups

↓

CPU

↓

Memory

↓

IO
```

---

# Consultar Límites

```bash
podman inspect web
```

Buscar

```text
Memory

CpuShares

CpuQuota
```

---

# Diagnóstico de Almacenamiento

Consultar

```bash
podman system df
```

---

Consultar imágenes

```bash
podman images
```

---

Consultar capas

```bash
podman image inspect
```

---

Consultar almacenamiento

```bash
podman info
```

Buscar

```text
GraphRoot
```

---

# Analizar OverlayFS

Consultar

```bash
mount | grep overlay
```

---

Consultar GraphRoot

```bash
podman info \
--format '{{.Store.GraphRoot}}'
```

---

# Recuperación de Contenedores

Primer paso

No eliminar inmediatamente.

Consultar

```bash
podman inspect
```

↓

Guardar información.

↓

Consultar logs.

↓

Analizar.

---

# Exportar Configuración

```bash
podman inspect web \
> web.json
```

---

Guardar Logs

```bash
podman logs web \
> web.log
```

---

Guardar Estado

```bash
podman ps -a
```

---

# Recuperar Imagen

Consultar

```bash
podman images
```

---

Descargar nuevamente

```bash
podman pull nginx
```

---

# Exportar Contenedor

```bash
podman export web \
-o web.tar
```

---

Importar

```bash
podman import web.tar
```

---

# Backup de Volúmenes

Consultar

```bash
podman volume ls
```

---

Respaldar

```bash
tar -cvf volumen.tar \
/var/lib/containers/storage/volumes
```

---

# Diagnóstico del Kernel

Consultar

```bash
dmesg
```

---

Consultar últimos mensajes

```bash
dmesg | tail
```

---

Consultar errores

```bash
journalctl -k
```

---

# OOM Killer

Consultar

```bash
journalctl -k | grep -i oom
```

---

También

```bash
dmesg | grep -i killed
```

---

Arquitectura

```text
Kernel

↓

OOM Killer

↓

Proceso eliminado

↓

Container detenido
```

---

# Escenario 1

## Consumo elevado de CPU

Herramientas

```bash
top

↓

podman stats

↓

podman top
```

---

# Escenario 2

## Memoria agotada

Consultar

```bash
free -h
```

↓

```bash
journalctl -k
```

↓

Buscar OOM.

---

# Escenario 3

## Disco lleno

Consultar

```bash
df -h
```

↓

```bash
podman system df
```

↓

```bash
du -sh
```

---

# Escenario 4

## I/O lento

Consultar

```bash
iostat
```

↓

```bash
iotop
```

↓

```bash
pidstat
```

---

# Escenario 5

## Aplicación lenta

Consultar

```bash
podman exec \
-it web bash
```

↓

```bash
top
```

↓

```bash
curl localhost
```

---

# Escenario 6

## Kernel elimina el contenedor

Consultar

```bash
journalctl -k
```

↓

Buscar

```text
Out of memory
```

---

# Escenario 7

## Namespaces dañados

Consultar

```bash
lsns
```

↓

Comparar

```bash
podman inspect
```

---

# Escenario 8

## Cgroups saturados

Consultar

```bash
systemd-cgtop
```

↓

Analizar consumo.

---

# Script de Diagnóstico

Guardar como

```text
performance_check.sh
```

```bash
#!/bin/bash

echo "==========================="
echo "PERFORMANCE REPORT"
echo "==========================="

echo
echo "CPU"
uptime

echo
echo "MEMORIA"
free -h

echo
echo "DISCO"
df -h

echo
echo "INODOS"
df -i

echo
echo "PODMAN"
podman stats --no-stream

echo
echo "ALMACENAMIENTO"
podman system df

echo
echo "CGROUPS"
systemd-cgtop --batch --iterations=1
```

---

# Laboratorio RHCSA

## Laboratorio 1

Consultar CPU.

```bash
top
```

---

## Laboratorio 2

Consultar memoria.

```bash
free -h
```

---

## Laboratorio 3

Consultar Swap.

```bash
swapon --show
```

---

## Laboratorio 4

Consultar disco.

```bash
df -h
```

---

## Laboratorio 5

Consultar inodos.

```bash
df -i
```

---

## Laboratorio 6

Consultar estadísticas.

```bash
podman stats
```

---

## Laboratorio 7

Consultar procesos.

```bash
podman top web
```

---

## Laboratorio 8

Consultar PID.

```bash
podman inspect \
--format '{{.State.Pid}}' web
```

---

## Laboratorio 9

Consultar namespaces.

```bash
lsns
```

---

## Laboratorio 10

Consultar cgroups.

```bash
systemd-cgls
```

---

## Laboratorio 11

Consultar consumo.

```bash
systemd-cgtop
```

---

## Laboratorio 12

Consultar almacenamiento.

```bash
podman system df
```

---

## Laboratorio 13

Consultar mensajes del Kernel.

```bash
journalctl -k
```

---

## Laboratorio 14

Buscar eventos OOM.

```bash
journalctl -k | grep -i oom
```

---

## Laboratorio 15

Ejecutar el script `performance_check.sh`, analizar el estado del servidor e identificar posibles cuellos de botella antes de proponer acciones correctivas.

---

# Buenas prácticas

- Analizar primero el Host y luego el contenedor.
- Monitorear el uso de CPU, memoria e I/O de forma periódica.
- Mantener respaldos de configuraciones y volúmenes antes de realizar cambios importantes.
- Investigar eventos OOM antes de aumentar los límites de memoria.
- Revisar los cgroups cuando varios contenedores compiten por recursos.
- Documentar todos los hallazgos y las acciones realizadas durante el proceso de diagnóstico.

---

# Errores comunes

## Error 1

Asumir que toda lentitud proviene del contenedor y no del Host.

---

## Error 2

Eliminar un contenedor antes de recopilar evidencias (`inspect`, `logs` y estado).

---

## Error 3

Ignorar los mensajes del Kernel relacionados con OOM Killer.

---

## Error 4

No verificar el consumo de I/O cuando el CPU y la memoria parecen normales.

---

## Error 5

No revisar los límites de recursos definidos mediante cgroups.

---

# Resumen

En esta tercera fase aprendimos a:

- Diagnosticar problemas avanzados de rendimiento.
- Analizar CPU, memoria, swap, disco e I/O.
- Utilizar herramientas como `top`, `vmstat`, `iostat`, `lsns`, `systemd-cgtop` y `journalctl -k`.
- Comprender la relación entre Podman, cgroups y namespaces.
- Recuperar información crítica antes de eliminar un contenedor.
- Identificar eventos del Kernel, incluyendo terminaciones por OOM Killer.
- Aplicar procedimientos avanzados de troubleshooting utilizados en entornos empresariales.

En la **Fase 4** integraremos todos los conocimientos del capítulo mediante escenarios reales de producción, análisis forense, procedimientos completos de recuperación, checklist profesional de troubleshooting, preguntas de repaso y un desafío final similar a los laboratorios del examen RHCSA.

-----

# 76. Troubleshooting de Contenedores con Podman (Fase 4)

> **Módulo 10 – Contenedores con Podman**
>
> **Manual RHCSA**
>
> **Archivo:** `76-troubleshooting-contenedores-podman.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase podrás:

- Resolver incidencias complejas en producción.
- Aplicar una metodología completa de troubleshooting.
- Analizar fallos desde el Host hasta la aplicación.
- Implementar procedimientos de recuperación.
- Automatizar auditorías de Podman.
- Prepararte para escenarios similares al examen RHCSA.

---

# Introducción

En un ambiente empresarial un administrador Linux rara vez recibe un mensaje como:

> "El contenedor tiene un problema."

Lo habitual es recibir reportes como:

- "La aplicación está lenta."
- "La API dejó de responder."
- "El sitio web está caído."
- "No puedo acceder al servicio."
- "Después del reinicio ya no funciona."
- "Consumió toda la memoria."
- "Los datos desaparecieron."

La responsabilidad del administrador consiste en encontrar la causa raíz utilizando un procedimiento ordenado.

---

# Metodología Profesional

```text
Usuario reporta problema

        │

        ▼

Verificar Host

        │

        ▼

Verificar Systemd

        │

        ▼

Verificar Contenedor

        │

        ▼

Verificar Red

        │

        ▼

Verificar Storage

        │

        ▼

Verificar Aplicación

        │

        ▼

Resolver

        │

        ▼

Documentar
```

---

# Checklist Inicial

Antes de modificar cualquier configuración responder:

```text
□ ¿El Host funciona?

□ ¿Existe espacio?

□ ¿Existe memoria?

□ ¿Está ejecutándose?

□ ¿Systemd lo inició?

□ ¿Existe conectividad?

□ ¿Los logs muestran errores?

□ ¿SELinux bloquea?

□ ¿El volumen está montado?

□ ¿La aplicación responde?
```

---

# Escenario Empresarial 1

## El servicio web no responde

Usuario:

```text
No puedo abrir la página.
```

Procedimiento

Consultar:

```bash
systemctl status container-web.service
```

↓

```bash
podman ps
```

↓

```bash
podman logs web
```

↓

```bash
ss -tulpn
```

↓

```bash
curl localhost
```

↓

Resolver.

---

# Escenario Empresarial 2

## Reinicios constantes

Consultar

```bash
systemctl status container-web.service
```

Buscar

```text
Restart Counter
```

↓

Consultar

```bash
podman logs web
```

↓

Consultar

```bash
journalctl \
-u container-web.service
```

Generalmente:

- aplicación falla
- variables incorrectas
- configuración inválida

---

# Escenario Empresarial 3

## Error después de reiniciar el servidor

Consultar

```bash
systemctl is-enabled \
container-web.service
```

↓

Consultar

```bash
systemctl list-dependencies \
container-web.service
```

↓

Consultar

```bash
systemctl status \
network-online.target
```

---

# Escenario Empresarial 4

## La aplicación perdió información

Consultar

```bash
podman inspect web
```

↓

Buscar

```text
Mounts
```

↓

Consultar

```bash
podman volume ls
```

↓

Consultar

```bash
podman volume inspect
```

---

# Escenario Empresarial 5

## SELinux bloquea acceso

Consultar

```bash
ls -lZ
```

↓

```bash
journalctl | grep AVC
```

↓

```bash
ausearch -m avc
```

↓

Corregir contexto.

Nunca:

```bash
setenforce 0
```

como solución permanente.

---

# Escenario Empresarial 6

## Rootless dejó de funcionar

Consultar

```bash
loginctl show-user $USER
```

↓

Buscar

```text
Linger=yes
```

↓

Consultar

```bash
echo $XDG_RUNTIME_DIR
```

↓

Consultar

```bash
systemctl --user status
```

---

# Escenario Empresarial 7

## El contenedor consume toda la memoria

Consultar

```bash
podman stats
```

↓

```bash
free -h
```

↓

```bash
journalctl -k
```

↓

Buscar

```text
OOM
```

---

# Escenario Empresarial 8

## El servidor completo está lento

Consultar

```bash
top
```

↓

```bash
vmstat
```

↓

```bash
iostat
```

↓

```bash
podman stats
```

↓

```bash
systemd-cgtop
```

---

# Escenario Empresarial 9

## No existe conectividad

Consultar

```bash
podman network ls
```

↓

```bash
podman network inspect
```

↓

```bash
ip addr
```

↓

```bash
ip route
```

↓

```bash
ping
```

---

# Escenario Empresarial 10

## DNS no funciona

Entrar

```bash
podman exec \
-it web bash
```

↓

Consultar

```bash
cat /etc/resolv.conf
```

↓

Consultar

```bash
ping google.com
```

↓

Consultar

```bash
dig google.com
```

(si está instalado)

---

# Escenario Empresarial 11

## Imagen dañada

Consultar

```bash
podman image inspect
```

↓

Eliminar

```bash
podman rmi imagen
```

↓

Descargar nuevamente

```bash
podman pull imagen
```

---

# Escenario Empresarial 12

## Quadlet no funciona

Consultar

```bash
systemctl --user daemon-reload
```

↓

```bash
systemctl --user status
```

↓

```bash
systemd-analyze verify
```

↓

```bash
journalctl --user
```

---

# Procedimiento Completo

```text
Problema

↓

systemctl

↓

journalctl

↓

podman ps

↓

podman logs

↓

podman inspect

↓

podman stats

↓

Network

↓

Storage

↓

SELinux

↓

Kernel

↓

Resolver
```

---

# Script Profesional de Auditoría

Guardar como

```text
podman_health_check.sh
```

```bash
#!/bin/bash

echo "======================================="
echo " PODMAN HEALTH CHECK"
echo "======================================="

echo
echo "Host:"
hostname

echo
echo "Fecha:"
date

echo
echo "======================================="
echo "SYSTEMD"
echo "======================================="
systemctl list-units --type=service --state=running

echo
echo "======================================="
echo "PODMAN"
echo "======================================="
podman ps -a

echo
echo "======================================="
echo "CPU"
echo "======================================="
uptime

echo
echo "======================================="
echo "MEMORIA"
echo "======================================="
free -h

echo
echo "======================================="
echo "DISCO"
echo "======================================="
df -h

echo
echo "======================================="
echo "ALMACENAMIENTO PODMAN"
echo "======================================="
podman system df

echo
echo "======================================="
echo "REDES"
echo "======================================="
podman network ls

echo
echo "======================================="
echo "VOLUMENES"
echo "======================================="
podman volume ls

echo
echo "======================================="
echo "SELINUX"
echo "======================================="
getenforce
```

---

# Script de Recolección de Evidencias

Guardar como

```text
collect_evidence.sh
```

```bash
#!/bin/bash

mkdir -p evidence

podman ps -a \
> evidence/containers.txt

podman images \
> evidence/images.txt

podman network ls \
> evidence/networks.txt

podman volume ls \
> evidence/volumes.txt

journalctl -n 500 \
> evidence/journal.txt

df -h \
> evidence/disk.txt

free -h \
> evidence/memory.txt

echo "Evidencias almacenadas."
```

---

# Script de Verificación de Contenedores

Guardar como

```text
check_containers.sh
```

```bash
#!/bin/bash

for c in $(podman ps --format "{{.Names}}")
do

echo
echo "==============================="
echo "$c"
echo "==============================="

podman inspect \
--format '{{.State.Status}}' \
"$c"

podman stats \
--no-stream \
"$c"

done
```

---

# Flujo de Recuperación

```text
Detectar problema

↓

Guardar evidencias

↓

Respaldar configuración

↓

Respaldar volumen

↓

Corregir

↓

Reiniciar

↓

Validar

↓

Documentar
```

---

# Checklist RHCSA

Antes de considerar resuelto un problema verificar:

```text
□ El Host responde.

□ CPU estable.

□ Memoria suficiente.

□ Disco disponible.

□ Inodos disponibles.

□ Contenedor activo.

□ Logs limpios.

□ Journal limpio.

□ SELinux correcto.

□ Volúmenes montados.

□ Red funcional.

□ DNS funcional.

□ Aplicación responde.

□ systemd habilitado.

□ Backup actualizado.
```

---

# Laboratorio RHCSA

## Laboratorio 1

Consultar estado del servicio.

```bash
systemctl status \
container-web.service
```

---

## Laboratorio 2

Consultar Journal.

```bash
journalctl \
-u container-web.service
```

---

## Laboratorio 3

Consultar logs.

```bash
podman logs web
```

---

## Laboratorio 4

Consultar inspect.

```bash
podman inspect web
```

---

## Laboratorio 5

Consultar estadísticas.

```bash
podman stats
```

---

## Laboratorio 6

Consultar Kernel.

```bash
journalctl -k
```

---

## Laboratorio 7

Consultar SELinux.

```bash
getenforce
```

---

## Laboratorio 8

Consultar almacenamiento.

```bash
podman system df
```

---

## Laboratorio 9

Consultar redes.

```bash
podman network ls
```

---

## Laboratorio 10

Consultar volúmenes.

```bash
podman volume ls
```

---

## Laboratorio 11

Ejecutar el script `podman_health_check.sh`.

---

## Laboratorio 12

Ejecutar `collect_evidence.sh`.

---

## Laboratorio 13

Ejecutar `check_containers.sh`.

---

## Laboratorio 14

Simular un fallo de red, recopilar evidencias, restaurar la conectividad y documentar cada paso realizado.

---

## Laboratorio 15

Diseñar un procedimiento completo de troubleshooting para un servidor Fedora con cuatro contenedores (Nginx, PostgreSQL, Redis y una API), identificando la causa raíz de un problema simulado, aplicando la corrección adecuada y verificando el correcto funcionamiento de todos los servicios.

---

# Preguntas de Repaso

1. ¿Cuál es el primer paso antes de modificar un contenedor con problemas?
2. ¿Qué diferencia existe entre `podman logs` y `journalctl`?
3. ¿Qué herramienta muestra el consumo de recursos de los contenedores?
4. ¿Cómo identificar un problema causado por OOM Killer?
5. ¿Qué comando permite inspeccionar completamente un contenedor?
6. ¿Qué utilidad tiene `systemd-cgtop`?
7. ¿Cómo verificar el estado de un volumen utilizado por un contenedor?
8. ¿Por qué no es recomendable desactivar SELinux como solución permanente?
9. ¿Qué información proporciona `podman network inspect`?
10. ¿Por qué es importante recopilar evidencias antes de eliminar un contenedor?

---

# Respuestas

1. Recopilar información y evidencias del problema.
2. `podman logs` muestra la salida del contenedor; `journalctl` muestra los registros administrados por systemd y el sistema.
3. `podman stats`.
4. Revisando `journalctl -k` o `dmesg` en busca de eventos OOM.
5. `podman inspect`.
6. Mostrar el consumo de recursos por cgroups administrados por systemd.
7. Mediante `podman volume inspect`.
8. Porque reduce la seguridad del sistema y oculta la causa real del problema.
9. Información sobre la configuración de la red, subredes, gateway y contenedores conectados.
10. Porque permite analizar la causa raíz y facilita la recuperación sin perder información importante.

---

# Desafío Final RHCSA

Dispones de un servidor Fedora que ejecuta los siguientes servicios mediante Podman:

- Nginx
- PostgreSQL
- Redis
- API REST

El servidor presenta los siguientes síntomas:

- La API no responde.
- Nginx devuelve errores **502 Bad Gateway**.
- Redis consume mucha memoria.
- PostgreSQL tarda varios segundos en responder.
- El Journal muestra errores relacionados con un volumen.
- SELinux registra eventos AVC.
- Después del último reinicio algunos servicios no iniciaron automáticamente.

Debes:

1. Aplicar la metodología completa de troubleshooting.
2. Identificar todas las causas raíz.
3. Corregir cada incidencia sin desactivar SELinux.
4. Validar el funcionamiento de cada contenedor.
5. Documentar las acciones realizadas.
6. Generar un informe de auditoría utilizando los scripts desarrollados durante este capítulo.

---

# Buenas prácticas

- Aplicar siempre una metodología estructurada.
- Recopilar evidencias antes de modificar configuraciones.
- Revisar tanto el Host como el contenedor.
- Mantener respaldos de configuraciones y volúmenes.
- Automatizar verificaciones mediante scripts.
- Documentar todas las incidencias y su resolución.
- Realizar pruebas de recuperación de forma periódica.

---

# Errores comunes

## Error 1

Eliminar un contenedor antes de guardar los logs y el resultado de `podman inspect`.

---

## Error 2

Centrarse únicamente en el contenedor sin revisar el estado del Host.

---

## Error 3

Ignorar eventos del Kernel y de SELinux durante el diagnóstico.

---

## Error 4

Modificar varias configuraciones simultáneamente, dificultando identificar la causa real.

---

## Error 5

No validar la solución después de aplicar los cambios.

---

# Resumen del Capítulo 76

En este capítulo aprendimos a:

- Aplicar una metodología profesional de troubleshooting.
- Diagnosticar problemas relacionados con contenedores, Host, redes, almacenamiento y recursos.
- Analizar CPU, memoria, disco, I/O, cgroups y namespaces.
- Resolver incidencias de permisos, SELinux, Rootless, Rootful y systemd.
- Automatizar auditorías mediante scripts Bash.
- Recuperar contenedores y recopilar evidencias antes de intervenir.
- Resolver escenarios reales utilizados en entornos empresariales con Fedora y Red Hat Enterprise Linux.

Con este capítulo finaliza el bloque dedicado a **Troubleshooting de Contenedores con Podman**, proporcionando las habilidades necesarias para administrar y diagnosticar contenedores en escenarios similares a los evaluados en el examen **RHCSA** y utilizados en ambientes de producción.

---

# Fin del Capítulo

<!-- ```text
76-troubleshooting-contenedores-podman.md
```

**Próximo capítulo:**

```text
77-resumen-general-contenedores-rhcsa.md
```
 -->

















