# 51. Introducción a la Administración del Software en Red Hat Enterprise Linux

> **Módulo 8: Administración del Software y Repositorios**  
> **Página 51 del Manual RHCSA**

---

# Objetivos de aprendizaje

Al finalizar esta lección serás capaz de:

- Comprender cómo se administra el software en Red Hat Enterprise Linux.
- Conocer los principales componentes del sistema de paquetes.
- Comprender la relación entre RPM, DNF, AppStream y los repositorios.
- Diferenciar la instalación de software en Linux respecto a Windows.
- Identificar el flujo completo de administración de software en RHEL.
- Prepararte para administrar paquetes de manera segura y eficiente.

---

# Introducción

Todo sistema operativo necesita una forma organizada de instalar, actualizar y eliminar aplicaciones.

En Red Hat Enterprise Linux este proceso está basado en un sistema robusto compuesto por varias tecnologías que trabajan juntas.

Las principales son:

- RPM
- DNF
- Repositorios
- AppStream
- Firmas GPG

Cada una cumple una función específica.

Durante este módulo aprenderás cómo interactúan para administrar miles de paquetes de forma segura.

---

# ¿Qué es un paquete?

Un paquete es un archivo que contiene todo lo necesario para instalar una aplicación.

Generalmente incluye:

- Archivos ejecutables
- Bibliotecas
- Archivos de configuración
- Documentación
- Scripts de instalación
- Información sobre dependencias

En RHEL los paquetes tienen extensión:

```
.rpm
```

Ejemplo:

```
httpd-2.4.62-1.el9.x86_64.rpm
```

---

# Componentes de la administración del software

```
              Usuario
                  │
                  ▼
                DNF
                  │
        ┌─────────┴─────────┐
        ▼                   ▼
   Repositorios          Paquetes RPM
        │                   │
        └─────────┬─────────┘
                  ▼
             Base de datos RPM
                  │
                  ▼
             Sistema Operativo
```

---

# ¿Qué es RPM?

RPM significa:

```
RPM Package Manager
```

Es el sistema encargado de:

- Instalar paquetes locales.
- Consultar paquetes instalados.
- Verificar archivos.
- Obtener información de paquetes.
- Mantener la base de datos de paquetes instalados.

RPM trabaja directamente con archivos:

```
archivo.rpm
```

---

# ¿Qué es DNF?

DNF significa:

```
Dandified YUM
```

Es el administrador moderno de paquetes utilizado por Red Hat Enterprise Linux.

DNF utiliza RPM internamente.

Su principal ventaja es que resuelve automáticamente las dependencias.

---

# Relación entre RPM y DNF

```
Usuario

↓

DNF

↓

RPM

↓

Sistema
```

Podemos decir que:

- RPM instala paquetes.
- DNF administra paquetes.

---

# ¿Qué son las dependencias?

Muchas aplicaciones necesitan otras bibliotecas para funcionar.

Ejemplo:

```
Programa A

↓

Necesita

↓

Biblioteca B

↓

Biblioteca C
```

Instalar únicamente el programa principal provocaría errores.

DNF detecta estas dependencias e instala automáticamente todos los paquetes necesarios.

---

# ¿Qué es un repositorio?

Un repositorio es un servidor que almacena paquetes RPM organizados.

Puede contener miles de aplicaciones listas para instalar.

Ejemplos:

- BaseOS
- AppStream
- EPEL
- Repositorios internos de empresas

---

# ¿Qué es AppStream?

AppStream es un repositorio especial introducido en Red Hat Enterprise Linux 8.

Permite instalar diferentes versiones de una misma aplicación mediante módulos.

Ejemplo:

```
Node.js 18

Node.js 20

Node.js 22
```

Sin necesidad de cambiar de sistema operativo.

---

# Flujo de instalación

Cuando ejecutamos:

```bash
sudo dnf install httpd
```

Ocurre el siguiente proceso:

```
Usuario

↓

DNF

↓

Consulta repositorios

↓

Descarga paquetes

↓

Verifica firmas GPG

↓

Resuelve dependencias

↓

RPM instala

↓

Actualiza la base de datos RPM

↓

Software disponible
```

---

# Base de datos RPM

Toda la información de los paquetes instalados se almacena en una base de datos local.

Gracias a ella podemos consultar:

- Qué paquetes están instalados.
- Qué archivos pertenecen a un paquete.
- Versiones.
- Dependencias.
- Fecha de instalación.

---

# ¿Dónde se encuentra?

En RHEL modernos:

```
/var/lib/rpm/
```

No debe modificarse manualmente.

---

# Tipos de operaciones

Durante la administración del software realizaremos tareas como:

- Instalar paquetes.
- Actualizar paquetes.
- Eliminar paquetes.
- Buscar paquetes.
- Consultar información.
- Verificar integridad.
- Configurar repositorios.
- Administrar módulos.

---

# Comparación con Windows

| Windows | RHEL |
|----------|------|
| setup.exe | RPM |
| Microsoft Store | Repositorios |
| Windows Update | DNF Update |
| Panel de Control | DNF/RPM |
| Instalación manual | RPM |
| Instalación automática | DNF |

---

# Ciclo de vida del software

```
Buscar

↓

Instalar

↓

Configurar

↓

Actualizar

↓

Verificar

↓

Eliminar
```

---

# Herramientas principales

| Herramienta | Función |
|-------------|----------|
| `rpm` | Administración de paquetes locales |
| `dnf` | Administración completa del software |
| `repoquery` | Consultar repositorios |
| `dnf history` | Historial de instalaciones |
| `dnf config-manager` | Configurar repositorios |
| `rpmkeys` | Verificar firmas GPG |

---

# Beneficios del sistema RPM/DNF

- Instalaciones consistentes.
- Gestión automática de dependencias.
- Actualizaciones seguras.
- Integridad del software.
- Fácil administración.
- Repositorios centralizados.
- Verificación mediante firmas digitales.

---

# Ejemplo práctico

Supongamos que deseamos instalar Apache.

En lugar de buscar un instalador en Internet:

```
Descargar

↓

Ejecutar

↓

Siguiente

↓

Siguiente

↓

Finalizar
```

Simplemente ejecutamos:

```bash
sudo dnf install httpd
```

DNF realiza automáticamente:

- Descarga.
- Dependencias.
- Instalación.
- Configuración básica.
- Registro del paquete.

---

# Flujo completo de administración

```
Repositorio

↓

DNF

↓

Dependencias

↓

RPM

↓

Base de datos

↓

Sistema
```

---

# ¿Qué aprenderás en este módulo?

Durante las siguientes lecciones aprenderás:

- Administración de paquetes RPM.
- Instalación de software con DNF.
- Configuración de repositorios.
- Uso de AppStream.
- Administración avanzada de DNF.
- Verificación mediante firmas GPG.
- Buenas prácticas para administrar software en RHEL.

---

# Buenas prácticas RHCSA

✔ Instalar software utilizando repositorios oficiales siempre que sea posible.

✔ Mantener el sistema actualizado.

✔ Verificar la procedencia de los paquetes antes de instalarlos.

✔ Evitar instalar paquetes manualmente cuando exista un repositorio confiable.

✔ No eliminar paquetes críticos del sistema.

✔ Utilizar DNF para resolver automáticamente las dependencias.

---

# Errores comunes

## Instalar paquetes desde sitios desconocidos

Puede comprometer la seguridad del sistema.

---

## Forzar instalaciones ignorando dependencias

Puede provocar que una aplicación no funcione correctamente.

---

## Modificar la base de datos RPM manualmente

Nunca debe editarse directamente.

---

## Mezclar repositorios incompatibles

Puede generar conflictos entre versiones de paquetes.

---

## No actualizar el sistema

Mantener paquetes desactualizados aumenta el riesgo de vulnerabilidades.

---

# Comandos importantes RHCSA

| Comando | Descripción |
|----------|-------------|
| `rpm` | Administrar paquetes RPM |
| `dnf` | Administrar software |
| `dnf install` | Instalar paquetes |
| `dnf remove` | Eliminar paquetes |
| `dnf update` | Actualizar paquetes |
| `dnf search` | Buscar paquetes |
| `dnf info` | Mostrar información |
| `dnf history` | Consultar historial |

---

# Resumen

En esta lección aprendiste:

- Cómo funciona la administración del software en Red Hat Enterprise Linux.
- La diferencia entre RPM y DNF.
- Qué son los repositorios y AppStream.
- Cómo se administran las dependencias.
- El flujo completo de instalación de software.
- Los componentes principales del ecosistema de paquetes de RHEL.

---

# Laboratorio práctico RHCSA

## Escenario 1

Verifica la versión del sistema operativo.

```bash
cat /etc/redhat-release
```

---

## Escenario 2

Comprueba que DNF está instalado.

```bash
dnf --version
```

---

## Escenario 3

Verifica la versión de RPM.

```bash
rpm --version
```

---

## Escenario 4

Lista los repositorios habilitados.

```bash
dnf repolist
```

Observa los nombres de los repositorios disponibles.

---

## Escenario 5

Consulta cuántos paquetes RPM están instalados en el sistema.

```bash
rpm -qa | wc -l
```

Después, muestra los primeros diez paquetes instalados.

```bash
rpm -qa | head
```

> **Objetivo general:** comprender la arquitectura de la administración de software en Red Hat Enterprise Linux y cómo interactúan **RPM**, **DNF**, **AppStream** y los **repositorios**. Este conocimiento constituye la base para el resto del módulo y es fundamental para administrar servidores RHEL de manera profesional y aprobar el examen **RHCSA**.