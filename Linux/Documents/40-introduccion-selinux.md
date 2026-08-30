# 40. Introducción a SELinux

> **Módulo 7: Seguridad del Sistema**  
> **Página 40 del Manual RHCSA**

---

# Objetivos de aprendizaje

Al finalizar esta lección serás capaz de:

- Comprender qué es SELinux.
- Entender por qué existe SELinux.
- Diferenciar DAC de MAC.
- Comprender cómo SELinux protege el sistema.
- Identificar los componentes principales de SELinux.
- Verificar el estado de SELinux.
- Comprender la terminología básica utilizada en Red Hat Enterprise Linux.

---

# Introducción

Uno de los componentes de seguridad más importantes de Red Hat Enterprise Linux es **SELinux (Security-Enhanced Linux)**.

SELinux proporciona una capa adicional de protección que limita lo que pueden hacer los procesos del sistema, incluso cuando estos poseen permisos tradicionales suficientes.

En otras palabras:

> **Aunque un usuario tenga permisos sobre un archivo, SELinux puede impedir el acceso si dicho acceso no está permitido por su política de seguridad.**

Esta característica convierte a SELinux en una de las tecnologías más importantes para proteger servidores Linux en entornos empresariales.

---

# ¿Qué es SELinux?

SELinux (**Security-Enhanced Linux**) es un sistema de **Control de Acceso Obligatorio (Mandatory Access Control - MAC)** integrado en el kernel de Linux.

Fue desarrollado originalmente por la **NSA (National Security Agency)** y actualmente es mantenido por la comunidad y utilizado por distribuciones empresariales como:

- Red Hat Enterprise Linux
- Rocky Linux
- AlmaLinux
- CentOS Stream
- Fedora

---

# ¿Por qué existe SELinux?

En Linux tradicional, la seguridad depende principalmente de:

- Usuarios
- Grupos
- Permisos (rwx)

Este modelo funciona correctamente, pero presenta una limitación importante.

Si un proceso obtiene permisos suficientes, puede acceder a cualquier archivo autorizado por esos permisos.

SELinux agrega una segunda capa de protección.

```
Permisos Linux

        +

SELinux

        =

Mayor Seguridad
```

---

# DAC vs MAC

## DAC (Discretionary Access Control)

Es el modelo tradicional de Linux.

El propietario del archivo decide quién puede acceder.

Ejemplo:

```
-rw-r-----

usuario

grupo
```

Los permisos se controlan mediante:

- chmod
- chown
- chgrp

---

## MAC (Mandatory Access Control)

SELinux utiliza este modelo.

Las reglas son definidas por una política de seguridad.

Ni siquiera el propietario del archivo puede ignorarlas.

---

# Comparación DAC vs MAC

| DAC | MAC |
|------|------|
| Basado en permisos | Basado en políticas |
| El propietario controla el acceso | El sistema controla el acceso |
| Flexible | Más seguro |
| Puede ser insuficiente ante ataques | Limita el impacto de procesos comprometidos |

---

# ¿Cómo funciona SELinux?

Cada proceso y cada objeto del sistema poseen un **Contexto de Seguridad**.

Cuando un proceso intenta acceder a un recurso:

```
Proceso

↓

SELinux verifica la política

↓

¿Permitido?

↓

Sí → Acceso

No → Acceso denegado
```

Todo ocurre antes de que la aplicación acceda al recurso.

---

# Ejemplo práctico

Supongamos un servidor Apache.

```
Apache

↓

/etc/passwd
```

Aunque Apache tenga permisos de lectura mediante DAC, SELinux puede impedir el acceso porque la política no permite que el servicio web lea ese archivo.

---

# Componentes principales de SELinux

SELinux está compuesto por varios elementos.

## Políticas (Policies)

Definen qué acciones están permitidas.

---

## Contextos (Security Contexts)

Identifican el tipo de cada archivo, proceso o puerto.

---

## Booleanos

Permiten modificar ciertos comportamientos sin cambiar la política.

---

## Módulos

Extienden o personalizan las políticas de SELinux.

---

# Arquitectura simplificada

```
Aplicación

↓

Kernel

↓

SELinux

↓

Política

↓

Archivo o recurso
```

---

# Verificar si SELinux está instalado

```bash
rpm -q selinux-policy
```

También puede consultarse:

```bash
rpm -qa | grep selinux
```

---

# Verificar el estado de SELinux

```bash
sestatus
```

Ejemplo:

```
SELinux status:                 enabled

Current mode:                   enforcing

Loaded policy name:             targeted
```

---

# Obtener únicamente el modo actual

```bash
getenforce
```

Resultado:

```
Enforcing
```

---

# ¿Qué significa "Targeted"?

La política **targeted** protege principalmente los servicios del sistema.

Ejemplos:

- Apache
- SSH
- PostgreSQL
- DNS
- MariaDB
- Samba

Es la política predeterminada en RHEL.

---

# Ver el archivo de configuración

```bash
cat /etc/selinux/config
```

Ejemplo:

```text
SELINUX=enforcing

SELINUXTYPE=targeted
```

---

# Modos de funcionamiento

SELinux puede trabajar en tres modos.

| Modo | Descripción |
|-------|-------------|
| Enforcing | Aplica las políticas y bloquea accesos no permitidos. |
| Permissive | No bloquea, pero registra las violaciones en los logs. |
| Disabled | SELinux está deshabilitado. No se recomienda en producción. |

Estos modos se estudiarán en detalle en la siguiente lección.

---

# Beneficios de SELinux

- Reduce el impacto de aplicaciones comprometidas.
- Limita la propagación de ataques.
- Protege servicios críticos.
- Aumenta la seguridad del sistema.
- Está integrado en el kernel.
- Es altamente configurable.

---

# Ejemplo de protección

Servidor Web.

```
Internet

↓

Apache

↓

SELinux

↓

Solo puede acceder

↓

/var/www
```

Aunque el proceso sea comprometido, seguirá limitado por la política de SELinux.

---

# Mitos sobre SELinux

## "SELinux rompe las aplicaciones"

Falso.

Generalmente el problema es una configuración incorrecta o un contexto de seguridad inadecuado.

---

## "Es mejor desactivarlo"

No.

La práctica recomendada por Red Hat es mantener SELinux habilitado y solucionar los problemas ajustando contextos o políticas.

---

## "Solo sirve para servidores grandes"

Incorrecto.

SELinux protege tanto servidores pequeños como infraestructuras empresariales.

---

# Herramientas relacionadas

| Herramienta | Función |
|-------------|---------|
| `sestatus` | Estado de SELinux |
| `getenforce` | Mostrar el modo actual |
| `setenforce` | Cambiar temporalmente el modo |
| `ls -Z` | Mostrar contextos de archivos |
| `ps -eZ` | Mostrar contextos de procesos |
| `id -Z` | Mostrar el contexto del usuario |
| `restorecon` | Restaurar contextos |
| `semanage` | Administrar políticas y contextos |
| `getsebool` | Consultar booleanos |
| `setsebool` | Modificar booleanos |

---

# Buenas prácticas RHCSA

✔ Mantener SELinux en modo **Enforcing**.

✔ No deshabilitar SELinux para resolver problemas.

✔ Utilizar `restorecon` cuando existan errores de contexto.

✔ Consultar los registros antes de modificar políticas.

✔ Comprender la diferencia entre permisos Linux y políticas SELinux.

✔ Documentar cualquier cambio realizado sobre SELinux.

---

# Errores comunes

## Deshabilitar SELinux inmediatamente

Es uno de los errores más frecuentes.

Lo correcto es identificar la causa del bloqueo.

---

## Confundir permisos con SELinux

Un archivo puede tener permisos **777** y aun así ser bloqueado por SELinux.

---

## Ignorar los registros

La mayoría de los problemas pueden diagnosticarse revisando los eventos registrados por SELinux.

---

## Modificar políticas sin comprenderlas

Antes de crear reglas personalizadas es recomendable entender los contextos y booleanos existentes.

---

# Comandos importantes RHCSA

| Comando | Descripción |
|----------|-------------|
| `sestatus` | Estado completo de SELinux |
| `getenforce` | Mostrar modo actual |
| `cat /etc/selinux/config` | Configuración permanente |
| `rpm -qa | grep selinux` | Paquetes relacionados |
| `ls -Z` | Contextos de archivos |
| `ps -eZ` | Contextos de procesos |
| `id -Z` | Contexto del usuario |

---

# Resumen

En esta lección aprendiste a:

- Comprender qué es SELinux.
- Diferenciar DAC y MAC.
- Entender el funcionamiento básico de SELinux.
- Identificar sus principales componentes.
- Verificar su estado.
- Reconocer la importancia de mantener SELinux habilitado en sistemas Red Hat Enterprise Linux.

---

# Laboratorio práctico RHCSA

## Escenario 1

Comprueba el estado completo de SELinux.

```bash
sestatus
```

Identifica:

- Estado.
- Modo actual.
- Tipo de política cargada.

---

## Escenario 2

Consulta únicamente el modo de funcionamiento.

```bash
getenforce
```

---

## Escenario 3

Visualiza el archivo de configuración.

```bash
cat /etc/selinux/config
```

Identifica los parámetros:

- `SELINUX`
- `SELINUXTYPE`

---

## Escenario 4

Lista los paquetes relacionados con SELinux.

```bash
rpm -qa | grep selinux
```

---

## Escenario 5

Consulta el contexto de seguridad de tu usuario actual.

```bash
id -Z
```

Luego observa el contexto de algunos archivos del sistema.

```bash
ls -Z /etc/passwd

ls -Z /var/www
```

> **Objetivo general:** comprender el propósito y funcionamiento de **SELinux**, diferenciándolo del sistema tradicional de permisos de Linux. Este conocimiento servirá como base para estudiar los modos de operación, los contextos de seguridad, los booleanos y la resolución de problemas en las siguientes lecciones del módulo.