# 42. Contextos de Seguridad (Security Contexts) en SELinux

> **Módulo 7: Seguridad del Sistema**  
> **Página 42 del Manual RHCSA**

---

# Objetivos de aprendizaje

Al finalizar esta lección serás capaz de:

- Comprender qué es un Contexto de Seguridad en SELinux.
- Interpretar los cuatro componentes de un contexto.
- Consultar contextos de archivos, directorios, procesos y usuarios.
- Comprender cómo SELinux toma decisiones basadas en los contextos.
- Identificar problemas comunes relacionados con contextos incorrectos.

---

# Introducción

Los **Contextos de Seguridad (Security Contexts)** son el elemento más importante de SELinux.

Cada:

- Archivo
- Directorio
- Proceso
- Puerto
- Usuario

posee un contexto que identifica su función dentro del sistema.

Cuando un proceso intenta acceder a un recurso, SELinux compara ambos contextos y consulta su política para decidir si el acceso está permitido.

En la mayoría de los problemas relacionados con SELinux, **el origen del problema suele ser un contexto incorrecto**.

---

# ¿Qué es un Contexto de Seguridad?

Un contexto es una etiqueta que identifica un objeto dentro de SELinux.

Ejemplo:

```
system_u:object_r:httpd_sys_content_t:s0
```

Aunque pueda parecer complejo al principio, cada parte tiene un significado específico.

---

# Estructura de un Contexto

```
usuario : rol : tipo : nivel
```

Ejemplo:

```
system_u:object_r:httpd_sys_content_t:s0
```

```
┌──────────┐
│system_u  │ Usuario SELinux
└──────────┘

        │

┌──────────┐
│object_r  │ Rol
└──────────┘

        │

┌─────────────────────────┐
│httpd_sys_content_t       │ Tipo
└─────────────────────────┘

        │

┌──────┐
│ s0   │ Nivel MLS/MCS
└──────┘
```

---

# El componente más importante

En la práctica, el elemento más importante es el **Tipo (Type)**.

Ejemplo:

```
httpd_sys_content_t
```

Este tipo indica que el archivo puede ser utilizado por el servidor web Apache.

Cuando SELinux decide si un acceso está permitido, normalmente basa su decisión en este campo.

---

# Contextos de Archivos

Consultar un archivo:

```bash
ls -Z /etc/passwd
```

Ejemplo:

```
system_u:object_r:passwd_file_t:s0
```

---

Consultar varios archivos:

```bash
ls -Z
```

---

Consultar un directorio completo:

```bash
ls -lZ /var/www
```

Ejemplo:

```
drwxr-xr-x.

system_u:object_r:httpd_sys_content_t:s0
```

---

# Contextos de Procesos

Consultar procesos:

```bash
ps -eZ
```

Ejemplo:

```
system_u:system_r:httpd_t:s0
```

Buscar únicamente Apache:

```bash
ps -eZ | grep httpd
```

---

Buscar PostgreSQL:

```bash
ps -eZ | grep postgres
```

---

Buscar SSH:

```bash
ps -eZ | grep sshd
```

---

# Contexto del Usuario

Consultar el contexto del usuario actual:

```bash
id -Z
```

Ejemplo:

```
unconfined_u:unconfined_r:unconfined_t:s0
```

---

# Contextos de Directorios

Ejemplo:

```bash
ls -ldZ /var/www/html
```

Resultado:

```
system_u:object_r:httpd_sys_content_t:s0
```

---

# ¿Cómo decide SELinux?

Supongamos el siguiente escenario:

```
Proceso Apache

↓

httpd_t

↓

Archivo

↓

httpd_sys_content_t
```

La política permite este acceso.

Resultado:

```
Acceso Permitido
```

---

Ahora otro escenario:

```
Proceso Apache

↓

httpd_t

↓

Archivo

↓

user_home_t
```

Resultado:

```
Acceso Denegado
```

Aunque los permisos Linux sean correctos.

---

# Ejemplo práctico

Supongamos que copiamos una página web desde nuestro directorio personal.

```
cp index.html /var/www/html
```

Los permisos Linux son correctos.

Pero:

```bash
ls -Z /var/www/html/index.html
```

Resultado:

```
user_home_t
```

Apache espera:

```
httpd_sys_content_t
```

Como los tipos no coinciden:

```
403 Forbidden
```

---

# Tipos comunes

| Tipo | Descripción |
|-------|-------------|
| `httpd_sys_content_t` | Contenido del servidor web |
| `httpd_sys_rw_content_t` | Contenido web con escritura |
| `ssh_home_t` | Archivos SSH del usuario |
| `user_home_t` | Directorio personal |
| `var_log_t` | Archivos de log |
| `tmp_t` | Archivos temporales |
| `etc_t` | Configuración del sistema |
| `passwd_file_t` | Archivo `/etc/passwd` |
| `postgresql_db_t` | Archivos de PostgreSQL |
| `mysqld_db_t` | Archivos de MariaDB/MySQL |

---

# Consultar el contexto de un enlace simbólico

```bash
ls -lZ enlace
```

---

# Consultar el contexto de varios archivos

```bash
ls -ZR /var/www
```

---

# Consultar procesos con formato completo

```bash
ps auxZ
```

---

# ¿Qué sucede cuando el contexto es incorrecto?

Ejemplo:

```
Permisos Linux

777

↓

Apache

↓

SELinux

↓

Contexto incorrecto

↓

Acceso Denegado
```

Esto demuestra que SELinux es independiente del sistema tradicional de permisos.

---

# Diferencia entre permisos y contextos

Archivo:

```
-rwxrwxrwx
```

Permisos Linux:

```
777
```

Contexto:

```
user_home_t
```

Resultado:

Apache seguirá sin poder acceder porque el contexto es incorrecto.

---

# Restaurar un contexto

Cuando un archivo tiene un contexto incorrecto se utiliza:

```bash
restorecon
```

Ejemplo:

```bash
sudo restorecon -Rv /var/www/html
```

Este comando restaura el contexto correcto según la política de SELinux.

El uso de `restorecon` y `semanage` se estudiará en la siguiente lección.

---

# Flujo de validación de SELinux

```
Proceso

↓

Contexto del proceso

↓

Archivo

↓

Contexto del archivo

↓

Política SELinux

↓

¿Permitido?

↓

Sí

↓

Acceso

o

↓

No

↓

Acceso Denegado
```

---

# Herramientas relacionadas

| Herramienta | Función |
|-------------|---------|
| `ls -Z` | Contexto de archivos |
| `ls -lZ` | Contexto detallado |
| `ps -eZ` | Contexto de procesos |
| `ps auxZ` | Procesos con contexto |
| `id -Z` | Contexto del usuario |
| `restorecon` | Restaurar contextos |
| `semanage` | Administrar contextos permanentes |

---

# Buenas prácticas RHCSA

✔ Verificar siempre el contexto antes de modificar permisos.

✔ Utilizar `restorecon` para restaurar contextos predeterminados.

✔ Evitar el uso de `chmod 777` como solución a problemas de acceso.

✔ Mantener los archivos de las aplicaciones con el tipo correcto.

✔ Verificar el contexto de los procesos cuando un servicio no funciona.

---

# Errores comunes

## Cambiar únicamente los permisos

Muchas veces el problema no son los permisos Linux, sino el contexto SELinux.

---

## Copiar archivos desde el directorio personal

Los archivos pueden conservar un contexto como:

```
user_home_t
```

cuando deberían tener:

```
httpd_sys_content_t
```

---

## Usar `chcon` sin conocer sus implicaciones

`chcon` modifica el contexto de forma temporal.

Después de ejecutar `restorecon`, el cambio se pierde.

Para cambios permanentes debe utilizarse `semanage`, tema de la próxima lección.

---

## Deshabilitar SELinux

No es necesario deshabilitar SELinux para resolver un problema de contexto.

---

# Comandos importantes RHCSA

| Comando | Descripción |
|----------|-------------|
| `ls -Z` | Mostrar contexto de archivos |
| `ls -lZ` | Mostrar contexto detallado |
| `ls -ZR` | Mostrar contextos recursivamente |
| `ps -eZ` | Mostrar contexto de procesos |
| `ps auxZ` | Procesos con contexto |
| `id -Z` | Contexto del usuario |
| `restorecon` | Restaurar contextos predeterminados |

---

# Resumen

En esta lección aprendiste a:

- Comprender qué es un Contexto de Seguridad.
- Interpretar los componentes de un contexto SELinux.
- Consultar contextos de archivos, procesos y usuarios.
- Comprender cómo SELinux toma decisiones basadas en los contextos.
- Identificar problemas provocados por contextos incorrectos.
- Conocer la importancia de `restorecon` para restaurar etiquetas de seguridad.

---

# Laboratorio práctico RHCSA

## Escenario 1

Consulta el contexto de tu directorio personal.

```bash
ls -ldZ ~
```

---

## Escenario 2

Consulta el contexto de los archivos del servidor web.

```bash
ls -lZ /var/www/html
```

---

## Escenario 3

Visualiza el contexto de todos los procesos en ejecución.

```bash
ps -eZ
```

Busca los procesos de:

- `sshd`
- `httpd`
- `postgres`

---

## Escenario 4

Consulta el contexto SELinux de tu usuario.

```bash
id -Z
```

---

## Escenario 5

Restaura los contextos predeterminados del directorio web.

```bash
sudo restorecon -Rv /var/www/html
```

Después, verifica nuevamente los contextos:

```bash
ls -lZ /var/www/html
```

> **Objetivo general:** dominar el concepto de **Contextos de Seguridad** en SELinux y comprender cómo influyen en las decisiones de acceso del sistema. Este conocimiento es fundamental para administrar correctamente servidores Red Hat Enterprise Linux y constituye una de las competencias más evaluadas en el examen **RHCSA**. En la siguiente lección aprenderás a administrar contextos permanentes utilizando `restorecon`, `chcon` y `semanage`.