# 63. Recuperación de la Contraseña de root

> **Módulo 9: Arranque y Recuperación**  
> **Página 63 del Manual RHCSA**

---

# Objetivos de aprendizaje

Al finalizar esta lección serás capaz de:

- Comprender cuándo es necesario recuperar la contraseña de `root`.
- Identificar los requisitos para realizar una recuperación.
- Interrumpir temporalmente el proceso de arranque desde GRUB2.
- Utilizar el parámetro `rd.break`.
- Comprender la función del directorio `/sysroot`.
- Remontar el sistema raíz en modo lectura y escritura.
- Acceder al sistema instalado mediante `chroot`.
- Cambiar la contraseña de `root`.
- Solicitar un reetiquetado de SELinux.
- Salir correctamente del entorno de recuperación.
- Verificar el acceso administrativo después del reinicio.
- Diagnosticar problemas frecuentes durante la recuperación.
- Aplicar medidas de seguridad para proteger el cargador de arranque.

---

# Introducción

La contraseña de la cuenta `root` puede necesitar recuperarse cuando:

- Fue olvidada.
- Expiró y no puede renovarse normalmente.
- La cuenta quedó bloqueada.
- No existe otro usuario con privilegios administrativos.
- La autenticación local dejó de funcionar.
- El sistema fue heredado sin credenciales administrativas.
- La contraseña fue modificada por error.
- Un problema en PAM impide iniciar sesión.

En sistemas Red Hat Enterprise Linux, Fedora y distribuciones relacionadas, una técnica habitual consiste en interrumpir el arranque mediante:

```text
rd.break
```

Este parámetro abre una shell dentro del entorno inicial de `initramfs`, antes de que el sistema instalado complete su arranque.

Desde ese entorno se puede:

1. Localizar el sistema raíz real.
2. Remontarlo con permisos de escritura.
3. Entrar mediante `chroot`.
4. Cambiar la contraseña.
5. Preparar el reetiquetado de SELinux.
6. Reiniciar normalmente.

---

# Advertencia de seguridad

La recuperación de la contraseña de `root` requiere acceso directo al proceso de arranque.

Normalmente se necesita:

- Acceso físico al equipo.
- Consola de una máquina virtual.
- Consola remota de administración.
- Acceso al menú de GRUB2.
- Permiso administrativo sobre el sistema.

Esta técnica debe utilizarse únicamente en sistemas que administras o para los cuales tienes autorización.

---

# Flujo general de recuperación

```text
Reiniciar el sistema
        │
        ▼
Mostrar el menú de GRUB2
        │
        ▼
Seleccionar una entrada
        │
        ▼
Presionar e
        │
        ▼
Agregar rd.break
        │
        ▼
Iniciar con Ctrl+x
        │
        ▼
Shell de initramfs
        │
        ▼
Remontar /sysroot en rw
        │
        ▼
chroot /sysroot
        │
        ▼
passwd root
        │
        ▼
touch /.autorelabel
        │
        ▼
Salir y reiniciar
        │
        ▼
Verificar acceso
```

---

# Comprender el entorno de recuperación

Cuando se utiliza:

```text
rd.break
```

el proceso de arranque se detiene dentro de `initramfs`.

En ese momento existen dos sistemas de archivos conceptuales:

```text
Entorno temporal de initramfs
```

y:

```text
Sistema instalado
```

El sistema instalado normalmente está montado en:

```text
/sysroot
```

---

# Estructura conceptual

```text
/
├── dev
├── proc
├── run
├── sys
├── sysroot
│   ├── etc
│   ├── home
│   ├── root
│   ├── usr
│   └── var
└── tmp
```

En este entorno:

```text
/
```

representa la raíz temporal de `initramfs`.

Mientras que:

```text
/sysroot
```

representa la raíz del sistema Linux instalado.

---

# ¿Qué es `rd.break`?

`rd.break` es un parámetro procesado por `dracut` durante las primeras etapas del arranque.

Su función es detener temporalmente la secuencia y proporcionar una shell administrativa.

```text
rd
```

hace referencia al entorno inicial administrado por `dracut`.

```text
break
```

indica que el arranque debe interrumpirse.

---

# Posición de `rd.break` en el arranque

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
dracut
  │
  ▼
rd.break
  │
  ▼
Shell de recuperación
  │
  ▼
Sistema raíz en /sysroot
```

---

# Requisitos previos

Antes de comenzar, debes disponer de:

- Acceso a la consola.
- Capacidad para reiniciar el sistema.
- Acceso al menú de GRUB2.
- Conocimiento básico de GRUB.
- Tiempo suficiente para completar un reetiquetado SELinux.
- Una ventana de mantenimiento si es un servidor productivo.

También debes considerar:

- Si el disco está cifrado.
- Si GRUB está protegido por contraseña.
- Si Secure Boot está habilitado.
- Si el sistema utiliza una consola serial.
- Si existe acceso mediante una consola virtual remota.

---

# Consolas remotas posibles

En servidores físicos puede utilizarse una consola de administración como:

```text
iDRAC
iLO
IPMI
KVM over IP
```

En entornos virtualizados puede utilizarse:

```text
VMware Console
Hyper-V Console
Proxmox Console
VirtualBox Console
Cockpit Machines
```

Una sesión SSH no es suficiente si el sistema todavía no ha iniciado su red.

---

# Procedimiento completo

## Paso 1: Reiniciar el sistema

Desde un sistema operativo funcional:

```bash
sudo systemctl reboot
```

También puede utilizarse:

```bash
sudo reboot
```

Si el sistema no responde, utiliza la consola administrativa de la plataforma.

---

# Paso 2: Mostrar el menú de GRUB2

Durante el inicio, intenta mostrar el menú mediante:

```text
Esc
```

En algunos sistemas puede funcionar mantener presionada:

```text
Shift
```

El comportamiento depende de:

- BIOS o UEFI.
- Configuración del menú.
- Tiempo de espera.
- Consola.
- Plataforma física o virtual.

---

# Paso 3: Seleccionar una entrada

Selecciona el kernel que normalmente utiliza el sistema.

Ejemplo:

```text
Red Hat Enterprise Linux (5.14.0-503.el9.x86_64)
```

Evita elegir inicialmente:

- Una entrada dañada.
- Un kernel que ya presentó errores.
- Una entrada de otro sistema operativo.

---

# Paso 4: Editar la entrada

Presiona:

```text
e
```

GRUB2 mostrará la configuración temporal de esa entrada.

---

# Paso 5: Localizar la línea del kernel

Busca una línea que comience con:

```text
linux
```

En algunos entornos puede aparecer:

```text
linuxefi
```

Ejemplo:

```text
linux /vmlinuz-5.14.0-503.el9.x86_64 root=/dev/mapper/rhel-root ro crashkernel=auto rd.lvm.lv=rhel/root rd.lvm.lv=rhel/swap rhgb quiet
```

---

# Paso 6: Agregar `rd.break`

Al final de la línea agrega:

```text
rd.break
```

Ejemplo:

```text
linux /vmlinuz-5.14.0-503.el9.x86_64 root=/dev/mapper/rhel-root ro rd.lvm.lv=rhel/root rhgb quiet rd.break
```

No elimines:

- `root=`
- `rd.lvm.lv=`
- `rd.luks.uuid=`
- Otros parámetros de almacenamiento necesarios.

---

# Paso 7: Iniciar la entrada modificada

Presiona:

```text
Ctrl+x
```

En algunos sistemas también puede funcionar:

```text
F10
```

El cambio es temporal y solo se aplica a ese arranque.

---

# Paso 8: Identificar la shell de recuperación

El sistema debe detenerse en una shell similar a:

```text
switch_root:/#
```

o:

```text
dracut:/#
```

El prompt puede variar según la versión.

---

# Paso 9: Consultar el sistema raíz

Verifica si `/sysroot` está montado:

```bash
mount | grep sysroot
```

También:

```bash
findmnt /sysroot
```

Salida conceptual:

```text
TARGET   SOURCE                 FSTYPE OPTIONS
/sysroot /dev/mapper/rhel-root  xfs    ro,relatime
```

La opción:

```text
ro
```

indica que está montado en modo de solo lectura.

---

# Paso 10: Remontar `/sysroot` en lectura y escritura

Ejecuta:

```bash
mount -o remount,rw /sysroot
```

Verifica:

```bash
findmnt /sysroot
```

Ahora debe aparecer:

```text
rw
```

---

# ¿Por qué es necesario remontarlo?

Cambiar una contraseña modifica archivos como:

```text
/etc/shadow
```

Si el sistema raíz está en solo lectura, el cambio no podrá guardarse.

---

# Error si permanece en solo lectura

Podría aparecer:

```text
passwd: Authentication token manipulation error
```

También:

```text
passwd: password unchanged
```

Por eso debe verificarse que `/sysroot` esté en modo:

```text
rw
```

---

# Paso 11: Entrar al sistema instalado

Ejecuta:

```bash
chroot /sysroot
```

Después de este comando, el directorio:

```text
/sysroot
```

se convierte en la raíz lógica:

```text
/
```

---

# Antes y después de `chroot`

Antes:

```text
/sysroot/etc/shadow
```

Después:

```text
/etc/shadow
```

---

# Verificar el entorno

Después de entrar mediante `chroot`:

```bash
pwd
```

Puede mostrar:

```text
/
```

Verifica el sistema:

```bash
cat /etc/redhat-release
```

Consulta el usuario `root`:

```bash
id root
```

---

# Paso 12: Cambiar la contraseña

Ejecuta:

```bash
passwd root
```

El sistema solicitará la nueva contraseña dos veces.

Ejemplo:

```text
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
```

---

# Requisitos para una contraseña segura

La contraseña debe:

- Ser suficientemente larga.
- Combinar letras mayúsculas y minúsculas.
- Incluir números.
- Incluir caracteres especiales.
- No contener el nombre del usuario.
- No ser una contraseña reutilizada.
- Cumplir la política PAM del sistema.

Ejemplo conceptual:

```text
R3cuperacion-Linux-2026!
```

No utilices este ejemplo como contraseña real.

---

# Verificar el estado de la contraseña

```bash
passwd -S root
```

Salida conceptual:

```text
root PS 2026-07-28 0 99999 7 -1
```

Donde:

```text
PS
```

indica que existe una contraseña configurada.

---

# Consultar información de expiración

```bash
chage -l root
```

Puede mostrar:

- Fecha del último cambio.
- Fecha de expiración.
- Días mínimos.
- Días máximos.
- Advertencia previa.
- Estado de la cuenta.

---

# Paso 13: Comprobar si la cuenta está bloqueada

Consultar:

```bash
passwd -S root
```

Un estado como:

```text
LK
```

puede indicar que la contraseña está bloqueada.

---

# Desbloquear la cuenta de root

Si es necesario:

```bash
passwd -u root
```

También puede utilizarse:

```bash
usermod -U root
```

Después verifica:

```bash
passwd -S root
```

> No desbloquees una cuenta sin comprender por qué fue bloqueada.

---

# Bloqueo por intentos fallidos

En sistemas que utilizan `faillock`, la cuenta puede estar bloqueada debido a múltiples intentos incorrectos.

Consultar:

```bash
faillock --user root
```

Restablecer el contador:

```bash
faillock --user root --reset
```

---

# Verificar el shell de root

```bash
getent passwd root
```

Salida conceptual:

```text
root:x:0:0:root:/root:/bin/bash
```

Comprueba que el shell no sea:

```text
/sbin/nologin
```

o:

```text
/bin/false
```

salvo que exista una política específica.

---

# Paso 14: Preparar SELinux

Después de modificar la contraseña desde un entorno de recuperación, se debe considerar el contexto SELinux de los archivos modificados.

La opción habitual es crear:

```text
/.autorelabel
```

Ejecuta dentro del `chroot`:

```bash
touch /.autorelabel
```

---

# ¿Qué hace `/.autorelabel`?

Este archivo indica que, durante el próximo arranque, el sistema debe revisar y restaurar los contextos SELinux.

```text
touch /.autorelabel
        │
        ▼
Reinicio
        │
        ▼
SELinux detecta solicitud
        │
        ▼
Reetiqueta archivos
        │
        ▼
Elimina /.autorelabel
        │
        ▼
Continúa el arranque
```

---

# ¿Por qué es importante SELinux?

Archivos como:

```text
/etc/shadow
/etc/passwd
/etc/group
```

deben tener contextos adecuados.

Consultar el contexto esperado de `/etc/shadow`:

```bash
ls -Z /etc/shadow
```

Ejemplo conceptual:

```text
system_u:object_r:shadow_t:s0 /etc/shadow
```

---

# Alternativa: restauración puntual

Puede intentarse restaurar el contexto de los archivos modificados:

```bash
restorecon -v /etc/shadow
```

También:

```bash
restorecon -v /etc/passwd
```

```bash
restorecon -v /etc/group
```

Sin embargo, durante una recuperación de contraseña se suele recomendar:

```bash
touch /.autorelabel
```

para asegurar una revisión más completa.

---

# Reetiquetado completo frente a puntual

| Método | Alcance |
|---|---|
| `restorecon /etc/shadow` | Archivo específico |
| `restorecon -RFv /etc` | Árbol específico |
| `touch /.autorelabel` | Reetiquetado durante el próximo arranque |

---

# Duración del reetiquetado

El proceso puede tardar varios minutos o más, dependiendo de:

- Cantidad de archivos.
- Capacidad del disco.
- Velocidad del almacenamiento.
- Tamaño de los sistemas montados.
- Cantidad de volúmenes.
- Hardware físico o virtual.

No interrumpas el proceso.

---

# Paso 15: Sincronizar los cambios

Antes de salir, puede ejecutarse:

```bash
sync
```

Esto solicita escribir en disco los cambios pendientes.

---

# Paso 16: Salir de `chroot`

Ejecuta:

```bash
exit
```

Volverás a la shell de `initramfs`.

---

# Paso 17: Salir del entorno de recuperación

Ejecuta nuevamente:

```bash
exit
```

El sistema intentará continuar el arranque.

Dependiendo del entorno, también puede utilizarse:

```bash
reboot -f
```

Sin embargo, es preferible salir normalmente cuando sea posible.

---

# Secuencia resumida de comandos

```bash
mount -o remount,rw /sysroot
chroot /sysroot
passwd root
touch /.autorelabel
sync
exit
exit
```

---

# Procedimiento completo resumido

```text
1. Reiniciar
2. Mostrar GRUB2
3. Seleccionar kernel
4. Presionar e
5. Agregar rd.break
6. Presionar Ctrl+x
7. mount -o remount,rw /sysroot
8. chroot /sysroot
9. passwd root
10. touch /.autorelabel
11. sync
12. exit
13. exit
14. Esperar reetiquetado
15. Probar acceso
```

---

# Verificar el acceso después del arranque

Una vez finalizado el arranque, accede mediante consola:

```text
login: root
Password:
```

También puede verificarse desde otro usuario administrativo:

```bash
su -
```

---

# Verificar identidad

```bash
id
```

Resultado esperado:

```text
uid=0(root) gid=0(root) groups=0(root)
```

---

# Verificar la contraseña y expiración

```bash
passwd -S root
```

```bash
chage -l root
```

---

# Comprobar SELinux

```bash
getenforce
```

Resultado normal:

```text
Enforcing
```

Verifica el contexto:

```bash
ls -Z /etc/shadow
```

---

# Revisar errores del arranque

```bash
journalctl -b -p err
```

Buscar mensajes SELinux:

```bash
journalctl -b | grep -i selinux
```

Buscar reetiquetado:

```bash
journalctl -b | grep -i relabel
```

---

# Recuperación cuando `/sysroot` no está montado

Si:

```bash
findmnt /sysroot
```

no muestra resultados, comprueba los dispositivos:

```bash
lsblk -f
```

```bash
blkid
```

---

# Activar volúmenes LVM

Consulta:

```bash
lvm pvs
```

```bash
lvm vgs
```

```bash
lvm lvs
```

Activa grupos de volúmenes:

```bash
lvm vgchange -ay
```

Después revisa:

```bash
ls /dev/mapper
```

---

# Montar manualmente el sistema raíz

Ejemplo:

```bash
mount /dev/mapper/rhel-root /sysroot
```

Si se requiere escritura:

```bash
mount -o rw /dev/mapper/rhel-root /sysroot
```

El nombre real debe confirmarse mediante:

```bash
lsblk -f
```

---

# Sistemas con partición `/usr` separada

Si `/usr` está en un sistema de archivos separado, puede ser necesario montarlo antes de `chroot`.

Ejemplo:

```bash
mount /dev/mapper/rhel-usr /sysroot/usr
```

También pueden requerirse:

```text
/var
/home
/boot
```

según la tarea.

Para cambiar una contraseña normalmente basta con acceder correctamente a:

```text
/etc
```

pero algunos comandos pueden depender de `/usr`.

---

# Sistemas con LUKS

Si el sistema raíz está cifrado, puede ser necesario desbloquearlo.

Consulta:

```bash
lsblk -f
```

Identifica un dispositivo:

```text
crypto_LUKS
```

Abrirlo:

```bash
cryptsetup open /dev/dispositivo nombre_cifrado
```

Después activa LVM si corresponde:

```bash
lvm vgchange -ay
```

Monta el volumen raíz:

```bash
mount /dev/mapper/rhel-root /sysroot
```

---

# Contraseña del disco frente a contraseña de root

La contraseña de LUKS no es la misma que la contraseña de `root`.

| Credencial | Función |
|---|---|
| Contraseña LUKS | Desbloquea el almacenamiento cifrado |
| Contraseña root | Autentica la cuenta administrativa |
| Contraseña GRUB | Protege la edición del cargador |
| Contraseña UEFI | Protege la configuración del firmware |

Recuperar la contraseña de `root` no permite evitar el cifrado del disco.

---

# Sistemas con GRUB protegido

Si GRUB2 tiene contraseña, al presionar:

```text
e
```

puede solicitar credenciales.

Sin estas credenciales, no será posible modificar temporalmente la entrada mediante el procedimiento normal.

En ese caso puede ser necesario:

- Utilizar credenciales autorizadas de GRUB.
- Arrancar desde un medio de rescate autorizado.
- Aplicar procedimientos corporativos.
- Contactar al administrador responsable.

---

# Alternativa mediante medio de instalación

También puede recuperarse el sistema utilizando un medio de instalación de RHEL.

Flujo general:

```text
Arrancar desde ISO
        │
        ▼
Troubleshooting
        │
        ▼
Rescue a Red Hat Enterprise Linux system
        │
        ▼
Montar sistema instalado
        │
        ▼
chroot
        │
        ▼
passwd root
        │
        ▼
Reetiquetado SELinux
```

---

# Entorno de rescate del instalador

El sistema instalado puede montarse en:

```text
/mnt/sysroot
```

o en otra ruta indicada por el entorno.

En ese caso:

```bash
chroot /mnt/sysroot
```

Después:

```bash
passwd root
```

```bash
touch /.autorelabel
```

La ruta debe confirmarse en la consola del medio de rescate.

---

# `rd.break` frente a medio de rescate

| Característica | `rd.break` | Medio de rescate |
|---|---|---|
| Requiere GRUB funcional | Sí | No necesariamente |
| Requiere ISO o USB | No | Sí |
| Acceso al sistema instalado | `/sysroot` | Frecuentemente `/mnt/sysroot` |
| Uso principal | Recuperación rápida | Fallos graves de arranque |
| Funciona con GRUB dañado | No siempre | Puede funcionar |
| Requiere cambiar orden de arranque | No | Posiblemente |

---

# Problema: `passwd` muestra error de manipulación

Mensaje posible:

```text
passwd: Authentication token manipulation error
```

Causas comunes:

- `/sysroot` continúa en solo lectura.
- No se ejecutó `chroot`.
- `/etc/shadow` tiene permisos incorrectos.
- El sistema de archivos está lleno.
- El sistema de archivos presenta errores.

---

# Solución inicial

Verifica:

```bash
findmnt /
```

Dentro del `chroot`, debe mostrar escritura:

```text
rw
```

Si es necesario, sal y remonta:

```bash
mount -o remount,rw /sysroot
```

---

# Verificar espacio disponible

Dentro de `chroot`:

```bash
df -h /
```

Consultar inodos:

```bash
df -i /
```

Un sistema de archivos lleno puede impedir modificar `/etc/shadow`.

---

# Verificar permisos de `/etc/shadow`

```bash
ls -l /etc/shadow
```

Salida esperada normalmente:

```text
-rw-r----- 1 root root ...
```

En algunos sistemas el grupo puede ser:

```text
shadow
```

No cambies permisos sin comparar con una instalación equivalente y comprender la política local.

---

# Verificar integridad de archivos de cuentas

```bash
pwck -r
```

Comprobar grupos:

```bash
grpck -r
```

La opción:

```text
-r
```

realiza una comprobación de solo lectura.

---

# Problema: la contraseña cambió, pero no permite acceso

Posibles causas:

- La cuenta continúa bloqueada.
- Existe un bloqueo de `faillock`.
- La contraseña expiró.
- El shell no permite iniciar sesión.
- PAM está mal configurado.
- El acceso de `root` está restringido.
- El acceso SSH de `root` está deshabilitado.
- El teclado utilizado al cambiar la contraseña tenía otra distribución.

---

# Verificar bloqueo

```bash
passwd -S root
```

```bash
faillock --user root
```

Restablecer si corresponde:

```bash
faillock --user root --reset
```

---

# Verificar expiración

```bash
chage -l root
```

Si debe eliminarse una fecha de expiración de cuenta:

```bash
chage -E -1 root
```

Si debe establecerse una nueva fecha de cambio:

```bash
chage -d "$(date +%F)" root
```

Realiza estos cambios únicamente cuando sean necesarios.

---

# Verificar shell

```bash
getent passwd root
```

El último campo normalmente debe ser un shell válido:

```text
/bin/bash
```

Consulta shells permitidos:

```bash
cat /etc/shells
```

---

# Acceso root por SSH

Cambiar la contraseña no garantiza que `root` pueda iniciar sesión por SSH.

Consulta:

```bash
sshd -T | grep permitrootlogin
```

Posibles valores:

```text
yes
no
prohibit-password
forced-commands-only
```

En entornos seguros suele deshabilitarse el acceso remoto directo de `root`.

La práctica recomendada es:

1. Iniciar sesión con un usuario nominal.
2. Utilizar `sudo`.
3. Auditar las acciones administrativas.

---

# Teclado y caracteres especiales

Durante GRUB o la shell de recuperación, la distribución del teclado puede ser distinta de la habitual.

Esto puede afectar caracteres como:

```text
@
#
$
%
&
/
-
_
```

Para evitar errores:

- Escribe cuidadosamente.
- Confirma la distribución del teclado.
- Utiliza temporalmente una contraseña segura pero que puedas introducir correctamente.
- Cámbiala nuevamente después de iniciar el sistema, si la política lo exige.

---

# Problema: `touch /.autorelabel` no persiste

Verifica que estás dentro del `chroot`.

```bash
pwd
```

Comprueba:

```bash
ls -la /.autorelabel
```

Si lo ejecutas fuera del `chroot`, el archivo puede crearse en el entorno temporal de `initramfs`, no en el sistema instalado.

---

# Ubicación correcta de `.autorelabel`

Antes de `chroot`:

```text
/sysroot/.autorelabel
```

Después de `chroot`:

```text
/.autorelabel
```

Por eso se recomienda:

```bash
chroot /sysroot
touch /.autorelabel
```

---

# Problema: el reetiquetado tarda demasiado

Es normal que tarde si:

- Existen millones de archivos.
- El disco es lento.
- El servidor tiene múltiples sistemas de archivos.
- La máquina virtual tiene recursos limitados.

No apagues el equipo durante el reetiquetado.

---

# Problema: el sistema se reinicia dos veces

Dependiendo de la versión y del procedimiento de reetiquetado, el sistema puede:

- Reetiquetar.
- Reiniciarse.
- Completar el arranque en un segundo ciclo.

Esto puede ser comportamiento esperado.

---

# Problema: el menú de GRUB no aparece

Consulta desde un sistema funcional:

```bash
grep -E \
'GRUB_TIMEOUT|GRUB_TIMEOUT_STYLE' \
/etc/default/grub
```

También:

```bash
grub2-editenv list
```

Puede existir:

```text
menu_auto_hide=1
```

La configuración debe cambiarse con cuidado y verificarse antes de reiniciar.

---

# Problema: `rd.break` no detiene el arranque

Verifica que:

- Fue agregado a la línea del kernel.
- No fue agregado a una línea incorrecta.
- La entrada modificada fue la utilizada.
- Se inició con `Ctrl+x`.
- El parámetro aparece exactamente como:

```text
rd.break
```

No debe escribirse:

```text
rd break
```

ni:

```text
rd-break
```

---

# Problema: el sistema inicia con otro kernel

Después de recuperar el acceso, consulta:

```bash
uname -r
```

```bash
grubby --default-kernel
```

```bash
grubby --info=ALL
```

Podrías haber editado una entrada diferente a la predeterminada.

---

# Restaurar contextos específicos

Después del arranque:

```bash
restorecon -v /etc/shadow
```

También puede verificarse:

```bash
matchpathcon /etc/shadow
```

Comparar:

```bash
ls -Z /etc/shadow
```

---

# Auditoría posterior a la recuperación

Después de recuperar el acceso, revisa:

```bash
journalctl -b
```

```bash
journalctl -b -p err
```

```bash
last
```

```bash
lastlog
```

```bash
faillock
```

```bash
ausearch -m USER_CHAUTHTOK -ts today
```

La disponibilidad de registros depende de cómo se realizó la recuperación y de la configuración del sistema.

---

# Documentar la recuperación

En un entorno corporativo registra:

- Fecha y hora.
- Servidor afectado.
- Motivo.
- Persona que autorizó el cambio.
- Consola utilizada.
- Kernel seleccionado.
- Método de recuperación.
- Cuenta modificada.
- Estado de SELinux.
- Resultado final.
- Evidencias de verificación.

No registres la contraseña.

---

# Seguridad física y recuperación de root

El procedimiento demuestra que una persona con acceso al arranque puede intentar obtener control administrativo.

Para reducir este riesgo:

- Protege el acceso físico.
- Configura una contraseña de firmware.
- Restringe el arranque desde USB.
- Protege la edición de GRUB.
- Utiliza cifrado LUKS.
- Habilita Secure Boot cuando corresponda.
- Controla las consolas remotas.
- Registra accesos a iDRAC, iLO o hipervisor.
- Usa separación de funciones administrativas.

---

# Protección de GRUB2

Puede generarse un hash mediante:

```bash
grub2-mkpasswd-pbkdf2
```

La protección puede restringir:

- Edición de entradas.
- Acceso a la consola de GRUB.
- Ejecución de entradas protegidas.

Su configuración exacta depende de la versión de RHEL y del uso de BLS.

---

# Protección mediante cifrado

Una contraseña de GRUB dificulta la edición del menú, pero no protege los datos si el disco puede extraerse.

El cifrado LUKS protege el contenido del almacenamiento.

```text
Contraseña GRUB
      │
      └── Protege el cargador

Cifrado LUKS
      │
      └── Protege los datos del disco
```

Ambos controles cumplen funciones diferentes.

---

# Recuperar root no equivale a habilitar acceso SSH

Después de cambiar la contraseña:

```bash
su -
```

puede funcionar localmente, mientras:

```bash
ssh root@servidor
```

continúa bloqueado.

Esto puede ser correcto debido a:

```text
PermitRootLogin no
```

o:

```text
PermitRootLogin prohibit-password
```

---

# Método recomendado de administración diaria

En lugar de trabajar permanentemente como `root`:

```bash
sudo comando
```

o:

```bash
sudo -i
```

Esto permite:

- Identificar quién realizó la acción.
- Aplicar permisos específicos.
- Registrar eventos.
- Evitar compartir la contraseña de `root`.
- Reducir el uso de acceso administrativo directo.

---

# Recuperación de un usuario administrativo alternativo

Si existe otro usuario con `sudo`, puede cambiarse la contraseña sin reiniciar:

```bash
sudo passwd root
```

También puede desbloquearse:

```bash
sudo passwd -u root
```

Restablecer `faillock`:

```bash
sudo faillock \
--user root \
--reset
```

Este es el método preferido cuando todavía existe acceso administrativo válido.

---

# Comprobación previa antes de reiniciar

Dentro de `chroot`, verifica:

```bash
passwd -S root
```

```bash
chage -l root
```

```bash
ls -l /etc/shadow
```

```bash
ls -Z /etc/shadow
```

```bash
ls -la /.autorelabel
```

```bash
df -h /
```

Después:

```bash
sync
```

---

# Flujo de diagnóstico

```text
No puedo iniciar como root
        │
        ▼
¿Existe otro usuario con sudo?
        │
        ├── Sí
        │    └── sudo passwd root
        │
        └── No
             │
             ▼
       Acceder a consola
             │
             ▼
       Editar GRUB con rd.break
             │
             ▼
       Verificar /sysroot
             │
             ▼
       Remontar en rw
             │
             ▼
       Ejecutar chroot
             │
             ▼
       Cambiar contraseña
             │
             ▼
       Desbloquear si es necesario
             │
             ▼
       Crear /.autorelabel
             │
             ▼
       Reiniciar y verificar
```

---

# Buenas prácticas RHCSA

✔ Utilizar primero otro usuario con `sudo` si está disponible.

✔ Realizar la recuperación desde consola, no únicamente por SSH.

✔ Verificar el kernel seleccionado antes de editarlo.

✔ Agregar `rd.break` sin eliminar parámetros de almacenamiento.

✔ Confirmar que `/sysroot` está montado.

✔ Remontar `/sysroot` en lectura y escritura.

✔ Ejecutar `chroot /sysroot` antes de cambiar la contraseña.

✔ Utilizar una contraseña segura.

✔ Consultar si la cuenta está bloqueada.

✔ Restablecer `faillock` solo cuando sea necesario.

✔ Crear `/.autorelabel` dentro del `chroot`.

✔ Ejecutar `sync` antes de salir.

✔ Esperar a que termine el reetiquetado.

✔ Verificar SELinux después del arranque.

✔ Documentar la recuperación sin registrar la contraseña.

✔ Proteger GRUB, firmware y acceso físico.

---

# Errores comunes

## Ejecutar `passwd root` antes de `chroot`

Esto puede intentar modificar el entorno temporal, no el sistema instalado.

---

## No remontar `/sysroot` en escritura

La contraseña no podrá guardarse.

---

## Crear `.autorelabel` fuera de `chroot`

El archivo puede quedar en el entorno temporal y desaparecer al reiniciar.

---

## Interrumpir el reetiquetado SELinux

Puede dejar archivos con contextos incorrectos.

---

## Eliminar parámetros LVM o LUKS en GRUB

El sistema puede dejar de encontrar el volumen raíz.

---

## Confundir contraseña root con contraseña LUKS

La contraseña de cifrado debe proporcionarse antes de acceder a los datos.

---

## Suponer que root podrá entrar por SSH

El acceso remoto puede estar deshabilitado por política.

---

## Desactivar SELinux permanentemente

No es necesario para recuperar la contraseña y reduce la seguridad.

---

## Usar una contraseña difícil de escribir en la consola

La distribución del teclado puede ser distinta durante la recuperación.

---

## No comprobar si la cuenta sigue bloqueada

Cambiar la contraseña no siempre elimina bloqueos de PAM o `faillock`.

---

## Utilizar `reboot -f` innecesariamente

Un reinicio forzado puede evitar que se escriban correctamente los cambios.

---

# Comandos importantes RHCSA

| Comando | Descripción |
|---|---|
| `rd.break` | Interrumpir el arranque en `initramfs` |
| `mount -o remount,rw /sysroot` | Remontar el sistema raíz |
| `findmnt /sysroot` | Verificar el montaje |
| `chroot /sysroot` | Entrar al sistema instalado |
| `passwd root` | Cambiar la contraseña |
| `passwd -S root` | Consultar estado de contraseña |
| `passwd -u root` | Desbloquear la contraseña |
| `chage -l root` | Consultar expiración |
| `faillock --user root` | Consultar intentos fallidos |
| `faillock --user root --reset` | Restablecer bloqueo |
| `touch /.autorelabel` | Solicitar reetiquetado SELinux |
| `restorecon -v /etc/shadow` | Restaurar contexto puntual |
| `ls -Z /etc/shadow` | Consultar contexto SELinux |
| `sync` | Escribir cambios pendientes |
| `exit` | Salir de `chroot` o de la shell |
| `lsblk -f` | Consultar sistemas de archivos |
| `lvm vgchange -ay` | Activar grupos LVM |
| `cryptsetup open` | Abrir un volumen LUKS |
| `journalctl -b` | Revisar el arranque actual |
| `getenforce` | Consultar el modo SELinux |

---

# Tabla de recuperación rápida

| Etapa | Acción |
|---|---|
| GRUB2 | Presionar `e` |
| Línea kernel | Agregar `rd.break` |
| Inicio | Presionar `Ctrl+x` |
| Shell inicial | Verificar `/sysroot` |
| Escritura | `mount -o remount,rw /sysroot` |
| Sistema instalado | `chroot /sysroot` |
| Contraseña | `passwd root` |
| SELinux | `touch /.autorelabel` |
| Escritura a disco | `sync` |
| Salida | `exit` dos veces |
| Verificación | Iniciar sesión y ejecutar `getenforce` |

---

# Resumen rápido

```text
Recuperación de root
        │
        ├── GRUB2
        │     ├── Presionar e
        │     └── Agregar rd.break
        │
        ├── initramfs
        │     └── /sysroot
        │
        ├── Escritura
        │     └── mount -o remount,rw
        │
        ├── Sistema instalado
        │     └── chroot /sysroot
        │
        ├── Credenciales
        │     ├── passwd root
        │     ├── passwd -u root
        │     └── faillock --reset
        │
        ├── SELinux
        │     └── touch /.autorelabel
        │
        └── Verificación
              ├── passwd -S root
              ├── getenforce
              └── journalctl -b
```

---

# Resumen

En esta lección aprendiste a:

- Comprender por qué puede ser necesario recuperar `root`.
- Utilizar `rd.break` desde GRUB2.
- Identificar la raíz real en `/sysroot`.
- Remontar el sistema con permisos de escritura.
- Entrar al sistema mediante `chroot`.
- Cambiar y desbloquear la contraseña.
- Restablecer bloqueos de `faillock`.
- Preparar un reetiquetado SELinux.
- Salir correctamente del entorno.
- Diagnosticar errores comunes.
- Recuperar sistemas con LVM o LUKS.
- Aplicar medidas de seguridad posteriores.

---

# Laboratorio práctico RHCSA

> Realiza este laboratorio únicamente en una máquina virtual de pruebas con acceso directo a la consola. Crea una instantánea antes de comenzar.

## Escenario 1: Preparar el laboratorio

Verifica el kernel actual:

```bash
uname -r
```

Consulta el modo SELinux:

```bash
getenforce
```

Verifica la cuenta:

```bash
sudo passwd -S root
```

---

## Escenario 2: Crear una instantánea

Desde el hipervisor crea una instantánea con un nombre similar a:

```text
antes-recuperacion-root
```

No continúes sin disponer de un método de reversión.

---

## Escenario 3: Reiniciar y editar GRUB

Reinicia:

```bash
sudo systemctl reboot
```

En el menú:

1. Selecciona el kernel.
2. Presiona `e`.
3. Localiza la línea `linux`.
4. Agrega:

```text
rd.break
```

5. Inicia con:

```text
Ctrl+x
```

---

## Escenario 4: Examinar el entorno

En la shell:

```bash
pwd
```

```bash
cat /proc/cmdline
```

```bash
findmnt /sysroot
```

```bash
ls /sysroot
```

Identifica la diferencia entre `/` y `/sysroot`.

---

## Escenario 5: Remontar el sistema

```bash
mount -o remount,rw /sysroot
```

Verifica:

```bash
findmnt /sysroot
```

Confirma que aparece:

```text
rw
```

---

## Escenario 6: Entrar mediante chroot

```bash
chroot /sysroot
```

Verifica:

```bash
cat /etc/redhat-release
```

```bash
id root
```

---

## Escenario 7: Cambiar la contraseña

```bash
passwd root
```

Utiliza una contraseña temporal segura para el laboratorio.

Verifica:

```bash
passwd -S root
```

---

## Escenario 8: Comprobar bloqueo

```bash
faillock --user root
```

Si aparecen intentos bloqueados:

```bash
faillock \
--user root \
--reset
```

---

## Escenario 9: Verificar expiración

```bash
chage -l root
```

Confirma que la cuenta no esté expirada.

---

## Escenario 10: Preparar SELinux

```bash
touch /.autorelabel
```

Verifica:

```bash
ls -la /.autorelabel
```

---

## Escenario 11: Sincronizar y salir

```bash
sync
```

```bash
exit
```

```bash
exit
```

Espera a que el sistema complete el reetiquetado.

---

## Escenario 12: Verificar acceso

Después del arranque:

```bash
su -
```

Introduce la nueva contraseña.

Verifica:

```bash
id
```

---

## Escenario 13: Verificar SELinux

```bash
getenforce
```

```bash
ls -Z /etc/shadow
```

```bash
matchpathcon /etc/shadow
```

---

## Escenario 14: Revisar registros

```bash
journalctl -b -p err
```

```bash
journalctl -b \
| grep -Ei 'selinux|relabel'
```

---

## Escenario 15: Cambiar la contraseña nuevamente

Después de comprobar el laboratorio, establece una contraseña administrativa definitiva:

```bash
passwd root
```

No reutilices la contraseña temporal.

---

# Script de verificación posterior

```bash
#!/bin/bash

REPORTE="$HOME/reporte-recuperacion-root.txt"

{
    echo "=================================================="
    echo "VERIFICACIÓN POSTERIOR A RECUPERACIÓN DE ROOT"
    echo "=================================================="
    echo

    echo "Fecha:"
    date
    echo

    echo "Hostname:"
    hostname
    echo

    echo "Kernel:"
    uname -r
    echo

    echo "Estado de la contraseña root:"
    passwd -S root
    echo

    echo "Expiración de root:"
    chage -l root
    echo

    echo "Entrada passwd de root:"
    getent passwd root
    echo

    echo "Intentos fallidos:"
    faillock --user root
    echo

    echo "Modo SELinux:"
    getenforce
    echo

    echo "Contexto de /etc/shadow:"
    ls -Z /etc/shadow
    echo

    echo "Contexto esperado:"
    matchpathcon /etc/shadow
    echo

    echo "Estado general:"
    systemctl is-system-running
    echo

    echo "Unidades fallidas:"
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
~/verificar-recuperacion-root.sh
```

Asignar permisos:

```bash
chmod +x \
~/verificar-recuperacion-root.sh
```

Ejecutar como `root`:

```bash
sudo \
~/verificar-recuperacion-root.sh
```

---

# Lista de comprobación

Marca cada tarea completada:

```text
[ ] Accedí al menú de GRUB2
[ ] Seleccioné el kernel correcto
[ ] Agregué rd.break
[ ] Inicié con Ctrl+x
[ ] Identifiqué /sysroot
[ ] Remonté /sysroot en rw
[ ] Ejecuté chroot /sysroot
[ ] Cambié la contraseña de root
[ ] Verifiqué el estado con passwd -S
[ ] Revisé faillock
[ ] Creé /.autorelabel
[ ] Ejecuté sync
[ ] Salí correctamente
[ ] Esperé el reetiquetado
[ ] Verifiqué el acceso
[ ] Confirmé SELinux Enforcing
[ ] Revisé los registros
[ ] Documenté la intervención
```

---

# Preguntas de repaso

1. ¿Cuándo puede ser necesario recuperar la contraseña de `root`?
2. ¿Qué acceso se necesita para modificar GRUB2?
3. ¿Qué función cumple `rd.break`?
4. ¿En qué etapa del arranque se detiene el sistema?
5. ¿Dónde se encuentra normalmente el sistema raíz real?
6. ¿Por qué `/sysroot` suele estar en solo lectura?
7. ¿Cómo se remonta en lectura y escritura?
8. ¿Qué función cumple `chroot /sysroot`?
9. ¿Qué archivo almacena los hashes de contraseña?
10. ¿Qué comando cambia la contraseña de `root`?
11. ¿Cómo se consulta el estado de la contraseña?
12. ¿Qué significa que una cuenta aparezca como `LK`?
13. ¿Cómo se desbloquea la contraseña?
14. ¿Qué es `faillock`?
15. ¿Cómo se restablece un bloqueo por intentos fallidos?
16. ¿Por qué debe crearse `/.autorelabel`?
17. ¿Dónde debe crearse si ya se ejecutó `chroot`?
18. ¿Qué diferencia existe entre `restorecon` y `.autorelabel`?
19. ¿Por qué el reetiquetado puede tardar?
20. ¿Qué diferencia existe entre contraseña LUKS y contraseña root?
21. ¿Qué ocurre si GRUB está protegido?
22. ¿Por qué una nueva contraseña no garantiza acceso SSH?
23. ¿Qué comando comprueba el modo SELinux?
24. ¿Qué debe verificarse después de la recuperación?
25. ¿Qué controles ayudan a prevenir una recuperación no autorizada?

---

# Desafío final

Realiza las siguientes tareas en una máquina virtual:

1. Crea una instantánea.
2. Consulta el kernel actual.
3. Verifica SELinux.
4. Reinicia y muestra GRUB2.
5. Edita la entrada.
6. Agrega `rd.break`.
7. Inicia con `Ctrl+x`.
8. Identifica `/sysroot`.
9. Verifica si está en solo lectura.
10. Remóntalo en escritura.
11. Ejecuta `chroot`.
12. Cambia la contraseña de `root`.
13. Consulta su estado.
14. Revisa `faillock`.
15. Crea `/.autorelabel`.
16. Ejecuta `sync`.
17. Sal del entorno.
18. Espera el reetiquetado.
19. Comprueba el acceso.
20. Verifica el contexto de `/etc/shadow`.
21. Confirma que SELinux esté en modo correcto.
22. Revisa unidades fallidas.
23. Genera un reporte.
24. Documenta el procedimiento sin revelar la contraseña.

> **Objetivo general:** recuperar de forma segura el acceso administrativo a un sistema Red Hat Enterprise Linux mediante GRUB2, `rd.break`, `/sysroot`, `chroot`, `passwd` y el reetiquetado de SELinux, manteniendo la integridad del sistema y aplicando controles adecuados de seguridad y auditoría.