# 75. Systemd y Contenedores con Podman (Fase 1)

> **Módulo 10 – Contenedores con Podman**
>
> **Manual RHCSA**
>
> **Archivo:** `75-systemd-contenedores-podman.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase podrás:

- Comprender la integración entre Podman y systemd.
- Entender por qué systemd es el método recomendado para administrar contenedores.
- Conocer la diferencia entre administrar un contenedor manualmente y mediante systemd.
- Comprender la diferencia entre servicios Rootless y Rootful.
- Aprender la estructura de una unidad (.service).
- Generar unidades automáticamente con Podman.
- Prepararte para administrar contenedores persistentes como se realiza en RHEL y Fedora.

---

# Introducción

Hasta este momento hemos ejecutado contenedores utilizando comandos como:

```bash
podman run
```

o

```bash
podman start
```

Esto funciona perfectamente para laboratorios.

Sin embargo, en producción aparecen nuevas necesidades.

Por ejemplo:

- iniciar automáticamente los contenedores al arrancar el servidor
- reiniciar servicios después de un fallo
- controlar el orden de inicio
- registrar eventos
- administrar dependencias
- monitorear el estado del servicio

Todo esto lo resuelve:

```text
systemd
```

---

# ¿Qué es systemd?

systemd es el sistema de inicialización y administración de servicios utilizado por Red Hat Enterprise Linux, Fedora y la mayoría de distribuciones Linux modernas.

Es el proceso:

```text
PID 1
```

del sistema operativo.

---

# Arquitectura

```text
          Encendido

              │

              ▼

            Kernel

              │

              ▼

          systemd (PID 1)

              │

      ┌───────┼────────┐

      ▼       ▼        ▼

 SSH     PostgreSQL   Podman

                        │

                        ▼

                  Contenedores
```

---

# ¿Por qué utilizar systemd?

Porque proporciona:

- Inicio automático.
- Reinicio automático.
- Registro mediante journal.
- Dependencias.
- Monitoreo.
- Control centralizado.
- Integración con el sistema operativo.

---

# Comparación

## Inicio manual

```text
Servidor

↓

Administrador

↓

podman run

↓

Contenedor
```

---

## Inicio mediante systemd

```text
Servidor

↓

systemd

↓

Servicio

↓

Contenedor
```

---

# Beneficios

| Característica | Manual | systemd |
|---------------|---------|----------|
| Inicio automático | No | Sí |
| Reinicio automático | No | Sí |
| Registro | Limitado | Completo |
| Dependencias | No | Sí |
| Estado del servicio | No | Sí |
| Integración con Linux | Baja | Excelente |

---

# Casos de uso

- PostgreSQL
- Nginx
- Apache
- Redis
- MongoDB
- API REST
- Microservicios
- Aplicaciones Java
- Aplicaciones Python

---

# Arquitectura Empresarial

```text
                 Servidor

                     │

                     ▼

                  systemd

         ┌───────────┼───────────┐

         ▼           ▼           ▼

 PostgreSQL      Redis       Nginx

         │           │           │

         ▼           ▼           ▼

      Podman      Podman      Podman
```

---

# ¿Cómo funciona?

Cuando el servidor inicia:

```text
BIOS / UEFI

↓

Kernel

↓

systemd

↓

Servicios

↓

Contenedores
```

No es necesario ejecutar nuevamente:

```bash
podman run
```

---

# Componentes

Una solución basada en Podman + systemd normalmente contiene:

- Imagen
- Contenedor
- Archivo .service
- Journal
- Dependencias

---

# ¿Qué es una Unit?

systemd administra recursos llamados:

```text
Units
```

Existen distintos tipos.

---

# Tipos principales

| Tipo | Extensión |
|--------|-----------|
| Servicio | .service |
| Timer | .timer |
| Socket | .socket |
| Target | .target |
| Mount | .mount |
| Path | .path |
| Device | .device |

---

# Tipo utilizado por Podman

Principalmente:

```text
.service
```

---

# Estructura de una Unit

Una unidad posee tres grandes secciones.

```text
[Unit]

↓

[Service]

↓

[Install]
```

---

# Sección Unit

Describe el servicio.

Ejemplo

```ini
[Unit]

Description=Servidor Web

After=network-online.target
```

---

# Explicación

```text
Description

↓

Descripción del servicio


After

↓

Esperar otro servicio
```

---

# Sección Service

Define cómo ejecutar el servicio.

Ejemplo

```ini
[Service]

ExecStart=

ExecStop=

Restart=
```

---

# Sección Install

Controla cuándo será habilitado.

Ejemplo

```ini
[Install]

WantedBy=multi-user.target
```

---

# Arquitectura

```text
Unit

│

├── Unit

├── Service

└── Install
```

---

# Directorios

## Rootful

```text
/etc/systemd/system
```

---

## Usuario

```text
~/.config/systemd/user
```

---

# Consultar servicios

Sistema

```bash
systemctl list-units
```

---

Usuario

```bash
systemctl --user list-units
```

---

# Estado de un servicio

Sistema

```bash
systemctl status nginx
```

---

Usuario

```bash
systemctl --user status
```

---

# Habilitar un servicio

Sistema

```bash
sudo systemctl enable servicio
```

---

Usuario

```bash
systemctl --user enable servicio
```

---

# Iniciar

Sistema

```bash
sudo systemctl start servicio
```

---

Usuario

```bash
systemctl --user start servicio
```

---

# Detener

Sistema

```bash
sudo systemctl stop servicio
```

---

Usuario

```bash
systemctl --user stop servicio
```

---

# Reiniciar

```bash
systemctl restart servicio
```

---

Usuario

```bash
systemctl --user restart servicio
```

---

# Consultar Journal

Sistema

```bash
journalctl -u nginx
```

---

Usuario

```bash
journalctl --user -u servicio
```

---

# Consultar últimas líneas

```bash
journalctl -u nginx -n 50
```

---

# Seguir en tiempo real

```bash
journalctl -fu nginx
```

---

# Integración con Podman

Podman puede generar automáticamente un archivo:

```text
.service
```

No es necesario escribirlo manualmente.

---

# Comando principal

```bash
podman generate systemd
```

---

# Ejemplo

Crear un contenedor.

```bash
podman run \
-d \
--name web \
nginx
```

---

Generar unidad.

```bash
podman generate systemd \
--name web
```

---

Resultado

Se mostrará el contenido completo del archivo.

---

# Crear el archivo automáticamente

```bash
podman generate systemd \
--files \
--name web
```

---

Resultado

```text
container-web.service
```

---

# Consultar

```bash
ls
```

Resultado

```text
container-web.service
```

---

# Arquitectura

```text
Container

↓

Generate Systemd

↓

.service

↓

systemd
```

---

# Copiar al sistema

Rootful

```bash
sudo cp \
container-web.service \
/etc/systemd/system/
```

---

Rootless

```bash
mkdir -p \
~/.config/systemd/user
```

```bash
cp \
container-web.service \
~/.config/systemd/user/
```

---

# Recargar systemd

Sistema

```bash
sudo systemctl daemon-reload
```

---

Usuario

```bash
systemctl --user daemon-reload
```

---

# Habilitar

Sistema

```bash
sudo systemctl enable \
container-web.service
```

---

Usuario

```bash
systemctl --user enable \
container-web.service
```

---

# Iniciar

Sistema

```bash
sudo systemctl start \
container-web.service
```

---

Usuario

```bash
systemctl --user start \
container-web.service
```

---

# Verificar

```bash
systemctl status \
container-web.service
```

---

Usuario

```bash
systemctl --user status \
container-web.service
```

---

# Flujo completo

```text
podman run

      │

      ▼

Contenedor

      │

      ▼

Generate systemd

      │

      ▼

.service

      │

      ▼

systemctl enable

      │

      ▼

Inicio Automático
```

---

# Laboratorio RHCSA

## Laboratorio 1

Consultar systemd.

```bash
ps -p 1
```

---

## Laboratorio 2

Consultar servicios.

```bash
systemctl list-units
```

---

## Laboratorio 3

Consultar servicios del usuario.

```bash
systemctl --user list-units
```

---

## Laboratorio 4

Crear un contenedor.

```bash
podman run \
-d \
--name web \
nginx
```

---

## Laboratorio 5

Generar la unidad.

```bash
podman generate systemd \
--name web
```

---

## Laboratorio 6

Crear el archivo.

```bash
podman generate systemd \
--files \
--name web
```

---

## Laboratorio 7

Consultar el archivo.

```bash
cat container-web.service
```

---

## Laboratorio 8

Crear el directorio.

```bash
mkdir -p \
~/.config/systemd/user
```

---

## Laboratorio 9

Copiar el archivo.

```bash
cp container-web.service \
~/.config/systemd/user/
```

---

## Laboratorio 10

Recargar systemd.

```bash
systemctl --user daemon-reload
```

---

## Laboratorio 11

Habilitar.

```bash
systemctl --user enable \
container-web.service
```

---

## Laboratorio 12

Iniciar.

```bash
systemctl --user start \
container-web.service
```

---

## Laboratorio 13

Consultar estado.

```bash
systemctl --user status \
container-web.service
```

---

## Laboratorio 14

Consultar journal.

```bash
journalctl --user \
-u container-web.service
```

---

## Laboratorio 15

Reiniciar el servicio.

```bash
systemctl --user restart \
container-web.service
```

---

# Buenas prácticas

- Administrar contenedores persistentes mediante systemd.
- Utilizar nombres descriptivos para los servicios.
- Mantener separados los servicios Rootless y Rootful.
- Documentar las dependencias entre servicios.
- Supervisar periódicamente los registros con `journalctl`.

---

# Errores comunes

## Error 1

Ejecutar `podman run` manualmente después de haber configurado un servicio systemd.

---

## Error 2

Olvidar ejecutar:

```bash
systemctl daemon-reload
```

después de copiar una nueva unidad.

---

## Error 3

Confundir:

```bash
systemctl
```

con

```bash
systemctl --user
```

---

## Error 4

Copiar un archivo `.service` en un directorio incorrecto.

---

## Error 5

No habilitar el servicio mediante:

```bash
enable
```

esperando que inicie automáticamente tras reiniciar el sistema.

---

# Resumen

En esta primera fase aprendimos:

- Qué es systemd.
- Cómo se integra con Podman.
- La estructura de una unidad `.service`.
- La diferencia entre administrar un contenedor manualmente y mediante systemd.
- Cómo generar automáticamente unidades con `podman generate systemd`.
- Cómo instalar, habilitar e iniciar servicios Rootless y Rootful.

En la **Fase 2** aprenderemos a analizar en profundidad los archivos `.service`, comprender todas sus directivas importantes (`ExecStart`, `ExecStop`, `Restart`, `TimeoutStopSec`, `KillMode`, `Environment`, `Requires`, `After`, `WantedBy`), personalizar unidades, administrar Pods completos con systemd y realizar laboratorios avanzados orientados al examen RHCSA.

----

# 75. Systemd y Contenedores con Podman (Fase 2)

> **Módulo 10 – Contenedores con Podman**
>
> **Manual RHCSA**
>
> **Archivo:** `75-systemd-contenedores-podman.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase podrás:

- Comprender completamente un archivo `.service`.
- Personalizar servicios generados por Podman.
- Configurar políticas de reinicio.
- Administrar dependencias entre servicios.
- Comprender las directivas más importantes de systemd.
- Crear servicios para Pods completos.
- Integrar variables de entorno.
- Aplicar configuraciones utilizadas en servidores empresariales.

---

# Introducción

En la fase anterior aprendimos que Podman puede generar automáticamente un archivo `.service`.

Sin embargo, un administrador Linux no debe limitarse a ejecutar:

```bash
podman generate systemd
```

También debe comprender completamente el archivo generado.

En ambientes empresariales es común modificar estas unidades para adaptarlas a los requerimientos de producción.

---

# Anatomía de un archivo .service

Un archivo generado por Podman suele tener esta estructura:

```ini
[Unit]

Description=Podman container-web.service

Documentation=man:podman-generate-systemd(1)

Wants=network-online.target

After=network-online.target

[Service]

Environment=PODMAN_SYSTEMD_UNIT=%n

Restart=on-failure

TimeoutStopSec=70

ExecStart=

ExecStop=

ExecStopPost=

PIDFile=

Type=forking

[Install]

WantedBy=default.target
```

---

# Arquitectura

```text
.service

│

├── Unit

├── Service

└── Install
```

---

# Sección [Unit]

Describe el servicio.

No inicia procesos.

Define:

- descripción
- dependencias
- documentación
- orden de inicio

---

# Description

Ejemplo

```ini
Description=Servidor Web Nginx
```

Se visualiza mediante:

```bash
systemctl status
```

---

# Documentation

Permite asociar documentación.

Ejemplo

```ini
Documentation=man:podman-generate-systemd(1)
```

También puede utilizar:

```ini
Documentation=https://intranet/documentacion
```

---

# After

Indica que el servicio debe iniciar después de otro.

Ejemplo

```ini
After=network-online.target
```

---

# Arquitectura

```text
Red

↓

Network Online

↓

Container
```

---

# Before

Realiza el efecto contrario.

```ini
Before=httpd.service
```

---

# Wants

Indica una dependencia débil.

Ejemplo

```ini
Wants=network-online.target
```

Si falla:

```text
Network

↓

Puede continuar
```

---

# Requires

Es una dependencia fuerte.

Ejemplo

```ini
Requires=postgresql.service
```

Si PostgreSQL falla:

```text
Container

↓

No inicia
```

---

# Comparación

| Directiva | Si falla el servicio requerido |
|------------|-------------------------------|
| Wants | Continúa |
| Requires | No inicia |

---

# Ejemplo Empresarial

```text
API

↓

Requires

↓

PostgreSQL
```

---

# Sección [Service]

Aquí ocurre toda la ejecución.

Es la parte más importante.

---

# Type

Define cómo systemd administrará el proceso.

Los valores más comunes son:

```text
simple

forking

oneshot

notify

exec
```

---

# Type=forking

Es el más común en unidades generadas por Podman.

El proceso crea otro proceso hijo y luego finaliza el inicial.

---

# Type=simple

Muy utilizado por aplicaciones modernas.

Ejemplo:

```ini
Type=simple
```

---

# Restart

Una de las directivas más importantes.

---

# always

```ini
Restart=always
```

Siempre reinicia.

---

# on-failure

```ini
Restart=on-failure
```

Sólo reinicia cuando existe un error.

---

# no

```ini
Restart=no
```

Nunca reinicia.

---

# Comparación

| Valor | Reinicio |
|---------|----------|
| always | Siempre |
| on-failure | Sólo errores |
| no | Nunca |
| on-abnormal | Errores graves |
| on-success | Sólo si termina correctamente |

---

# RestartSec

Tiempo antes del reinicio.

```ini
RestartSec=5
```

---

# TimeoutStopSec

Tiempo máximo para detener el servicio.

Ejemplo

```ini
TimeoutStopSec=70
```

Después de este tiempo:

systemd forzará la finalización.

---

# KillMode

Controla cómo terminar procesos.

Ejemplo

```ini
KillMode=none
```

Otros valores:

```text
control-group

mixed

process

none
```

---

# Environment

Permite definir variables.

Ejemplo

```ini
Environment=APP_ENV=production
```

Consultar:

```bash
systemctl show
```

---

# EnvironmentFile

Permite cargar variables desde un archivo.

Ejemplo

```ini
EnvironmentFile=/etc/sysconfig/web
```

---

# Archivo

```text
/etc/sysconfig/web
```

Contenido

```text
PORT=8080

DEBUG=false
```

---

# Ventajas

No modificar el archivo `.service`.

Solo cambiar:

```text
/etc/sysconfig
```

---

# ExecStart

Es el comando que inicia el contenedor.

Ejemplo simplificado

```ini
ExecStart=/usr/bin/podman start web
```

---

# ExecStop

Detiene el servicio.

```ini
ExecStop=/usr/bin/podman stop web
```

---

# ExecReload

Recarga configuración.

```ini
ExecReload=/bin/kill -HUP $MAINPID
```

---

# ExecStopPost

Se ejecuta después del apagado.

Ejemplo

```ini
ExecStopPost=/usr/bin/podman rm web
```

---

# PIDFile

Indica dónde se encuentra el PID.

Ejemplo

```ini
PIDFile=/run/user/1000/container.pid
```

---

# StandardOutput

Controla la salida estándar.

```ini
StandardOutput=journal
```

---

# StandardError

Controla errores.

```ini
StandardError=journal
```

---

# Arquitectura

```text
Container

↓

stdout

↓

journal

↓

journalctl
```

---

# Sección [Install]

Controla cuándo será habilitado.

---

# WantedBy

Ejemplo

```ini
WantedBy=multi-user.target
```

---

# Rootless

Normalmente:

```ini
WantedBy=default.target
```

---

# Rootful

Generalmente:

```ini
WantedBy=multi-user.target
```

---

# Dependencias

Ejemplo

```text
Network

↓

PostgreSQL

↓

API

↓

Nginx
```

---

# Orden correcto

```ini
After=network-online.target

Requires=postgresql.service
```

---

# Ver dependencias

```bash
systemctl list-dependencies
```

---

# Ver árbol

```bash
systemctl list-dependencies \
container-web.service
```

---

# Consultar propiedades

```bash
systemctl show \
container-web.service
```

---

# Mostrar únicamente Restart

```bash
systemctl show \
-p Restart \
container-web.service
```

---

# Mostrar Environment

```bash
systemctl show \
-p Environment \
container-web.service
```

---

# Editar un servicio

Nunca modificar directamente.

Utilizar:

```bash
systemctl edit \
container-web.service
```

---

# Drop-In

Systemd crea:

```text
override.conf
```

---

# Arquitectura

```text
Original

↓

Override

↓

Configuración final
```

---

# Ubicación

```text
/etc/systemd/system/

container-web.service.d/

override.conf
```

---

# Ventajas

Cuando Podman genere nuevamente el servicio:

El archivo original puede reemplazarse.

El override permanecerá.

---

# Ver configuración completa

```bash
systemctl cat \
container-web.service
```

---

# Crear servicio para un Pod

Primero crear el Pod.

```bash
podman pod create \
--name webpod
```

---

Crear contenedores.

```bash
podman run \
--pod webpod \
-d \
nginx
```

---

Generar unidad.

```bash
podman generate systemd \
--files \
--name webpod
```

---

Resultado

```text
pod-webpod.service
```

---

# Arquitectura

```text
Pod

│

├── Nginx

├── Redis

└── API

↓

Systemd

↓

Pod
```

---

# Ventajas

Con un único servicio:

- inicia todos los contenedores
- detiene todos
- reinicia todos

---

# Rootless

Instalar en:

```text
~/.config/systemd/user
```

---

# Rootful

Instalar en:

```text
/ etc/systemd/system
```

---

# Consultar Journal

```bash
journalctl \
-u container-web.service
```

---

Tiempo real

```bash
journalctl \
-fu container-web.service
```

---

Últimos errores

```bash
journalctl \
-p err
```

---

Últimas 100 líneas

```bash
journalctl \
-u container-web.service \
-n 100
```

---

# Laboratorio RHCSA

## Laboratorio 1

Consultar un archivo.

```bash
cat container-web.service
```

---

## Laboratorio 2

Consultar Restart.

```bash
systemctl show \
-p Restart \
container-web.service
```

---

## Laboratorio 3

Consultar Environment.

```bash
systemctl show \
-p Environment \
container-web.service
```

---

## Laboratorio 4

Consultar dependencias.

```bash
systemctl list-dependencies \
container-web.service
```

---

## Laboratorio 5

Mostrar configuración.

```bash
systemctl cat \
container-web.service
```

---

## Laboratorio 6

Editar mediante override.

```bash
systemctl edit \
container-web.service
```

---

## Laboratorio 7

Consultar Journal.

```bash
journalctl \
-u container-web.service
```

---

## Laboratorio 8

Seguir Journal.

```bash
journalctl \
-fu container-web.service
```

---

## Laboratorio 9

Mostrar últimas líneas.

```bash
journalctl \
-u container-web.service \
-n 50
```

---

## Laboratorio 10

Crear Pod.

```bash
podman pod create \
--name webpod
```

---

## Laboratorio 11

Agregar contenedor.

```bash
podman run \
-d \
--pod webpod \
nginx
```

---

## Laboratorio 12

Generar unidad.

```bash
podman generate systemd \
--files \
--name webpod
```

---

## Laboratorio 13

Consultar propiedades.

```bash
systemctl show \
container-web.service
```

---

## Laboratorio 14

Consultar estado.

```bash
systemctl status \
container-web.service
```

---

## Laboratorio 15

Modificar `Restart=always` utilizando un archivo `override.conf` y verificar el cambio con:

```bash
systemctl show -p Restart container-web.service
```

---

# Buenas prácticas

- Utilizar `systemctl edit` en lugar de modificar directamente un archivo `.service`.
- Preferir archivos `EnvironmentFile` para variables de configuración.
- Documentar las dependencias críticas mediante `Requires=` y `After=`.
- Utilizar `Restart=on-failure` para la mayoría de aplicaciones de producción.
- Revisar periódicamente el Journal para detectar errores tempranos.

---

# Errores comunes

## Error 1

Modificar directamente un archivo generado por Podman y perder los cambios al regenerarlo.

---

## Error 2

Confundir `After=` con `Requires=`.

---

## Error 3

Olvidar ejecutar:

```bash
systemctl daemon-reload
```

después de modificar una unidad.

---

## Error 4

Utilizar `Restart=always` en servicios que deben detenerse deliberadamente.

---

## Error 5

Guardar credenciales sensibles directamente dentro del archivo `.service` en lugar de utilizar `EnvironmentFile`.

---

# Resumen

En esta segunda fase aprendimos a:

- Analizar completamente la estructura de un archivo `.service`.
- Comprender el propósito de las secciones `[Unit]`, `[Service]` y `[Install]`.
- Configurar dependencias mediante `After=`, `Requires=` y `Wants=`.
- Administrar políticas de reinicio y tiempos de espera.
- Utilizar variables de entorno y archivos de configuración externos.
- Personalizar servicios mediante archivos `override.conf`.
- Generar servicios para Pods completos.
- Inspeccionar propiedades, dependencias y registros utilizando `systemctl` y `journalctl`.

En la **Fase 3** aprenderemos a administrar servicios avanzados de Podman con systemd, incluyendo Quadlets (`.container`, `.pod`, `.network`, `.volume`), plantillas (`@.service`), timers, integración con actualizaciones automáticas (`podman auto-update`), recuperación ante fallos y escenarios empresariales utilizados en RHEL 9 y Fedora.

----

# 75. Systemd y Contenedores con Podman (Fase 3)

> **Módulo 10 – Contenedores con Podman**
>
> **Manual RHCSA**
>
> **Archivo:** `75-systemd-contenedores-podman.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase podrás:

- Comprender qué son los Quadlets.
- Administrar contenedores mediante archivos `.container`.
- Crear Pods mediante archivos `.pod`.
- Administrar redes mediante `.network`.
- Administrar volúmenes mediante `.volume`.
- Comprender las plantillas (`@.service`).
- Implementar actualizaciones automáticas mediante `podman auto-update`.
- Integrar systemd con escenarios empresariales.
- Prepararte para administrar contenedores modernos en Fedora y RHEL.

---

# Introducción

Durante varios años la forma tradicional de integrar Podman con systemd consistía en generar automáticamente archivos `.service`.

Ejemplo:

```bash
podman generate systemd
```

Aunque este método sigue funcionando, actualmente Red Hat recomienda utilizar una tecnología más moderna denominada:

```text
Quadlets
```

---

# ¿Qué son los Quadlets?

Los Quadlets son archivos de configuración que permiten describir un contenedor sin escribir manualmente un archivo `.service`.

Systemd interpreta estos archivos y genera automáticamente la unidad correspondiente.

---

# Arquitectura

```text
Archivo .container

        │

        ▼

Systemd Generator

        │

        ▼

Archivo .service

        │

        ▼

Contenedor
```

---

# Beneficios

Los Quadlets ofrecen numerosas ventajas:

- Configuración más sencilla.
- Menor cantidad de código.
- Mejor mantenimiento.
- Mayor integración con systemd.
- Menor probabilidad de errores.
- Reutilización.
- Mejor compatibilidad con versiones recientes de Podman.

---

# Tipos de Quadlets

| Archivo | Función |
|----------|----------|
| `.container` | Crear un contenedor |
| `.pod` | Crear un Pod |
| `.network` | Crear una red |
| `.volume` | Crear un volumen |
| `.image` | Descargar imágenes automáticamente |
| `.build` | Construir imágenes desde un Containerfile |

---

# Directorios

## Rootful

```text
/etc/containers/systemd/
```

---

## Rootless

```text
~/.config/containers/systemd/
```

---

# Arquitectura

```text
Usuario

        │

~/.config/containers/systemd

        │

        ▼

Quadlets

        │

        ▼

Systemd
```

---

# Crear el directorio

```bash
mkdir -p ~/.config/containers/systemd
```

---

# Primer Quadlet

Crear:

```text
web.container
```

---

Contenido

```ini
[Unit]

Description=Servidor Web

[Container]

Image=docker.io/library/nginx

PublishPort=8080:80

[Install]

WantedBy=default.target
```

---

# Explicación

```text
Unit

↓

Información del servicio

↓

Container

↓

Configuración Podman

↓

Install

↓

Inicio automático
```

---

# Recargar

```bash
systemctl --user daemon-reload
```

---

# Iniciar

```bash
systemctl --user start web.service
```

---

# Habilitar

```bash
systemctl --user enable web.service
```

---

# Consultar

```bash
systemctl --user status web.service
```

---

# Consultar Journal

```bash
journalctl --user \
-u web.service
```

---

# Crear un Pod

Archivo

```text
webpod.pod
```

---

Contenido

```ini
[Pod]

PodName=webpod

PublishPort=8080:80
```

---

# Iniciar

```bash
systemctl --user start webpod-pod.service
```

---

# Arquitectura

```text
Pod

│

├── API

├── Redis

└── Nginx
```

---

# Crear un Volumen

Archivo

```text
datos.volume
```

---

Contenido

```ini
[Volume]

VolumeName=datos
```

---

# Iniciar

```bash
systemctl --user start datos-volume.service
```

---

# Crear una Red

Archivo

```text
red.network
```

---

Contenido

```ini
[Network]

NetworkName=backend
```

---

# Iniciar

```bash
systemctl --user start backend-network.service
```

---

# Descargar Imágenes

Archivo

```text
ubi.image
```

---

Contenido

```ini
[Image]

Image=docker.io/redhat/ubi9
```

---

Resultado

La imagen será descargada automáticamente.

---

# Construcción Automática

Archivo

```text
app.build
```

---

Contenido

```ini
[Build]

ImageTag=miapp

File=Containerfile
```

---

# Arquitectura Completa

```text
.container

↓

.service

↓

Container


.pod

↓

.service

↓

Pod


.volume

↓

.service

↓

Volume


.network

↓

.service

↓

Network
```

---

# Plantillas de systemd

Systemd permite crear servicios reutilizables.

Ejemplo

```text
container@.service
```

---

# Instancias

```text
container@web

container@redis

container@api
```

---

# Arquitectura

```text
Plantilla

        │

────────┼─────────

        │

        ▼

Web

Redis

API
```

---

# Variables

Dentro de una plantilla:

```text
%i
```

representa el nombre de la instancia.

---

Ejemplo

```ini
ExecStart=/usr/bin/podman start %i
```

---

# Auto Update

Una característica muy utilizada.

Permite actualizar imágenes automáticamente.

---

Agregar al contenedor

```bash
--label io.containers.autoupdate=registry
```

---

Consultar

```bash
podman auto-update
```

---

Modo Dry Run

```bash
podman auto-update \
--dry-run
```

---

Actualizar

```bash
podman auto-update
```

---

Arquitectura

```text
Registry

↓

Nueva Imagen

↓

Auto Update

↓

Restart
```

---

# Timer

Systemd puede ejecutar:

```text
podman auto-update
```

periódicamente.

---

Consultar

```bash
systemctl list-timers
```

---

Ejemplo

```text
podman-auto-update.timer
```

---

Estado

```bash
systemctl status \
podman-auto-update.timer
```

---

Habilitar

```bash
sudo systemctl enable \
podman-auto-update.timer
```

---

Iniciar

```bash
sudo systemctl start \
podman-auto-update.timer
```

---

# Rollback

Si una actualización falla:

```bash
podman image ls
```

↓

Seleccionar versión anterior.

↓

Actualizar el Quadlet.

↓

Reiniciar.

---

# Logs

Consultar

```bash
journalctl \
-u podman-auto-update
```

---

# Arquitectura Empresarial

```text
Registro

        │

        ▼

Auto Update

        │

        ▼

Nueva Imagen

        │

        ▼

Restart

        │

        ▼

Servicio
```

---

# Escenario Empresarial

```text
Servidor

│

├── API.container

├── DB.container

├── Redis.container

├── Web.container

└── Backend.network
```

---

# Comparación

| Método | Recomendación |
|----------|---------------|
| podman run | Laboratorios |
| generate systemd | Compatible |
| Quadlets | Recomendado |
| Kubernetes | Grandes clústeres |

---

# Flujo recomendado

```text
Container

↓

Quadlet

↓

Systemd

↓

Journal

↓

Monitoreo

↓

Producción
```

---

# Laboratorio RHCSA

## Laboratorio 1

Crear el directorio.

```bash
mkdir -p \
~/.config/containers/systemd
```

---

## Laboratorio 2

Crear un archivo:

```text
web.container
```

---

## Laboratorio 3

Recargar systemd.

```bash
systemctl --user daemon-reload
```

---

## Laboratorio 4

Habilitar.

```bash
systemctl --user enable web.service
```

---

## Laboratorio 5

Iniciar.

```bash
systemctl --user start web.service
```

---

## Laboratorio 6

Consultar estado.

```bash
systemctl --user status web.service
```

---

## Laboratorio 7

Consultar Journal.

```bash
journalctl --user \
-u web.service
```

---

## Laboratorio 8

Crear un archivo `.pod`.

---

## Laboratorio 9

Crear un archivo `.volume`.

---

## Laboratorio 10

Crear un archivo `.network`.

---

## Laboratorio 11

Crear un archivo `.image`.

---

## Laboratorio 12

Consultar imágenes.

```bash
podman images
```

---

## Laboratorio 13

Consultar Timer.

```bash
systemctl list-timers
```

---

## Laboratorio 14

Ejecutar.

```bash
podman auto-update \
--dry-run
```

---

## Laboratorio 15

Consultar logs.

```bash
journalctl \
-u podman-auto-update
```

---

# Buenas prácticas

- Utilizar Quadlets para nuevos proyectos.
- Mantener un archivo por servicio.
- Utilizar nombres descriptivos.
- Documentar cada Quadlet.
- Utilizar `podman auto-update` únicamente cuando exista un proceso de validación.
- Probar nuevas imágenes antes de habilitar actualizaciones automáticas en producción.
- Separar redes, volúmenes y Pods mediante Quadlets independientes.

---

# Errores comunes

## Error 1

Modificar directamente archivos `.service` generados automáticamente por Quadlets.

---

## Error 2

Olvidar ejecutar:

```bash
systemctl --user daemon-reload
```

después de crear o modificar un Quadlet.

---

## Error 3

No habilitar el servicio correspondiente tras crear el archivo `.container`.

---

## Error 4

Ejecutar `podman auto-update` sin verificar previamente la compatibilidad de la nueva imagen.

---

## Error 5

Agrupar múltiples aplicaciones no relacionadas dentro del mismo Pod, dificultando su mantenimiento y escalabilidad.

---

# Resumen

En esta tercera fase aprendimos a:

- Comprender el funcionamiento de los Quadlets.
- Crear archivos `.container`, `.pod`, `.network`, `.volume`, `.image` y `.build`.
- Utilizar plantillas (`@.service`) para reutilizar configuraciones.
- Implementar actualizaciones automáticas con `podman auto-update`.
- Integrar Quadlets con systemd para administrar contenedores de forma moderna.
- Aplicar una arquitectura recomendada para servidores Fedora y Red Hat Enterprise Linux.

En la **Fase 4** consolidaremos todos estos conceptos mediante troubleshooting avanzado, análisis detallado de logs, recuperación ante fallos, auditorías de servicios, scripts de diagnóstico, escenarios reales de producción, checklist RHCSA, preguntas de repaso y un laboratorio final integral similar al examen oficial.

----


# 75. Systemd y Contenedores con Podman (Fase 4)

> **Módulo 10 – Contenedores con Podman**
>
> **Manual RHCSA**
>
> **Archivo:** `75-systemd-contenedores-podman.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase podrás:

- Diagnosticar problemas relacionados con systemd y Podman.
- Resolver errores en archivos `.service` y Quadlets.
- Auditar servicios Rootless y Rootful.
- Automatizar verificaciones mediante scripts.
- Aplicar procedimientos de recuperación ante fallos.
- Implementar buenas prácticas utilizadas en entornos empresariales.
- Resolver escenarios similares a los del examen RHCSA.

---

# Introducción

En las fases anteriores aprendimos:

- Cómo funciona systemd.
- Cómo integrar Podman con systemd.
- Cómo utilizar archivos `.service`.
- Cómo utilizar Quadlets.
- Cómo administrar Pods completos.

En esta última fase aprenderemos cómo diagnosticar problemas reales y administrar servicios de producción.

La mayoría de las incidencias en ambientes empresariales no ocurren durante la creación del contenedor, sino después de semanas o meses de funcionamiento.

Por ello, todo administrador RHCSA debe dominar las herramientas de diagnóstico.

---

# Flujo General de Diagnóstico

```text
Servicio falla

      │

      ▼

systemctl status

      │

      ▼

journalctl

      │

      ▼

podman ps

      │

      ▼

podman logs

      │

      ▼

podman inspect

      │

      ▼

Resolver
```

---

# Verificar el estado del servicio

Sistema

```bash
sudo systemctl status container-web.service
```

Usuario

```bash
systemctl --user status container-web.service
```

Consultar:

- Active
- Loaded
- Main PID
- Exit Code
- Journal

---

# Estados más comunes

| Estado | Significado |
|----------|-------------|
| active (running) | Servicio ejecutándose |
| inactive | Servicio detenido |
| failed | Error |
| activating | Iniciando |
| deactivating | Deteniéndose |
| exited | Finalizó correctamente |

---

# Escenario 1

## El servicio no inicia

Consultar

```bash
systemctl status container-web.service
```

Luego

```bash
journalctl -u container-web.service
```

Posibles causas:

- Imagen inexistente
- Error de sintaxis
- Puerto ocupado
- Volumen inexistente
- Dependencias no disponibles

---

# Escenario 2

## Error "Unit not found"

Consultar

```bash
systemctl list-unit-files
```

Verificar ubicación:

Rootful

```text
/etc/systemd/system/
```

Rootless

```text
~/.config/systemd/user/
```

---

# Escenario 3

## Se modificó el archivo pero no cambia el servicio

Olvidaste ejecutar:

```bash
systemctl daemon-reload
```

o

```bash
systemctl --user daemon-reload
```

---

# Escenario 4

## El servicio inicia manualmente pero no después del reinicio

Consultar

```bash
systemctl is-enabled container-web.service
```

Resultado esperado

```text
enabled
```

Si aparece

```text
disabled
```

Ejecutar

```bash
systemctl enable container-web.service
```

---

# Escenario 5

## Rootless deja de funcionar al cerrar sesión

Consultar

```bash
loginctl show-user $USER
```

Buscar

```text
Linger=yes
```

Si no existe

```bash
sudo loginctl enable-linger $USER
```

---

# Escenario 6

## El contenedor se reinicia continuamente

Consultar

```bash
podman ps
```

↓

```bash
podman logs web
```

↓

```bash
systemctl status
```

Buscar

```text
Restart Counter
```

---

# Escenario 7

## Puerto ocupado

Consultar

```bash
ss -ltnp
```

o

```bash
ss -tulpn
```

Buscar

```text
:80

:443

:8080
```

---

# Escenario 8

## El contenedor inicia pero responde con errores

Consultar

```bash
podman logs web
```

↓

```bash
journalctl -u container-web.service
```

↓

```bash
podman inspect web
```

---

# Escenario 9

## Error con SELinux

Consultar

```bash
getenforce
```

↓

```bash
ls -lZ
```

↓

```bash
podman inspect
```

Verificar:

- Bind Mounts
- Etiquetas SELinux
- Opciones `:Z` o `:z`

---

# Escenario 10

## Problemas con Quadlets

Consultar

```bash
systemctl --user daemon-reload
```

↓

```bash
systemctl --user status web.service
```

↓

```bash
journalctl --user -u web.service
```

---

# Diagnóstico completo

```text
systemctl status

↓

journalctl

↓

podman ps

↓

podman logs

↓

podman inspect

↓

SELinux

↓

Storage

↓

Network

↓

Solución
```

---

# Consultar Journal

Últimas líneas

```bash
journalctl \
-u container-web.service \
-n 100
```

---

Tiempo real

```bash
journalctl \
-fu container-web.service
```

---

Errores

```bash
journalctl \
-p err
```

---

Desde el último arranque

```bash
journalctl \
-b
```

---

Consultar únicamente hoy

```bash
journalctl \
--since today
```

---

# Verificar Quadlets

Consultar

```bash
ls ~/.config/containers/systemd
```

---

Consultar archivos

```bash
cat web.container
```

---

Consultar servicio generado

```bash
systemctl --user cat web.service
```

---

# Validar sintaxis

Consultar

```bash
systemd-analyze verify \
container-web.service
```

Para Rootless

```bash
systemd-analyze verify \
~/.config/systemd/user/container-web.service
```

---

# Ver tiempos de inicio

Consultar

```bash
systemd-analyze blame
```

---

Dependencias críticas

```bash
systemd-analyze critical-chain
```

---

# Consultar propiedades

```bash
systemctl show \
container-web.service
```

---

Consultar únicamente Restart

```bash
systemctl show \
-p Restart \
container-web.service
```

---

Consultar PID

```bash
systemctl show \
-p MainPID \
container-web.service
```

---

Consultar memoria

```bash
systemctl show \
-p MemoryCurrent \
container-web.service
```

---

# Auditoría completa

```text
Servicio

↓

Estado

↓

Journal

↓

Inspect

↓

Logs

↓

Recursos

↓

Conclusión
```

---

# Script de Auditoría

Guardar como

```text
systemd_podman_audit.sh
```

```bash
#!/bin/bash

echo "====================================="
echo " SYSTEMD + PODMAN AUDIT"
echo "====================================="

echo
echo "Servicios activos:"
systemctl list-units --type=service

echo
echo "Contenedores:"
podman ps

echo
echo "Imágenes:"
podman images

echo
echo "Volúmenes:"
podman volume ls

echo
echo "Redes:"
podman network ls

echo
echo "Uso de almacenamiento:"
podman system df

echo
echo "Espacio disponible:"
df -h

echo
echo "SELinux:"
getenforce
```

Permisos

```bash
chmod +x systemd_podman_audit.sh
```

---

# Script para revisar servicios

Guardar como

```text
check_services.sh
```

```bash
#!/bin/bash

SERVICES=(
container-web.service
container-api.service
container-db.service
)

for s in "${SERVICES[@]}"
do

echo
echo "======================="
echo "$s"
echo "======================="

systemctl status "$s" \
--no-pager

done
```

---

# Script de Logs

```bash
#!/bin/bash

for c in $(podman ps --format "{{.Names}}")
do

echo
echo "====================="
echo "$c"
echo "====================="

podman logs \
--tail 20 \
"$c"

done
```

---

# Escenario Empresarial

```text
Servidor

│

├── PostgreSQL

├── Redis

├── API

├── Nginx

└── Prometheus

↓

systemd

↓

Podman
```

---

# Procedimiento de Recuperación

```text
Error

↓

Logs

↓

Identificar causa

↓

Corregir

↓

daemon-reload

↓

Restart

↓

Verificación
```

---

# Checklist Empresarial

Antes de colocar un servicio en producción verificar:

```text
□ enable

□ daemon-reload

□ journal limpio

□ logs sin errores

□ Restart configurado

□ SELinux validado

□ Volúmenes persistentes

□ Red configurada

□ Recursos limitados

□ Backup probado

□ Auto Update documentado

□ Dependencias verificadas
```

---

# Laboratorio RHCSA

## Laboratorio 1

Consultar estado.

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

Consultar almacenamiento.

```bash
podman system df
```

---

## Laboratorio 6

Consultar redes.

```bash
podman network ls
```

---

## Laboratorio 7

Consultar volúmenes.

```bash
podman volume ls
```

---

## Laboratorio 8

Consultar estado Rootless.

```bash
systemctl --user status
```

---

## Laboratorio 9

Consultar tiempos.

```bash
systemd-analyze blame
```

---

## Laboratorio 10

Consultar dependencias.

```bash
systemd-analyze critical-chain
```

---

## Laboratorio 11

Verificar sintaxis.

```bash
systemd-analyze verify \
container-web.service
```

---

## Laboratorio 12

Ejecutar auditoría.

```bash
./systemd_podman_audit.sh
```

---

## Laboratorio 13

Ejecutar revisión de servicios.

```bash
./check_services.sh
```

---

## Laboratorio 14

Consultar propiedades.

```bash
systemctl show \
container-web.service
```

---

## Laboratorio 15

Reiniciar un servicio y verificar que:

- El contenedor inicia correctamente.
- El Journal no muestra errores.
- El estado final es **active (running)**.
- La política `Restart=` funciona según lo esperado.

---

# Preguntas de Repaso

1. ¿Cuál es la diferencia entre `systemctl` y `systemctl --user`?
2. ¿Qué comando permite recargar las unidades después de modificarlas?
3. ¿Qué utilidad tiene `journalctl`?
4. ¿Cómo verificar si un servicio está habilitado?
5. ¿Qué hace `systemd-analyze verify`?
6. ¿Qué comando muestra los servicios más lentos durante el arranque?
7. ¿Qué herramienta permite inspeccionar completamente un contenedor?
8. ¿Qué comando muestra el consumo de almacenamiento de Podman?
9. ¿Qué archivo utiliza un Quadlet para describir un contenedor?
10. ¿Por qué es recomendable utilizar archivos `override.conf`?

---

# Respuestas

1. `systemctl` administra servicios del sistema; `systemctl --user` administra servicios del usuario.
2. `systemctl daemon-reload` o `systemctl --user daemon-reload`.
3. Consultar los registros generados por systemd y los servicios.
4. `systemctl is-enabled nombre_del_servicio`.
5. Verifica la sintaxis y consistencia de una unidad systemd.
6. `systemd-analyze blame`.
7. `podman inspect`.
8. `podman system df`.
9. Un archivo con extensión `.container`.
10. Porque permiten personalizar un servicio sin modificar el archivo original generado automáticamente.

---

# Desafío Final RHCSA

Un servidor Fedora contiene una aplicación formada por:

- Nginx
- PostgreSQL
- Redis

Debes realizar las siguientes tareas:

1. Crear Quadlets para cada servicio.
2. Crear un Quadlet para la red.
3. Crear Quadlets para los volúmenes persistentes.
4. Habilitar todos los servicios mediante systemd.
5. Configurar reinicio automático cuando ocurra un fallo.
6. Validar la sintaxis de todas las unidades.
7. Consultar el Journal de cada servicio.
8. Verificar el consumo de almacenamiento.
9. Ejecutar una auditoría completa utilizando un script Bash.
10. Documentar el procedimiento de recuperación ante fallos.

---

# Buenas prácticas

- Utilizar Quadlets para nuevas implementaciones.
- Validar las unidades con `systemd-analyze verify` antes de habilitarlas.
- Revisar periódicamente el Journal.
- Automatizar auditorías mediante scripts.
- Configurar políticas de reinicio apropiadas (`Restart=on-failure` suele ser una buena opción).
- Mantener separados los servicios Rootless y Rootful.
- Probar los procedimientos de recuperación antes de pasar a producción.
- Documentar todas las personalizaciones realizadas mediante archivos `override.conf`.

---

# Errores comunes

## Error 1

Olvidar ejecutar `daemon-reload` después de modificar una unidad o un Quadlet.

---

## Error 2

No revisar el Journal y tratar de resolver problemas únicamente con `podman logs`.

---

## Error 3

Modificar directamente un archivo generado automáticamente por Podman o por un Quadlet.

---

## Error 4

No habilitar (`enable`) los servicios que deben iniciarse automáticamente tras un reinicio.

---

## Error 5

No validar la sintaxis de una unidad antes de ponerla en producción.

---

# Resumen del Capítulo 75

En este capítulo aprendimos a:

- Integrar Podman con systemd.
- Generar y administrar unidades `.service`.
- Comprender la estructura de las unidades de systemd.
- Configurar dependencias, políticas de reinicio y variables de entorno.
- Administrar contenedores mediante Quadlets (`.container`, `.pod`, `.network`, `.volume`, `.image` y `.build`).
- Implementar actualizaciones automáticas mediante `podman auto-update`.
- Diagnosticar problemas utilizando `systemctl`, `journalctl`, `systemd-analyze` y `podman inspect`.
- Automatizar auditorías mediante scripts Bash.
- Aplicar buenas prácticas de administración utilizadas en servidores Fedora y Red Hat Enterprise Linux.

---

# Fin del Capítulo









