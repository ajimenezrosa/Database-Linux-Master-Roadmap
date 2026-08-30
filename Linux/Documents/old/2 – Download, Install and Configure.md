# 📘 Complete Linux Training Course to Get Your Dream IT Job

## 📚 Syllabus — Module 2: Download, Install and Configure

---

# Module 2 – Download, Install and Configure

## 🎯 Objetivos del Módulo

Al finalizar este módulo serás capaz de:

* Comprender qué es una máquina virtual.
* Instalar Oracle VirtualBox.
* Crear y administrar máquinas virtuales.
* Conocer las principales distribuciones de Linux.
* Instalar Linux CentOS en un entorno virtual.
* Comprender las diferencias entre Linux y Windows.
* Identificar dónde se utiliza Linux en la industria tecnológica.

---

# 🖥️ ¿Qué es Oracle VirtualBox?

Oracle VirtualBox es un software de virtualización gratuito desarrollado por Oracle que permite ejecutar varios sistemas operativos dentro de un mismo computador físico.

En lugar de instalar Linux directamente sobre tu computadora, VirtualBox crea una computadora virtual completamente independiente.

## ¿Qué significa virtualizar?

La virtualización consiste en crear una computadora simulada mediante software.

Una máquina virtual posee sus propios componentes virtuales:

* CPU
* Memoria RAM
* Disco duro
* Tarjeta de red
* Tarjeta gráfica
* USB
* Audio

Todo funciona como si fuera un computador físico.

---

# ✅ Ventajas de utilizar VirtualBox

* Gratuito y de código abierto.
* Fácil instalación.
* Compatible con Windows, Linux y macOS.
* Permite crear múltiples laboratorios.
* Ideal para aprender Linux sin afectar el sistema principal.
* Posibilidad de tomar Snapshots.
* Permite compartir carpetas entre Host y Guest.
* Soporta múltiples adaptadores de red.

---

# 📥 Descargando e Instalando Oracle VirtualBox

## Paso 1

Descargar VirtualBox desde el sitio oficial de Oracle.

Selecciona la versión correspondiente a tu sistema operativo:

* Windows
* Fedora
* Ubuntu
* Debian
* macOS

---

## Paso 2

Ejecutar el instalador.

La instalación es prácticamente automática.

Normalmente solo es necesario hacer clic en:

* Next
* Next
* Install
* Finish

---

## Paso 3

Abrir VirtualBox y verificar que el administrador de máquinas virtuales aparezca correctamente.

---

# 💻 Creando una Máquina Virtual

Una máquina virtual representa un computador completamente funcional.

## Información básica

* Nombre
* Tipo de sistema operativo
* Versión

Ejemplo:

Nombre:

```
Linux-Lab-01
```

Tipo:

```
Linux
```

Versión:

```
Red Hat (64-bit)
```

---

## Asignación de recursos

### Memoria RAM

Recomendado:

* Mínimo: 2 GB
* Ideal: 4 GB
* Laboratorios avanzados: 8 GB

---

### Procesadores

Asignar:

* 2 CPU
* 4 CPU si el equipo lo permite

---

### Disco Virtual

Formato recomendado:

VDI

Asignación:

* Dinámica

Espacio recomendado:

40 GB

---

# 🐧 Distribuciones Linux

Linux no es un único sistema operativo.

Existen cientos de distribuciones.

Las más utilizadas son:

| Distribución             | Uso Principal                      |
| ------------------------ | ---------------------------------- |
| Fedora                   | Desarrollo y tecnologías recientes |
| Ubuntu                   | Escritorio y servidores            |
| Debian                   | Estabilidad                        |
| Rocky Linux              | Servidores empresariales           |
| AlmaLinux                | Reemplazo de CentOS                |
| Red Hat Enterprise Linux | Empresas                           |
| Kali Linux               | Seguridad informática              |
| openSUSE                 | Desarrollo y administración        |

---

# 💿 Formas de Instalar Linux

Linux puede instalarse de distintas maneras dependiendo del objetivo del usuario.

## Instalación Física

Linux reemplaza completamente al sistema operativo actual.

Ideal para:

* Servidores
* Equipos dedicados

---

## Dual Boot

Permite tener dos sistemas operativos.

Ejemplo:

* Windows
* Fedora

El usuario selecciona cuál iniciar.

---

## Máquina Virtual

La opción más segura para aprender.

Ventajas:

* No modifica el disco principal.
* Fácil de eliminar.
* Permite crear múltiples laboratorios.
* Se pueden tomar Snapshots.

---

## Cloud

Linux también puede ejecutarse directamente en la nube.

Ejemplos:

* AWS EC2
* Azure VM
* Google Cloud Compute Engine
* Oracle Cloud

---

# 📥 Instalando Linux (CentOS)

Durante la instalación normalmente se configuran:

* Idioma
* Zona horaria
* Teclado
* Disco
* Red
* Usuario administrador
* Contraseña Root

Una vez finalizada la instalación, el sistema reinicia automáticamente.

---

# 🔴 Instalación de Red Hat Enterprise Linux (Opcional)

Red Hat Enterprise Linux (RHEL) es la distribución empresarial más utilizada del mundo.

Características:

* Soporte comercial.
* Certificaciones oficiales.
* Actualizaciones garantizadas.
* Alta estabilidad.
* Amplio uso en bancos, gobiernos y grandes empresas.

---

# 🖥️ El Escritorio Linux (GUI)

GUI significa:

**Graphical User Interface**

Permite interactuar con Linux mediante ventanas, iconos y menús.

Algunos escritorios populares:

* GNOME
* KDE Plasma
* XFCE
* Cinnamon
* MATE

Aunque Linux posee interfaz gráfica, la mayoría de administradores trabajan principalmente desde la Terminal.

---

# ⚙️ Administración de Máquinas Virtuales

VirtualBox permite administrar completamente los laboratorios.

Operaciones comunes:

* Encender
* Apagar
* Reiniciar
* Clonar
* Exportar
* Importar
* Crear Snapshots
* Restaurar Snapshots
* Modificar Hardware Virtual

Los Snapshots permiten regresar la máquina exactamente al estado anterior.

Son una herramienta indispensable durante el aprendizaje.

---

# ⚔️ Linux vs Windows

| Característica    | Linux       | Windows            |
| ----------------- | ----------- | ------------------ |
| Licencia          | Open Source | Propietaria        |
| Precio            | Gratuito    | Licencia comercial |
| Seguridad         | Muy alta    | Alta               |
| Personalización   | Muy alta    | Limitada           |
| Uso empresarial   | Muy alto    | Muy alto           |
| Línea de comandos | Bash        | PowerShell         |
| Estabilidad       | Excelente   | Excelente          |

---

# 🌍 ¿Quién utiliza Linux?

Linux domina gran parte de la infraestructura tecnológica mundial.

Algunas organizaciones que utilizan Linux:

* Google
* Amazon
* Microsoft Azure
* Meta
* NASA
* IBM
* Oracle
* Red Hat
* Cisco
* Netflix
* PayPal
* Tesla

También es el sistema operativo principal en:

* Servidores Web
* Supercomputadoras
* Contenedores Docker
* Kubernetes
* Cloud Computing
* Inteligencia Artificial
* Bases de Datos
* Ciberseguridad

---

# 📝 Quiz

1. ¿Qué es una máquina virtual?
2. ¿Qué ventajas ofrece VirtualBox?
3. ¿Cuál es la diferencia entre instalar Linux físicamente y en una máquina virtual?
4. ¿Qué distribución utiliza Red Hat como versión empresarial?
5. ¿Qué es un Snapshot?
6. Menciona tres empresas que utilizan Linux.
7. ¿Qué ventajas ofrece Linux frente a Windows para servidores?

---

# 📚 Homework

## Práctica 1

Instalar Oracle VirtualBox.

---

## Práctica 2

Crear una máquina virtual con:

* 4 GB RAM
* 2 CPU
* Disco dinámico de 40 GB

---

## Práctica 3

Descargar la imagen ISO de CentOS o una distribución compatible como Rocky Linux, AlmaLinux o Fedora.

---

## Práctica 4

Instalar Linux dentro de la máquina virtual.

---

## Práctica 5

Explorar el escritorio Linux e identificar:

* Terminal
* Administrador de archivos
* Configuración
* Navegador
* Centro de Software

---

# 📎 Handouts

* Guía de instalación de Oracle VirtualBox.
* Guía de instalación de Linux.
* Comparativa de distribuciones Linux.
* Tabla Linux vs Windows.
* Requisitos mínimos de hardware.
* Atajos básicos de VirtualBox.

---

# 🎓 Resumen del Módulo

En este módulo aprendiste a preparar el entorno de laboratorio donde realizarás todas las prácticas del curso. Comprendiste el funcionamiento de las máquinas virtuales, conociste las principales distribuciones de Linux, instalaste un sistema operativo Linux y aprendiste a administrar tu laboratorio mediante Oracle VirtualBox.

Con este entorno listo, ya estás preparado para comenzar a trabajar directamente con Linux desde la terminal en los siguientes módulos del curso.

---
