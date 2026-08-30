# 52. Administración de Paquetes RPM

> **Módulo 8: Administración del Software y Repositorios**  
> **Página 52 del Manual RHCSA**

---

# Objetivos de aprendizaje

Al finalizar esta lección serás capaz de:

- Comprender qué es RPM.
- Instalar paquetes locales utilizando RPM.
- Consultar la información de paquetes instalados.
- Verificar la integridad de un paquete.
- Identificar a qué paquete pertenece un archivo.
- Comprender las limitaciones de RPM frente a DNF.
- Aplicar buenas prácticas durante la administración de paquetes.

---

# Introducción

El formato **RPM (RPM Package Manager)** constituye la base del sistema de administración de software en Red Hat Enterprise Linux.

Aunque normalmente utilizaremos **DNF** para instalar aplicaciones, es fundamental comprender cómo funciona RPM, ya que DNF utiliza esta herramienta internamente.

RPM permite trabajar directamente con paquetes locales sin necesidad de acceder a un repositorio.

---

# ¿Qué es RPM?

RPM significa:

```
RPM Package Manager
```

Es una herramienta encargada de:

- Instalar paquetes.
- Actualizar paquetes.
- Eliminar paquetes.
- Consultar información.
- Verificar la integridad del software.
- Mantener la base de datos de paquetes instalados.

---

# ¿Qué es un paquete RPM?

Un paquete RPM es un archivo con extensión:

```
.rpm
```

Ejemplo:

```
httpd-2.4.62-1.el9.x86_64.rpm
```

Normalmente el nombre contiene:

```
Programa

↓

Versión

↓

Release

↓

Arquitectura
```

---

# Anatomía de un paquete

Ejemplo:

```
vim-enhanced-9.0.123-5.el9.x86_64.rpm
```

```
vim-enhanced

↓

Nombre

9.0.123

↓

Versión

5.el9

↓

Release

x86_64

↓

Arquitectura
```

---

# ¿Cuándo utilizar RPM?

RPM es útil cuando:

- Se dispone de un paquete local.
- No existe acceso a Internet.
- Se trabaja con repositorios internos.
- Se desea consultar información de un paquete.
- Es necesario verificar archivos instalados.

Para instalaciones normales se recomienda utilizar **DNF**.

---

# Verificar la versión de RPM

```bash
rpm --version
```

Ejemplo:

```
RPM version 4.18
```

---

# Instalar un paquete local

```bash
sudo rpm -ivh paquete.rpm
```

Ejemplo:

```bash
sudo rpm -ivh htop.rpm
```

---

# Significado de las opciones

| Opción | Descripción |
|---------|-------------|
| `-i` | Instalar |
| `-v` | Mostrar información detallada |
| `-h` | Mostrar barra de progreso |

---

# Flujo de instalación

```
Paquete RPM

↓

rpm -ivh

↓

Base de datos RPM

↓

Sistema
```

---

# Actualizar un paquete

```bash
sudo rpm -Uvh paquete.rpm
```

La opción:

```
-U
```

significa:

```
Upgrade
```

Si el paquete ya existe:

```
Actualiza
```

Si no existe:

```
Instala
```

---

# Reinstalar un paquete

```bash
sudo rpm -Uvh --replacepkgs paquete.rpm
```

---

# Forzar una instalación

```bash
sudo rpm -ivh --force paquete.rpm
```

⚠ **No se recomienda**, salvo que exista una razón técnica justificada.

---

# Eliminar un paquete

```bash
sudo rpm -e nombre_paquete
```

Ejemplo:

```bash
sudo rpm -e htop
```

No se utiliza el nombre del archivo `.rpm`, sino el nombre del paquete instalado.

---

# Consultar paquetes instalados

Mostrar todos:

```bash
rpm -qa
```

---

Buscar un paquete:

```bash
rpm -qa | grep httpd
```

---

Contar paquetes instalados:

```bash
rpm -qa | wc -l
```

---

# Obtener información de un paquete instalado

```bash
rpm -qi httpd
```

Ejemplo de salida:

```
Name

Version

Release

Architecture

Install Date

License

Summary
```

---

# Consultar los archivos instalados por un paquete

```bash
rpm -ql httpd
```

Ejemplo:

```
/usr/sbin/httpd

/etc/httpd

/usr/lib/systemd/system/httpd.service
```

---

# Consultar la documentación

```bash
rpm -qd httpd
```

---

# Consultar archivos de configuración

```bash
rpm -qc httpd
```

---

# ¿Qué paquete instaló este archivo?

Ejemplo:

```bash
rpm -qf /usr/sbin/httpd
```

Resultado:

```
httpd
```

Muy útil para identificar el origen de un archivo.

---

# Verificar un paquete instalado

```bash
rpm -V httpd
```

Si no aparece ninguna salida:

```
El paquete no presenta modificaciones.
```

---

# Interpretar la verificación

Si un archivo fue modificado, puede aparecer:

```
S

M

5

T
```

Significado:

| Código | Descripción |
|----------|-------------|
| `S` | Tamaño diferente |
| `M` | Permisos modificados |
| `5` | Checksum diferente |
| `T` | Fecha modificada |
| `U` | Usuario diferente |
| `G` | Grupo diferente |

---

# Consultar dependencias

```bash
rpm -qR httpd
```

Resultado:

```
libc.so

libssl.so

systemd
```

---

# Consultar qué paquete requiere otro

```bash
rpm -q --whatrequires openssl
```

---

# Consultar qué paquete proporciona un archivo

```bash
rpm -q --whatprovides /usr/bin/python3
```

---

# Consultar un paquete sin instalarlo

Mostrar información:

```bash
rpm -qip paquete.rpm
```

---

Mostrar archivos:

```bash
rpm -qlp paquete.rpm
```

---

Consultar dependencias:

```bash
rpm -qpR paquete.rpm
```

Estas opciones son muy útiles antes de instalar un paquete descargado manualmente.

---

# Verificar la firma de un paquete

```bash
rpm -K paquete.rpm
```

Ejemplo:

```
digests signatures OK
```

Este tema se ampliará en la lección sobre firmas GPG.

---

# Base de datos RPM

Toda la información se almacena automáticamente.

Consultar un paquete:

```bash
rpm -qi bash
```

No es necesario acceder directamente a la base de datos.

---

# Limitaciones de RPM

RPM **no resuelve automáticamente las dependencias**.

Ejemplo:

```
Aplicación

↓

Necesita

↓

Biblioteca

↓

No instalada

↓

Instalación falla
```

Por ello, en instalaciones normales se recomienda utilizar DNF.

---

# Diferencia entre RPM y DNF

| RPM | DNF |
|------|-----|
| Trabaja con paquetes locales | Trabaja con repositorios |
| No resuelve dependencias automáticamente | Resuelve dependencias automáticamente |
| Instalación manual | Instalación automática |
| Consulta paquetes | Administración completa del software |
| Más básico | Más avanzado |

---

# Flujo recomendado

```
Repositorio

↓

DNF

↓

RPM

↓

Sistema
```

En la práctica:

- RPM para consultas y paquetes locales.
- DNF para instalaciones y actualizaciones.

---

# Herramientas relacionadas

| Herramienta | Función |
|-------------|----------|
| `rpm` | Administración de paquetes |
| `dnf` | Administración completa del software |
| `rpm2cpio` | Extraer contenido de un paquete RPM |
| `cpio` | Manipular archivos extraídos |

---

# Buenas prácticas RHCSA

✔ Utilizar DNF para instalaciones normales.

✔ Utilizar RPM para consultas y paquetes locales.

✔ Verificar las firmas digitales antes de instalar un paquete descargado.

✔ Revisar las dependencias antes de instalar software manualmente.

✔ No utilizar `--force` salvo que sea estrictamente necesario.

✔ No modificar manualmente la base de datos RPM.

---

# Errores comunes

## Instalar con RPM cuando existen dependencias

La instalación puede fallar porque RPM no descarga automáticamente los paquetes requeridos.

---

## Eliminar paquetes críticos

Puede afectar el funcionamiento del sistema operativo.

---

## Forzar instalaciones

El uso indiscriminado de:

```bash
--force
```

puede generar inconsistencias.

---

## Ignorar la verificación

Antes de instalar un paquete descargado manualmente, verifica su origen y su firma.

---

# Comandos importantes RHCSA

| Comando | Descripción |
|----------|-------------|
| `rpm --version` | Mostrar la versión de RPM |
| `rpm -ivh paquete.rpm` | Instalar un paquete |
| `rpm -Uvh paquete.rpm` | Actualizar un paquete |
| `rpm -e paquete` | Eliminar un paquete |
| `rpm -qa` | Listar paquetes instalados |
| `rpm -qi paquete` | Información del paquete |
| `rpm -ql paquete` | Archivos del paquete |
| `rpm -qc paquete` | Archivos de configuración |
| `rpm -qd paquete` | Documentación |
| `rpm -qf archivo` | Identificar el paquete propietario |
| `rpm -V paquete` | Verificar integridad |
| `rpm -qR paquete` | Mostrar dependencias |
| `rpm -K paquete.rpm` | Verificar firma del paquete |

---

# Resumen

En esta lección aprendiste a:

- Comprender el funcionamiento de RPM.
- Instalar y eliminar paquetes locales.
- Consultar información sobre paquetes instalados.
- Identificar los archivos pertenecientes a un paquete.
- Verificar la integridad del software.
- Comprender las diferencias entre RPM y DNF.
- Aplicar buenas prácticas durante la administración de paquetes.

---

# Laboratorio práctico RHCSA

## Escenario 1

Consulta la versión instalada de RPM.

```bash
rpm --version
```

---

## Escenario 2

Cuenta cuántos paquetes están instalados.

```bash
rpm -qa | wc -l
```

Después, muestra los primeros diez.

```bash
rpm -qa | head
```

---

## Escenario 3

Obtén información del paquete `bash`.

```bash
rpm -qi bash
```

Lista los archivos instalados por dicho paquete.

```bash
rpm -ql bash
```

---

## Escenario 4

Identifica a qué paquete pertenece el ejecutable de Bash.

```bash
rpm -qf /usr/bin/bash
```

Verifica la integridad del paquete.

```bash
rpm -V bash
```

---

## Escenario 5

Si dispones de un archivo `.rpm` descargado, consulta su información **sin instalarlo**.

```bash
rpm -qip paquete.rpm

rpm -qlp paquete.rpm

rpm -qpR paquete.rpm

rpm -K paquete.rpm
```

> **Objetivo general:** dominar el uso de **RPM** para instalar, consultar, verificar y administrar paquetes locales en Red Hat Enterprise Linux. Aunque en la administración diaria se utiliza principalmente **DNF**, conocer RPM es fundamental para comprender el funcionamiento interno del sistema de paquetes y constituye un tema frecuente en el examen **RHCSA**.