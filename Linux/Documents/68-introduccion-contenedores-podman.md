# 68. Introducción a los Contenedores y Podman

> **Módulo 10: Contenedores con Podman**  
> **Manual de preparación RHCSA**  
> **Archivo:** `68-introduccion-contenedores-podman.md`

---

# Objetivos de aprendizaje

Al finalizar este capítulo podrás:

- Explicar qué es un contenedor.
- Diferenciar un contenedor de una máquina virtual.
- Comprender el propósito de las imágenes de contenedores.
- Identificar los principales componentes de la arquitectura de contenedores.
- Comprender el papel de los estándares OCI.
- Explicar qué es Podman y para qué se utiliza.
- Diferenciar Podman de Docker.
- Comprender el modelo sin daemon de Podman.
- Diferenciar contenedores `rootless` y `rootful`.
- Reconocer la función de `conmon`, `crun`, Buildah y Skopeo.
- Comprender el ciclo de vida básico de un contenedor.
- Ejecutar, detener, iniciar y eliminar contenedores sencillos.
- Consultar imágenes y contenedores locales.
- Inspeccionar información básica del entorno Podman.
- Identificar las principales ubicaciones de configuración y almacenamiento.
- Aplicar buenas prácticas iniciales de seguridad.
- Resolver ejercicios introductorios similares a los encontrados en RHCSA.

---

# Introducción

Los contenedores constituyen una forma de empaquetar y ejecutar aplicaciones junto con las bibliotecas, configuraciones y dependencias que necesitan.

Una aplicación tradicional puede depender de:

- Una versión específica de Python.
- Una versión particular de Java.
- Bibliotecas del sistema.
- Variables de entorno.
- Archivos de configuración.
- Usuarios y grupos.
- Puertos.
- Directorios.
- Permisos.
- Herramientas adicionales.

Cuando estas dependencias se instalan directamente en el sistema operativo, pueden producirse conflictos.

Por ejemplo:

```text
Aplicación A necesita Python 3.9
Aplicación B necesita Python 3.12
Aplicación C requiere una biblioteca incompatible
```

Con contenedores, cada aplicación puede ejecutarse en un entorno aislado que contiene sus propias dependencias.

```text
Servidor Linux
│
├── Contenedor A
│   ├── Aplicación A
│   ├── Bibliotecas A
│   └── Configuración A
│
├── Contenedor B
│   ├── Aplicación B
│   ├── Bibliotecas B
│   └── Configuración B
│
└── Contenedor C
    ├── Aplicación C
    ├── Bibliotecas C
    └── Configuración C
```

Los contenedores no son máquinas virtuales pequeñas.

Aunque proporcionan aislamiento, comparten el kernel del sistema anfitrión.

---

# 1. ¿Qué es un contenedor?

Un contenedor es un proceso o conjunto de procesos aislados que se ejecutan sobre el kernel del sistema anfitrión.

El aislamiento se consigue principalmente mediante tecnologías del kernel Linux como:

- Namespaces.
- Control groups o `cgroups`.
- Capabilities.
- SELinux.
- Seccomp.
- Sistemas de archivos por capas.
- User namespaces.

Desde el punto de vista del sistema operativo, un contenedor sigue siendo un proceso.

Puedes observarlo con comandos como:

```bash
ps aux
```

```bash
pstree
```

```bash
podman top nombre_contenedor
```

La diferencia es que ese proceso dispone de una vista aislada de determinados recursos.

---

# 2. Recursos que pueden aislarse

Un contenedor puede disponer de una vista independiente de:

- Procesos.
- Red.
- Puntos de montaje.
- Nombre del host.
- Usuarios.
- Grupos.
- Comunicación entre procesos.
- Recursos de CPU.
- Memoria.
- Dispositivos.
- Identificadores del sistema.

Diagrama conceptual:

```text
Sistema anfitrión
│
├── Namespace de procesos
│   └── El contenedor ve sus propios PID
│
├── Namespace de red
│   └── Interfaces, direcciones y rutas propias
│
├── Namespace de montaje
│   └── Vista independiente del sistema de archivos
│
├── Namespace UTS
│   └── Hostname propio
│
├── Namespace de usuario
│   └── Mapeo de UID y GID
│
└── Cgroups
    └── Límites y control de recursos
```

---

# 3. Los contenedores son procesos

Supongamos que ejecutas:

```bash
podman run --name servidor-web \
docker.io/library/httpd:latest
```

Podman:

1. Localiza la imagen solicitada.
2. Descarga la imagen si no existe localmente.
3. Prepara el sistema de archivos del contenedor.
4. Configura namespaces.
5. Configura cgroups.
6. Aplica opciones de seguridad.
7. Inicia el proceso principal.
8. Supervisa el contenedor.

El proceso principal del contenedor puede ser:

```text
httpd
```

```text
nginx
```

```text
postgres
```

```text
python
```

```text
bash
```

Cuando el proceso principal termina, el contenedor normalmente deja de ejecutarse.

```text
Proceso principal activo
        │
        ▼
Contenedor ejecutándose

Proceso principal termina
        │
        ▼
Contenedor detenido
```

---

# 4. Contenedor frente a máquina virtual

Una máquina virtual emula un sistema informático completo.

Cada máquina virtual normalmente dispone de:

- Kernel propio.
- Sistema operativo invitado.
- Memoria asignada.
- CPU virtual.
- Disco virtual.
- Servicios del sistema.
- Controladores virtuales.

Un contenedor comparte el kernel del anfitrión.

---

## Arquitectura de una máquina virtual

```text
Hardware físico
      │
      ▼
Sistema operativo anfitrión o hipervisor
      │
      ├── Máquina virtual 1
      │   ├── Kernel
      │   ├── Sistema operativo
      │   ├── Bibliotecas
      │   └── Aplicación
      │
      └── Máquina virtual 2
          ├── Kernel
          ├── Sistema operativo
          ├── Bibliotecas
          └── Aplicación
```

---

## Arquitectura de contenedores

```text
Hardware físico
      │
      ▼
Sistema operativo Linux
      │
      ▼
Kernel compartido
      │
      ├── Contenedor 1
      │   ├── Bibliotecas
      │   └── Aplicación
      │
      └── Contenedor 2
          ├── Bibliotecas
          └── Aplicación
```

---

# 5. Comparación entre máquinas virtuales y contenedores

| Característica | Máquina virtual | Contenedor |
|---|---|---|
| Kernel | Propio | Compartido con el anfitrión |
| Sistema operativo completo | Sí | No necesariamente |
| Tiempo de inicio | Generalmente mayor | Generalmente menor |
| Consumo de recursos | Mayor | Menor |
| Aislamiento | Muy fuerte | Aislamiento a nivel de procesos |
| Tamaño | Gigabytes habitualmente | Megabytes o gigabytes |
| Portabilidad | Imagen de VM | Imagen de contenedor |
| Densidad | Menor | Mayor |
| Administración | Sistema operativo completo | Aplicación y dependencias |
| Casos de uso | Sistemas completos | Aplicaciones y servicios |

---

# 6. ¿Cuándo utilizar una máquina virtual?

Una máquina virtual puede ser apropiada cuando:

- Se necesita otro kernel.
- Se requiere otro sistema operativo.
- Deben ejecutarse aplicaciones incompatibles con el kernel anfitrión.
- Se necesita aislamiento mediante virtualización completa.
- Debe simularse una infraestructura completa.
- Se necesitan características específicas del hardware virtual.
- Se requiere ejecutar Windows sobre un anfitrión Linux.

---

# 7. ¿Cuándo utilizar un contenedor?

Un contenedor puede ser apropiado cuando:

- Se desea desplegar una aplicación rápidamente.
- Se necesitan entornos reproducibles.
- Se desea separar dependencias.
- Se ejecutan microservicios.
- Se requiere automatizar despliegues.
- Se necesita escalar múltiples instancias.
- Se desea reducir el consumo de recursos.
- Se necesita mover una aplicación entre entornos.
- Se desea aislar un servicio sin crear una VM completa.

---

# 8. Qué no es un contenedor

Un contenedor no es:

- Una máquina virtual completa.
- Un mecanismo de seguridad infalible.
- Un sustituto automático de las copias de seguridad.
- Un sistema de alta disponibilidad por sí mismo.
- Una solución que elimina la necesidad de actualizar aplicaciones.
- Un entorno necesariamente persistente.
- Una imagen.
- Un registro de imágenes.
- Un pod.
- Un servicio systemd.

Estos conceptos están relacionados, pero no son equivalentes.

---

# 9. Imagen y contenedor

Una imagen es una plantilla inmutable utilizada para crear contenedores.

Un contenedor es una instancia ejecutable creada a partir de una imagen.

```text
Imagen
  │
  ├── Contenedor 1
  ├── Contenedor 2
  └── Contenedor 3
```

Ejemplo:

```text
Imagen:
docker.io/library/httpd:latest

Contenedores:
web01
web02
web03
```

Todos pueden utilizar la misma imagen, pero cada contenedor tiene:

- Nombre propio.
- Identificador propio.
- Procesos propios.
- Configuración propia.
- Capa de escritura propia.
- Estado propio.
- Opciones de red propias.

---

# 10. Analogía sencilla

Puede entenderse una imagen como una plantilla y un contenedor como una instancia creada a partir de esa plantilla.

```text
Plano de una casa
        │
        ▼
Imagen

Casa construida desde el plano
        │
        ▼
Contenedor
```

Una misma imagen puede crear muchos contenedores.

Eliminar un contenedor no elimina necesariamente la imagen.

Eliminar una imagen no debería realizarse mientras existan contenedores que dependan de ella, salvo que se utilicen opciones forzadas y se comprendan sus consecuencias.

---

# 11. Capas de una imagen

Las imágenes suelen estar compuestas por capas.

Ejemplo conceptual:

```text
┌──────────────────────────────────┐
│ Configuración de la aplicación   │
├──────────────────────────────────┤
│ Aplicación                       │
├──────────────────────────────────┤
│ Bibliotecas adicionales          │
├──────────────────────────────────┤
│ Sistema base                     │
└──────────────────────────────────┘
```

Cada instrucción relevante de un `Containerfile` puede producir una nueva capa.

Ejemplo:

```Dockerfile
FROM registry.access.redhat.com/ubi9/ubi

RUN dnf install -y httpd && dnf clean all

COPY index.html /var/www/html/index.html

CMD ["/usr/sbin/httpd", "-DFOREGROUND"]
```

Representación conceptual:

```text
Capa 4: CMD de inicio
Capa 3: index.html
Capa 2: instalación de httpd
Capa 1: imagen UBI
```

---

# 12. Capa de escritura del contenedor

Las capas de la imagen son tratadas normalmente como capas de solo lectura.

Cuando se crea un contenedor, se agrega una capa de escritura.

```text
Contenedor
┌─────────────────────────────┐
│ Capa de escritura           │
├─────────────────────────────┤
│ Capa de aplicación          │
├─────────────────────────────┤
│ Capa de bibliotecas         │
├─────────────────────────────┤
│ Capa base                   │
└─────────────────────────────┘
```

Los cambios realizados dentro del contenedor se almacenan en su capa de escritura, a menos que se utilice almacenamiento persistente.

Si se elimina el contenedor, su capa de escritura normalmente también se elimina.

Por esta razón, los datos importantes no deben depender exclusivamente de la capa escribible del contenedor.

---

# 13. Datos efímeros y datos persistentes

## Datos efímeros

Son datos asociados al ciclo de vida del contenedor.

Ejemplos:

- Archivos temporales.
- Caché.
- Archivos generados para una ejecución.
- Cambios realizados dentro de la capa del contenedor.

---

## Datos persistentes

Son datos que deben conservarse aunque el contenedor sea eliminado.

Ejemplos:

- Bases de datos.
- Archivos subidos por usuarios.
- Configuraciones externas.
- Registros que deben conservarse.
- Documentos.
- Copias de seguridad.

Para persistencia se utilizan principalmente:

- Bind mounts.
- Volúmenes administrados por Podman.
- Sistemas de almacenamiento externos.
- Almacenamiento de red.

---

# 14. ¿Qué es Podman?

Podman significa conceptualmente **Pod Manager**.

Es un motor de contenedores que permite administrar:

- Imágenes.
- Contenedores.
- Pods.
- Redes.
- Volúmenes.
- Registros.
- Builds.
- Recursos asociados al almacenamiento.
- Contenedores rootless.
- Contenedores rootful.

El comando principal es:

```bash
podman
```

Para consultar ayuda:

```bash
podman --help
```

Para consultar la versión:

```bash
podman --version
```

Para información detallada:

```bash
podman version
```

---

# 15. Características principales de Podman

Podman se caracteriza por:

- No requerir un daemon central permanente para la operación normal.
- Permitir ejecución rootless.
- Permitir ejecución rootful.
- Trabajar con imágenes compatibles con estándares OCI.
- Administrar contenedores y pods.
- Integrarse con systemd.
- Permitir el uso de `Containerfile` y `Dockerfile`.
- Proporcionar una interfaz de línea de comandos similar a Docker.
- Integrarse con herramientas como Buildah y Skopeo.
- Ofrecer una API compatible con determinados flujos de trabajo.
- Aplicar mecanismos de seguridad del sistema Linux.

---

# 16. Arquitectura daemonless

Una característica importante de Podman es que no depende de un daemon central permanente para administrar cada operación.

Arquitectura conceptual de un motor centralizado:

```text
Usuario
   │
   ▼
Cliente
   │
   ▼
Daemon central
   │
   ├── Contenedor 1
   ├── Contenedor 2
   └── Contenedor 3
```

Arquitectura conceptual de Podman:

```text
Usuario
   │
   ▼
Comando podman
   │
   ▼
Runtime OCI
   │
   ▼
Proceso del contenedor
```

Esto no significa que Podman nunca utilice procesos auxiliares.

Significa que su administración básica no depende de un único daemon central que deba estar ejecutándose continuamente para controlar todos los contenedores.

---

# 17. Ventajas del modelo sin daemon central

Entre sus ventajas se encuentran:

- Menor dependencia de un proceso privilegiado central.
- Ejecución directa bajo la identidad del usuario.
- Mejor integración con herramientas tradicionales de Linux.
- Administración mediante systemd.
- Separación de contenedores por usuario.
- Menor impacto de un fallo de un servicio central.
- Facilidad para ejecutar contenedores rootless.
- Modelo de procesos más cercano a la administración Linux convencional.

---

# 18. ¿Qué es un daemon?

Un daemon es un proceso que se ejecuta en segundo plano proporcionando un servicio.

Ejemplos:

```text
sshd
```

```text
crond
```

```text
systemd-journald
```

```text
NetworkManager
```

Podman puede ofrecer un socket o servicio API cuando es necesario, pero su funcionamiento básico por línea de comandos no requiere que un daemon central esté activo permanentemente.

---

# 19. Podman frente a Docker

Podman y Docker comparten muchos conceptos y comandos similares, pero no son idénticos.

| Característica | Podman | Docker |
|---|---|---|
| Modelo tradicional | Sin daemon central obligatorio | Cliente y daemon |
| Rootless | Soportado | Disponible, con arquitectura propia |
| Pods | Soporte directo | No es el concepto principal |
| Integración con systemd | Muy estrecha | Posible |
| OCI | Sí | Sí |
| CLI | Similar a Docker | CLI propia |
| Construcción de imágenes | Podman/Buildah | Docker Build/BuildKit |
| Administración remota | API o cliente remoto | Daemon/API |
| Enfoque RHEL | Herramienta principal | No es la herramienta predeterminada |

No debe asumirse que todos los parámetros o comportamientos son exactamente iguales.

---

# 20. Compatibilidad de comandos

Muchos comandos son similares.

Ejemplo con Podman:

```bash
podman run \
--name web01 \
-d \
-p 8080:80 \
docker.io/library/httpd:latest
```

Ejemplo equivalente en Docker:

```bash
docker run \
--name web01 \
-d \
-p 8080:80 \
docker.io/library/httpd:latest
```

Esta similitud facilita la transición, pero siempre debe consultarse la documentación de la herramienta utilizada.

---

# 21. ¿Qué es OCI?

OCI significa:

```text
Open Container Initiative
```

Es una iniciativa que define estándares abiertos relacionados con contenedores.

Sus principales especificaciones abarcan:

- Formato de imágenes.
- Ejecución de contenedores.
- Distribución de imágenes.

---

# 22. Especificación de imágenes OCI

La especificación de imagen define cómo se representa una imagen.

Incluye conceptos como:

- Manifest.
- Configuración.
- Capas.
- Índices de imágenes.
- Descriptores.
- Identificadores de contenido.
- Tipos de medios.

Diagrama simplificado:

```text
Imagen OCI
│
├── Manifest
│   ├── Referencia a configuración
│   └── Referencias a capas
│
├── Configuración
│   ├── Arquitectura
│   ├── Sistema operativo
│   ├── Variables
│   ├── Comando
│   └── Metadatos
│
└── Capas
    ├── Capa 1
    ├── Capa 2
    └── Capa 3
```

---

# 23. Especificación de runtime OCI

La especificación de runtime establece cómo debe ejecutarse un contenedor a partir de un bundle preparado.

Un runtime OCI se ocupa de tareas como:

- Crear namespaces.
- Configurar cgroups.
- Configurar el sistema de archivos.
- Aplicar capabilities.
- Configurar dispositivos.
- Iniciar el proceso principal.
- Gestionar el estado de ejecución.

Ejemplos de runtimes:

```text
crun
```

```text
runc
```

---

# 24. Especificación de distribución OCI

La especificación de distribución define mecanismos para distribuir contenido de imágenes mediante registros.

Conceptualmente:

```text
Cliente Podman
      │
      ▼
Registro
      │
      ├── Repositorio
      │   ├── Imagen
      │   ├── Etiqueta
      │   ├── Manifest
      │   └── Capas
      │
      └── Autenticación y transferencia
```

---

# 25. Componentes de la arquitectura Podman

Una arquitectura simplificada puede representarse así:

```text
Usuario
   │
   ▼
CLI de Podman
   │
   ├── Gestión de imágenes
   ├── Gestión de contenedores
   ├── Gestión de pods
   ├── Gestión de redes
   └── Gestión de volúmenes
   │
   ▼
Bibliotecas de contenedores
   │
   ├── Almacenamiento
   ├── Imágenes
   ├── Configuración
   └── Seguridad
   │
   ▼
Runtime OCI
   │
   ├── crun
   └── runc
   │
   ▼
Kernel Linux
   ├── namespaces
   ├── cgroups
   ├── SELinux
   ├── seccomp
   └── capabilities
```

---

# 26. Función de `crun`

`crun` es un runtime OCI utilizado para crear y ejecutar contenedores.

Se encarga de interactuar con funciones del kernel para:

- Crear namespaces.
- Aplicar límites.
- Configurar el proceso.
- Preparar el entorno aislado.
- Ejecutar el comando principal del contenedor.

Consultar el runtime configurado:

```bash
podman info
```

Filtrar información:

```bash
podman info \
--format '{{.Host.OCIRuntime.Name}}'
```

La disponibilidad exacta de campos puede variar según la versión.

---

# 27. Función de `runc`

`runc` es otro runtime OCI.

Cumple una función similar:

- Recibe una configuración OCI.
- Prepara el contenedor.
- Aplica aislamiento.
- Ejecuta el proceso principal.

En un sistema puede estar disponible uno o varios runtimes.

La selección depende de:

- Distribución.
- Configuración.
- Versión.
- Paquetes instalados.
- Preferencias administrativas.

---

# 28. Función de `conmon`

`conmon` es un proceso de monitorización utilizado en la ejecución de contenedores.

Entre sus responsabilidades se encuentran:

- Supervisar el proceso del contenedor.
- Gestionar la salida estándar.
- Gestionar la salida de error.
- Registrar el código de salida.
- Ayudar a mantener información de ejecución.
- Facilitar la interacción entre Podman y el proceso del contenedor.

Diagrama conceptual:

```text
Podman
  │
  ▼
conmon
  │
  ▼
Runtime OCI
  │
  ▼
Proceso del contenedor
```

---

# 29. Función de Buildah

Buildah es una herramienta especializada en construir imágenes de contenedores.

Puede utilizarse para:

- Crear imágenes.
- Ejecutar instrucciones de construcción.
- Trabajar con `Containerfile`.
- Modificar sistemas de archivos de imágenes.
- Crear imágenes mediante scripts.
- Construir sin un daemon central.

Podman puede construir imágenes utilizando componentes y bibliotecas del ecosistema de contenedores.

Ejemplo:

```bash
podman build \
-t mi-aplicacion:1.0 \
.
```

También puede utilizarse:

```bash
buildah bud \
-t mi-aplicacion:1.0 \
.
```

---

# 30. Función de Skopeo

Skopeo es una herramienta para trabajar con imágenes y registros sin tener que ejecutar contenedores.

Permite:

- Inspeccionar imágenes remotas.
- Copiar imágenes entre registros.
- Copiar imágenes a almacenamiento local.
- Consultar manifests.
- Eliminar imágenes remotas cuando está permitido.
- Sincronizar contenido.

Ejemplo conceptual:

```bash
skopeo inspect \
docker://docker.io/library/alpine:latest
```

---

# 31. Podman, Buildah y Skopeo

| Herramienta | Función principal |
|---|---|
| Podman | Ejecutar y administrar contenedores, imágenes y pods |
| Buildah | Construir imágenes |
| Skopeo | Inspeccionar y transferir imágenes |
| crun/runc | Ejecutar el contenedor según OCI |
| conmon | Monitorizar el proceso del contenedor |

Diagrama:

```text
                 Imágenes
                    │
       ┌────────────┼────────────┐
       │            │            │
       ▼            ▼            ▼
    Podman       Buildah       Skopeo
       │            │            │
       │            │            └── Copiar e inspeccionar
       │            └── Construir
       └── Ejecutar contenedores
```

---

# 32. ¿Qué es un registro de contenedores?

Un registro es un servicio utilizado para almacenar y distribuir imágenes.

Ejemplos frecuentes:

```text
registry.access.redhat.com
```

```text
registry.redhat.io
```

```text
quay.io
```

```text
docker.io
```

Un registro puede ser:

- Público.
- Privado.
- Corporativo.
- Local.
- Autenticado.
- Anónimo.
- Seguro mediante TLS.
- Configurado para bloquear determinados repositorios.

---

# 33. Estructura de una referencia de imagen

Ejemplo:

```text
docker.io/library/httpd:latest
```

Descomposición:

```text
docker.io
└── Registro

library
└── Namespace u organización

httpd
└── Repositorio o imagen

latest
└── Etiqueta
```

Otro ejemplo:

```text
quay.io/organizacion/aplicacion:2.1
```

---

# 34. Referencias completas y nombres cortos

Referencia completa:

```text
docker.io/library/alpine:latest
```

Nombre corto:

```text
alpine
```

Utilizar referencias completas reduce ambigüedad.

Buena práctica:

```bash
podman pull \
docker.io/library/alpine:latest
```

En lugar de depender únicamente de:

```bash
podman pull alpine
```

Un nombre corto puede requerir resolución mediante la configuración de registros.

---

# 35. Etiquetas de imágenes

Una etiqueta identifica una versión o variante de una imagen.

Ejemplos:

```text
httpd:latest
```

```text
httpd:2.4
```

```text
mi-aplicacion:1.0
```

```text
mi-aplicacion:produccion
```

La etiqueta:

```text
latest
```

no significa necesariamente:

- Más segura.
- Más estable.
- Más nueva en todos los contextos.
- Adecuada para producción.
- Inmutable.

Una etiqueta puede cambiar para apuntar a otro contenido.

Para máxima reproducibilidad pueden utilizarse referencias por digest.

---

# 36. Digest de una imagen

Un digest identifica contenido mediante un hash.

Ejemplo conceptual:

```text
sha256:abc123...
```

Referencia:

```text
registro/organizacion/imagen@sha256:abc123...
```

Ventajas:

- Identificación precisa.
- Reproducibilidad.
- Verificación de contenido.
- Menor ambigüedad que una etiqueta mutable.

---

# 37. Ciclo de vida de una imagen

```text
Buscar
  │
  ▼
Descargar
  │
  ▼
Inspeccionar
  │
  ▼
Crear contenedor
  │
  ▼
Utilizar
  │
  ▼
Actualizar o reemplazar
  │
  ▼
Eliminar cuando ya no sea necesaria
```

Comandos relacionados:

```bash
podman search
```

```bash
podman pull
```

```bash
podman images
```

```bash
podman image inspect
```

```bash
podman rmi
```

---

# 38. Ciclo de vida de un contenedor

```text
Imagen
  │
  ▼
Crear
  │
  ▼
Iniciar
  │
  ▼
Ejecutar
  │
  ├── Detener
  ├── Reiniciar
  ├── Pausar
  └── Inspeccionar
  │
  ▼
Eliminar
```

Comandos:

```bash
podman create
```

```bash
podman start
```

```bash
podman run
```

```bash
podman stop
```

```bash
podman restart
```

```bash
podman rm
```

---

# 39. Diferencia entre `podman create` y `podman run`

## `podman create`

Crea el contenedor, pero no lo inicia.

```bash
podman create \
--name demo \
docker.io/library/alpine:latest
```

Consultar:

```bash
podman ps -a
```

Iniciar:

```bash
podman start demo
```

---

## `podman run`

Crea e inicia el contenedor en una sola operación.

```bash
podman run \
--name demo \
docker.io/library/alpine:latest
```

Flujo:

```text
podman create
      +
podman start
      =
podman run
```

---

# 40. Primer contenedor

Ejecutar:

```bash
podman run \
--rm \
docker.io/library/alpine:latest \
echo "Hola desde Podman"
```

Explicación:

| Elemento | Función |
|---|---|
| `podman` | Ejecuta Podman |
| `run` | Crea e inicia un contenedor |
| `--rm` | Elimina el contenedor cuando termina |
| `docker.io/library/alpine:latest` | Imagen |
| `echo` | Comando dentro del contenedor |
| `"Hola desde Podman"` | Argumento |

---

# 41. Qué ocurre al ejecutar el primer contenedor

```text
podman run
    │
    ▼
¿Existe la imagen localmente?
    │
    ├── Sí
    │   └── Utilizar imagen local
    │
    └── No
        └── Descargar desde el registro
             │
             ▼
        Crear contenedor
             │
             ▼
        Ejecutar proceso
             │
             ▼
        Mostrar resultado
             │
             ▼
        Eliminar con --rm
```

---

# 42. Ejecutar un contenedor interactivo

```bash
podman run \
--rm \
-it \
docker.io/library/alpine:latest \
/bin/sh
```

Opciones:

| Opción | Función |
|---|---|
| `-i` | Mantiene abierta la entrada estándar |
| `-t` | Asigna una terminal |
| `--rm` | Elimina al finalizar |

Dentro:

```sh
hostname
```

```sh
id
```

```sh
ps
```

```sh
cat /etc/os-release
```

Salir:

```sh
exit
```

---

# 43. Observar el aislamiento

Dentro del contenedor:

```sh
hostname
```

```sh
ps aux
```

```sh
ip address
```

```sh
mount
```

```sh
id
```

En el anfitrión, estos comandos muestran una vista diferente.

Esto demuestra que el contenedor puede tener:

- Hostname diferente.
- Procesos visibles diferentes.
- Red diferente.
- Montajes diferentes.
- Identidades mapeadas.

---

# 44. Contenedores en primer plano

Ejemplo:

```bash
podman run \
--name prueba \
docker.io/library/alpine:latest \
sleep 30
```

La terminal permanece vinculada al proceso.

Al terminar `sleep`, el contenedor se detiene.

Consultar:

```bash
podman ps -a
```

---

# 45. Contenedores en segundo plano

Para ejecutar en modo separado:

```bash
podman run \
--name web01 \
-d \
docker.io/library/httpd:latest
```

La opción:

```text
-d
```

significa modo separado o `detached`.

Consultar contenedores activos:

```bash
podman ps
```

Consultar todos:

```bash
podman ps -a
```

---

# 46. Nombres e identificadores

Cada contenedor tiene:

- Un identificador largo.
- Un identificador corto.
- Un nombre.

Ejemplo:

```bash
podman ps -a
```

Salida conceptual:

```text
CONTAINER ID  IMAGE                           COMMAND    STATUS     NAMES
a1b2c3d4e5f6  docker.io/library/httpd:latest  httpd...   Up 1 min   web01
```

Puede administrarse mediante el nombre:

```bash
podman stop web01
```

o mediante el identificador:

```bash
podman stop a1b2c3d4e5f6
```

Es recomendable asignar nombres descriptivos.

---

# 47. Listar contenedores

Solo activos:

```bash
podman ps
```

Todos:

```bash
podman ps -a
```

Mostrar solo identificadores:

```bash
podman ps -aq
```

Mostrar tamaño:

```bash
podman ps -a --size
```

Filtrar por estado:

```bash
podman ps -a \
--filter status=exited
```

Filtrar por nombre:

```bash
podman ps -a \
--filter name=web
```

---

# 48. Estados de un contenedor

Un contenedor puede encontrarse en estados como:

- Created.
- Running.
- Paused.
- Stopped.
- Exited.
- Configured.
- Removing.
- Unknown.

Consultar:

```bash
podman ps -a
```

Para información detallada:

```bash
podman inspect web01
```

---

# 49. Detener un contenedor

```bash
podman stop web01
```

Podman intenta detenerlo de forma ordenada enviando una señal apropiada.

Puede indicarse un tiempo:

```bash
podman stop \
--time 20 \
web01
```

Después del tiempo establecido, puede recurrirse a una terminación forzada.

---

# 50. Iniciar un contenedor detenido

```bash
podman start web01
```

Iniciar y adjuntar salida:

```bash
podman start -a web01
```

Iniciar en modo interactivo cuando corresponde:

```bash
podman start -ai web01
```

---

# 51. Reiniciar un contenedor

```bash
podman restart web01
```

Con tiempo de espera:

```bash
podman restart \
--time 20 \
web01
```

---

# 52. Eliminar un contenedor

Primero detener:

```bash
podman stop web01
```

Después eliminar:

```bash
podman rm web01
```

Forzar:

```bash
podman rm -f web01
```

La opción forzada debe usarse con precaución.

---

# 53. Eliminar contenedores detenidos

```bash
podman container prune
```

Podman solicita confirmación.

Para automatización controlada:

```bash
podman container prune -f
```

Antes de eliminar, revisar:

```bash
podman ps -a
```

---

# 54. Listar imágenes

```bash
podman images
```

También:

```bash
podman image list
```

Salida conceptual:

```text
REPOSITORY                 TAG      IMAGE ID       CREATED      SIZE
docker.io/library/alpine   latest   123456789abc   2 weeks ago  8 MB
```

---

# 55. Descargar una imagen

```bash
podman pull \
docker.io/library/alpine:latest
```

Después:

```bash
podman images
```

---

# 56. Inspeccionar una imagen

```bash
podman image inspect \
docker.io/library/alpine:latest
```

Consultar un campo:

```bash
podman image inspect \
docker.io/library/alpine:latest \
--format '{{.Id}}'
```

Mostrar arquitectura:

```bash
podman image inspect \
docker.io/library/alpine:latest \
--format '{{.Architecture}}'
```

---

# 57. Historial de una imagen

```bash
podman history \
docker.io/library/alpine:latest
```

Este comando ayuda a visualizar:

- Capas.
- Instrucciones de construcción.
- Tamaños.
- Metadatos disponibles.

No siempre permite reconstruir completamente el `Containerfile` original.

---

# 58. Eliminar una imagen

```bash
podman rmi \
docker.io/library/alpine:latest
```

También:

```bash
podman image rm \
docker.io/library/alpine:latest
```

Si existen contenedores asociados, la eliminación puede fallar.

Primero consultar:

```bash
podman ps -a
```

---

# 59. ¿Qué es un pod?

Un pod es un grupo de uno o más contenedores que pueden compartir determinados recursos.

El concepto es similar al utilizado en Kubernetes.

Los contenedores de un pod pueden compartir:

- Namespace de red.
- Puertos.
- Determinados namespaces.
- Ciclo administrativo relacionado.

Diagrama:

```text
Pod
│
├── Contenedor de aplicación
├── Contenedor auxiliar
└── Contenedor de monitorización

Recursos compartidos:
- Red
- Dirección IP
- Puertos
```

---

# 60. Crear un pod

```bash
podman pod create \
--name pod-demo
```

Listar:

```bash
podman pod ps
```

Agregar un contenedor:

```bash
podman run \
-d \
--pod pod-demo \
--name contenedor-demo \
docker.io/library/httpd:latest
```

Consultar:

```bash
podman pod inspect pod-demo
```

---

# 61. El contenedor de infraestructura

Dependiendo de la configuración y del modo de creación, un pod puede utilizar un contenedor de infraestructura para mantener determinados namespaces compartidos.

Consultar:

```bash
podman pod inspect pod-demo
```

Los detalles exactos dependen de la versión y configuración.

---

# 62. Eliminar un pod

Detener:

```bash
podman pod stop pod-demo
```

Eliminar:

```bash
podman pod rm pod-demo
```

Forzar:

```bash
podman pod rm -f pod-demo
```

---

# 63. Contenedores rootful

Un contenedor rootful es administrado mediante privilegios de `root`.

Ejemplo:

```bash
sudo podman run \
--name rootful-demo \
docker.io/library/alpine:latest \
id
```

El almacenamiento, configuración y recursos se encuentran asociados al entorno administrativo de `root`.

---

# 64. Contenedores rootless

Un contenedor rootless es administrado por un usuario sin utilizar privilegios administrativos para la operación normal.

Ejemplo:

```bash
podman run \
--name rootless-demo \
docker.io/library/alpine:latest \
id
```

Ventajas:

- Menor exposición de privilegios.
- Separación por usuario.
- Menor impacto si una aplicación es comprometida.
- No requiere administrar todos los contenedores desde una cuenta privilegiada.
- Facilita servicios de usuario.

---

# 65. Root dentro de un contenedor rootless

Dentro de un contenedor rootless, el proceso puede mostrar:

```text
uid=0(root)
```

Sin embargo, ese UID puede estar mapeado a un UID sin privilegios en el anfitrión.

```text
Contenedor
UID 0
  │
  ▼
Mapeo de user namespace
  │
  ▼
UID no privilegiado en el anfitrión
```

Por lo tanto:

```text
root dentro del contenedor
```

no equivale necesariamente a:

```text
root real en el sistema anfitrión
```

---

# 66. SubUID y SubGID

Los contenedores rootless pueden utilizar rangos subordinados de usuarios y grupos.

Archivos:

```text
/etc/subuid
```

```text
/etc/subgid
```

Consultar:

```bash
grep "^$(id -un):" \
/etc/subuid \
/etc/subgid
```

Salida conceptual:

```text
/etc/subuid/alejandro:100000:65536
/etc/subgid/alejandro:100000:65536
```

Esto representa:

```text
Usuario: alejandro
Inicio del rango: 100000
Cantidad: 65536
```

---

# 67. Diferencias entre rootless y rootful

| Característica | Rootless | Rootful |
|---|---|---|
| Usuario administrador | No requerido normalmente | Requerido |
| Almacenamiento | En el perfil del usuario | Almacenamiento del sistema |
| Puertos privilegiados | Puede tener restricciones | Mayor capacidad |
| Acceso a dispositivos | Limitado | Más amplio |
| Riesgo | Menor privilegio | Mayor impacto potencial |
| Servicios systemd | Servicios de usuario | Servicios del sistema |
| Identidades | User namespace | Identidades del sistema |
| Visibilidad | Contenedores del usuario | Contenedores de root |

---

# 68. Separación entre entornos

Los contenedores rootless de un usuario no aparecen necesariamente al ejecutar Podman como `root`.

Como usuario:

```bash
podman ps -a
```

Como `root`:

```bash
sudo podman ps -a
```

Estos comandos pueden mostrar conjuntos diferentes.

Regla importante:

```text
Podman administra el almacenamiento y los contenedores
del contexto de usuario desde el cual se ejecuta.
```

---

# 69. No mezclar `sudo podman` y `podman`

Ejemplo:

```bash
podman run ...
```

crea un contenedor rootless.

Mientras:

```bash
sudo podman run ...
```

crea un contenedor rootful.

Después, ejecutar:

```bash
podman ps -a
```

no mostrará necesariamente el contenedor creado con:

```bash
sudo podman
```

Este es uno de los errores más frecuentes al comenzar.

---

# 70. Seguridad de contenedores

Los contenedores utilizan varias capas de seguridad.

```text
Aplicación
    │
    ▼
Usuario del contenedor
    │
    ▼
Capabilities
    │
    ▼
Seccomp
    │
    ▼
SELinux
    │
    ▼
Namespaces
    │
    ▼
Cgroups
    │
    ▼
Kernel Linux
```

Ningún mecanismo debe considerarse suficiente por sí solo.

La seguridad se basa en defensa en profundidad.

---

# 71. Namespaces

Los namespaces aíslan la vista de recursos.

Principales tipos:

| Namespace | Aislamiento |
|---|---|
| PID | Procesos |
| NET | Red |
| MNT | Puntos de montaje |
| UTS | Hostname y dominio |
| IPC | Comunicación entre procesos |
| USER | UID y GID |
| CGROUP | Vista de cgroups |
| TIME | Determinados relojes del sistema |

---

# 72. Cgroups

Los control groups permiten organizar, medir y limitar recursos.

Pueden controlar:

- CPU.
- Memoria.
- Procesos.
- Entrada y salida.
- Dispositivos.
- Prioridades.

Ejemplo conceptual:

```bash
podman run \
--memory 512m \
--cpus 1 \
docker.io/library/alpine:latest \
sleep 300
```

Este contenedor tendría límites configurados para memoria y CPU.

La sintaxis y capacidad efectiva dependen del entorno y la versión de cgroups.

---

# 73. Capabilities

Tradicionalmente, `root` dispone de amplios privilegios.

Linux divide parte de esos privilegios en capabilities.

Ejemplos:

```text
CAP_NET_ADMIN
```

```text
CAP_CHOWN
```

```text
CAP_DAC_OVERRIDE
```

```text
CAP_SYS_ADMIN
```

Los contenedores no reciben necesariamente todas las capabilities.

Consultar:

```bash
podman inspect nombre_contenedor
```

Agregar una capability debe hacerse solo cuando sea necesario:

```bash
podman run \
--cap-add CAP_NET_RAW \
imagen
```

Eliminar:

```bash
podman run \
--cap-drop CAP_NET_RAW \
imagen
```

---

# 74. Contenedores privilegiados

La opción:

```text
--privileged
```

reduce muchas restricciones de aislamiento.

Ejemplo:

```bash
podman run \
--privileged \
imagen
```

Debe evitarse salvo que exista una justificación técnica clara.

Un contenedor privilegiado puede obtener acceso considerable a recursos del anfitrión.

Buena práctica:

```text
Conceder solamente los permisos mínimos necesarios.
```

---

# 75. Seccomp

Seccomp permite filtrar llamadas al sistema.

Un perfil puede impedir que un proceso invoque determinadas operaciones del kernel.

Esto ayuda a reducir la superficie de ataque.

No debe desactivarse indiscriminadamente para resolver errores.

---

# 76. SELinux y contenedores

SELinux añade control de acceso obligatorio.

En RHEL, Fedora y sistemas relacionados, SELinux es una capa crítica de seguridad para contenedores.

Consultar:

```bash
getenforce
```

Resultado recomendado:

```text
Enforcing
```

Los procesos y archivos de contenedores reciben contextos específicos.

Consultar procesos:

```bash
ps -eZ \
| grep -E \
'container|conmon'
```

Consultar almacenamiento:

```bash
ls -Zd \
$HOME/.local/share/containers
```

---

# 77. Etiquetas `:z` y `:Z`

Cuando un directorio del anfitrión se monta dentro de un contenedor, SELinux puede impedir el acceso.

Ejemplo:

```bash
podman run \
-v /srv/datos:/datos:Z \
imagen
```

Significado general:

| Opción | Uso |
|---|---|
| `:z` | Contenido compartido por varios contenedores |
| `:Z` | Contenido privado para un contenedor |

No deben agregarse automáticamente sin comprender el efecto sobre los contextos SELinux.

---

# 78. Contenedor y persistencia

Ejemplo sin persistencia:

```bash
podman run \
--name base-datos \
imagen-base-datos
```

Si la aplicación escribe exclusivamente en la capa del contenedor, eliminarlo puede eliminar esos datos.

Ejemplo con volumen:

```bash
podman volume create datos-db
```

```bash
podman run \
--name base-datos \
-v datos-db:/var/lib/datos \
imagen-base-datos
```

---

# 79. Red de contenedores

Un contenedor puede utilizar diferentes modos de red.

Ejemplos conceptuales:

- Red privada.
- Red bridge.
- Red del anfitrión.
- Sin red.
- Red rootless.
- Red personalizada.
- Red de un pod.

Publicar un puerto:

```bash
podman run \
-d \
--name web01 \
-p 8080:80 \
docker.io/library/httpd:latest
```

Interpretación:

```text
Puerto 8080 del anfitrión
            │
            ▼
Puerto 80 del contenedor
```

---

# 80. Probar un servicio web

Crear:

```bash
podman run \
-d \
--name web01 \
-p 8080:80 \
docker.io/library/httpd:latest
```

Comprobar:

```bash
podman ps
```

Probar:

```bash
curl http://127.0.0.1:8080
```

Consultar puertos:

```bash
podman port web01
```

---

# 81. Ver los logs

```bash
podman logs web01
```

Seguir en tiempo real:

```bash
podman logs -f web01
```

Mostrar las últimas líneas:

```bash
podman logs \
--tail 20 \
web01
```

Mostrar marcas de tiempo:

```bash
podman logs -t web01
```

---

# 82. Ejecutar comandos dentro de un contenedor

```bash
podman exec \
web01 \
ps aux
```

Abrir una shell:

```bash
podman exec \
-it \
web01 \
/bin/sh
```

Si existe Bash:

```bash
podman exec \
-it \
web01 \
/bin/bash
```

No todas las imágenes contienen Bash.

---

# 83. Diferencia entre `run` y `exec`

## `podman run`

Crea un contenedor nuevo.

```bash
podman run imagen comando
```

## `podman exec`

Ejecuta un comando en un contenedor que ya está activo.

```bash
podman exec contenedor comando
```

```text
run
└── Contenedor nuevo

exec
└── Contenedor existente
```

---

# 84. Adjuntarse a un contenedor

```bash
podman attach nombre_contenedor
```

Este comando conecta la terminal al proceso principal.

Debe utilizarse con precaución, porque determinadas combinaciones de teclas o señales pueden detener el proceso principal.

Para tareas administrativas suele ser más seguro:

```bash
podman exec -it \
nombre_contenedor \
/bin/sh
```

---

# 85. Copiar archivos

Desde el anfitrión hacia el contenedor:

```bash
podman cp \
archivo.txt \
web01:/tmp/archivo.txt
```

Desde el contenedor hacia el anfitrión:

```bash
podman cp \
web01:/etc/httpd/conf/httpd.conf \
./httpd.conf
```

---

# 86. Inspeccionar un contenedor

```bash
podman inspect web01
```

La salida utiliza formato JSON.

Consultar dirección IP:

```bash
podman inspect \
web01 \
--format '{{.NetworkSettings.IPAddress}}'
```

Consultar estado:

```bash
podman inspect \
web01 \
--format '{{.State.Status}}'
```

Consultar PID:

```bash
podman inspect \
web01 \
--format '{{.State.Pid}}'
```

Los campos disponibles pueden variar según la versión y el modo de red.

---

# 87. Mostrar procesos del contenedor

```bash
podman top web01
```

Mostrar campos específicos:

```bash
podman top \
web01 \
pid user args
```

Comparar con el anfitrión:

```bash
ps aux \
| grep httpd
```

Esto ayuda a comprender que los procesos del contenedor también existen en el anfitrión.

---

# 88. Mostrar uso de recursos

```bash
podman stats
```

Para un contenedor:

```bash
podman stats web01
```

Sin transmisión continua:

```bash
podman stats \
--no-stream \
web01
```

Puede mostrar:

- CPU.
- Memoria.
- Red.
- Entrada y salida.
- PID.

---

# 89. Consultar eventos

```bash
podman events
```

En otra terminal pueden ejecutarse acciones:

```bash
podman start web01
```

```bash
podman stop web01
```

La primera terminal mostrará eventos relacionados.

Filtrar:

```bash
podman events \
--filter container=web01
```

---

# 90. Información del sistema Podman

```bash
podman info
```

Este comando puede mostrar:

- Runtime OCI.
- Versión.
- Almacenamiento.
- Controlador de almacenamiento.
- Cgroups.
- Sistema operativo.
- Arquitectura.
- Configuración rootless.
- Número de imágenes.
- Número de contenedores.
- Registros configurados.
- Componentes de red.

Formato resumido:

```bash
podman info \
--format json
```

---

# 91. Espacio utilizado

```bash
podman system df
```

Información detallada:

```bash
podman system df -v
```

Puede mostrar:

- Imágenes.
- Contenedores.
- Volúmenes.
- Espacio potencialmente recuperable.

---

# 92. Limpieza general

```bash
podman system prune
```

Puede eliminar recursos sin uso.

Antes de ejecutarlo:

```bash
podman ps -a
```

```bash
podman images
```

```bash
podman volume ls
```

La limpieza indiscriminada puede eliminar recursos que aún sean importantes.

---

# 93. Archivos de configuración

Las ubicaciones habituales incluyen:

```text
/etc/containers/
```

Archivos importantes:

```text
/etc/containers/containers.conf
```

```text
/etc/containers/registries.conf
```

```text
/etc/containers/storage.conf
```

También pueden existir fragmentos en directorios terminados en:

```text
.conf.d
```

y configuraciones específicas de usuario.

---

# 94. `containers.conf`

Este archivo puede definir valores predeterminados para:

- Runtime.
- Variables.
- Opciones de contenedores.
- Configuración del motor.
- Redes.
- Límites.
- Comportamiento de Podman.

La configuración efectiva puede combinar:

- Valores internos.
- Configuración del sistema.
- Fragmentos del sistema.
- Configuración del usuario.
- Opciones de la línea de comandos.

Las opciones de línea de comandos suelen tener precedencia para la ejecución específica.

---

# 95. `registries.conf`

Este archivo administra aspectos como:

- Registros de búsqueda.
- Registros bloqueados.
- Mirrors.
- Ubicaciones de registros.
- Resolución de nombres cortos.
- Registros marcados como inseguros.

Consultar:

```bash
cat /etc/containers/registries.conf
```

No debe marcarse un registro como inseguro sin comprender las implicaciones de TLS.

---

# 96. `storage.conf`

Este archivo controla el almacenamiento de contenedores.

Puede definir:

- Driver de almacenamiento.
- Ubicación de datos.
- Ubicación de ejecución.
- Opciones del driver.
- Configuración rootless.
- Parámetros de OverlayFS.

Consultar:

```bash
cat /etc/containers/storage.conf
```

Información efectiva:

```bash
podman info
```

---

# 97. Almacenamiento rootful

El almacenamiento rootful suele encontrarse bajo rutas administradas por el sistema, como:

```text
/var/lib/containers/storage
```

Los datos temporales de ejecución pueden encontrarse bajo:

```text
/run/containers/storage
```

Las rutas exactas deben comprobarse mediante:

```bash
podman info
```

ejecutado en el contexto correspondiente.

---

# 98. Almacenamiento rootless

El almacenamiento rootless suele encontrarse dentro del perfil del usuario, por ejemplo:

```text
$HOME/.local/share/containers/storage
```

Los datos temporales pueden utilizar directorios asociados a la sesión del usuario.

Consultar:

```bash
podman info \
--format '{{.Store.GraphRoot}}'
```

Consultar ubicación de ejecución:

```bash
podman info \
--format '{{.Store.RunRoot}}'
```

---

# 99. El paquete `container-tools`

En sistemas RHEL puede existir un conjunto de herramientas de contenedores que incluye componentes como:

- Podman.
- Buildah.
- Skopeo.
- Herramientas relacionadas.

Consultar paquetes:

```bash
rpm -qa \
| grep -E \
'podman|buildah|skopeo|container'
```

Consultar Podman:

```bash
rpm -q podman
```

---

# 100. Instalar Podman

En sistemas compatibles:

```bash
sudo dnf install -y podman
```

Verificar:

```bash
podman --version
```

```bash
podman info
```

No es necesario iniciar un daemon central para ejecutar el primer contenedor básico.

---

# 101. Ayuda de Podman

Ayuda general:

```bash
podman --help
```

Ayuda de un subcomando:

```bash
podman run --help
```

Manual:

```bash
man podman
```

```bash
man podman-run
```

Buscar:

```bash
apropos podman
```

---

# 102. Estructura de los comandos

Formato general:

```text
podman [opciones-globales] subcomando [opciones] [argumentos]
```

Ejemplo:

```bash
podman run \
--name web01 \
-d \
-p 8080:80 \
docker.io/library/httpd:latest
```

Descomposición:

```text
podman
└── Comando principal

run
└── Subcomando

--name web01
└── Nombre

-d
└── Modo separado

-p 8080:80
└── Publicación de puerto

docker.io/library/httpd:latest
└── Imagen
```

---

# 103. Aliases y compatibilidad

Algunos entornos permiten utilizar comandos similares a Docker mediante paquetes o aliases.

Ejemplo conceptual:

```bash
alias docker=podman
```

Sin embargo, un alias no garantiza compatibilidad absoluta.

Debe verificarse cada flujo de trabajo, especialmente cuando intervienen:

- Compose.
- Sockets.
- APIs.
- Plugins.
- Volúmenes.
- Redes.
- BuildKit.
- Herramientas de terceros.

---

# 104. Contenedores y systemd

Podman puede integrarse con systemd para iniciar aplicaciones de forma persistente.

Dos enfoques importantes son:

- Unidades generadas o administradas.
- Quadlet.

El flujo conceptual es:

```text
systemd
   │
   ▼
Unidad de contenedor
   │
   ▼
Podman
   │
   ▼
Contenedor
   │
   ▼
Aplicación
```

Esto será estudiado en profundidad en el capítulo:

```text
76-contenedores-systemd.md
```

---

# 105. Quadlet

Quadlet permite describir recursos de contenedores mediante archivos que systemd transforma en unidades administrables.

Tipos relacionados pueden incluir archivos como:

```text
.container
```

```text
.volume
```

```text
.network
```

```text
.pod
```

El concepto será desarrollado posteriormente.

---

# 106. Contenedores rootless persistentes

Los servicios de usuario pueden necesitar continuar después de cerrar sesión.

Una herramienta relacionada es:

```bash
loginctl enable-linger usuario
```

Antes de utilizarla debe comprenderse:

- Qué usuario ejecutará el contenedor.
- Qué servicio de usuario se habilitará.
- Qué recursos permanecerán activos.
- Qué implicaciones administrativas tiene.

---

# 107. Nombres, etiquetas y anotaciones

Los recursos pueden incluir metadatos.

Asignar nombre:

```bash
podman run \
--name aplicacion-web \
imagen
```

Asignar etiqueta:

```bash
podman run \
--label entorno=produccion \
--label aplicacion=web \
imagen
```

Filtrar:

```bash
podman ps -a \
--filter label=entorno=produccion
```

Los nombres y etiquetas facilitan:

- Automatización.
- Inventario.
- Diagnóstico.
- Organización.
- Limpieza selectiva.

---

# 108. Variables de entorno

Pasar una variable:

```bash
podman run \
--rm \
-e MENSAJE="Hola" \
docker.io/library/alpine:latest \
sh -c 'echo "$MENSAJE"'
```

Utilizar archivo:

```bash
podman run \
--env-file variables.env \
imagen
```

No deben almacenarse secretos sensibles en archivos sin controles adecuados.

---

# 109. Hostname del contenedor

```bash
podman run \
--rm \
--hostname servidor-interno \
docker.io/library/alpine:latest \
hostname
```

Esto modifica la vista del hostname dentro del contenedor.

---

# 110. Directorio de trabajo

```bash
podman run \
--rm \
--workdir /tmp \
docker.io/library/alpine:latest \
pwd
```

El directorio debe existir o ser preparado según el comportamiento de la imagen y de Podman.

---

# 111. Usuario dentro del contenedor

```bash
podman run \
--rm \
--user 1000 \
docker.io/library/alpine:latest \
id
```

También puede utilizarse un nombre de usuario si existe dentro de la imagen:

```bash
podman run \
--user usuario \
imagen \
id
```

La identidad dentro del contenedor debe distinguirse de la identidad real en el anfitrión.

---

# 112. Comando predeterminado de una imagen

Una imagen puede definir:

- `ENTRYPOINT`.
- `CMD`.

Consultar:

```bash
podman image inspect \
imagen
```

Cuando se agrega un comando al final de `podman run`, puede reemplazarse o complementar el comando predeterminado dependiendo de la configuración de la imagen.

---

# 113. Contenedores de larga y corta duración

## Corta duración

```bash
podman run \
--rm \
docker.io/library/alpine:latest \
date
```

El proceso termina inmediatamente.

---

## Larga duración

```bash
podman run \
-d \
--name espera \
docker.io/library/alpine:latest \
sleep 3600
```

El proceso permanece activo durante una hora.

---

# 114. Por qué un contenedor se detiene inmediatamente

Ejemplo:

```bash
podman run \
--name alpine-demo \
docker.io/library/alpine:latest
```

Puede terminar porque la imagen no mantiene un proceso de larga duración en ese contexto.

Diagnóstico:

```bash
podman ps -a
```

```bash
podman logs alpine-demo
```

```bash
podman inspect alpine-demo
```

```bash
podman inspect \
alpine-demo \
--format '{{.State.ExitCode}}'
```

---

# 115. Código de salida

Los procesos utilizan códigos de salida.

Convención general:

```text
0
└── Operación exitosa

Distinto de 0
└── Error o condición especial
```

Consultar el contenedor:

```bash
podman inspect \
nombre \
--format '{{.State.ExitCode}}'
```

Ejemplo:

```bash
podman run \
--name fallo-demo \
docker.io/library/alpine:latest \
sh -c 'exit 7'
```

Consultar:

```bash
podman inspect \
fallo-demo \
--format '{{.State.ExitCode}}'
```

Resultado esperado:

```text
7
```

---

# 116. Señales

Los contenedores utilizan señales Linux para controlar procesos.

Ejemplos:

```text
SIGTERM
```

```text
SIGKILL
```

```text
SIGHUP
```

Al ejecutar:

```bash
podman stop contenedor
```

se intenta una terminación ordenada.

Al utilizar:

```bash
podman kill contenedor
```

puede enviarse una señal específica.

Ejemplo:

```bash
podman kill \
--signal TERM \
contenedor
```

---

# 117. Estado deseado y estado real

Es importante diferenciar:

```text
Configuración
```

de:

```text
Estado de ejecución
```

Un contenedor puede existir, pero estar detenido.

```bash
podman ps -a
```

Una imagen puede existir sin ningún contenedor.

```bash
podman images
```

Un volumen puede existir sin estar montado por un contenedor activo.

```bash
podman volume ls
```

---

# 118. Contenedores y actualizaciones

Actualizar una imagen no actualiza automáticamente un contenedor existente.

Flujo habitual:

```text
Descargar nueva imagen
        │
        ▼
Detener contenedor anterior
        │
        ▼
Crear contenedor nuevo
        │
        ▼
Conectar almacenamiento persistente
        │
        ▼
Validar
        │
        ▼
Eliminar versión anterior
```

Un contenedor no debe tratarse como un servidor tradicional que siempre se actualiza manualmente desde su interior.

---

# 119. Contenedores inmutables

La filosofía recomendada suele ser:

```text
No modificar manualmente un contenedor en producción.
```

En su lugar:

1. Modificar el `Containerfile`.
2. Construir una nueva imagen.
3. Probarla.
4. Desplegar un contenedor nuevo.
5. Conservar los datos persistentes fuera de la capa escribible.
6. Retirar el contenedor anterior.

---

# 120. Ganado frente a mascotas

Una analogía utilizada en infraestructura distingue entre:

```text
Mascotas
```

y:

```text
Ganado
```

Un servidor tratado como mascota:

- Tiene modificaciones manuales.
- Es difícil de recrear.
- Posee configuración única.
- Se repara continuamente.

Un contenedor tratado como recurso reproducible:

- Se crea desde una definición.
- Puede reemplazarse.
- Conserva los datos fuera de la instancia.
- Utiliza automatización.

La analogía no debe interpretarse de forma literal, sino como un principio de reproducibilidad.

---

# 121. Buenas prácticas iniciales

- Utilizar contenedores rootless cuando sea posible.
- Utilizar referencias completas de imágenes.
- Evitar depender de `latest` en producción.
- Utilizar imágenes de fuentes confiables.
- Revisar la procedencia de las imágenes.
- Mantener imágenes actualizadas.
- Ejecutar un solo servicio principal por contenedor cuando sea razonable.
- Evitar contenedores privilegiados.
- Conceder capabilities mínimas.
- Mantener SELinux en Enforcing.
- Utilizar almacenamiento persistente para datos importantes.
- Definir límites de recursos.
- Utilizar nombres descriptivos.
- Documentar puertos y volúmenes.
- Revisar logs y códigos de salida.
- Eliminar recursos sin uso de manera controlada.
- No incluir secretos directamente en imágenes.
- No guardar contraseñas en el historial del shell.
- Automatizar la construcción de imágenes.
- Probar restauraciones de datos.

---

# 122. Errores comunes

## Error 1: confundir imagen y contenedor

Incorrecto:

```text
La imagen está ejecutándose.
```

Más preciso:

```text
El contenedor creado desde la imagen está ejecutándose.
```

---

## Error 2: mezclar rootless y rootful

```bash
podman ps -a
```

y:

```bash
sudo podman ps -a
```

pueden mostrar recursos distintos.

---

## Error 3: utilizar nombres cortos ambiguos

```bash
podman pull aplicacion
```

Es preferible especificar:

```bash
podman pull \
registro.example.com/equipo/aplicacion:version
```

---

## Error 4: almacenar datos críticos en la capa del contenedor

Eliminar el contenedor puede eliminar esos datos.

---

## Error 5: utilizar `--privileged` para solucionar permisos

Esto puede ocultar la causa y ampliar excesivamente los privilegios.

---

## Error 6: deshabilitar SELinux

Debe diagnosticarse:

```bash
ausearch -m AVC -ts recent
```

y corregirse:

- El contexto.
- El montaje.
- La política.
- La configuración.

---

## Error 7: asumir que `latest` es inmutable

La etiqueta puede cambiar.

---

## Error 8: eliminar recursos sin revisar

```bash
podman system prune -a
```

puede eliminar imágenes necesarias para futuros despliegues.

---

## Error 9: ejecutar una shell y esperar que el contenedor permanezca activo

Si el proceso principal termina, el contenedor se detiene.

---

## Error 10: modificar contenedores manualmente

Los cambios deben incorporarse preferiblemente a la imagen o a una configuración externa reproducible.

---

# 123. Metodología de diagnóstico inicial

Cuando un contenedor no funciona:

```text
1. Confirmar que existe
2. Consultar su estado
3. Revisar código de salida
4. Revisar logs
5. Inspeccionar configuración
6. Verificar imagen
7. Verificar red
8. Verificar almacenamiento
9. Verificar SELinux
10. Verificar permisos
```

---

# 124. Comandos de diagnóstico

```bash
podman ps -a
```

```bash
podman logs nombre
```

```bash
podman inspect nombre
```

```bash
podman top nombre
```

```bash
podman stats --no-stream nombre
```

```bash
podman port nombre
```

```bash
podman events
```

```bash
podman info
```

```bash
journalctl --user
```

Para contenedores rootful administrados por servicios:

```bash
sudo journalctl -b
```

---

# 125. Árbol de diagnóstico básico

```text
¿El contenedor aparece en podman ps -a?
        │
        ├── No
        │   ├── ¿Se creó con otro usuario?
        │   ├── ¿Se utilizó sudo?
        │   └── ¿Falló la creación?
        │
        └── Sí
            │
            ▼
¿Está en ejecución?
        │
        ├── Sí
        │   ├── Revisar puertos
        │   ├── Revisar red
        │   ├── Revisar logs
        │   └── Revisar aplicación
        │
        └── No
            │
            ├── Revisar ExitCode
            ├── Revisar logs
            ├── Revisar comando
            ├── Revisar permisos
            └── Revisar SELinux
```

---

# 126. Tabla de comandos esenciales

| Comando | Función |
|---|---|
| `podman --version` | Mostrar versión |
| `podman info` | Mostrar información del entorno |
| `podman pull` | Descargar una imagen |
| `podman images` | Listar imágenes |
| `podman image inspect` | Inspeccionar una imagen |
| `podman history` | Mostrar historial de capas |
| `podman rmi` | Eliminar una imagen |
| `podman create` | Crear un contenedor |
| `podman run` | Crear e iniciar un contenedor |
| `podman ps` | Listar contenedores activos |
| `podman ps -a` | Listar todos los contenedores |
| `podman start` | Iniciar un contenedor |
| `podman stop` | Detener un contenedor |
| `podman restart` | Reiniciar un contenedor |
| `podman rm` | Eliminar un contenedor |
| `podman logs` | Consultar logs |
| `podman inspect` | Inspeccionar configuración |
| `podman exec` | Ejecutar un comando interno |
| `podman top` | Mostrar procesos |
| `podman stats` | Mostrar consumo de recursos |
| `podman cp` | Copiar archivos |
| `podman port` | Mostrar puertos publicados |
| `podman events` | Mostrar eventos |
| `podman pod create` | Crear un pod |
| `podman pod ps` | Listar pods |
| `podman volume ls` | Listar volúmenes |
| `podman system df` | Mostrar uso de espacio |
| `podman system prune` | Limpiar recursos sin uso |

---

# 127. Laboratorio introductorio

> Ejecuta este laboratorio en una máquina de práctica.  
> No utilices contenedores de producción.

---

## Parte 1: comprobar el sistema

```bash
cat /etc/os-release
```

```bash
uname -r
```

```bash
getenforce
```

```bash
id
```

---

## Parte 2: comprobar Podman

```bash
podman --version
```

```bash
podman info
```

---

## Parte 3: confirmar modo rootless

```bash
podman info \
--format '{{.Host.Security.Rootless}}'
```

También:

```bash
id -u
```

No utilices `sudo` durante las prácticas rootless.

---

## Parte 4: consultar imágenes

```bash
podman images
```

---

## Parte 5: descargar Alpine

```bash
podman pull \
docker.io/library/alpine:latest
```

---

## Parte 6: verificar imagen

```bash
podman images
```

```bash
podman image inspect \
docker.io/library/alpine:latest
```

---

## Parte 7: ejecutar comando sencillo

```bash
podman run \
--rm \
docker.io/library/alpine:latest \
echo "Primer contenedor con Podman"
```

---

## Parte 8: abrir una shell

```bash
podman run \
--rm \
-it \
--name alpine-shell \
docker.io/library/alpine:latest \
/bin/sh
```

Dentro:

```sh
hostname
```

```sh
id
```

```sh
ps
```

```sh
cat /etc/os-release
```

```sh
mount
```

Salir:

```sh
exit
```

---

## Parte 9: crear un contenedor sin iniciarlo

```bash
podman create \
--name espera01 \
docker.io/library/alpine:latest \
sleep 600
```

---

## Parte 10: consultar estado

```bash
podman ps -a
```

Debe aparecer como creado.

---

## Parte 11: iniciar

```bash
podman start espera01
```

---

## Parte 12: comprobar

```bash
podman ps
```

---

## Parte 13: consultar procesos

```bash
podman top espera01
```

---

## Parte 14: inspeccionar

```bash
podman inspect espera01
```

Consultar PID:

```bash
podman inspect \
espera01 \
--format '{{.State.Pid}}'
```

---

## Parte 15: ejecutar un comando interno

```bash
podman exec \
espera01 \
hostname
```

```bash
podman exec \
espera01 \
id
```

---

## Parte 16: consultar estadísticas

```bash
podman stats \
--no-stream \
espera01
```

---

## Parte 17: detener

```bash
podman stop espera01
```

---

## Parte 18: comprobar contenedores detenidos

```bash
podman ps -a
```

---

## Parte 19: iniciar nuevamente

```bash
podman start espera01
```

---

## Parte 20: reiniciar

```bash
podman restart espera01
```

---

## Parte 21: detener y eliminar

```bash
podman stop espera01
```

```bash
podman rm espera01
```

---

## Parte 22: crear servicio web

```bash
podman pull \
docker.io/library/httpd:latest
```

```bash
podman run \
-d \
--name web-lab \
-p 8080:80 \
docker.io/library/httpd:latest
```

---

## Parte 23: comprobar

```bash
podman ps
```

```bash
podman port web-lab
```

---

## Parte 24: probar HTTP

```bash
curl http://127.0.0.1:8080
```

---

## Parte 25: consultar logs

```bash
podman logs web-lab
```

---

## Parte 26: consultar procesos

```bash
podman top web-lab
```

---

## Parte 27: abrir shell

```bash
podman exec \
-it \
web-lab \
/bin/sh
```

Dentro:

```sh
ps aux
```

```sh
hostname
```

```sh
exit
```

---

## Parte 28: copiar archivo al contenedor

Crear:

```bash
cat > index.html <<'EOF'
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Podman RHCSA</title>
</head>
<body>
  <h1>Servidor web ejecutado con Podman</h1>
</body>
</html>
EOF
```

Copiar:

```bash
podman cp \
index.html \
web-lab:/usr/local/apache2/htdocs/index.html
```

La ruta puede variar según la imagen utilizada.

Probar:

```bash
curl http://127.0.0.1:8080
```

---

## Parte 29: observar eventos

En una terminal:

```bash
podman events \
--filter container=web-lab
```

En otra:

```bash
podman restart web-lab
```

---

## Parte 30: detener y eliminar

```bash
podman stop web-lab
```

```bash
podman rm web-lab
```

---

## Parte 31: comprobar imágenes restantes

```bash
podman images
```

---

## Parte 32: consultar espacio

```bash
podman system df
```

```bash
podman system df -v
```

---

## Parte 33: comparar rootless y rootful

Como usuario:

```bash
podman ps -a
```

Como `root`:

```bash
sudo podman ps -a
```

Explica por qué las salidas pueden ser diferentes.

---

## Parte 34: consultar almacenamiento rootless

```bash
podman info \
--format '{{.Store.GraphRoot}}'
```

```bash
podman info \
--format '{{.Store.RunRoot}}'
```

---

## Parte 35: consultar configuración

```bash
ls -l /etc/containers
```

```bash
grep -vE \
'^[[:space:]]*(#|$)' \
/etc/containers/registries.conf
```

---

# 128. Ejercicio de diagnóstico

Crea un contenedor que termine con error:

```bash
podman run \
--name fallo01 \
docker.io/library/alpine:latest \
sh -c 'echo "Error simulado" >&2; exit 12'
```

Consultar:

```bash
podman ps -a
```

```bash
podman logs fallo01
```

```bash
podman inspect \
fallo01 \
--format '{{.State.Status}}'
```

```bash
podman inspect \
fallo01 \
--format '{{.State.ExitCode}}'
```

Resultado esperado:

```text
Estado: exited
Código: 12
```

Eliminar:

```bash
podman rm fallo01
```

---

# 129. Ejercicio de aislamiento

Ejecuta:

```bash
podman run \
--rm \
--hostname contenedor-rhcsa \
docker.io/library/alpine:latest \
sh -c '
echo "Hostname:";
hostname;
echo;
echo "Usuario:";
id;
echo;
echo "Procesos:";
ps;
echo;
echo "Sistema:";
cat /etc/os-release
'
```

Después ejecuta en el anfitrión:

```bash
hostname
```

```bash
id
```

```bash
ps
```

```bash
cat /etc/os-release
```

Documenta las diferencias.

---

# 130. Ejercicio de proceso principal

Ejecuta:

```bash
podman run \
--name proceso-principal \
docker.io/library/alpine:latest \
sleep 20
```

En otra terminal:

```bash
podman ps
```

Después de 20 segundos:

```bash
podman ps
```

```bash
podman ps -a
```

Explica por qué el contenedor dejó de ejecutarse.

---

# 131. Preguntas de repaso

1. ¿Qué es un contenedor?
2. ¿Un contenedor posee necesariamente un kernel propio?
3. ¿Qué kernel utiliza un contenedor Linux?
4. ¿Qué diferencia existe entre una imagen y un contenedor?
5. ¿Qué es la capa de escritura?
6. ¿Qué sucede normalmente con esa capa al eliminar el contenedor?
7. ¿Cómo deben conservarse los datos importantes?
8. ¿Qué son los namespaces?
9. ¿Qué recurso aísla el namespace PID?
10. ¿Qué recurso aísla el namespace NET?
11. ¿Qué función cumplen los cgroups?
12. ¿Qué es una capability?
13. ¿Qué función cumple seccomp?
14. ¿Qué función cumple SELinux?
15. ¿Qué riesgo implica `--privileged`?
16. ¿Qué es Podman?
17. ¿Qué significa que Podman sea daemonless?
18. ¿Podman puede ejecutar contenedores rootless?
19. ¿Qué diferencia existe entre rootless y rootful?
20. ¿Por qué no deben mezclarse `podman` y `sudo podman`?
21. ¿Qué archivos contienen rangos de UID y GID subordinados?
22. ¿Qué significa OCI?
23. ¿Qué especificaciones principales mantiene OCI?
24. ¿Qué es un runtime OCI?
25. ¿Qué función cumple `crun`?
26. ¿Qué función cumple `conmon`?
27. ¿Qué función principal tiene Buildah?
28. ¿Qué función principal tiene Skopeo?
29. ¿Qué es un registro?
30. ¿Qué partes forman una referencia completa de imagen?
31. ¿Qué es una etiqueta?
32. ¿Por qué `latest` no garantiza reproducibilidad?
33. ¿Qué es un digest?
34. ¿Qué hace `podman pull`?
35. ¿Qué hace `podman images`?
36. ¿Qué diferencia existe entre `podman create` y `podman run`?
37. ¿Qué muestra `podman ps`?
38. ¿Qué muestra `podman ps -a`?
39. ¿Qué hace `podman exec`?
40. ¿Qué diferencia existe entre `run` y `exec`?
41. ¿Qué hace `podman logs`?
42. ¿Qué hace `podman inspect`?
43. ¿Qué hace `podman top`?
44. ¿Qué hace `podman stats`?
45. ¿Qué es un pod?
46. ¿Qué recursos pueden compartir los contenedores de un pod?
47. ¿Qué archivo configura registros?
48. ¿Qué archivo configura almacenamiento?
49. ¿Qué archivo proporciona valores predeterminados para contenedores?
50. ¿Qué comando muestra información completa del entorno Podman?

---

# 132. Respuestas breves

## 1. ¿Qué es un contenedor?

Un conjunto de procesos aislados que se ejecutan utilizando el kernel del anfitrión.

---

## 2. Imagen frente a contenedor

Una imagen es una plantilla; un contenedor es una instancia creada a partir de ella.

---

## 3. Namespace

Mecanismo del kernel que proporciona una vista aislada de un recurso.

---

## 4. Cgroup

Mecanismo para organizar, medir y limitar recursos.

---

## 5. Rootless

Ejecución y administración de contenedores por un usuario sin privilegios administrativos para la operación normal.

---

## 6. Podman

Motor de contenedores para administrar imágenes, contenedores, pods, redes y volúmenes.

---

## 7. OCI

Organización que define estándares interoperables para imágenes, runtimes y distribución de contenedores.

---

## 8. `podman run`

Crea e inicia un contenedor.

---

## 9. `podman ps -a`

Muestra contenedores activos y detenidos.

---

## 10. `podman inspect`

Muestra configuración y estado detallados en formato estructurado.

---

# 133. Desafío final

## Escenario

Debes desplegar un servidor web de prueba con estas condiciones:

- Debe ejecutarse como usuario sin privilegios.
- Debe utilizar una referencia completa de imagen.
- Debe llamarse `web-rhcsa`.
- Debe ejecutarse en segundo plano.
- El puerto 8088 del anfitrión debe redirigirse al puerto 80 del contenedor.
- Debes comprobar su funcionamiento.
- Debes consultar sus logs.
- Debes identificar su PID.
- Debes mostrar sus procesos.
- Debes detenerlo.
- Debes iniciarlo nuevamente.
- Debes eliminarlo sin eliminar la imagen.
- Debes documentar todos los comandos.

---

## Solución propuesta

Descargar:

```bash
podman pull \
docker.io/library/httpd:latest
```

Crear e iniciar:

```bash
podman run \
-d \
--name web-rhcsa \
-p 8088:80 \
docker.io/library/httpd:latest
```

Verificar:

```bash
podman ps
```

Comprobar puerto:

```bash
podman port web-rhcsa
```

Probar:

```bash
curl http://127.0.0.1:8088
```

Logs:

```bash
podman logs web-rhcsa
```

PID:

```bash
podman inspect \
web-rhcsa \
--format '{{.State.Pid}}'
```

Procesos:

```bash
podman top web-rhcsa
```

Estadísticas:

```bash
podman stats \
--no-stream \
web-rhcsa
```

Detener:

```bash
podman stop web-rhcsa
```

Comprobar:

```bash
podman ps -a
```

Iniciar:

```bash
podman start web-rhcsa
```

Probar nuevamente:

```bash
curl http://127.0.0.1:8088
```

Eliminar:

```bash
podman stop web-rhcsa
```

```bash
podman rm web-rhcsa
```

Confirmar que la imagen permanece:

```bash
podman images
```

---

# 134. Criterios de evaluación

```text
[ ] Podman está instalado
[ ] El usuario trabaja sin sudo
[ ] La imagen fue descargada correctamente
[ ] Se utilizó una referencia completa
[ ] El contenedor tiene el nombre solicitado
[ ] El contenedor se ejecuta en segundo plano
[ ] El puerto fue publicado correctamente
[ ] curl devuelve contenido
[ ] Los logs pueden consultarse
[ ] El PID fue identificado
[ ] Los procesos fueron mostrados
[ ] El contenedor pudo detenerse
[ ] El contenedor pudo iniciarse nuevamente
[ ] El contenedor fue eliminado
[ ] La imagen permanece disponible
[ ] Los comandos fueron documentados
```

---

# 135. Lista de comprobación del capítulo

```text
[ ] Comprendo qué es un contenedor
[ ] Diferencio contenedores y máquinas virtuales
[ ] Diferencio imágenes y contenedores
[ ] Comprendo el concepto de capas
[ ] Comprendo la capa escribible
[ ] Comprendo la necesidad de persistencia
[ ] Identifico namespaces
[ ] Identifico cgroups
[ ] Comprendo capabilities
[ ] Comprendo la función de SELinux
[ ] Comprendo qué es OCI
[ ] Identifico un runtime OCI
[ ] Comprendo la función de conmon
[ ] Distingo Podman, Buildah y Skopeo
[ ] Comprendo el modelo daemonless
[ ] Diferencio rootless y rootful
[ ] Evito mezclar podman y sudo podman
[ ] Puedo descargar una imagen
[ ] Puedo listar imágenes
[ ] Puedo crear un contenedor
[ ] Puedo iniciar y detener contenedores
[ ] Puedo consultar logs
[ ] Puedo ejecutar comandos internos
[ ] Puedo inspeccionar un contenedor
[ ] Puedo mostrar sus procesos
[ ] Puedo consultar estadísticas
[ ] Puedo publicar un puerto
[ ] Comprendo qué es un pod
[ ] Conozco los archivos principales de configuración
[ ] Puedo eliminar recursos de manera controlada
```

---

# Resumen

Un contenedor es un proceso aislado que utiliza el kernel Linux del sistema anfitrión.

No es una máquina virtual completa.

```text
Máquina virtual
└── Kernel propio

Contenedor
└── Kernel compartido
```

Una imagen es una plantilla inmutable compuesta por capas.

```text
Imagen
  │
  ├── Contenedor A
  ├── Contenedor B
  └── Contenedor C
```

Podman permite administrar:

- Imágenes.
- Contenedores.
- Pods.
- Volúmenes.
- Redes.
- Registros.

Su arquitectura no depende de un daemon central obligatorio para las operaciones normales.

Podman puede ejecutarse:

```text
Rootless
```

o:

```text
Rootful
```

El modo rootless reduce privilegios y es preferible cuando los requisitos técnicos lo permiten.

Los comandos fundamentales de este capítulo son:

```bash
podman info
```

```bash
podman pull
```

```bash
podman images
```

```bash
podman run
```

```bash
podman ps -a
```

```bash
podman start
```

```bash
podman stop
```

```bash
podman logs
```

```bash
podman inspect
```

```bash
podman exec
```

```bash
podman rm
```

La metodología inicial debe recordarse así:

```text
Obtener imagen
      │
      ▼
Crear contenedor
      │
      ▼
Ejecutar proceso
      │
      ▼
Observar estado y logs
      │
      ▼
Detener o reemplazar
      │
      ▼
Eliminar de forma controlada
```

---

# Fin del capítulo
