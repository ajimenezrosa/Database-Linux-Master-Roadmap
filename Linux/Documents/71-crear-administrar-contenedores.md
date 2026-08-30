# 71. Crear y Administrar Contenedores (Fase 1)

> **Módulo 10 – Contenedores con Podman**
>
> **Manual RHCSA**
>
> **Archivo:** `71-crear-administrar-contenedores.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase podrás:

- Comprender qué es un contenedor.
- Diferenciar una imagen de un contenedor.
- Comprender el ciclo de vida de un contenedor.
- Crear contenedores utilizando Podman.
- Comprender las diferencias entre `create` y `run`.
- Ejecutar contenedores interactivos y en segundo plano.
- Asignar nombres personalizados.
- Configurar variables de entorno.
- Publicar puertos.
- Configurar el hostname del contenedor.
- Configurar políticas de reinicio.
- Eliminar automáticamente contenedores temporales.
- Prepararte para los ejercicios prácticos del examen RHCSA.

---

# Introducción

Hasta este momento hemos aprendido:

- Instalar Podman.
- Configurar Podman.
- Administrar imágenes.

Ahora llega el momento más importante del módulo:

**Crear contenedores.**

Un contenedor representa una instancia en ejecución creada a partir de una imagen.

La imagen permanece inmutable.

El contenedor es quien ejecuta procesos.

---

# ¿Qué es un contenedor?

Un contenedor es un proceso aislado que comparte el kernel del sistema operativo anfitrión.

Puede contener:

- Aplicaciones
- Bibliotecas
- Archivos
- Variables de entorno
- Configuración
- Procesos

Todo ello ejecutándose de forma aislada.

---

# Imagen vs Contenedor

```text
             Imagen

             alpine

                │

          podman run

                │

                ▼

         Contenedor

        alpine-test
```

La imagen nunca cambia.

El contenedor sí.

---

# Características de un contenedor

Un contenedor:

- Comparte el kernel del host.
- Tiene su propio sistema de archivos.
- Tiene su propia red.
- Posee su propio espacio de procesos.
- Puede limitar CPU y memoria.
- Puede eliminarse sin afectar la imagen original.

---

# Contenedor vs Máquina Virtual

| Contenedor | Máquina Virtual |
|------------|-----------------|
| Comparte el kernel | Tiene kernel propio |
| Muy ligero | Más pesado |
| Inicio en segundos | Inicio en minutos |
| Consume poca memoria | Consume más memoria |
| Excelente para microservicios | Excelente para aislamiento completo |

---

# Arquitectura

```text
                Usuario

                   │

                   ▼

               Podman CLI

                   │

                   ▼

             Imagen OCI

                   │

                   ▼

            Contenedor

                   │

                   ▼

             Kernel Linux
```

---

# Ciclo de vida

Todo contenedor pasa por diferentes estados.

```text
Imagen

   │

   ▼

Create

   │

   ▼

Created

   │

   ▼

Start

   │

   ▼

Running

   │

   ▼

Stop

   │

   ▼

Exited

   │

   ▼

Remove
```

---

# Estados principales

| Estado | Descripción |
|----------|-------------|
| Created | Creado |
| Running | Ejecutándose |
| Paused | Pausado |
| Stopped | Detenido |
| Exited | Finalizado |
| Removed | Eliminado |

---

# Comando create

Sintaxis:

```bash
podman create [opciones] imagen
```

Este comando:

- crea el contenedor
- NO lo inicia

Ejemplo:

```bash
podman create alpine
```

---

# Verificar

```bash
podman ps -a
```

Resultado:

```text
STATUS

Created
```

---

# ¿Cuándo utilizar create?

Es útil cuando queremos:

- preparar un contenedor
- modificar parámetros
- iniciarlo posteriormente

---

# Comando run

Es el comando más utilizado.

Sintaxis:

```bash
podman run imagen
```

Internamente realiza:

```text
Create

↓

Start
```

en una sola operación.

---

# Ejemplo

```bash
podman run alpine
```

El contenedor se ejecutará inmediatamente.

---

# Diferencias entre create y run

| create | run |
|---------|-----|
| Solo crea | Crea e inicia |
| No ejecuta procesos | Ejecuta inmediatamente |
| Estado Created | Estado Running |

---

# Contenedor interactivo

Ejecutar:

```bash
podman run -it alpine /bin/sh
```

Opciones:

```text
-i

Entrada estándar
```

```text
-t

Terminal interactiva
```

---

# Salir

Dentro del contenedor:

```bash
exit
```

---

# Ejecutar Bash

Algunas imágenes incluyen Bash.

```bash
podman run -it ubuntu bash
```

---

# Ejecutar Shell

En Alpine:

```bash
podman run -it alpine sh
```

---

# Ejecutar un comando

```bash
podman run alpine uname -a
```

El contenedor ejecuta el comando.

Después termina.

---

# Ejecutar múltiples comandos

```bash
podman run alpine sh -c \
"date && hostname && id"
```

---

# Contenedores temporales

Agregar:

```bash
--rm
```

Ejemplo:

```bash
podman run --rm alpine date
```

Cuando termina:

El contenedor desaparece.

---

# ¿Cuándo utilizar --rm?

Muy útil para:

- pruebas
- scripts
- automatización
- pipelines CI/CD

---

# Ejecutar en segundo plano

Agregar:

```bash
-d
```

Ejemplo:

```bash
podman run -d nginx
```

---

# ¿Qué significa Detached?

El usuario recupera inmediatamente el prompt.

El contenedor continúa ejecutándose.

---

# Visualización

```text
Usuario

│

Prompt disponible

│

▼

Contenedor

Running
```

---

# Obtener ID

Al ejecutar:

```bash
podman run -d nginx
```

Se devuelve:

```text
4cfdd8ab4...
```

Es el Container ID.

---

# Asignar un nombre

```bash
podman run \
--name web \
nginx
```

---

# Beneficio

En lugar de:

```text
4cfdd8ab4
```

podemos utilizar:

```text
web
```

---

# Nombres únicos

Cada contenedor debe poseer un nombre distinto.

Ejemplo:

```text
web

db

redis

monitor
```

---

# Error común

Intentar crear:

```bash
podman run \
--name web \
nginx
```

cuando ya existe:

```text
web
```

Resultado:

```text
Error:
container name already exists
```

---

# Variables de entorno

Sintaxis:

```bash
-e
```

Ejemplo:

```bash
podman run \
-e APP=Produccion \
alpine env
```

---

# Varias variables

```bash
podman run \
-e APP=Web \
-e DB=Postgres \
-e VERSION=17 \
alpine env
```

---

# Archivo de variables

También es posible utilizar:

```bash
--env-file
```

Ejemplo:

```bash
podman run \
--env-file variables.env \
alpine
```

Contenido:

```text
APP=WEB

DB=POSTGRES

ENV=PROD
```

---

# Hostname

Asignar:

```bash
--hostname
```

Ejemplo:

```bash
podman run \
--hostname servidorweb \
alpine hostname
```

Resultado:

```text
servidorweb
```

---

# Publicación de puertos

Uno de los parámetros más importantes.

Sintaxis:

```bash
-p HOST:CONTENEDOR
```

---

# Ejemplo

```bash
podman run \
-d \
-p 8080:80 \
nginx
```

Interpretación:

```text
Host

8080

↓

Contenedor

80
```

---

# Otro ejemplo

```bash
podman run \
-d \
-p 5432:5432 \
postgres
```

---

# Publicar varios puertos

```bash
podman run \
-p 80:80 \
-p 443:443 \
httpd
```

---

# Verificar puertos

```bash
podman port web
```

---

# Reinicio automático

Parámetro:

```bash
--restart
```

---

# Opciones disponibles

| Política | Descripción |
|-----------|-------------|
| no | Nunca reiniciar |
| on-failure | Solo si falla |
| always | Siempre |
| unless-stopped | Hasta detenerlo manualmente |

---

# Ejemplo

```bash
podman run \
-d \
--restart=always \
nginx
```

---

# Política on-failure

```bash
podman run \
--restart=on-failure \
imagen
```

Útil para procesos críticos.

---

# Política unless-stopped

Muy utilizada para servicios.

```bash
podman run \
-d \
--restart=unless-stopped \
nginx
```

---

# Contenedores efímeros

```bash
podman run \
--rm \
alpine hostname
```

No dejan rastros.

---

# Contenedores persistentes

Sin:

```bash
--rm
```

permanecerán disponibles.

---

# Verificar

```bash
podman ps -a
```

---

# Información resumida

```bash
podman ps
```

Muestra únicamente:

- Running

---

# Información completa

```bash
podman ps -a
```

Muestra:

- Running
- Exited
- Created

---

# Flujo completo

```text
Imagen

↓

podman run

↓

Container Created

↓

Running

↓

Exit

↓

Stopped

↓

Remove
```

---

# Laboratorio RHCSA

## Laboratorio 1

Crear un contenedor.

```bash
podman create alpine
```

---

## Laboratorio 2

Verificar.

```bash
podman ps -a
```

---

## Laboratorio 3

Ejecutar Alpine.

```bash
podman run alpine date
```

---

## Laboratorio 4

Ejecutar modo interactivo.

```bash
podman run -it alpine sh
```

---

## Laboratorio 5

Salir.

```bash
exit
```

---

## Laboratorio 6

Ejecutar en segundo plano.

```bash
podman run -d nginx
```

---

## Laboratorio 7

Asignar nombre.

```bash
podman run \
-d \
--name web \
nginx
```

---

## Laboratorio 8

Asignar variables.

```bash
podman run \
-e APP=WEB \
alpine env
```

---

## Laboratorio 9

Publicar un puerto.

```bash
podman run \
-d \
-p 8080:80 \
nginx
```

---

## Laboratorio 10

Asignar hostname.

```bash
podman run \
--hostname servidor1 \
alpine hostname
```

---

## Laboratorio 11

Usar política de reinicio.

```bash
podman run \
-d \
--restart=always \
nginx
```

---

## Laboratorio 12

Crear un contenedor temporal.

```bash
podman run \
--rm \
alpine hostname
```

---

# Buenas prácticas

- Utilizar nombres descriptivos para los contenedores.
- Ejecutar los contenedores en modo **Rootless** siempre que sea posible.
- Publicar únicamente los puertos estrictamente necesarios.
- Utilizar `--rm` para tareas temporales.
- Evitar ejecutar procesos innecesarios dentro de un mismo contenedor.
- Especificar versiones concretas de las imágenes en lugar de depender de `latest`.
- Documentar las variables de entorno utilizadas por cada aplicación.

---

# Errores comunes

## Error 1

Confundir `create` con `run`.

---

## Error 2

Olvidar publicar los puertos necesarios.

---

## Error 3

No asignar nombres a los contenedores.

---

## Error 4

Utilizar `latest` en producción.

---

## Error 5

Intentar crear un contenedor con un nombre ya existente.

---

# Resumen

En esta primera fase aprendimos:

- Qué es un contenedor.
- Su ciclo de vida completo.
- Diferencias entre `create` y `run`.
- Cómo ejecutar contenedores interactivos y en segundo plano.
- Cómo asignar nombres, variables de entorno y hostname.
- Cómo publicar puertos.
- Cómo configurar políticas de reinicio.
- Cuándo utilizar `--rm` para crear contenedores efímeros.

En la **Fase 2** aprenderemos a administrar el ciclo de vida de los contenedores mediante comandos como `podman start`, `stop`, `restart`, `pause`, `kill`, `logs`, `inspect`, `stats`, `top`, `diff` y otras herramientas fundamentales para la administración diaria y la preparación para el examen **RHCSA**.

---

# 71. Crear y Administrar Contenedores (Fase 2)

> **Módulo 10 – Contenedores con Podman**
>
> **Manual RHCSA**
>
> **Archivo:** `71-crear-administrar-contenedores.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase podrás:

- Administrar el ciclo de vida completo de un contenedor.
- Iniciar, detener y reiniciar contenedores.
- Pausar y reanudar procesos.
- Finalizar procesos de manera controlada o forzada.
- Renombrar contenedores.
- Eliminar contenedores.
- Inspeccionar contenedores.
- Consultar logs.
- Ver procesos internos.
- Monitorear recursos.
- Identificar cambios en el sistema de archivos.
- Consultar puertos publicados.
- Esperar la finalización de un contenedor.
- Resolver escenarios comunes del examen RHCSA.

---

# Introducción

Crear un contenedor es solamente el primer paso.

El trabajo diario de un administrador consiste principalmente en administrar contenedores ya existentes.

Las tareas más frecuentes son:

- iniciarlos
- detenerlos
- reiniciarlos
- inspeccionarlos
- consultar sus registros
- monitorear recursos
- eliminar los que ya no son necesarios

---

# El ciclo de vida administrativo

```text
             Create

                │

                ▼

            Running

                │

        ┌───────┴────────┐

        ▼                ▼

      Pause           Restart

        │                │

        ▼                ▼

     Unpause         Running

        │

        ▼

      Stop

        │

        ▼

     Exited

        │

        ▼

      Remove
```

---

# Listar contenedores

El comando más utilizado es:

```bash
podman ps
```

Muestra únicamente los contenedores en ejecución.

Ejemplo:

```text
CONTAINER ID

IMAGE

STATUS

PORTS

NAMES
```

---

# Mostrar todos

```bash
podman ps -a
```

Incluye:

- Running
- Created
- Exited
- Paused

---

# Mostrar únicamente IDs

```bash
podman ps -q
```

Muy útil para scripts.

---

# Mostrar el último contenedor

```bash
podman ps -l
```

---

# Filtrar por estado

```bash
podman ps \
--filter status=running
```

---

Otro ejemplo

```bash
podman ps \
--filter status=exited
```

---

# Filtrar por nombre

```bash
podman ps \
--filter name=web
```

---

# Filtrar por imagen

```bash
podman ps \
--filter ancestor=nginx
```

---

# Iniciar un contenedor

Si un contenedor se encuentra detenido:

```bash
podman start web
```

---

También puede utilizarse el ID:

```bash
podman start a82f39b
```

---

# Iniciar varios

```bash
podman start web db redis
```

---

# Verificar

```bash
podman ps
```

---

# Detener un contenedor

```bash
podman stop web
```

---

¿Qué ocurre?

Podman envía inicialmente:

```text
SIGTERM
```

para permitir que la aplicación cierre correctamente.

---

Luego de un tiempo

Si la aplicación no responde:

```text
SIGKILL
```

---

# Cambiar el tiempo de espera

```bash
podman stop \
--time 30 web
```

Esperará treinta segundos.

---

# Detener varios

```bash
podman stop web db redis
```

---

# Reiniciar

```bash
podman restart web
```

Internamente ejecuta:

```text
Stop

↓

Start
```

---

# Reiniciar varios

```bash
podman restart web db
```

---

# Pausar un contenedor

```bash
podman pause web
```

---

¿Qué sucede?

Los procesos permanecen en memoria.

Pero dejan de ejecutarse.

---

Visualización

```text
CPU

↓

0%

Procesos

↓

Congelados
```

---

# Consultar

```bash
podman ps
```

Estado:

```text
Paused
```

---

# Reanudar

```bash
podman unpause web
```

Los procesos continúan exactamente donde quedaron.

---

# ¿Cuándo utilizar Pause?

Muy útil para:

- mantenimiento
- pruebas
- liberar CPU temporalmente

---

# Finalizar inmediatamente

```bash
podman kill web
```

Este comando envía:

```text
SIGKILL
```

sin esperar.

---

# Enviar otra señal

Ejemplo:

```bash
podman kill \
--signal SIGTERM \
web
```

---

# Diferencias

| stop | kill |
|------|------|
| Ordenado | Inmediato |
| Espera | No espera |
| SIGTERM | SIGKILL |

---

# Renombrar

```bash
podman rename \
web servidor-web
```

---

Consultar

```bash
podman ps -a
```

---

# Eliminar un contenedor

```bash
podman rm web
```

---

Debe estar detenido.

Si está ejecutándose:

```text
Error
```

---

Eliminar forzadamente

```bash
podman rm -f web
```

---

Eliminar varios

```bash
podman rm web db redis
```

---

Eliminar todos

```bash
podman rm -a
```

---

# Inspeccionar un contenedor

Uno de los comandos más importantes.

```bash
podman inspect web
```

---

Información obtenida

- ID
- Imagen
- Estado
- IP
- Redes
- Variables
- Montajes
- Hostname
- PID
- Procesos
- Labels
- Entrypoint

---

Consultar únicamente IP

```bash
podman inspect \
--format '{{.NetworkSettings.IPAddress}}' \
web
```

---

Consultar Estado

```bash
podman inspect \
--format '{{.State.Status}}' \
web
```

---

Consultar PID

```bash
podman inspect \
--format '{{.State.Pid}}' \
web
```

---

Consultar Imagen

```bash
podman inspect \
--format '{{.ImageName}}' \
web
```

---

Consultar Hostname

```bash
podman inspect \
--format '{{.Config.Hostname}}' \
web
```

---

# Logs

Consultar:

```bash
podman logs web
```

---

Seguir los logs

```bash
podman logs -f web
```

---

Mostrar últimas líneas

```bash
podman logs \
--tail 20 web
```

---

Mostrar timestamps

```bash
podman logs -t web
```

---

# Procesos internos

Consultar:

```bash
podman top web
```

---

Ejemplo

```text
PID

USER

COMMAND
```

---

# Monitoreo

Consultar consumo

```bash
podman stats
```

---

Ejemplo

```text
CPU

MEMORIA

NET

BLOCK IO
```

---

Monitorear un contenedor

```bash
podman stats web
```

---

Salir

```
CTRL+C
```

---

# Cambios realizados

Consultar:

```bash
podman diff web
```

---

Ejemplo

```text
A

/etc/test
```

```text
C

/etc/passwd
```

```text
D

/tmp/archivo
```

---

Significado

| Letra | Acción |
|--------|---------|
| A | Archivo agregado |
| C | Archivo cambiado |
| D | Archivo eliminado |

---

# Consultar puertos

```bash
podman port web
```

Resultado

```text
80/tcp

↓

8080
```

---

# Esperar la finalización

```bash
podman wait web
```

El comando permanece esperando.

Cuando el contenedor termina:

devuelve:

```text
0
```

---

Muy utilizado en scripts.

---

# Flujo administrativo

```text
Run

↓

Running

↓

Logs

↓

Inspect

↓

Stats

↓

Stop

↓

Remove
```

---

# Laboratorio RHCSA

## Laboratorio 1

Mostrar contenedores activos.

```bash
podman ps
```

---

## Laboratorio 2

Mostrar todos.

```bash
podman ps -a
```

---

## Laboratorio 3

Iniciar un contenedor.

```bash
podman start web
```

---

## Laboratorio 4

Detenerlo.

```bash
podman stop web
```

---

## Laboratorio 5

Reiniciarlo.

```bash
podman restart web
```

---

## Laboratorio 6

Pausarlo.

```bash
podman pause web
```

---

## Laboratorio 7

Reanudarlo.

```bash
podman unpause web
```

---

## Laboratorio 8

Consultar información.

```bash
podman inspect web
```

---

## Laboratorio 9

Consultar únicamente el estado.

```bash
podman inspect \
--format '{{.State.Status}}' \
web
```

---

## Laboratorio 10

Consultar IP.

```bash
podman inspect \
--format '{{.NetworkSettings.IPAddress}}' \
web
```

---

## Laboratorio 11

Consultar logs.

```bash
podman logs web
```

---

## Laboratorio 12

Seguir logs.

```bash
podman logs -f web
```

---

## Laboratorio 13

Consultar procesos.

```bash
podman top web
```

---

## Laboratorio 14

Consultar estadísticas.

```bash
podman stats web
```

---

## Laboratorio 15

Consultar cambios.

```bash
podman diff web
```

---

## Laboratorio 16

Consultar puertos.

```bash
podman port web
```

---

## Laboratorio 17

Renombrar.

```bash
podman rename web nginx-prod
```

---

## Laboratorio 18

Eliminar.

```bash
podman rm nginx-prod
```

---

# Escenario RHCSA 1

El servicio web no responde.

Procedimiento recomendado:

```bash
podman ps

↓

podman logs web

↓

podman inspect web

↓

podman top web

↓

podman stats web
```

---

# Escenario RHCSA 2

Un contenedor consume demasiada CPU.

Diagnóstico:

```bash
podman stats
```

Luego:

```bash
podman top
```

Finalmente:

```bash
podman inspect
```

---

# Escenario RHCSA 3

El contenedor finaliza inmediatamente después de iniciarse.

Consultar:

```bash
podman logs
```

Luego:

```bash
podman inspect
```

Revisar el proceso principal (PID 1) y el comando ejecutado por el contenedor.

---

# Buenas prácticas

- Asignar nombres descriptivos a todos los contenedores.
- Revisar los logs antes de reiniciar un servicio.
- Utilizar `podman inspect` para obtener información detallada en lugar de hacer suposiciones.
- Supervisar periódicamente el consumo de CPU y memoria mediante `podman stats`.
- Eliminar únicamente contenedores que ya no sean necesarios.
- Utilizar `podman stop` antes de recurrir a `podman kill`, salvo en situaciones de emergencia.

---

# Errores comunes

## Error 1

Eliminar un contenedor que aún se encuentra en ejecución.

---

## Error 2

Utilizar `kill` cuando un `stop` sería suficiente.

---

## Error 3

No revisar los logs antes de reiniciar un contenedor.

---

## Error 4

Confundir el estado **Paused** con **Stopped**.

---

## Error 5

Modificar un contenedor sin inspeccionar previamente su configuración.

---

# Resumen

En esta segunda fase aprendimos a:

- Listar y filtrar contenedores con `podman ps`.
- Iniciar, detener y reiniciar contenedores.
- Pausar y reanudar procesos.
- Finalizar contenedores mediante `podman kill`.
- Renombrar y eliminar contenedores.
- Inspeccionar información detallada con `podman inspect`.
- Consultar registros mediante `podman logs`.
- Visualizar procesos internos con `podman top`.
- Monitorear el consumo de recursos con `podman stats`.
- Detectar cambios en el sistema de archivos mediante `podman diff`.
- Consultar puertos publicados y sincronizar scripts con `podman wait`.

En la **Fase 3** aprenderemos a trabajar dentro del contenedor utilizando `podman exec`, `attach`, `cp`, administración de archivos, manejo de señales, límites de CPU y memoria, códigos de salida y técnicas avanzadas utilizadas en entornos de producción.


---


# 71. Crear y Administrar Contenedores (Fase 3)

> **Módulo 10 – Contenedores con Podman**
>
> **Manual RHCSA**
>
> **Archivo:** `71-crear-administrar-contenedores.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase podrás:

- Ejecutar comandos dentro de un contenedor.
- Comprender la diferencia entre `exec` y `attach`.
- Copiar archivos entre el host y un contenedor.
- Analizar el estado interno de un contenedor.
- Comprender los códigos de salida (Exit Codes).
- Administrar señales de Linux.
- Limitar CPU y memoria.
- Comprender cómo afectan los recursos al rendimiento.
- Resolver problemas frecuentes encontrados en producción.

---

# Introducción

Crear un contenedor y administrarlo es solo una parte del trabajo.

En producción es muy común que debamos:

- ingresar al contenedor
- ejecutar comandos
- copiar archivos
- revisar configuraciones
- obtener logs
- analizar procesos
- limitar recursos
- solucionar problemas

Todo ello sin reconstruir la imagen.

---

# Arquitectura

```text
               Host Fedora

                   │

                   ▼

             Podman Engine

                   │

                   ▼

            Contenedor Running

         ┌─────────┼─────────┐

         ▼         ▼         ▼

      Exec      Attach      Copy
```

---

# Ejecutar comandos

El comando más utilizado es:

```bash
podman exec
```

Permite ejecutar procesos dentro de un contenedor que ya está en ejecución.

---

# Sintaxis

```bash
podman exec [opciones] CONTENEDOR COMANDO
```

---

# Ejemplo

```bash
podman exec web hostname
```

Resultado:

```text
servidorweb
```

---

# Ejecutar Date

```bash
podman exec web date
```

---

# Consultar Usuario

```bash
podman exec web whoami
```

---

# Consultar Directorio

```bash
podman exec web pwd
```

---

# Ejecutar Bash

```bash
podman exec -it web bash
```

---

# Algunas imágenes

No incluyen Bash.

En Alpine:

```bash
podman exec -it alpine sh
```

---

# ¿Qué ocurre internamente?

```text
Contenedor Running

        │

        ▼

Nuevo proceso

        │

        ▼

Finaliza

        │

        ▼

Contenedor sigue funcionando
```

---

# Ejecutar múltiples comandos

```bash
podman exec web sh -c \
"hostname && date && id"
```

---

# Ejecutar como Root

```bash
podman exec \
-u root \
web bash
```

---

# Ejecutar como otro usuario

```bash
podman exec \
-u nginx \
web whoami
```

---

# Variables temporales

```bash
podman exec \
-e DEBUG=true \
web env
```

---

# Diferencia entre Exec y Attach

Es una de las preguntas más frecuentes.

---

## Exec

```text
Contenedor

↓

Nuevo proceso

↓

Finaliza
```

---

## Attach

```text
Nos conectamos

al proceso principal

(PID 1)
```

---

# Resumen

| Exec | Attach |
|------|---------|
| Ejecuta un proceso nuevo | Se conecta al proceso principal |
| Muy seguro | Puede afectar la aplicación |
| Más utilizado | Uso limitado |

---

# Attach

Sintaxis

```bash
podman attach web
```

---

# ¿Qué sucede?

Nos conectamos directamente al proceso principal del contenedor.

---

# Riesgo

Si presionamos:

```
CTRL+C
```

podríamos detener el proceso principal.

Por esta razón, en producción normalmente se utiliza:

```bash
podman exec
```

---

# Desconectarse sin detener el contenedor

Secuencia:

```text
CTRL + P

CTRL + Q
```

El contenedor continúa ejecutándose.

---

# Copiar archivos

Podman permite copiar archivos entre el host y el contenedor.

Comando:

```bash
podman cp
```

---

# Host → Contenedor

```bash
podman cp \
index.html \
web:/usr/share/nginx/html/
```

---

# Contenedor → Host

```bash
podman cp \
web:/etc/nginx/nginx.conf .
```

---

# Copiar directorios

```bash
podman cp \
configuracion/ \
web:/tmp/
```

---

# Copiar múltiples archivos

Puede utilizarse:

```bash
tar
```

o

```bash
rsync
```

junto con `podman cp` para grandes volúmenes de información.

---

# Verificar copia

```bash
podman exec web ls -l /tmp
```

---

# Estados del contenedor

Consultar

```bash
podman inspect web \
--format '{{.State.Status}}'
```

---

Estados

```text
Created
```

```text
Running
```

```text
Paused
```

```text
Exited
```

---

# Código de salida

Consultar:

```bash
podman inspect \
--format '{{.State.ExitCode}}' \
web
```

---

# Exit Code 0

```text
Éxito
```

---

# Exit Code diferente de cero

Generalmente indica:

- error
- excepción
- fallo
- interrupción

---

# Algunos códigos comunes

| Código | Significado |
|---------|-------------|
| 0 | Ejecución correcta |
| 1 | Error general |
| 2 | Error de uso del comando |
| 125 | Error de Podman |
| 126 | No ejecutable |
| 127 | Comando no encontrado |
| 137 | Finalizado por SIGKILL |
| 143 | Finalizado por SIGTERM |

---

# Señales Linux

Los contenedores utilizan señales POSIX.

Las más comunes son:

| Señal | Descripción |
|--------|-------------|
| SIGTERM | Finalización ordenada |
| SIGKILL | Finalización inmediata |
| SIGINT | Interrupción |
| SIGHUP | Recargar configuración |
| SIGQUIT | Salida con diagnóstico |

---

# Enviar una señal

```bash
podman kill \
--signal SIGTERM \
web
```

---

# Recargar configuración

Muchas aplicaciones soportan:

```bash
podman kill \
--signal SIGHUP \
web
```

Sin reiniciar completamente el servicio.

---

# Recursos

Un contenedor comparte los recursos del Host.

Podemos limitar:

- CPU
- Memoria
- Procesos
- I/O

---

# Limitar CPU

```bash
podman run \
--cpus=1 \
nginx
```

---

# Medio núcleo

```bash
podman run \
--cpus=0.5 \
nginx
```

---

# Dos núcleos

```bash
podman run \
--cpus=2 \
nginx
```

---

# Afinidad CPU

Ejecutar únicamente sobre CPUs específicas.

```bash
podman run \
--cpuset-cpus="0,1" \
nginx
```

---

# Limitar Memoria

```bash
podman run \
--memory=512m \
nginx
```

---

# 2 GB

```bash
podman run \
--memory=2g \
postgres
```

---

# Swap

```bash
podman run \
--memory=1g \
--memory-swap=2g \
nginx
```

---

# Limitar procesos

```bash
podman run \
--pids-limit=100 \
nginx
```

---

# OOM Killer

Si un contenedor supera el límite de memoria:

```text
OOM Killer
```

podría finalizar el proceso automáticamente.

---

# Monitorear consumo

```bash
podman stats
```

---

Consultar un contenedor

```bash
podman stats postgres
```

---

# Escenario

Servidor:

16 GB RAM

Tenemos:

```text
Web

2 GB
```

```text
PostgreSQL

8 GB
```

```text
Redis

512 MB
```

```text
Grafana

512 MB
```

Todavía queda memoria disponible para el sistema operativo y otros servicios.

---

# Caso práctico

Servidor de producción:

```text
CPU

16 cores
```

```text
RAM

64 GB
```

Configuración:

```text
PostgreSQL

16 GB
```

```text
Nginx

1 CPU
```

```text
Redis

2 GB
```

Esta distribución evita que un único contenedor consuma todos los recursos.

---

# Laboratorio RHCSA

## Laboratorio 1

Entrar al contenedor.

```bash
podman exec -it web bash
```

---

## Laboratorio 2

Consultar hostname.

```bash
hostname
```

---

## Laboratorio 3

Salir.

```bash
exit
```

---

## Laboratorio 4

Ejecutar Date.

```bash
podman exec web date
```

---

## Laboratorio 5

Consultar usuario.

```bash
podman exec web whoami
```

---

## Laboratorio 6

Copiar un archivo.

```bash
podman cp prueba.txt web:/tmp/
```

---

## Laboratorio 7

Verificar.

```bash
podman exec web ls /tmp
```

---

## Laboratorio 8

Copiar nuevamente al Host.

```bash
podman cp \
web:/tmp/prueba.txt .
```

---

## Laboratorio 9

Consultar Exit Code.

```bash
podman inspect \
--format '{{.State.ExitCode}}' \
web
```

---

## Laboratorio 10

Consultar estado.

```bash
podman inspect \
--format '{{.State.Status}}' \
web
```

---

## Laboratorio 11

Limitar memoria.

```bash
podman run \
--memory=512m \
nginx
```

---

## Laboratorio 12

Limitar CPU.

```bash
podman run \
--cpus=1 \
nginx
```

---

## Laboratorio 13

Monitorear.

```bash
podman stats
```

---

## Laboratorio 14

Enviar SIGTERM.

```bash
podman kill \
--signal SIGTERM \
web
```

---

## Laboratorio 15

Enviar SIGHUP.

```bash
podman kill \
--signal SIGHUP \
web
```

---

# Escenario RHCSA 1

Un contenedor responde lentamente.

Procedimiento recomendado:

```text
podman stats

↓

podman top

↓

podman inspect

↓

Revisar límites de CPU y memoria
```

---

# Escenario RHCSA 2

Necesitas modificar un archivo de configuración sin reconstruir la imagen.

Procedimiento:

```text
podman cp

↓

podman exec

↓

Editar archivo

↓

Reiniciar el servicio si es necesario
```

---

# Escenario RHCSA 3

El contenedor termina inmediatamente después de iniciarse.

Pasos recomendados:

```text
podman logs

↓

podman inspect

↓

Revisar Exit Code

↓

Verificar el proceso principal (PID 1)
```

---

# Buenas prácticas

- Preferir `podman exec` sobre `podman attach` para tareas administrativas.
- Limitar CPU y memoria de los contenedores críticos.
- Utilizar `podman cp` para cambios puntuales y mantener la configuración persistente mediante volúmenes cuando corresponda.
- Supervisar periódicamente el consumo de recursos con `podman stats`.
- Analizar siempre los códigos de salida antes de reiniciar un contenedor.

---

# Errores comunes

## Error 1

Utilizar `attach` cuando bastaba con `exec`.

---

## Error 2

Ejecutar todos los contenedores sin límites de recursos.

---

## Error 3

Ignorar los códigos de salida de los procesos.

---

## Error 4

Copiar archivos manualmente dentro del contenedor sin documentar los cambios.

---

## Error 5

Olvidar que los cambios realizados directamente dentro del contenedor pueden perderse al recrearlo si no se utilizan volúmenes.

---

# Resumen

En esta tercera fase aprendimos a:

- Ejecutar comandos dentro de un contenedor con `podman exec`.
- Comprender las diferencias entre `exec` y `attach`.
- Copiar archivos entre el host y el contenedor mediante `podman cp`.
- Consultar el estado y los códigos de salida de un contenedor.
- Comprender y utilizar señales POSIX como `SIGTERM`, `SIGKILL` y `SIGHUP`.
- Limitar el consumo de CPU, memoria y procesos.
- Monitorear recursos y diagnosticar problemas relacionados con el rendimiento.

En la **Fase 4** integraremos todos estos conocimientos mediante un laboratorio completo de administración de contenedores, resolución de problemas reales, un script de auditoría, un checklist RHCSA, preguntas de repaso y un desafío final con escenarios similares a los del examen oficial.

----

# 71. Crear y Administrar Contenedores (Fase 4)

> **Módulo 10 – Contenedores con Podman**
>
> **Manual RHCSA**
>
> **Archivo:** `71-crear-administrar-contenedores.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase podrás:

- Diagnosticar problemas comunes relacionados con contenedores.
- Resolver errores de ejecución.
- Analizar fallos mediante logs e inspección.
- Auditar el estado de los contenedores.
- Automatizar verificaciones mediante scripts.
- Aplicar procedimientos utilizados en ambientes empresariales.
- Resolver escenarios similares al examen RHCSA.

---

# Metodología de Diagnóstico

Cuando un contenedor presenta problemas, evita reiniciarlo inmediatamente.

Sigue siempre un proceso estructurado.

```text
                 Problema

                     │

                     ▼

         ¿Existe el contenedor?

                     │

                     ▼

        ¿Está ejecutándose?

                     │

                     ▼

         Revisar los Logs

                     │

                     ▼

        Inspeccionar configuración

                     │

                     ▼

         Revisar procesos

                     │

                     ▼

        Revisar recursos

                     │

                     ▼

      Aplicar la solución adecuada
```

---

# Comandos esenciales de diagnóstico

```bash
podman ps -a
```

```bash
podman inspect
```

```bash
podman logs
```

```bash
podman top
```

```bash
podman stats
```

```bash
podman diff
```

```bash
podman port
```

```bash
podman exec
```

```bash
podman cp
```

---

# Escenario 1
## El contenedor no inicia

Intentamos:

```bash
podman start web
```

Resultado:

```text
Exited
```

Diagnóstico:

```bash
podman logs web
```

Luego:

```bash
podman inspect web
```

Revisar:

- comando ejecutado
- variables
- imagen utilizada
- ExitCode

---

# Escenario 2
## El contenedor termina inmediatamente

Consultar:

```bash
podman logs web
```

Posteriormente:

```bash
podman inspect \
--format '{{.State.ExitCode}}' \
web
```

Códigos frecuentes:

| Código | Significado |
|----------|-------------|
| 0 | Finalizó correctamente |
| 1 | Error general |
| 126 | No ejecutable |
| 127 | Comando inexistente |
| 137 | Finalizado por SIGKILL |
| 143 | Finalizado por SIGTERM |

---

# Escenario 3
## El sitio web no responde

Verificar:

```bash
podman ps
```

Consultar puertos:

```bash
podman port web
```

Verificar publicación:

```text
Host

8080

↓

Container

80
```

También revisar el firewall del host.

---

# Escenario 4
## El contenedor consume demasiada memoria

Consultar:

```bash
podman stats
```

Luego:

```bash
podman inspect web
```

Revisar:

```text
Memory

CPUs

PIDs
```

---

# Escenario 5
## Alto consumo de CPU

Consultar:

```bash
podman stats
```

Luego:

```bash
podman top web
```

Determinar qué proceso consume CPU.

---

# Escenario 6
## No puedo ingresar al contenedor

Intentamos:

```bash
podman exec -it web bash
```

Error:

```text
bash not found
```

Muchas imágenes mínimas utilizan:

```bash
sh
```

Ejemplo:

```bash
podman exec -it alpine sh
```

---

# Escenario 7
## No encuentro un archivo

Ingresar:

```bash
podman exec -it web sh
```

Buscar:

```bash
find / -name archivo.conf
```

---

# Escenario 8
## Se modificó una configuración

Consultar:

```bash
podman diff web
```

Resultado:

```text
A

/etc/test.conf
```

```text
C

/etc/nginx.conf
```

```text
D

/tmp/file
```

---

# Escenario 9
## Error de permisos

Consultar usuario:

```bash
podman exec web id
```

Consultar propietario:

```bash
podman exec web ls -l
```

---

# Escenario 10
## Variables incorrectas

Consultar:

```bash
podman inspect web
```

O directamente:

```bash
podman exec web env
```

---

# Escenario 11
## Reinicios continuos

Consultar:

```bash
podman inspect web
```

Revisar:

```text
Restart Policy
```

Luego:

```bash
podman logs web
```

---

# Escenario 12
## Contenedor detenido

Consultar:

```bash
podman ps -a
```

Estado:

```text
Exited
```

Iniciar nuevamente:

```bash
podman start web
```

---

# Procedimiento recomendado

```text
1.

podman ps -a

↓

2.

podman logs

↓

3.

podman inspect

↓

4.

podman top

↓

5.

podman stats

↓

6.

Resolver
```

---

# Script de Auditoría

Guardar como:

```text
audit_containers.sh
```

```bash
#!/bin/bash

echo "=================================="
echo " AUDITORÍA DE CONTENEDORES PODMAN "
echo "=================================="

echo
echo "VERSIÓN"
podman --version

echo
echo "CONTENEDORES"
podman ps -a

echo
echo "ESTADÍSTICAS"
podman stats --no-stream

echo
echo "ALMACENAMIENTO"
podman system df

echo
echo "REDES"
podman network ls

echo
echo "IMÁGENES"
podman images

echo
echo "GRAPHROOT"
podman info --format '{{.Store.GraphRoot}}'

echo
echo "RUNTIME"
podman info --format '{{.Host.OCIRuntime.Name}}'
```

Permisos:

```bash
chmod +x audit_containers.sh
```

---

# Script para revisar todos los contenedores

```bash
#!/bin/bash

for c in $(podman ps -aq)
do
    echo
    echo "=========================="
    echo "$c"
    echo "=========================="

    podman inspect \
    --format 'Nombre: {{.Name}} Estado: {{.State.Status}} ExitCode: {{.State.ExitCode}}' \
    "$c"
done
```

---

# Script para mostrar los logs de todos los contenedores

```bash
#!/bin/bash

for c in $(podman ps -aq)
do
    echo
    echo "===================="
    echo "$c"
    echo "===================="

    podman logs --tail 20 "$c"
done
```

---

# Laboratorio RHCSA

## Laboratorio 1

Crear un contenedor.

```bash
podman run -d \
--name web \
nginx
```

---

## Laboratorio 2

Consultar estado.

```bash
podman ps
```

---

## Laboratorio 3

Consultar información.

```bash
podman inspect web
```

---

## Laboratorio 4

Consultar IP.

```bash
podman inspect \
--format '{{.NetworkSettings.IPAddress}}' \
web
```

---

## Laboratorio 5

Consultar logs.

```bash
podman logs web
```

---

## Laboratorio 6

Seguir logs.

```bash
podman logs -f web
```

---

## Laboratorio 7

Entrar al contenedor.

```bash
podman exec -it web bash
```

---

## Laboratorio 8

Consultar procesos.

```bash
podman top web
```

---

## Laboratorio 9

Consultar recursos.

```bash
podman stats web
```

---

## Laboratorio 10

Crear un archivo.

```bash
podman exec web touch /tmp/prueba
```

---

## Laboratorio 11

Consultar cambios.

```bash
podman diff web
```

---

## Laboratorio 12

Consultar puertos.

```bash
podman port web
```

---

## Laboratorio 13

Detener el contenedor.

```bash
podman stop web
```

---

## Laboratorio 14

Consultar estado.

```bash
podman ps -a
```

---

## Laboratorio 15

Reiniciarlo.

```bash
podman start web
```

---

## Laboratorio 16

Consultar ExitCode.

```bash
podman inspect \
--format '{{.State.ExitCode}}' \
web
```

---

## Laboratorio 17

Ejecutar auditoría.

```bash
./audit_containers.sh
```

---

## Laboratorio 18

Ejecutar revisión masiva.

```bash
./revision_contenedores.sh
```

---

## Laboratorio 19

Ejecutar revisión de logs.

```bash
./logs_contenedores.sh
```

---

## Laboratorio 20

Eliminar el contenedor.

```bash
podman stop web

podman rm web
```

---

# Checklist RHCSA

```text
□ Podman instalado.

□ Imagen descargada correctamente.

□ Contenedor creado.

□ Nombre asignado.

□ Variables de entorno verificadas.

□ Hostname configurado.

□ Puertos publicados correctamente.

□ Estado consultado.

□ Logs revisados.

□ Información inspeccionada.

□ Procesos consultados.

□ Recursos monitoreados.

□ Cambios verificados mediante diff.

□ ExitCode revisado.

□ Scripts ejecutados correctamente.

□ Contenedor eliminado correctamente.
```

---

# Preguntas de Repaso

1. ¿Cuál es la diferencia entre `podman create` y `podman run`?
2. ¿Qué hace `podman exec`?
3. ¿En qué se diferencia `exec` de `attach`?
4. ¿Cómo copiar un archivo del host al contenedor?
5. ¿Cómo copiar un archivo del contenedor al host?
6. ¿Qué comando muestra los logs?
7. ¿Cómo seguir los logs en tiempo real?
8. ¿Qué hace `podman inspect`?
9. ¿Qué comando muestra los procesos internos?
10. ¿Qué comando muestra el consumo de CPU y memoria?
11. ¿Qué hace `podman diff`?
12. ¿Qué hace `podman port`?
13. ¿Qué representa un Exit Code 137?
14. ¿Qué diferencia existe entre `stop` y `kill`?
15. ¿Qué señal utiliza normalmente `podman stop`?
16. ¿Qué hace `podman pause`?
17. ¿Qué hace `podman unpause`?
18. ¿Qué hace `podman wait`?
19. ¿Cuál es el comando más recomendable para ingresar a un contenedor en producción?
20. ¿Cuál es el procedimiento recomendado cuando un contenedor presenta problemas?

---

# Respuestas

1. `create` solo crea el contenedor; `run` lo crea y lo inicia.
2. Ejecuta un proceso adicional dentro de un contenedor en ejecución.
3. `exec` crea un nuevo proceso; `attach` conecta la terminal al proceso principal del contenedor.
4. `podman cp archivo contenedor:/ruta`
5. `podman cp contenedor:/ruta/archivo .`
6. `podman logs`
7. `podman logs -f`
8. Muestra toda la configuración y el estado interno del contenedor.
9. `podman top`
10. `podman stats`
11. Muestra los cambios realizados sobre el sistema de archivos del contenedor.
12. Muestra la correspondencia entre los puertos del host y del contenedor.
13. El proceso fue finalizado por una señal **SIGKILL**.
14. `stop` intenta una finalización ordenada; `kill` finaliza inmediatamente.
15. **SIGTERM**
16. Suspende temporalmente la ejecución de los procesos del contenedor.
17. Reanuda la ejecución de un contenedor previamente pausado.
18. Espera hasta que un contenedor termine y devuelve su código de salida.
19. `podman exec`
20. Revisar el estado, analizar los logs, inspeccionar la configuración, verificar procesos y recursos, y aplicar la corrección adecuada.

---

# Desafío Final RHCSA

Se entrega un servidor Fedora con Podman instalado.

Debes realizar las siguientes tareas sin consultar documentación externa:

1. Descargar una imagen oficial de Nginx.
2. Crear un contenedor llamado `web`.
3. Publicar el puerto **8080** del host hacia el puerto **80** del contenedor.
4. Configurar una política de reinicio `unless-stopped`.
5. Verificar que el contenedor se encuentre en estado **Running**.
6. Consultar la IP asignada al contenedor.
7. Ingresar al contenedor y crear un archivo `/tmp/rhcsa.txt`.
8. Copiar dicho archivo desde el contenedor hacia el host.
9. Consultar los procesos internos del contenedor.
10. Mostrar el consumo actual de CPU y memoria.
11. Revisar los últimos 20 registros del contenedor.
12. Identificar cualquier cambio realizado en el sistema de archivos del contenedor.
13. Detener el contenedor de forma ordenada.
14. Reiniciarlo y comprobar que vuelve a estar disponible.
15. Ejecutar el script `audit_containers.sh`.
16. Eliminar el contenedor sin dejar recursos innecesarios.

---

# Buenas prácticas

- Utilizar siempre `podman exec` para tareas administrativas en lugar de `attach`.
- Asignar nombres descriptivos y consistentes a los contenedores.
- Supervisar periódicamente el consumo de CPU, memoria y almacenamiento.
- Analizar los logs antes de reiniciar un servicio.
- Aplicar límites de recursos a aplicaciones críticas.
- Automatizar auditorías mediante scripts para detectar problemas antes de que afecten la producción.
- Documentar cualquier cambio manual realizado dentro de un contenedor.

---

# Resumen del Capítulo 71

En este capítulo aprendimos a:

- Crear contenedores a partir de imágenes OCI.
- Comprender el ciclo de vida completo de un contenedor.
- Ejecutar contenedores en modo interactivo y en segundo plano.
- Administrar su estado mediante `start`, `stop`, `restart`, `pause`, `unpause`, `kill` y `rm`.
- Inspeccionar su configuración y estado interno con `podman inspect`.
- Analizar registros mediante `podman logs`.
- Monitorear procesos y consumo de recursos con `podman top` y `podman stats`.
- Ejecutar comandos dentro del contenedor mediante `podman exec`.
- Copiar archivos entre el host y el contenedor utilizando `podman cp`.
- Comprender los códigos de salida y las señales POSIX.
- Diagnosticar problemas frecuentes mediante un procedimiento estructurado.
- Automatizar auditorías y verificaciones utilizando scripts Bash.

---

# Fin del capítulo
