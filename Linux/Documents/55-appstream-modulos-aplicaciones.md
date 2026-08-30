# 55. AppStream y Módulos de Aplicaciones

> **Módulo 8: Administración del Software y Repositorios**  
> **Página 55 del Manual RHCSA**

---

# Objetivos de aprendizaje

Al finalizar esta lección serás capaz de:

- Comprender qué es AppStream.
- Identificar la diferencia entre BaseOS y AppStream.
- Comprender qué son los módulos de aplicaciones.
- Consultar módulos y streams disponibles.
- Habilitar, deshabilitar y restablecer módulos.
- Instalar software desde un stream específico.
- Comprender el concepto de perfil.
- Diagnosticar problemas relacionados con módulos.
- Aplicar buenas prácticas en entornos RHEL.

---

# Introducción

Red Hat Enterprise Linux necesita mantener un equilibrio entre dos objetivos:

- Ofrecer una plataforma estable durante muchos años.
- Permitir que los administradores utilicen versiones diferentes de determinadas aplicaciones.

Tradicionalmente, un sistema operativo incluía una sola versión principal de cada paquete.

Por ejemplo:

```text
PHP 7
```

Si el administrador necesitaba otra versión, debía:

- Utilizar repositorios externos.
- Compilar el software.
- Instalar paquetes manualmente.
- Crear entornos separados.

Para resolver esta situación, Red Hat introdujo **AppStream**.

AppStream permite distribuir aplicaciones, lenguajes y herramientas en diferentes versiones sin modificar la base estable del sistema operativo.

---

# ¿Qué es AppStream?

AppStream significa:

```text
Application Stream
```

Es uno de los repositorios principales de Red Hat Enterprise Linux.

Contiene:

- Lenguajes de programación.
- Servidores de bases de datos.
- Servidores web.
- Frameworks.
- Herramientas de desarrollo.
- Aplicaciones.
- Módulos con diferentes versiones.

Ejemplos de software que puede aparecer en AppStream:

- PHP.
- MariaDB.
- PostgreSQL.
- Node.js.
- Perl.
- Python.
- Ruby.
- Nginx.
- Redis.

---

# BaseOS y AppStream

En RHEL moderno, el contenido suele dividirse principalmente entre:

```text
BaseOS
```

y:

```text
AppStream
```

---

# BaseOS

BaseOS contiene los componentes fundamentales del sistema operativo.

Ejemplos:

- Kernel.
- systemd.
- Bash.
- RPM.
- DNF.
- SELinux.
- Coreutils.
- Bibliotecas esenciales.
- Herramientas básicas de administración.

BaseOS proporciona la plataforma estable sobre la cual funciona el sistema.

---

# AppStream

AppStream contiene aplicaciones, lenguajes y componentes que pueden evolucionar con mayor flexibilidad.

Ejemplos:

- PostgreSQL.
- MariaDB.
- PHP.
- Node.js.
- Ruby.
- Perl.
- Nginx.

---

# Comparación entre BaseOS y AppStream

| BaseOS | AppStream |
|--------|-----------|
| Componentes esenciales | Aplicaciones y herramientas |
| Kernel y sistema base | Lenguajes y servidores |
| Alta estabilidad | Mayor flexibilidad de versiones |
| Necesario para arrancar y operar | Amplía las capacidades del sistema |
| Cambios controlados | Puede ofrecer varios streams |

---

# Arquitectura general

```text
Sistema RHEL
    │
    ├── BaseOS
    │     ├── Kernel
    │     ├── systemd
    │     ├── Bash
    │     ├── RPM
    │     └── Herramientas básicas
    │
    └── AppStream
          ├── PHP
          ├── PostgreSQL
          ├── MariaDB
          ├── Node.js
          ├── Ruby
          └── Nginx
```

---

# ¿Qué es un módulo?

Un módulo es un conjunto organizado de paquetes RPM relacionados.

Un módulo puede contener diferentes versiones de una aplicación.

Ejemplo conceptual:

```text
Módulo: nodejs
    │
    ├── Stream 18
    ├── Stream 20
    └── Stream 22
```

Cada versión recibe el nombre de:

```text
stream
```

---

# ¿Qué es un stream?

Un stream representa una versión o línea de mantenimiento específica de una aplicación.

Ejemplo:

```text
postgresql:13
```

Se interpreta como:

```text
Módulo: postgresql

Stream: 13
```

Otro ejemplo:

```text
nodejs:20
```

Significa:

```text
Módulo: nodejs

Stream: 20
```

---

# ¿Qué es un perfil?

Un perfil es una selección predefinida de paquetes dentro de un módulo.

Los perfiles permiten instalar únicamente los componentes necesarios para un propósito determinado.

Ejemplo conceptual:

```text
Módulo: postgresql
    │
    ├── Perfil client
    └── Perfil server
```

Un perfil puede instalar:

- Solo el cliente.
- Solo el servidor.
- Herramientas de desarrollo.
- Un conjunto completo.
- Componentes mínimos.

---

# Estructura de un módulo

```text
Módulo
   │
   ├── Nombre
   │
   ├── Stream
   │
   ├── Perfil
   │
   └── Paquetes RPM
```

Ejemplo:

```text
postgresql
   │
   ├── Stream: 15
   │
   ├── Perfil: server
   │
   └── Paquetes:
         ├── postgresql
         ├── postgresql-server
         ├── postgresql-libs
         └── postgresql-contrib
```

---

# Verificar los repositorios

Antes de trabajar con AppStream, verifica que los repositorios necesarios estén habilitados.

```bash
dnf repolist
```

Debes observar repositorios equivalentes a:

```text
BaseOS
AppStream
```

En RHEL con suscripción pueden aparecer nombres más largos.

Ejemplo:

```text
rhel-9-for-x86_64-baseos-rpms
rhel-9-for-x86_64-appstream-rpms
```

---

# Listar módulos disponibles

```bash
dnf module list
```

Este comando muestra:

- Nombre del módulo.
- Streams disponibles.
- Stream predeterminado.
- Stream habilitado.
- Perfiles.
- Estado del módulo.

---

# Ejemplo de salida

```text
Name        Stream      Profiles                 Summary
nodejs      18          common [d], development  Javascript runtime
nodejs      20 [d]      common [d], development  Javascript runtime
postgresql  13          client, server [d]       PostgreSQL server
postgresql  15 [d]      client, server [d]       PostgreSQL server
```

---

# Interpretar los indicadores

La salida puede mostrar indicadores como:

| Indicador | Significado |
|-----------|-------------|
| `[d]` | Predeterminado |
| `[e]` | Habilitado |
| `[x]` | Deshabilitado |
| `[i]` | Instalado |

Ejemplo:

```text
15 [d][e]
```

Puede indicar que el stream:

- Es el predeterminado.
- Está habilitado.

---

# Consultar un módulo específico

```bash
dnf module list postgresql
```

Otro ejemplo:

```bash
dnf module list nodejs
```

---

# Mostrar información detallada

```bash
dnf module info postgresql
```

Para un stream específico:

```bash
dnf module info postgresql:15
```

La salida puede incluir:

- Nombre.
- Stream.
- Versión.
- Contexto.
- Arquitectura.
- Perfiles.
- Paquetes.
- Descripción.
- Dependencias del módulo.

---

# Consultar los perfiles

```bash
dnf module info postgresql:15
```

Busca la sección:

```text
Profiles
```

Ejemplo:

```text
client
server [d]
```

El indicador:

```text
[d]
```

identifica el perfil predeterminado.

---

# Habilitar un stream

Para habilitar una versión específica:

```bash
sudo dnf module enable postgresql:15
```

Otro ejemplo:

```bash
sudo dnf module enable nodejs:20
```

Este comando selecciona el stream que DNF debe utilizar.

---

# Verificar el stream habilitado

```bash
dnf module list postgresql
```

El stream habilitado puede aparecer con:

```text
[e]
```

---

# Instalar un módulo

Después de habilitar un stream:

```bash
sudo dnf module install postgresql:15
```

Este comando instala el perfil predeterminado.

---

# Habilitar e instalar en una sola operación

```bash
sudo dnf module install postgresql:15
```

DNF normalmente habilitará el stream e instalará su perfil predeterminado.

---

# Instalar un perfil específico

Sintaxis:

```bash
sudo dnf module install modulo:stream/perfil
```

Ejemplo:

```bash
sudo dnf module install postgresql:15/server
```

Otro ejemplo:

```bash
sudo dnf module install nodejs:20/development
```

---

# Sintaxis completa

```text
módulo:stream/perfil
```

Ejemplo:

```text
postgresql:15/server
```

Se interpreta como:

```text
Módulo: postgresql

Stream: 15

Perfil: server
```

---

# Instalar paquetes después de habilitar un stream

También es posible habilitar el stream y luego instalar paquetes individuales.

```bash
sudo dnf module enable postgresql:15
```

Después:

```bash
sudo dnf install postgresql-server
```

DNF utilizará la versión correspondiente al stream habilitado.

---

# Stream predeterminado

Algunos módulos tienen un stream predeterminado.

Puede identificarse con:

```text
[d]
```

Ejemplo:

```text
postgresql 15 [d]
```

Si instalas el módulo sin especificar el stream:

```bash
sudo dnf module install postgresql
```

DNF utilizará el stream predeterminado.

Para una administración más controlada, es recomendable indicar el stream explícitamente.

---

# Deshabilitar un módulo

```bash
sudo dnf module disable postgresql
```

Esto bloquea los streams del módulo.

Después de deshabilitarlo, sus paquetes modulares no estarán disponibles normalmente para nuevas instalaciones.

---

# Restablecer un módulo

```bash
sudo dnf module reset postgresql
```

El comando `reset` elimina la selección explícita de stream.

El módulo vuelve a su estado inicial.

---

# Diferencia entre disable y reset

| Acción | Resultado |
|--------|-----------|
| `disable` | Deshabilita todos los streams del módulo |
| `reset` | Elimina la selección actual y vuelve al estado inicial |
| `enable` | Selecciona un stream |
| `install` | Habilita e instala paquetes o perfiles |

---

# Flujo recomendado

```text
Consultar módulo
        │
        ▼
Revisar streams
        │
        ▼
Elegir versión
        │
        ▼
Habilitar stream
        │
        ▼
Elegir perfil
        │
        ▼
Instalar
        │
        ▼
Verificar
```

---

# Ejemplo con PostgreSQL

## Paso 1: Listar streams

```bash
dnf module list postgresql
```

---

## Paso 2: Consultar información

```bash
dnf module info postgresql:15
```

---

## Paso 3: Instalar el perfil servidor

```bash
sudo dnf module install postgresql:15/server
```

---

## Paso 4: Verificar paquetes

```bash
rpm -qa | grep postgresql
```

---

## Paso 5: Inicializar la base de datos

Dependiendo de la versión de RHEL y del paquete instalado:

```bash
sudo postgresql-setup --initdb
```

---

## Paso 6: Habilitar el servicio

```bash
sudo systemctl enable --now postgresql
```

---

## Paso 7: Verificar

```bash
systemctl status postgresql
```

---

# Ejemplo con Node.js

## Consultar streams

```bash
dnf module list nodejs
```

---

## Consultar un stream

```bash
dnf module info nodejs:20
```

---

## Instalar

```bash
sudo dnf module install nodejs:20
```

---

## Verificar

```bash
node --version
```

```bash
npm --version
```

---

# Ejemplo con PHP

Consultar streams:

```bash
dnf module list php
```

Habilitar un stream:

```bash
sudo dnf module enable php:8.2
```

Instalar:

```bash
sudo dnf module install php:8.2
```

Verificar:

```bash
php --version
```

> Los streams disponibles dependen de la versión de RHEL y de los repositorios habilitados.

---

# Cambiar de stream

Cambiar de stream requiere precaución.

Ejemplo conceptual:

```text
postgresql:13

↓

postgresql:15
```

No siempre es suficiente ejecutar:

```bash
dnf module enable postgresql:15
```

si ya existe otro stream instalado.

---

# Procedimiento general

Primero identifica el estado actual:

```bash
dnf module list postgresql
```

Consulta los paquetes instalados:

```bash
rpm -qa | grep postgresql
```

Después puede ser necesario:

```bash
sudo dnf module reset postgresql
```

Habilitar el nuevo stream:

```bash
sudo dnf module enable postgresql:15
```

Sincronizar paquetes:

```bash
sudo dnf distro-sync
```

---

# module switch-to

En entornos donde esté disponible, puede utilizarse:

```bash
sudo dnf module switch-to postgresql:15
```

Este comando intenta cambiar al nuevo stream y sincronizar los paquetes relacionados.

Antes de ejecutarlo:

- Realiza respaldo.
- Consulta dependencias.
- Revisa la transacción.
- Verifica la compatibilidad de la aplicación.

---

# Advertencia sobre bases de datos

Cambiar el stream de una base de datos no equivale necesariamente a actualizar sus datos internos.

Por ejemplo, cambiar paquetes de PostgreSQL:

```text
13 → 15
```

puede requerir:

- Respaldo lógico.
- Migración.
- `pg_upgrade`.
- Restauración.
- Validación de extensiones.
- Pruebas de compatibilidad.

DNF actualiza paquetes, pero no siempre realiza la migración de los datos de la aplicación.

---

# Eliminar un módulo

```bash
sudo dnf module remove postgresql:15
```

También puede utilizarse:

```bash
sudo dnf module remove postgresql:15/server
```

Revisa siempre la lista de paquetes que serán eliminados.

---

# Eliminar paquetes instalados mediante un módulo

Después de eliminar los paquetes, puede restablecerse el estado:

```bash
sudo dnf module reset postgresql
```

---

# Consultar módulos habilitados

```bash
dnf module list --enabled
```

---

# Consultar módulos instalados

```bash
dnf module list --installed
```

---

# Consultar módulos deshabilitados

```bash
dnf module list --disabled
```

---

# Consultar únicamente un stream

```bash
dnf module list postgresql:15
```

---

# Instalar sin módulos

No todos los paquetes de AppStream utilizan modularidad.

AppStream también puede contener paquetes RPM tradicionales.

Ejemplo:

```bash
sudo dnf install paquete
```

Esto significa que:

```text
AppStream ≠ únicamente módulos
```

AppStream es el repositorio.

Los módulos son una de las formas de organizar parte de su contenido.

---

# AppStream y paquetes no modulares

```text
AppStream
    │
    ├── Paquetes modulares
    │     ├── Streams
    │     └── Perfiles
    │
    └── Paquetes no modulares
          └── RPM tradicionales
```

---

# Filtrado modular

Cuando se habilita un stream, DNF filtra los paquetes disponibles para evitar combinar versiones incompatibles.

Ejemplo:

```text
Stream habilitado: nodejs:20
```

DNF prioriza los paquetes correspondientes a Node.js 20 y evita seleccionar paquetes de otros streams incompatibles.

---

# ¿Qué significa "filtered out by modular filtering"?

Puede aparecer un mensaje similar a:

```text
package is filtered out by modular filtering
```

Esto normalmente significa que:

- Existe un stream habilitado incompatible.
- El paquete pertenece a otro stream.
- El módulo está deshabilitado.
- Los metadatos están desactualizados.
- Se mezclaron repositorios incompatibles.

---

# Diagnóstico del filtrado modular

Consultar módulos:

```bash
dnf module list
```

Buscar el módulo relacionado:

```bash
dnf module list nombre_modulo
```

Consultar paquetes disponibles:

```bash
dnf list available nombre_paquete
```

Limpiar metadatos:

```bash
sudo dnf clean all
sudo dnf makecache
```

Restablecer el módulo si es necesario:

```bash
sudo dnf module reset nombre_modulo
```

---

# Error: conflicto entre streams

Ejemplo:

```text
Cannot enable stream because another stream is enabled
```

Solución general:

```bash
sudo dnf module reset nombre_modulo
```

Después:

```bash
sudo dnf module enable nombre_modulo:nuevo_stream
```

---

# Error: módulo no encontrado

Ejemplo:

```text
Unable to resolve argument nombre_modulo
```

Posibles causas:

- AppStream no está habilitado.
- El módulo no existe en esa versión de RHEL.
- El nombre está mal escrito.
- Los metadatos están desactualizados.
- La suscripción no permite acceder al repositorio.

Verificar:

```bash
dnf repolist
```

Actualizar metadatos:

```bash
sudo dnf clean all
sudo dnf makecache --refresh
```

Buscar:

```bash
dnf module list | grep -i nombre
```

---

# Error: no hay perfiles disponibles

Algunos módulos pueden no tener perfiles definidos.

En ese caso:

```bash
sudo dnf module enable modulo:stream
```

Después instala los paquetes individuales:

```bash
sudo dnf install nombre_paquete
```

---

# Error: paquetes instalados de otro stream

Consultar:

```bash
dnf module list nombre_modulo
```

```bash
rpm -qa | grep nombre_paquete
```

Puede ser necesario ejecutar:

```bash
sudo dnf distro-sync
```

o:

```bash
sudo dnf module switch-to modulo:stream
```

Siempre revisando la transacción antes de confirmar.

---

# Consultar transacciones antes de confirmar

DNF muestra una vista previa con:

- Paquetes que se instalarán.
- Paquetes que se actualizarán.
- Paquetes que se eliminarán.
- Dependencias.
- Tamaño de descarga.

Antes de escribir:

```text
y
```

revisa cuidadosamente la operación.

---

# AppStream en RHEL 8 y RHEL 9

La forma exacta en que Red Hat distribuye los módulos puede variar entre versiones.

En algunos entornos RHEL 9:

- Existen menos módulos que en RHEL 8.
- Algunos paquetes se entregan directamente como RPM.
- Determinadas versiones se distribuyen mediante repositorios adicionales.

Por eso siempre debe verificarse el sistema real con:

```bash
dnf module list
```

No debe asumirse que todos los módulos están disponibles en todas las versiones.

---

# Consultar la versión de RHEL

```bash
cat /etc/redhat-release
```

También:

```bash
cat /etc/os-release
```

---

# Consultar el origen de un paquete

```bash
dnf info nombre_paquete
```

Busca:

```text
From repo
```

También puede utilizarse:

```bash
dnf repoquery \
--qf '%{name} %{version}-%{release} %{repoid}' \
nombre_paquete
```

---

# Verificar paquetes instalados

```bash
dnf list installed
```

Para una aplicación:

```bash
dnf list installed postgresql\*
```

También:

```bash
rpm -qa | grep postgresql
```

---

# Relación entre módulos y RPM

Los módulos no reemplazan a RPM.

Su función es organizar y seleccionar conjuntos de paquetes.

```text
Módulo
    │
    ├── Stream
    │
    ├── Perfil
    │
    └── Paquetes RPM
```

Al final, los paquetes son instalados mediante RPM.

---

# Relación entre módulos y DNF

DNF interpreta los metadatos modulares y decide qué paquetes instalar.

```text
Usuario
    │
    ▼
DNF
    │
    ├── Consulta AppStream
    ├── Lee módulos
    ├── Selecciona stream
    ├── Selecciona perfil
    ├── Resuelve dependencias
    └── Instala paquetes RPM
```

---

# Ventajas de AppStream

- Permite utilizar versiones diferentes de aplicaciones.
- Mantiene estable el sistema base.
- Facilita la administración del ciclo de vida.
- Evita compilar software manualmente.
- Reduce la dependencia de repositorios externos.
- Organiza aplicaciones por streams.
- Proporciona perfiles de instalación.
- Facilita la estandarización empresarial.

---

# Limitaciones

- No todos los paquetes son modulares.
- No todos los streams están disponibles en todas las versiones.
- Cambiar de stream puede requerir migración.
- Mezclar repositorios puede generar conflictos.
- La actualización de paquetes no siempre migra datos.
- Algunos streams tienen ciclos de soporte diferentes.

---

# Ciclo de vida de un stream

Cada stream puede tener su propio periodo de soporte.

Ejemplo conceptual:

```text
Stream A
    ├── Publicación
    ├── Actualizaciones
    ├── Correcciones
    └── Fin de soporte
```

Antes de elegir un stream en producción, debe verificarse:

- Periodo de soporte.
- Compatibilidad.
- Dependencias.
- Requisitos de la aplicación.
- Política de actualización.

---

# Buenas prácticas RHCSA

✔ Verificar los streams disponibles antes de instalar.

✔ Utilizar repositorios oficiales.

✔ Especificar el stream cuando sea importante controlar la versión.

✔ Revisar los perfiles disponibles.

✔ No cambiar de stream sin evaluar el impacto.

✔ Realizar respaldos antes de actualizar servidores de bases de datos.

✔ Limpiar los metadatos cuando existan inconsistencias.

✔ Revisar siempre la transacción de DNF.

✔ Evitar mezclar repositorios destinados a versiones diferentes de RHEL.

✔ Documentar el stream seleccionado en servidores de producción.

---

# Errores comunes

## Instalar sin revisar el stream predeterminado

Puede instalarse una versión diferente a la esperada.

Verifica primero:

```bash
dnf module list nombre_modulo
```

---

## Cambiar de stream directamente

Ejecutar solamente:

```bash
sudo dnf module enable nuevo_stream
```

puede fallar si otro stream ya está habilitado.

Primero puede ser necesario:

```bash
sudo dnf module reset nombre_modulo
```

---

## Confundir AppStream con módulos

AppStream es un repositorio.

Los módulos son una forma de organizar parte del contenido de AppStream.

---

## Pensar que cambiar paquetes migra los datos

Actualizar PostgreSQL, MariaDB o una aplicación similar puede requerir un proceso adicional de migración.

---

## Deshabilitar un módulo pensando que elimina paquetes

```bash
dnf module disable
```

no necesariamente desinstala los paquetes existentes.

---

## Restablecer un módulo pensando que elimina software

```bash
dnf module reset
```

solo modifica el estado del módulo.

No elimina automáticamente los paquetes instalados.

---

## Mezclar streams incompatibles

Esto puede producir:

- Dependencias rotas.
- Paquetes filtrados.
- Conflictos de versión.
- Aplicaciones inestables.

---

# Comandos importantes RHCSA

| Comando | Descripción |
|---------|-------------|
| `dnf module list` | Listar módulos |
| `dnf module list modulo` | Consultar un módulo |
| `dnf module info modulo` | Mostrar información |
| `dnf module enable modulo:stream` | Habilitar un stream |
| `dnf module install modulo:stream` | Instalar un módulo |
| `dnf module install modulo:stream/perfil` | Instalar un perfil |
| `dnf module disable modulo` | Deshabilitar un módulo |
| `dnf module reset modulo` | Restablecer el módulo |
| `dnf module remove modulo:stream` | Eliminar paquetes del módulo |
| `dnf module list --enabled` | Mostrar módulos habilitados |
| `dnf module list --installed` | Mostrar módulos instalados |
| `dnf module switch-to modulo:stream` | Cambiar de stream |
| `dnf distro-sync` | Sincronizar paquetes |
| `dnf clean all` | Limpiar la caché |
| `dnf makecache` | Regenerar metadatos |

---

# Resumen rápido

```text
AppStream
    │
    ├── Aplicaciones
    ├── Lenguajes
    ├── Servidores
    ├── Paquetes RPM
    └── Módulos
          │
          ├── Streams
          ├── Perfiles
          └── Paquetes
```

---

# Resumen

En esta lección aprendiste a:

- Comprender la función de AppStream.
- Diferenciar BaseOS y AppStream.
- Comprender los conceptos de módulo, stream y perfil.
- Consultar los módulos disponibles.
- Habilitar e instalar streams.
- Instalar perfiles específicos.
- Deshabilitar y restablecer módulos.
- Cambiar entre streams.
- Diagnosticar problemas de filtrado modular.
- Aplicar buenas prácticas en servidores RHEL.

---

# Laboratorio práctico RHCSA

## Escenario 1: Verificar AppStream

Consulta los repositorios habilitados:

```bash
dnf repolist
```

Comprueba que AppStream esté disponible.

Después, muestra los módulos:

```bash
dnf module list
```

---

## Escenario 2: Explorar PostgreSQL

Consulta los streams disponibles:

```bash
dnf module list postgresql
```

Muestra la información de uno de ellos:

```bash
dnf module info postgresql:15
```

> Sustituye `15` por un stream que exista en tu sistema.

---

## Escenario 3: Habilitar un stream

Habilita el stream elegido:

```bash
sudo dnf module enable postgresql:15
```

Verifica:

```bash
dnf module list postgresql
```

Identifica el indicador:

```text
[e]
```

---

## Escenario 4: Instalar un perfil

Consulta los perfiles:

```bash
dnf module info postgresql:15
```

Instala el perfil servidor:

```bash
sudo dnf module install postgresql:15/server
```

Comprueba los paquetes instalados:

```bash
rpm -qa | grep postgresql
```

---

## Escenario 5: Consultar módulos instalados

```bash
dnf module list --installed
```

Consulta módulos habilitados:

```bash
dnf module list --enabled
```

---

## Escenario 6: Restablecer el módulo

Primero elimina los paquetes del laboratorio si corresponde:

```bash
sudo dnf module remove postgresql:15
```

Después restablece su estado:

```bash
sudo dnf module reset postgresql
```

Verifica:

```bash
dnf module list postgresql
```

---

## Escenario 7: Diagnóstico de filtrado modular

Busca un paquete:

```bash
dnf list available postgresql-server
```

Consulta los streams:

```bash
dnf module list postgresql
```

Limpia la caché:

```bash
sudo dnf clean all
sudo dnf makecache --refresh
```

Comprueba nuevamente:

```bash
dnf list available postgresql-server
```

---

# Preguntas de repaso

1. ¿Cuál es la diferencia entre BaseOS y AppStream?
2. ¿Qué es un módulo?
3. ¿Qué representa un stream?
4. ¿Qué función cumple un perfil?
5. ¿Qué comando lista los módulos disponibles?
6. ¿Qué diferencia existe entre `disable` y `reset`?
7. ¿Cómo se instala un perfil específico?
8. ¿Por qué cambiar un stream de base de datos puede requerir una migración?
9. ¿Qué significa el filtrado modular?
10. ¿AppStream contiene únicamente módulos?

---

# Desafío final

Realiza las siguientes tareas sin consultar la documentación:

1. Lista todos los módulos.
2. Busca un módulo disponible.
3. Consulta sus streams.
4. Identifica el stream predeterminado.
5. Habilita un stream diferente.
6. Instala un perfil.
7. Verifica los paquetes instalados.
8. Elimina el módulo.
9. Restablece su estado.
10. Comprueba que no permanezca habilitado.

> **Objetivo general:** dominar la administración de **AppStream**, módulos, streams y perfiles en Red Hat Enterprise Linux. Estos conceptos permiten seleccionar versiones específicas de aplicaciones sin comprometer la estabilidad del sistema base y forman parte de las habilidades necesarias para administrar software profesionalmente y resolver tareas prácticas del examen **RHCSA**.