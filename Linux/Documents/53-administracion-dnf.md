# 53. Administración de Software con DNF

> **Módulo 8: Administración del Software y Repositorios**  
> **Página 53 del Manual RHCSA**

---

# Objetivos de aprendizaje

Al finalizar esta lección serás capaz de:

- Comprender qué es DNF y su función en RHEL.
- Instalar, actualizar y eliminar paquetes.
- Buscar software en los repositorios.
- Consultar información sobre paquetes.
- Resolver dependencias automáticamente.
- Administrar grupos de paquetes.
- Aplicar buenas prácticas durante la administración del software.

---

# Introducción

Aunque RPM es el sistema encargado de instalar paquetes, en la práctica los administradores utilizan **DNF** para administrar el software del sistema.

DNF automatiza tareas como:

- Resolver dependencias.
- Descargar paquetes.
- Verificar firmas GPG.
- Actualizar software.
- Consultar repositorios.
- Administrar grupos de paquetes.

Es la herramienta principal para la administración del software en **Red Hat Enterprise Linux 8 y 9**.

---

# ¿Qué es DNF?

DNF significa:

```
Dandified YUM
```

Es el administrador moderno de paquetes de Red Hat Enterprise Linux.

DNF trabaja sobre RPM, agregando funciones avanzadas como:

- Resolución automática de dependencias.
- Administración de repositorios.
- Descarga de actualizaciones.
- Manejo de AppStream.
- Historial de operaciones.

---

# Arquitectura

```
Administrador

↓

DNF

↓

Repositorios

↓

RPM

↓

Sistema Operativo
```

---

# Verificar la versión

```bash
dnf --version
```

Ejemplo:

```
4.14.0
```

---

# Obtener ayuda

```bash
dnf --help
```

También:

```bash
man dnf
```

---

# Buscar un paquete

```bash
dnf search httpd
```

Ejemplo:

```
httpd.x86_64

Apache HTTP Server
```

---

Buscar utilizando varias palabras:

```bash
dnf search web server
```

---

# Consultar información

```bash
dnf info httpd
```

Resultado:

```
Nombre

Versión

Repositorio

Arquitectura

Descripción

Tamaño

Licencia
```

---

# Instalar un paquete

```bash
sudo dnf install httpd
```

DNF:

- Consulta repositorios.
- Resuelve dependencias.
- Descarga paquetes.
- Verifica firmas.
- Instala automáticamente.

---

# Instalar varios paquetes

```bash
sudo dnf install httpd mariadb-server php
```

---

# Instalar desde un archivo RPM

```bash
sudo dnf install paquete.rpm
```

A diferencia de `rpm`, DNF intentará resolver automáticamente las dependencias necesarias.

---

# Confirmación automática

Para evitar preguntas:

```bash
sudo dnf install -y httpd
```

La opción:

```
-y
```

responde automáticamente:

```
Yes
```

---

# Actualizar un paquete

```bash
sudo dnf upgrade httpd
```

También:

```bash
sudo dnf update httpd
```

En RHEL 8 y 9 ambos comandos son equivalentes.

---

# Actualizar todo el sistema

```bash
sudo dnf update
```

o

```bash
sudo dnf upgrade
```

---

# Comprobar actualizaciones

```bash
dnf check-update
```

Muestra únicamente los paquetes con versiones disponibles.

---

# Eliminar un paquete

```bash
sudo dnf remove httpd
```

DNF también eliminará las dependencias que ya no sean necesarias cuando corresponda.

---

# Instalar únicamente una arquitectura

Ejemplo:

```bash
sudo dnf install glibc.x86_64
```

---

# Reinstalar un paquete

```bash
sudo dnf reinstall httpd
```

Muy útil cuando algún archivo del paquete fue eliminado o está dañado.

---

# Descargar sin instalar

```bash
sudo dnf download httpd
```

> Este comando requiere el complemento **dnf-plugins-core**.

---

# Mostrar los paquetes instalados

```bash
dnf list installed
```

---

Buscar un paquete instalado:

```bash
dnf list installed | grep httpd
```

---

# Mostrar paquetes disponibles

```bash
dnf list available
```

---

Mostrar todos:

```bash
dnf list all
```

---

# Mostrar un paquete específico

```bash
dnf list httpd
```

---

# Consultar dependencias

```bash
dnf repoquery --requires httpd
```

---

Consultar qué paquete proporciona un archivo.

```bash
dnf provides /usr/bin/ssh
```

También:

```bash
dnf provides "*/ssh"
```

---

# Buscar el paquete propietario de un comando

Ejemplo:

```bash
dnf provides /usr/bin/firewall-cmd
```

Resultado:

```
firewalld
```

---

# Limpiar la caché

```bash
sudo dnf clean all
```

Limpiar únicamente metadatos:

```bash
sudo dnf clean metadata
```

Limpiar paquetes descargados:

```bash
sudo dnf clean packages
```

---

# Regenerar la caché

```bash
sudo dnf makecache
```

---

# Trabajar con grupos de paquetes

Listar grupos disponibles:

```bash
dnf group list
```

---

Mostrar información:

```bash
dnf group info "Development Tools"
```

---

Instalar un grupo:

```bash
sudo dnf group install "Development Tools"
```

---

Eliminar un grupo:

```bash
sudo dnf group remove "Development Tools"
```

---

# Flujo de instalación

```
Usuario

↓

DNF

↓

Repositorio

↓

Dependencias

↓

RPM

↓

Sistema
```

---

# Diferencias entre RPM y DNF

| RPM | DNF |
|------|-----|
| Instala paquetes locales | Administra software completo |
| No resuelve dependencias automáticamente | Resuelve dependencias automáticamente |
| No administra repositorios | Administra repositorios |
| Funciones básicas | Funciones avanzadas |
| Administración manual | Administración automatizada |

---

# Herramientas relacionadas

| Herramienta | Función |
|-------------|----------|
| `dnf` | Administración del software |
| `rpm` | Administración de paquetes |
| `repoquery` | Consultar repositorios |
| `dnf history` | Historial (se estudiará más adelante) |
| `dnf config-manager` | Configuración de repositorios |

---

# Buenas prácticas RHCSA

✔ Utilizar DNF para instalar software.

✔ Mantener el sistema actualizado.

✔ Limpiar periódicamente la caché.

✔ Utilizar repositorios oficiales.

✔ Revisar la información de un paquete antes de instalarlo.

✔ Instalar únicamente el software necesario.

✔ Verificar las actualizaciones disponibles con frecuencia.

---

# Errores comunes

## Utilizar RPM para instalar software desde repositorios

Para ello debe utilizarse DNF.

---

## Eliminar paquetes críticos

Puede afectar el funcionamiento del sistema.

Antes de confirmar una eliminación, revisa cuidadosamente la lista de paquetes que DNF propone eliminar.

---

## Ignorar dependencias

Aunque DNF las administra automáticamente, es recomendable revisar qué paquetes adicionales serán instalados.

---

## No limpiar la caché

Con el tiempo pueden acumularse cientos de megabytes de información descargada.

---

## Mezclar repositorios incompatibles

Puede producir conflictos entre versiones de paquetes.

---

# Comandos importantes RHCSA

| Comando | Descripción |
|----------|-------------|
| `dnf --version` | Mostrar versión |
| `dnf search paquete` | Buscar paquetes |
| `dnf info paquete` | Información del paquete |
| `dnf install paquete` | Instalar software |
| `dnf install -y paquete` | Instalar sin confirmación |
| `dnf remove paquete` | Eliminar paquete |
| `dnf reinstall paquete` | Reinstalar paquete |
| `dnf update` | Actualizar sistema |
| `dnf upgrade` | Actualizar sistema |
| `dnf check-update` | Buscar actualizaciones |
| `dnf list installed` | Paquetes instalados |
| `dnf provides archivo` | Buscar paquete propietario |
| `dnf clean all` | Limpiar caché |
| `dnf makecache` | Regenerar caché |
| `dnf group list` | Listar grupos |
| `dnf group install` | Instalar grupo |

---

# Resumen

En esta lección aprendiste a:

- Comprender el funcionamiento de DNF.
- Buscar e instalar paquetes.
- Actualizar el sistema.
- Eliminar y reinstalar software.
- Consultar información sobre paquetes.
- Administrar grupos de paquetes.
- Aplicar buenas prácticas para la administración del software.

---

# Laboratorio práctico RHCSA

## Escenario 1

Consulta la versión de DNF.

```bash
dnf --version
```

Busca el paquete Apache.

```bash
dnf search httpd
```

---

## Escenario 2

Consulta la información del paquete.

```bash
dnf info httpd
```

Instálalo.

```bash
sudo dnf install httpd
```

Comprueba que quedó instalado.

```bash
dnf list installed httpd
```

---

## Escenario 3

Consulta qué paquete proporciona el comando `firewall-cmd`.

```bash
dnf provides /usr/bin/firewall-cmd
```

Después identifica el paquete que proporciona `sshd`.

```bash
dnf provides /usr/sbin/sshd
```

---

## Escenario 4

Consulta si existen actualizaciones disponibles.

```bash
dnf check-update
```

Limpia la caché del sistema y vuelve a generar los metadatos.

```bash
sudo dnf clean all

sudo dnf makecache
```

---

## Escenario 5

Explora los grupos de paquetes disponibles.

```bash
dnf group list
```

Obtén información del grupo **Development Tools**.

```bash
dnf group info "Development Tools"
```

> **Objetivo general:** dominar el uso de **DNF** para administrar el software en Red Hat Enterprise Linux. DNF es la herramienta principal para instalar, actualizar y eliminar paquetes, resolver dependencias automáticamente y trabajar con los repositorios oficiales, por lo que constituye una de las competencias fundamentales evaluadas en el examen **RHCSA**.