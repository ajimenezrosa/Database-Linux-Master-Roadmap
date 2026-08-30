# 69. Instalación y Configuración de Podman (Fase 4)

> **Módulo 10 – Contenedores con Podman**
>
> **Manual RHCSA**
>
> **Archivo:** `69-instalacion-configuracion-podman.md`

---

# Objetivos de esta fase

Al finalizar esta fase podrás:

- Validar completamente una instalación de Podman.
- Diagnosticar problemas de configuración.
- Resolver incidencias relacionadas con almacenamiento.
- Resolver incidencias relacionadas con redes.
- Resolver incidencias relacionadas con SELinux.
- Resolver incidencias relacionadas con permisos.
- Automatizar auditorías.
- Prepararte para el examen RHCSA.
- Resolver escenarios similares a producción.

---

# Metodología de Diagnóstico RHCSA

Cuando un contenedor presenta problemas **no se debe comenzar modificando archivos al azar**.

Siempre sigue una metodología.

```text
                 Problema

                     │

                     ▼

        ¿Podman está instalado?

                     │

          Sí                     No

          │                      │

          ▼                      ▼

 ¿Podman funciona?         Instalar Podman

          │

          ▼

¿La imagen existe?

          │

          ▼

¿El contenedor existe?

          │

          ▼

¿Está ejecutándose?

          │

          ▼

¿Qué dicen los logs?

          │

          ▼

¿Hay errores de SELinux?

          │

          ▼

¿Problema de red?

          │

          ▼

¿Problema de almacenamiento?

          │

          ▼

Resolver
```

Nunca saltes pasos.

---

# Lista rápida de diagnóstico

Los primeros comandos que un administrador RHCSA suele ejecutar son:

```bash
podman info
```

```bash
podman version
```

```bash
podman ps -a
```

```bash
podman images
```

```bash
podman network ls
```

```bash
podman volume ls
```

```bash
podman system df
```

```bash
getenforce
```

```bash
firewall-cmd --list-all
```

---

# Escenario 1
## Podman no está instalado

Diagnóstico:

```bash
podman
```

Resultado:

```text
command not found
```

Verificar:

```bash
rpm -q podman
```

Solución:

```bash
sudo dnf install -y podman
```

---

# Escenario 2
## Podman está instalado pero no funciona

Consultar:

```bash
podman info
```

Revisar:

- Runtime
- Storage
- Configuración

---

# Escenario 3
## Error descargando imágenes

Ejemplo:

```bash
podman pull alpine
```

Error:

```text
network unreachable
```

Verificar:

```bash
ping quay.io
```

```bash
curl https://quay.io
```

---

# Escenario 4
## Error de DNS

Consultar:

```bash
cat /etc/resolv.conf
```

También:

```bash
podman run \
--rm \
docker.io/library/alpine \
cat /etc/resolv.conf
```

---

# Escenario 5
## Error de permisos

Ejemplo:

```text
Permission denied
```

Verificar:

```bash
ls -l
```

```bash
ls -Zd
```

```bash
getenforce
```

---

# Escenario 6
## Error de SELinux

Consultar:

```bash
ausearch \
-m AVC \
-ts recent
```

También:

```bash
journalctl \
-t setroubleshoot
```

---

# Escenario 7
## Error de almacenamiento

Consultar:

```bash
podman system df
```

Espacio libre:

```bash
df -h
```

Inodos:

```bash
df -i
```

---

# Escenario 8
## Error de OverlayFS

Consultar:

```bash
podman info
```

Buscar:

```text
GraphDriverName
```

Debe indicar:

```text
overlay
```

---

# Escenario 9
## El contenedor termina inmediatamente

Consultar:

```bash
podman ps -a
```

Después:

```bash
podman logs nombre
```

Consultar código:

```bash
podman inspect nombre \
--format '{{.State.ExitCode}}'
```

---

# Escenario 10
## No responde el puerto

Consultar:

```bash
podman port nombre
```

Verificar:

```bash
ss -lnt
```

Firewall:

```bash
firewall-cmd --list-all
```

---

# Escenario 11
## Error de Rootless

Consultar:

```bash
podman info \
--format '{{.Host.Security.Rootless}}'
```

Revisar:

```bash
cat /etc/subuid
```

```bash
cat /etc/subgid
```

---

# Escenario 12
## Error de Runtime

Consultar:

```bash
podman info
```

Buscar:

```text
OCIRuntime
```

Verificar:

```bash
rpm -q crun
```

---

# Escenario 13
## Error de Red

Consultar:

```bash
podman network ls
```

Después:

```bash
podman network inspect podman
```

---

# Escenario 14
## Error en GraphRoot

Consultar:

```bash
podman info \
--format '{{.Store.GraphRoot}}'
```

Verificar permisos.

---

# Escenario 15
## Imagen dañada

Eliminar:

```bash
podman rmi imagen
```

Descargar nuevamente.

---

# Script de Auditoría RHCSA

Guardar como:

```text
audit_podman.sh
```

```bash
#!/bin/bash

echo "==============================="
echo " AUDITORIA PODMAN"
echo "==============================="

echo
echo "VERSION"
podman --version

echo
echo "INFO"
podman info

echo
echo "IMAGENES"
podman images

echo
echo "CONTENEDORES"
podman ps -a

echo
echo "REDES"
podman network ls

echo
echo "VOLUMENES"
podman volume ls

echo
echo "ALMACENAMIENTO"
podman system df

echo
echo "SELINUX"
getenforce

echo
echo "FIREWALL"
firewall-cmd --list-all

echo
echo "ESPACIO"
df -h

echo
echo "INODOS"
df -i
```

Permisos:

```bash
chmod +x audit_podman.sh
```

---

# Laboratorio RHCSA

## Laboratorio 1

Verificar instalación.

```bash
podman version
```

---

## Laboratorio 2

Consultar información.

```bash
podman info
```

---

## Laboratorio 3

Ver imágenes.

```bash
podman images
```

---

## Laboratorio 4

Descargar Alpine.

```bash
podman pull docker.io/library/alpine
```

---

## Laboratorio 5

Ejecutar.

```bash
podman run \
--rm \
docker.io/library/alpine \
hostname
```

---

## Laboratorio 6

Entrar.

```bash
podman run \
-it \
docker.io/library/alpine \
/bin/sh
```

---

## Laboratorio 7

Ver Rootless.

```bash
podman info \
--format '{{.Host.Security.Rootless}}'
```

---

## Laboratorio 8

Consultar GraphRoot.

```bash
podman info \
--format '{{.Store.GraphRoot}}'
```

---

## Laboratorio 9

Consultar Runtime.

```bash
podman info \
--format '{{.Host.OCIRuntime.Name}}'
```

---

## Laboratorio 10

Ver redes.

```bash
podman network ls
```

---

## Laboratorio 11

Inspeccionar red.

```bash
podman network inspect podman
```

---

## Laboratorio 12

Crear volumen.

```bash
podman volume create datos
```

---

## Laboratorio 13

Ver volúmenes.

```bash
podman volume ls
```

---

## Laboratorio 14

Eliminar volumen.

```bash
podman volume rm datos
```

---

## Laboratorio 15

Ver almacenamiento.

```bash
podman system df
```

---

## Laboratorio 16

Ver espacio.

```bash
df -h
```

---

## Laboratorio 17

Consultar firewall.

```bash
firewall-cmd --list-all
```

---

## Laboratorio 18

Consultar SELinux.

```bash
getenforce
```

---

## Laboratorio 19

Buscar AVC.

```bash
ausearch \
-m AVC \
-ts recent
```

---

## Laboratorio 20

Ejecutar auditoría.

```bash
./audit_podman.sh
```

---

## Laboratorio 21

Crear un contenedor.

```bash
podman run \
-d \
--name web \
-p 8080:80 \
docker.io/library/httpd
```

---

## Laboratorio 22

Consultar.

```bash
podman ps
```

---

## Laboratorio 23

Consultar logs.

```bash
podman logs web
```

---

## Laboratorio 24

Entrar.

```bash
podman exec \
-it web \
/bin/bash
```

---

## Laboratorio 25

Consultar procesos.

```bash
podman top web
```

---

## Laboratorio 26

Consultar estadísticas.

```bash
podman stats
```

---

## Laboratorio 27

Detener.

```bash
podman stop web
```

---

## Laboratorio 28

Iniciar.

```bash
podman start web
```

---

## Laboratorio 29

Eliminar.

```bash
podman rm -f web
```

---

## Laboratorio 30

Eliminar imágenes no utilizadas.

```bash
podman image prune
```

---

# Checklist RHCSA

```text
□ Podman instalado

□ Buildah instalado

□ Skopeo instalado

□ conmon instalado

□ crun instalado

□ Rootless operativo

□ SELinux Enforcing

□ OverlayFS operativo

□ Runtime validado

□ GraphRoot validado

□ RunRoot validado

□ Redes operativas

□ Volúmenes operativos

□ Descarga de imágenes correcta

□ Contenedores funcionales

□ Auditoría ejecutada

□ Firewall revisado

□ Logs revisados

□ Permisos revisados

□ Diagnóstico completado
```

---

# Preguntas de Repaso

1. ¿Qué archivo configura los registros?
2. ¿Qué archivo configura el almacenamiento?
3. ¿Qué archivo configura Podman?
4. ¿Qué es GraphRoot?
5. ¿Qué es RunRoot?
6. ¿Qué es OverlayFS?
7. ¿Qué es Rootless?
8. ¿Qué es User Namespace?
9. ¿Qué es SubUID?
10. ¿Qué es SubGID?
11. ¿Qué función tiene SELinux?
12. ¿Qué hace `podman info`?
13. ¿Qué hace `podman system df`?
14. ¿Qué hace `podman network inspect`?
15. ¿Qué hace `podman volume ls`?
16. ¿Cómo validar el Runtime?
17. ¿Cómo validar Rootless?
18. ¿Cómo validar GraphRoot?
19. ¿Cómo validar OverlayFS?
20. ¿Cuál es el primer comando de diagnóstico?

---

# Respuestas

1. `registries.conf`
2. `storage.conf`
3. `containers.conf`
4. Directorio persistente de imágenes y capas.
5. Directorio temporal de ejecución.
6. Sistema de archivos por capas.
7. Contenedores sin privilegios administrativos.
8. Mapeo de usuarios del contenedor al host.
9. Rango de UID subordinados.
10. Rango de GID subordinados.
11. Control de acceso obligatorio (MAC).
12. Muestra la configuración completa de Podman.
13. Muestra el uso del almacenamiento.
14. Inspecciona una red de Podman.
15. Lista los volúmenes.
16. `podman info`
17. `podman info --format '{{.Host.Security.Rootless}}'`
18. `podman info --format '{{.Store.GraphRoot}}'`
19. `podman info --format '{{.Store.GraphDriverName}}'`
20. `podman info`

---

# Desafío Final RHCSA

Se entrega un servidor Fedora recién instalado.

Debe realizarse lo siguiente:

1. Instalar Podman.
2. Verificar la versión.
3. Confirmar que el modo Rootless funciona.
4. Validar SELinux.
5. Confirmar el Runtime OCI.
6. Confirmar OverlayFS.
7. Descargar una imagen oficial.
8. Ejecutar un contenedor HTTP.
9. Publicar el puerto **8080**.
10. Confirmar el acceso mediante `curl`.
11. Consultar los logs.
12. Inspeccionar el contenedor.
13. Revisar el almacenamiento.
14. Crear y eliminar un volumen.
15. Ejecutar el script de auditoría.
16. Documentar todos los comandos utilizados.

---

# Buenas Prácticas

- Mantener actualizado Podman y sus dependencias.
- Utilizar imágenes oficiales y de confianza.
- Trabajar en modo **Rootless** siempre que sea posible.
- Mantener **SELinux** en modo **Enforcing**.
- Evitar el uso de `--privileged` salvo necesidad justificada.
- Utilizar nombres completos de imágenes (`registro/namespace/repositorio:etiqueta`).
- Revisar periódicamente el espacio de almacenamiento con `podman system df`.
- Auditar el entorno con scripts automatizados antes de poner un servidor en producción.
- Realizar pruebas en un entorno de laboratorio antes de modificar configuraciones globales.

---

# Resumen del Capítulo 69

En este capítulo aprendimos a:

- Instalar Podman correctamente.
- Comprender la estructura de configuración de `/etc/containers`.
- Configurar `containers.conf`, `registries.conf` y `storage.conf`.
- Entender el funcionamiento de **Rootless**, **User Namespaces**, **SubUID/SubGID** y **OverlayFS**.
- Integrar Podman con **SELinux** y el firewall.
- Diagnosticar y resolver problemas comunes de instalación y configuración.
- Auditar el entorno mediante scripts y herramientas integradas.
- Aplicar una metodología sistemática de troubleshooting, similar a la utilizada en entornos empresariales y en el examen **RHCSA**.

---

# Fin del capítulo

