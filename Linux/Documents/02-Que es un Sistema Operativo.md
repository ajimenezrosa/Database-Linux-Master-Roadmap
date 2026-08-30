# ¿Qué es un Sistema Operativo?

![Centro de datos con servidores Linux](images/que-es-un-sistema-operativo.jpg)

> **Objetivo de la lección**
>
> Comprender qué es un sistema operativo, cuál es su función dentro de una computadora y conocer los diferentes tipos de sistemas operativos existentes antes de comenzar el estudio de Red Hat Enterprise Linux (RHCSA).

---

# Introducción

Antes de aprender Linux, es importante comprender qué es un **Sistema Operativo (Operating System u OS)**.

Muchas personas utilizan una computadora todos los días, pero pocas saben realmente qué hace el sistema operativo o por qué es tan importante.

Cuando encendemos una computadora, un teléfono móvil o incluso un televisor inteligente, el primer software que se ejecuta es el sistema operativo. Sin él, el hardware sería prácticamente inútil.

En este curso aprenderemos **Linux**, pero primero debemos entender el papel que desempeña un sistema operativo.

---

# ¿Qué es un Sistema Operativo?

Un **Sistema Operativo (Operating System - OS)** es el software encargado de administrar todos los recursos del computador y servir como intermediario entre el usuario y el hardware.

En otras palabras, el sistema operativo hace posible que podamos utilizar la computadora de una forma sencilla sin tener que comunicarnos directamente con los componentes electrónicos.

Su función principal consiste en coordinar el funcionamiento de todos los dispositivos del equipo.

---

# El sistema operativo como intermediario

![Diagrama del sistema operativo](images/os-como-intermediario.png)

Podemos imaginar el sistema operativo como un puente entre el usuario y el hardware.

```
             Usuario
                 │
                 ▼
      Sistema Operativo
                 │
                 ▼
      Hardware del computador
```

Por ejemplo:

1. El usuario presiona una tecla.
2. El teclado envía una señal eléctrica.
3. El sistema operativo interpreta esa señal.
4. La aplicación recibe la información.
5. La letra aparece en pantalla.

Todo este proceso ocurre en una fracción de segundo.

Sin un sistema operativo sería necesario comunicarse directamente con el hardware, algo extremadamente complejo.

---

# Funciones principales de un Sistema Operativo

Un sistema operativo realiza cientos de tareas de manera automática.

Entre las más importantes se encuentran:

- Administrar la memoria RAM.
- Controlar el procesador (CPU).
- Gestionar discos duros y SSD.
- Administrar archivos y carpetas.
- Controlar dispositivos como teclado, mouse e impresoras.
- Administrar usuarios y permisos.
- Ejecutar programas.
- Gestionar conexiones de red.
- Proteger el sistema mediante mecanismos de seguridad.

---

# Tipos de Sistemas Operativos

Existen diferentes categorías de sistemas operativos según el dispositivo donde se utilicen.

---

# 1. Sistemas Operativos de Escritorio

![Computadora de escritorio](images/sistema-operativo-escritorio.jpg)

Son los que utilizamos diariamente en computadoras personales.

Ejemplos:

- Microsoft Windows
- macOS
- Ubuntu
- Fedora
- Linux Mint

Estos sistemas están diseñados para facilitar el trabajo cotidiano mediante interfaces gráficas amigables.

---

# 2. Sistemas Operativos para Servidores

![Centro de datos](images/servidores-linux.jpg)

Los servidores requieren sistemas operativos optimizados para ofrecer estabilidad, seguridad y alto rendimiento.

Entre los más utilizados encontramos:

- Red Hat Enterprise Linux (RHEL)
- Rocky Linux
- AlmaLinux
- Ubuntu Server
- Windows Server

Durante este curso trabajaremos principalmente con **Red Hat Enterprise Linux**, ya que es el sistema operativo utilizado para la certificación RHCSA.

---

# 3. Sistemas Operativos Móviles

![Teléfono Android](images/android-smartphone.jpg)

Los teléfonos inteligentes también utilizan sistemas operativos.

Los principales son:

- Android
- iOS

Android utiliza el **kernel de Linux**, aunque incorpora su propio entorno y aplicaciones.

---

# 4. Sistemas Operativos Embebidos

![Router WiFi](images/router-linux.jpg)

Los sistemas embebidos están diseñados para realizar tareas específicas.

Se encuentran en dispositivos como:

- Routers
- Televisores inteligentes
- Cámaras IP
- Refrigeradores inteligentes
- Lavadoras
- Consolas multimedia
- Equipos industriales

Generalmente utilizan versiones reducidas de Linux.

---

# 5. Sistemas Operativos en Tiempo Real (RTOS)

![Equipo médico](images/rtos-medico.jpg)

Un sistema operativo en tiempo real (Real-Time Operating System) está diseñado para responder en tiempos extremadamente cortos.

Se utiliza en:

- Equipos médicos
- Sistemas aeroespaciales
- Automóviles
- Defensa
- Robots industriales
- Firewalls especializados
- Equipos de telecomunicaciones

En estos sistemas una demora de algunos milisegundos puede provocar fallos importantes.

---

# ¿Por qué existen tantos sistemas operativos?

Cada tipo de dispositivo tiene necesidades diferentes.

Por ejemplo:

| Dispositivo | Necesidad principal |
|-------------|--------------------|
| PC de escritorio | Facilidad de uso |
| Servidor | Estabilidad y seguridad |
| Teléfono | Bajo consumo de energía |
| Router | Redes |
| Automóvil | Tiempo real |
| Equipo médico | Alta disponibilidad |

No existe un único sistema operativo perfecto para todos los escenarios.

---

# Linux dentro de los Sistemas Operativos

Linux es uno de los sistemas operativos más importantes del mundo.

Se utiliza en:

- Centros de datos.
- Empresas.
- Supercomputadoras.
- Plataformas en la nube.
- Routers.
- Android.
- Dispositivos IoT.
- Sistemas industriales.
- Servidores web.
- Bases de datos.

Red Hat Enterprise Linux es una distribución empresarial de Linux diseñada para entornos corporativos.

---

# ¿Qué aprenderás en RHCSA?

Durante este curso aprenderás a administrar un servidor Linux profesional.

Entre los temas más importantes se encuentran:

- Instalación de Red Hat Enterprise Linux.
- Administración de usuarios.
- Permisos.
- Procesos.
- Servicios.
- Redes.
- Almacenamiento.
- SELinux.
- Firewalld.
- Automatización.
- Contenedores con Podman.
- Solución de problemas.

---

# Resumen

Un sistema operativo es el software que permite que una computadora funcione correctamente.

Actúa como intermediario entre el usuario y el hardware, administrando todos los recursos del sistema.

Existen diferentes tipos de sistemas operativos según el dispositivo donde se utilicen:

- Escritorio
- Servidores
- Móviles
- Embebidos
- Tiempo Real

Linux destaca por su estabilidad, seguridad y flexibilidad, razón por la cual es ampliamente utilizado tanto en servidores empresariales como en millones de dispositivos alrededor del mundo.

---

# Preguntas de Repaso

1. ¿Qué es un sistema operativo?
2. ¿Cuál es la función principal de un sistema operativo?
3. ¿Por qué decimos que el sistema operativo actúa como intermediario?
4. ¿Qué diferencias existen entre un sistema operativo de escritorio y uno para servidores?
5. ¿Qué dispositivos utilizan sistemas operativos embebidos?
6. ¿Qué significa RTOS?
7. ¿Qué sistema operativo estudiarás durante este curso?

---

# Próxima lección

➡ **Linux en la vida cotidiana**