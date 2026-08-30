# 59. Proceso de Arranque de Linux

> **Módulo 9: Arranque y Recuperación**  
> **Página 59 del Manual RHCSA**

---

# Objetivos de aprendizaje

Al finalizar esta lección serás capaz de:

- Comprender las etapas principales del arranque de Linux.
- Diferenciar BIOS y UEFI.
- Comprender la función de GRUB2.
- Explicar cómo se cargan el kernel y `initramfs`.
- Comprender cómo se localiza y monta el sistema de archivos raíz.
- Identificar el papel de `systemd`.
- Consultar el target predeterminado.
- Analizar mensajes y tiempos del proceso de arranque.
- Identificar en qué etapa puede producirse un fallo.
- Utilizar comandos básicos de diagnóstico de arranque.

---

# Introducción

El proceso de arranque de Linux comienza cuando el equipo se enciende y termina cuando el sistema está disponible para iniciar sesión.

Durante este proceso intervienen varios componentes:

```text
Firmware
Cargador de arranque
Kernel
initramfs
Sistema de archivos raíz
systemd
Servicios
```

Cada componente tiene una responsabilidad específica.

Comprender este proceso es fundamental para diagnosticar problemas como:

- El sistema no muestra el menú de GRUB.
- El kernel no puede cargarse.
- El sistema no encuentra el disco raíz.
- Un volumen LVM no puede activarse.
- Existe un error en `/etc/fstab`.
- `systemd` no puede iniciar un servicio crítico.
- El sistema entra en modo de emergencia.
- La interfaz gráfica no se inicia.

---

# Visión general del proceso de arranque

```text
Encendido del equipo
        │
        ▼
BIOS o UEFI
        │
        ▼
POST y detección de hardware
        │
        ▼
Selección del dispositivo de arranque
        │
        ▼
GRUB2
        │
        ▼
Kernel de Linux
        │
        ▼
initramfs
        │
        ▼
Detección del sistema de archivos raíz
        │
        ▼
Montaje del sistema raíz real
        │
        ▼
systemd
        │
        ▼
Target predeterminado
        │
        ▼
Servicios del sistema
        │
        ▼
Pantalla de inicio de sesión
```

---

# Etapas principales

El proceso puede dividirse en siete etapas:

```text
1. Firmware: BIOS o UEFI
2. POST y detección de hardware
3. Cargador de arranque GRUB2
4. Kernel de Linux
5. initramfs
6. systemd
7. Servicios y acceso al sistema
```

---

# Etapa 1: Encendido del equipo

Cuando el equipo se enciende, el procesador comienza a ejecutar instrucciones almacenadas en el firmware.

El firmware puede ser:

```text
BIOS
```

o:

```text
UEFI
```

Su función inicial es preparar el hardware y localizar un dispositivo desde el cual iniciar el sistema operativo.

---

# ¿Qué es BIOS?

BIOS significa:

```text
Basic Input/Output System
```

Es el firmware tradicional utilizado en equipos antiguos y en algunos sistemas configurados en modo de compatibilidad.

Sus características principales incluyen:

- Inicialización básica del hardware.
- Ejecución del POST.
- Uso tradicional de discos con tabla MBR.
- Búsqueda de código de arranque en el primer sector del disco.
- Limitaciones frente a discos de gran tamaño.
- Interfaz generalmente basada en texto.

---

# BIOS y MBR

En un sistema BIOS tradicional, el firmware busca el código inicial de arranque en el:

```text
Master Boot Record
```

El MBR ocupa el primer sector del disco.

Estructura conceptual:

```text
Disco
  │
  ├── MBR
  │     ├── Código de arranque
  │     └── Tabla de particiones
  │
  ├── Partición 1
  ├── Partición 2
  └── Partición 3
```

El espacio disponible en el MBR es limitado, por lo que GRUB2 utiliza varias etapas para completar su carga.

---

# ¿Qué es UEFI?

UEFI significa:

```text
Unified Extensible Firmware Interface
```

Es el reemplazo moderno de BIOS.

Sus características principales incluyen:

- Soporte para discos con tabla GPT.
- Compatibilidad con discos de gran capacidad.
- Una partición especial de sistema EFI.
- Capacidad para cargar archivos ejecutables `.efi`.
- Interfaz gráfica en muchos equipos.
- Soporte para Secure Boot.
- Variables de arranque almacenadas en firmware.

---

# UEFI y la partición EFI

En sistemas UEFI se utiliza una partición especial llamada:

```text
EFI System Partition
```

Abreviada como:

```text
ESP
```

Normalmente utiliza:

```text
FAT32
```

y suele montarse en:

```text
/boot/efi
```

Ejemplo conceptual:

```text
Disco GPT
    │
    ├── EFI System Partition
    │      └── /boot/efi
    │            └── EFI/
    │                  └── redhat/
    │
    ├── /boot
    ├── /
    └── swap
```

---

# Comparación entre BIOS y UEFI

| Característica | BIOS | UEFI |
|---|---|---|
| Tecnología | Tradicional | Moderna |
| Tabla común | MBR | GPT |
| Código de arranque | Primer sector del disco | Archivo ejecutable EFI |
| Partición especial | No obligatoria | EFI System Partition |
| Soporte de discos grandes | Limitado | Mejor soporte |
| Secure Boot | No | Sí |
| Variables de arranque | Limitadas | Almacenadas en firmware |
| Interfaz | Generalmente texto | Puede ser gráfica |

---

# Identificar si el sistema utiliza BIOS o UEFI

Comprueba si existe el directorio:

```bash
ls /sys/firmware/efi
```

Si existe, el sistema probablemente inició mediante UEFI.

También:

```bash
test -d /sys/firmware/efi \
&& echo "Sistema iniciado con UEFI" \
|| echo "Sistema iniciado con BIOS"
```

---

# Consultar la partición EFI

```bash
findmnt /boot/efi
```

También:

```bash
lsblk -f
```

Ejemplo:

```text
NAME        FSTYPE FSVER LABEL UUID                                 MOUNTPOINTS
nvme0n1
├─nvme0n1p1 vfat   FAT32       A1B2-C3D4                            /boot/efi
├─nvme0n1p2 xfs                1234-5678                            /boot
└─nvme0n1p3 LVM2_member
```

---

# Etapa 2: POST y detección de hardware

Después del encendido, el firmware ejecuta el:

```text
Power-On Self-Test
```

Abreviado:

```text
POST
```

El POST comprueba componentes básicos como:

- Procesador.
- Memoria RAM.
- Teclado.
- Controladores.
- Tarjetas de expansión.
- Discos.
- Dispositivos de arranque.

---

# Selección del dispositivo de arranque

El firmware consulta el orden configurado de dispositivos.

Ejemplo:

```text
1. Disco NVMe
2. Disco SATA
3. Unidad USB
4. Red PXE
```

Después intenta iniciar desde el primer dispositivo válido.

Si el dispositivo no contiene un cargador de arranque válido, puede aparecer un mensaje como:

```text
No bootable device
```

o:

```text
Operating system not found
```

---

# Etapa 3: Cargador de arranque GRUB2

GRUB significa:

```text
GRand Unified Bootloader
```

En RHEL se utiliza principalmente:

```text
GRUB2
```

GRUB2 se encarga de:

- Mostrar el menú de arranque.
- Mostrar los kernels instalados.
- Permitir seleccionar un kernel.
- Pasar parámetros al kernel.
- Cargar el kernel en memoria.
- Cargar la imagen `initramfs`.
- Transferir el control al kernel.

---

# Menú de GRUB2

El menú puede mostrar entradas como:

```text
Red Hat Enterprise Linux
Red Hat Enterprise Linux con kernel anterior
Rescue kernel
```

Ejemplo conceptual:

```text
GNU GRUB

Red Hat Enterprise Linux (5.14.0-500.el9.x86_64)
Red Hat Enterprise Linux (5.14.0-480.el9.x86_64)
Red Hat Enterprise Linux (0-rescue)
```

---

# Elementos cargados por GRUB2

Normalmente GRUB2 carga dos archivos principales:

```text
Kernel
```

y:

```text
initramfs
```

Ejemplo:

```text
/boot/vmlinuz-5.14.0-500.el9.x86_64
/boot/initramfs-5.14.0-500.el9.x86_64.img
```

---

# Consultar los kernels disponibles

```bash
ls -lh /boot/vmlinuz-*
```

Consultar las imágenes `initramfs`:

```bash
ls -lh /boot/initramfs-*
```

---

# Consultar el kernel predeterminado

```bash
grubby --default-kernel
```

Ejemplo:

```text
/boot/vmlinuz-5.14.0-500.el9.x86_64
```

---

# Consultar todas las entradas de kernel

```bash
grubby --info=ALL
```

La salida puede mostrar:

- Índice.
- Ruta del kernel.
- Argumentos.
- Ruta de `initramfs`.
- Título.
- Identificador.

---

# Archivos relacionados con GRUB2

Archivos y directorios importantes:

```text
/etc/default/grub
/etc/grub.d/
/boot/grub2/grub.cfg
/boot/loader/entries/
/boot/efi/EFI/redhat/
```

La ubicación exacta puede variar según:

- La versión de RHEL.
- BIOS o UEFI.
- El esquema de arranque.
- La utilización de Boot Loader Specification.

---

# Archivo `/etc/default/grub`

Contiene opciones generales utilizadas para generar la configuración.

Ejemplo:

```ini
GRUB_TIMEOUT=5
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="rhgb quiet"
GRUB_DISABLE_RECOVERY="true"
```

---

# No editar directamente `grub.cfg`

El archivo:

```text
/boot/grub2/grub.cfg
```

normalmente se genera automáticamente.

No debe editarse manualmente porque los cambios pueden perderse al:

- Instalar un kernel.
- Regenerar GRUB.
- Actualizar paquetes.
- Cambiar la configuración.

La configuración permanente debe realizarse mediante las herramientas y archivos apropiados.

---

# Etapa 4: Carga del kernel de Linux

Después de cargar los archivos necesarios, GRUB2 transfiere el control al kernel.

El kernel es el componente central de Linux.

Sus responsabilidades incluyen:

- Administrar la CPU.
- Administrar la memoria.
- Administrar procesos.
- Detectar hardware.
- Cargar controladores.
- Proporcionar acceso a dispositivos.
- Administrar sistemas de archivos.
- Implementar redes.
- Aplicar mecanismos de seguridad.

---

# Archivo del kernel

El kernel normalmente se encuentra en `/boot`:

```text
/boot/vmlinuz-versión
```

Ejemplo:

```text
/boot/vmlinuz-5.14.0-500.el9.x86_64
```

---

# Consultar el kernel en ejecución

```bash
uname -r
```

Ejemplo:

```text
5.14.0-500.el9.x86_64
```

Mostrar información más amplia:

```bash
uname -a
```

---

# Parámetros del kernel

GRUB2 pasa parámetros al kernel durante el arranque.

Consultar los parámetros utilizados en el arranque actual:

```bash
cat /proc/cmdline
```

Ejemplo:

```text
BOOT_IMAGE=(hd0,gpt2)/vmlinuz-5.14.0-500.el9.x86_64 root=/dev/mapper/rhel-root ro crashkernel=1G-4G:192M rhgb quiet
```

---

# Parámetros comunes

| Parámetro | Función |
|---|---|
| `root=` | Indica el sistema de archivos raíz |
| `ro` | Monta inicialmente la raíz en solo lectura |
| `rw` | Solicita montaje en lectura y escritura |
| `quiet` | Reduce mensajes visibles |
| `rhgb` | Activa el arranque gráfico de Red Hat |
| `rd.break` | Interrumpe el proceso dentro de initramfs |
| `systemd.unit=` | Define un target temporal |
| `crashkernel=` | Reserva memoria para kdump |
| `selinux=0` | Desactiva SELinux durante el arranque |
| `enforcing=0` | Inicia SELinux en modo permisivo |

> Desactivar SELinux no debe utilizarse como solución normal para problemas de arranque.

---

# Mensajes del kernel

El kernel genera mensajes mientras detecta y configura el hardware.

Consultar:

```bash
dmesg
```

Con marcas de tiempo legibles:

```bash
dmesg -T
```

Mostrar las últimas líneas:

```bash
dmesg -T | tail -n 50
```

---

# Consultar mensajes del kernel con Journal

```bash
journalctl -k
```

Solo el arranque actual:

```bash
journalctl -k -b
```

Solo errores:

```bash
journalctl -k -b -p err
```

---

# Etapa 5: initramfs

`initramfs` significa:

```text
Initial RAM File System
```

Es un sistema de archivos temporal cargado en memoria durante las primeras etapas del arranque.

---

# ¿Por qué se necesita initramfs?

El kernel puede no tener incorporados todos los controladores necesarios para acceder al sistema raíz.

Por ejemplo, el sistema raíz podría encontrarse en:

- LVM.
- RAID.
- Un disco cifrado.
- Un dispositivo NVMe.
- Una SAN.
- iSCSI.
- Un sistema de archivos que requiere módulos adicionales.

`initramfs` proporciona temporalmente las herramientas necesarias.

---

# Contenido de initramfs

Puede contener:

- Módulos del kernel.
- Controladores de almacenamiento.
- Herramientas LVM.
- Herramientas RAID.
- Soporte para cifrado.
- Reglas udev.
- Scripts de inicialización.
- Utilidades de montaje.
- Configuración de red temprana.
- Herramientas de diagnóstico.

---

# Flujo de initramfs

```text
GRUB2
   │
   ├── Carga kernel
   └── Carga initramfs
            │
            ▼
     Kernel inicia initramfs
            │
            ▼
     Detecta almacenamiento
            │
            ▼
     Activa LVM, RAID o cifrado
            │
            ▼
     Localiza el sistema raíz
            │
            ▼
     Monta la raíz real
            │
            ▼
     Transfiere el control a systemd
```

---

# Consultar la imagen initramfs

```bash
ls -lh /boot/initramfs-$(uname -r).img
```

---

# Examinar su contenido

```bash
lsinitrd
```

Para una imagen específica:

```bash
lsinitrd \
/boot/initramfs-$(uname -r).img
```

Buscar módulos relacionados con LVM:

```bash
lsinitrd | grep -i lvm
```

Buscar controladores:

```bash
lsinitrd | grep -i nvme
```

---

# dracut

En RHEL, la herramienta utilizada para crear `initramfs` es:

```text
dracut
```

Consultar versión:

```bash
dracut --version
```

Recrear la imagen del kernel activo:

```bash
sudo dracut --force
```

Este tema se desarrollará con mayor profundidad en una lección posterior.

---

# Etapa 6: Montaje del sistema de archivos raíz

Después de detectar el almacenamiento, `initramfs` localiza el sistema de archivos raíz real.

El parámetro:

```text
root=
```

indica dónde se encuentra.

Ejemplo:

```text
root=/dev/mapper/rhel-root
```

También puede utilizarse un UUID:

```text
root=UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

---

# Cambio de raíz

Durante el arranque se produce una transición:

```text
Raíz temporal de initramfs
            │
            ▼
Sistema de archivos raíz real
```

Una vez disponible la raíz real:

- Se monta `/`.
- Se prepara el entorno.
- Se ejecuta el proceso inicial.
- El control pasa a `systemd`.

---

# Consultar el sistema raíz

```bash
findmnt /
```

Ejemplo:

```text
TARGET SOURCE                 FSTYPE OPTIONS
/      /dev/mapper/rhel-root  xfs    rw,relatime,seclabel
```

---

# Consultar dispositivos y sistemas de archivos

```bash
lsblk -f
```

También:

```bash
blkid
```

---

# Consultar parámetros de montaje

```bash
findmnt -o TARGET,SOURCE,FSTYPE,OPTIONS
```

---

# Etapa 7: Inicio de systemd

Después de montar el sistema raíz, el kernel inicia el primer proceso del espacio de usuario.

En sistemas RHEL modernos, este proceso es:

```text
systemd
```

Su identificador de proceso es:

```text
PID 1
```

---

# Verificar el proceso PID 1

```bash
ps -p 1 -o pid,comm,args
```

Ejemplo:

```text
PID COMMAND COMMAND
1   systemd /usr/lib/systemd/systemd --switched-root --system
```

También:

```bash
readlink -f /sbin/init
```

Puede mostrar:

```text
/usr/lib/systemd/systemd
```

---

# Funciones de systemd

`systemd` se encarga de:

- Montar sistemas de archivos.
- Activar swap.
- Iniciar servicios.
- Configurar dispositivos.
- Crear sockets.
- Administrar sesiones.
- Iniciar targets.
- Supervisar procesos.
- Registrar eventos mediante Journal.
- Llevar el sistema al estado operativo.

---

# Unidades de systemd

`systemd` administra diferentes tipos de unidades.

| Tipo | Extensión | Función |
|---|---|---|
| Servicio | `.service` | Procesos y servicios |
| Target | `.target` | Agrupación de unidades |
| Montaje | `.mount` | Sistemas de archivos |
| Automontaje | `.automount` | Montajes automáticos |
| Socket | `.socket` | Activación por socket |
| Temporizador | `.timer` | Ejecución programada |
| Dispositivo | `.device` | Hardware |
| Swap | `.swap` | Espacio de intercambio |
| Ruta | `.path` | Supervisión de rutas |

---

# Targets de systemd

Un target representa un estado operativo del sistema.

Ejemplos:

```text
multi-user.target
graphical.target
rescue.target
emergency.target
```

---

# Target predeterminado

Consultar:

```bash
systemctl get-default
```

Ejemplo en servidor:

```text
multi-user.target
```

Ejemplo en estación gráfica:

```text
graphical.target
```

---

# Relación con los antiguos runlevels

| Runlevel tradicional | Target de systemd |
|---:|---|
| 0 | `poweroff.target` |
| 1 | `rescue.target` |
| 2 | `multi-user.target` |
| 3 | `multi-user.target` |
| 4 | `multi-user.target` |
| 5 | `graphical.target` |
| 6 | `reboot.target` |

---

# Dependencias de un target

Consultar:

```bash
systemctl list-dependencies \
graphical.target
```

Para el target predeterminado:

```bash
systemctl list-dependencies \
"$(systemctl get-default)"
```

---

# Inicio de servicios

`systemd` inicia los servicios requeridos por el target.

Ejemplos:

```text
sshd.service
NetworkManager.service
chronyd.service
firewalld.service
crond.service
```

Consultar servicios activos:

```bash
systemctl list-units \
--type=service \
--state=running
```

---

# Servicios habilitados y activos

Un servicio habilitado no necesariamente está ejecutándose en este momento.

Un servicio activo no necesariamente está habilitado para el próximo arranque.

Consultar:

```bash
systemctl is-enabled sshd
```

```bash
systemctl is-active sshd
```

---

# Cadena completa de arranque

```text
Encendido
    │
    ▼
Firmware BIOS o UEFI
    │
    ▼
POST
    │
    ▼
Dispositivo de arranque
    │
    ▼
GRUB2
    │
    ├── Kernel
    └── initramfs
           │
           ▼
Detección de almacenamiento
           │
           ▼
Sistema raíz
           │
           ▼
systemd PID 1
           │
           ▼
Target predeterminado
           │
           ▼
Servicios
           │
           ▼
Inicio de sesión
```

---

# Inicio de sesión

Al finalizar el arranque puede aparecer:

## Consola de texto

Administrada normalmente mediante:

```text
getty
```

Ejemplo de unidad:

```text
getty@tty1.service
```

---

## Interfaz gráfica

Puede utilizar un display manager como:

- GDM.
- SDDM.
- LightDM.

En RHEL con GNOME es común:

```text
gdm.service
```

---

# Consultar el estado general del sistema

```bash
systemctl status
```

Este comando muestra:

- Estado del sistema.
- Cantidad de trabajos.
- Unidades cargadas.
- Unidades fallidas.
- Estado general.

---

# Analizar el arranque con Journal

Consultar el arranque actual:

```bash
journalctl -b
```

---

# Consultar el arranque anterior

```bash
journalctl -b -1
```

Listar arranques disponibles:

```bash
journalctl --list-boots
```

Ejemplo:

```text
-2  abcdef...  Mon 2026-07-20 08:00:00
-1  bcdefa...  Tue 2026-07-21 08:15:00
 0  cdefab...  Wed 2026-07-22 09:00:00
```

---

# Mostrar errores del arranque

```bash
journalctl -b -p err
```

Errores y alertas más graves:

```bash
journalctl -b \
-p warning
```

---

# Mostrar mensajes recientes

```bash
journalctl -b -n 100
```

Seguir mensajes en tiempo real:

```bash
journalctl -f
```

---

# Analizar el tiempo de arranque

```bash
systemd-analyze
```

Ejemplo:

```text
Startup finished in 4.201s (kernel) + 9.417s (userspace) = 13.618s
graphical.target reached after 9.100s in userspace
```

---

# Interpretar el resultado

```text
Firmware
Bootloader
Kernel
Userspace
```

No todos los sistemas muestran todas las fases.

Por ejemplo, en máquinas virtuales puede no aparecer el tiempo de firmware.

---

# Servicios más lentos

```bash
systemd-analyze blame
```

Ejemplo:

```text
5.402s NetworkManager-wait-online.service
2.011s firewalld.service
1.508s chronyd.service
```

---

# Cadena crítica

```bash
systemd-analyze critical-chain
```

Este comando muestra la secuencia de unidades que retrasaron la llegada al target.

Para un target específico:

```bash
systemd-analyze critical-chain \
multi-user.target
```

---

# Crear un gráfico del arranque

```bash
systemd-analyze plot \
> boot.svg
```

El archivo puede abrirse con un navegador gráfico.

```bash
xdg-open boot.svg
```

---

# Servicios fallidos

```bash
systemctl --failed
```

Solo servicios fallidos:

```bash
systemctl list-units \
--type=service \
--state=failed
```

---

# Analizar un servicio fallido

```bash
systemctl status nombre.service
```

Después:

```bash
journalctl -u nombre.service \
-b
```

Ejemplo:

```bash
systemctl status sshd
```

```bash
journalctl -u sshd -b
```

---

# Puntos de fallo del arranque

Cada etapa puede producir problemas diferentes.

| Etapa | Problemas comunes |
|---|---|
| Firmware | Disco no detectado, orden de arranque incorrecto |
| GRUB2 | Configuración dañada, entrada inexistente |
| Kernel | Kernel dañado, módulo incompatible |
| initramfs | Controlador faltante, LVM no activado |
| Raíz | UUID incorrecto, sistema de archivos dañado |
| systemd | Target incorrecto, servicio crítico fallido |
| Servicios | Dependencias, permisos, configuración |
| Inicio de sesión | getty o display manager fallido |

---

# Cómo identificar la etapa del fallo

## No aparece ningún mensaje de Linux

El problema puede estar en:

- Firmware.
- Hardware.
- Orden de arranque.
- Disco.
- Cargador de arranque.

---

## Aparece GRUB pero Linux no inicia

El problema puede estar en:

- Entrada de GRUB.
- Kernel.
- `initramfs`.
- Parámetros del kernel.
- Sistema raíz.

---

## Aparecen mensajes del kernel y luego falla

El problema puede estar en:

- Controladores.
- Almacenamiento.
- LVM.
- RAID.
- Cifrado.
- `initramfs`.

---

## El sistema entra en emergency mode

El problema puede estar en:

- `/etc/fstab`.
- Sistema de archivos.
- UUID.
- Montajes obligatorios.
- Dependencias de systemd.

---

## El sistema inicia pero no muestra interfaz gráfica

El problema puede estar en:

- `graphical.target`.
- GDM.
- Controlador gráfico.
- Xorg.
- Wayland.
- Paquetes gráficos.

---

# `/etc/fstab` y el proceso de arranque

El archivo:

```text
/etc/fstab
```

define sistemas de archivos que deben montarse.

Ejemplo:

```fstab
UUID=xxxx-xxxx /      xfs  defaults  0 0
UUID=yyyy-yyyy /boot  xfs  defaults  0 0
UUID=ZZZZ-ZZZZ /boot/efi vfat umask=0077 0 2
```

Un error puede provocar que el sistema entre en modo de emergencia.

---

# Verificar `/etc/fstab`

```bash
cat /etc/fstab
```

Validar sin reiniciar:

```bash
sudo mount -a
```

Con salida detallada:

```bash
sudo mount -av
```

> Nunca reinicies inmediatamente después de modificar `/etc/fstab`. Primero ejecuta `mount -a` o `mount -av`.

---

# Consultar UUID

```bash
blkid
```

También:

```bash
lsblk -f
```

Comparar los UUID reales con los definidos en `/etc/fstab`.

---

# Verificar unidades de montaje

```bash
systemctl list-units \
--type=mount
```

Consultar una unidad específica:

```bash
systemctl status boot.mount
```

---

# Kernel actual frente a kernel predeterminado

Kernel activo:

```bash
uname -r
```

Kernel predeterminado:

```bash
grubby --default-kernel
```

Estos valores pueden ser diferentes cuando:

- Se instaló un kernel nuevo.
- El sistema no se ha reiniciado.
- Se seleccionó temporalmente un kernel anterior.
- Se cambió la entrada predeterminada.

---

# Consultar paquetes de kernel

```bash
rpm -q kernel
```

También:

```bash
dnf list installed kernel
```

---

# Verificar espacio en `/boot`

```bash
df -h /boot
```

Consultar inodos:

```bash
df -i /boot
```

Un `/boot` lleno puede impedir:

- Instalar un kernel.
- Crear `initramfs`.
- Generar archivos de GRUB.
- Completar actualizaciones.

---

# Comandos esenciales de diagnóstico

| Comando | Función |
|---|---|
| `uname -r` | Mostrar kernel activo |
| `cat /proc/cmdline` | Mostrar parámetros del kernel |
| `ls /boot` | Consultar archivos de arranque |
| `grubby --default-kernel` | Mostrar kernel predeterminado |
| `grubby --info=ALL` | Mostrar entradas de kernel |
| `systemctl get-default` | Mostrar target predeterminado |
| `systemctl --failed` | Mostrar unidades fallidas |
| `journalctl -b` | Mostrar mensajes del arranque |
| `journalctl -b -1` | Mostrar arranque anterior |
| `journalctl -k -b` | Mostrar mensajes del kernel |
| `dmesg -T` | Mostrar mensajes del kernel |
| `systemd-analyze` | Mostrar tiempo de arranque |
| `systemd-analyze blame` | Mostrar unidades más lentas |
| `systemd-analyze critical-chain` | Mostrar cadena crítica |
| `findmnt /` | Mostrar sistema raíz |
| `lsblk -f` | Mostrar discos y sistemas de archivos |
| `blkid` | Mostrar UUID |
| `mount -a` | Validar `/etc/fstab` |

---

# Flujo de diagnóstico recomendado

```text
Identificar el síntoma
        │
        ▼
Determinar la etapa del fallo
        │
        ▼
Revisar mensajes visibles
        │
        ▼
Consultar journalctl -b
        │
        ▼
Consultar systemctl --failed
        │
        ▼
Revisar kernel y parámetros
        │
        ▼
Revisar discos y montajes
        │
        ▼
Revisar GRUB e initramfs
        │
        ▼
Aplicar corrección
        │
        ▼
Verificar antes de reiniciar
```

---

# Ejemplo práctico de análisis

Supongamos que el servidor tarda demasiado en arrancar.

## Paso 1: Consultar tiempo total

```bash
systemd-analyze
```

---

## Paso 2: Identificar unidades lentas

```bash
systemd-analyze blame
```

---

## Paso 3: Consultar cadena crítica

```bash
systemd-analyze critical-chain
```

---

## Paso 4: Revisar servicios fallidos

```bash
systemctl --failed
```

---

## Paso 5: Consultar errores

```bash
journalctl -b -p err
```

---

## Paso 6: Analizar una unidad

```bash
systemctl status \
NetworkManager-wait-online.service
```

```bash
journalctl \
-u NetworkManager-wait-online.service \
-b
```

---

# Buenas prácticas RHCSA

✔ Comprender el orden completo de las etapas de arranque.

✔ Identificar si el sistema utiliza BIOS o UEFI.

✔ Consultar `/proc/cmdline` antes de cambiar parámetros.

✔ No editar manualmente `grub.cfg`.

✔ Mantener espacio suficiente en `/boot`.

✔ Conservar varios kernels funcionales.

✔ Revisar `journalctl -b` después de un fallo.

✔ Utilizar `systemctl --failed` para localizar unidades problemáticas.

✔ Validar `/etc/fstab` antes de reiniciar.

✔ Realizar cambios de arranque primero de manera temporal.

✔ Mantener respaldos de archivos críticos.

✔ Documentar cambios en GRUB, kernel y montajes.

---

# Errores comunes

## Confundir BIOS con GRUB

BIOS o UEFI es el firmware.

GRUB2 es el cargador del sistema operativo.

---

## Pensar que GRUB2 es el kernel

GRUB2 carga el kernel, pero no forma parte del kernel.

---

## Confundir initramfs con el sistema raíz

`initramfs` es un sistema temporal en memoria.

El sistema raíz real se monta después.

---

## Editar directamente `grub.cfg`

Los cambios pueden perderse y la configuración puede quedar inconsistente.

---

## Eliminar kernels manualmente desde `/boot`

Esto puede dejar paquetes registrados incorrectamente y entradas de arranque dañadas.

Los kernels deben administrarse mediante DNF o RPM.

---

## Reiniciar después de modificar `/etc/fstab` sin probar

Un error puede impedir que el sistema complete el arranque.

Siempre prueba:

```bash
sudo mount -av
```

---

## Ignorar servicios fallidos

Un sistema puede llegar al login y todavía tener fallos importantes.

Consulta:

```bash
systemctl --failed
```

---

## Utilizar `selinux=0` como solución permanente

Desactivar SELinux puede ocultar el problema real y reducir la seguridad.

---

# Resumen rápido

```text
Arranque de Linux
    │
    ├── BIOS o UEFI
    │     └── Detecta hardware
    │
    ├── GRUB2
    │     ├── Carga kernel
    │     └── Carga initramfs
    │
    ├── Kernel
    │     ├── Administra hardware
    │     └── Inicia initramfs
    │
    ├── initramfs
    │     ├── Detecta almacenamiento
    │     └── Localiza la raíz
    │
    ├── Sistema raíz
    │     └── Monta /
    │
    ├── systemd
    │     ├── PID 1
    │     └── Activa target
    │
    └── Servicios
          └── Inicio de sesión
```

---

# Resumen

En esta lección aprendiste a:

- Comprender el proceso completo de arranque de Linux.
- Diferenciar BIOS y UEFI.
- Identificar la función de GRUB2.
- Comprender cómo se carga el kernel.
- Explicar la función de `initramfs`.
- Comprender el montaje del sistema raíz.
- Identificar a `systemd` como PID 1.
- Consultar el target predeterminado.
- Analizar mensajes y tiempos de arranque.
- Identificar posibles etapas de fallo.
- Utilizar comandos básicos de diagnóstico.

---

# Laboratorio práctico RHCSA

## Escenario 1: Identificar BIOS o UEFI

Ejecuta:

```bash
test -d /sys/firmware/efi \
&& echo "UEFI" \
|| echo "BIOS"
```

Si utilizas UEFI:

```bash
findmnt /boot/efi
```

---

## Escenario 2: Consultar el kernel

```bash
uname -r
```

```bash
grubby --default-kernel
```

```bash
grubby --info=ALL
```

Responde:

1. ¿Cuál es el kernel activo?
2. ¿Cuál es el kernel predeterminado?
3. ¿Cuántas entradas existen?
4. ¿Existe un kernel de rescate?

---

## Escenario 3: Examinar `/boot`

```bash
ls -lh /boot
```

Después:

```bash
ls -lh /boot/vmlinuz-*
```

```bash
ls -lh /boot/initramfs-*
```

Identifica qué imagen `initramfs` corresponde al kernel activo.

---

## Escenario 4: Consultar parámetros

```bash
cat /proc/cmdline
```

Identifica:

- `root=`
- `ro` o `rw`
- `rhgb`
- `quiet`
- `crashkernel`

---

## Escenario 5: Examinar initramfs

```bash
lsinitrd \
/boot/initramfs-$(uname -r).img \
| less
```

Busca:

```bash
lsinitrd | grep -i lvm
```

```bash
lsinitrd | grep -i xfs
```

---

## Escenario 6: Identificar systemd

```bash
ps -p 1 -o pid,comm,args
```

También:

```bash
readlink -f /sbin/init
```

---

## Escenario 7: Consultar el target

```bash
systemctl get-default
```

Consulta dependencias:

```bash
systemctl list-dependencies \
"$(systemctl get-default)"
```

---

## Escenario 8: Revisar el arranque

```bash
journalctl -b \
-n 100
```

Errores:

```bash
journalctl -b -p err
```

Mensajes del kernel:

```bash
journalctl -k -b \
-n 100
```

---

## Escenario 9: Analizar tiempos

```bash
systemd-analyze
```

```bash
systemd-analyze blame \
| head -n 20
```

```bash
systemd-analyze critical-chain
```

---

## Escenario 10: Verificar servicios fallidos

```bash
systemctl --failed
```

Para cada unidad fallida:

```bash
systemctl status nombre-unidad
```

```bash
journalctl -u nombre-unidad -b
```

---

## Escenario 11: Verificar el sistema raíz

```bash
findmnt /
```

```bash
lsblk -f
```

```bash
blkid
```

Identifica el dispositivo correspondiente a `/`.

---

## Escenario 12: Validar `/etc/fstab`

Crea un respaldo:

```bash
sudo cp \
/etc/fstab \
/etc/fstab.bak
```

Valida:

```bash
sudo mount -av
```

Comprueba que no existan errores.

---

## Escenario 13: Comprobar `/boot`

```bash
df -h /boot
```

```bash
df -i /boot
```

Consulta los kernels instalados:

```bash
rpm -q kernel
```

---

# Script opcional de diagnóstico

```bash
#!/bin/bash

REPORTE="$HOME/reporte-arranque.txt"

{
    echo "=================================================="
    echo "REPORTE DEL PROCESO DE ARRANQUE"
    echo "=================================================="
    echo

    echo "Fecha:"
    date
    echo

    echo "Hostname:"
    hostname
    echo

    echo "Firmware:"
    if [ -d /sys/firmware/efi ]; then
        echo "UEFI"
    else
        echo "BIOS"
    fi
    echo

    echo "Kernel activo:"
    uname -r
    echo

    echo "Kernel predeterminado:"
    grubby --default-kernel 2>/dev/null
    echo

    echo "Parámetros del kernel:"
    cat /proc/cmdline
    echo

    echo "Target predeterminado:"
    systemctl get-default
    echo

    echo "Sistema raíz:"
    findmnt /
    echo

    echo "Espacio en /boot:"
    df -h /boot
    echo

    echo "Tiempo de arranque:"
    systemd-analyze
    echo

    echo "Servicios fallidos:"
    systemctl --failed --no-pager
    echo

    echo "Errores del arranque:"
    journalctl -b -p err --no-pager
    echo

    echo "=================================================="
    echo "FIN DEL REPORTE"
    echo "=================================================="

} > "$REPORTE" 2>&1

echo "Reporte generado en: $REPORTE"
```

Guardar como:

```text
~/diagnosticar-arranque.sh
```

Asignar permisos:

```bash
chmod +x \
~/diagnosticar-arranque.sh
```

Ejecutar:

```bash
~/diagnosticar-arranque.sh
```

---

# Preguntas de repaso

1. ¿Cuáles son las principales etapas del proceso de arranque?
2. ¿Qué diferencia existe entre BIOS y UEFI?
3. ¿Qué es la EFI System Partition?
4. ¿Cuál es la función de GRUB2?
5. ¿Qué dos archivos principales carga GRUB2?
6. ¿Qué función cumple el kernel?
7. ¿Qué es `initramfs`?
8. ¿Por qué puede ser necesario `initramfs` para LVM?
9. ¿Qué proceso utiliza PID 1?
10. ¿Qué función cumple un target de systemd?
11. ¿Cómo se consulta el target predeterminado?
12. ¿Cómo se consultan los parámetros del kernel?
13. ¿Qué comando muestra los mensajes del arranque actual?
14. ¿Cómo se consulta el arranque anterior?
15. ¿Qué comando muestra los servicios fallidos?
16. ¿Qué comando muestra los servicios más lentos?
17. ¿Por qué un error en `/etc/fstab` puede impedir el arranque?
18. ¿Qué debe hacerse antes de reiniciar después de modificar `/etc/fstab`?
19. ¿Cómo se consulta el kernel activo?
20. ¿Cómo se consulta el kernel predeterminado?

---

# Desafío final

Realiza las siguientes tareas sin consultar los ejemplos:

1. Determina si el sistema inició con BIOS o UEFI.
2. Identifica el kernel activo.
3. Identifica el kernel predeterminado.
4. Lista todas las entradas de kernel.
5. Examina los parámetros del arranque actual.
6. Identifica el dispositivo raíz.
7. Consulta el target predeterminado.
8. Lista las dependencias del target.
9. Consulta los errores del arranque actual.
10. Consulta los mensajes del kernel.
11. Identifica los cinco servicios más lentos.
12. Consulta las unidades fallidas.
13. Verifica el espacio disponible en `/boot`.
14. Valida `/etc/fstab`.
15. Genera un reporte con los resultados.

> **Objetivo general:** comprender en profundidad el proceso de arranque de Linux, desde la inicialización del firmware hasta el inicio de servicios administrados por `systemd`. Este conocimiento permite localizar rápidamente la etapa donde ocurre un fallo y constituye la base para las tareas de recuperación, administración de GRUB2 y diagnóstico requeridas en entornos RHEL y en el examen **RHCSA**.
