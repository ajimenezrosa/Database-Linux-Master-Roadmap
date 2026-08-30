# **67. Laboratorio de Arranque y Recuperación**

El archivo correspondiente será:

```text
67-laboratorio-arranque-recuperacion.md
```

Este capítulo será el laboratorio integrador del **Módulo 9: Arranque y Recuperación**. Su objetivo será aplicar, en escenarios controlados, todo lo estudiado en las páginas anteriores:

```text
59. Proceso de arranque Linux
60. GRUB2 y cargador de arranque
61. Targets y modos systemd
62. Parámetros del kernel durante el arranque
63. Recuperación de contraseña root
64. Modos Rescue y Emergency
65. Initramfs y Dracut
66. Diagnóstico de problemas de arranque
67. Laboratorio de Arranque y Recuperación
```

---

# Estructura propuesta

## 1. Objetivos del laboratorio

Al finalizar el capítulo, el estudiante podrá:

* Diagnosticar problemas en cada etapa del arranque.
* Trabajar con GRUB2 y entradas BLS.
* Seleccionar kernels alternativos.
* Modificar temporalmente parámetros del kernel.
* Iniciar en `rescue.target`.
* Iniciar en `emergency.target`.
* Recuperar la contraseña de `root`.
* Corregir problemas en `/etc/fstab`.
* Activar volúmenes LVM manualmente.
* Diagnosticar fallos de `initramfs`.
* Regenerar imágenes mediante Dracut.
* Recuperar un sistema desde una ISO.
* Corregir errores SELinux relacionados con la recuperación.
* Analizar el Journal del arranque actual y anterior.
* Documentar profesionalmente una intervención.

---

# 2. Advertencias del laboratorio

El capítulo comenzará con advertencias claras:

```text
No ejecutar estas prácticas en servidores productivos.
```

Se recomendará:

* Máquina virtual.
* Instantánea previa.
* Acceso directo a consola.
* Medio ISO disponible.
* Dos kernels instalados.
* Uno o dos discos adicionales.
* LVM configurado.
* SELinux en Enforcing.
* Acceso administrativo.
* Copia de `/etc/fstab`.
* Copia de `/boot`.

---

# 3. Topología del laboratorio

```text
┌─────────────────────────────────────────────┐
│              Máquina virtual RHEL           │
├─────────────────────────────────────────────┤
│ Disco 1                                     │
│ ├── EFI o BIOS Boot                         │
│ ├── /boot                                   │
│ └── LVM                                     │
│     ├── root                                │
│     ├── home                                │
│     ├── var                                 │
│     └── swap                                │
│                                             │
│ Disco 2                                     │
│ └── XFS montado en /datos                   │
│                                             │
│ Disco 3                                     │
│ └── ext4 montado en /respaldo               │
│                                             │
│ ISO de instalación conectada                │
└─────────────────────────────────────────────┘
```

---

# 4. Preparación del entorno

Comandos iniciales:

```bash
hostnamectl
```

```bash
cat /etc/os-release
```

```bash
uname -r
```

```bash
lsblk -f
```

```bash
findmnt
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
systemctl get-default
```

```bash
getenforce
```

---

# 5. Creación de evidencia inicial

Se creará:

```text
/root/laboratorio-arranque
```

Comando:

```bash
mkdir -p /root/laboratorio-arranque
```

Inventario:

```bash
{
    date
    hostnamectl
    cat /etc/os-release
    uname -a
    cat /proc/cmdline
    systemctl get-default
    lsblk -f
    findmnt
    pvs
    vgs
    lvs
    getenforce
} > /root/laboratorio-arranque/estado-inicial.txt
```

---

# 6. Laboratorio 1 — Reconocer las etapas del arranque

El estudiante deberá documentar:

```text
Firmware
   │
   ▼
GRUB2
   │
   ▼
Kernel
   │
   ▼
initramfs
   │
   ▼
Sistema raíz
   │
   ▼
systemd
   │
   ▼
Target predeterminado
   │
   ▼
Servicios
```

Comandos:

```bash
journalctl -b
```

```bash
journalctl -k -b
```

```bash
systemd-analyze
```

```bash
systemd-analyze critical-chain
```

---

# 7. Laboratorio 2 — Analizar GRUB2

Comandos:

```bash
grubby --info=ALL
```

```bash
grubby --default-kernel
```

```bash
grub2-editenv list
```

```bash
ls -l /boot/loader/entries
```

```bash
ls -lh /boot/vmlinuz-* /boot/initramfs-*
```

El estudiante deberá identificar:

* Kernel predeterminado.
* Kernel actualmente activo.
* Entradas disponibles.
* Imagen `initramfs` asociada.
* Parámetros usados por cada entrada.

---

# 8. Laboratorio 3 — Iniciar con un kernel anterior

Procedimiento:

1. Reiniciar.
2. Mostrar el menú GRUB.
3. Seleccionar un kernel anterior.
4. Iniciar.
5. Verificar:

```bash
uname -r
```

6. Comparar con:

```bash
grubby --default-kernel
```

7. Volver al kernel más reciente.

---

# 9. Laboratorio 4 — Modificación temporal de parámetros

Desde GRUB:

1. Seleccionar entrada.
2. Presionar `e`.
3. Localizar la línea del kernel.
4. Eliminar temporalmente:

```text
quiet
```

```text
rhgb
```

5. Agregar:

```text
systemd.log_level=debug
```

6. Iniciar con:

```text
Ctrl+x
```

Después:

```bash
cat /proc/cmdline
```

---

# 10. Laboratorio 5 — Iniciar en `multi-user.target`

Agregar temporalmente desde GRUB:

```text
systemd.unit=multi-user.target
```

Después verificar:

```bash
systemctl is-active multi-user.target
```

```bash
systemctl list-units \
--type=target \
--state=active
```

---

# 11. Laboratorio 6 — Iniciar en Rescue

Agregar:

```text
systemd.unit=rescue.target
```

Verificar:

```bash
systemctl is-active rescue.target
```

```bash
systemctl is-system-running
```

```bash
systemctl list-units \
--type=service \
--state=running
```

Salir:

```bash
systemctl default
```

---

# 12. Laboratorio 7 — Iniciar en Emergency

Agregar desde GRUB:

```text
systemd.unit=emergency.target
```

Verificar:

```bash
systemctl is-active emergency.target
```

```bash
findmnt /
```

```bash
systemctl is-system-running
```

Si la raíz está en solo lectura:

```bash
mount -o remount,rw /
```

Continuar:

```bash
systemctl default
```

---

# 13. Laboratorio 8 — Recuperar contraseña root

Procedimiento mediante:

```text
rd.break
```

Flujo:

```text
GRUB
  │
  ▼
rd.break
  │
  ▼
/sysroot
  │
  ▼
Remontar rw
  │
  ▼
chroot
  │
  ▼
passwd root
  │
  ▼
autorelabel
```

Comandos:

```bash
mount -o remount,rw /sysroot
```

```bash
chroot /sysroot
```

```bash
passwd root
```

```bash
touch /.autorelabel
```

```bash
exit
```

```bash
exit
```

---

# 14. Laboratorio 9 — Crear un montaje XFS

Sobre un disco de práctica:

```bash
mkfs.xfs /dev/sdb1
```

```bash
mkdir -p /datos
```

```bash
blkid /dev/sdb1
```

Agregar a `/etc/fstab`:

```text
UUID=UUID_REAL /datos xfs defaults 0 0
```

Validar:

```bash
findmnt --verify --verbose
```

```bash
mount -av
```

---

# 15. Laboratorio 10 — Provocar un UUID incorrecto

Respaldar:

```bash
cp -a \
/etc/fstab \
/root/laboratorio-arranque/fstab.correcto
```

Modificar el UUID:

```text
UUID=UUID-INEXISTENTE /datos xfs defaults 0 0
```

Validar antes de reiniciar:

```bash
findmnt --verify --verbose
```

```bash
mount -av
```

Posteriormente reiniciar para observar la entrada automática en Emergency.

---

# 16. Laboratorio 11 — Recuperar `/etc/fstab`

Desde Emergency:

```bash
systemctl --failed
```

```bash
journalctl -xb
```

```bash
lsblk -f
```

```bash
blkid
```

```bash
findmnt /
```

```bash
mount -o remount,rw /
```

Corregir:

```bash
vi /etc/fstab
```

Validar:

```bash
findmnt --verify --verbose
```

```bash
mount -av
```

Continuar:

```bash
systemctl daemon-reload
```

```bash
systemctl reset-failed
```

```bash
systemctl default
```

---

# 17. Laboratorio 12 — Disco opcional con `nofail`

Modificar la entrada:

```text
UUID=UUID_REAL /datos xfs defaults,nofail,x-systemd.device-timeout=10s 0 0
```

El estudiante deberá comprobar que la ausencia del dispositivo no bloquea el arranque.

---

# 18. Laboratorio 13 — Diagnóstico LVM

Comandos:

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
pvscan
```

```bash
vgscan
```

```bash
lvscan
```

---

# 19. Laboratorio 14 — Desactivar y activar un LV

En un volumen no crítico y desmontado:

```bash
umount /datos-lvm
```

```bash
lvchange -an /dev/vglab/lvdatos
```

Confirmar:

```bash
lvs
```

Activar:

```bash
lvchange -ay /dev/vglab/lvdatos
```

Montar:

```bash
mount /datos-lvm
```

---

# 20. Laboratorio 15 — Activar todos los grupos

Desde Rescue o Emergency:

```bash
vgchange -ay
```

Verificar:

```bash
ls -l /dev/mapper
```

```bash
lvs
```

---

# 21. Laboratorio 16 — Revisar XFS

Desmontar:

```bash
umount /datos
```

Comprobar:

```bash
xfs_repair -n /dev/sdb1
```

Montar nuevamente:

```bash
mount /datos
```

---

# 22. Laboratorio 17 — Revisar ext4

Crear sistema de práctica:

```bash
mkfs.ext4 /dev/sdc1
```

Comprobar desmontado:

```bash
e2fsck -n /dev/sdc1
```

Forzar revisión:

```bash
e2fsck -f /dev/sdc1
```

---

# 23. Laboratorio 18 — Inspeccionar initramfs

Comandos:

```bash
uname -r
```

```bash
ls -lh \
/boot/initramfs-$(uname -r).img
```

```bash
lsinitrd \
/boot/initramfs-$(uname -r).img
```

Buscar módulos:

```bash
lsinitrd \
/boot/initramfs-$(uname -r).img \
| grep -E \
'lvm|xfs|dm-mod'
```

---

# 24. Laboratorio 19 — Regenerar initramfs

Respaldar:

```bash
cp -a \
/boot/initramfs-$(uname -r).img \
/boot/initramfs-$(uname -r).img.bak
```

Regenerar:

```bash
dracut -f -v \
/boot/initramfs-$(uname -r).img \
$(uname -r)
```

Validar:

```bash
lsinitrd \
/boot/initramfs-$(uname -r).img
```

---

# 25. Laboratorio 20 — Crear configuración Dracut

Crear:

```bash
vi /etc/dracut.conf.d/rhcsa-lab.conf
```

Ejemplo:

```text
add_dracutmodules+=" lvm "
add_drivers+=" dm_mod "
```

Regenerar:

```bash
dracut -f -v
```

---

# 26. Laboratorio 21 — Diagnosticar Dracut shell

Agregar desde GRUB:

```text
rd.break
```

o provocar un escenario controlado donde no se encuentre la raíz.

Comandos disponibles:

```bash
cat /proc/cmdline
```

```bash
lsblk
```

```bash
blkid
```

```bash
lvm lvs
```

```bash
lvm vgchange -ay
```

```bash
cat /run/initramfs/rdsosreport.txt
```

---

# 27. Laboratorio 22 — Analizar `rdsosreport.txt`

```bash
less \
/run/initramfs/rdsosreport.txt
```

Buscar:

```bash
grep -Ei \
'error|warning|timeout|failed|missing' \
/run/initramfs/rdsosreport.txt
```

---

# 28. Laboratorio 23 — Crear servicio defectuoso

Archivo:

```text
/etc/systemd/system/lab-arranque.service
```

Contenido:

```ini
[Unit]
Description=Servicio defectuoso del laboratorio
After=network.target

[Service]
Type=simple
ExecStart=/ruta/inexistente/programa
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

---

# 29. Laboratorio 24 — Diagnosticar el servicio

```bash
systemd-analyze verify \
/etc/systemd/system/lab-arranque.service
```

```bash
systemctl daemon-reload
```

```bash
systemctl enable \
lab-arranque.service
```

```bash
systemctl start \
lab-arranque.service
```

Diagnóstico:

```bash
systemctl status \
lab-arranque.service
```

```bash
journalctl \
-u lab-arranque.service \
-b
```

---

# 30. Laboratorio 25 — Deshabilitar y enmascarar

```bash
systemctl disable \
lab-arranque.service
```

```bash
systemctl mask \
lab-arranque.service
```

Probar:

```bash
systemctl start \
lab-arranque.service
```

Desenmascarar:

```bash
systemctl unmask \
lab-arranque.service
```

---

# 31. Laboratorio 26 — Reparar la unidad

Cambiar:

```ini
ExecStart=/usr/bin/sleep 300
```

Recargar:

```bash
systemctl daemon-reload
```

Validar:

```bash
systemd-analyze verify \
/etc/systemd/system/lab-arranque.service
```

Iniciar:

```bash
systemctl start \
lab-arranque.service
```

---

# 32. Laboratorio 27 — Problema SELinux controlado

Crear:

```bash
mkdir -p /srv/rhcsa-web
```

Consultar:

```bash
ls -Zd /srv/rhcsa-web
```

Definir contexto:

```bash
semanage fcontext \
-a \
-t httpd_sys_content_t \
'/srv/rhcsa-web(/.*)?'
```

Aplicar:

```bash
restorecon -Rv \
/srv/rhcsa-web
```

Verificar:

```bash
ls -Zd /srv/rhcsa-web
```

---

# 33. Laboratorio 28 — Target predeterminado incorrecto

Consultar:

```bash
systemctl get-default
```

Cambiar temporalmente para practicar:

```bash
systemctl set-default \
rescue.target
```

Verificar:

```bash
readlink -f \
/etc/systemd/system/default.target
```

Restaurar:

```bash
systemctl set-default \
multi-user.target
```

---

# 34. Laboratorio 29 — Analizar el tiempo de arranque

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
systemd-analyze plot \
> /root/laboratorio-arranque/arranque.svg
```

---

# 35. Laboratorio 30 — Analizar arranques anteriores

```bash
journalctl --list-boots
```

```bash
journalctl -b -1
```

```bash
journalctl -b -1 -p err
```

```bash
journalctl -k -b -1
```

---

# 36. Laboratorio 31 — Simular `/var` lleno

Se realizará con un volumen o sistema de archivos de laboratorio, nunca sobre `/var` real.

Ejemplo:

```bash
mkdir -p /mnt/var-lab
```

Crear archivo controlado:

```bash
fallocate -l 1G \
/mnt/var-lab/archivo-grande
```

Diagnosticar:

```bash
df -hT /mnt/var-lab
```

```bash
df -i /mnt/var-lab
```

```bash
du -xhd1 /mnt/var-lab
```

Eliminar:

```bash
rm -f \
/mnt/var-lab/archivo-grande
```

---

# 37. Laboratorio 32 — Archivos eliminados abiertos

Crear archivo:

```bash
dd if=/dev/zero \
of=/tmp/archivo-abierto.log \
bs=1M \
count=100
```

Mantenerlo abierto con un proceso de laboratorio y eliminarlo.

Detectar:

```bash
lsof +L1
```

El estudiante deberá explicar por qué el espacio no se libera hasta cerrar el descriptor.

---

# 38. Laboratorio 33 — Verificar paquetes críticos

```bash
rpm -V \
systemd
```

```bash
rpm -V \
grub2-tools
```

```bash
rpm -V \
dracut
```

```bash
rpm -V \
kernel-core
```

---

# 39. Laboratorio 34 — Validar configuraciones críticas

```bash
visudo -c
```

```bash
sshd -t
```

```bash
findmnt --verify
```

```bash
systemd-analyze verify \
/etc/systemd/system/*.service
```

---

# 40. Laboratorio 35 — Recuperación desde ISO

Procedimiento general:

1. Arrancar desde la ISO.
2. Elegir Troubleshooting.
3. Elegir Rescue.
4. Detectar el sistema instalado.
5. Montarlo en:

```text
/mnt/sysroot
```

6. Entrar:

```bash
chroot /mnt/sysroot
```

7. Verificar:

```bash
lsblk -f
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

8. Corregir el problema.
9. Regenerar `initramfs`.
10. Revisar GRUB.
11. Salir.
12. Reiniciar.

---

# 41. Laboratorio 36 — Regenerar GRUB desde chroot

Para BIOS:

```bash
grub2-mkconfig \
-o /boot/grub2/grub.cfg
```

Para UEFI, se explicará la relación con:

```text
/boot/efi
```

y las entradas BLS, evitando sobrescribir rutas incorrectas sin identificar primero la plataforma.

---

# 42. Laboratorio 37 — Regenerar initramfs desde ISO

Dentro del `chroot`:

```bash
ls -lh \
/boot/vmlinuz-* \
/boot/initramfs-*
```

```bash
dracut -f -v
```

Para un kernel concreto:

```bash
dracut -f -v \
/boot/initramfs-VERSION.img \
VERSION
```

---

# 43. Laboratorio 38 — Auditoría integral

El estudiante ejecutará el script desarrollado en la página 64:

```bash
/usr/local/sbin/auditoria-rescue-emergency.sh \
/root/laboratorio-arranque/auditoria
```

Después analizará:

```bash
grep -Ei \
'error|failed|warning|timeout|denied|corrupt' \
/root/laboratorio-arranque/auditoria/*.log
```

---

# 44. Laboratorio 39 — Comprobación final

```bash
systemctl is-system-running
```

```bash
systemctl --failed
```

```bash
systemctl get-default
```

```bash
findmnt --verify
```

```bash
mount -av
```

```bash
lsblk -f
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
getenforce
```

```bash
journalctl -b -p err
```

---

# 45. Laboratorio 40 — Informe técnico

El estudiante deberá elaborar:

```text
/root/laboratorio-arranque/informe-final.md
```

Con esta estructura:

```markdown
# Informe final de arranque y recuperación

## Información del sistema

## Topología

## Estado inicial

## Problemas simulados

## Evidencia recopilada

## Diagnóstico

## Causa raíz

## Correcciones realizadas

## Archivos modificados

## Validaciones

## Estado final

## Riesgos identificados

## Recomendaciones preventivas

## Conclusiones
```

---

# Casos de examen RHCSA

El capítulo incluirá ejercicios directos como:

1. Restablecer la contraseña de `root`.
2. Corregir un UUID incorrecto.
3. Hacer opcional un montaje.
4. Iniciar en Rescue.
5. Iniciar en Emergency.
6. Cambiar el target predeterminado.
7. Seleccionar un kernel anterior.
8. Regenerar `initramfs`.
9. Activar un VG.
10. Montar un LV.
11. Diagnosticar una unidad fallida.
12. Corregir un contexto SELinux.
13. Validar `/etc/fstab`.
14. Recuperar desde ISO.
15. Interpretar `journalctl -xb`.

---

# Script de validación del laboratorio

El capítulo incluirá un script que verifique:

* Estado de `systemd`.
* Target predeterminado.
* Unidades fallidas.
* Validez de `/etc/fstab`.
* Montajes.
* Espacio.
* Inodos.
* Imágenes de kernel.
* Imágenes `initramfs`.
* Entradas BLS.
* LVM.
* SELinux.
* Estado del Journal.

Archivo:

```text
/usr/local/sbin/validar-laboratorio-arranque.sh
```

---

# Lista de comprobación del estudiante

```text
[ ] Identifiqué las etapas del arranque
[ ] Consulté la configuración de GRUB2
[ ] Inicié con un kernel anterior
[ ] Modifiqué parámetros temporalmente
[ ] Inicié en multi-user.target
[ ] Inicié en rescue.target
[ ] Inicié en emergency.target
[ ] Recuperé la contraseña de root
[ ] Corregí un UUID inválido
[ ] Validé /etc/fstab
[ ] Utilicé nofail
[ ] Activé volúmenes LVM
[ ] Revisé un sistema XFS
[ ] Revisé un sistema ext4
[ ] Inspeccioné initramfs
[ ] Regeneré initramfs
[ ] Analicé rdsosreport.txt
[ ] Diagnostiqué un servicio defectuoso
[ ] Utilicé disable y mask
[ ] Corregí contextos SELinux
[ ] Restauré el target predeterminado
[ ] Analicé el rendimiento del arranque
[ ] Consulté un arranque anterior
[ ] Utilicé una ISO de rescate
[ ] Trabajé dentro de chroot
[ ] Regeneré GRUB
[ ] Ejecuté la auditoría integral
[ ] Verifiqué el estado final
[ ] Elaboré un informe técnico
```

---

# Preguntas de repaso

El capítulo incluirá aproximadamente 50 preguntas, entre ellas:

1. ¿Qué etapa ocurre después de GRUB2?
2. ¿Qué función cumple `initramfs`?
3. ¿Cuál es la diferencia entre Rescue y Emergency?
4. ¿Qué hace `rd.break`?
5. ¿Dónde se monta la raíz real durante `rd.break`?
6. ¿Cómo se remonta `/sysroot` en escritura?
7. ¿Por qué debe crearse `/.autorelabel`?
8. ¿Cómo se valida `/etc/fstab`?
9. ¿Cómo se prueban sus montajes?
10. ¿Qué hace `nofail`?
11. ¿Cómo se activa LVM?
12. ¿Qué comando muestra el kernel actual?
13. ¿Cómo se selecciona un kernel anterior?
14. ¿Cómo se inspecciona un `initramfs`?
15. ¿Cómo se regenera?
16. ¿Qué contiene `rdsosreport.txt`?
17. ¿Qué herramienta se usa para XFS?
18. ¿Qué herramienta se usa para ext4?
19. ¿Por qué no se repara un sistema montado?
20. ¿Cómo se consulta el arranque anterior?
21. ¿Cómo se identifican unidades fallidas?
22. ¿Qué diferencia existe entre `disable` y `mask`?
23. ¿Cómo se restaura un contexto SELinux?
24. ¿Qué comando muestra la cadena crítica?
25. ¿Cuándo se necesita una ISO de rescate?

---

# Desafío final integrador

El escenario final combinará varios fallos:

```text
1. El target predeterminado fue cambiado a Rescue.
2. /datos tiene un UUID incorrecto.
3. El volumen /var está inactivo.
4. El initramfs del kernel más reciente está incompleto.
5. El kernel más reciente no arranca.
6. Existe un kernel anterior funcional.
7. Un servicio personalizado está en bucle.
8. SELinux bloquea el acceso a /srv/reportes.
9. El sistema entra en Emergency.
10. Debe recuperarse sin deshabilitar SELinux.
```

El estudiante deberá:

* Arrancar con el kernel anterior.
* Acceder a Emergency.
* Activar LVM.
* Corregir `/etc/fstab`.
* Hacer opcional `/datos`, si corresponde.
* Corregir el target predeterminado.
* Detener y deshabilitar el servicio defectuoso.
* Corregir el contexto SELinux.
* Regenerar el `initramfs`.
* Verificarlo con `lsinitrd`.
* Reiniciar con el kernel nuevo.
* Comprobar que no existan unidades fallidas.
* Entregar evidencia e informe técnico.

---

# División recomendada

Por su extensión, el capítulo puede desarrollarse en cuatro fases:

## **Fase 1 — Preparación y fundamentos**

* Objetivos.
* Advertencias.
* Topología.
* Inventario.
* Evidencia inicial.
* GRUB2.
* Kernels.
* Parámetros temporales.
* Targets.

## **Fase 2 — Recuperación y almacenamiento**

* Rescue.
* Emergency.
* `rd.break`.
* Contraseña root.
* `/etc/fstab`.
* Montajes.
* XFS.
* ext4.
* LVM.

## **Fase 3 — Initramfs, Dracut y servicios**

* Inspección de `initramfs`.
* Regeneración.
* Dracut shell.
* `rdsosreport.txt`.
* Servicios defectuosos.
* SELinux.
* Rendimiento del arranque.

## **Fase 4 — ISO, auditoría y evaluación final**

* Recuperación desde ISO.
* `chroot`.
* GRUB2.
* Script de validación.
* Laboratorio integral.
* Checklist.
* Preguntas.
* Desafío final.
* Informe técnico.

