# 69. Instalación y Configuración de Podman (Fase 1)

> **Módulo 10: Contenedores con Podman**  
> **Manual RHCSA**  
> **Archivo:** `69-instalacion-configuracion-podman.md`

---

# Objetivos de aprendizaje

Al finalizar este capítulo podrás:

- Comprender por qué Red Hat utiliza Podman como motor de contenedores.
- Identificar los requisitos para instalar Podman.
- Instalar Podman en RHEL, Rocky Linux, AlmaLinux, CentOS Stream y Fedora.
- Verificar que la instalación sea correcta.
- Conocer los paquetes que forman parte del ecosistema de contenedores.
- Comprender el propósito del paquete **container-tools**.
- Verificar versiones de Podman y componentes relacionados.
- Ejecutar el primer diagnóstico del entorno.
- Comprender dónde almacena Podman su información.
- Preparar un servidor para comenzar a trabajar con contenedores.
- Realizar un laboratorio completo de instalación.

---

# Introducción

Antes de crear un contenedor debemos asegurarnos de que el sistema operativo dispone de todas las herramientas necesarias.

En RHEL 8 y RHEL 9, Red Hat reemplazó Docker como herramienta recomendada y adoptó **Podman** como su motor oficial de contenedores.

Podman forma parte del ecosistema oficial de Red Hat y está diseñado para trabajar junto con otras herramientas como:

- Buildah
- Skopeo
- conmon
- crun
- Netavark
- Aardvark DNS

Estas herramientas permiten construir imágenes, ejecutarlas y administrarlas sin depender de un daemon central permanente.

---

# 1. ¿Por qué Red Hat utiliza Podman?

Las principales razones son:

- Arquitectura sin daemon permanente.
- Mejor integración con systemd.
- Soporte completo para contenedores Rootless.
- Mayor seguridad.
- Compatibilidad con OCI.
- Integración con SELinux.
- Compatibilidad con Kubernetes.
- Administración por usuario.
- Menor superficie de ataque.

Red Hat considera a Podman la herramienta estándar para la administración de contenedores en RHEL.

---

# 2. Ecosistema de herramientas de contenedores

```text
                 Usuario
                     │
                     ▼
                 Podman CLI
                     │
         ┌───────────┼───────────┐
         │           │           │
         ▼           ▼           ▼
     Buildah      Skopeo     Systemd
         │
         ▼
      Runtime OCI
         │
         ▼
       conmon
         │
         ▼
        crun
         │
         ▼
     Kernel Linux
```

Cada componente cumple una función distinta.

---

# 3. Requisitos del sistema

Para trabajar cómodamente con Podman se recomienda:

- Sistema operativo compatible.
- Acceso administrativo.
- Espacio suficiente en disco.
- SELinux habilitado.
- Kernel actualizado.
- Repositorios oficiales configurados.

Recomendaciones de laboratorio:

| Recurso | Recomendado |
|----------|------------|
| RAM | 4 GB o más |
| CPU | 2 núcleos |
| Disco | 20 GB libres |
| SELinux | Enforcing |
| Internet | Sí |

---

# 4. Sistemas compatibles

Podman está disponible oficialmente para:

- Red Hat Enterprise Linux
- Fedora
- Rocky Linux
- AlmaLinux
- CentOS Stream

También existen versiones para:

- Ubuntu
- Debian
- openSUSE
- Arch Linux

En este manual trabajaremos principalmente sobre plataformas compatibles con RHEL.

---

# 5. Comprobar la distribución

Antes de instalar cualquier paquete conviene identificar el sistema operativo.

```bash
cat /etc/os-release
```

Ejemplo:

```text
NAME="Red Hat Enterprise Linux"
VERSION="9.6"
```

También:

```bash
hostnamectl
```

o

```bash
cat /etc/redhat-release
```

---

# 6. Comprobar la versión del Kernel

```bash
uname -r
```

Ejemplo:

```text
5.14.0-570.el9.x86_64
```

El kernel es fundamental porque Podman utiliza características del kernel Linux como:

- Namespaces
- cgroups
- SELinux
- OverlayFS

---

# 7. Verificar SELinux

```bash
getenforce
```

Resultado recomendado:

```text
Enforcing
```

También:

```bash
sestatus
```

Salida resumida:

```text
SELinux status: enabled
Current mode: enforcing
```

No es recomendable deshabilitar SELinux para utilizar contenedores.

---

# 8. Verificar acceso a Internet

```bash
ping -c 4 registry.redhat.io
```

o

```bash
ping -c 4 quay.io
```

También puede comprobarse mediante:

```bash
curl https://quay.io
```

La conectividad será necesaria para descargar imágenes.

---

# 9. Actualizar el sistema

Antes de instalar Podman es recomendable actualizar los paquetes.

```bash
sudo dnf update
```

o

```bash
sudo dnf upgrade
```

Posteriormente puede ser necesario reiniciar si se actualizó el kernel.

---

# 10. Comprobar repositorios

```bash
dnf repolist
```

En RHEL normalmente aparecerán:

```text
BaseOS
AppStream
```

En Fedora:

```text
fedora
updates
```

---

# 11. Buscar Podman

```bash
dnf search podman
```

También:

```bash
dnf info podman
```

La información mostrará:

- Versión
- Arquitectura
- Repositorio
- Descripción
- Dependencias

---

# 12. Instalar Podman

En la mayoría de distribuciones compatibles con RHEL:

```bash
sudo dnf install -y podman
```

El gestor de paquetes resolverá automáticamente las dependencias necesarias.

---

# 13. ¿Qué paquetes se instalan?

Podemos consultarlos mediante:

```bash
rpm -q podman
```

Y revisar sus dependencias:

```bash
rpm -qR podman
```

Entre los componentes habituales encontraremos:

- conmon
- crun
- containers-common
- containers-image
- netavark
- aardvark-dns

La lista exacta puede variar según la versión del sistema.

---

# 14. El paquete `container-tools`

En RHEL existe un grupo de herramientas denominado **container-tools**.

Puede consultarse con:

```bash
dnf module list container-tools
```

o

```bash
dnf group list
```

Este conjunto incluye herramientas relacionadas con contenedores.

---

# 15. Buildah

Verificar instalación:

```bash
buildah --version
```

Su función principal es construir imágenes OCI.

---

# 16. Skopeo

```bash
skopeo --version
```

Permite copiar e inspeccionar imágenes sin ejecutarlas.

---

# 17. conmon

Verificar:

```bash
rpm -q conmon
```

Conmon monitoriza los procesos de los contenedores.

---

# 18. crun

```bash
rpm -q crun
```

Es el runtime OCI utilizado por defecto en muchas instalaciones de RHEL.

---

# 19. Netavark

```bash
rpm -q netavark
```

Netavark administra las redes de Podman.

---

# 20. Aardvark DNS

```bash
rpm -q aardvark-dns
```

Proporciona resolución DNS para contenedores.

---

# 21. Verificar la instalación

Consultar versión:

```bash
podman --version
```

Salida ejemplo:

```text
podman version 5.x.x
```

---

# 22. Información completa

```bash
podman version
```

Muestra:

- Cliente
- API
- Runtime
- Buildah
- Go Version
- Sistema operativo

---

# 23. Información del entorno

```bash
podman info
```

Este comando es uno de los más importantes.

Muestra información sobre:

- Runtime OCI
- Driver de almacenamiento
- Rootless
- cgroups
- Red
- Kernel
- Distribución
- Arquitectura

---

# 24. Verificar si estamos en modo Rootless

```bash
podman info --format '{{.Host.Security.Rootless}}'
```

Resultado:

```text
true
```

o

```text
false
```

---

# 25. Arquitectura

Consultar:

```bash
podman info --format '{{.Host.Arch}}'
```

Ejemplo:

```text
amd64
```

---

# 26. Sistema operativo

```bash
podman info --format '{{.Host.OS}}'
```

---

# 27. Runtime utilizado

```bash
podman info --format '{{.Host.OCIRuntime.Name}}'
```

Ejemplo:

```text
crun
```

---

# 28. Driver de almacenamiento

```bash
podman info --format '{{.Store.GraphDriverName}}'
```

Generalmente:

```text
overlay
```

---

# 29. Root del almacenamiento

```bash
podman info --format '{{.Store.GraphRoot}}'
```

---

# 30. Directorio temporal

```bash
podman info --format '{{.Store.RunRoot}}'
```

---

# 31. Verificar imágenes

```bash
podman images
```

Al principio probablemente no existan imágenes descargadas.

---

# 32. Verificar contenedores

```bash
podman ps -a
```

La salida inicial será vacía.

---

# 33. Primer diagnóstico

Podemos confirmar que el entorno está listo comprobando:

```bash
podman info
```

```bash
podman version
```

```bash
podman images
```

```bash
podman ps -a
```

---

# 34. Primer contenedor

```bash
podman run --rm docker.io/library/alpine:latest echo "Podman instalado correctamente"
```

Si el mensaje aparece correctamente, la instalación es funcional.

---

# 35. Laboratorio RHCSA

## Ejercicio 1

Identificar la distribución.

```bash
cat /etc/os-release
```

---

## Ejercicio 2

Comprobar el kernel.

```bash
uname -r
```

---

## Ejercicio 3

Comprobar SELinux.

```bash
getenforce
```

---

## Ejercicio 4

Actualizar paquetes.

```bash
sudo dnf update
```

---

## Ejercicio 5

Buscar Podman.

```bash
dnf search podman
```

---

## Ejercicio 6

Instalar Podman.

```bash
sudo dnf install -y podman
```

---

## Ejercicio 7

Verificar la versión.

```bash
podman --version
```

---

## Ejercicio 8

Mostrar información.

```bash
podman info
```

---

## Ejercicio 9

Comprobar el runtime.

```bash
podman info --format '{{.Host.OCIRuntime.Name}}'
```

---

## Ejercicio 10

Ejecutar Alpine.

```bash
podman run --rm docker.io/library/alpine:latest uname -a
```

---

# Buenas prácticas

- Mantener actualizado el sistema.
- Utilizar repositorios oficiales.
- Trabajar en modo Rootless cuando sea posible.
- Mantener SELinux en Enforcing.
- Verificar el runtime utilizado.
- Comprobar el driver OverlayFS.
- Utilizar versiones soportadas de Podman.
- Revisar periódicamente las actualizaciones.

---

# Errores comunes

## Error 1

Instalar Podman desde repositorios no confiables.

---

## Error 2

Deshabilitar SELinux para solucionar problemas.

---

## Error 3

Trabajar siempre como root.

---

## Error 4

No actualizar el sistema antes de instalar.

---

## Error 5

Ignorar la salida de `podman info`.

---

# Resumen

En este capítulo aprendimos a:

- Preparar el sistema para Podman.
- Instalar Podman mediante DNF.
- Identificar los componentes del ecosistema.
- Verificar la instalación.
- Ejecutar el primer contenedor.
- Realizar un diagnóstico básico del entorno.

En la **Fase 2** profundizaremos en la configuración de Podman, estudiando en detalle los archivos:

- `/etc/containers/containers.conf`
- `/etc/containers/registries.conf`
- `/etc/containers/storage.conf`

así como la configuración **Rootless**, **Rootful**, el almacenamiento y la administración de registros de imágenes.