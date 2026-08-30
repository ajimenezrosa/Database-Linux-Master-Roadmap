# 73. Almacenamiento y Volúmenes en Podman (Fase 1)

> **Módulo 10 – Contenedores con Podman**
>
> **Manual RHCSA**
>
> **Archivo:** `73-almacenamiento-volumenes-podman.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase podrás:

- Comprender cómo almacena información un contenedor.
- Diferenciar entre almacenamiento efímero y persistente.
- Comprender la arquitectura Copy-on-Write (CoW).
- Entender el papel del Storage Driver.
- Conocer OverlayFS.
- Comprender qué son los volúmenes.
- Diferenciar Bind Mounts y Named Volumes.
- Prepararte para administrar almacenamiento en Podman como exige RHCSA.

---

# Introducción

Uno de los errores más comunes al comenzar a trabajar con contenedores consiste en asumir que los datos almacenados dentro del contenedor permanecerán para siempre.

Esto es falso.

Los contenedores fueron diseñados para ser **efímeros**.

Si un contenedor es eliminado, normalmente también desaparecerán todos los datos almacenados en su sistema de archivos interno, salvo que se hayan utilizado mecanismos de persistencia.

Por esta razón, comprender el almacenamiento es una habilidad esencial para cualquier administrador Linux.

---

# ¿Cómo almacena datos un contenedor?

Un contenedor utiliza varias capas de almacenamiento.

```text
             Aplicación

                  │

                  ▼

          Capa Escritura
          (Writable Layer)

                  │

                  ▼

           Imagen OCI
       (Capas de solo lectura)

                  │

                  ▼

          Storage Driver

                  │

                  ▼

            Sistema Linux
```

---

# Componentes

Todo contenedor está formado por:

- Imagen
- Capas de solo lectura
- Capa de escritura
- Storage Driver
- Sistema de archivos del Host

---

# ¿Qué ocurre cuando modificamos un archivo?

Supongamos que la imagen contiene:

```text
/etc/nginx/nginx.conf
```

Cuando el contenedor modifica dicho archivo, la imagen original NO cambia.

En su lugar ocurre lo siguiente:

```text
Imagen

↓

Archivo Original

↓

Copy-On-Write

↓

Nueva copia modificada
```

---

# Copy-On-Write (CoW)

Podman utiliza el mecanismo denominado:

```text
Copy-On-Write
```

También conocido como:

```text
CoW
```

---

# ¿Qué significa?

Solo se copia un archivo cuando necesita modificarse.

Mientras permanezca sin cambios, todas las capas comparten el mismo archivo.

---

# Ventajas

- Menor consumo de disco.
- Mayor velocidad.
- Reutilización de capas.
- Menor tiempo de descarga.
- Mejor rendimiento.

---

# Ejemplo

Imagen:

```text
50 MB
```

Diez contenedores creados desde ella:

```text
NO ocupan

500 MB
```

Todos comparten las mismas capas.

Únicamente aumenta el tamaño de la capa de escritura cuando existen modificaciones.

---

# Arquitectura Copy-On-Write

```text
             Imagen Base

                  │

      ┌───────────┼───────────┐

      ▼           ▼           ▼

 Container1   Container2   Container3

      │           │           │

 Writable   Writable   Writable
   Layer       Layer      Layer
```

---

# Capa de escritura (Writable Layer)

Cada contenedor posee una única capa de escritura.

Todo cambio ocurre allí.

Ejemplos:

- Crear archivos
- Eliminar archivos
- Modificar configuraciones
- Instalar paquetes
- Generar logs

---

# ¿Qué ocurre al eliminar el contenedor?

La capa de escritura desaparece.

```text
Contenedor

↓

Writable Layer

↓

Eliminación

↓

Datos Perdidos
```

---

# Ejemplo práctico

Creamos un archivo.

```bash
podman exec web touch /tmp/prueba.txt
```

Verificamos.

```bash
podman exec web ls /tmp
```

Ahora eliminamos el contenedor.

```bash
podman rm -f web
```

Creamos nuevamente otro contenedor.

```bash
podman run -d --name web nginx
```

El archivo:

```text
/tmp/prueba.txt
```

ya no existe.

---

# Conclusión

Los datos almacenados únicamente dentro del contenedor son temporales.

---

# Storage Driver

El Storage Driver administra:

- imágenes
- capas
- contenedores
- almacenamiento

---

# Consultar información

```bash
podman info
```

Buscar:

```text
store
```

---

# Consultar Driver

```bash
podman info --format '{{.Store.GraphDriverName}}'
```

Ejemplo:

```text
overlay
```

---

# OverlayFS

El Storage Driver más utilizado en Linux es:

```text
OverlayFS
```

---

# ¿Qué es OverlayFS?

Es un sistema de archivos por capas.

Permite unir múltiples directorios en uno solo.

---

# Arquitectura

```text
            Upper Layer

                │

                ▼

            Lower Layer

                │

                ▼

        Sistema de Archivos
```

---

# Upper Layer

Contiene:

- Cambios
- Archivos nuevos
- Eliminaciones
- Modificaciones

---

# Lower Layer

Contiene:

- Imagen
- Archivos originales
- Solo lectura

---

# Vista completa

```text
               OverlayFS

          Upper (RW)

                ▲

                │

         Lower (RO)

                │

                ▼

            Imagen OCI
```

---

# Directorio de almacenamiento

En instalaciones Rootful normalmente encontramos:

```text
/var/lib/containers/storage
```

---

# Consultarlo

```bash
podman info \
--format '{{.Store.GraphRoot}}'
```

---

# Rootless

Para usuarios normales:

```text
$HOME/.local/share/containers/storage
```

---

# Consultarlo

```bash
podman info \
--format '{{.Store.GraphRoot}}'
```

---

# Espacio utilizado

Consultar:

```bash
podman system df
```

Ejemplo:

```text
TYPE

Images

Containers

Volumes
```

---

# Limpiar almacenamiento

```bash
podman system prune
```

---

# Eliminar también imágenes

```bash
podman system prune -a
```

---

# ¿Qué es un Volumen?

Un volumen es un espacio de almacenamiento persistente administrado por Podman.

Los datos permanecen incluso si el contenedor es eliminado.

---

# Arquitectura

```text
          Container

               │

               ▼

           Volumen

               │

               ▼

        Disco del Host
```

---

# Beneficios

- Persistencia.
- Independencia del contenedor.
- Fácil respaldo.
- Fácil migración.
- Compartición entre contenedores.

---

# Diferencia

## Sin volumen

```text
Container

↓

Datos

↓

Eliminar

↓

Datos Perdidos
```

---

## Con volumen

```text
Container

↓

Volumen

↓

Eliminar Container

↓

Datos Conservados
```

---

# Tipos de almacenamiento persistente

Los más utilizados son:

| Tipo | Administración |
|--------|----------------|
| Named Volume | Podman |
| Bind Mount | Administrador |
| tmpfs | Memoria RAM |

---

# Named Volume

Es administrado completamente por Podman.

Ejemplo:

```bash
podman volume create datos
```

---

# Arquitectura

```text
Container

↓

Named Volume

↓

Podman

↓

Disco
```

---

# Bind Mount

Consiste en utilizar un directorio existente del Host.

Ejemplo:

```text
/opt/datos
```

↓

```text
Contenedor
```

---

# Arquitectura

```text
Host

/opt/datos

      │

      ▼

Container
```

---

# ¿Cuál utilizar?

| Escenario | Recomendación |
|------------|---------------|
| Bases de Datos | Named Volume |
| Desarrollo | Bind Mount |
| Configuración | Bind Mount |
| Datos persistentes | Named Volume |
| Compartir archivos del Host | Bind Mount |

---

# tmpfs

Almacena la información únicamente en memoria RAM.

Ejemplo:

```bash
podman run \
--tmpfs /tmp \
nginx
```

---

# Características

- Muy rápido.
- No utiliza disco.
- Se pierde al detener el contenedor.
- Ideal para archivos temporales.

---

# Comparación general

| Característica | Writable Layer | Named Volume | Bind Mount | tmpfs |
|----------------|---------------|--------------|------------|--------|
| Persistente | No | Sí | Sí | No |
| Compartible | No | Sí | Sí | No |
| Administrado por Podman | No | Sí | No | Sí |
| Usa disco | Sí | Sí | Sí | No |

---

# Laboratorio RHCSA

## Laboratorio 1

Consultar el Driver.

```bash
podman info \
--format '{{.Store.GraphDriverName}}'
```

---

## Laboratorio 2

Consultar GraphRoot.

```bash
podman info \
--format '{{.Store.GraphRoot}}'
```

---

## Laboratorio 3

Consultar almacenamiento.

```bash
podman system df
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

Crear un archivo.

```bash
podman exec web touch /tmp/prueba.txt
```

---

## Laboratorio 6

Eliminar el contenedor.

```bash
podman rm -f web
```

---

## Laboratorio 7

Crear nuevamente el contenedor.

```bash
podman run \
-d \
--name web \
nginx
```

Verificar que el archivo desapareció.

---

## Laboratorio 8

Crear un volumen.

```bash
podman volume create datos
```

---

## Laboratorio 9

Consultar el espacio utilizado.

```bash
podman system df
```

---

## Laboratorio 10

Consultar información general.

```bash
podman info
```

---

# Buenas prácticas

- Nunca almacenar información crítica únicamente en la capa de escritura del contenedor.
- Utilizar volúmenes persistentes para bases de datos y aplicaciones con estado.
- Mantener separado el almacenamiento de los datos y el de la aplicación.
- Supervisar periódicamente el espacio utilizado con `podman system df`.
- Limpiar imágenes y contenedores que ya no sean necesarios.

---

# Errores comunes

## Error 1

Pensar que los datos permanecen después de eliminar un contenedor.

---

## Error 2

Guardar bases de datos dentro de la Writable Layer.

---

## Error 3

Eliminar contenedores sin respaldar los datos.

---

## Error 4

Confundir un Named Volume con un Bind Mount.

---

## Error 5

Ejecutar `podman system prune -a` sin revisar previamente qué recursos serán eliminados.

---

# Resumen

En esta primera fase aprendimos:

- Cómo almacena información un contenedor.
- El funcionamiento del mecanismo Copy-on-Write.
- El papel del Storage Driver y OverlayFS.
- La diferencia entre almacenamiento temporal y persistente.
- Qué son los Named Volumes, Bind Mounts y tmpfs.
- Cuándo utilizar cada mecanismo de almacenamiento.

En la **Fase 2** aprenderemos a crear, administrar e inspeccionar volúmenes, montar directorios del Host, utilizar la opción `-v` y `--mount`, comprender las etiquetas SELinux (`:Z` y `:z`), compartir almacenamiento entre múltiples contenedores y realizar laboratorios avanzados similares al examen RHCSA.

---

# 73. Almacenamiento y Volúmenes en Podman (Fase 2)

> **Módulo 10 – Contenedores con Podman**
>
> **Manual RHCSA**
>
> **Archivo:** `73-almacenamiento-volumenes-podman.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase podrás:

- Crear y administrar volúmenes.
- Inspeccionar volúmenes existentes.
- Compartir datos entre múltiples contenedores.
- Utilizar Bind Mounts.
- Comprender la diferencia entre `-v` y `--mount`.
- Administrar permisos y propietarios.
- Comprender la integración con SELinux.
- Utilizar las opciones `:Z` y `:z`.
- Aplicar buenas prácticas de almacenamiento persistente.

---

# Introducción

En la fase anterior aprendimos que la **Writable Layer** desaparece cuando el contenedor es eliminado.

Ahora veremos cómo mantener la información de forma permanente mediante volúmenes y montajes.

Estos conceptos son utilizados diariamente en:

- PostgreSQL
- MariaDB
- MongoDB
- Redis
- Apache
- Nginx
- Servidores de aplicaciones

---

# Crear un volumen

Sintaxis

```bash
podman volume create NOMBRE
```

Ejemplo

```bash
podman volume create postgres_data
```

Resultado

```text
postgres_data
```

---

# Crear múltiples volúmenes

```bash
podman volume create web_data

podman volume create logs

podman volume create backup
```

---

# Listar volúmenes

```bash
podman volume ls
```

Ejemplo

```text
DRIVER

local

NAME

postgres_data

web_data
```

---

# Inspeccionar un volumen

```bash
podman volume inspect postgres_data
```

Información obtenida:

- Nombre
- Driver
- Punto de montaje
- Opciones
- Etiquetas

---

# Consultar únicamente el MountPoint

```bash
podman volume inspect \
--format '{{.Mountpoint}}' \
postgres_data
```

Ejemplo

```text
/var/lib/containers/storage/volumes/postgres_data/_data
```

---

# Arquitectura

```text
               Podman

                  │

                  ▼

         Named Volume

                  │

                  ▼

     /var/lib/containers/
```

---

# Utilizar un volumen

```bash
podman run \
-d \
--name postgres \
-v postgres_data:/var/lib/postgresql/data \
postgres:17
```

---

# Flujo

```text
Container

      │

      ▼

/var/lib/postgresql/data

      │

      ▼

Named Volume

postgres_data
```

---

# Verificar

```bash
podman inspect postgres
```

Buscar:

```text
Mounts
```

---

# Crear archivos

```bash
podman exec postgres touch \
/var/lib/postgresql/data/test.txt
```

---

# Eliminar el contenedor

```bash
podman rm -f postgres
```

---

# Crear nuevamente

```bash
podman run \
-d \
--name postgres \
-v postgres_data:/var/lib/postgresql/data \
postgres:17
```

---

# Verificar persistencia

```bash
podman exec postgres ls \
/var/lib/postgresql/data
```

El archivo seguirá existiendo.

---

# Eliminar un volumen

```bash
podman volume rm postgres_data
```

---

# Restricción

Un volumen no puede eliminarse si está siendo utilizado.

---

# Eliminar volúmenes no utilizados

```bash
podman volume prune
```

---

# Confirmación

```text
WARNING!

This will remove all unused volumes.
```

---

# Bind Mount

A diferencia del Named Volume, el directorio ya existe en el Host.

Ejemplo

```text
/opt/web
```

---

# Crear directorio

```bash
mkdir -p /opt/web
```

---

# Montar el directorio

```bash
podman run \
-d \
-v /opt/web:/usr/share/nginx/html \
nginx
```

---

# Arquitectura

```text
Host

/opt/web

      │

      ▼

Container

/usr/share/nginx/html
```

---

# Resultado

Todo archivo copiado al Host aparecerá inmediatamente dentro del contenedor.

---

# Ejemplo

Host

```bash
echo "Hola RHCSA" > \
/opt/web/index.html
```

---

Dentro del contenedor

```bash
cat \
/usr/share/nginx/html/index.html
```

Resultado

```text
Hola RHCSA
```

---

# Compartir entre dos contenedores

Supongamos

```text
Container A

↓

Volume

↓

Container B
```

Ambos accederán exactamente a la misma información.

---

# Ejemplo

```bash
podman run \
-d \
--name web1 \
-v datos:/datos \
alpine sleep infinity
```

---

Segundo contenedor

```bash
podman run \
-d \
--name web2 \
-v datos:/datos \
alpine sleep infinity
```

---

Crear archivo

```bash
podman exec web1 \
touch /datos/demo.txt
```

---

Consultar desde el segundo

```bash
podman exec web2 \
ls /datos
```

Resultado

```text
demo.txt
```

---

# Opción -v

Es la forma tradicional.

```bash
-v origen:destino
```

Ejemplo

```bash
-v datos:/backup
```

---

# Opción --mount

Forma moderna.

```bash
--mount
```

Es más legible cuando existen muchas opciones.

---

# Sintaxis

```bash
--mount \
type=volume,\
source=datos,\
target=/backup
```

---

# Comparación

## -v

```bash
-v datos:/backup
```

---

## --mount

```bash
--mount \
type=volume,\
source=datos,\
target=/backup
```

---

# ¿Cuál utilizar?

| Escenario | Recomendación |
|-----------|---------------|
| Comandos rápidos | `-v` |
| Scripts complejos | `--mount` |
| Producción | `--mount` |
| Laboratorios | `-v` |

---

# Solo lectura

Podemos impedir modificaciones.

```bash
podman run \
-v datos:/datos:ro \
nginx
```

---

Resultado

```text
Read Only
```

---

# Verificar

Intentar crear un archivo.

```bash
touch /datos/test
```

Resultado

```text
Read-only filesystem
```

---

# Lectura y escritura

```bash
-v datos:/datos:rw
```

Es el comportamiento por defecto.

---

# SELinux

Fedora utiliza SELinux.

Esto afecta directamente los volúmenes.

---

# Problema común

Montamos:

```text
/opt/web
```

Pero Nginx devuelve:

```text
Permission denied
```

Aunque Linux indique permisos correctos.

---

# Causa

SELinux bloquea el acceso.

---

# Solución

Etiqueta privada

```bash
-v /opt/web:/usr/share/nginx/html:Z
```

---

# ¿Qué hace :Z?

Asigna una etiqueta SELinux exclusiva para ese contenedor.

---

# Arquitectura

```text
Host

/opt/web

SELinux

↓

Etiqueta privada

↓

Container
```

---

# Compartir entre varios contenedores

Utilizar

```text
:z
```

Ejemplo

```bash
-v /opt/web:/datos:z
```

---

# Diferencia

| Opción | Uso |
|---------|-----|
| :Z | Un único contenedor |
| :z | Compartido entre varios contenedores |

---

# Consultar contexto SELinux

Host

```bash
ls -lZ /opt/web
```

---

Ejemplo

```text
system_u:object_r:container_file_t
```

---

# Cambiar contexto manualmente

```bash
chcon \
-R \
-t container_file_t \
/opt/web
```

---

# Restaurar contexto

```bash
restorecon \
-R \
/opt/web
```

---

# Permisos Linux

Consultar

```bash
ls -l
```

---

Consultar propietario

```bash
stat archivo.txt
```

---

Consultar UID

```bash
id
```

---

# Problema frecuente

El usuario del contenedor:

```text
1001
```

No coincide con el propietario del Host.

---

# Solución

Cambiar propietario.

```bash
chown \
-R \
1001:1001 \
/opt/web
```

---

# Arquitectura completa

```text
             Fedora Host

                  │

         /opt/web

                  │

          SELinux Label

                  │

             Bind Mount

                  │

                  ▼

        Nginx Container
```

---

# Laboratorio RHCSA

## Laboratorio 1

Crear un volumen.

```bash
podman volume create datos
```

---

## Laboratorio 2

Listarlo.

```bash
podman volume ls
```

---

## Laboratorio 3

Inspeccionarlo.

```bash
podman volume inspect datos
```

---

## Laboratorio 4

Crear un contenedor.

```bash
podman run \
-d \
--name web \
-v datos:/datos \
alpine sleep infinity
```

---

## Laboratorio 5

Crear un archivo.

```bash
podman exec web \
touch /datos/test.txt
```

---

## Laboratorio 6

Eliminar el contenedor.

```bash
podman rm -f web
```

---

## Laboratorio 7

Crearlo nuevamente.

```bash
podman run \
-d \
--name web \
-v datos:/datos \
alpine sleep infinity
```

---

## Laboratorio 8

Verificar persistencia.

```bash
podman exec web \
ls /datos
```

---

## Laboratorio 9

Crear un Bind Mount.

```bash
mkdir /opt/web
```

---

## Laboratorio 10

Montarlo.

```bash
podman run \
-d \
-v /opt/web:/datos \
nginx
```

---

## Laboratorio 11

Utilizar SELinux.

```bash
podman run \
-v /opt/web:/datos:Z \
nginx
```

---

## Laboratorio 12

Consultar contexto.

```bash
ls -lZ /opt/web
```

---

## Laboratorio 13

Cambiar propietario.

```bash
sudo chown \
-R \
1001:1001 \
/opt/web
```

---

## Laboratorio 14

Eliminar volúmenes no utilizados.

```bash
podman volume prune
```

---

## Laboratorio 15

Consultar almacenamiento.

```bash
podman system df
```

---

# Buenas prácticas

- Utilizar **Named Volumes** para bases de datos y aplicaciones con estado.
- Utilizar **Bind Mounts** para archivos de configuración y desarrollo.
- Preferir `--mount` en scripts largos por su mayor claridad.
- Montar volúmenes en modo de solo lectura (`:ro`) cuando la aplicación no necesita escribir.
- Verificar siempre los contextos SELinux en Fedora.
- Mantener separados los datos de la aplicación y los archivos de configuración.

---

# Errores comunes

## Error 1

Eliminar un volumen creyendo que el contenedor conserva los datos.

---

## Error 2

Olvidar agregar `:Z` o `:z` en sistemas con SELinux habilitado.

---

## Error 3

Montar un directorio del Host con permisos insuficientes para el usuario del contenedor.

---

## Error 4

Modificar directamente el contenido de la Writable Layer cuando debería utilizarse un volumen persistente.

---

## Error 5

Utilizar un Bind Mount para datos críticos sin una estrategia de respaldo ni control de permisos.

---

# Resumen

En esta segunda fase aprendimos a:

- Crear, inspeccionar y eliminar volúmenes persistentes.
- Utilizar volúmenes administrados por Podman.
- Compartir datos entre múltiples contenedores.
- Trabajar con Bind Mounts.
- Comprender las diferencias entre `-v` y `--mount`.
- Utilizar montajes de solo lectura.
- Administrar permisos y propietarios.
- Resolver problemas relacionados con SELinux mediante las opciones `:Z` y `:z`.

En la **Fase 3** aprenderemos sobre copias de seguridad y restauración de volúmenes, exportación e importación de datos, migración entre servidores, volúmenes para bases de datos, rendimiento, cuotas de almacenamiento y escenarios avanzados utilizados en entornos empresariales y en el examen RHCSA.

----

# 73. Almacenamiento y Volúmenes en Podman (Fase 3)

> **Módulo 10 – Contenedores con Podman**
>
> **Manual RHCSA**
>
> **Archivo:** `73-almacenamiento-volumenes-podman.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase podrás:

- Realizar copias de seguridad de volúmenes.
- Restaurar información desde respaldos.
- Migrar volúmenes entre servidores.
- Exportar e importar datos.
- Comprender estrategias de respaldo para producción.
- Administrar almacenamiento de bases de datos.
- Optimizar el rendimiento de los volúmenes.
- Comprender cuotas y uso del almacenamiento.
- Diagnosticar problemas relacionados con almacenamiento persistente.

---

# Introducción

Los volúmenes permiten conservar la información cuando un contenedor es eliminado.

Sin embargo, la persistencia por sí sola no garantiza la disponibilidad de los datos.

En cualquier ambiente empresarial también es necesario:

- respaldar la información
- restaurarla rápidamente
- migrarla entre servidores
- monitorear el espacio utilizado
- verificar la integridad de los datos

---

# Estrategia recomendada

```text
            Aplicación

                 │

                 ▼

             Volumen

                 │

        ┌────────┴────────┐

        ▼                 ▼

    Backup Diario     Monitoreo

        │                 │

        └────────┬────────┘

                 ▼

            Recuperación
```

---

# ¿Qué respaldar?

Siempre deben respaldarse:

- Bases de datos
- Configuraciones
- Certificados
- Archivos cargados por usuarios
- Scripts personalizados
- Información de aplicaciones

---

# ¿Qué normalmente NO se respalda?

- Imágenes descargadas
- Contenedores temporales
- Writable Layer
- Cachés
- Directorios temporales

---

# Consultar MountPoint

Antes de respaldar un volumen es recomendable identificar su ubicación.

```bash
podman volume inspect \
--format '{{.Mountpoint}}' \
postgres_data
```

Ejemplo

```text
/var/lib/containers/storage/volumes/postgres_data/_data
```

---

# Método 1

## Respaldo utilizando tar

Consultar el directorio:

```bash
podman volume inspect postgres_data
```

Luego:

```bash
tar -czf \
postgres_backup.tar.gz \
-C /var/lib/containers/storage/volumes/postgres_data/_data .
```

---

# Explicación

```text
tar

↓

Comprime

↓

Todos los datos

↓

Archivo .tar.gz
```

---

# Restauración

Crear nuevamente el volumen.

```bash
podman volume create postgres_data
```

Extraer:

```bash
tar -xzf \
postgres_backup.tar.gz \
-C /var/lib/containers/storage/volumes/postgres_data/_data
```

---

# Arquitectura

```text
Volumen

      │

      ▼

Backup

.tar.gz

      │

      ▼

Nuevo Volumen
```

---

# Método 2

## Respaldo utilizando un contenedor temporal

Este método evita acceder directamente al directorio interno del almacenamiento.

```bash
podman run --rm \
-v postgres_data:/datos:ro \
-v $(pwd):/backup \
alpine \
tar czf /backup/postgres.tar.gz \
-C /datos .
```

---

# Ventajas

- Portable.
- Seguro.
- Independiente del sistema de archivos del Host.
- Muy utilizado en producción.

---

# Restauración

```bash
podman run --rm \
-v postgres_data:/datos \
-v $(pwd):/backup \
alpine \
tar xzf /backup/postgres.tar.gz \
-C /datos
```

---

# Migración entre servidores

Servidor A

```text
postgres.tar.gz
```

↓

Copiar mediante:

```text
scp
```

↓

Servidor B

↓

Restaurar

---

# Flujo completo

```text
Servidor A

     │

Backup

     │

scp

     │

Servidor B

     │

Restore

     │

Contenedor
```

---

# Copiar utilizando rsync

```bash
rsync -av \
postgres.tar.gz \
usuario@servidor:/backup
```

---

# Utilizando scp

```bash
scp \
postgres.tar.gz \
usuario@servidor:/backup
```

---

# Exportar un contenedor

Es importante comprender la diferencia.

```bash
podman export web \
-o web.tar
```

---

# ¿Qué exporta?

Exporta el sistema de archivos del contenedor.

No exporta:

- Volúmenes
- Imágenes
- Configuración de redes

---

# Importar

```bash
podman import \
web.tar \
web_image
```

---

# Exportar una imagen

No debe confundirse con:

```bash
podman save
```

---

# Guardar imagen

```bash
podman save \
-o nginx.tar \
nginx
```

---

# Restaurar imagen

```bash
podman load \
-i nginx.tar
```

---

# Comparación

| Comando | Qué guarda |
|----------|------------|
| podman export | Sistema de archivos del contenedor |
| podman save | Imagen OCI |
| tar | Datos del volumen |
| rsync | Archivos del Host |

---

# Bases de Datos

Las bases de datos siempre deben utilizar volúmenes persistentes.

Ejemplo PostgreSQL

```bash
podman run \
-d \
-v postgres_data:/var/lib/postgresql/data \
postgres:17
```

---

# Arquitectura

```text
          PostgreSQL

               │

               ▼

     /var/lib/postgresql/data

               │

               ▼

         Named Volume
```

---

# Nginx

Sitio Web

```bash
podman run \
-d \
-v web_data:/usr/share/nginx/html \
nginx
```

---

# Apache

```bash
podman run \
-d \
-v apache_data:/usr/local/apache2/htdocs \
httpd
```

---

# MongoDB

```bash
podman run \
-d \
-v mongo_data:/data/db \
mongo
```

---

# MariaDB

```bash
podman run \
-d \
-v mariadb_data:/var/lib/mysql \
mariadb
```

---

# Redis

```bash
podman run \
-d \
-v redis_data:/data \
redis
```

---

# Compartir Configuración

Host

```text
/etc/nginx
```

↓

Bind Mount

↓

Container

---

# Ejemplo

```bash
podman run \
-d \
-v /etc/nginx:/etc/nginx:ro,Z \
nginx
```

---

# Rendimiento

Factores que afectan el rendimiento:

- Disco
- SSD/NVMe
- Sistema de archivos
- Cantidad de archivos
- Tamaño de archivos
- OverlayFS
- SELinux
- I/O concurrente

---

# Recomendaciones

- Utilizar SSD o NVMe para bases de datos.
- Evitar miles de archivos pequeños en un mismo directorio.
- Separar datos y logs.
- Utilizar volúmenes dedicados.

---

# Monitorear espacio

Consultar:

```bash
podman system df
```

---

Consultar disco

```bash
df -h
```

---

Consultar uso del directorio

```bash
du -sh \
/var/lib/containers
```

---

# Monitorear I/O

```bash
iostat
```

---

También

```bash
iotop
```

(si está instalado).

---

# Verificar inodos

```bash
df -i
```

---

# ¿Por qué son importantes?

Puede existir espacio libre pero no inodos disponibles.

---

# Problema frecuente

```text
No space left on device
```

Aunque:

```text
df -h
```

muestre espacio disponible.

La causa puede ser:

```text
Sin inodos
```

---

# Cuotas

En algunos sistemas pueden implementarse cuotas de disco.

Objetivos:

- Evitar que una aplicación consuma todo el almacenamiento.
- Limitar el crecimiento de datos.
- Mejorar la administración del servidor.

---

# Arquitectura Empresarial

```text
                 SSD

                  │

         Volúmenes Podman

        ┌──────┼─────────┐

        ▼      ▼         ▼

     DB      Logs     Backups
```

---

# Estrategia 3-2-1

Una estrategia ampliamente utilizada para respaldos es la regla **3-2-1**.

Consiste en mantener:

- **3 copias** de los datos.
- **2 medios** de almacenamiento diferentes.
- **1 copia** almacenada fuera del servidor principal.

---

# Ejemplo

```text
Producción

      │

      ▼

Backup Local

      │

      ▼

NAS

      │

      ▼

Nube
```

---

# Automatización

Ejemplo de cron diario.

```cron
0 2 * * * /opt/scripts/backup_volumes.sh
```

---

# Script de respaldo

Guardar como:

```text
backup_volume.sh
```

```bash
#!/bin/bash

VOLUME=$1
DESTINO=$2

if [ -z "$VOLUME" ] || [ -z "$DESTINO" ]
then
    echo "Uso:"
    echo "./backup_volume.sh volumen destino"
    exit 1
fi

podman run --rm \
-v ${VOLUME}:/datos:ro \
-v ${DESTINO}:/backup \
alpine \
tar czf /backup/${VOLUME}_$(date +%F).tar.gz \
-C /datos .

echo
echo "Backup finalizado."
```

---

# Script de restauración

Guardar como:

```text
restore_volume.sh
```

```bash
#!/bin/bash

VOLUME=$1
ARCHIVO=$2

if [ -z "$VOLUME" ] || [ -z "$ARCHIVO" ]
then
    echo "Uso:"
    echo "./restore_volume.sh volumen archivo.tar.gz"
    exit 1
fi

podman run --rm \
-v ${VOLUME}:/datos \
-v $(dirname "$ARCHIVO"):/backup \
alpine \
tar xzf /backup/$(basename "$ARCHIVO") \
-C /datos

echo
echo "Restauración completada."
```

---

# Laboratorio RHCSA

## Laboratorio 1

Crear un volumen.

```bash
podman volume create postgres_data
```

---

## Laboratorio 2

Crear PostgreSQL.

```bash
podman run \
-d \
-v postgres_data:/var/lib/postgresql/data \
postgres:17
```

---

## Laboratorio 3

Consultar el MountPoint.

```bash
podman volume inspect postgres_data
```

---

## Laboratorio 4

Crear un respaldo.

```bash
tar -czf postgres.tar.gz \
-C /var/lib/containers/storage/volumes/postgres_data/_data .
```

---

## Laboratorio 5

Crear un respaldo utilizando un contenedor temporal.

```bash
podman run --rm \
-v postgres_data:/datos:ro \
-v $(pwd):/backup \
alpine \
tar czf /backup/postgres.tar.gz \
-C /datos .
```

---

## Laboratorio 6

Eliminar el contenedor.

```bash
podman rm -f postgres
```

---

## Laboratorio 7

Restaurar el respaldo.

```bash
podman run --rm \
-v postgres_data:/datos \
-v $(pwd):/backup \
alpine \
tar xzf /backup/postgres.tar.gz \
-C /datos
```

---

## Laboratorio 8

Consultar almacenamiento.

```bash
podman system df
```

---

## Laboratorio 9

Consultar espacio libre.

```bash
df -h
```

---

## Laboratorio 10

Consultar uso del directorio.

```bash
du -sh \
/var/lib/containers
```

---

## Laboratorio 11

Consultar inodos.

```bash
df -i
```

---

## Laboratorio 12

Guardar una imagen.

```bash
podman save \
-o nginx.tar \
nginx
```

---

## Laboratorio 13

Cargar la imagen.

```bash
podman load \
-i nginx.tar
```

---

## Laboratorio 14

Exportar un contenedor.

```bash
podman export web \
-o web.tar
```

---

## Laboratorio 15

Importar el contenedor.

```bash
podman import \
web.tar \
web_image
```

---

# Buenas prácticas

- Realizar respaldos periódicos de todos los volúmenes críticos.
- Probar regularmente los procedimientos de restauración.
- Utilizar la estrategia de respaldo **3-2-1**.
- Mantener los datos separados de los archivos de configuración.
- Supervisar el espacio disponible y el consumo de inodos.
- Documentar las rutas de almacenamiento y los procedimientos de recuperación.
- Automatizar los respaldos mediante `cron` o `systemd timers`.

---

# Errores comunes

## Error 1

Respaldar únicamente el contenedor y olvidar el volumen asociado.

---

## Error 2

Confundir `podman export` con `podman save`.

---

## Error 3

No verificar la integridad del respaldo antes de eliminar los datos originales.

---

## Error 4

Restaurar archivos sobre un volumen en uso por una base de datos activa.

---

## Error 5

No supervisar el uso de inodos, provocando errores de escritura a pesar de disponer de espacio libre.

---

# Resumen

En esta tercera fase aprendimos a:

- Realizar copias de seguridad y restauraciones de volúmenes.
- Migrar datos entre servidores mediante `tar`, `scp` y `rsync`.
- Comprender la diferencia entre `podman export`, `podman import`, `podman save` y `podman load`.
- Administrar almacenamiento persistente para bases de datos y aplicaciones web.
- Monitorear espacio, uso de disco e inodos.
- Automatizar respaldos mediante scripts y tareas programadas.
- Aplicar estrategias de respaldo utilizadas en entornos empresariales.

En la **Fase 4** integraremos todos estos conceptos en un laboratorio completo con escenarios reales de producción, auditoría de volúmenes, resolución de problemas, checklist RHCSA, preguntas de repaso y un desafío final similar al examen oficial.

----


# 73. Almacenamiento y Volúmenes en Podman (Fase 4)

> **Módulo 10 – Contenedores con Podman**
>
> **Manual RHCSA**
>
> **Archivo:** `73-almacenamiento-volumenes-podman.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase podrás:

- Diagnosticar problemas relacionados con almacenamiento persistente.
- Resolver errores comunes en volúmenes y Bind Mounts.
- Auditar el almacenamiento de Podman.
- Automatizar revisiones mediante scripts.
- Aplicar buenas prácticas empresariales.
- Resolver escenarios similares al examen RHCSA.

---

# Metodología de Diagnóstico

Cuando una aplicación pierde datos o no puede escribir en disco, el problema rara vez está en Podman.

Siempre debe seguirse un procedimiento ordenado.

```text
               Aplicación

                    │

                    ▼

         ¿Contenedor ejecutándose?

                    │

                    ▼

      ¿Volumen correctamente montado?

                    │

                    ▼

        ¿Permisos suficientes?

                    │

                    ▼

          ¿SELinux bloquea?

                    │

                    ▼

        ¿Espacio disponible?

                    │

                    ▼

       ¿Inodos disponibles?

                    │

                    ▼

        Aplicar la corrección
```

---

# Lista de verificación

Siempre revisar:

```bash
podman ps
```

↓

```bash
podman volume ls
```

↓

```bash
podman inspect
```

↓

```bash
podman volume inspect
```

↓

```bash
df -h
```

↓

```bash
df -i
```

↓

```bash
ls -lZ
```

↓

```bash
restorecon
```

---

# Escenario 1

## La base de datos perdió toda la información

Posible causa:

```text
No se utilizó un volumen persistente.
```

Configuración incorrecta:

```bash
podman run \
-d \
postgres:17
```

Configuración correcta:

```bash
podman run \
-d \
-v postgres_data:/var/lib/postgresql/data \
postgres:17
```

---

# Escenario 2

## El volumen existe pero la aplicación no escribe

Consultar:

```bash
podman inspect postgres
```

Buscar:

```text
Mounts
```

Verificar:

- Source
- Destination
- ReadOnly

---

# Escenario 3

## Error "Permission denied"

Consultar:

```bash
ls -ld /opt/datos
```

Consultar:

```bash
ls -lZ /opt/datos
```

Posibles causas:

- propietario incorrecto
- permisos Linux
- SELinux

---

# Solución

Aplicar contexto:

```bash
restorecon -R /opt/datos
```

o

```bash
chcon -R \
-t container_file_t \
/opt/datos
```

---

# Escenario 4

## Error de propietario

Consultar:

```bash
id
```

Dentro del contenedor.

Luego:

```bash
ls -ln /opt/datos
```

Host.

Corregir:

```bash
chown -R 1001:1001 \
/opt/datos
```

---

# Escenario 5

## El volumen desapareció

Consultar:

```bash
podman volume ls
```

Si no existe:

Revisar:

```bash
podman volume prune
```

Puede haberse eliminado por error.

---

# Escenario 6

## No hay espacio

Consultar:

```bash
df -h
```

Luego:

```bash
podman system df
```

---

# Escenario 7

## Espacio disponible pero continúa el error

Consultar:

```bash
df -i
```

Puede existir:

```text
No space left on device
```

por agotamiento de inodos.

---

# Escenario 8

## SELinux bloquea un Bind Mount

Configuración incorrecta

```bash
-v /opt/web:/datos
```

Configuración correcta

```bash
-v /opt/web:/datos:Z
```

o

```bash
-v /opt/web:/datos:z
```

---

# Escenario 9

## Los datos no aparecen

Consultar:

```bash
podman volume inspect datos
```

Verificar:

```text
Mountpoint
```

Comprobar que realmente el contenedor utiliza ese volumen.

---

# Escenario 10

## El respaldo no funciona

Consultar:

```bash
podman volume inspect
```

Verificar:

- ruta correcta
- permisos
- espacio libre

---

# Flujo recomendado

```text
Problema

      │

      ▼

podman inspect

      │

      ▼

Volume Inspect

      │

      ▼

Permisos

      │

      ▼

SELinux

      │

      ▼

Espacio

      │

      ▼

Restaurar
```

---

# Herramientas de diagnóstico

## Podman

```bash
podman volume ls
```

```bash
podman volume inspect
```

```bash
podman inspect
```

```bash
podman system df
```

```bash
podman system prune
```

---

## Linux

```bash
df -h
```

```bash
df -i
```

```bash
du -sh
```

```bash
mount
```

```bash
findmnt
```

```bash
lsblk
```

```bash
ls -lZ
```

```bash
restorecon
```

---

# Script de Auditoría de Volúmenes

Guardar como:

```text
volume_audit.sh
```

```bash
#!/bin/bash

echo "========================================"
echo "      AUDITORÍA DE VOLÚMENES PODMAN"
echo "========================================"

echo
echo "VOLÚMENES"
podman volume ls

echo
echo "ESPACIO PODMAN"
podman system df

echo
echo "ESPACIO DEL DISCO"
df -h

echo
echo "INODOS"
df -i

echo
echo "GRAPHROOT"
podman info --format '{{.Store.GraphRoot}}'
```

Permisos:

```bash
chmod +x volume_audit.sh
```

---

# Script para inspeccionar todos los volúmenes

```bash
#!/bin/bash

for v in $(podman volume ls --format "{{.Name}}")
do
    echo
    echo "=============================="
    echo "$v"
    echo "=============================="

    podman volume inspect "$v"
done
```

---

# Script para respaldar todos los volúmenes

Guardar como:

```text
backup_all_volumes.sh
```

```bash
#!/bin/bash

DESTINO=/backup/podman

mkdir -p "$DESTINO"

for v in $(podman volume ls --format "{{.Name}}")
do

echo
echo "Respaldando $v"

podman run --rm \
-v ${v}:/datos:ro \
-v ${DESTINO}:/backup \
alpine \
tar czf /backup/${v}.tar.gz \
-C /datos .

done

echo
echo "Todos los respaldos finalizaron."
```

---

# Script para restaurar un volumen

```bash
#!/bin/bash

VOLUMEN=$1

ARCHIVO=$2

podman run --rm \
-v ${VOLUMEN}:/datos \
-v $(dirname "$ARCHIVO"):/backup \
alpine \
tar xzf /backup/$(basename "$ARCHIVO") \
-C /datos
```

---

# Arquitectura Empresarial

```text
                Usuarios

                     │

                     ▼

              Aplicaciones

         ┌────────┼────────┐

         ▼        ▼        ▼

      PostgreSQL MongoDB Redis

         │        │        │

         └────────┼────────┘

                  ▼

            Named Volumes

                  │

                  ▼

             SSD / RAID

                  │

                  ▼

         Backups Automatizados
```

---

# Arquitectura de Alta Disponibilidad

```text
             Producción

                  │

                  ▼

           Named Volume

                  │

        ┌─────────┴─────────┐

        ▼                   ▼

 Backup Diario        Snapshot

        │                   │

        └─────────┬─────────┘

                  ▼

          Servidor Secundario
```

---

# Laboratorio RHCSA

## Laboratorio 1

Crear un volumen.

```bash
podman volume create datos
```

---

## Laboratorio 2

Listarlo.

```bash
podman volume ls
```

---

## Laboratorio 3

Inspeccionarlo.

```bash
podman volume inspect datos
```

---

## Laboratorio 4

Crear un contenedor.

```bash
podman run \
-d \
-v datos:/datos \
--name laboratorio \
alpine sleep infinity
```

---

## Laboratorio 5

Crear un archivo.

```bash
podman exec laboratorio \
touch /datos/demo.txt
```

---

## Laboratorio 6

Eliminar el contenedor.

```bash
podman rm -f laboratorio
```

---

## Laboratorio 7

Crear nuevamente el contenedor.

```bash
podman run \
-d \
-v datos:/datos \
--name laboratorio \
alpine sleep infinity
```

---

## Laboratorio 8

Verificar persistencia.

```bash
podman exec laboratorio \
ls /datos
```

---

## Laboratorio 9

Consultar GraphRoot.

```bash
podman info \
--format '{{.Store.GraphRoot}}'
```

---

## Laboratorio 10

Consultar almacenamiento.

```bash
podman system df
```

---

## Laboratorio 11

Consultar espacio.

```bash
df -h
```

---

## Laboratorio 12

Consultar inodos.

```bash
df -i
```

---

## Laboratorio 13

Ejecutar auditoría.

```bash
./volume_audit.sh
```

---

## Laboratorio 14

Respaldar todos los volúmenes.

```bash
./backup_all_volumes.sh
```

---

## Laboratorio 15

Restaurar un volumen.

```bash
./restore_volume.sh datos datos.tar.gz
```

---

# Checklist RHCSA

```text
□ Los datos críticos utilizan Named Volumes.

□ Los Bind Mounts poseen permisos correctos.

□ SELinux permite el acceso.

□ Los propietarios son correctos.

□ Existe espacio libre suficiente.

□ Existen inodos disponibles.

□ Se verificó el MountPoint.

□ Los respaldos fueron realizados.

□ Las restauraciones fueron probadas.

□ Los scripts funcionan correctamente.

□ No existen volúmenes huérfanos.

□ El almacenamiento está documentado.
```

---

# Preguntas de Repaso

1. ¿Qué ocurre con la Writable Layer al eliminar un contenedor?
2. ¿Cuál es la diferencia entre un Named Volume y un Bind Mount?
3. ¿Qué comando crea un volumen?
4. ¿Cómo listar todos los volúmenes?
5. ¿Qué hace `podman volume inspect`?
6. ¿Cuál es la diferencia entre `-v` y `--mount`?
7. ¿Qué hace la opción `:ro`?
8. ¿Cuál es la diferencia entre `:Z` y `:z`?
9. ¿Qué comando muestra el espacio utilizado por Podman?
10. ¿Cómo consultar el directorio donde Podman almacena los datos?
11. ¿Qué diferencia existe entre `podman export` y `podman save`?
12. ¿Qué utilidad tiene `podman load`?
13. ¿Qué utilidad tiene `podman volume prune`?
14. ¿Qué comando muestra los inodos disponibles?
15. ¿Qué estrategia de respaldo es ampliamente recomendada?
16. ¿Qué herramienta permite restaurar el contexto SELinux?
17. ¿Qué utilidad tiene `podman system df`?
18. ¿Por qué es recomendable probar periódicamente los respaldos?
19. ¿Cuándo utilizarías un Bind Mount en lugar de un Named Volume?
20. ¿Qué revisarías primero si una aplicación devuelve "Permission denied" al escribir en un volumen?

---

# Respuestas

1. Se elimina junto con el contenedor y sus datos se pierden.
2. El Named Volume es administrado por Podman; el Bind Mount utiliza un directorio existente del Host.
3. `podman volume create`
4. `podman volume ls`
5. Muestra información detallada del volumen, incluido su punto de montaje.
6. Ambos montan almacenamiento; `--mount` ofrece una sintaxis más explícita y flexible.
7. Monta el volumen en modo de solo lectura.
8. `:Z` aplica una etiqueta SELinux exclusiva para un contenedor; `:z` crea una etiqueta compartida para varios contenedores.
9. `podman system df`
10. `podman info --format '{{.Store.GraphRoot}}'`
11. `podman export` exporta el sistema de archivos de un contenedor; `podman save` exporta una imagen OCI.
12. Importar una imagen previamente guardada.
13. Elimina volúmenes que no están siendo utilizados por ningún contenedor.
14. `df -i`
15. La estrategia **3-2-1**.
16. `restorecon`
17. Mostrar el uso de almacenamiento de imágenes, contenedores y volúmenes administrados por Podman.
18. Porque un respaldo no verificado puede resultar inutilizable durante una recuperación real.
19. Cuando se necesita acceder directamente a un directorio específico del Host, por ejemplo para desarrollo o archivos de configuración.
20. Los permisos Linux, el propietario del directorio y el contexto SELinux.

---

# Desafío Final RHCSA

Se entrega un servidor Fedora con Podman instalado.

Realiza las siguientes tareas:

1. Crear un volumen llamado `postgres_data`.
2. Ejecutar un contenedor PostgreSQL utilizando dicho volumen.
3. Crear un respaldo del volumen mediante un contenedor temporal con Alpine.
4. Verificar el punto de montaje del volumen.
5. Consultar el espacio utilizado por Podman.
6. Comprobar el espacio libre y los inodos del sistema.
7. Crear un Bind Mount para un sitio web ubicado en `/opt/web`.
8. Configurar correctamente el contexto SELinux utilizando `:Z`.
9. Crear un archivo desde el Host y verificar que sea visible dentro del contenedor.
10. Eliminar y recrear el contenedor comprobando que los datos del volumen permanecen.
11. Ejecutar el script `volume_audit.sh`.
12. Restaurar un respaldo previamente generado.
13. Eliminar únicamente los recursos no utilizados sin afectar los datos en producción.
14. Documentar la estrategia de respaldo y recuperación implementada.

---

# Buenas prácticas

- Utilizar volúmenes persistentes para cualquier aplicación con estado.
- Mantener separados los datos, las configuraciones y los respaldos.
- Implementar una política de copias de seguridad automatizadas y verificadas.
- Supervisar periódicamente el consumo de espacio e inodos.
- Aprovechar las etiquetas SELinux (`:Z` y `:z`) en sistemas Fedora y RHEL.
- Evitar almacenar información crítica en la Writable Layer.
- Probar regularmente los procedimientos de recuperación en un entorno de laboratorio.

---

# Resumen del Capítulo 73

En este capítulo aprendimos a:

- Comprender la arquitectura de almacenamiento de Podman basada en OverlayFS y Copy-on-Write.
- Diferenciar entre almacenamiento efímero y persistente.
- Crear, administrar e inspeccionar Named Volumes.
- Trabajar con Bind Mounts y montajes temporales (`tmpfs`).
- Comprender las diferencias entre `-v` y `--mount`.
- Resolver problemas de permisos y SELinux.
- Realizar copias de seguridad, restauraciones y migraciones de volúmenes.
- Supervisar el almacenamiento mediante herramientas de Linux y Podman.
- Automatizar auditorías y respaldos con scripts Bash.
- Resolver escenarios prácticos similares a los encontrados en el examen **RHCSA** y en entornos empresariales.

---

# Fin del capítulo











