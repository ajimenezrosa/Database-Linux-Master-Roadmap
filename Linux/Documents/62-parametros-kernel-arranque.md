# 62. Parámetros del Kernel durante el Arranque

> **Módulo 9: Arranque y Recuperación**  
> **Página 62 del Manual RHCSA**

---

# Objetivos de aprendizaje

Al finalizar esta lección serás capaz de:

- Comprender qué son los parámetros del kernel.
- Consultar los parámetros utilizados en el arranque actual.
- Diferenciar parámetros temporales y persistentes.
- Editar temporalmente una entrada desde GRUB2.
- Agregar y eliminar argumentos persistentes mediante `grubby`.
- Iniciar temporalmente en diferentes targets de `systemd`.
- Utilizar `rd.break` para interrumpir el proceso de arranque.
- Comprender parámetros relacionados con almacenamiento, SELinux, red y consola.
- Diagnosticar problemas producidos por argumentos incorrectos.
- Restaurar una configuración funcional.
- Aplicar buenas prácticas de administración y recuperación RHCSA.

---

# Introducción

Los parámetros del kernel son argumentos que GRUB2 entrega al kernel de Linux durante el arranque.

Estos parámetros permiten controlar aspectos como:

- Ubicación del sistema de archivos raíz.
- Modo de montaje inicial.
- Target de `systemd`.
- Controladores.
- LVM.
- Discos cifrados.
- Consolas.
- SELinux.
- Registro de mensajes.
- Entorno de recuperación.
- Comportamiento de dispositivos.

Flujo conceptual:

```text
GRUB2
  │
  ├── Selecciona kernel
  ├── Carga initramfs
  └── Entrega parámetros
             │
             ▼
          Kernel
             │
             ▼
          initramfs
             │
             ▼
          systemd
```

Un parámetro incorrecto puede impedir que el sistema encuentre el disco raíz, cargue un controlador o alcance el target esperado.

---

# ¿Qué es la línea de comandos del kernel?

La línea de comandos del kernel es el conjunto de argumentos usados para iniciar Linux.

Ejemplo:

```text
BOOT_IMAGE=(hd0,gpt2)/vmlinuz-5.14.0-500.el9.x86_64 root=/dev/mapper/rhel-root ro crashkernel=1G-4G:192M rd.lvm.lv=rhel/root rd.lvm.lv=rhel/swap rhgb quiet
```

Cada elemento modifica o describe una parte del proceso de arranque.

---

# Consultar los parámetros del arranque actual

```bash
cat /proc/cmdline
```

También:

```bash
tr ' ' '\n' < /proc/cmdline
```

Esta segunda forma muestra un argumento por línea.

Ejemplo:

```text
BOOT_IMAGE=(hd0,gpt2)/vmlinuz-5.14.0-500.el9.x86_64
root=/dev/mapper/rhel-root
ro
crashkernel=1G-4G:192M
rd.lvm.lv=rhel/root
rd.lvm.lv=rhel/swap
rhgb
quiet
```

---

# `/proc/cmdline`

El archivo:

```text
/proc/cmdline
```

representa los argumentos que recibió el kernel durante el arranque actual.

No es un archivo que deba editarse.

```text
/proc/cmdline
      │
      ├── Es de solo consulta
      ├── Representa el arranque actual
      └── Se regenera en cada inicio
```

---

# Consultar los argumentos almacenados

Para consultar las entradas configuradas:

```bash
grubby --info=ALL
```

Mostrar únicamente las líneas de argumentos:

```bash
grubby --info=ALL | grep '^args='
```

Consultar el kernel activo:

```bash
grubby \
--info=/boot/vmlinuz-$(uname -r)
```

---

# Configuración almacenada frente a configuración activa

| Fuente | Información |
|---|---|
| `/proc/cmdline` | Parámetros usados en el arranque actual |
| `grubby --info=ALL` | Parámetros almacenados en las entradas |
| `/etc/default/grub` | Configuración general de GRUB |
| `/boot/loader/entries/` | Entradas BLS de los kernels |
| Menú de GRUB | Modificación temporal para un solo arranque |

---

# Parámetros temporales y persistentes

Los argumentos pueden aplicarse de dos formas:

```text
Temporal
```

o:

```text
Persistente
```

---

# Parámetro temporal

Se agrega manualmente desde el menú de GRUB.

Características:

- Se utiliza en un solo arranque.
- No modifica permanentemente los archivos.
- Desaparece después del reinicio.
- Es ideal para pruebas y recuperación.

Ejemplo:

```text
systemd.unit=rescue.target
```

---

# Parámetro persistente

Se guarda en las entradas del cargador.

Características:

- Se aplica en futuros arranques.
- Puede afectar uno o todos los kernels.
- Debe verificarse cuidadosamente.
- Normalmente se administra mediante `grubby`.

Ejemplo:

```bash
sudo grubby \
--update-kernel=ALL \
--args="audit=1"
```

---

# Comparación

| Característica | Temporal | Persistente |
|---|---|---|
| Duración | Un arranque | Futuros arranques |
| Método | Menú de GRUB | `grubby` |
| Riesgo | Menor | Mayor |
| Uso | Pruebas y recuperación | Configuración permanente |
| Reinicio adicional | No | Normalmente sí |
| Restauración | Reiniciar | Eliminar el argumento |

---

# Editar una entrada temporalmente

Procedimiento:

1. Reinicia el sistema.
2. Accede al menú de GRUB2.
3. Selecciona una entrada.
4. Presiona:

```text
e
```

5. Localiza la línea que comienza con:

```text
linux
```

En algunos sistemas puede aparecer:

```text
linuxefi
```

6. Agrega, modifica o elimina argumentos.
7. Inicia con:

```text
Ctrl+x
```

En algunos entornos también puede funcionar:

```text
F10
```

---

# Flujo de modificación temporal

```text
Reiniciar
   │
   ▼
Menú de GRUB
   │
   ▼
Presionar e
   │
   ▼
Editar línea linux
   │
   ▼
Ctrl+x
   │
   ▼
Arranque temporal
   │
   ▼
Próximo reinicio restaura configuración
```

---

# Línea `linux`

Ejemplo conceptual:

```text
linux /vmlinuz-5.14.0-500.el9.x86_64 root=/dev/mapper/rhel-root ro rd.lvm.lv=rhel/root rhgb quiet
```

Esta línea contiene:

- Ruta del kernel.
- Dispositivo raíz.
- Modo de montaje.
- Volúmenes LVM.
- Parámetros gráficos.
- Argumentos adicionales.

---

# Recomendación antes de editar

Antes de cambiar argumentos:

- Toma una fotografía o nota de la línea original.
- Modifica un solo elemento a la vez.
- No elimines `root=` sin conocer su función.
- No elimines argumentos de LVM o cifrado sin verificar.
- Mantén acceso a consola.
- Prefiere cambios temporales antes de hacerlos persistentes.

---

# Parámetro `root=`

Indica dónde se encuentra el sistema de archivos raíz.

Ejemplos:

```text
root=/dev/mapper/rhel-root
```

```text
root=/dev/sda3
```

```text
root=UUID=11111111-2222-3333-4444-555555555555
```

---

# Importancia de `root=`

Sin un valor correcto, el kernel o `initramfs` no podrán localizar `/`.

Errores posibles:

```text
Warning: /dev/mapper/rhel-root does not exist
```

```text
Kernel panic - not syncing: VFS: Unable to mount root fs
```

```text
Entering emergency mode
```

---

# Verificar el sistema raíz actual

```bash
findmnt /
```

También:

```bash
lsblk -f
```

```bash
blkid
```

---

# Parámetros `ro` y `rw`

## `ro`

```text
ro
```

Solicita montar inicialmente el sistema raíz en modo de solo lectura.

Es común durante el arranque temprano.

---

## `rw`

```text
rw
```

Solicita montar la raíz en lectura y escritura.

El sistema puede remontarla posteriormente según la configuración y el proceso de arranque.

---

# Verificar el estado de montaje

```bash
findmnt -o TARGET,SOURCE,FSTYPE,OPTIONS /
```

Ejemplo:

```text
TARGET SOURCE                FSTYPE OPTIONS
/      /dev/mapper/rhel-root xfs    rw,relatime,seclabel
```

---

# Parámetro `quiet`

```text
quiet
```

Reduce la cantidad de mensajes mostrados en pantalla durante el arranque.

Ventaja:

- Arranque visualmente limpio.

Desventaja:

- Oculta información útil para diagnóstico.

---

# Parámetro `rhgb`

```text
rhgb
```

Significa:

```text
Red Hat Graphical Boot
```

Activa una pantalla gráfica durante el inicio.

---

# Diagnóstico mostrando mensajes

Para observar mensajes detallados:

1. Edita la entrada.
2. Elimina temporalmente:

```text
rhgb quiet
```

3. Inicia con `Ctrl+x`.

Esto ayuda a detectar:

- Esperas.
- Errores de montaje.
- Fallos de controladores.
- Problemas de servicios.
- Errores de almacenamiento.

---

# Restaurar mensajes gráficos

Si se eliminaron persistentemente:

```bash
sudo grubby \
--update-kernel=ALL \
--args="rhgb quiet"
```

---

# Iniciar en un target temporal

Se utiliza:

```text
systemd.unit=TARGET
```

Ejemplos:

```text
systemd.unit=multi-user.target
```

```text
systemd.unit=graphical.target
```

```text
systemd.unit=rescue.target
```

```text
systemd.unit=emergency.target
```

---

# `systemd.unit=multi-user.target`

Inicia en modo multiusuario sin requerir interfaz gráfica.

Uso:

- Diagnosticar fallos gráficos.
- Trabajar en modo texto.
- Evitar iniciar el display manager.
- Realizar mantenimiento.

---

# `systemd.unit=graphical.target`

Solicita iniciar el entorno gráfico.

Solo funcionará si existen:

- Paquetes gráficos.
- Display manager.
- Controladores adecuados.
- Servicios necesarios.

---

# `systemd.unit=rescue.target`

Inicia un entorno mínimo de rescate.

Normalmente proporciona:

- Sistemas de archivos locales.
- Servicios básicos.
- Shell administrativa.
- Menos servicios que el modo multiusuario.

---

# `systemd.unit=emergency.target`

Inicia un entorno extremadamente mínimo.

Puede proporcionar:

- Shell administrativa.
- Sistema raíz en solo lectura.
- Muy pocos servicios.
- Sin red.
- Sin montajes adicionales.

---

# Target temporal frente a target predeterminado

Agregar:

```text
systemd.unit=multi-user.target
```

no modifica:

```bash
systemctl get-default
```

El target predeterminado permanente permanece igual.

---

# Verificar después del arranque

```bash
systemctl list-units \
--type=target \
--state=active
```

Consultar el predeterminado:

```bash
systemctl get-default
```

Consultar los argumentos usados:

```bash
cat /proc/cmdline
```

---

# Parámetro `rd.break`

```text
rd.break
```

Interrumpe el proceso de arranque dentro del entorno `initramfs`.

Se utiliza para:

- Recuperar la contraseña de `root`.
- Modificar archivos antes del arranque completo.
- Examinar el sistema raíz.
- Corregir configuraciones.
- Diagnosticar almacenamiento.
- Trabajar antes de iniciar `systemd` normalmente.

---

# Flujo con `rd.break`

```text
GRUB2
  │
  ▼
Kernel
  │
  ▼
initramfs
  │
  ▼
rd.break
  │
  ▼
Shell de dracut
  │
  ▼
/sysroot
```

---

# Entorno de `rd.break`

Al utilizar `rd.break`, el sistema raíz real suele estar disponible en:

```text
/sysroot
```

Inicialmente puede estar montado en solo lectura.

Consultar:

```bash
mount | grep sysroot
```

---

# Remontar `/sysroot`

```bash
mount -o remount,rw /sysroot
```

Verificar:

```bash
mount | grep sysroot
```

---

# Entrar al sistema instalado

```bash
chroot /sysroot
```

Después de `chroot`, la ruta:

```text
/sysroot
```

se convierte conceptualmente en:

```text
/
```

---

# Salir del entorno

```bash
exit
```

Después:

```bash
exit
```

El primer `exit` abandona `chroot`.

El segundo permite continuar o salir de la shell de `initramfs`.

---

# `rd.break` frente a `emergency.target`

| Característica | `rd.break` | `emergency.target` |
|---|---|---|
| Momento | Dentro de initramfs | Después de iniciar systemd |
| Raíz real | Normalmente en `/sysroot` | Normalmente montada como `/` |
| Uso | Recuperación temprana | Diagnóstico mínimo |
| Necesidad de `chroot` | Frecuentemente sí | Normalmente no |
| Servicios | No iniciados normalmente | Muy pocos |

---

# Parámetros de dracut

Los argumentos que comienzan con:

```text
rd.
```

suelen ser procesados durante el entorno inicial de `dracut`.

Ejemplos:

```text
rd.break
rd.lvm.lv=
rd.luks.uuid=
rd.md.uuid=
rd.neednet=1
rd.shell
rd.debug
```

---

# Parámetro `rd.lvm.lv=`

Indica un volumen lógico necesario durante el arranque.

Ejemplo:

```text
rd.lvm.lv=rhel/root
```

Otro:

```text
rd.lvm.lv=rhel/swap
```

---

# Importancia de `rd.lvm.lv=`

Ayuda a `initramfs` a identificar y activar volúmenes LVM necesarios.

Eliminarlo incorrectamente puede provocar:

- Volumen raíz no encontrado.
- Swap no activado.
- Entrada en shell de dracut.
- Fallo de arranque.

---

# Verificar volúmenes LVM

```bash
sudo lvs
```

```bash
sudo vgs
```

```bash
sudo pvs
```

Consultar el origen de `/`:

```bash
findmnt /
```

---

# Parámetro `rd.luks.uuid=`

Indica un dispositivo cifrado LUKS requerido.

Ejemplo conceptual:

```text
rd.luks.uuid=luks-11111111-2222-3333-4444-555555555555
```

---

# Verificar dispositivos cifrados

```bash
lsblk -f
```

```bash
sudo cryptsetup luksDump /dev/dispositivo
```

```bash
cat /etc/crypttab
```

No elimines parámetros `rd.luks.uuid=` si el sistema raíz depende de cifrado.

---

# Parámetro `rd.md.uuid=`

Indica un conjunto RAID por software requerido durante el arranque.

Ejemplo:

```text
rd.md.uuid=11111111:22222222:33333333:44444444
```

---

# Verificar RAID

```bash
cat /proc/mdstat
```

```bash
sudo mdadm --detail --scan
```

---

# Parámetro `resume=`

Puede indicar el dispositivo utilizado para reanudar desde hibernación.

Ejemplo:

```text
resume=/dev/mapper/rhel-swap
```

No es esencial en todos los servidores.

---

# Parámetro `crashkernel=`

Reserva memoria para el kernel utilizado por `kdump`.

Ejemplo:

```text
crashkernel=1G-4G:192M,4G-64G:256M,64G-:512M
```

Consultar:

```bash
cat /proc/cmdline
```

Verificar kdump:

```bash
systemctl status kdump
```

---

# No eliminar `crashkernel=` sin evaluación

Eliminarlo puede impedir el funcionamiento de `kdump`.

Antes de modificarlo:

- Verifica si el servidor utiliza análisis de fallos.
- Consulta las políticas de soporte.
- Revisa la memoria disponible.
- Documenta el valor original.

---

# Parámetro `audit=`

Controla aspectos del sistema de auditoría durante el arranque.

Ejemplo:

```text
audit=1
```

Este parámetro puede asegurar que la auditoría esté activa desde etapas tempranas.

---

# Agregar `audit=1`

```bash
sudo grubby \
--update-kernel=ALL \
--args="audit=1"
```

Verificar:

```bash
grubby --info=ALL | grep '^args='
```

Después del reinicio:

```bash
cat /proc/cmdline
```

---

# Parámetros relacionados con SELinux

Parámetros comunes:

```text
enforcing=0
```

```text
selinux=0
```

```text
autorelabel=1
```

---

# `enforcing=0`

Inicia SELinux en modo permisivo.

```text
enforcing=0
```

SELinux continúa cargado, pero las denegaciones no se aplican de forma bloqueante.

Consultar después:

```bash
getenforce
```

Resultado esperado:

```text
Permissive
```

---

# `selinux=0`

Desactiva SELinux durante el arranque.

```text
selinux=0
```

Esto es más invasivo que `enforcing=0`.

No debe utilizarse como solución normal.

---

# Diferencia entre `enforcing=0` y `selinux=0`

| Parámetro | Resultado |
|---|---|
| `enforcing=0` | SELinux cargado en modo permisivo |
| `selinux=0` | SELinux desactivado |
| Sin parámetro | Se utiliza la configuración normal |

---

# Riesgo de `selinux=0`

Con SELinux desactivado:

- Los archivos pueden crearse sin contextos adecuados.
- Los problemas reales pueden quedar ocultos.
- Puede ser necesario reetiquetar el sistema.
- Se reduce la seguridad.

---

# Reetiquetado SELinux

Para solicitar un reetiquetado durante el próximo arranque:

```bash
sudo touch /.autorelabel
```

También puede utilizarse temporalmente:

```text
autorelabel=1
```

El proceso puede tardar según:

- Cantidad de archivos.
- Velocidad del almacenamiento.
- Tamaño del sistema.

---

# Parámetro `nomodeset`

```text
nomodeset
```

Evita que el kernel active ciertos modos gráficos avanzados durante el arranque.

Puede utilizarse temporalmente para:

- Problemas de pantalla negra.
- Fallos de controladores gráficos.
- Diagnóstico de GPU.
- Acceso temporal a consola.

No debe asumirse como solución permanente sin investigar el controlador.

---

# Parámetros de consola

Ejemplos:

```text
console=tty0
```

```text
console=ttyS0,115200
```

---

# `console=tty0`

Envía mensajes a la consola virtual principal.

---

# `console=ttyS0,115200`

Configura una consola serial.

Uso frecuente:

- Servidores sin monitor.
- Máquinas virtuales.
- Consolas remotas.
- Entornos cloud.
- Hardware con administración fuera de banda.

---

# Varias consolas

Puede configurarse más de una:

```text
console=tty0 console=ttyS0,115200
```

La última consola suele convertirse en la consola principal para ciertos mensajes y entrada.

---

# Agregar consola serial persistentemente

```bash
sudo grubby \
--update-kernel=ALL \
--args="console=tty0 console=ttyS0,115200"
```

Antes de aplicarlo:

- Confirma el dispositivo serial.
- Verifica la velocidad.
- Mantén una consola alternativa.
- Documenta los valores originales.

---

# Parámetro `loglevel=`

Controla el nivel de mensajes del kernel mostrados en consola.

Ejemplo:

```text
loglevel=7
```

Un valor alto muestra más información.

Es útil temporalmente para diagnóstico.

---

# Parámetro `ignore_loglevel`

```text
ignore_loglevel
```

Solicita mostrar mensajes del kernel independientemente del nivel configurado.

Puede combinarse temporalmente con la eliminación de:

```text
quiet
```

---

# Parámetro `debug`

```text
debug
```

Activa mensajes adicionales de depuración del kernel.

Puede producir gran cantidad de salida.

Se recomienda utilizarlo solo temporalmente.

---

# Parámetro `rd.debug`

```text
rd.debug
```

Activa más información del entorno `dracut`.

Es útil cuando el problema ocurre antes de montar el sistema raíz.

---

# Parámetro `rd.shell`

```text
rd.shell
```

Permite abrir una shell de recuperación de dracut si el arranque inicial falla.

Deshabilitar esta capacidad puede formar parte de endurecimiento, pero dificulta la recuperación local.

---

# Parámetro `rd.neednet=1`

```text
rd.neednet=1
```

Indica que la red es necesaria durante el entorno `initramfs`.

Puede utilizarse en sistemas cuyo almacenamiento raíz depende de:

- iSCSI.
- NFS.
- Red de almacenamiento.
- Recursos remotos.

---

# Parámetro `ip=`

Puede configurar red durante el arranque temprano.

Ejemplo conceptual:

```text
ip=192.168.1.50::192.168.1.1:255.255.255.0:servidor:eth0:none
```

La sintaxis depende del escenario.

Debe documentarse cuidadosamente antes de aplicarse.

---

# Parámetro `ipv6.disable=1`

```text
ipv6.disable=1
```

Desactiva IPv6 a nivel del kernel.

No debe utilizarse simplemente porque IPv6 no esté configurado.

Puede afectar:

- Aplicaciones.
- Servicios.
- Resolución de nombres.
- Dependencias de red.
- Soporte del sistema.

---

# Parámetro `net.ifnames=0`

```text
net.ifnames=0
```

Deshabilita los nombres de interfaces predecibles.

Puede provocar nombres como:

```text
eth0
eth1
```

No debe cambiarse en sistemas existentes sin revisar:

- Perfiles de NetworkManager.
- Reglas de firewall.
- Scripts.
- Servicios.
- Automatizaciones.

---

# Parámetro `biosdevname=0`

```text
biosdevname=0
```

Puede afectar el esquema de nombres de interfaces en ciertos sistemas.

El uso combinado con:

```text
net.ifnames=0
```

puede cambiar los nombres de red.

---

# Parámetro `acpi=off`

```text
acpi=off
```

Desactiva ACPI.

Es un parámetro de diagnóstico extremo.

Puede afectar:

- Administración de energía.
- Detección de hardware.
- Apagado.
- CPU.
- Suspensión.
- Dispositivos.

No debe aplicarse permanentemente sin una razón comprobada.

---

# Parámetro `noapic`

```text
noapic
```

Desactiva funciones relacionadas con APIC.

Puede utilizarse en diagnósticos específicos de hardware, pero puede reducir funcionalidad y rendimiento.

---

# Parámetro `intel_iommu=on`

```text
intel_iommu=on
```

Activa IOMMU en plataformas Intel compatibles.

Puede utilizarse para:

- Virtualización.
- Passthrough de dispositivos.
- Aislamiento DMA.

Para AMD puede existir:

```text
amd_iommu=on
```

---

# Parámetros de mitigación y seguridad

Existen argumentos relacionados con vulnerabilidades y mitigaciones, por ejemplo:

```text
mitigations=auto
```

```text
mitigations=off
```

Desactivar mitigaciones puede aumentar el rendimiento en ciertos casos, pero reduce la seguridad.

No debe hacerse sin:

- Evaluación de riesgos.
- Aprobación.
- Pruebas.
- Documentación.
- Conocimiento del entorno.

---

# Herramienta `grubby`

`grubby` permite administrar argumentos persistentes de forma segura en RHEL.

Funciones principales:

- Consultar entradas.
- Agregar argumentos.
- Eliminar argumentos.
- Actualizar un kernel.
- Actualizar todos los kernels.
- Cambiar el kernel predeterminado.

---

# Consultar ayuda

```bash
grubby --help
```

Manual:

```bash
man grubby
```

---

# Agregar un argumento a todos los kernels

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

# Agregar a un kernel específico

```bash
sudo grubby \
--update-kernel=/boot/vmlinuz-$(uname -r) \
--args="audit=1"
```

---

# Eliminar un argumento de todos los kernels

```bash
sudo grubby \
--update-kernel=ALL \
--remove-args="audit=1"
```

---

# Eliminar varios argumentos

```bash
sudo grubby \
--update-kernel=ALL \
--remove-args="rhgb quiet"
```

---

# Agregar varios argumentos

```bash
sudo grubby \
--update-kernel=ALL \
--args="console=tty0 console=ttyS0,115200"
```

---

# Verificar después del cambio

```bash
grubby --info=ALL
```

Mostrar solo argumentos:

```bash
grubby --info=ALL | grep '^args='
```

---

# Los cambios requieren reinicio

Modificar los argumentos almacenados no altera el kernel activo.

```text
grubby
   │
   ▼
Actualiza entrada
   │
   ▼
Reinicio
   │
   ▼
Nuevo argumento en /proc/cmdline
```

---

# Respaldar los argumentos actuales

```bash
grubby --info=ALL \
> ~/grubby-antes-del-cambio.txt
```

Guardar también:

```bash
cat /proc/cmdline \
> ~/cmdline-arranque-actual.txt
```

---

# Comparar antes y después

```bash
diff \
~/grubby-antes-del-cambio.txt \
~/grubby-despues-del-cambio.txt
```

---

# Entradas BLS

En sistemas modernos, las entradas pueden encontrarse en:

```text
/boot/loader/entries/
```

Consultar:

```bash
ls -l /boot/loader/entries/
```

---

# Examinar argumentos BLS

```bash
grep '^options ' \
/boot/loader/entries/*.conf
```

Ejemplo:

```text
options root=/dev/mapper/rhel-root ro crashkernel=auto rd.lvm.lv=rhel/root rhgb quiet
```

---

# No editar entradas BLS sin necesidad

Aunque son archivos de texto, se recomienda utilizar herramientas como:

```text
grubby
```

para evitar inconsistencias entre kernels y configuración.

---

# `/etc/default/grub`

Puede contener argumentos generales en:

```ini
GRUB_CMDLINE_LINUX="..."
```

Consultar:

```bash
grep '^GRUB_CMDLINE_LINUX' \
/etc/default/grub
```

---

# `grubby` frente a `/etc/default/grub`

| Método | Uso recomendado |
|---|---|
| `grubby` | Modificar entradas existentes |
| `/etc/default/grub` | Configuración general usada al generar GRUB |
| Edición temporal | Prueba o recuperación |
| BLS manual | Casos especiales y controlados |

---

# Aplicar argumentos a nuevos kernels

Al instalar kernels nuevos, los argumentos se heredan según la configuración del sistema.

Por eso conviene verificar después de una actualización:

```bash
grubby --info=ALL
```

---

# Eliminar un parámetro problemático desde GRUB

Si un argumento persistente impide iniciar:

1. Accede al menú.
2. Selecciona la entrada.
3. Presiona `e`.
4. Elimina el argumento incorrecto.
5. Inicia con `Ctrl+x`.
6. Una vez dentro, elimínalo persistentemente con `grubby`.

Ejemplo:

```bash
sudo grubby \
--update-kernel=ALL \
--remove-args="parametro_problematico"
```

---

# Diagnóstico: el sistema no encuentra la raíz

Revisa temporalmente la línea de GRUB.

Parámetros importantes:

```text
root=
rd.lvm.lv=
rd.luks.uuid=
rd.md.uuid=
```

Desde un sistema funcional:

```bash
findmnt /
```

```bash
lsblk -f
```

```bash
blkid
```

```bash
grubby --info=ALL
```

---

# Diagnóstico: shell de dracut

Si aparece:

```text
dracut:/#
```

puede indicar:

- Sistema raíz no encontrado.
- LVM no activado.
- Dispositivo cifrado no disponible.
- RAID no detectado.
- `initramfs` incompleto.
- Parámetro incorrecto.
- Controlador faltante.

---

# Comandos útiles en dracut

```bash
cat /proc/cmdline
```

```bash
ls /dev
```

```bash
lsblk
```

```bash
lvm pvscan
```

```bash
lvm vgscan
```

```bash
lvm vgchange -ay
```

```bash
blkid
```

```bash
journalctl
```

La disponibilidad de comandos depende del contenido de `initramfs`.

---

# Diagnóstico: pantalla negra

Prueba temporalmente:

1. Eliminar:

```text
rhgb quiet
```

2. Agregar:

```text
nomodeset
```

3. Iniciar con `Ctrl+x`.

Si el sistema inicia, revisa:

- Controlador de vídeo.
- Kernel.
- Firmware.
- Logs.
- Display manager.

---

# Diagnóstico: target incorrecto

Consulta:

```bash
cat /proc/cmdline
```

Busca:

```text
systemd.unit=
```

Después:

```bash
systemctl get-default
```

Un target temporal puede explicar por qué el sistema inició en modo diferente.

---

# Diagnóstico: SELinux impide iniciar un servicio

No desactives SELinux inmediatamente.

Primero revisa:

```bash
getenforce
```

```bash
ausearch -m AVC -ts boot
```

```bash
journalctl -b | grep -i selinux
```

Para una prueba temporal puede utilizarse:

```text
enforcing=0
```

Después debe investigarse la causa real.

---

# Diagnóstico: parámetro desconocido

El kernel puede ignorar ciertos argumentos desconocidos o entregarlos a otros componentes.

Revisa:

```bash
journalctl -k -b
```

```bash
dmesg -T
```

```bash
cat /proc/cmdline
```

Busca mensajes como:

```text
Unknown kernel command line parameters
```

---

# Diagnóstico: argumento duplicado

Una línea puede contener argumentos repetidos:

```text
console=tty0 console=ttyS0,115200
```

Esto puede ser intencional.

Pero otros duplicados pueden provocar confusión:

```text
systemd.unit=rescue.target systemd.unit=graphical.target
```

En estos casos, el comportamiento puede depender del componente que procese el argumento.

Evita configuraciones contradictorias.

---

# Diagnóstico: argumento no aparece después del reinicio

Verifica:

```bash
grubby --info=ALL
```

```bash
grubby --default-kernel
```

```bash
cat /proc/cmdline
```

Posibles causas:

- Se modificó otro kernel.
- Se inició una entrada diferente.
- El cambio fue temporal.
- El argumento se agregó incorrectamente.
- La entrada predeterminada no es la esperada.

---

# Diagnóstico: argumento sigue apareciendo

Consulta todas las fuentes:

```bash
grubby --info=ALL
```

```bash
grep '^options ' \
/boot/loader/entries/*.conf
```

```bash
grep '^GRUB_CMDLINE_LINUX' \
/etc/default/grub
```

Puede haberse agregado en más de una ubicación.

---

# Flujo recomendado para modificar parámetros

```text
Identificar necesidad
        │
        ▼
Consultar /proc/cmdline
        │
        ▼
Respaldar configuración
        │
        ▼
Probar temporalmente en GRUB
        │
        ▼
Validar el sistema
        │
        ▼
Aplicar con grubby
        │
        ▼
Verificar entradas
        │
        ▼
Reiniciar con consola disponible
        │
        ▼
Confirmar /proc/cmdline
```

---

# Ejemplo: iniciar temporalmente sin GUI

1. Reinicia.
2. Presiona `e`.
3. Agrega:

```text
systemd.unit=multi-user.target
```

4. Inicia con `Ctrl+x`.
5. Verifica:

```bash
systemctl list-units \
--type=target \
--state=active
```

6. Confirma que el valor permanente no cambió:

```bash
systemctl get-default
```

---

# Ejemplo: mostrar mensajes detallados

1. Edita la entrada.
2. Elimina:

```text
rhgb quiet
```

3. Inicia.
4. Consulta:

```bash
cat /proc/cmdline
```

5. Revisa:

```bash
journalctl -b -p warning
```

---

# Ejemplo: agregar auditoría persistente

Respaldar:

```bash
grubby --info=ALL \
> ~/grubby-respaldo.txt
```

Agregar:

```bash
sudo grubby \
--update-kernel=ALL \
--args="audit=1"
```

Verificar:

```bash
grubby --info=ALL | grep '^args='
```

Reiniciar:

```bash
sudo systemctl reboot
```

Confirmar:

```bash
cat /proc/cmdline
```

---

# Ejemplo: eliminar auditoría

```bash
sudo grubby \
--update-kernel=ALL \
--remove-args="audit=1"
```

Después reinicia y verifica:

```bash
cat /proc/cmdline
```

---

# Ejemplo: recuperar un argumento incorrecto

Supongamos que se agregó:

```text
systemd.unit=emergency.target
```

persistentemente.

Inicia temporalmente eliminándolo desde GRUB.

Después ejecuta:

```bash
sudo grubby \
--update-kernel=ALL \
--remove-args="systemd.unit=emergency.target"
```

Verifica:

```bash
grubby --info=ALL | grep '^args='
```

---

# Buenas prácticas RHCSA

✔ Consultar siempre `/proc/cmdline` antes de realizar cambios.

✔ Respaldar la salida de `grubby --info=ALL`.

✔ Probar argumentos temporalmente desde GRUB.

✔ Aplicar persistentemente solo después de validar.

✔ Cambiar un parámetro a la vez.

✔ Mantener un kernel anterior funcional.

✔ Mantener acceso a consola durante las pruebas.

✔ No eliminar `root=` sin conocer la ubicación real de `/`.

✔ No eliminar parámetros LVM, RAID o LUKS sin verificar.

✔ Evitar `selinux=0` como solución permanente.

✔ Utilizar `enforcing=0` solo como prueba controlada.

✔ No desactivar mitigaciones de seguridad sin aprobación.

✔ Verificar el argumento después del reinicio.

✔ Documentar cada argumento agregado o eliminado.

✔ Utilizar `grubby` en lugar de editar manualmente todas las entradas.

---

# Errores comunes

## Editar `/proc/cmdline`

Este archivo no contiene configuración persistente y no debe editarse.

---

## Confundir un cambio temporal con uno persistente

Editar desde GRUB afecta únicamente un arranque.

---

## Agregar un parámetro a un solo kernel sin saberlo

El sistema puede iniciar con otra entrada y no aplicar el argumento.

---

## Eliminar `root=`

Puede impedir que el sistema localice el sistema raíz.

---

## Eliminar `rd.lvm.lv=`

Puede impedir activar volúmenes LVM necesarios.

---

## Utilizar `selinux=0` permanentemente

Reduce la seguridad y puede generar problemas de etiquetado.

---

## Reiniciar sin verificar las entradas

Antes de reiniciar:

```bash
grubby --info=ALL
```

---

## Aplicar varios cambios simultáneamente

Dificulta identificar cuál argumento causó el problema.

---

## Usar parámetros de hardware copiados de otro sistema

Opciones como:

```text
acpi=off
noapic
nomodeset
```

deben probarse únicamente cuando sean pertinentes.

---

## Ejecutar cambios críticos solo mediante SSH

Una configuración incorrecta puede impedir la red o el arranque.

---

# Comandos importantes RHCSA

| Comando | Descripción |
|---|---|
| `cat /proc/cmdline` | Mostrar parámetros activos |
| `tr ' ' '\n' < /proc/cmdline` | Mostrar un argumento por línea |
| `grubby --info=ALL` | Consultar argumentos almacenados |
| `grubby --default-kernel` | Mostrar kernel predeterminado |
| `grubby --update-kernel=ALL --args=` | Agregar argumentos |
| `grubby --update-kernel=ALL --remove-args=` | Eliminar argumentos |
| `grep '^options ' /boot/loader/entries/*.conf` | Consultar opciones BLS |
| `findmnt /` | Consultar sistema raíz |
| `lsblk -f` | Consultar discos y UUID |
| `blkid` | Mostrar identificadores |
| `systemctl get-default` | Mostrar target predeterminado |
| `systemctl list-units --type=target` | Mostrar targets activos |
| `getenforce` | Consultar estado de SELinux |
| `journalctl -k -b` | Mostrar mensajes del kernel |
| `journalctl -b -p err` | Mostrar errores del arranque |
| `mount -o remount,rw /sysroot` | Remontar raíz con `rd.break` |
| `chroot /sysroot` | Entrar al sistema instalado |
| `touch /.autorelabel` | Solicitar reetiquetado SELinux |

---

# Parámetros importantes

| Parámetro | Función |
|---|---|
| `root=` | Define el sistema raíz |
| `ro` | Montaje inicial de solo lectura |
| `rw` | Solicita lectura y escritura |
| `quiet` | Reduce mensajes |
| `rhgb` | Arranque gráfico |
| `systemd.unit=` | Selecciona target temporal |
| `rd.break` | Interrumpe en initramfs |
| `rd.lvm.lv=` | Define volumen LVM requerido |
| `rd.luks.uuid=` | Define volumen cifrado |
| `rd.md.uuid=` | Define RAID requerido |
| `rd.debug` | Depuración de dracut |
| `rd.neednet=1` | Requiere red en initramfs |
| `enforcing=0` | SELinux permisivo |
| `selinux=0` | SELinux desactivado |
| `autorelabel=1` | Solicita reetiquetado |
| `nomodeset` | Desactiva mode setting gráfico |
| `console=` | Configura consola |
| `loglevel=` | Ajusta nivel de mensajes |
| `audit=1` | Activa auditoría temprana |
| `crashkernel=` | Reserva memoria para kdump |

---

# Resumen rápido

```text
Parámetros del kernel
        │
        ├── Consulta
        │     ├── /proc/cmdline
        │     └── grubby --info=ALL
        │
        ├── Temporales
        │     ├── Menú GRUB
        │     ├── Presionar e
        │     └── Ctrl+x
        │
        ├── Persistentes
        │     └── grubby
        │
        ├── Recuperación
        │     ├── rd.break
        │     ├── rescue.target
        │     └── emergency.target
        │
        ├── Almacenamiento
        │     ├── root=
        │     ├── rd.lvm.lv=
        │     ├── rd.luks.uuid=
        │     └── rd.md.uuid=
        │
        └── Diagnóstico
              ├── quiet
              ├── rhgb
              ├── rd.debug
              └── loglevel=
```

---

# Resumen

En esta lección aprendiste a:

- Comprender la línea de comandos del kernel.
- Consultar los argumentos activos.
- Diferenciar cambios temporales y persistentes.
- Modificar una entrada desde GRUB2.
- Iniciar con targets temporales.
- Utilizar `rd.break`.
- Comprender parámetros de LVM, LUKS y RAID.
- Administrar argumentos mediante `grubby`.
- Diagnosticar problemas gráficos y de almacenamiento.
- Trabajar con parámetros relacionados con SELinux.
- Recuperar el sistema cuando un argumento impide iniciar.
- Aplicar cambios de manera segura y controlada.

---

# Laboratorio práctico RHCSA

> Realiza las tareas en una máquina virtual y conserva acceso a consola. No practiques recuperación de arranque únicamente mediante SSH.

## Escenario 1: Consultar parámetros activos

```bash
cat /proc/cmdline
```

Después:

```bash
tr ' ' '\n' < /proc/cmdline
```

Identifica:

- `root=`
- `ro` o `rw`
- `rd.lvm.lv=`
- `rhgb`
- `quiet`
- `crashkernel=`

---

## Escenario 2: Consultar parámetros almacenados

```bash
grubby --info=ALL
```

Mostrar solo argumentos:

```bash
grubby --info=ALL | grep '^args='
```

---

## Escenario 3: Crear un respaldo

```bash
grubby --info=ALL \
> ~/parametros-kernel-respaldo.txt
```

Guardar la línea actual:

```bash
cat /proc/cmdline \
> ~/proc-cmdline-respaldo.txt
```

---

## Escenario 4: Iniciar temporalmente en modo texto

Desde GRUB:

1. Selecciona una entrada.
2. Presiona `e`.
3. Agrega:

```text
systemd.unit=multi-user.target
```

4. Inicia con `Ctrl+x`.

Después:

```bash
systemctl list-units \
--type=target \
--state=active
```

---

## Escenario 5: Verificar que el cambio no fue permanente

```bash
systemctl get-default
```

```bash
cat /proc/cmdline
```

---

## Escenario 6: Mostrar mensajes detallados

Desde GRUB:

1. Presiona `e`.
2. Elimina:

```text
rhgb quiet
```

3. Inicia con `Ctrl+x`.
4. Observa los mensajes.

Después:

```bash
journalctl -b -p warning
```

---

## Escenario 7: Agregar un argumento persistente

```bash
sudo grubby \
--update-kernel=ALL \
--args="audit=1"
```

Verifica:

```bash
grubby --info=ALL | grep '^args='
```

---

## Escenario 8: Eliminar el argumento

```bash
sudo grubby \
--update-kernel=ALL \
--remove-args="audit=1"
```

Verifica nuevamente:

```bash
grubby --info=ALL | grep '^args='
```

---

## Escenario 9: Examinar entradas BLS

```bash
ls -l /boot/loader/entries/
```

```bash
grep '^options ' \
/boot/loader/entries/*.conf
```

---

## Escenario 10: Identificar el sistema raíz

```bash
findmnt /
```

```bash
lsblk -f
```

```bash
blkid
```

Compara con el valor de:

```text
root=
```

---

## Escenario 11: Identificar parámetros LVM

```bash
cat /proc/cmdline \
| grep -o 'rd.lvm.lv=[^ ]*'
```

Después:

```bash
sudo lvs
```

Compara los nombres.

---

## Escenario 12: Iniciar temporalmente en rescue

Desde GRUB agrega:

```text
systemd.unit=rescue.target
```

Después inicia con `Ctrl+x`.

Verifica:

```bash
systemctl is-active rescue.target
```

---

## Escenario 13: Examinar `rd.break`

En una máquina virtual:

1. Reinicia.
2. Edita la entrada.
3. Agrega:

```text
rd.break
```

4. Inicia con `Ctrl+x`.
5. Consulta:

```bash
cat /proc/cmdline
```

6. Verifica `/sysroot`:

```bash
mount | grep sysroot
```

7. No modifiques contraseñas en este ejercicio.
8. Sal con:

```bash
exit
```

---

## Escenario 14: Probar modo permisivo temporal

Desde GRUB agrega:

```text
enforcing=0
```

Después de iniciar:

```bash
getenforce
```

Reinicia normalmente y verifica:

```bash
getenforce
```

Debe volver al modo configurado permanentemente.

---

## Escenario 15: Auditar argumentos de todos los kernels

```bash
for kernel in /boot/vmlinuz-*; do
    echo "========================================"
    echo "$kernel"
    grubby --info="$kernel" 2>/dev/null \
    | grep -E '^(kernel|args|initrd|title)='
done
```

---

# Script opcional de auditoría

```bash
#!/bin/bash

REPORTE="$HOME/reporte-parametros-kernel.txt"

{
    echo "=================================================="
    echo "REPORTE DE PARÁMETROS DEL KERNEL"
    echo "=================================================="
    echo

    echo "Fecha:"
    date
    echo

    echo "Hostname:"
    hostname
    echo

    echo "Kernel activo:"
    uname -r
    echo

    echo "Kernel predeterminado:"
    grubby --default-kernel
    echo

    echo "Parámetros activos:"
    cat /proc/cmdline
    echo

    echo "Parámetros activos, uno por línea:"
    tr ' ' '\n' < /proc/cmdline
    echo

    echo "Entradas configuradas:"
    grubby --info=ALL
    echo

    echo "Target predeterminado:"
    systemctl get-default
    echo

    echo "Targets activos:"
    systemctl list-units \
    --type=target \
    --state=active \
    --no-pager
    echo

    echo "Sistema raíz:"
    findmnt /
    echo

    echo "Discos y sistemas de archivos:"
    lsblk -f
    echo

    echo "Estado SELinux:"
    getenforce
    echo

    echo "Entradas BLS:"
    grep '^options ' \
    /boot/loader/entries/*.conf 2>/dev/null
    echo

    echo "Errores del kernel:"
    journalctl -k -b -p err --no-pager
    echo

    echo "=================================================="
    echo "FIN DEL REPORTE"
    echo "=================================================="

} > "$REPORTE" 2>&1

echo "Reporte generado en: $REPORTE"
```

Guardar:

```text
~/auditar-parametros-kernel.sh
```

Asignar permisos:

```bash
chmod +x \
~/auditar-parametros-kernel.sh
```

Ejecutar:

```bash
~/auditar-parametros-kernel.sh
```

---

# Preguntas de repaso

1. ¿Qué son los parámetros del kernel?
2. ¿Qué archivo muestra los argumentos activos?
3. ¿Se puede editar `/proc/cmdline`?
4. ¿Qué diferencia existe entre un argumento temporal y uno persistente?
5. ¿Qué tecla permite editar una entrada de GRUB?
6. ¿Cómo se inicia después de modificarla?
7. ¿Qué función cumple `root=`?
8. ¿Qué diferencia existe entre `ro` y `rw`?
9. ¿Qué función cumple `quiet`?
10. ¿Qué función cumple `rhgb`?
11. ¿Cómo se inicia temporalmente en modo texto?
12. ¿Cómo se inicia en rescue target?
13. ¿Cómo se inicia en emergency target?
14. ¿Qué función cumple `rd.break`?
15. ¿Dónde se encuentra normalmente la raíz real durante `rd.break`?
16. ¿Para qué se utiliza `chroot /sysroot`?
17. ¿Qué función cumple `rd.lvm.lv=`?
18. ¿Qué función cumple `rd.luks.uuid=`?
19. ¿Qué diferencia existe entre `enforcing=0` y `selinux=0`?
20. ¿Por qué debe evitarse `selinux=0`?
21. ¿Qué función cumple `nomodeset`?
22. ¿Cómo se agrega un argumento a todos los kernels?
23. ¿Cómo se elimina?
24. ¿Por qué se requiere reiniciar después de usar `grubby`?
25. ¿Cómo se recupera el sistema si un argumento persistente impide iniciar?

---

# Desafío final

Realiza las siguientes tareas:

1. Consulta la línea de comandos actual.
2. Muestra un argumento por línea.
3. Identifica el dispositivo raíz.
4. Identifica argumentos LVM.
5. Consulta todas las entradas con `grubby`.
6. Guarda un respaldo de la configuración.
7. Inicia temporalmente en `multi-user.target`.
8. Confirma que el target predeterminado no cambió.
9. Elimina temporalmente `rhgb quiet`.
10. Analiza los mensajes mostrados.
11. Agrega `audit=1` persistentemente.
12. Verifica la configuración almacenada.
13. Elimina `audit=1`.
14. Examina las entradas BLS.
15. Inicia temporalmente en rescue target.
16. Accede al entorno `rd.break`.
17. Identifica `/sysroot`.
18. Sal sin modificar el sistema.
19. Audita los argumentos de todos los kernels.
20. Genera un reporte final.

> **Objetivo general:** administrar de forma segura los parámetros utilizados durante el arranque de Linux, comprender su relación con GRUB2, el kernel, `initramfs`, almacenamiento, SELinux y `systemd`, y utilizar modificaciones temporales o persistentes para configurar, diagnosticar y recuperar sistemas Red Hat Enterprise Linux.