# 69. Instalación y Configuración de Podman (Fase 2)

> **Módulo 10 – Contenedores con Podman**
>
> **Archivo:** `69-instalacion-configuracion-podman.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase podrás:

- Comprender la estructura del directorio `/etc/containers`.
- Configurar Podman a nivel global y por usuario.
- Comprender la precedencia de los archivos de configuración.
- Configurar registros (registries).
- Configurar almacenamiento.
- Comprender el funcionamiento de OverlayFS.
- Administrar GraphRoot y RunRoot.
- Configurar registros inseguros.
- Configurar mirrors.
- Comprender la resolución de nombres cortos.
- Personalizar el comportamiento predeterminado de Podman.
- Validar la configuración utilizando `podman info`.

---

# Introducción

Una vez instalado Podman, el siguiente paso consiste en comprender cómo está configurado.

A diferencia de Docker, Podman permite una configuración muy flexible.

Puede configurarse:

- A nivel del sistema.
- A nivel de usuario.
- Mediante variables de entorno.
- Mediante parámetros en línea de comandos.

La configuración se encuentra principalmente bajo:

```text
/etc/containers
```

y también dentro del directorio HOME de cada usuario.

---

# 36. Arquitectura de configuración

```text
                   Línea de comandos
                           │
                           ▼
                 Configuración del usuario
                           │
                           ▼
             Configuración global (/etc)
                           │
                           ▼
                Valores internos de Podman
```

Las opciones más específicas tienen prioridad.

---

# 37. Directorio /etc/containers

Visualizar:

```bash
ls -l /etc/containers
```

Ejemplo:

```text
containers.conf
policy.json
registries.conf
storage.conf
registries.conf.d/
```

Dependiendo de la versión pueden existir archivos adicionales.

---

# 38. Archivos principales

| Archivo | Función |
|----------|---------|
| containers.conf | Configuración general |
| registries.conf | Registros de imágenes |
| storage.conf | Almacenamiento |
| policy.json | Políticas de confianza |
| registries.conf.d | Configuración adicional |

---

# 39. Jerarquía de configuración

```text
Podman

│

├── /usr/share/containers/
│
├── /etc/containers/
│
└── ~/.config/containers/
```

El usuario puede sobrescribir configuraciones globales.

---

# 40. Configuración por usuario

Crear el directorio:

```bash
mkdir -p ~/.config/containers
```

Listarlo:

```bash
tree ~/.config/containers
```

Inicialmente puede estar vacío.

---

# 41. Consultar configuración efectiva

Uno de los comandos más útiles:

```bash
podman info
```

Toda la configuración utilizada por Podman puede observarse aquí.

---

# 42. containers.conf

Este archivo controla el comportamiento predeterminado de Podman.

Ubicación:

```text
/etc/containers/containers.conf
```

Visualizar:

```bash
less /etc/containers/containers.conf
```

---

# 43. Estructura de containers.conf

Está organizado en secciones.

Ejemplo:

```toml
[containers]

[engine]

[machine]

[network]
```

Cada sección controla un conjunto diferente de parámetros.

---

# 44. Sección [containers]

Aquí se configuran parámetros por defecto de los contenedores.

Por ejemplo:

- Umask
- DNS
- Hosts
- Timezone
- Log Driver
- IPC
- PID
- UTS
- UserNS

---

# 45. Sección [engine]

Controla el motor Podman.

Ejemplos:

- Runtime
- Events
- Cgroup Manager
- Pull Policy
- Volume Path

---

# 46. Sección [network]

Controla:

- DNS
- Network Backend
- MTU
- Default Network

---

# 47. Validar sintaxis

Después de modificar un archivo:

```bash
podman info
```

Si existe un error de sintaxis Podman normalmente lo reportará.

---

# 48. Configuración por usuario

Copiar el archivo global:

```bash
cp \
/etc/containers/containers.conf \
~/.config/containers/
```

Modificar únicamente las opciones necesarias.

---

# 49. Precedencia

Si un parámetro existe:

```text
/etc/containers/containers.conf
```

y también:

```text
~/.config/containers/containers.conf
```

prevalece la configuración del usuario.

---

# 50. registries.conf

Ubicación:

```text
/etc/containers/registries.conf
```

Consultar:

```bash
less /etc/containers/registries.conf
```

---

# 51. Función de registries.conf

Controla:

- Registros permitidos.
- Registros inseguros.
- Mirrors.
- Búsqueda de imágenes.
- Resolución de nombres cortos.

---

# 52. Registros

Ejemplos:

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

# 53. Resolución de nombres cortos

Cuando ejecutamos:

```bash
podman pull alpine
```

Podman debe decidir desde qué registro descargar.

Esto depende de:

```text
unqualified-search-registries
```

---

# 54. Consultar registros

Buscar:

```bash
grep unqualified \
/etc/containers/registries.conf
```

Ejemplo:

```toml
unqualified-search-registries=[
"registry.access.redhat.com",
"registry.redhat.io",
"docker.io"
]
```

---

# 55. Buenas prácticas

En producción es recomendable utilizar siempre nombres completos.

Ejemplo:

Correcto:

```bash
podman pull docker.io/library/alpine:latest
```

En lugar de:

```bash
podman pull alpine
```

---

# 56. Mirrors

Un mirror es un registro alternativo.

```text
Cliente

│

▼

Mirror

│

▼

Registro principal
```

Reduce tráfico y mejora disponibilidad.

---

# 57. Registros inseguros

Puede declararse un registro interno sin TLS.

Ejemplo conceptual:

```toml
[[registry]]

location="registry.interno"

insecure=true
```

**No es recomendable** utilizar registros inseguros en producción salvo que exista una justificación técnica y se comprendan los riesgos.

---

# 58. Bloquear registros

También pueden bloquearse registros.

Ejemplo:

```toml
blocked=true
```

Esto impide descargar imágenes desde dicho registro.

---

# 59. policy.json

Ubicación:

```text
/etc/containers/policy.json
```

Consultar:

```bash
less /etc/containers/policy.json
```

---

# 60. Función de policy.json

Controla la política de confianza.

Ejemplo:

- ¿Qué imágenes pueden ejecutarse?
- ¿Deben verificarse firmas?
- ¿Qué registros son confiables?

---

# 61. storage.conf

Ubicación:

```text
/etc/containers/storage.conf
```

Consultar:

```bash
less /etc/containers/storage.conf
```

---

# 62. Función de storage.conf

Controla:

- Driver de almacenamiento.
- Directorio GraphRoot.
- Directorio RunRoot.
- OverlayFS.
- Opciones de almacenamiento.

---

# 63. Driver Overlay

Consultar:

```bash
podman info \
--format '{{.Store.GraphDriverName}}'
```

Generalmente:

```text
overlay
```

---

# 64. ¿Qué es OverlayFS?

OverlayFS permite combinar varias capas.

```text
Imagen

│

├── Layer 1

├── Layer 2

├── Layer 3

└── Writable Layer
```

Cada contenedor agrega una capa de escritura.

---

# 65. GraphRoot

Consultar:

```bash
podman info \
--format '{{.Store.GraphRoot}}'
```

Ejemplo Rootful:

```text
/var/lib/containers/storage
```

Rootless:

```text
~/.local/share/containers/storage
```

---

# 66. RunRoot

Consultar:

```bash
podman info \
--format '{{.Store.RunRoot}}'
```

Ejemplo:

```text
/run/containers/storage
```

---

# 67. Diferencia entre GraphRoot y RunRoot

| GraphRoot | RunRoot |
|------------|---------|
| Persistente | Temporal |
| Imágenes | Datos de ejecución |
| Sobrevive reinicios | Normalmente no |

---

# 68. Directorios de almacenamiento

Visualizar:

```bash
tree \
-L 2 \
/var/lib/containers/storage
```

En Rootless:

```bash
tree \
-L 2 \
~/.local/share/containers/storage
```

---

# 69. Variables de entorno

Podman puede utilizar:

```text
CONTAINERS_STORAGE_CONF
```

```text
CONTAINERS_CONF
```

```text
REGISTRIES_CONFIG_PATH
```

Ver:

```bash
env | grep CONTAINERS
```

---

# 70. Configuración temporal

Puede utilizarse una configuración distinta:

```bash
CONTAINERS_CONF=mi.conf podman info
```

Muy útil para pruebas.

---

# 71. Información del almacenamiento

Consultar:

```bash
podman system df
```

Versión detallada:

```bash
podman system df -v
```

---

# 72. Verificar imágenes

```bash
podman images
```

---

# 73. Verificar volúmenes

```bash
podman volume ls
```

---

# 74. Verificar redes

```bash
podman network ls
```

---

# 75. Diagnóstico completo

```bash
podman info
```

Debe revisarse:

- Runtime.
- Driver.
- GraphRoot.
- RunRoot.
- Rootless.
- Versiones.
- Cgroups.

---

# 76. Laboratorio RHCSA

## Ejercicio 1

Listar:

```bash
ls /etc/containers
```

---

## Ejercicio 2

Abrir:

```bash
less /etc/containers/containers.conf
```

---

## Ejercicio 3

Consultar:

```bash
less /etc/containers/registries.conf
```

---

## Ejercicio 4

Consultar:

```bash
less /etc/containers/storage.conf
```

---

## Ejercicio 5

Mostrar:

```bash
podman info
```

---

## Ejercicio 6

Obtener GraphRoot.

```bash
podman info \
--format '{{.Store.GraphRoot}}'
```

---

## Ejercicio 7

Obtener RunRoot.

```bash
podman info \
--format '{{.Store.RunRoot}}'
```

---

## Ejercicio 8

Obtener Driver.

```bash
podman info \
--format '{{.Store.GraphDriverName}}'
```

---

## Ejercicio 9

Verificar Rootless.

```bash
podman info \
--format '{{.Host.Security.Rootless}}'
```

---

## Ejercicio 10

Consultar redes.

```bash
podman network ls
```

---

## Ejercicio 11

Consultar volúmenes.

```bash
podman volume ls
```

---

## Ejercicio 12

Consultar espacio.

```bash
podman system df
```

---

# Buenas prácticas

- No modificar archivos sin realizar copia de seguridad.
- Utilizar registros oficiales.
- Evitar registros inseguros.
- Utilizar nombres completos de imágenes.
- Mantener configuración por usuario separada.
- Verificar siempre con `podman info`.

---

# Errores comunes

## Error 1

Modificar GraphRoot manualmente sin mover el contenido.

---

## Error 2

Eliminar directorios de almacenamiento.

---

## Error 3

Usar registros inseguros innecesariamente.

---

## Error 4

Modificar `containers.conf` sin validar.

---

## Error 5

Confundir Rootless con Rootful.

---

# Resumen

En esta fase aprendimos:

- La estructura completa de `/etc/containers`.
- El propósito de `containers.conf`.
- La función de `registries.conf`.
- La administración del almacenamiento mediante `storage.conf`.
- El funcionamiento de OverlayFS.
- La diferencia entre `GraphRoot` y `RunRoot`.
- La configuración por usuario y la precedencia de archivos.

La **Fase 3** profundizará en:

- User Namespaces.
- SubUID y SubGID.
- Rootless Networking.
- Rootless Storage.
- SELinux.
- Firewall.
- Diagnóstico avanzado.
- Solución de problemas y auditoría.