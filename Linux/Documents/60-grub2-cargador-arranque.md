# 60. GRUB2 y el Cargador de Arranque

> **Módulo 9: Arranque y Recuperación**  
> **Página 60 del Manual RHCSA**

---

# Objetivos de aprendizaje

Al finalizar esta lección serás capaz de:

- Comprender la función de GRUB2.
- Identificar las etapas en las que participa durante el arranque.
- Diferenciar el funcionamiento de GRUB2 en BIOS y UEFI.
- Consultar las entradas de arranque disponibles.
- Identificar el kernel predeterminado.
- Seleccionar temporalmente otro kernel.
- Editar temporalmente una entrada de arranque.
- Agregar y eliminar parámetros del kernel.
- Administrar entradas mediante `grubby`.
- Comprender la estructura Boot Loader Specification.
- Regenerar la configuración de GRUB2 cuando corresponda.
- Diagnosticar problemas comunes del cargador de arranque.
- Aplicar buenas prácticas de administración y recuperación.

---

# Introducción

GRUB2 es el cargador de arranque utilizado habitualmente en Red Hat Enterprise Linux.

Su función principal es iniciar el sistema operativo.

GRUB2 se ejecuta después del firmware y antes del kernel.

```text
Encendido
    │
    ▼
BIOS o UEFI
    │
    ▼
GRUB2
    │
    ├── Selecciona una entrada
    ├── Carga el kernel
    ├── Carga initramfs
    └── Pasa parámetros al kernel
            │
            ▼
        Kernel Linux
```

Sin un cargador funcional, el firmware puede detectar el disco, pero el sistema operativo no podrá iniciarse correctamente.

---

# ¿Qué significa GRUB?

GRUB significa:

```text
GRand Unified Bootloader
```

La versión moderna se conoce como:

```text
GRUB2
```

GRUB2 puede iniciar:

- Diferentes versiones del kernel.
- Diferentes sistemas operativos.
- Entradas de recuperación.
- Kernels con parámetros temporales.
- Sistemas desde diferentes particiones o discos.

---

# Funciones principales de GRUB2

GRUB2 se encarga de:

- Presentar el menú de arranque.
- Leer las entradas configuradas.
- Seleccionar el kernel predeterminado.
- Permitir elegir otro kernel.
- Cargar el archivo `vmlinuz`.
- Cargar la imagen `initramfs`.
- Pasar argumentos al kernel.
- Iniciar un entorno de rescate.
- Permitir modificaciones temporales.
- Transferir el control al kernel.

---

# Posición de GRUB2 en el arranque

```text
Firmware
   │
   ▼
Localiza el cargador
   │
   ▼
GRUB2
   │
   ├── Menú
   ├── Entrada de arranque
   ├── Kernel
   ├── initramfs
   └── Parámetros
          │
          ▼
       Kernel
```

---

# GRUB2 en sistemas BIOS

En un sistema BIOS tradicional, el firmware lee el código inicial desde el disco configurado como dispositivo de arranque.

Normalmente intervienen:

- El MBR.
- El código principal de GRUB2.
- El directorio `/boot/grub2`.
- El archivo de configuración de GRUB.

Flujo conceptual:

```text
BIOS
  │
  ▼
MBR del disco
  │
  ▼
Código de GRUB2
  │
  ▼
/boot/grub2
  │
  ▼
Menú de arranque
```

---

# GRUB2 en sistemas UEFI

En sistemas UEFI, el firmware no depende directamente del código de arranque almacenado en el MBR.

UEFI carga un ejecutable desde la:

```text
EFI System Partition
```

La partición EFI suele estar montada en:

```text
/boot/efi
```

Flujo conceptual:

```text
UEFI
  │
  ▼
Variable de arranque del firmware
  │
  ▼
EFI System Partition
  │
  ▼
Archivo ejecutable .efi
  │
  ▼
GRUB2
  │
  ▼
Kernel
```

---

# Identificar BIOS o UEFI

```bash
if [ -d /sys/firmware/efi ]; then
    echo "El sistema arrancó mediante UEFI"
else
    echo "El sistema arrancó mediante BIOS"
fi
```

También:

```bash
ls /sys/firmware/efi
```

Si el directorio existe, el sistema actual probablemente inició mediante UEFI.

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
NAME        FSTYPE LABEL UUID       MOUNTPOINTS
nvme0n1
├─nvme0n1p1 vfat         A12B-C34D  /boot/efi
├─nvme0n1p2 xfs          ...        /boot
└─nvme0n1p3 LVM2_member
```

---

# Archivos cargados por GRUB2

GRUB2 carga principalmente:

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

# Verificar el kernel activo

```bash
uname -r
```

---

# Verificar el kernel predeterminado

```bash
grubby --default-kernel
```

Ejemplo:

```text
/boot/vmlinuz-5.14.0-500.el9.x86_64
```

El kernel activo y el predeterminado pueden ser diferentes.

Esto ocurre cuando:

- Se instaló un kernel nuevo y aún no se ha reiniciado.
- Se inició manualmente con un kernel anterior.
- Se cambió la entrada predeterminada.
- Se inició con una entrada de rescate.

---

# Consultar todas las entradas de arranque

```bash
grubby --info=ALL
```

La salida puede incluir:

```text
index=0
kernel="/boot/vmlinuz-5.14.0-500.el9.x86_64"
args="ro crashkernel=1G-4G:192M rhgb quiet"
root="/dev/mapper/rhel-root"
initrd="/boot/initramfs-5.14.0-500.el9.x86_64.img"
title="Red Hat Enterprise Linux"
id="..."
```

---

# Consultar una entrada específica

Por índice:

```bash
grubby --info=0
```

Por ruta del kernel:

```bash
grubby \
--info=/boot/vmlinuz-$(uname -r)
```

---

# Mostrar solo títulos

```bash
grubby --info=ALL | grep '^title='
```

---

# Mostrar solo kernels

```bash
grubby --info=ALL | grep '^kernel='
```

---

# Mostrar identificadores

```bash
grubby --info=ALL | grep '^id='
```

---

# Índices de GRUB

Cada entrada puede tener un índice.

Ejemplo:

```text
index=0
index=1
index=2
```

Normalmente:

```text
0
```

representa la primera entrada.

Sin embargo, es preferible administrar entradas mediante:

- Ruta del kernel.
- Identificador.
- Herramienta `grubby`.

Esto reduce el riesgo de seleccionar una entrada incorrecta después de una actualización.

---

# Menú de GRUB2

El menú puede mostrar entradas como:

```text
Red Hat Enterprise Linux (5.14.0-500.el9.x86_64)
Red Hat Enterprise Linux (5.14.0-480.el9.x86_64)
Red Hat Enterprise Linux (0-rescue)
```

Desde este menú se puede:

- Iniciar con la entrada predeterminada.
- Seleccionar otro kernel.
- Editar temporalmente una entrada.
- Acceder a un kernel de rescate.
- Abrir la consola de GRUB cuando esté permitido.

---

# Mostrar el menú durante el arranque

Dependiendo de la configuración, el menú puede estar oculto.

Para mostrarlo se puede intentar:

```text
Esc
```

o mantener presionada:

```text
Shift
```

El comportamiento puede variar según:

- BIOS o UEFI.
- Versión de GRUB2.
- Configuración del tiempo de espera.
- Consola utilizada.
- Máquina física o virtual.

---

# Seleccionar un kernel anterior

En el menú de GRUB:

1. Detén la cuenta regresiva.
2. Selecciona una entrada anterior.
3. Presiona `Enter`.
4. Espera a que el sistema inicie.
5. Verifica el kernel activo.

```bash
uname -r
```

Esto resulta útil cuando:

- Un kernel nuevo no inicia.
- Existe incompatibilidad con un controlador.
- Falló la creación de `initramfs`.
- Se necesita recuperar el sistema.

---

# Entrada de rescate

RHEL suele generar una entrada de rescate.

Puede aparecer con un nombre similar a:

```text
Red Hat Enterprise Linux (0-rescue-...)
```

Consultar:

```bash
grubby --info=ALL | grep -i rescue
```

Archivos posibles:

```text
/boot/vmlinuz-0-rescue-IDENTIFICADOR
/boot/initramfs-0-rescue-IDENTIFICADOR.img
```

---

# Edición temporal de una entrada

GRUB2 permite modificar una entrada solamente para el siguiente arranque.

Procedimiento:

1. Accede al menú de GRUB.
2. Selecciona una entrada.
3. Presiona:

```text
e
```

4. Localiza la línea que comienza con:

```text
linux
```

o en algunos sistemas:

```text
linuxefi
```

5. Agrega o elimina parámetros.
6. Inicia con:

```text
Ctrl+x
```

En algunos entornos también puede utilizarse:

```text
F10
```

---

# Naturaleza temporal de los cambios

Los cambios realizados desde el menú no modifican permanentemente los archivos del sistema.

```text
Editar entrada
      │
      ▼
Arrancar una vez
      │
      ▼
Reiniciar
      │
      ▼
Configuración original restaurada
```

Esto permite probar parámetros de manera segura antes de aplicarlos permanentemente.

---

# Ejemplo: iniciar sin interfaz gráfica

Editar la línea del kernel y agregar:

```text
systemd.unit=multi-user.target
```

Después iniciar con:

```text
Ctrl+x
```

El cambio se aplicará únicamente a ese arranque.

---

# Ejemplo: iniciar en rescue target

Agregar:

```text
systemd.unit=rescue.target
```

---

# Ejemplo: iniciar en emergency target

Agregar:

```text
systemd.unit=emergency.target
```

---

# Ejemplo: interrumpir initramfs

Agregar:

```text
rd.break
```

Este parámetro se utiliza frecuentemente para:

- Recuperar la contraseña de `root`.
- Examinar el sistema raíz.
- Corregir archivos antes de iniciar completamente.
- Trabajar dentro del entorno de `initramfs`.

---

# Ejemplo: desactivar el arranque silencioso

Eliminar temporalmente:

```text
rhgb
```

y:

```text
quiet
```

Esto permite mostrar más mensajes durante el arranque.

Puede ayudar a identificar:

- Errores de dispositivos.
- Problemas de montaje.
- Unidades fallidas.
- Fallos de `initramfs`.
- Esperas prolongadas.

---

# Parámetros comunes del kernel

| Parámetro | Función |
|---|---|
| `root=` | Define el sistema raíz |
| `ro` | Monta inicialmente la raíz en solo lectura |
| `rw` | Solicita lectura y escritura |
| `quiet` | Reduce mensajes |
| `rhgb` | Activa la pantalla gráfica de arranque |
| `rd.break` | Interrumpe el proceso en initramfs |
| `systemd.unit=` | Define un target temporal |
| `enforcing=0` | Inicia SELinux en permisivo |
| `selinux=0` | Desactiva SELinux |
| `nomodeset` | Desactiva configuración avanzada de vídeo |
| `rd.lvm.lv=` | Define un volumen lógico requerido |
| `rd.luks.uuid=` | Define un dispositivo cifrado |
| `crashkernel=` | Reserva memoria para kdump |

> `selinux=0` no debe utilizarse como solución habitual. Puede requerir un reetiquetado completo antes de volver a activar SELinux.

---

# Consola de comandos de GRUB

Desde el menú puede abrirse la consola de GRUB con:

```text
c
```

La consola utiliza comandos propios de GRUB, no comandos normales de Bash.

Ejemplos:

```text
ls
```

```text
set
```

```text
ls (hd0,gpt1)/
```

```text
ls (hd0,gpt2)/boot/
```

---

# Nomenclatura de discos en GRUB

GRUB utiliza una nomenclatura diferente a Linux.

Ejemplo:

```text
(hd0)
```

Primer disco detectado.

```text
(hd0,gpt1)
```

Primera partición GPT del primer disco.

```text
(hd0,msdos1)
```

Primera partición MBR del primer disco.

---

# Comparación de nombres

| Linux | GRUB |
|---|---|
| `/dev/sda` | `(hd0)` |
| `/dev/sda1` | `(hd0,gpt1)` o `(hd0,msdos1)` |
| `/dev/nvme0n1` | `(hd0)` |
| `/dev/nvme0n1p2` | `(hd0,gpt2)` |

La relación exacta depende del orden en que GRUB detecta los dispositivos.

---

# Consultar variables de GRUB

Dentro de la consola:

```text
set
```

Puede mostrar variables como:

```text
prefix
root
cmdpath
pager
```

---

# Variable `root` de GRUB

La variable `root` de GRUB no representa necesariamente el sistema de archivos raíz de Linux.

En GRUB puede indicar la partición desde la cual se leen:

- Kernel.
- `initramfs`.
- Archivos de configuración.
- Módulos de GRUB.

Ejemplo:

```text
root=(hd0,gpt2)
```

---

# Variable `prefix`

Puede apuntar al directorio de GRUB:

```text
prefix=(hd0,gpt2)/grub2
```

Si esta variable es incorrecta, GRUB puede no encontrar sus módulos o su configuración.

---

# Comando `grubby`

En RHEL, `grubby` es la herramienta recomendada para administrar entradas del cargador y parámetros persistentes del kernel.

Permite:

- Consultar entradas.
- Cambiar el kernel predeterminado.
- Agregar argumentos.
- Eliminar argumentos.
- Actualizar una entrada.
- Actualizar todas las entradas.

Red Hat recomienda utilizar `grubby` para cambiar el kernel predeterminado y administrar argumentos permanentes de las entradas de arranque. :contentReference[oaicite:0]{index=0}

---

# Consultar ayuda de `grubby`

```bash
grubby --help
```

Manual:

```bash
man grubby
```

---

# Cambiar el kernel predeterminado

Primero consulta las entradas:

```bash
grubby --info=ALL
```

Después establece una ruta:

```bash
sudo grubby \
--set-default \
/boot/vmlinuz-VERSION
```

Ejemplo:

```bash
sudo grubby \
--set-default \
/boot/vmlinuz-5.14.0-480.el9.x86_64
```

Verifica:

```bash
grubby --default-kernel
```

---

# Seleccionar la primera entrada como predeterminada

```bash
sudo grubby --set-default-index=0
```

Verifica:

```bash
grubby --default-index
```

Sin embargo, utilizar la ruta del kernel suele ser más claro.

---

# Agregar un argumento a todas las entradas

Sintaxis:

```bash
sudo grubby \
--update-kernel=ALL \
--args="argumento"
```

Ejemplo:

```bash
sudo grubby \
--update-kernel=ALL \
--args="audit=1"
```

---

# Agregar un argumento al kernel activo

```bash
sudo grubby \
--update-kernel=/boot/vmlinuz-$(uname -r) \
--args="audit=1"
```

---

# Eliminar un argumento

```bash
sudo grubby \
--update-kernel=ALL \
--remove-args="audit=1"
```

---

# Agregar varios argumentos

```bash
sudo grubby \
--update-kernel=ALL \
--args="console=tty0 console=ttyS0,115200"
```

---

# Eliminar varios argumentos

```bash
sudo grubby \
--update-kernel=ALL \
--remove-args="rhgb quiet"
```

---

# Verificar argumentos configurados

```bash
grubby --info=ALL
```

También:

```bash
cat /proc/cmdline
```

Recuerda:

- `grubby --info=ALL` muestra la configuración almacenada.
- `/proc/cmdline` muestra los parámetros utilizados en el arranque actual.

---

# Cambios almacenados frente a cambios activos

```text
Modificar con grubby
        │
        ▼
Entrada actualizada
        │
        ▼
Reinicio requerido
        │
        ▼
Nuevo parámetro activo
```

Agregar un parámetro con `grubby` no modifica el kernel que ya está ejecutándose.

---

# Boot Loader Specification

RHEL moderno utiliza el estándar:

```text
Boot Loader Specification
```

Abreviado:

```text
BLS
```

Las entradas pueden almacenarse en:

```text
/boot/loader/entries/
```

---

# Consultar entradas BLS

```bash
ls -l /boot/loader/entries/
```

Ejemplo:

```text
machine-id-5.14.0-500.el9.x86_64.conf
machine-id-5.14.0-480.el9.x86_64.conf
machine-id-0-rescue.conf
```

---

# Examinar una entrada BLS

```bash
sudo cat \
/boot/loader/entries/*.conf
```

Ejemplo conceptual:

```ini
title Red Hat Enterprise Linux
version 5.14.0-500.el9.x86_64
linux /vmlinuz-5.14.0-500.el9.x86_64
initrd /initramfs-5.14.0-500.el9.x86_64.img
options root=/dev/mapper/rhel-root ro rhgb quiet
grub_users $grub_users
grub_arg --unrestricted
grub_class rhel
```

---

# Campos principales de una entrada BLS

| Campo | Descripción |
|---|---|
| `title` | Nombre mostrado en el menú |
| `version` | Versión del kernel |
| `linux` | Ruta del kernel |
| `initrd` | Ruta de initramfs |
| `options` | Parámetros del kernel |
| `id` | Identificador de la entrada |
| `grub_users` | Usuarios autorizados |
| `grub_class` | Clase visual o lógica |

---

# Ventajas de BLS

- Una entrada por kernel.
- Menor necesidad de regenerar todo el menú.
- Mejor integración con la instalación de kernels.
- Administración más sencilla.
- Compatibilidad con `grubby`.
- Reducción del riesgo de sobrescribir entradas.

---

# Configuración general de GRUB

Archivo principal:

```text
/etc/default/grub
```

Mostrar:

```bash
cat /etc/default/grub
```

Ejemplo:

```ini
GRUB_TIMEOUT=5
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="rhgb quiet"
GRUB_DISABLE_RECOVERY="true"
GRUB_ENABLE_BLSCFG=true
```

---

# Parámetros frecuentes de `/etc/default/grub`

| Variable | Función |
|---|---|
| `GRUB_TIMEOUT` | Tiempo de espera del menú |
| `GRUB_DEFAULT` | Entrada predeterminada |
| `GRUB_TIMEOUT_STYLE` | Forma de mostrar el menú |
| `GRUB_CMDLINE_LINUX` | Parámetros generales del kernel |
| `GRUB_TERMINAL_OUTPUT` | Salida de terminal |
| `GRUB_ENABLE_BLSCFG` | Habilita entradas BLS |
| `GRUB_DISABLE_SUBMENU` | Controla submenús |
| `GRUB_DISABLE_RECOVERY` | Controla entradas de recuperación |

---

# Tiempo de espera

Ejemplo:

```ini
GRUB_TIMEOUT=10
```

Esto configura una espera de diez segundos.

---

# Estilo del menú

Mostrar menú:

```ini
GRUB_TIMEOUT_STYLE=menu
```

Ocultarlo:

```ini
GRUB_TIMEOUT_STYLE=hidden
```

---

# Parámetros generales

Ejemplo:

```ini
GRUB_CMDLINE_LINUX="rhgb quiet audit=1"
```

Sin embargo, en RHEL moderno es preferible utilizar `grubby` para agregar o eliminar parámetros de las entradas existentes.

---

# Directorio `/etc/grub.d`

Contiene scripts utilizados para generar partes de la configuración.

```bash
ls -l /etc/grub.d/
```

Ejemplos posibles:

```text
00_header
01_users
10_linux
20_linux_xen
30_os-prober
40_custom
41_custom
```

---

# Archivo `40_custom`

Puede utilizarse para agregar entradas personalizadas.

```text
/etc/grub.d/40_custom
```

Ejemplo conceptual:

```text
menuentry 'Sistema personalizado' {
    linux /vmlinuz-personalizado root=/dev/mapper/rhel-root ro
    initrd /initramfs-personalizado.img
}
```

Debe utilizarse únicamente cuando exista una necesidad clara.

---

# Archivo generado `grub.cfg`

Una ruta importante es:

```text
/boot/grub2/grub.cfg
```

Consultar:

```bash
sudo less /boot/grub2/grub.cfg
```

Este archivo contiene la configuración procesada por GRUB.

---

# No editar directamente `grub.cfg`

No se recomienda editar manualmente:

```text
/boot/grub2/grub.cfg
```

porque:

- Es un archivo generado.
- Los cambios pueden perderse.
- Puede quedar inconsistente.
- Una actualización puede sobrescribirlo.
- Un error puede impedir el arranque.

---

# Regenerar la configuración

Cuando corresponda:

```bash
sudo grub2-mkconfig \
-o /boot/grub2/grub.cfg
```

En RHEL 9, tanto en BIOS como en UEFI, la documentación actual de Red Hat utiliza `/boot/grub2/grub.cfg` como destino al reconstruir la configuración. :contentReference[oaicite:1]{index=1}

---

# Crear respaldo antes de regenerar

```bash
sudo cp \
/boot/grub2/grub.cfg \
/boot/grub2/grub.cfg.bak
```

Después:

```bash
sudo grub2-mkconfig \
-o /boot/grub2/grub.cfg
```

---

# Verificar sintaxis generada

```bash
sudo grub2-script-check \
/boot/grub2/grub.cfg
```

Si no aparece salida, no se detectaron errores de sintaxis básicos.

---

# `grub2-mkconfig` y BLS

En sistemas con BLS habilitado, `grub.cfg` puede actuar como configuración principal que carga las entradas almacenadas en:

```text
/boot/loader/entries/
```

Por ello:

- No todo el contenido del kernel aparece directamente como bloques tradicionales.
- Instalar un kernel puede crear una nueva entrada BLS.
- `grubby` puede modificar las entradas sin recrear manualmente todo el archivo.

---

# Entorno persistente de GRUB

GRUB puede guardar variables en su archivo de entorno.

Consultar:

```bash
grub2-editenv list
```

Variables posibles:

```text
saved_entry
menu_auto_hide
boot_success
boot_indeterminate
```

---

# Mostrar la entrada guardada

```bash
grub2-editenv list | grep saved_entry
```

---

# Menú oculto automáticamente

En algunos sistemas puede existir:

```text
menu_auto_hide=1
```

Para desactivar temporalmente el ocultamiento automático:

```bash
sudo grub2-editenv - unset menu_auto_hide
```

Antes de realizar cambios, revisa:

```bash
grub2-editenv list
```

---

# Verificar paquetes de GRUB

```bash
rpm -qa | grep '^grub2'
```

También:

```bash
dnf list installed 'grub2*'
```

Paquetes posibles:

```text
grub2-common
grub2-tools
grub2-tools-minimal
grub2-pc
grub2-efi-x64
```

La lista depende de la arquitectura y del tipo de firmware.

---

# Verificar archivos de un paquete

```bash
rpm -ql grub2-tools
```

Verificar integridad:

```bash
rpm -V grub2-tools
```

---

# Secure Boot

UEFI puede utilizar:

```text
Secure Boot
```

Secure Boot ayuda a verificar que los componentes de arranque estén firmados por una autoridad confiable.

En RHEL intervienen componentes como:

```text
shim
GRUB2
Kernel firmado
```

Flujo conceptual:

```text
Firmware UEFI
      │
      ▼
Verifica shim
      │
      ▼
shim verifica GRUB2
      │
      ▼
GRUB2 carga kernel firmado
```

---

# Consultar estado de Secure Boot

Si `mokutil` está instalado:

```bash
mokutil --sb-state
```

Ejemplo:

```text
SecureBoot enabled
```

o:

```text
SecureBoot disabled
```

---

# Consultar variables UEFI

Instala las herramientas si se requieren:

```bash
sudo dnf install efibootmgr
```

Consultar:

```bash
sudo efibootmgr
```

Salida conceptual:

```text
BootCurrent: 0001
BootOrder: 0001,0002,0003
Boot0001* Red Hat Enterprise Linux
Boot0002* UEFI Network
```

---

# Información detallada de UEFI

```bash
sudo efibootmgr -v
```

Puede mostrar:

- Orden de arranque.
- Entrada actual.
- Rutas EFI.
- Identificadores.
- Disco y partición.

---

# Precaución con `efibootmgr`

Modificar variables UEFI incorrectamente puede dejar el sistema sin una entrada de arranque válida.

Antes de cambiar:

```bash
sudo efibootmgr -v \
> ~/efibootmgr-respaldo.txt
```

---

# Protección de GRUB con contraseña

Sin protección, una persona con acceso físico puede:

- Editar parámetros.
- Iniciar en modo de rescate.
- Utilizar `rd.break`.
- Intentar obtener acceso administrativo.

GRUB2 puede protegerse mediante una contraseña.

---

# Generar hash de contraseña

```bash
grub2-mkpasswd-pbkdf2
```

Solicitará la contraseña y generará una cadena similar a:

```text
grub.pbkdf2.sha512.10000....
```

---

# Archivo de usuarios

En RHEL puede utilizarse:

```text
/etc/grub.d/01_users
```

o configuraciones relacionadas con:

```text
/etc/grub2.cfg
```

y entradas BLS.

La implementación exacta depende de la versión de RHEL.

---

# Consideraciones de protección

Una contraseña de GRUB puede restringir:

- Edición de entradas.
- Acceso a la consola de GRUB.
- Inicio de entradas protegidas.

No sustituye:

- Cifrado de disco.
- Seguridad física.
- Contraseña de firmware.
- Secure Boot.
- Políticas de acceso al servidor.

---

# Riesgo del acceso físico

Una persona con acceso físico podría:

- Arrancar desde USB.
- Usar un medio de instalación.
- Modificar el disco fuera del sistema.
- Extraer el almacenamiento.
- Cambiar parámetros de GRUB.

Por ello, en servidores críticos también se recomienda:

- Proteger BIOS o UEFI.
- Deshabilitar arranque externo.
- Utilizar cifrado.
- Controlar acceso físico.
- Habilitar Secure Boot cuando corresponda.

---

# Diagnóstico: no aparece el menú

Posibles causas:

- Tiempo de espera muy corto.
- Menú oculto.
- Solo existe una entrada.
- Configuración de terminal.
- Entrada automática guardada.
- Consola serial no configurada.

Comandos:

```bash
grep -E \
'GRUB_TIMEOUT|GRUB_TIMEOUT_STYLE|GRUB_TERMINAL' \
/etc/default/grub
```

```bash
grub2-editenv list
```

---

# Diagnóstico: kernel no aparece

Verifica si está instalado:

```bash
rpm -q kernel
```

Consulta los archivos:

```bash
ls -lh /boot/vmlinuz-*
```

Consulta entradas:

```bash
grubby --info=ALL
```

Consulta BLS:

```bash
ls -l /boot/loader/entries/
```

---

# Diagnóstico: falta initramfs

Verifica:

```bash
ls -lh \
/boot/initramfs-$(uname -r).img
```

Para otro kernel:

```bash
ls -lh \
/boot/initramfs-VERSION.img
```

Recrear:

```bash
sudo dracut \
--force \
/boot/initramfs-VERSION.img \
VERSION
```

---

# Diagnóstico: entrada apunta a archivos inexistentes

Consultar:

```bash
grubby --info=ALL
```

Después verificar:

```bash
ls -l /boot/vmlinuz-VERSION
```

```bash
ls -l /boot/initramfs-VERSION.img
```

Si los archivos no existen, la entrada no podrá iniciar correctamente.

---

# Diagnóstico: `/boot` lleno

```bash
df -h /boot
```

```bash
df -i /boot
```

Consultar kernels instalados:

```bash
rpm -q kernel
```

No elimines manualmente archivos de `/boot`.

Utiliza DNF para retirar kernels antiguos de manera controlada.

---

# Consultar el límite de kernels

```bash
grep '^installonly_limit' \
/etc/dnf/dnf.conf
```

Ejemplo:

```ini
installonly_limit=3
```

---

# Eliminar un kernel antiguo

Primero identifica el activo:

```bash
uname -r
```

Lista paquetes:

```bash
rpm -q kernel
```

Después elimina únicamente una versión antigua:

```bash
sudo dnf remove \
kernel-VERSION
```

> Nunca elimines el kernel que está ejecutándose ni el único kernel funcional conocido.

---

# Diagnóstico: GRUB muestra `grub rescue>`

Esto puede ocurrir cuando GRUB no encuentra:

- La partición correcta.
- El directorio de módulos.
- El archivo de configuración.
- La variable `prefix`.
- La partición `/boot`.

Comandos posibles en la consola:

```text
ls
```

```text
ls (hd0,gpt1)/
```

```text
ls (hd0,gpt2)/
```

```text
set
```

La recuperación completa depende de:

- BIOS o UEFI.
- Ubicación de `/boot`.
- LVM.
- Cifrado.
- Tabla de particiones.

---

# Diagnóstico: `unknown filesystem`

Puede indicar:

- Partición incorrecta.
- Sistema de archivos no soportado por el módulo cargado.
- Corrupción.
- Disco no detectado.
- Cambio en el orden de discos.

Consulta las particiones desde GRUB:

```text
ls
```

Después explora:

```text
ls (hd0,gpt1)/
```

---

# Diagnóstico: no se encuentra el sistema raíz

GRUB puede cargar correctamente el kernel, pero el kernel no localizar la raíz.

Revisa:

```bash
cat /proc/cmdline
```

```bash
grubby --info=ALL
```

```bash
lsblk -f
```

```bash
blkid
```

Posibles causas:

- Argumento `root=` incorrecto.
- UUID cambiado.
- Volumen LVM no disponible.
- Controlador faltante en `initramfs`.
- Dispositivo cifrado no activado.

---

# Diagnóstico: nuevo kernel no inicia

Procedimiento recomendado:

1. Reinicia.
2. Muestra el menú.
3. Selecciona un kernel anterior.
4. Inicia el sistema.
5. Verifica:

```bash
uname -r
```

6. Consulta errores del arranque anterior:

```bash
journalctl -b -1 -p err
```

7. Revisa la entrada:

```bash
grubby --info=ALL
```

8. Verifica `initramfs`.
9. Revisa espacio en `/boot`.
10. Reinstala o reconstruye según corresponda.

---

# Reinstalar componentes de GRUB

En caso de archivos dañados puede utilizarse DNF.

## BIOS

Ejemplo:

```bash
sudo dnf reinstall \
grub2-tools \
grub2-pc \
grub2-pc-modules
```

## UEFI

Ejemplo:

```bash
sudo dnf reinstall \
grub2-efi-x64 \
shim-x64 \
grub2-tools \
grub2-common
```

Los nombres exactos pueden variar según:

- Arquitectura.
- Versión de RHEL.
- Repositorios disponibles.

Red Hat diferencia los paquetes que deben reinstalarse en sistemas BIOS y UEFI. :contentReference[oaicite:2]{index=2}

---

# Reinstalar GRUB en BIOS

En sistemas BIOS puede utilizarse:

```bash
sudo grub2-install /dev/sda
```

Después:

```bash
sudo grub2-mkconfig \
-o /boot/grub2/grub.cfg
```

> Debes indicar el disco completo, no una partición como `/dev/sda1`.

---

# Precaución con `grub2-install`

Antes de ejecutarlo, confirma:

- Que el sistema usa BIOS.
- Cuál es el disco de arranque.
- Que `/boot` está montado.
- Que el disco correcto fue identificado.
- Que existe un respaldo o consola de recuperación.

```bash
lsblk
```

```bash
findmnt /boot
```

---

# UEFI y `grub2-install`

En sistemas RHEL UEFI modernos, no se recomienda asumir que `grub2-install` debe usarse de la misma forma que en BIOS.

Normalmente se administran:

- Paquetes `grub2-efi`.
- Paquete `shim`.
- Archivos de la EFI System Partition.
- Entradas de firmware.
- Configuración generada.

---

# Flujo de administración recomendado

```text
Necesidad de cambio
        │
        ▼
Consultar configuración actual
        │
        ▼
Realizar respaldo
        │
        ▼
Aplicar cambio con grubby
        │
        ▼
Verificar entradas
        │
        ▼
Conservar kernel alternativo
        │
        ▼
Reiniciar con consola disponible
        │
        ▼
Validar kernel y servicios
```

---

# Ejemplo: agregar un parámetro de manera segura

Consultar estado:

```bash
grubby --info=ALL
```

Agregar:

```bash
sudo grubby \
--update-kernel=ALL \
--args="audit=1"
```

Verificar:

```bash
grubby --info=ALL | grep args
```

Reiniciar:

```bash
sudo systemctl reboot
```

Después:

```bash
cat /proc/cmdline
```

---

# Ejemplo: eliminar `rhgb quiet`

Eliminar:

```bash
sudo grubby \
--update-kernel=ALL \
--remove-args="rhgb quiet"
```

Revisar:

```bash
grubby --info=ALL | grep args
```

Reiniciar y observar mensajes detallados.

Para restaurar:

```bash
sudo grubby \
--update-kernel=ALL \
--args="rhgb quiet"
```

---

# Ejemplo: establecer un kernel anterior

Listar:

```bash
grubby --info=ALL
```

Seleccionar:

```bash
sudo grubby \
--set-default \
/boot/vmlinuz-5.14.0-480.el9.x86_64
```

Verificar:

```bash
grubby --default-kernel
```

---

# Comprobar configuración antes de reiniciar

```bash
grubby --default-kernel
```

```bash
grubby --info=ALL
```

```bash
ls -lh /boot/vmlinuz-*
```

```bash
ls -lh /boot/initramfs-*
```

```bash
df -h /boot
```

```bash
sudo grub2-script-check \
/boot/grub2/grub.cfg
```

---

# Buenas prácticas RHCSA

✔ Identificar primero si el sistema utiliza BIOS o UEFI.

✔ Consultar el kernel activo y el predeterminado.

✔ Conservar al menos un kernel anterior funcional.

✔ Utilizar `grubby` para cambios persistentes.

✔ Probar cambios temporalmente desde el menú cuando sea posible.

✔ No editar directamente `grub.cfg`.

✔ Realizar respaldo antes de regenerar la configuración.

✔ Confirmar que el kernel y `initramfs` existan.

✔ Mantener espacio libre en `/boot`.

✔ Utilizar rutas completas al establecer el kernel predeterminado.

✔ Documentar parámetros agregados al kernel.

✔ Mantener acceso a consola durante cambios críticos.

✔ Proteger el acceso físico al servidor.

✔ No utilizar `grub2-install` sin identificar correctamente el modo de firmware.

---

# Errores comunes

## Confundir GRUB2 con el kernel

GRUB2 carga el kernel, pero no es el kernel.

---

## Cambiar el kernel predeterminado sin verificar archivos

La entrada puede apuntar a un kernel o `initramfs` inexistente.

---

## Editar directamente `/boot/grub2/grub.cfg`

Los cambios pueden perderse o dañar el arranque.

---

## Regenerar GRUB sin respaldo

Una configuración incorrecta puede dejar el sistema sin menú funcional.

---

## Usar una ruta UEFI antigua sin verificar la versión

En RHEL moderno, la administración de `grub.cfg` y BLS puede diferir de versiones anteriores.

Debes verificar:

```bash
cat /etc/redhat-release
```

```bash
findmnt /boot/efi
```

```bash
grubby --info=ALL
```

---

## Eliminar kernels manualmente

Borrar archivos desde `/boot` no actualiza correctamente la base RPM ni las entradas.

Utiliza DNF.

---

## Eliminar el kernel activo

Puede dejar el sistema sin una opción funcional para el siguiente reinicio.

---

## Aplicar cambios permanentes antes de probarlos

Primero realiza una modificación temporal en GRUB cuando sea posible.

---

## Confundir la variable `root` de GRUB con `/` de Linux

El `root` de GRUB representa la ubicación desde la que GRUB lee sus archivos.

---

## Ejecutar `grub2-install` en el disco equivocado

Puede sobrescribir el cargador de otro sistema o dejar el servidor sin arranque.

---

# Comandos importantes RHCSA

| Comando | Descripción |
|---|---|
| `uname -r` | Mostrar kernel activo |
| `grubby --default-kernel` | Mostrar kernel predeterminado |
| `grubby --default-index` | Mostrar índice predeterminado |
| `grubby --info=ALL` | Mostrar entradas |
| `grubby --info=0` | Mostrar una entrada |
| `grubby --set-default RUTA` | Establecer kernel predeterminado |
| `grubby --set-default-index=0` | Establecer índice predeterminado |
| `grubby --update-kernel=ALL --args=` | Agregar argumentos |
| `grubby --update-kernel=ALL --remove-args=` | Eliminar argumentos |
| `grub2-editenv list` | Consultar variables persistentes |
| `grub2-mkconfig -o ...` | Regenerar configuración |
| `grub2-script-check archivo` | Verificar sintaxis |
| `ls /boot/loader/entries` | Mostrar entradas BLS |
| `ls /boot/vmlinuz-*` | Mostrar kernels |
| `ls /boot/initramfs-*` | Mostrar imágenes initramfs |
| `cat /proc/cmdline` | Mostrar parámetros activos |
| `findmnt /boot` | Verificar montaje de `/boot` |
| `findmnt /boot/efi` | Verificar partición EFI |
| `efibootmgr -v` | Consultar entradas UEFI |
| `mokutil --sb-state` | Consultar Secure Boot |

---

# Resumen rápido

```text
GRUB2
  │
  ├── Firmware
  │     ├── BIOS
  │     └── UEFI
  │
  ├── Menú
  │     ├── Kernel actual
  │     ├── Kernel anterior
  │     └── Rescue
  │
  ├── Archivos
  │     ├── vmlinuz
  │     ├── initramfs
  │     ├── grub.cfg
  │     └── entradas BLS
  │
  ├── Administración
  │     ├── grubby
  │     ├── grub2-editenv
  │     └── grub2-mkconfig
  │
  └── Recuperación
        ├── Editar con e
        ├── rd.break
        ├── Kernel anterior
        └── Consola de GRUB
```

---

# Resumen

En esta lección aprendiste a:

- Comprender la función de GRUB2.
- Diferenciar su funcionamiento en BIOS y UEFI.
- Consultar los kernels y entradas disponibles.
- Identificar el kernel activo y el predeterminado.
- Seleccionar temporalmente otro kernel.
- Editar una entrada desde el menú.
- Utilizar parámetros temporales.
- Administrar cambios persistentes con `grubby`.
- Comprender las entradas BLS.
- Regenerar la configuración cuando corresponda.
- Consultar variables de GRUB.
- Reconocer la función de Secure Boot.
- Diagnosticar problemas comunes del cargador.

---

# Laboratorio práctico RHCSA

> Realiza las tareas en una máquina virtual o en un sistema con acceso a consola. Evita efectuar cambios críticos únicamente mediante SSH.

## Escenario 1: Identificar el tipo de firmware

```bash
if [ -d /sys/firmware/efi ]; then
    echo "UEFI"
else
    echo "BIOS"
fi
```

Si utiliza UEFI:

```bash
findmnt /boot/efi
```

---

## Escenario 2: Consultar el kernel activo

```bash
uname -r
```

Guarda el resultado:

```bash
uname -r > ~/kernel-activo.txt
```

---

## Escenario 3: Consultar el predeterminado

```bash
grubby --default-kernel
```

```bash
grubby --default-index
```

Compara con:

```bash
uname -r
```

---

## Escenario 4: Listar entradas

```bash
grubby --info=ALL
```

Extraer títulos:

```bash
grubby --info=ALL | grep '^title='
```

Extraer kernels:

```bash
grubby --info=ALL | grep '^kernel='
```

---

## Escenario 5: Examinar BLS

```bash
ls -l /boot/loader/entries/
```

Mostrar una entrada:

```bash
sudo sed -n '1,120p' \
/boot/loader/entries/*.conf
```

Identifica:

- Título.
- Versión.
- Kernel.
- `initramfs`.
- Parámetros.

---

## Escenario 6: Examinar la configuración

```bash
cat /etc/default/grub
```

```bash
ls -l /etc/grub.d/
```

```bash
sudo less /boot/grub2/grub.cfg
```

---

## Escenario 7: Crear un respaldo

```bash
sudo cp \
/etc/default/grub \
/etc/default/grub.bak
```

```bash
sudo cp \
/boot/grub2/grub.cfg \
/boot/grub2/grub.cfg.bak
```

---

## Escenario 8: Agregar un parámetro temporal

Reinicia la máquina virtual.

En el menú:

1. Selecciona el kernel.
2. Presiona `e`.
3. Localiza la línea `linux`.
4. Agrega:

```text
systemd.unit=multi-user.target
```

5. Inicia con `Ctrl+x`.

Después verifica:

```bash
systemctl get-default
```

```bash
systemctl list-units \
--type=target \
--state=active
```

El target predeterminado permanente no debe haber cambiado.

---

## Escenario 9: Mostrar mensajes detallados

En el menú de GRUB:

1. Presiona `e`.
2. Elimina temporalmente:

```text
rhgb quiet
```

3. Inicia con `Ctrl+x`.
4. Observa los mensajes.

Después:

```bash
cat /proc/cmdline
```

---

## Escenario 10: Agregar un parámetro persistente

Agrega un parámetro de laboratorio:

```bash
sudo grubby \
--update-kernel=ALL \
--args="audit=1"
```

Verifica:

```bash
grubby --info=ALL | grep args
```

No es necesario reiniciar todavía.

---

## Escenario 11: Eliminar el parámetro

```bash
sudo grubby \
--update-kernel=ALL \
--remove-args="audit=1"
```

Verifica:

```bash
grubby --info=ALL | grep args
```

---

## Escenario 12: Consultar el entorno

```bash
grub2-editenv list
```

Identifica si existen:

- `saved_entry`
- `menu_auto_hide`
- `boot_success`

---

## Escenario 13: Verificar configuración

```bash
sudo grub2-script-check \
/boot/grub2/grub.cfg
```

---

## Escenario 14: Consultar archivos de arranque

```bash
ls -lh /boot/vmlinuz-*
```

```bash
ls -lh /boot/initramfs-*
```

```bash
df -h /boot
```

---

## Escenario 15: Consultar Secure Boot

Si existe `mokutil`:

```bash
mokutil --sb-state
```

Si no está instalado:

```bash
sudo dnf install mokutil
```

---

## Escenario 16: Consultar UEFI

Si el sistema utiliza UEFI:

```bash
sudo efibootmgr
```

```bash
sudo efibootmgr -v
```

No modifiques las entradas.

---

# Script opcional de auditoría

```bash
#!/bin/bash

REPORTE="$HOME/reporte-grub2.txt"

{
    echo "=================================================="
    echo "REPORTE DE GRUB2"
    echo "=================================================="
    echo

    echo "Fecha:"
    date
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
    grubby --default-kernel
    echo

    echo "Índice predeterminado:"
    grubby --default-index
    echo

    echo "Parámetros activos:"
    cat /proc/cmdline
    echo

    echo "Entradas configuradas:"
    grubby --info=ALL
    echo

    echo "Archivos del kernel:"
    ls -lh /boot/vmlinuz-*
    echo

    echo "Archivos initramfs:"
    ls -lh /boot/initramfs-*
    echo

    echo "Entradas BLS:"
    ls -l /boot/loader/entries/ 2>/dev/null
    echo

    echo "Entorno GRUB:"
    grub2-editenv list
    echo

    echo "Espacio en /boot:"
    df -h /boot
    echo

    if [ -d /sys/firmware/efi ]; then
        echo "Partición EFI:"
        findmnt /boot/efi
        echo

        echo "Entradas UEFI:"
        efibootmgr -v 2>/dev/null
        echo
    fi

    echo "=================================================="
    echo "FIN DEL REPORTE"
    echo "=================================================="

} > "$REPORTE" 2>&1

echo "Reporte generado en: $REPORTE"
```

Guardar:

```text
~/auditar-grub2.sh
```

Asignar permisos:

```bash
chmod +x ~/auditar-grub2.sh
```

Ejecutar:

```bash
~/auditar-grub2.sh
```

---

# Preguntas de repaso

1. ¿Cuál es la función principal de GRUB2?
2. ¿En qué momento del arranque se ejecuta?
3. ¿Qué archivos principales carga?
4. ¿Cómo funciona en BIOS?
5. ¿Cómo funciona en UEFI?
6. ¿Cómo se identifica el tipo de firmware?
7. ¿Qué comando muestra el kernel activo?
8. ¿Qué comando muestra el predeterminado?
9. ¿Cómo se listan todas las entradas?
10. ¿Qué es una entrada BLS?
11. ¿Dónde suelen almacenarse las entradas BLS?
12. ¿Qué tecla permite editar una entrada?
13. ¿Cómo se inicia después de editar?
14. ¿Los cambios desde el menú son permanentes?
15. ¿Qué parámetro inicia en rescue target?
16. ¿Qué parámetro inicia en emergency target?
17. ¿Qué función cumple `rd.break`?
18. ¿Cómo se agrega un argumento persistente?
19. ¿Cómo se elimina un argumento?
20. ¿Por qué no debe editarse directamente `grub.cfg`?
21. ¿Qué función cumple `grub2-mkconfig`?
22. ¿Qué muestra `grub2-editenv list`?
23. ¿Qué función cumple Secure Boot?
24. ¿Por qué debe conservarse un kernel anterior?
25. ¿Qué riesgo tiene ejecutar `grub2-install` en el disco equivocado?

---

# Desafío final

Realiza las siguientes tareas:

1. Identifica BIOS o UEFI.
2. Consulta el kernel activo.
3. Consulta el kernel predeterminado.
4. Lista todas las entradas.
5. Identifica una entrada de rescate.
6. Examina los archivos BLS.
7. Consulta `/etc/default/grub`.
8. Verifica los parámetros activos.
9. Realiza un cambio temporal desde el menú.
10. Inicia en `multi-user.target` temporalmente.
11. Agrega un argumento con `grubby`.
12. Verifica el argumento almacenado.
13. Elimina el argumento.
14. Comprueba la sintaxis de `grub.cfg`.
15. Consulta las variables persistentes.
16. Verifica el espacio de `/boot`.
17. Consulta Secure Boot.
18. Genera un reporte de auditoría.

> **Objetivo general:** administrar GRUB2 de forma segura, comprender su relación con BIOS, UEFI, el kernel y `initramfs`, y utilizar herramientas como `grubby`, BLS, `grub2-editenv` y `grub2-mkconfig` para configurar y diagnosticar el proceso de arranque en sistemas Red Hat Enterprise Linux.