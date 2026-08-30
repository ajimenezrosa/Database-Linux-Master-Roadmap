# 70. Gestión de Imágenes de Contenedores (Fase 1)

> **Módulo 10 – Contenedores con Podman**
>
> **Manual RHCSA**
>
> **Archivo:** `70-gestion-imagenes-contenedores.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Comprender qué es una imagen de contenedor.
- Diferenciar claramente una imagen de un contenedor.
- Comprender el estándar OCI.
- Entender la estructura interna de una imagen.
- Comprender el funcionamiento de las capas (Layers).
- Entender el mecanismo Copy-on-Write (CoW).
- Comprender qué son los Manifest y Config.
- Identificar Tags y Digests.
- Comprender el funcionamiento de un Registry.
- Buscar y descargar imágenes.
- Administrar las imágenes almacenadas localmente.

---

# Introducción

Todo contenedor nace a partir de una **imagen**.

Una imagen puede compararse con una plantilla o una fotografía del sistema operativo y las aplicaciones que contendrá un futuro contenedor.

Sin imágenes, no existen contenedores.

Podman únicamente ejecuta imágenes previamente descargadas o construidas.

---

# ¿Qué es una imagen?

Una imagen de contenedor es un conjunto de archivos de solo lectura que contienen:

- Sistema operativo base
- Bibliotecas
- Binarios
- Aplicaciones
- Configuración
- Variables de entorno
- Metadatos

Una imagen es **inmutable**.

No cambia mientras esté almacenada.

---

# Analogía

Pensemos en una máquina virtual.

```text
Plantilla VMware
        │
        ▼
 Máquina Virtual
```

En Podman ocurre algo similar.

```text
Imagen
   │
   ▼
Contenedor
```

La imagen es la plantilla.

El contenedor es la instancia en ejecución.

---

# Imagen vs Contenedor

| Imagen | Contenedor |
|---------|------------|
| Plantilla | Instancia |
| Solo lectura | Escritura |
| Inmutable | Cambia constantemente |
| Puede generar muchos contenedores | Se crea desde una sola imagen |
| No consume CPU | Puede consumir CPU |
| No ejecuta procesos | Ejecuta procesos |

---

# Ciclo de vida

```text
Registry

    │

    ▼

Imagen

    │

    ▼

Contenedor

    │

    ▼

Proceso
```

---

# Una imagen NO está ejecutándose

Muchas personas creen que una imagen "está viva".

No.

Una imagen únicamente ocupa espacio en disco.

Ejemplo:

```bash
podman images
```

La salida únicamente muestra archivos almacenados.

No procesos.

---

# Un contenedor sí ejecuta procesos

Ejemplo:

```bash
podman ps
```

Aquí sí veremos procesos activos.

---

# Estándar OCI

OCI significa:

**Open Container Initiative**

Fue creada para estandarizar el formato de las imágenes y los runtimes de contenedores.

Gracias a OCI:

- Docker
- Podman
- Buildah
- CRI-O

pueden compartir imágenes.

---

# Componentes del estándar OCI

OCI define principalmente:

- Image Specification
- Runtime Specification
- Distribution Specification

---

# OCI Image Specification

Define:

- Cómo está construida una imagen.
- Cómo se almacenan las capas.
- Cómo se guardan los metadatos.
- Cómo identificar una imagen.

---

# OCI Runtime Specification

Define cómo ejecutar una imagen.

El runtime más utilizado es:

```text
crun
```

También existe:

```text
runc
```

---

# OCI Distribution Specification

Define:

- Descarga
- Subida
- Registro
- API

Gracias a este estándar cualquier Registry puede intercambiar imágenes.

---

# Anatomía de una imagen

Una imagen está formada por varias partes.

```text
                Imagen OCI

                   │

      ┌────────────┼────────────┐

      ▼            ▼            ▼

 Manifest      Config       Layers
```

---

# Componentes

Una imagen contiene:

- Manifest
- Config
- Layers
- Digest
- Metadatos

---

# Layers

Una imagen está formada por múltiples capas.

Ejemplo:

```text
Layer 5

Layer 4

Layer 3

Layer 2

Layer 1
```

Cada capa representa un cambio.

---

# ¿Qué contiene una Layer?

Puede contener:

- Directorios
- Archivos
- Librerías
- Aplicaciones
- Configuración

---

# Ejemplo

Una imagen Ubuntu puede tener:

```text
Layer 1

Sistema base
```

```text
Layer 2

Actualizaciones
```

```text
Layer 3

Python
```

```text
Layer 4

Apache
```

```text
Layer 5

Aplicación Web
```

---

# Beneficio de las capas

Las capas permiten reutilizar información.

Supongamos dos imágenes.

```text
Imagen A

Layer 1

Layer 2

Layer 3

Layer 4
```

```text
Imagen B

Layer 1

Layer 2

Layer 3

Layer 5
```

Las tres primeras capas se comparten.

No se duplican.

---

# Copy-on-Write (CoW)

Podman utiliza Copy-on-Write.

Significa:

Mientras una capa no cambie:

No se copia.

Solo cuando un archivo se modifica:

Se crea una copia.

---

# Diagrama Copy-on-Write

```text
Imagen

Layer1

Layer2

Layer3

        │

        ▼

Contenedor

Writable Layer
```

Las capas inferiores permanecen intactas.

---

# Writable Layer

Cuando un contenedor inicia:

Se agrega una capa superior.

```text
Writable Layer

──────────────

Layer 5

Layer 4

Layer 3

Layer 2

Layer 1
```

Solo esa capa cambia.

---

# ¿Qué ocurre al eliminar el contenedor?

La capa escribible desaparece.

Las capas originales permanecen.

---

# Manifest

El Manifest describe:

- Layers
- Digest
- Config
- Arquitectura

Es el índice principal de la imagen.

---

# Config

La configuración contiene:

- Variables ENV
- CMD
- ENTRYPOINT
- Usuario
- Directorio de trabajo
- Arquitectura

---

# Digest

Cada imagen posee un identificador único.

Ejemplo:

```text
sha256:
91c95931...
```

El Digest garantiza la integridad.

---

# ¿Por qué es importante?

Si un solo byte cambia:

El Digest cambia completamente.

---

# SHA256

SHA256 es un algoritmo criptográfico.

Ejemplo:

```text
sha256:3f5f...
```

---

# Tag

El Tag es una etiqueta amigable.

Ejemplos:

```text
latest
```

```text
8.10
```

```text
9.5
```

```text
17
```

---

# Ejemplo

```text
docker.io/library/alpine:latest
```

Componentes:

```text
docker.io
```

Registry.

```text
library
```

Repositorio.

```text
alpine
```

Imagen.

```text
latest
```

Tag.

---

# Digest vs Tag

| Tag | Digest |
|------|---------|
| Puede cambiar | Nunca cambia |
| Fácil de recordar | Largo |
| Flexible | Inmutable |
| Uso cotidiano | Uso en producción |

---

# Repositorio

Un repositorio almacena diferentes versiones.

Ejemplo:

```text
postgres

15

16

17
```

Todas pertenecen al mismo repositorio.

---

# Registry

Un Registry almacena múltiples repositorios.

```text
Registry

│

├── postgres

├── nginx

├── alpine

└── redis
```

---

# Registries conocidos

Los más utilizados son:

- docker.io
- quay.io
- registry.redhat.io
- registry.access.redhat.com

---

# Flujo completo

```text
Usuario

    │

podman pull

    │

    ▼

Registry

    │

    ▼

Manifest

    │

    ▼

Layers

    │

    ▼

Disco Local

    │

    ▼

Contenedor
```

---

# Buscar imágenes

```bash
podman search nginx
```

---

# Buscar PostgreSQL

```bash
podman search postgres
```

---

# Buscar Alpine

```bash
podman search alpine
```

---

# Descargar una imagen

```bash
podman pull docker.io/library/alpine
```

---

# Descargar una versión específica

```bash
podman pull docker.io/library/alpine:3.22
```

---

# Descargar usando Digest

```bash
podman pull docker.io/library/alpine@sha256:...
```

---

# Listar imágenes

```bash
podman images
```

Ejemplo:

```text
REPOSITORY     TAG

alpine         latest

httpd          latest

postgres       17
```

---

# Listado detallado

```bash
podman images --digests
```

También mostrará el SHA256.

---

# Filtrar imágenes

```bash
podman images alpine
```

---

# Ordenar resultados

```bash
podman images --sort created
```

---

# Ver solo IDs

```bash
podman images --quiet
```

---

# Inspección inicial

```bash
podman image inspect alpine
```

---

# Arquitectura

Dentro del resultado aparecerán datos como:

```text
amd64

arm64

ppc64le
```

---

# Sistema Operativo

También veremos:

```text
linux
```

---

# Tamaño

Consultar:

```bash
podman images
```

Observar la columna:

```text
SIZE
```

---

# ¿Dónde se almacenan?

Consultar:

```bash
podman info \
--format '{{.Store.GraphRoot}}'
```

Generalmente:

```text
~/.local/share/containers/storage
```

o

```text
/var/lib/containers/storage
```

---

# Visualizar el almacenamiento

```bash
tree -L 2 \
~/.local/share/containers/storage
```

---

# Laboratorio RHCSA

## Laboratorio 1

Buscar Alpine.

```bash
podman search alpine
```

---

## Laboratorio 2

Buscar Ubuntu.

```bash
podman search ubuntu
```

---

## Laboratorio 3

Buscar PostgreSQL.

```bash
podman search postgres
```

---

## Laboratorio 4

Descargar Alpine.

```bash
podman pull docker.io/library/alpine
```

---

## Laboratorio 5

Descargar BusyBox.

```bash
podman pull docker.io/library/busybox
```

---

## Laboratorio 6

Listar imágenes.

```bash
podman images
```

---

## Laboratorio 7

Mostrar Digest.

```bash
podman images --digests
```

---

## Laboratorio 8

Mostrar únicamente IDs.

```bash
podman images --quiet
```

---

## Laboratorio 9

Inspeccionar Alpine.

```bash
podman image inspect alpine
```

---

## Laboratorio 10

Consultar GraphRoot.

```bash
podman info \
--format '{{.Store.GraphRoot}}'
```

---

# Buenas prácticas

- Descargar imágenes desde registros oficiales.
- Utilizar versiones específicas en lugar de `latest` en producción.
- Verificar el **Digest** cuando la integridad sea crítica.
- Eliminar imágenes que ya no se utilicen para ahorrar espacio.
- Revisar periódicamente el almacenamiento con `podman system df`.

---

# Errores comunes

## Error 1

Confundir una imagen con un contenedor.

---

## Error 2

Modificar manualmente el almacenamiento de Podman.

---

## Error 3

Utilizar siempre la etiqueta `latest`.

---

## Error 4

Descargar imágenes desde registros no confiables.

---

## Error 5

Ignorar el Digest al desplegar aplicaciones críticas.

---

# Resumen

En esta primera fase aprendimos:

- Qué es una imagen de contenedor.
- Diferencias entre imagen y contenedor.
- El estándar OCI y sus componentes.
- La anatomía interna de una imagen.
- El funcionamiento de las capas y Copy-on-Write.
- La diferencia entre Tags y Digests.
- Qué es un Registry y un repositorio.
- Cómo buscar, descargar y listar imágenes con Podman.

En la **Fase 2** profundizaremos en la inspección avanzada y administración de imágenes mediante comandos como `podman inspect`, `podman history`, `podman save`, `podman load`, `podman export`, `podman import` y otras herramientas esenciales para la certificación RHCSA.


# 

# 70. Gestión de Imágenes de Contenedores (Fase 2)

> **Módulo 10 – Contenedores con Podman**
>
> **Manual RHCSA**
>
> **Archivo:** `70-gestion-imagenes-contenedores.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase podrás:

- Descargar imágenes desde distintos registros.
- Buscar imágenes oficiales.
- Inspeccionar completamente una imagen.
- Consultar el historial de construcción.
- Verificar la existencia de imágenes.
- Montar una imagen sin ejecutarla.
- Desmontar imágenes.
- Exportar imágenes.
- Importar imágenes.
- Guardar imágenes para respaldo.
- Restaurar imágenes desde archivos.
- Comprender los formatos OCI y Docker Archive.
- Preparar imágenes para ambientes sin conexión (Offline).

---

# Introducción

Una de las tareas más frecuentes de un administrador RHCSA consiste en administrar imágenes.

Las imágenes pueden:

- descargarse
- inspeccionarse
- copiarse
- exportarse
- importarse
- respaldarse
- eliminarse

sin necesidad de ejecutar un solo contenedor.

---

# ¿Qué significa administrar imágenes?

Administrar imágenes implica controlar todo su ciclo de vida.

```text
Registry

    │

    ▼

Search

    │

    ▼

Pull

    │

    ▼

Inspect

    │

    ▼

Save

    │

    ▼

Transfer

    │

    ▼

Load

    │

    ▼

Run
```

---

# 1. Buscar imágenes

El comando:

```bash
podman search
```

consulta los registros configurados.

Ejemplo:

```bash
podman search nginx
```

Resultado:

```text
NAME

docker.io/library/nginx

docker.io/bitnami/nginx

quay.io/nginx
```

---

# Filtrar resultados

Mostrar únicamente imágenes oficiales:

```bash
podman search nginx --filter=is-official
```

---

# Limitar resultados

```bash
podman search nginx --limit 5
```

---

# Mostrar descripción

```bash
podman search postgres
```

La salida incluye:

- nombre
- descripción
- oficial
- automatizada

---

# Buenas prácticas

Siempre preferir:

```text
docker.io/library/
```

o

```text
registry.redhat.io
```

antes que repositorios desconocidos.

---

# 2. Descargar imágenes

El comando principal es:

```bash
podman pull
```

Ejemplo:

```bash
podman pull docker.io/library/httpd
```

---

# Descargar una versión específica

```bash
podman pull docker.io/library/httpd:2.4
```

---

# Descargar PostgreSQL

```bash
podman pull docker.io/library/postgres:17
```

---

# Descargar Ubuntu

```bash
podman pull docker.io/library/ubuntu:24.04
```

---

# Descargar varias imágenes

```bash
podman pull docker.io/library/alpine

podman pull docker.io/library/httpd

podman pull docker.io/library/nginx
```

---

# Pull silencioso

```bash
podman pull --quiet docker.io/library/alpine
```

---

# Descargar utilizando Digest

Ejemplo:

```bash
podman pull \
docker.io/library/alpine@sha256:xxxxxxxx
```

Esta técnica garantiza exactamente la misma imagen.

---

# Ver progreso

Durante la descarga aparecerán mensajes como:

```text
Copying blob...

Copying config...

Writing manifest...
```

Cada blob corresponde a una capa.

---

# 3. Verificar imágenes descargadas

```bash
podman images
```

---

# Mostrar Digest

```bash
podman images --digests
```

---

# Mostrar únicamente IDs

```bash
podman images --quiet
```

---

# Filtrar

```bash
podman images postgres
```

---

# Ordenar

```bash
podman images --sort size
```

---

# Orden descendente

```bash
podman images --sort created
```

---

# 4. Inspeccionar imágenes

Uno de los comandos más importantes:

```bash
podman image inspect alpine
```

---

# Información obtenida

Entre otros datos:

- Digest
- Arquitectura
- SO
- Variables ENV
- Entrypoint
- Labels
- Layers
- Fecha de creación
- Autor
- Tamaño

---

# Inspección simplificada

Arquitectura:

```bash
podman image inspect alpine \
--format '{{.Architecture}}'
```

---

Sistema Operativo:

```bash
podman image inspect alpine \
--format '{{.Os}}'
```

---

Autor

```bash
podman image inspect alpine \
--format '{{.Author}}'
```

---

Tamaño

```bash
podman image inspect alpine \
--format '{{.Size}}'
```

---

Digest

```bash
podman image inspect alpine \
--format '{{.Digest}}'
```

---

# 5. Historial de construcción

Consultar:

```bash
podman history alpine
```

Ejemplo:

```text
IMAGE

CREATED

SIZE

COMMENT
```

---

# ¿Qué representa?

Cada línea corresponde normalmente a una capa.

```text
Layer 5

↓

Layer 4

↓

Layer 3

↓

Layer 2

↓

Layer 1
```

---

# Historial detallado

```bash
podman history --human alpine
```

---

# 6. Verificar existencia

Antes de descargar:

```bash
podman image exists alpine
```

Consultar el código de retorno:

```bash
echo $?
```

Resultado:

```text
0
```

Existe.

```text
1
```

No existe.

Muy útil para scripts.

---

# Ejemplo de script

```bash
if podman image exists alpine
then
    echo "La imagen existe"
else
    podman pull alpine
fi
```

---

# 7. Árbol de dependencias

Consultar:

```bash
podman image tree alpine
```

Ejemplo conceptual:

```text
alpine

│

├── Layer

├── Layer

├── Layer

└── Layer
```

---

# ¿Para qué sirve?

Permite visualizar:

- capas
- dependencias
- reutilización

---

# 8. Montar una imagen

Podman permite montar una imagen sin ejecutarla.

```bash
podman image mount alpine
```

Resultado:

```text
/tmp/.../merged
```

---

# Explorar

```bash
ls
```

```bash
tree
```

```bash
find
```

Todo sin iniciar el contenedor.

---

# Desmontar

```bash
podman image unmount alpine
```

---

# ¿Cuándo es útil?

- Auditorías.
- Investigación.
- Recuperación.
- Análisis forense.
- Validación.

---

# 9. Guardar imágenes

Podemos respaldar una imagen.

```bash
podman save \
-o alpine.tar \
alpine
```

---

# Resultado

```text
alpine.tar
```

Este archivo puede copiarse a otro servidor.

---

# Guardar varias imágenes

```bash
podman save \
-o laboratorio.tar \
alpine \
httpd \
nginx
```

---

# Formato OCI

Guardar:

```bash
podman save \
--format oci-archive \
-o alpine.oci \
alpine
```

---

# Formato Docker

```bash
podman save \
--format docker-archive \
-o alpine.tar \
alpine
```

---

# Diferencias

| OCI | Docker |
|------|---------|
| Estándar OCI | Docker |
| Recomendado | Compatibilidad |
| Portable | Muy común |

---

# 10. Restaurar imágenes

Si recibimos:

```text
alpine.tar
```

Importamos:

```bash
podman load \
-i alpine.tar
```

---

# Verificar

```bash
podman images
```

---

# Escenario Offline

Servidor A

↓

```bash
podman save
```

↓

USB

↓

Servidor B

↓

```bash
podman load
```

No requiere Internet.

---

# 11. Exportar contenedores

No debe confundirse con Save.

Exportar:

```bash
podman export
```

trabaja sobre:

```text
CONTENEDOR
```

Mientras que:

```bash
podman save
```

trabaja sobre:

```text
IMAGEN
```

---

# Diferencias

| Save | Export |
|------|---------|
| Imagen | Contenedor |
| Conserva Layers | No |
| Conserva historial | No |
| Recomendado para respaldo | Sí |

---

# 12. Importar

Después de un export:

```bash
podman import
```

crea una nueva imagen.

---

# Exportar ejemplo

```bash
podman export \
-o servidor.tar \
web
```

---

# Importar ejemplo

```bash
podman import \
servidor.tar \
mi-servidor
```

---

# Comparación general

```text
Imagen

↓

Save

↓

Load

----------------

Contenedor

↓

Export

↓

Import
```

---

# Laboratorio RHCSA

## Laboratorio 1

Buscar Redis.

```bash
podman search redis
```

---

## Laboratorio 2

Descargar Redis.

```bash
podman pull docker.io/library/redis
```

---

## Laboratorio 3

Descargar Nginx.

```bash
podman pull docker.io/library/nginx
```

---

## Laboratorio 4

Mostrar imágenes.

```bash
podman images
```

---

## Laboratorio 5

Mostrar Digest.

```bash
podman images --digests
```

---

## Laboratorio 6

Inspeccionar Redis.

```bash
podman image inspect redis
```

---

## Laboratorio 7

Consultar arquitectura.

```bash
podman image inspect redis \
--format '{{.Architecture}}'
```

---

## Laboratorio 8

Consultar historial.

```bash
podman history redis
```

---

## Laboratorio 9

Verificar existencia.

```bash
podman image exists redis

echo $?
```

---

## Laboratorio 10

Montar la imagen.

```bash
podman image mount redis
```

---

## Laboratorio 11

Explorar archivos.

```bash
tree
```

---

## Laboratorio 12

Desmontar.

```bash
podman image unmount redis
```

---

## Laboratorio 13

Guardar imagen.

```bash
podman save \
-o redis.tar \
redis
```

---

## Laboratorio 14

Eliminar la imagen.

```bash
podman rmi redis
```

---

## Laboratorio 15

Restaurarla.

```bash
podman load \
-i redis.tar
```

---

## Laboratorio 16

Verificar.

```bash
podman images
```

---

# Buenas prácticas

- Descargar únicamente imágenes oficiales.
- Utilizar versiones específicas en producción.
- Respaldar imágenes críticas mediante `podman save`.
- Verificar el Digest antes de desplegar aplicaciones sensibles.
- Utilizar el formato OCI cuando sea posible.
- Verificar el contenido de una imagen mediante `podman image inspect` antes de ejecutarla.

---

# Errores comunes

## Error 1

Confundir `save` con `export`.

---

## Error 2

Confundir `load` con `import`.

---

## Error 3

Modificar manualmente las capas almacenadas.

---

## Error 4

Eliminar una imagen utilizada por contenedores activos.

---

## Error 5

Utilizar siempre la etiqueta `latest` en entornos de producción.

---

# Resumen

En esta segunda fase aprendimos a:

- Buscar imágenes en distintos registros.
- Descargar imágenes y versiones específicas.
- Inspeccionar imágenes en profundidad.
- Analizar su historial de construcción.
- Verificar su existencia mediante scripts.
- Montar imágenes sin ejecutarlas.
- Guardar y restaurar imágenes.
- Comprender las diferencias entre `save`, `load`, `export` e `import`.
- Preparar imágenes para entornos sin acceso a Internet.

En la **Fase 3** estudiaremos la administración avanzada de imágenes: etiquetado (`tag`), eliminación (`rmi`), limpieza (`image prune`), manejo de imágenes huérfanas, versionado, registros privados, mirrors y estrategias de optimización del almacenamiento.

---
# 70. Gestión de Imágenes de Contenedores (Fase 3)

> **Módulo 10 – Contenedores con Podman**
>
> **Manual RHCSA**
>
> **Archivo:** `70-gestion-imagenes-contenedores.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase podrás:

- Administrar etiquetas (Tags).
- Comprender la diferencia entre Tags y Digests.
- Eliminar imágenes correctamente.
- Liberar espacio de almacenamiento.
- Comprender las imágenes huérfanas.
- Administrar múltiples versiones de una imagen.
- Trabajar con registros públicos y privados.
- Configurar mirrors.
- Optimizar el almacenamiento de imágenes.
- Aplicar buenas prácticas utilizadas en producción.

---

# Introducción

Una vez que las imágenes se encuentran almacenadas localmente, el administrador debe mantenerlas organizadas.

Entre las tareas más comunes se encuentran:

- Etiquetar imágenes.
- Renombrarlas.
- Eliminar versiones antiguas.
- Liberar espacio.
- Administrar múltiples versiones.
- Utilizar registros privados.
- Optimizar el almacenamiento.

---

# 1. Etiquetas (Tags)

Una imagen puede tener varias etiquetas.

Ejemplo:

```text
Imagen

SHA256

│

├── latest

├── stable

├── production

└── v1.0
```

Todas apuntan exactamente a la misma imagen.

---

# Consultar etiquetas

```bash
podman images
```

Ejemplo:

```text
REPOSITORY

TAG

IMAGE ID

alpine

latest

91ef...

```

---

# Crear una etiqueta

```bash
podman tag alpine alpine:lab
```

---

# Verificar

```bash
podman images
```

Resultado:

```text
alpine latest

alpine lab
```

Observa que:

```text
IMAGE ID
```

es exactamente el mismo.

---

# Etiquetar para otro repositorio

```bash
podman tag \
alpine \
miempresa/alpine:v1
```

---

# Crear varias etiquetas

```bash
podman tag alpine alpine:test

podman tag alpine alpine:dev

podman tag alpine alpine:qa
```

---

# Visualización

```text
SHA256

│

├── dev

├── test

├── qa

└── latest
```

No existen cuatro imágenes.

Existe una sola imagen.

---

# Buenas prácticas

No utilizar:

```text
latest
```

como única referencia en producción.

Preferir:

```text
17.0

17.1

17.2
```

---

# Versionado Semántico

Es muy utilizado.

Formato:

```text
Mayor.Menor.Parche
```

Ejemplo:

```text
2.3.8
```

---

# Ejemplos

```text
postgres:17.0
```

```text
postgres:17.1
```

```text
postgres:17.2
```

---

# latest

La etiqueta:

```text
latest
```

NO significa:

> "la versión más reciente".

Simplemente significa que alguien publicó una etiqueta llamada:

```text
latest
```

---

# Error frecuente

Muchos administradores creen que:

```bash
podman pull postgres
```

siempre descarga la última versión publicada.

No necesariamente.

Descarga la etiqueta:

```text
latest
```

---

# Digest vs Tag

```text
Digest

↓

Inmutable

Nunca cambia
```

```text
Tag

↓

Puede cambiar
```

---

# Producción

Para ambientes críticos:

Preferir:

```text
sha256:...
```

sobre:

```text
latest
```

---

# 2. Renombrar imágenes

No existe un comando específico.

Se utiliza:

```bash
podman tag
```

seguido de:

```bash
podman rmi
```

---

Ejemplo:

```bash
podman tag alpine laboratorio

podman rmi alpine
```

---

# 3. Eliminar imágenes

Eliminar una imagen:

```bash
podman rmi alpine
```

---

Eliminar utilizando ID

```bash
podman rmi 91ef...
```

---

Eliminar varias

```bash
podman rmi alpine nginx httpd
```

---

Eliminar todas

```bash
podman rmi -a
```

---

Eliminar forzadamente

```bash
podman rmi -f alpine
```

---

# ¿Qué ocurre si un contenedor utiliza la imagen?

Ejemplo:

```text
Error

Image is being used by a container
```

Primero debe eliminarse el contenedor.

---

Consultar

```bash
podman ps -a
```

---

Eliminar el contenedor

```bash
podman rm nombre
```

---

Luego

```bash
podman rmi alpine
```

---

# 4. Imágenes huérfanas

También llamadas:

Dangling Images.

Consultar:

```bash
podman images --filter dangling=true
```

---

¿Qué son?

Son imágenes sin etiqueta.

Ejemplo:

```text
<none>

<none>

sha256...
```

---

Origen

Generalmente aparecen cuando:

- se reconstruye una imagen
- cambia una etiqueta
- finaliza un build

---

# Limpiar imágenes huérfanas

```bash
podman image prune
```

---

Eliminar automáticamente

```bash
podman image prune -f
```

---

¿Qué elimina?

- imágenes sin uso
- capas huérfanas

---

# Eliminar todo lo no utilizado

```bash
podman system prune
```

---

Modo forzado

```bash
podman system prune -f
```

---

Incluyendo volúmenes

```bash
podman system prune -a
```

---

# Precaución

Antes de ejecutar:

```bash
podman system prune
```

verificar:

```bash
podman ps -a
```

---

# 5. Optimización del almacenamiento

Consultar espacio utilizado

```bash
podman system df
```

---

Modo detallado

```bash
podman system df -v
```

---

Ejemplo

```text
Images

Containers

Volumes

Local Storage
```

---

# ¿Qué ocupa más espacio?

Normalmente:

- imágenes grandes
- múltiples versiones
- bases de datos

---

# Compartición de capas

Supongamos:

```text
postgres:17

postgres:17.1

postgres:17.2
```

Comparten gran parte de las capas.

No se duplican completamente.

---

Visualización

```text
Layer 1

Layer 2

Layer 3

↓

Compartidas

↓

Versiones distintas
```

---

# Evitar duplicados

Consultar:

```bash
podman image tree postgres
```

---

# 6. Registros Públicos

Los principales son:

```text
docker.io
```

```text
quay.io
```

```text
registry.redhat.io
```

```text
registry.access.redhat.com
```

---

# Docker Hub

Ejemplo:

```bash
podman pull docker.io/library/nginx
```

---

# Quay

Ejemplo:

```bash
podman pull quay.io/podman/hello
```

---

# Registry Red Hat

Ejemplo:

```bash
podman pull registry.redhat.io/rhel9/httpd-24
```

Requiere autenticación.

---

# Login

```bash
podman login registry.redhat.io
```

---

Ver registros autenticados

```bash
cat ~/.config/containers/auth.json
```

---

Cerrar sesión

```bash
podman logout registry.redhat.io
```

---

# Registros Privados

Muchas empresas poseen:

```text
registry.miempresa.com
```

Las imágenes internas se descargan desde allí.

---

Autenticarse

```bash
podman login registry.miempresa.com
```

---

# Mirrors

Un mirror es una copia local de un registro.

```text
Docker Hub

↓

Mirror Empresa

↓

Servidor
```

---

Ventajas

- mayor velocidad
- menor consumo de Internet
- alta disponibilidad

---

# Short Names

Evitar:

```bash
podman pull nginx
```

Preferir:

```bash
podman pull docker.io/library/nginx
```

---

# Ver autenticaciones

```bash
podman login --get-login registry.redhat.io
```

---

# Cambiar una etiqueta

```bash
podman tag \
postgres:17 \
postgres:produccion
```

---

Eliminar únicamente una etiqueta

```bash
podman rmi postgres:produccion
```

La imagen permanece mientras exista otra etiqueta.

---

# Laboratorio RHCSA

## Laboratorio 1

Descargar PostgreSQL.

```bash
podman pull postgres:17
```

---

## Laboratorio 2

Crear una nueva etiqueta.

```bash
podman tag postgres:17 postgres:lab
```

---

## Laboratorio 3

Listar imágenes.

```bash
podman images
```

---

## Laboratorio 4

Ver Digest.

```bash
podman images --digests
```

---

## Laboratorio 5

Eliminar la etiqueta.

```bash
podman rmi postgres:lab
```

---

## Laboratorio 6

Crear tres etiquetas.

```bash
podman tag postgres:17 postgres:dev

podman tag postgres:17 postgres:test

podman tag postgres:17 postgres:qa
```

---

## Laboratorio 7

Eliminar una.

```bash
podman rmi postgres:test
```

---

## Laboratorio 8

Consultar imágenes huérfanas.

```bash
podman images \
--filter dangling=true
```

---

## Laboratorio 9

Limpiar.

```bash
podman image prune
```

---

## Laboratorio 10

Consultar almacenamiento.

```bash
podman system df
```

---

## Laboratorio 11

Consultar información detallada.

```bash
podman system df -v
```

---

## Laboratorio 12

Autenticarse en un registro.

```bash
podman login registry.redhat.io
```

---

## Laboratorio 13

Consultar autenticación.

```bash
podman login \
--get-login \
registry.redhat.io
```

---

## Laboratorio 14

Cerrar sesión.

```bash
podman logout registry.redhat.io
```

---

## Laboratorio 15

Eliminar todas las imágenes no utilizadas.

```bash
podman system prune
```

---

# Buenas prácticas

- Utilizar etiquetas descriptivas y consistentes.
- Evitar el uso exclusivo de `latest`.
- Limpiar periódicamente imágenes huérfanas.
- Mantener solo las versiones necesarias.
- Utilizar registros privados para aplicaciones internas.
- Verificar el espacio consumido con `podman system df`.
- Autenticarse únicamente en registros confiables.

---

# Errores comunes

## Error 1

Creer que un Tag identifica una imagen de forma única.

---

## Error 2

Eliminar una imagen utilizada por un contenedor.

---

## Error 3

Usar siempre `latest` en producción.

---

## Error 4

No limpiar imágenes huérfanas durante largos períodos.

---

## Error 5

Confundir la eliminación de una etiqueta con la eliminación de la imagen.

---

# Resumen

En esta fase aprendimos a:

- Administrar etiquetas mediante `podman tag`.
- Comprender las diferencias entre **Tag** y **Digest**.
- Eliminar imágenes correctamente con `podman rmi`.
- Limpiar imágenes huérfanas con `podman image prune`.
- Liberar recursos utilizando `podman system prune`.
- Gestionar versiones mediante versionado semántico.
- Autenticarse en registros públicos y privados.
- Comprender el uso de mirrors y registros corporativos.
- Aplicar estrategias para optimizar el almacenamiento de imágenes.

En la **Fase 4** integraremos todos estos conocimientos mediante un laboratorio completo de administración de imágenes, resolución de problemas reales, scripts de auditoría, checklist RHCSA, preguntas de repaso y un desafío final similar al examen oficial.

---

# 70. Gestión de Imágenes de Contenedores (Fase 4)

> **Módulo 10 – Contenedores con Podman**
>
> **Manual RHCSA**
>
> **Archivo:** `70-gestion-imagenes-contenedores.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase podrás:

- Diagnosticar problemas relacionados con imágenes.
- Resolver errores frecuentes durante descargas.
- Validar la integridad de las imágenes.
- Auditar el almacenamiento utilizado por Podman.
- Aplicar buenas prácticas de administración.
- Resolver escenarios similares al examen RHCSA.
- Automatizar verificaciones mediante scripts.
- Prepararte para trabajar en ambientes empresariales.

---

# Metodología de Diagnóstico

Siempre que exista un problema relacionado con una imagen, sigue un procedimiento ordenado.

```text
                Problema

                    │

                    ▼

¿La imagen existe?

        │

        ▼

¿El Tag es correcto?

        │

        ▼

¿Existe conectividad?

        │

        ▼

¿Hay espacio en disco?

        │

        ▼

¿Existe autenticación?

        │

        ▼

¿La imagen está dañada?

        │

        ▼

Resolver
```

Nunca elimines imágenes o directorios de almacenamiento sin haber identificado primero la causa del problema.

---

# Lista rápida de comandos de diagnóstico

```bash
podman images
```

```bash
podman images --digests
```

```bash
podman image inspect imagen
```

```bash
podman history imagen
```

```bash
podman system df
```

```bash
podman image tree imagen
```

```bash
podman image exists imagen
```

```bash
podman info
```

```bash
df -h
```

```bash
df -i
```

---

# Escenario 1
## La imagen no existe

Intentamos ejecutar:

```bash
podman run nginx
```

Error:

```text
Error: image not known
```

Diagnóstico:

```bash
podman images
```

Solución:

```bash
podman pull docker.io/library/nginx
```

---

# Escenario 2
## Tag incorrecto

Intentamos:

```bash
podman pull postgres:99
```

Error:

```text
manifest unknown
```

Diagnóstico:

Buscar las versiones disponibles en la documentación oficial o utilizar un tag válido.

Ejemplo:

```bash
podman pull postgres:17
```

---

# Escenario 3
## Sin acceso a Internet

Error:

```text
connection refused
```

Verificar:

```bash
ping quay.io
```

```bash
curl https://quay.io
```

También comprobar DNS:

```bash
cat /etc/resolv.conf
```

---

# Escenario 4
## Espacio insuficiente

Consultar:

```bash
df -h
```

También:

```bash
podman system df
```

Liberar espacio:

```bash
podman image prune
```

---

# Escenario 5
## Inodos agotados

Consultar:

```bash
df -i
```

Aunque exista espacio libre, un sistema sin inodos disponibles no podrá crear nuevos archivos.

---

# Escenario 6
## Imagen corrupta

Eliminar:

```bash
podman rmi imagen
```

Volver a descargar:

```bash
podman pull imagen
```

---

# Escenario 7
## Error de autenticación

Intentar:

```bash
podman pull registry.redhat.io/rhel9/httpd-24
```

Error:

```text
unauthorized
```

Solución:

```bash
podman login registry.redhat.io
```

---

# Escenario 8
## Digest diferente

Comparar:

```bash
podman images --digests
```

Si el Digest no coincide con el esperado, descargar nuevamente la imagen.

---

# Escenario 9
## Imagen duplicada

Consultar:

```bash
podman images
```

Puede existir la misma imagen con distintos Tags.

Verificar el IMAGE ID.

---

# Escenario 10
## Demasiadas versiones

Consultar:

```bash
podman images
```

Eliminar versiones antiguas:

```bash
podman rmi imagen:tag
```

---

# Escenario 11
## Imágenes huérfanas

Consultar:

```bash
podman images --filter dangling=true
```

Eliminar:

```bash
podman image prune
```

---

# Escenario 12
## Almacenamiento excesivo

Consultar:

```bash
podman system df -v
```

Analizar:

- imágenes grandes
- imágenes repetidas
- imágenes sin uso

---

# Script de Auditoría

Guardar como:

```text
audit_images.sh
```

```bash
#!/bin/bash

echo "=============================="
echo " AUDITORÍA DE IMÁGENES PODMAN "
echo "=============================="

echo
echo "VERSIÓN"
podman --version

echo
echo "IMÁGENES"
podman images

echo
echo "DIGESTS"
podman images --digests

echo
echo "ESPACIO"
podman system df

echo
echo "ALMACENAMIENTO"
df -h

echo
echo "INODOS"
df -i

echo
echo "GRAPHROOT"
podman info --format '{{.Store.GraphRoot}}'

echo
echo "RUNTIME"
podman info --format '{{.Host.OCIRuntime.Name}}'
```

Permisos:

```bash
chmod +x audit_images.sh
```

---

# Laboratorio RHCSA

## Laboratorio 1

Buscar imágenes oficiales de Redis.

```bash
podman search redis
```

---

## Laboratorio 2

Descargar Redis.

```bash
podman pull docker.io/library/redis
```

---

## Laboratorio 3

Descargar PostgreSQL.

```bash
podman pull docker.io/library/postgres:17
```

---

## Laboratorio 4

Mostrar imágenes.

```bash
podman images
```

---

## Laboratorio 5

Mostrar Digests.

```bash
podman images --digests
```

---

## Laboratorio 6

Consultar el historial.

```bash
podman history postgres
```

---

## Laboratorio 7

Inspeccionar la imagen.

```bash
podman image inspect postgres
```

---

## Laboratorio 8

Verificar existencia.

```bash
podman image exists postgres

echo $?
```

---

## Laboratorio 9

Crear una etiqueta.

```bash
podman tag postgres:17 postgres:produccion
```

---

## Laboratorio 10

Crear otra etiqueta.

```bash
podman tag postgres:17 postgres:desarrollo
```

---

## Laboratorio 11

Eliminar una etiqueta.

```bash
podman rmi postgres:desarrollo
```

---

## Laboratorio 12

Guardar la imagen.

```bash
podman save \
-o postgres17.tar \
postgres:17
```

---

## Laboratorio 13

Eliminar la imagen.

```bash
podman rmi postgres:17
```

---

## Laboratorio 14

Restaurarla.

```bash
podman load \
-i postgres17.tar
```

---

## Laboratorio 15

Consultar el almacenamiento.

```bash
podman system df
```

---

## Laboratorio 16

Consultar el espacio libre.

```bash
df -h
```

---

## Laboratorio 17

Consultar inodos.

```bash
df -i
```

---

## Laboratorio 18

Consultar imágenes huérfanas.

```bash
podman images --filter dangling=true
```

---

## Laboratorio 19

Eliminar imágenes huérfanas.

```bash
podman image prune
```

---

## Laboratorio 20

Ejecutar la auditoría.

```bash
./audit_images.sh
```

---

# Checklist RHCSA

```text
□ Podman instalado.

□ Registro oficial utilizado.

□ Imagen descargada correctamente.

□ Digest validado.

□ Tag identificado.

□ Historial consultado.

□ Imagen inspeccionada.

□ GraphRoot validado.

□ Espacio libre suficiente.

□ Inodos disponibles.

□ Imágenes huérfanas eliminadas.

□ Respaldo realizado mediante podman save.

□ Restauración validada mediante podman load.

□ Script de auditoría probado.

□ Almacenamiento optimizado.
```

---

# Preguntas de Repaso

1. ¿Qué es una imagen OCI?
2. ¿Cuál es la diferencia entre una imagen y un contenedor?
3. ¿Qué hace `podman pull`?
4. ¿Qué información muestra `podman image inspect`?
5. ¿Qué representa un Digest?
6. ¿Cuál es la diferencia entre un Tag y un Digest?
7. ¿Qué hace `podman history`?
8. ¿Para qué sirve `podman image exists`?
9. ¿Qué hace `podman save`?
10. ¿Qué hace `podman load`?
11. ¿Cuál es la diferencia entre `save/load` y `export/import`?
12. ¿Qué comando elimina imágenes no utilizadas?
13. ¿Qué comando elimina imágenes huérfanas?
14. ¿Qué hace `podman system df`?
15. ¿Qué es una Dangling Image?
16. ¿Qué ventaja ofrece el uso de Digests en producción?
17. ¿Qué comando muestra únicamente los Digests?
18. ¿Qué registros públicos son los más utilizados?
19. ¿Qué utilidad tiene un Mirror?
20. ¿Cuál es el primer comando que ejecutarías si sospechas un problema con una imagen?

---

# Respuestas

1. Una imagen compatible con el estándar OCI utilizada para crear contenedores.
2. La imagen es una plantilla inmutable; el contenedor es una instancia ejecutable con una capa de escritura.
3. Descarga una imagen desde un registro.
4. Muestra metadatos completos como arquitectura, sistema operativo, variables de entorno, etiquetas, capas y digest.
5. Un identificador criptográfico único basado en SHA-256.
6. El Tag puede cambiar; el Digest es inmutable.
7. Muestra el historial de creación de la imagen y sus capas.
8. Verifica si una imagen existe localmente.
9. Guarda una imagen en un archivo para respaldo o transporte.
10. Restaura una imagen previamente guardada.
11. `save/load` trabajan con imágenes; `export/import` trabajan con contenedores.
12. `podman system prune`
13. `podman image prune`
14. Muestra el uso del almacenamiento de imágenes, contenedores y volúmenes.
15. Una imagen sin etiquetas que ya no está referenciada.
16. Garantiza que se utilice exactamente la misma imagen.
17. `podman images --digests`
18. Docker Hub, Quay.io, registry.redhat.io y registry.access.redhat.com.
19. Reducir tráfico, acelerar descargas y mejorar disponibilidad.
20. `podman images` (seguido de `podman image inspect` si la imagen existe).

---

# Desafío Final RHCSA

Se entrega un servidor Fedora con Podman instalado.

Debes realizar las siguientes tareas:

1. Buscar una imagen oficial de PostgreSQL.
2. Descargar la versión **17**.
3. Verificar el **Digest**.
4. Inspeccionar completamente la imagen.
5. Mostrar el historial de construcción.
6. Crear dos etiquetas adicionales (`qa` y `produccion`).
7. Guardar la imagen en un archivo `postgres17.tar`.
8. Eliminar una de las etiquetas sin afectar la imagen.
9. Consultar el espacio utilizado por las imágenes.
10. Identificar imágenes huérfanas.
11. Limpiar el almacenamiento.
12. Restaurar la imagen desde el archivo generado.
13. Ejecutar el script `audit_images.sh`.
14. Documentar todos los comandos ejecutados y explicar el propósito de cada uno.

---

# Buenas prácticas

- Utilizar siempre imágenes oficiales o verificadas.
- Fijar versiones específicas en producción en lugar de depender de `latest`.
- Validar el **Digest** en despliegues críticos.
- Mantener un inventario de imágenes utilizadas.
- Eliminar periódicamente imágenes y capas sin uso.
- Respaldar imágenes importantes antes de realizar tareas de mantenimiento.
- Supervisar regularmente el consumo de almacenamiento con `podman system df`.

---

# Resumen del Capítulo 70

En este capítulo aprendimos a:

- Comprender la estructura y funcionamiento de las imágenes OCI.
- Buscar, descargar e inspeccionar imágenes.
- Administrar etiquetas y Digests.
- Exportar, importar, guardar y restaurar imágenes.
- Optimizar el almacenamiento y eliminar imágenes innecesarias.
- Trabajar con registros públicos y privados.
- Diagnosticar problemas comunes relacionados con imágenes.
- Automatizar auditorías mediante scripts.
- Aplicar procedimientos y buenas prácticas alineados con los objetivos del examen **RHCSA**.

---

# Fin del capítulo

