# **66. Diagnóstico de problemas de arranque**

El archivo correspondiente será:

```text
66-diagnostico-problemas-arranque.md
```

Este capítulo integrará todo lo estudiado sobre GRUB2, kernel, parámetros de arranque, `systemd`, Rescue, Emergency, `initramfs` y Dracut, enfocándose en una metodología profesional para localizar exactamente en qué etapa falla un sistema RHEL.

## Estructura propuesta

### 1. Objetivos de aprendizaje

* Reconocer las distintas etapas del arranque.
* Identificar en qué etapa ocurre un fallo.
* Diferenciar problemas de firmware, GRUB2, kernel, `initramfs` y `systemd`.
* Analizar mensajes del arranque actual y anteriores.
* Diagnosticar montajes, dispositivos, servicios y targets.
* Utilizar herramientas de recuperación.
* Aplicar procedimientos similares a los evaluados en RHCSA.

---

### 2. Introducción al diagnóstico de arranque

El capítulo comenzará con el flujo completo:

```text
Encendido
   │
   ▼
BIOS / UEFI
   │
   ▼
GRUB2
   │
   ▼
Kernel
   │
   ▼
initramfs / dracut
   │
   ▼
Sistema de archivos raíz
   │
   ▼
systemd
   │
   ▼
Targets
   │
   ▼
Servicios
   │
   ▼
Inicio de sesión
```

La regla principal será:

```text
Identificar la última etapa que funcionó correctamente.
```

---

### 3. Clasificación de fallos

| Etapa            | Síntomas habituales                               |
| ---------------- | ------------------------------------------------- |
| Firmware         | No se detecta el disco o no aparece GRUB          |
| GRUB2            | `grub rescue>`, menú ausente o entrada incorrecta |
| Kernel           | Kernel panic, bloqueo o controlador ausente       |
| initramfs        | `dracut-initqueue timeout`, raíz no encontrada    |
| Montaje raíz     | No se puede montar `/`                            |
| systemd          | Emergency Mode, unidades fallidas                 |
| Servicios        | Sistema degradado o servicio crítico detenido     |
| Inicio de sesión | Consola, SSH o entorno gráfico no disponibles     |

---

### 4. Diagnóstico desde la pantalla

Se explicará cómo interpretar mensajes como:

```text
No bootable device
```

```text
grub rescue>
```

```text
error: file not found
```

```text
Kernel panic - not syncing
```

```text
dracut-initqueue timeout
```

```text
Warning: /dev/mapper/rhel-root does not exist
```

```text
Failed to mount /var
```

```text
Dependency failed for Local File Systems
```

```text
You are in emergency mode
```

```text
Start request repeated too quickly
```

---

### 5. Diagnóstico del firmware

* BIOS frente a UEFI.
* Orden de arranque.
* Detección de discos.
* Secure Boot.
* Entradas UEFI.
* Partición EFI.
* Herramientas:

```bash
efibootmgr -v
```

```bash
lsblk -f
```

```bash
findmnt /boot/efi
```

```bash
mokutil --sb-state
```

---

### 6. Diagnóstico de GRUB2

* Menú GRUB ausente.
* Entrada BLS incorrecta.
* Kernel o `initramfs` inexistente.
* Prefijo y raíz de GRUB.
* Consola `grub>`.
* Consola `grub rescue>`.
* Variables de entorno.
* Configuración persistente.

Comandos:

```bash
grubby --info=ALL
```

```bash
grub2-editenv list
```

```bash
grub2-script-check /boot/grub2/grub.cfg
```

```bash
ls -l /boot/loader/entries
```

```bash
ls -lh /boot/vmlinuz-* /boot/initramfs-*
```

---

### 7. Diagnóstico del kernel

* Kernel panic.
* Módulos ausentes.
* Argumentos incorrectos.
* Kernel predeterminado.
* Kernel anterior.
* Selección temporal desde GRUB.
* Controladores bloqueados.
* Parámetros problemáticos.

Comandos:

```bash
uname -r
```

```bash
cat /proc/cmdline
```

```bash
grubby --default-kernel
```

```bash
grubby --info=ALL
```

```bash
journalctl -k -b
```

```bash
dmesg -T
```

---

### 8. Diagnóstico de initramfs y Dracut

* Imagen ausente.
* Imagen corrupta.
* Controlador no incluido.
* LVM no detectado.
* LUKS no desbloqueado.
* RAID no ensamblado.
* UUID raíz incorrecto.
* Shell `dracut:/#`.

Comandos:

```bash
lsinitrd
```

```bash
dracut --list-modules
```

```bash
dracut -f -v
```

```bash
lsblk -f
```

```bash
blkid
```

```bash
lvm lvs
```

```bash
cat /run/initramfs/rdsosreport.txt
```

---

### 9. El informe `rdsosreport.txt`

Se explicará el archivo:

```text
/run/initramfs/rdsosreport.txt
```

Consultas:

```bash
less /run/initramfs/rdsosreport.txt
```

```bash
grep -Ei \
'error|warning|timeout|failed|missing' \
/run/initramfs/rdsosreport.txt
```

También se explicará cómo copiarlo a un dispositivo externo para analizarlo posteriormente.

---

### 10. Diagnóstico de la raíz

* UUID incorrecto.
* LV inactivo.
* Disco ausente.
* Sistema de archivos dañado.
* Parámetros `root=` y `rd.lvm.lv=`.
* Montaje `ro` frente a `rw`.

Comandos:

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
pvs
```

```bash
vgs
```

```bash
lvs
```

```bash
vgchange -ay
```

---

### 11. Diagnóstico de `/etc/fstab`

* Errores de sintaxis.
* UUID inexistentes.
* Tipos incorrectos.
* Puntos de montaje ausentes.
* Dispositivos opcionales.
* `nofail`.
* Tiempo de espera de dispositivos.
* Unidades `.mount`.

Comandos:

```bash
findmnt --verify --verbose
```

```bash
mount -av
```

```bash
systemctl --failed --type=mount
```

```bash
systemctl status local-fs.target
```

---

### 12. Diagnóstico de systemd

* Estado `running`.
* Estado `degraded`.
* Estado `maintenance`.
* Unidades fallidas.
* Dependencias.
* Jobs pendientes.
* Targets no alcanzados.

Comandos:

```bash
systemctl is-system-running
```

```bash
systemctl --failed
```

```bash
systemctl list-jobs
```

```bash
systemctl list-dependencies
```

```bash
systemctl get-default
```

---

### 13. Análisis del Journal

Se desarrollarán ampliamente:

```bash
journalctl -xb
```

```bash
journalctl -b
```

```bash
journalctl -b -1
```

```bash
journalctl -k -b
```

```bash
journalctl -b -p err
```

```bash
journalctl --list-boots
```

```bash
journalctl -u servicio.service -b
```

---

### 14. Interpretar prioridades del Journal

| Prioridad | Nombre  |
| --------: | ------- |
|         0 | emerg   |
|         1 | alert   |
|         2 | crit    |
|         3 | err     |
|         4 | warning |
|         5 | notice  |
|         6 | info    |
|         7 | debug   |

Ejemplo:

```bash
journalctl -b -p 0..3
```

---

### 15. Diagnóstico de rendimiento del arranque

Comandos:

```bash
systemd-analyze
```

```bash
systemd-analyze blame
```

```bash
systemd-analyze critical-chain
```

```bash
systemd-analyze critical-chain multi-user.target
```

```bash
systemd-analyze plot > arranque.svg
```

Se explicará la diferencia entre:

* Servicio lento.
* Servicio bloqueado.
* Dependencia lenta.
* Tiempo del kernel.
* Tiempo de `initramfs`.
* Tiempo de espacio de usuario.

---

### 16. Diagnóstico de servicios críticos

* Servicio fallido.
* Configuración inválida.
* Permisos.
* Contextos SELinux.
* Binario ausente.
* Dependencia incorrecta.
* Bucle de reinicio.
* `Start request repeated too quickly`.

Comandos:

```bash
systemctl status servicio.service
```

```bash
journalctl -u servicio.service -b
```

```bash
systemctl cat servicio.service
```

```bash
systemctl show servicio.service
```

```bash
systemctl list-dependencies servicio.service
```

```bash
systemctl list-dependencies --reverse servicio.service
```

---

### 17. Validación de unidades systemd

```bash
systemd-analyze verify \
/etc/systemd/system/aplicacion.service
```

También se explicarán:

* `ExecStart`.
* `User`.
* `Group`.
* `WorkingDirectory`.
* `EnvironmentFile`.
* `Restart`.
* `After`.
* `Wants`.
* `Requires`.

---

### 18. Diagnóstico de almacenamiento

* Errores de entrada y salida.
* Discos ausentes.
* Dispositivos NVMe.
* SCSI.
* SAN.
* Multipath.
* RAID.
* LVM.
* Espacio e inodos.

Comandos:

```bash
dmesg \
| grep -Ei \
'i/o error|medium error|buffer i/o|nvme|scsi|ata'
```

```bash
df -hT
```

```bash
df -i
```

```bash
multipath -ll
```

```bash
mdadm --detail --scan
```

---

### 19. Diagnóstico de sistemas de archivos

Se cubrirán:

```bash
e2fsck -n /dev/dispositivo
```

```bash
e2fsck -f /dev/dispositivo
```

```bash
xfs_repair -n /dev/dispositivo
```

```bash
xfs_repair /dev/dispositivo
```

Con énfasis en no reparar sistemas montados.

---

### 20. Diagnóstico de SELinux

* Denegaciones AVC.
* Contextos incorrectos.
* Problemas después de restauraciones.
* Servicios que funcionan en Permissive pero no en Enforcing.

Comandos:

```bash
getenforce
```

```bash
ausearch -m AVC -ts boot
```

```bash
ls -Zd /ruta
```

```bash
matchpathcon /ruta
```

```bash
restorecon -Rv /ruta
```

---

### 21. Diagnóstico de red durante el arranque

* NetworkManager.
* Interfaces ausentes.
* Dispositivo no administrado.
* Dirección IP.
* Puerta de enlace.
* DNS.
* `network-online.target`.
* Servicios que arrancan antes de tener conectividad.

Comandos:

```bash
nmcli device status
```

```bash
nmcli connection show
```

```bash
ip address
```

```bash
ip route
```

```bash
systemctl status NetworkManager
```

```bash
systemctl status NetworkManager-wait-online.service
```

---

### 22. Diagnóstico del entorno gráfico

* `graphical.target`.
* Display manager.
* GDM.
* Controlador gráfico.
* Problemas con Wayland.
* Inicio en consola para recuperar.

Comandos:

```bash
systemctl get-default
```

```bash
systemctl status display-manager.service
```

```bash
journalctl -u gdm -b
```

```bash
systemctl isolate multi-user.target
```

---

### 23. Diagnóstico de autenticación

* Contraseña de root.
* Cuenta bloqueada.
* Shell inválido.
* `/etc/passwd`.
* `/etc/shadow`.
* PAM.
* `faillock`.
* Sudoers.

Comandos:

```bash
passwd -S root
```

```bash
faillock --user root
```

```bash
pwck -r
```

```bash
grpck -r
```

```bash
visudo -c
```

---

### 24. Rescue, Emergency, `rd.break` o ISO

| Problema                   | Método recomendado         |
| -------------------------- | -------------------------- |
| Servicio defectuoso        | Rescue                     |
| Error en fstab             | Emergency                  |
| Contraseña root            | `rd.break`                 |
| Raíz no detectada          | Dracut shell               |
| Initramfs dañado           | ISO de rescate             |
| Reparar raíz XFS           | ISO de rescate             |
| GRUB dañado                | ISO o consola GRUB         |
| Kernel reciente defectuoso | Kernel anterior desde GRUB |

---

### 25. Arranque con kernel anterior

Procedimiento:

1. Mostrar GRUB.
2. Seleccionar una entrada anterior.
3. Iniciar.
4. Confirmar versión:

```bash
uname -r
```

5. Revisar imágenes:

```bash
ls -lh /boot/vmlinuz-* /boot/initramfs-*
```

6. Corregir o eliminar el kernel defectuoso únicamente después de validar.

---

### 26. Recuperación desde una ISO

Se explicará:

* Arranque desde medio de instalación.
* Selección de Rescue.
* Montaje en `/mnt/sysroot`.
* Montaje de `/boot` y `/boot/efi`.
* `chroot`.
* Regeneración de `initramfs`.
* Reconstrucción de GRUB.
* Corrección de `/etc/fstab`.
* Restauración de SELinux.

Flujo:

```text
ISO de rescate
      │
      ▼
Detectar LVM
      │
      ▼
Montar raíz
      │
      ▼
Montar /boot y /boot/efi
      │
      ▼
chroot /mnt/sysroot
      │
      ▼
Corregir
      │
      ▼
dracut / GRUB
      │
      ▼
Salir y reiniciar
```

---

### 27. Casos prácticos completos

El capítulo incluirá casos como:

1. GRUB no encuentra el kernel.
2. Kernel nuevo provoca panic.
3. `initramfs` no detecta LVM.
4. UUID raíz incorrecto.
5. `/etc/fstab` bloquea el arranque.
6. `/var` está lleno.
7. Un servicio entra en bucle.
8. SELinux bloquea un servicio crítico.
9. El target predeterminado es incorrecto.
10. El entorno gráfico no inicia.
11. El servidor queda en estado `degraded`.
12. El arranque tarda varios minutos.
13. El disco SAN no aparece.
14. El volumen XFS requiere reparación.
15. El `initramfs` debe regenerarse.

---

## División recomendada

Por la extensión y profundidad del capítulo, conviene desarrollarlo en cuatro fases:

### **Fase 1 — Fundamentos y clasificación**

* Metodología de diagnóstico.
* Etapas del arranque.
* Síntomas.
* Firmware.
* GRUB2.
* Kernel.
* Árboles de decisión.
* Primeras herramientas.

### **Fase 2 — initramfs, raíz, almacenamiento y montajes**

* Dracut.
* `rdsosreport.txt`.
* LVM.
* LUKS.
* RAID.
* Sistemas de archivos.
* `/etc/fstab`.
* Unidades `.mount`.
* Errores de entrada y salida.

### **Fase 3 — systemd, servicios, SELinux, red y rendimiento**

* Journal.
* Unidades fallidas.
* Dependencias.
* Targets.
* Servicios.
* SELinux.
* NetworkManager.
* Entorno gráfico.
* `systemd-analyze`.

### **Fase 4 — Recuperación integral y práctica RHCSA**

* ISO de rescate.
* `chroot`.
* Regeneración de `initramfs`.
* Recuperación de GRUB.
* Script de auditoría.
* Laboratorio de aproximadamente 30 escenarios.
* Checklist profesional.
* 50 preguntas de repaso.
* Desafío final RHCSA.

El resultado completo será el archivo:

