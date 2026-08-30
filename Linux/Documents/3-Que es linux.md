# ¿Qué es Linux?

![Administrador trabajando con servidores Linux](images/03-que-es-linux-portada.jpg)

> **Objetivo de la lección**
>
> Comprender qué es Linux, qué significan los conceptos de software libre y código abierto, cómo se instala y por qué aprender Linux puede mejorar las oportunidades profesionales en tecnología.

---

## Introducción

Linux es el tema principal de este manual y una de las tecnologías más importantes dentro de la administración moderna de sistemas.

En términos sencillos, **Linux es un sistema operativo libre y de código abierto**. Se utiliza en servidores, plataformas de nube, centros de datos, computadoras personales, teléfonos, dispositivos de red, sistemas embebidos y muchas otras tecnologías.

Antes de continuar, debemos recordar que un sistema operativo actúa como intermediario entre:

- El usuario.
- Las aplicaciones.
- Los recursos físicos del equipo.

Linux permite que los programas utilicen correctamente el procesador, la memoria, los discos, las interfaces de red y los demás dispositivos conectados a la computadora.

---

## 1. ¿Qué significa Linux?

Cuando hablamos de Linux, podemos referirnos de manera general a un sistema operativo construido alrededor del **kernel de Linux**.

El kernel es el componente central que se comunica directamente con el hardware y administra los recursos principales del sistema.

Entre sus responsabilidades se encuentran:

- Administrar los procesos.
- Controlar la memoria.
- Comunicarse con los dispositivos.
- Gestionar los sistemas de archivos.
- Facilitar las conexiones de red.
- Aplicar mecanismos de seguridad.
- Coordinar el acceso al procesador.

En la práctica, Linux se combina con numerosas herramientas, bibliotecas y aplicaciones para formar un sistema operativo completo.

---

## 2. Linux es software libre

![Computadora ejecutando una distribución Linux](images/03-linux-software-libre.jpg)

Una de las características más conocidas de Linux es que puede descargarse, instalarse y utilizarse sin pagar por una licencia tradicional de uso.

Esto permite que una persona pueda instalar distribuciones como Fedora, Ubuntu, Debian, Rocky Linux o AlmaLinux en su computadora sin tener que comprar una licencia del sistema operativo.

Sin embargo, es importante diferenciar entre:

- **Usar Linux gratuitamente.**
- **Contratar soporte empresarial.**

Algunas distribuciones empresariales ofrecen servicios comerciales que pueden incluir:

- Soporte técnico.
- Actualizaciones certificadas.
- Correcciones de seguridad.
- Acceso a repositorios empresariales.
- Documentación especializada.
- Certificación para determinadas aplicaciones y plataformas.

Por ejemplo, una organización puede pagar una suscripción de Red Hat Enterprise Linux para recibir soporte y servicios empresariales, aunque el software esté basado en tecnologías de código abierto.

> **Importante:** libre no significa necesariamente que todos los servicios relacionados sean gratuitos.

---

## 3. Linux es de código abierto

El código fuente de Linux está disponible para que desarrolladores, empresas e investigadores puedan estudiarlo, modificarlo y contribuir a su desarrollo, siempre respetando las condiciones de sus licencias.

Esto permite:

- Revisar cómo funciona el sistema.
- Detectar y corregir errores.
- Crear versiones adaptadas.
- Añadir soporte para nuevo hardware.
- Desarrollar nuevas funcionalidades.
- Construir sistemas especializados.
- Colaborar con una comunidad internacional.

La naturaleza abierta de Linux ha permitido que miles de personas y organizaciones participen en su evolución.

---

## 4. Linux frente a Windows y macOS

Linux, Windows y macOS son sistemas operativos, pero existen diferencias importantes entre ellos.

| Característica | Linux | Windows | macOS |
|---|---|---|---|
| Código fuente | Principalmente abierto | Propietario | Principalmente propietario |
| Fabricante principal | Comunidad y múltiples empresas | Microsoft | Apple |
| Hardware | Gran variedad | Gran variedad | Principalmente equipos Apple |
| Personalización | Muy alta | Media | Limitada |
| Uso en servidores | Muy extendido | Extendido | Poco habitual |
| Línea de comandos | Fundamental | Disponible | Disponible |
| Distribuciones o ediciones | Muchas | Varias ediciones | Versiones oficiales de Apple |

Linux destaca especialmente por su flexibilidad, estabilidad, seguridad y capacidad de personalización.

---

## 5. ¿Qué es una distribución de Linux?

![Diferentes distribuciones de Linux](images/03-distribuciones-linux.jpg)

Linux no se presenta como un único producto.

Existen diferentes versiones completas conocidas como **distribuciones de Linux** o simplemente **distros**.

Una distribución normalmente incluye:

- El kernel de Linux.
- Un instalador.
- Un sistema de administración de paquetes.
- Herramientas de línea de comandos.
- Bibliotecas.
- Servicios.
- Aplicaciones.
- Opcionalmente, una interfaz gráfica.
- Repositorios de software.

Algunas distribuciones conocidas son:

- Red Hat Enterprise Linux.
- Fedora.
- Ubuntu.
- Debian.
- Rocky Linux.
- AlmaLinux.
- SUSE Linux Enterprise.
- openSUSE.
- Arch Linux.
- Linux Mint.

Cada distribución puede estar orientada a diferentes usuarios o propósitos.

| Distribución | Uso frecuente |
|---|---|
| Red Hat Enterprise Linux | Empresas y servidores |
| Fedora | Innovación, aprendizaje y estaciones de trabajo |
| Ubuntu | Escritorio, servidores y nube |
| Debian | Servidores y sistemas estables |
| Rocky Linux | Servidores compatibles con el ecosistema de RHEL |
| AlmaLinux | Servidores compatibles con el ecosistema de RHEL |
| Arch Linux | Usuarios avanzados y alta personalización |

Durante la preparación para RHCSA, el enfoque principal es **Red Hat Enterprise Linux**.

---

## 6. ¿Cómo se instala Linux?

![Instalación de Linux desde una memoria USB](images/03-instalacion-linux-usb.jpg)

Una computadora sin sistema operativo no puede ofrecer al usuario un entorno normal de trabajo.

Para instalar Linux se necesita obtener una imagen de instalación, normalmente en formato ISO.

Una imagen ISO es un archivo que contiene todos los componentes necesarios para iniciar el instalador del sistema operativo.

Actualmente, una instalación suele realizarse mediante:

- Una memoria USB arrancable.
- Una unidad virtual conectada a una máquina virtual.
- Una consola remota de administración.
- Un servidor de instalación por red.
- Una plataforma de nube.

El procedimiento general es el siguiente:

1. Descargar la imagen ISO de la distribución.
2. Crear una memoria USB arrancable o conectar la ISO a una máquina virtual.
3. Configurar el equipo para iniciar desde ese medio.
4. Abrir el asistente de instalación.
5. Seleccionar idioma, teclado y zona horaria.
6. Configurar el almacenamiento.
7. Configurar la red.
8. Crear los usuarios necesarios.
9. Iniciar la instalación.
10. Reiniciar el equipo.
11. Acceder al nuevo sistema Linux.

Al finalizar, el usuario contará con un sistema operativo funcional.

---

## 7. Linux como intermediario entre el usuario y el hardware

![Usuario, Linux y hardware](images/03-linux-usuario-hardware.png)

Linux permite que el usuario y las aplicaciones utilicen los recursos físicos de la computadora.

```text
┌──────────────────────────────┐
│            Usuario           │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│ Aplicaciones y comandos      │
│ Bash, navegador, base de     │
│ datos, servidor web, etc.    │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│ Sistema operativo GNU/Linux  │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│ Hardware                     │
│ CPU, RAM, discos, red, USB,  │
│ teclado, pantalla, impresora │
└──────────────────────────────┘
```

### Ejemplo: imprimir un documento

Cuando el usuario solicita imprimir un archivo:

1. La aplicación envía la solicitud al sistema operativo.
2. Linux identifica la impresora configurada.
3. El sistema prepara el trabajo de impresión.
4. El controlador se comunica con el dispositivo.
5. La impresora recibe los datos.
6. El documento se imprime.

### Ejemplo: abrir un navegador

Cuando el usuario ejecuta un navegador:

1. Linux localiza el programa.
2. Carga sus archivos en memoria.
3. Crea uno o varios procesos.
4. Asigna tiempo del procesador.
5. Permite que el navegador utilice la red.
6. Presenta la interfaz en la pantalla.

El sistema operativo realiza todas estas tareas en segundo plano.

---

## 8. ¿Por qué Linux es tan importante?

Linux ocupa un lugar fundamental en la infraestructura tecnológica moderna.

Se utiliza ampliamente en:

- Servidores empresariales.
- Plataformas de computación en la nube.
- Centros de datos.
- Supercomputadoras.
- Contenedores.
- Sistemas de bases de datos.
- Plataformas web.
- Automatización.
- Redes.
- Ciberseguridad.
- DevOps.
- Dispositivos embebidos.

Aunque muchas personas utilizan Windows o macOS en sus computadoras personales, una gran parte de los servicios remotos que consumen puede ejecutarse sobre Linux.

---

## 9. Linux en servidores y computación en la nube

![Centro de datos con servidores Linux](images/03-linux-servidores-cloud.jpg)

Linux es especialmente popular en entornos empresariales.

Las organizaciones lo utilizan para ejecutar:

- Servidores web.
- Bases de datos.
- Aplicaciones corporativas.
- Plataformas de comercio electrónico.
- Sistemas de archivos.
- Servicios de autenticación.
- Contenedores.
- Máquinas virtuales.
- Herramientas de monitoreo.
- Sistemas de respaldo.

También es una tecnología central en plataformas de nube pública y privada.

Los profesionales que trabajan con servicios en la nube necesitan comprender Linux porque muchas máquinas virtuales, imágenes, contenedores y herramientas de automatización se ejecutan sobre este sistema.

---

## 10. La línea de comandos de Linux

![Terminal de administración Linux](images/03-terminal-linux.jpg)

Linux puede utilizarse mediante una interfaz gráfica, pero la línea de comandos es una de sus herramientas más importantes.

La terminal permite:

- Administrar servidores remotamente.
- Ejecutar tareas con rapidez.
- Automatizar operaciones.
- Procesar grandes cantidades de información.
- Configurar servicios.
- Analizar registros.
- Administrar usuarios y permisos.
- Solucionar problemas.
- Crear scripts.

Algunos comandos básicos son:

```bash
pwd
ls
cd
mkdir
cp
mv
rm
cat
grep
find
ps
systemctl
journalctl
```

En un entorno gráfico, muchas tareas dependen de hacer clic y navegar por ventanas. En la terminal, una operación compleja puede ejecutarse mediante un solo comando o un script reutilizable.

> La interfaz gráfica es útil, pero para administrar Linux profesionalmente es esencial aprender la línea de comandos.

---

## 11. Linux y el rendimiento

La línea de comandos puede ofrecer una administración rápida y eficiente porque:

- Consume menos recursos que un entorno gráfico completo.
- Permite ejecutar comandos directamente.
- Facilita la automatización.
- Puede utilizarse mediante conexiones remotas.
- Permite trabajar en servidores sin monitor.
- Facilita repetir procedimientos con precisión.

Sin embargo, el rendimiento de Linux no depende únicamente de usar la terminal. También intervienen factores como:

- La configuración del sistema.
- El hardware.
- Los servicios activos.
- El sistema de archivos.
- La carga de trabajo.
- La administración de memoria.
- La configuración de red.

---

## 12. Seguridad en Linux

Linux incluye múltiples mecanismos de seguridad.

Entre ellos se encuentran:

- Usuarios y grupos.
- Permisos de lectura, escritura y ejecución.
- Control de acceso mediante `sudo`.
- Autenticación.
- Firewalls.
- Registros del sistema.
- Actualizaciones de seguridad.
- Políticas de SELinux.
- Separación de procesos.
- Cifrado.
- Acceso remoto seguro mediante SSH.

Durante el curso RHCSA aprenderás a configurar y administrar varios de estos componentes.

La seguridad no depende únicamente del sistema operativo. También requiere una configuración correcta, actualizaciones periódicas y buenas prácticas de administración.

---

## 13. Capacidad de personalización

Gracias a su naturaleza abierta, Linux puede adaptarse a diferentes necesidades.

Es posible personalizar:

- El entorno de escritorio.
- Los servicios activos.
- El kernel.
- Los paquetes instalados.
- La configuración de red.
- El sistema de inicio.
- Las políticas de seguridad.
- La interfaz gráfica.
- Las herramientas de administración.

Esta flexibilidad permite utilizar Linux tanto en una pequeña computadora como en una infraestructura empresarial de gran escala.

---

## 14. Comunidad y soporte

Linux cuenta con una enorme comunidad de usuarios y desarrolladores.

Es posible encontrar ayuda mediante:

- Documentación oficial.
- Manuales.
- Foros.
- Listas de correo.
- Comunidades técnicas.
- Repositorios de código.
- Artículos.
- Cursos.
- Videos.
- Proveedores empresariales.

En el caso de Red Hat Enterprise Linux, también existe soporte comercial y documentación oficial especializada.

---

## 15. Linux y las oportunidades profesionales

![Profesional administrando infraestructura Linux](images/03-linux-oportunidades-profesionales.jpg)

Aprender Linux abre oportunidades en diferentes áreas de tecnología.

Algunos perfiles profesionales relacionados son:

- Administrador de sistemas Linux.
- Ingeniero de infraestructura.
- Administrador de bases de datos.
- Ingeniero de soporte.
- Especialista en redes.
- Ingeniero DevOps.
- Ingeniero de nube.
- Especialista en ciberseguridad.
- Site Reliability Engineer.
- Administrador de contenedores.
- Ingeniero de plataformas.

Las habilidades de Linux también son útiles para trabajar con:

- Docker.
- Podman.
- Kubernetes.
- Ansible.
- Terraform.
- AWS.
- Microsoft Azure.
- Google Cloud.
- PostgreSQL.
- MySQL.
- Oracle Database.
- Servidores web como Apache y Nginx.

---

## 16. Relación con RHCSA

La certificación **Red Hat Certified System Administrator** valida habilidades prácticas para administrar sistemas Red Hat Enterprise Linux.

Durante la preparación aprenderás a:

- Acceder a sistemas Linux.
- Utilizar la línea de comandos.
- Administrar archivos.
- Crear usuarios y grupos.
- Configurar permisos.
- Administrar procesos.
- Controlar servicios con `systemd`.
- Configurar almacenamiento.
- Administrar redes.
- Trabajar con SELinux.
- Configurar el firewall.
- Utilizar contenedores.
- Solucionar problemas.

RHCSA no se centra solamente en memorizar conceptos. Requiere desarrollar la capacidad de ejecutar tareas administrativas en un sistema real.

---

## Comparación: interfaz gráfica y línea de comandos

| Interfaz gráfica | Línea de comandos |
|---|---|
| Fácil para usuarios principiantes | Requiere aprender comandos |
| Utiliza ventanas, iconos y menús | Utiliza una terminal |
| Puede consumir más recursos | Generalmente consume menos recursos |
| Adecuada para trabajo de escritorio | Ideal para servidores |
| Más difícil de automatizar | Fácil de automatizar con scripts |
| Requiere acceso gráfico | Puede utilizarse remotamente por SSH |

Ambas interfaces pueden coexistir. La selección depende de las necesidades del sistema y del administrador.

---

## Actividad práctica

Realiza la siguiente investigación:

1. Identifica la distribución Linux instalada en tu laboratorio.
2. Averigua la versión del kernel.
3. Comprueba el nombre del equipo.
4. Identifica el usuario actual.
5. Revisa cuánto tiempo lleva encendido el sistema.

Ejecuta los siguientes comandos:

```bash
cat /etc/os-release
uname -r
hostnamectl
whoami
uptime
```

### Resultado esperado

Debes poder responder:

- ¿Qué distribución utilizas?
- ¿Cuál es su versión?
- ¿Qué versión del kernel está instalada?
- ¿Cuál es el nombre del equipo?
- ¿Con qué usuario estás conectado?
- ¿Cuánto tiempo lleva funcionando el sistema?

---

## Ejercicio adicional

Ejecuta:

```bash
lsblk
free -h
lscpu
ip address
```

Investiga qué información proporciona cada comando.

| Comando | Información mostrada |
|---|---|
| `lsblk` | Discos, particiones y dispositivos de bloques |
| `free -h` | Uso de memoria RAM y swap |
| `lscpu` | Información del procesador |
| `ip address` | Interfaces y direcciones de red |

---

## Preguntas de repaso

1. ¿Qué es Linux?
2. ¿Qué función realiza el kernel?
3. ¿Qué significa que Linux sea software libre?
4. ¿Qué significa código abierto?
5. ¿Todos los servicios relacionados con Linux son gratuitos?
6. ¿Qué es una distribución de Linux?
7. Menciona cinco distribuciones.
8. ¿Por qué Linux es popular en servidores?
9. ¿Por qué es importante aprender la línea de comandos?
10. ¿Qué ventajas ofrece Linux en materia de personalización?
11. ¿Qué áreas profesionales requieren conocimientos de Linux?
12. ¿Cuál es la distribución principal estudiada para RHCSA?

---

## Resumen

Linux es un sistema operativo libre, abierto, estable, seguro y altamente configurable.

Puede instalarse en equipos físicos, máquinas virtuales, servidores y plataformas de nube. Su funcionamiento permite que las aplicaciones se comuniquen con los recursos físicos del equipo.

La existencia de múltiples distribuciones hace posible adaptar Linux a diferentes tipos de usuarios y entornos.

La línea de comandos es una herramienta central para la administración profesional porque facilita la velocidad, el acceso remoto, la automatización y la solución de problemas.

Aprender Linux ofrece una base sólida para desarrollar una carrera en administración de sistemas, bases de datos, redes, nube, DevOps y ciberseguridad.

---

## Próxima lección

➡️ **Historia de Linux y evolución de las distribuciones**

---

## Imágenes recomendadas para esta página

Guarda las imágenes dentro de la carpeta `images` del proyecto utilizando estos nombres:

```text
images/
├── 03-que-es-linux-portada.jpg
├── 03-linux-software-libre.jpg
├── 03-distribuciones-linux.jpg
├── 03-instalacion-linux-usb.jpg
├── 03-linux-usuario-hardware.png
├── 03-linux-servidores-cloud.jpg
├── 03-terminal-linux.jpg
└── 03-linux-oportunidades-profesionales.jpg
```

Las imágenes deberían mostrar escenas realistas como:

- Un administrador trabajando en un centro de datos.
- Una computadora ejecutando Linux.
- Una memoria USB utilizada para instalar Linux.
- Servidores empresariales.
- Una terminal Linux con comandos reales.
- Un profesional administrando infraestructura.
- Un diagrama limpio entre usuario, sistema operativo y hardware.

> Para evitar que las imágenes desaparezcan o cambien, descárgalas y almacénalas dentro del repositorio de tu página.
