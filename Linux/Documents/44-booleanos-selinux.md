# 44. Booleanos de SELinux

> **Módulo 7: Seguridad del Sistema**  
> **Página 44 del Manual RHCSA**

---

# Objetivos de aprendizaje

Al finalizar esta lección serás capaz de:

- Comprender qué son los booleanos de SELinux.
- Consultar los booleanos disponibles.
- Identificar el estado de un booleano.
- Modificar booleanos temporal y permanentemente.
- Comprender cuándo utilizar un booleano en lugar de modificar contextos.
- Diagnosticar problemas comunes relacionados con booleanos.

---

# Introducción

En ocasiones un servicio necesita realizar una acción que **la política de SELinux no permite por defecto**.

Por ejemplo:

- Apache necesita conectarse a una base de datos.
- Apache debe enviar correos electrónicos.
- Un servidor FTP debe acceder a directorios personales.
- Samba debe compartir directorios específicos.

Modificar toda la política SELinux sería complejo.

Para resolver estos escenarios, SELinux incorpora los **Booleanos (SELinux Booleans)**.

Los booleanos permiten **activar o desactivar determinadas funcionalidades de forma sencilla y segura**, sin modificar la política principal.

---

# ¿Qué es un Booleano?

Un booleano es un parámetro que únicamente puede tener dos valores:

```
ON

o

OFF
```

También se representan como:

```
1

0
```

o

```
true

false
```

---

# ¿Cómo funcionan?

```
Aplicación

↓

Política SELinux

↓

Booleano

↓

Permitir o Denegar
```

Cuando un booleano cambia de estado, la política de SELinux adapta su comportamiento automáticamente.

---

# Ejemplo

Servidor Apache.

Por defecto:

```
Apache

↓

No puede conectarse a PostgreSQL
```

Activando un booleano:

```
Apache

↓

Puede conectarse a PostgreSQL
```

Todo ello sin modificar los contextos de seguridad.

---

# Consultar todos los booleanos

```bash
getsebool -a
```

Ejemplo:

```
httpd_can_network_connect --> off

httpd_enable_cgi --> on

ftp_home_dir --> off
```

---

# Buscar un booleano específico

Ejemplo:

```bash
getsebool -a | grep httpd
```

---

Buscar Samba:

```bash
getsebool -a | grep samba
```

---

Buscar FTP:

```bash
getsebool -a | grep ftp
```

---

# Consultar un único booleano

```bash
getsebool httpd_can_network_connect
```

Resultado:

```
httpd_can_network_connect --> off
```

---

# Cambiar un booleano temporalmente

Ejemplo:

```bash
sudo setsebool httpd_can_network_connect on
```

Comprobar:

```bash
getsebool httpd_can_network_connect
```

Resultado:

```
on
```

---

# Cambiar un booleano permanentemente

```bash
sudo setsebool -P httpd_can_network_connect on
```

La opción:

```
-P
```

guarda el cambio en la política y permanece después de reiniciar el sistema.

---

# Desactivar un booleano

Temporal:

```bash
sudo setsebool ftp_home_dir off
```

Permanente:

```bash
sudo setsebool -P ftp_home_dir off
```

---

# Diferencia entre temporal y permanente

## Temporal

```bash
sudo setsebool httpd_can_network_connect on
```

- No requiere reinicio.
- Se pierde tras reiniciar.

---

## Permanente

```bash
sudo setsebool -P httpd_can_network_connect on
```

- Sobrevive al reinicio.
- Modifica la política de SELinux.

---

# Consultar información detallada

```bash
semanage boolean -l
```

Ejemplo:

```
SELinux boolean

Description

Current State

Persistent State
```

Filtrar:

```bash
semanage boolean -l | grep httpd
```

---

# Booleanos comunes

## Apache

Permitir conexiones de red salientes.

```bash
httpd_can_network_connect
```

---

Permitir envío de correo.

```bash
httpd_can_sendmail
```

---

Permitir scripts CGI.

```bash
httpd_enable_cgi
```

---

Permitir acceso de escritura.

```bash
httpd_unified
```

---

# FTP

Permitir acceso a directorios personales.

```bash
ftp_home_dir
```

---

# Samba

Permitir compartir directorios personales.

```bash
samba_enable_home_dirs
```

---

Permitir exportar todos los directorios.

```bash
samba_export_all_rw
```

---

# NFS

Permitir exportar directorios.

```bash
use_nfs_home_dirs
```

---

# PostgreSQL

En la mayoría de los casos PostgreSQL funciona sin modificar booleanos.

Sin embargo, cuando Apache necesita conectarse a PostgreSQL, normalmente se utiliza:

```bash
httpd_can_network_connect
```

---

# Ejemplo práctico

Servidor Web.

Aplicación PHP intenta conectarse a PostgreSQL.

Todo parece correcto:

- Firewall
- PostgreSQL
- Usuario
- Contraseña

Pero la conexión falla.

Consultar:

```bash
getsebool httpd_can_network_connect
```

Resultado:

```
off
```

Activar:

```bash
sudo setsebool -P httpd_can_network_connect on
```

La aplicación puede conectarse correctamente.

---

# Flujo recomendado

```
Aplicación falla

↓

Verificar logs

↓

Consultar contexto

↓

Consultar booleanos

↓

Modificar booleano

↓

Probar nuevamente
```

---

# ¿Cuándo utilizar un booleano?

Utiliza un booleano cuando:

- La funcionalidad ya está contemplada por SELinux.
- No es necesario modificar contextos.
- La documentación recomienda habilitar un booleano específico.

No utilices un booleano para solucionar problemas de contextos incorrectos.

---

# Diferencia entre Contexto y Booleano

## Contexto

Define:

```
¿Qué es este archivo o proceso?
```

Ejemplo:

```
httpd_sys_content_t
```

---

## Booleano

Define:

```
¿Qué comportamiento está permitido?
```

Ejemplo:

```
httpd_can_network_connect
```

---

# Herramientas relacionadas

| Herramienta | Función |
|-------------|---------|
| `getsebool` | Consultar booleanos |
| `setsebool` | Modificar booleanos |
| `semanage boolean -l` | Información detallada |
| `sestatus` | Estado general de SELinux |
| `restorecon` | Restaurar contextos |
| `ls -Z` | Consultar contextos |

---

# Buenas prácticas RHCSA

✔ Revisar primero si existe un booleano antes de modificar la política.

✔ Utilizar `-P` únicamente cuando el cambio deba mantenerse.

✔ Documentar los booleanos modificados.

✔ Revisar la documentación oficial de Red Hat cuando una aplicación requiera un booleano específico.

✔ No activar booleanos innecesarios.

---

# Errores comunes

## Cambiar contextos cuando el problema es un booleano

No todos los problemas de SELinux se resuelven con `restorecon`.

---

## Activar todos los booleanos

Disminuye el nivel de seguridad.

Activa únicamente los necesarios.

---

## Olvidar la opción -P

El cambio desaparecerá tras reiniciar el sistema.

---

## Ignorar los registros

Antes de modificar un booleano, revisa los eventos registrados por SELinux.

---

# Comandos importantes RHCSA

| Comando | Descripción |
|----------|-------------|
| `getsebool -a` | Listar todos los booleanos |
| `getsebool nombre` | Consultar un booleano |
| `setsebool nombre on` | Activar temporalmente |
| `setsebool nombre off` | Desactivar temporalmente |
| `setsebool -P nombre on` | Activar permanentemente |
| `setsebool -P nombre off` | Desactivar permanentemente |
| `semanage boolean -l` | Información detallada |

---

# Resumen

En esta lección aprendiste a:

- Comprender qué son los booleanos de SELinux.
- Consultar el estado de un booleano.
- Activar y desactivar booleanos.
- Diferenciar cambios temporales y permanentes.
- Identificar cuándo utilizar un booleano en lugar de modificar contextos.
- Aplicar buenas prácticas durante la administración de SELinux.

---

# Laboratorio práctico RHCSA

## Escenario 1

Lista todos los booleanos del sistema.

```bash
getsebool -a
```

---

## Escenario 2

Consulta todos los booleanos relacionados con Apache.

```bash
getsebool -a | grep httpd
```

---

## Escenario 3

Consulta el estado del booleano que permite conexiones de red desde Apache.

```bash
getsebool httpd_can_network_connect
```

Actívalo temporalmente:

```bash
sudo setsebool httpd_can_network_connect on
```

Comprueba nuevamente su estado.

---

## Escenario 4

Haz permanente el cambio.

```bash
sudo setsebool -P httpd_can_network_connect on
```

Verifica el estado persistente:

```bash
semanage boolean -l | grep httpd_can_network_connect
```

---

## Escenario 5

Consulta los booleanos relacionados con FTP y Samba.

```bash
getsebool -a | grep ftp

getsebool -a | grep samba
```

Identifica cuáles están habilitados y analiza para qué podrían utilizarse.

> **Objetivo general:** aprender a administrar los **booleanos de SELinux**, comprendiendo cómo modifican el comportamiento de la política de seguridad sin alterar los contextos. Esta es una habilidad muy utilizada en entornos empresariales y frecuentemente evaluada en el examen **RHCSA**.