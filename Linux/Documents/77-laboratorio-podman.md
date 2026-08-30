# 77. Laboratorio Integral de Podman para RHCSA (Fase 1)

> **Módulo 10 – Contenedores con Podman**
>
> **Manual RHCSA**
>
> **Archivo:** `77-laboratorio-podman.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Aplicar todos los conocimientos adquiridos durante el módulo de Podman.
- Construir un laboratorio empresarial completo.
- Implementar una arquitectura multicontenedor.
- Configurar almacenamiento persistente.
- Configurar redes personalizadas.
- Administrar Pods.
- Administrar contenedores mediante systemd.
- Prepararte para un escenario muy similar al examen RHCSA.

---

# Introducción

Durante los capítulos anteriores aprendimos:

- Instalación
- Imágenes
- Contenedores
- Redes
- Volúmenes
- Pods
- Rootless
- Rootful
- systemd
- Quadlets
- Troubleshooting

Ahora integraremos todo en un único laboratorio.

Este laboratorio simula el despliegue de una aplicación empresarial.

---

# Escenario

Una empresa desea implementar una aplicación interna compuesta por:

- Servidor Web (Nginx)
- API
- Base de datos PostgreSQL
- Redis

Todo debe ejecutarse mediante Podman.

---

# Arquitectura

```text
                    Usuarios

                        │

                        ▼

                 Puerto 8080

                        │

                        ▼

                    Nginx Web

                        │

        ┌───────────────┴───────────────┐

        ▼                               ▼

      API                           Redis Cache

        │

        ▼

 PostgreSQL Database
```

---

# Componentes

| Servicio | Imagen |
|----------|---------|
| Web | nginx |
| Base de datos | postgres |
| Cache | redis |
| API | ubi9 |

---

# Requisitos

El servidor debe cumplir:

- Red privada
- Volúmenes persistentes
- Inicio automático
- Reinicio automático
- Logs disponibles
- Fácil recuperación

---

# Recursos del laboratorio

| Recurso | Nombre |
|----------|---------|
| Red | backend |
| Pod | empresa |
| Volumen PostgreSQL | postgres_data |
| Volumen Redis | redis_data |
| Contenedor Web | nginx-web |
| Contenedor DB | postgres-db |
| Contenedor Redis | redis-cache |
| Contenedor API | api-service |

---

# Paso 1

Actualizar Podman.

```bash
sudo dnf update
```

---

Verificar versión.

```bash
podman version
```

---

Verificar información.

```bash
podman info
```

---

# Paso 2

Crear red.

```bash
podman network create backend
```

---

Consultar.

```bash
podman network ls
```

---

Arquitectura

```text
backend

│

├── nginx

├── api

├── redis

└── postgres
```

---

# Paso 3

Crear volúmenes.

```bash
podman volume create postgres_data
```

---

```bash
podman volume create redis_data
```

---

Consultar.

```bash
podman volume ls
```

---

Arquitectura

```text
Volumes

│

├── postgres_data

└── redis_data
```

---

# Paso 4

Crear Pod.

```bash
podman pod create \
--name empresa \
-p 8080:80
```

---

Consultar.

```bash
podman pod ls
```

---

Arquitectura

```text
empresa

│

├── nginx

├── api

├── postgres

└── redis
```

---

# Paso 5

Crear PostgreSQL.

```bash
podman run -d \
--name postgres-db \
--pod empresa \
-v postgres_data:/var/lib/postgresql/data \
-e POSTGRES_PASSWORD=RHCSA123 \
docker.io/library/postgres
```

---

Consultar.

```bash
podman ps
```

---

# Paso 6

Crear Redis.

```bash
podman run -d \
--name redis-cache \
--pod empresa \
-v redis_data:/data \
docker.io/library/redis
```

---

# Paso 7

Crear Nginx.

```bash
podman run -d \
--name nginx-web \
--pod empresa \
docker.io/library/nginx
```

---

# Paso 8

Crear API.

```bash
podman run -dt \
--name api-service \
--pod empresa \
registry.access.redhat.com/ubi9 \
bash
```

---

# Arquitectura General

```text
                 Pod empresa

        │

        ├──────────────┐

        ▼              ▼

     Nginx         PostgreSQL

        │

        ▼

      API

        │

        ▼

      Redis
```

---

# Paso 9

Consultar Pods.

```bash
podman pod ps
```

---

Consultar contenedores.

```bash
podman ps
```

---

Consultar imágenes.

```bash
podman images
```

---

Consultar redes.

```bash
podman network ls
```

---

Consultar volúmenes.

```bash
podman volume ls
```

---

# Paso 10

Entrar al Pod.

```bash
podman exec \
-it postgres-db bash
```

---

Entrar a Redis.

```bash
podman exec \
-it redis-cache bash
```

---

Entrar a Nginx.

```bash
podman exec \
-it nginx-web bash
```

---

Entrar a la API.

```bash
podman exec \
-it api-service bash
```

---

# Verificar comunicación

Desde la API.

```bash
ping postgres-db
```

---

```bash
ping redis-cache
```

---

```bash
ping nginx-web
```

---

# Verificar puertos

```bash
ss -tln
```

---

Consultar publicación.

```bash
podman port nginx-web
```

---

# Verificar almacenamiento

Consultar.

```bash
podman volume inspect \
postgres_data
```

---

Consultar.

```bash
podman volume inspect \
redis_data
```

---

# Verificar Pod

Consultar.

```bash
podman pod inspect empresa
```

---

Arquitectura

```text
Pod

↓

Containers

↓

Volumes

↓

Network
```

---

# Monitoreo

Consultar.

```bash
podman stats
```

---

Consultar procesos.

```bash
podman top nginx-web
```

---

Consultar logs.

```bash
podman logs nginx-web
```

---

Consultar inspect.

```bash
podman inspect nginx-web
```

---

# Comandos importantes

| Comando | Función |
|----------|----------|
| podman ps | Contenedores |
| podman pod ps | Pods |
| podman volume ls | Volúmenes |
| podman network ls | Redes |
| podman inspect | Configuración |
| podman logs | Logs |
| podman exec | Acceso |
| podman stats | Recursos |

---

# Laboratorio RHCSA

## Laboratorio 1

Actualizar Podman.

---

## Laboratorio 2

Crear una red personalizada.

---

## Laboratorio 3

Crear dos volúmenes persistentes.

---

## Laboratorio 4

Crear un Pod.

---

## Laboratorio 5

Crear PostgreSQL.

---

## Laboratorio 6

Crear Redis.

---

## Laboratorio 7

Crear Nginx.

---

## Laboratorio 8

Crear una API basada en UBI.

---

## Laboratorio 9

Verificar que todos los contenedores están ejecutándose.

---

## Laboratorio 10

Entrar a PostgreSQL.

---

## Laboratorio 11

Entrar a Redis.

---

## Laboratorio 12

Verificar la comunicación entre los contenedores.

---

## Laboratorio 13

Consultar estadísticas.

---

## Laboratorio 14

Consultar la configuración del Pod.

---

## Laboratorio 15

Documentar toda la arquitectura creada mediante un diagrama y explicar la función de cada componente.

---

# Buenas prácticas

- Utilizar nombres descriptivos para Pods y contenedores.
- Separar los datos persistentes mediante volúmenes dedicados.
- Mantener todos los servicios relacionados dentro del mismo Pod cuando compartan el mismo ciclo de vida.
- Verificar la conectividad entre los contenedores antes de continuar con la configuración.
- Documentar la arquitectura antes de pasar al siguiente laboratorio.

---

# Errores comunes

## Error 1

Crear los contenedores antes de crear la red o los volúmenes.

---

## Error 2

No utilizar almacenamiento persistente para PostgreSQL.

---

## Error 3

Exponer múltiples puertos innecesarios en el Host.

---

## Error 4

No verificar que todos los contenedores pertenecen al Pod correcto.

---

## Error 5

No comprobar la comunicación entre los servicios antes de desplegar la aplicación.

---

# Resumen

En esta primera fase construimos la infraestructura base de un entorno empresarial utilizando Podman:

- Red personalizada.
- Volúmenes persistentes.
- Pod principal.
- Cuatro contenedores de servicios.
- Monitoreo inicial.
- Validación de conectividad.
- Arquitectura multicontenedor.

En la **Fase 2** integraremos esta infraestructura con **systemd**, implementaremos **Quadlets**, configuraremos **inicio automático**, **reinicios**, **actualizaciones automáticas**, **logs centralizados** y aplicaremos prácticas utilizadas en servidores Red Hat Enterprise Linux de producción.

---
# 77. Laboratorio Integral de Podman para RHCSA (Fase 2)

> **Módulo 10 – Contenedores con Podman**
>
> **Manual RHCSA**
>
> **Archivo:** `77-laboratorio-podman.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase podrás:

- Integrar el laboratorio con systemd.
- Administrar Pods mediante servicios.
- Crear Quadlets para producción.
- Implementar inicio automático.
- Configurar reinicio automático.
- Automatizar actualizaciones.
- Centralizar los registros.
- Preparar una arquitectura similar a la utilizada en empresas.

---

# Introducción

Hasta este momento hemos construido una infraestructura completa utilizando Podman.

Sin embargo, un entorno empresarial necesita mucho más que ejecutar contenedores.

Debe garantizar:

- Inicio automático.
- Recuperación ante fallos.
- Administración centralizada.
- Integración con systemd.
- Actualizaciones controladas.
- Auditoría.
- Facilidad de mantenimiento.

En esta fase convertiremos nuestro laboratorio en una infraestructura preparada para producción.

---

# Arquitectura Objetivo

```text
                Fedora Server

                       │

                systemd Manager

                       │

        ┌──────────────┴──────────────┐

        ▼                             ▼

     Quadlets                   Journal

        │

        ▼

     Pod Empresa

        │

 ┌──────┼──────────────┐

 ▼      ▼              ▼

Nginx PostgreSQL     Redis

        │

        ▼

      API
```

---

# Paso 1

## Verificar el estado del Pod

Consultar

```bash
podman pod ps
```

Resultado esperado

```text
NAME

STATUS

CREATED
```

---

# Consultar los contenedores

```bash
podman ps
```

Todos los servicios deben aparecer como:

```text
Up
```

---

# Paso 2

## Generar el servicio del Pod

Consultar

```bash
podman generate systemd \
--files \
--name empresa
```

Resultado

```text
pod-empresa.service
```

---

# Verificar el archivo

```bash
ls
```

Resultado esperado

```text
pod-empresa.service
```

---

# Paso 3

## Instalar el servicio

Rootful

```bash
sudo cp pod-empresa.service \
/etc/systemd/system/
```

---

Recargar systemd

```bash
sudo systemctl daemon-reload
```

---

# Habilitar

```bash
sudo systemctl enable \
pod-empresa.service
```

---

# Iniciar

```bash
sudo systemctl start \
pod-empresa.service
```

---

# Consultar

```bash
systemctl status \
pod-empresa.service
```

---

# Arquitectura

```text
systemd

↓

pod-empresa.service

↓

Pod Empresa

↓

Todos los contenedores
```

---

# Paso 4

## Reinicio Automático

Editar

```bash
sudo systemctl edit \
pod-empresa.service
```

Agregar

```ini
[Service]

Restart=always

RestartSec=5
```

---

Recargar

```bash
sudo systemctl daemon-reload
```

---

Reiniciar

```bash
sudo systemctl restart \
pod-empresa.service
```

---

Consultar

```bash
systemctl show \
-p Restart \
pod-empresa.service
```

---

# Arquitectura

```text
Falla

↓

systemd

↓

5 segundos

↓

Nuevo inicio
```

---

# Paso 5

## Crear un Quadlet

Crear directorio

```bash
sudo mkdir -p \
/etc/containers/systemd
```

---

Crear archivo

```text
empresa.container
```

---

Contenido

```ini
[Unit]

Description=Servidor API Empresa

[Container]

Image=registry.access.redhat.com/ubi9

ContainerName=empresa-api

Exec=bash

[Install]

WantedBy=multi-user.target
```

---

# Recargar

```bash
sudo systemctl daemon-reload
```

---

Consultar

```bash
systemctl status \
empresa.service
```

---

# Arquitectura

```text
Quadlet

↓

Generator

↓

Service

↓

Container
```

---

# Paso 6

## Crear un Quadlet para PostgreSQL

Archivo

```text
postgres.container
```

Contenido

```ini
[Unit]

Description=PostgreSQL

[Container]

Image=docker.io/library/postgres

ContainerName=postgres-db

Volume=postgres_data:/var/lib/postgresql/data

Environment=POSTGRES_PASSWORD=RHCSA123

[Install]

WantedBy=multi-user.target
```

---

# Paso 7

## Crear un Quadlet para Redis

Archivo

```text
redis.container
```

---

Contenido

```ini
[Container]

Image=docker.io/library/redis

ContainerName=redis-cache

Volume=redis_data:/data

[Install]

WantedBy=multi-user.target
```

---

# Paso 8

## Crear un Quadlet para Nginx

Archivo

```text
nginx.container
```

---

Contenido

```ini
[Container]

Image=docker.io/library/nginx

ContainerName=nginx-web

PublishPort=8080:80

[Install]

WantedBy=multi-user.target
```

---

# Paso 9

## Validar Quadlets

Consultar

```bash
systemctl daemon-reload
```

---

Consultar

```bash
systemctl status \
nginx.service
```

---

Consultar

```bash
systemctl status \
postgres.service
```

---

Consultar

```bash
systemctl status \
redis.service
```

---

# Paso 10

## Configurar Auto Update

Consultar imagen

```bash
podman images
```

---

Agregar Label

```text
io.containers.autoupdate=registry
```

---

Consultar

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

# Arquitectura

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

# Paso 11

## Configurar Timer

Consultar

```bash
systemctl list-timers
```

---

Consultar

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

# Paso 12

## Consultar Logs

Consultar

```bash
journalctl \
-u pod-empresa.service
```

---

Tiempo real

```bash
journalctl \
-fu pod-empresa.service
```

---

Últimos errores

```bash
journalctl \
-p err
```

---

# Paso 13

## Validar Dependencias

Consultar

```bash
systemctl list-dependencies \
pod-empresa.service
```

---

Consultar

```bash
systemctl show \
pod-empresa.service
```

---

# Paso 14

## Verificar Recursos

Consultar

```bash
podman stats
```

---

Consultar

```bash
systemd-cgtop
```

---

Consultar

```bash
free -h
```

---

Consultar

```bash
df -h
```

---

# Arquitectura Final

```text
                 Fedora

                    │

              systemd

                    │

       pod-empresa.service

                    │

              Pod Empresa

      ┌─────────┼───────────┐

      ▼         ▼           ▼

   Nginx     PostgreSQL   Redis

                    │

                    ▼

                 API UBI
```

---

# Laboratorio RHCSA

## Laboratorio 1

Generar el archivo systemd del Pod.

---

## Laboratorio 2

Instalar el servicio.

---

## Laboratorio 3

Habilitar el inicio automático.

---

## Laboratorio 4

Configurar reinicio automático.

---

## Laboratorio 5

Crear el directorio de Quadlets.

---

## Laboratorio 6

Crear el Quadlet de PostgreSQL.

---

## Laboratorio 7

Crear el Quadlet de Redis.

---

## Laboratorio 8

Crear el Quadlet de Nginx.

---

## Laboratorio 9

Crear el Quadlet de la API.

---

## Laboratorio 10

Recargar systemd.

---

## Laboratorio 11

Verificar el estado de todos los servicios.

---

## Laboratorio 12

Consultar los registros del Journal.

---

## Laboratorio 13

Ejecutar `podman auto-update --dry-run`.

---

## Laboratorio 14

Verificar el consumo de recursos mediante `podman stats` y `systemd-cgtop`.

---

## Laboratorio 15

Reiniciar el servidor y comprobar que:

- Todos los Quadlets se cargan correctamente.
- El Pod inicia automáticamente.
- Los contenedores recuperan su estado.
- Los volúmenes mantienen la información.
- El Journal no muestra errores.

---

# Buenas prácticas

- Administrar servicios de producción mediante systemd.
- Utilizar Quadlets para nuevas implementaciones.
- Habilitar únicamente los servicios necesarios.
- Configurar políticas de reinicio apropiadas.
- Centralizar los registros en Journal.
- Verificar periódicamente las actualizaciones automáticas.
- Probar el inicio automático después de cada cambio importante.

---

# Errores comunes

## Error 1

Olvidar ejecutar:

```bash
systemctl daemon-reload
```

después de crear un Quadlet.

---

## Error 2

Modificar directamente archivos `.service` generados automáticamente.

---

## Error 3

No habilitar (`enable`) los servicios antes de reiniciar el servidor.

---

## Error 4

No comprobar el Journal tras habilitar un nuevo servicio.

---

## Error 5

Configurar `Restart=always` sin investigar primero la causa de una falla repetitiva.

---

# Resumen

En esta segunda fase transformamos el laboratorio básico en una infraestructura administrada por **systemd**, incorporando:

- Servicios persistentes.
- Quadlets.
- Inicio automático.
- Reinicio automático.
- Actualizaciones automáticas.
- Monitoreo mediante Journal.
- Integración con herramientas de administración de Fedora y Red Hat Enterprise Linux.

En la **Fase 3** realizaremos pruebas de fallos controlados, simularemos incidentes reales de producción, aplicaremos procedimientos de recuperación, automatizaremos auditorías y pondremos a prueba toda la arquitectura mediante escenarios similares a los encontrados por un Administrador Linux en un entorno empresarial.
---

# 77. Laboratorio Integral de Podman para RHCSA (Fase 3)

> **Módulo 10 – Contenedores con Podman**
>
> **Manual RHCSA**
>
> **Archivo:** `77-laboratorio-podman.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase podrás:

- Simular fallos reales en un entorno empresarial.
- Diagnosticar problemas utilizando una metodología profesional.
- Recuperar servicios sin pérdida de datos.
- Analizar logs del sistema y de los contenedores.
- Validar la persistencia de los volúmenes.
- Automatizar verificaciones de salud.
- Desarrollar habilidades similares a las requeridas en el examen RHCSA.

---

# Introducción

En producción los problemas rara vez aparecen de forma aislada.

Generalmente un incidente afecta varios componentes al mismo tiempo.

Por ejemplo:

- Un volumen deja de montarse.
- PostgreSQL deja de responder.
- Nginx devuelve errores 502.
- Redis consume toda la memoria.
- Un reinicio inesperado impide iniciar los servicios.
- SELinux bloquea un directorio.

El objetivo de esta fase es aprender a diagnosticar y recuperar el sistema completo.

---

# Arquitectura del Laboratorio

```text
                     Usuarios

                         │

                         ▼

                    Puerto 8080

                         │

                         ▼

                    Nginx Web

                         │

        ┌────────────────┴────────────────┐

        ▼                                 ▼

      API                           Redis Cache

        │

        ▼

 PostgreSQL Database

        │

        ▼

 Volúmenes Persistentes

        │

        ▼

     Fedora + systemd
```

---

# Metodología de Recuperación

```text
Incidente

↓

Identificar

↓

Recolectar Evidencias

↓

Analizar

↓

Corregir

↓

Validar

↓

Documentar
```

---

# Escenario 1

# PostgreSQL no inicia

Síntoma

```text
database system is shut down
```

Consultar

```bash
systemctl status postgres.service
```

Consultar

```bash
journalctl -u postgres.service
```

Consultar

```bash
podman logs postgres-db
```

Consultar

```bash
podman inspect postgres-db
```

Validar

- Volumen
- Password
- Espacio disponible
- Permisos

---

# Escenario 2

# Redis consume demasiada memoria

Consultar

```bash
podman stats
```

Consultar

```bash
free -h
```

Consultar

```bash
podman exec \
-it redis-cache redis-cli INFO memory
```

Consultar Kernel

```bash
journalctl -k
```

Buscar

```text
OOM
```

---

# Escenario 3

# Nginx devuelve Error 502

Consultar

```bash
podman logs nginx-web
```

Consultar

```bash
curl localhost
```

Consultar

```bash
podman exec \
-it nginx-web bash
```

Verificar

```bash
curl api-service
```

Si falla

↓

Problema de conectividad.

---

# Escenario 4

# API no responde

Consultar

```bash
podman logs api-service
```

Consultar

```bash
podman exec \
-it api-service bash
```

Verificar procesos

```bash
ps aux
```

Verificar puertos

```bash
ss -tln
```

---

# Escenario 5

# El Pod desapareció

Consultar

```bash
podman pod ps
```

Consultar

```bash
podman ps -a
```

Consultar

```bash
systemctl status \
pod-empresa.service
```

---

# Escenario 6

# El servidor reinició

Verificar

```bash
systemctl status \
pod-empresa.service
```

Consultar

```bash
podman ps
```

Validar

```bash
systemctl is-enabled \
pod-empresa.service
```

---

# Escenario 7

# Volumen no monta

Consultar

```bash
podman volume ls
```

Consultar

```bash
podman volume inspect \
postgres_data
```

Consultar

```bash
ls -lZ
```

---

# Escenario 8

# SELinux bloquea escritura

Consultar

```bash
ausearch -m avc
```

Consultar

```bash
journalctl | grep AVC
```

Consultar

```bash
ls -lZ
```

Corregir

```text
:Z
```

---

# Escenario 9

# Puerto ocupado

Consultar

```bash
ss -tulpn
```

Buscar

```text
8080
```

---

# Escenario 10

# DNS falla

Entrar

```bash
podman exec \
-it api-service bash
```

Consultar

```bash
cat /etc/resolv.conf
```

Consultar

```bash
ping postgres-db
```

Consultar

```bash
ping redis-cache
```

---

# Recuperación Completa

```text
Host

↓

systemd

↓

Pod

↓

Containers

↓

Network

↓

Storage

↓

SELinux

↓

Application
```

---

# Auditoría del Laboratorio

Verificar

```bash
podman ps
```

↓

```bash
podman pod ps
```

↓

```bash
podman volume ls
```

↓

```bash
podman network ls
```

↓

```bash
systemctl status
```

↓

```bash
journalctl
```

---

# Script de Estado General

Guardar como

```text
lab_status.sh
```

```bash
#!/bin/bash

echo "===================================="
echo " LAB STATUS"
echo "===================================="

echo
echo "Pods"
podman pod ps

echo
echo "Containers"
podman ps

echo
echo "Volumes"
podman volume ls

echo
echo "Networks"
podman network ls

echo
echo "Resources"
podman stats --no-stream

echo
echo "Disk"
df -h

echo
echo "Memory"
free -h
```

---

# Script de Validación

Guardar como

```text
validate_lab.sh
```

```bash
#!/bin/bash

echo "=========================="

echo "Checking Containers"

echo "=========================="

CONTAINERS=(
nginx-web
postgres-db
redis-cache
api-service
)

for c in "${CONTAINERS[@]}"
do

echo

echo "$c"

podman inspect \
--format '{{.State.Status}}' \
$c

done
```

---

# Script de Backup

Guardar como

```text
backup_lab.sh
```

```bash
#!/bin/bash

mkdir -p backup

echo "Exportando Contenedores..."

for c in nginx-web postgres-db redis-cache api-service
do

podman inspect "$c" \
> backup/$c.json

done

echo

echo "Exportando Logs"

for c in nginx-web postgres-db redis-cache api-service
do

podman logs "$c" \
> backup/$c.log

done

echo

echo "Backup Finalizado."
```

---

# Script de Monitoreo

Guardar como

```text
watch_lab.sh
```

```bash
#!/bin/bash

watch -n 5 '
echo "===== PODMAN ====="
podman ps

echo

echo "===== STATS ====="
podman stats --no-stream
'
```

---

# Verificación de Persistencia

Crear un archivo

```bash
podman exec \
-it postgres-db bash
```

Crear

```bash
touch \
/var/lib/postgresql/data/test.txt
```

Salir

↓

Eliminar el contenedor

↓

Crearlo nuevamente

↓

Verificar

```bash
ls
```

Si el archivo existe

↓

El volumen funciona correctamente.

---

# Simulación de Recuperación

```text
Eliminar Contenedor

↓

Mantener Volumen

↓

Crear nuevamente

↓

Montar mismo volumen

↓

Validar Información
```

---

# Validación Final

Comprobar

```text
□ Pod activo

□ Todos los contenedores activos

□ PostgreSQL responde

□ Redis responde

□ Nginx responde

□ API responde

□ Volúmenes presentes

□ Red presente

□ Journal limpio

□ Sin errores SELinux
```

---

# Arquitectura Final

```text
                    Fedora

                       │

                  systemd

                       │

                  Pod Empresa

        ┌─────────┼──────────────┐

        ▼         ▼              ▼

     Nginx     PostgreSQL      Redis

                     │

                     ▼

                   API

                     │

                     ▼

          Volúmenes Persistentes

                     │

                     ▼

             Monitorización
```

---

# Laboratorio RHCSA

## Laboratorio 1

Detener PostgreSQL y recuperarlo utilizando únicamente los logs.

---

## Laboratorio 2

Simular un puerto ocupado y resolver el conflicto.

---

## Laboratorio 3

Crear un error de permisos en un volumen y corregirlo.

---

## Laboratorio 4

Generar un evento SELinux y resolverlo correctamente.

---

## Laboratorio 5

Eliminar un contenedor sin eliminar el volumen y comprobar la persistencia.

---

## Laboratorio 6

Simular una caída de Redis y verificar el reinicio automático.

---

## Laboratorio 7

Deshabilitar temporalmente un servicio systemd y restaurarlo.

---

## Laboratorio 8

Ejecutar el script `lab_status.sh`.

---

## Laboratorio 9

Ejecutar `validate_lab.sh`.

---

## Laboratorio 10

Ejecutar `backup_lab.sh`.

---

## Laboratorio 11

Ejecutar `watch_lab.sh`.

---

## Laboratorio 12

Reiniciar el servidor y validar toda la infraestructura.

---

## Laboratorio 13

Analizar el Journal completo buscando errores relacionados con los servicios del laboratorio.

---

## Laboratorio 14

Simular la pérdida de conectividad entre dos contenedores y restaurarla utilizando una red personalizada.

---

## Laboratorio 15

Documentar cada incidente, la evidencia recopilada, la causa raíz, la solución aplicada y las pruebas realizadas para confirmar la recuperación del entorno.

---

# Buenas prácticas

- Mantener scripts de validación y auditoría bajo control de versiones.
- Realizar pruebas de recuperación periódicamente.
- Verificar la persistencia antes de actualizar imágenes o contenedores.
- Automatizar la recopilación de evidencias durante un incidente.
- Documentar cada procedimiento de recuperación.
- Revisar periódicamente el Journal y el consumo de recursos.
- Validar la arquitectura después de cada cambio importante.

---

# Errores comunes

## Error 1

Eliminar un contenedor antes de respaldar su configuración y registros.

---

## Error 2

No comprobar la persistencia de los volúmenes después de recrear un contenedor.

---

## Error 3

Ignorar los mensajes de SELinux y asumir que el problema es de permisos tradicionales.

---

## Error 4

Modificar varias configuraciones al mismo tiempo durante un incidente.

---

## Error 5

No verificar el funcionamiento completo del laboratorio después de aplicar una corrección.

---

# Resumen

En esta tercera fase desarrollamos un entorno de pruebas orientado a la recuperación de desastres y al diagnóstico de incidencias reales. Aprendimos a:

- Simular fallos de servicios críticos.
- Recuperar contenedores utilizando una metodología estructurada.
- Validar la persistencia de datos.
- Automatizar auditorías y respaldos.
- Supervisar el estado del laboratorio mediante scripts.
- Preparar un entorno de entrenamiento muy similar al utilizado por administradores Linux en producción.

En la **Fase 4** realizaremos un laboratorio final tipo **RHCSA**, donde se integrarán todos los conocimientos del módulo de Podman mediante un examen práctico completo, un checklist profesional, preguntas de repaso y un desafío final que simulará una implementación empresarial de principio a fin.
----


# 77. Laboratorio Integral de Podman para RHCSA (Fase 4)

> **Módulo 10 – Contenedores con Podman**
>
> **Manual RHCSA**
>
> **Archivo:** `77-laboratorio-podman.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase podrás:

- Resolver un laboratorio completo tipo RHCSA.
- Implementar una infraestructura empresarial desde cero.
- Diagnosticar múltiples fallos simultáneamente.
- Recuperar servicios críticos.
- Validar una implementación completa.
- Documentar un procedimiento profesional.
- Prepararte para escenarios reales de producción.

---

# Introducción

Esta fase representa el examen práctico del módulo de Podman.

No se trata únicamente de ejecutar comandos, sino de aplicar una metodología ordenada para:

- Implementar.
- Verificar.
- Diagnosticar.
- Recuperar.
- Documentar.

El objetivo es demostrar que puedes administrar una plataforma de contenedores de principio a fin.

---

# Escenario Empresarial

La empresa **RHCSA Solutions** ha desplegado una aplicación compuesta por:

- Nginx
- API REST
- PostgreSQL
- Redis

Después de una actualización del servidor comenzaron a presentarse diversos problemas.

El administrador debe restaurar completamente el servicio.

---

# Arquitectura Esperada

```text
                     Internet

                         │

                         ▼

                    Puerto 8080

                         │

                    Nginx Reverse Proxy

                         │

          ┌──────────────┴──────────────┐

          ▼                             ▼

      API REST                    Redis Cache

          │

          ▼

     PostgreSQL

          │

          ▼

 Volumen Persistente

          │

          ▼

 Fedora + Podman + systemd
```

---

# Incidencias Simuladas

El administrador recibe el siguiente reporte:

```text
□ Nginx devuelve HTTP 502.

□ PostgreSQL no inicia.

□ Redis consume demasiada memoria.

□ La API no responde.

□ El servidor fue reiniciado.

□ SELinux registra eventos AVC.

□ Un volumen no monta.

□ El Journal muestra errores.

□ Algunos servicios no inician automáticamente.

□ El espacio en disco está al 95%.
```

---

# Procedimiento Profesional

```text
Recibir Incidente

        │

        ▼

Recolectar Evidencias

        │

        ▼

Verificar Host

        │

        ▼

Verificar systemd

        │

        ▼

Verificar Pod

        │

        ▼

Verificar Contenedores

        │

        ▼

Verificar Red

        │

        ▼

Verificar Storage

        │

        ▼

Verificar SELinux

        │

        ▼

Corregir

        │

        ▼

Validar

        │

        ▼

Documentar
```

---

# Paso 1

## Estado del servidor

Consultar

```bash
uptime
```

Consultar

```bash
hostnamectl
```

Consultar

```bash
free -h
```

Consultar

```bash
df -h
```

Consultar

```bash
df -i
```

---

# Paso 2

## Verificar systemd

Consultar

```bash
systemctl status
```

Consultar

```bash
systemctl list-units \
--failed
```

Consultar

```bash
systemctl list-units \
--type=service
```

---

# Paso 3

## Verificar Pod

Consultar

```bash
podman pod ps
```

Consultar

```bash
podman pod inspect empresa
```

---

# Paso 4

## Verificar Contenedores

Consultar

```bash
podman ps -a
```

Consultar

```bash
podman stats
```

Consultar

```bash
podman top nginx-web
```

Consultar

```bash
podman inspect postgres-db
```

---

# Paso 5

## Verificar Logs

Consultar

```bash
podman logs nginx-web
```

Consultar

```bash
podman logs postgres-db
```

Consultar

```bash
podman logs api-service
```

Consultar

```bash
podman logs redis-cache
```

---

# Paso 6

## Revisar Journal

Consultar

```bash
journalctl -xe
```

Consultar

```bash
journalctl \
-u pod-empresa.service
```

Consultar

```bash
journalctl -k
```

---

# Paso 7

## Revisar SELinux

Consultar

```bash
getenforce
```

Consultar

```bash
ausearch -m avc
```

Consultar

```bash
journalctl | grep AVC
```

Consultar

```bash
ls -lZ
```

---

# Paso 8

## Revisar Red

Consultar

```bash
podman network ls
```

Consultar

```bash
podman network inspect backend
```

Consultar

```bash
ss -tulpn
```

Consultar

```bash
ip addr
```

Consultar

```bash
ip route
```

---

# Paso 9

## Revisar Volúmenes

Consultar

```bash
podman volume ls
```

Consultar

```bash
podman volume inspect postgres_data
```

Consultar

```bash
podman volume inspect redis_data
```

---

# Paso 10

## Validar Persistencia

Entrar

```bash
podman exec \
-it postgres-db bash
```

Crear

```bash
touch \
/var/lib/postgresql/data/prueba.txt
```

Salir.

Recrear el contenedor.

Verificar nuevamente el archivo.

---

# Paso 11

## Validar Recursos

Consultar

```bash
systemd-cgtop
```

Consultar

```bash
top
```

Consultar

```bash
vmstat
```

Consultar

```bash
podman stats
```

---

# Paso 12

## Recuperación

```text
Host OK

↓

systemd OK

↓

Pod OK

↓

Containers OK

↓

Network OK

↓

Storage OK

↓

SELinux OK

↓

Aplicación OK
```

---

# Script Final de Auditoría

Guardar como

```text
full_audit.sh
```

```bash
#!/bin/bash

echo "==============================="
echo " FULL AUDIT REPORT"
echo "==============================="

echo
echo "HOST"
hostnamectl

echo
echo "UPTIME"
uptime

echo
echo "DISK"
df -h

echo
echo "INODES"
df -i

echo
echo "MEMORY"
free -h

echo
echo "PODS"
podman pod ps

echo
echo "CONTAINERS"
podman ps -a

echo
echo "VOLUMES"
podman volume ls

echo
echo "NETWORKS"
podman network ls

echo
echo "SYSTEMD"
systemctl --failed

echo
echo "SELINUX"
getenforce

echo
echo "STATS"
podman stats --no-stream

echo
echo "AUDIT COMPLETED"
```

---

# Script de Verificación Final

Guardar como

```text
verify_environment.sh
```

```bash
#!/bin/bash

echo "Checking services..."

SERVICES=(
nginx-web
postgres-db
redis-cache
api-service
)

for s in "${SERVICES[@]}"
do
echo
echo "$s"

podman inspect \
--format '{{.State.Status}}' \
$s
done

echo
echo "Checking Pod"

podman pod ps

echo
echo "Checking Network"

podman network ls

echo
echo "Checking Volumes"

podman volume ls

echo
echo "Environment OK"
```

---

# Checklist Profesional

```text
□ Host operativo

□ Espacio suficiente

□ Memoria disponible

□ Swap estable

□ Pod activo

□ Todos los contenedores activos

□ PostgreSQL responde

□ Redis responde

□ API responde

□ Nginx responde

□ Red funcional

□ DNS correcto

□ Volúmenes presentes

□ Persistencia validada

□ Journal limpio

□ SELinux sin bloqueos

□ Reinicio automático probado

□ Backup disponible

□ Monitoreo funcionando

□ Documentación actualizada
```

---

# Examen Práctico RHCSA

## Ejercicio 1

Crear una red personalizada denominada `backend-rhcsa`.

---

## Ejercicio 2

Crear dos volúmenes persistentes para PostgreSQL y Redis.

---

## Ejercicio 3

Crear un Pod llamado `empresa`.

---

## Ejercicio 4

Desplegar Nginx, PostgreSQL, Redis y una API basada en UBI.

---

## Ejercicio 5

Configurar inicio automático mediante systemd.

---

## Ejercicio 6

Crear Quadlets para todos los servicios.

---

## Ejercicio 7

Verificar la comunicación entre todos los contenedores.

---

## Ejercicio 8

Simular un fallo de PostgreSQL y recuperarlo utilizando los registros.

---

## Ejercicio 9

Corregir un problema de contexto SELinux sin desactivar SELinux.

---

## Ejercicio 10

Eliminar un contenedor manteniendo los datos mediante un volumen persistente.

---

## Ejercicio 11

Configurar actualizaciones automáticas con `podman auto-update`.

---

## Ejercicio 12

Generar un informe completo utilizando `full_audit.sh`.

---

## Ejercicio 13

Reiniciar el servidor y comprobar que toda la infraestructura se recupera automáticamente.

---

## Ejercicio 14

Resolver un problema de conectividad entre Nginx y la API utilizando herramientas de diagnóstico de red.

---

## Ejercicio 15

Documentar todas las acciones realizadas indicando:

- Problema.
- Evidencia.
- Diagnóstico.
- Solución.
- Validación.
- Recomendaciones para evitar que el incidente se repita.

---

# Preguntas de Repaso

1. ¿Cuál es el primer paso antes de modificar un contenedor con problemas?
2. ¿Qué diferencia existe entre `podman logs` y `journalctl`?
3. ¿Qué utilidad tiene `podman inspect`?
4. ¿Cómo validar que un volumen es persistente?
5. ¿Qué comando muestra el consumo de recursos de un contenedor?
6. ¿Qué herramienta permite revisar los eventos del Kernel?
7. ¿Cómo identificar bloqueos relacionados con SELinux?
8. ¿Qué función cumplen los Quadlets?
9. ¿Qué ventajas ofrece systemd para administrar Podman?
10. ¿Por qué es importante documentar todas las incidencias?

---

# Resumen del Laboratorio

Durante este laboratorio integral aprendimos a:

- Construir una arquitectura empresarial completa con Podman.
- Implementar Pods, redes y volúmenes persistentes.
- Integrar Podman con systemd.
- Utilizar Quadlets para administrar servicios.
- Diagnosticar fallos complejos.
- Recuperar contenedores y servicios.
- Automatizar auditorías mediante scripts.
- Validar la persistencia de los datos.
- Aplicar procedimientos profesionales de troubleshooting.
- Documentar un proceso completo de recuperación.

---

# Conclusión del Módulo 10

Con este laboratorio concluye el módulo dedicado a **Podman**.

A estas alturas ya dominas:

- Instalación de Podman.
- Administración de imágenes.
- Creación y gestión de contenedores.
- Redes.
- Volúmenes.
- Pods.
- Rootless y Rootful.
- Integración con systemd.
- Quadlets.
- Actualizaciones automáticas.
- Troubleshooting.
- Implementaciones empresariales.
- Recuperación ante fallos.
- Automatización mediante Bash.
- Buenas prácticas para entornos Red Hat Enterprise Linux.

Este conjunto de conocimientos proporciona una base sólida para afrontar tanto el examen **RHCSA** como tareas de administración de contenedores en entornos de producción.

---

# Fin del Archivo

<!-- ```text
77-laboratorio-podman.md
```

**Próximo capítulo:**

```text
78-introduccion-ansible.md
```

 -->


