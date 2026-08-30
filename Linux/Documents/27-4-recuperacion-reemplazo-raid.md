# 27.4 Recuperación, reemplazo de discos y reconstrucción

La recuperación de RAID debe realizarse con extrema precaución.

---

# Objetivos

- Recuperar arreglos degradados.
- Reemplazar discos.
- Ensamblar manualmente.
- Identificar superblocks.
- Resolver diferencias de eventos.
- Recuperar tras reinicio.
- Trabajar con arreglos inactivos.

---

# Estado degradado

Consultar:

```bash
cat /proc/mdstat
```

```bash
sudo mdadm --detail /dev/md0
```

Ejemplo:

```text
State : clean, degraded
Active Devices : 1
Failed Devices : 1
```

---

# Identificar disco fallado

```bash
sudo mdadm --detail /dev/md0
```

Buscar:

```text
faulty
removed
spare rebuilding
```

También revisar kernel:

```bash
journalctl -k | grep -Ei "md0|sdb|I/O error|failed"
```

---

# Procedimiento de reemplazo

## Paso 1: marcar como fallado

```bash
sudo mdadm /dev/md0 --fail /dev/sdb1
```

## Paso 2: remover

```bash
sudo mdadm /dev/md0 --remove /dev/sdb1
```

## Paso 3: reemplazar físicamente

Identifica por:

```bash
lsblk -o NAME,SERIAL,SIZE,MODEL
```

## Paso 4: replicar partición

Con `sgdisk`:

```bash
sudo sgdisk --replicate=/dev/sdb /dev/sdc
sudo sgdisk --randomize-guids /dev/sdb
```

Advertencia: confirma origen y destino.

## Paso 5: releer tabla

```bash
sudo partprobe /dev/sdb
```

## Paso 6: agregar miembro

```bash
sudo mdadm /dev/md0 --add /dev/sdb1
```

## Paso 7: monitorear

```bash
watch -n 2 cat /proc/mdstat
```

---

# Reemplazar sin marcar manualmente

Si el disco desapareció, puede aparecer como removed.

Agregar el nuevo:

```bash
sudo mdadm /dev/md0 --add /dev/sdb1
```

---

# Ensamblado automático

```bash
sudo mdadm --assemble --scan
```

Verificar:

```bash
cat /proc/mdstat
```

---

# Ensamblado manual

```bash
sudo mdadm --assemble /dev/md0 /dev/sdb1 /dev/sdc1
```

---

# Ensamblar degradado

```bash
sudo mdadm --assemble --run /dev/md0 /dev/sdb1
```

Solo debe hacerse si se comprende el estado del arreglo.

---

# Forzar ensamblado

```bash
sudo mdadm --assemble --force /dev/md0 /dev/sdb1 /dev/sdc1
```

`--force` puede provocar pérdida o corrupción si se seleccionan miembros inconsistentes.

Debe utilizarse únicamente tras examinar los eventos.

---

# Comparar eventos

```bash
for d in /dev/sdb1 /dev/sdc1 /dev/sdd1; do
    echo "=== $d ==="
    sudo mdadm --examine "$d" | grep -E "Array UUID|Events|Device Role|State"
done
```

El miembro con mayor contador de eventos suele ser el más actualizado, pero debe analizarse el contexto.

---

# Arreglo inactivo

```text
md0 : inactive sdb1[0] sdc1[1]
```

Puede intentar:

```bash
sudo mdadm --stop /dev/md0
sudo mdadm --assemble --scan
```

---

# Detener arreglo

Primero desmontar:

```bash
sudo umount /raid/datos
```

Luego:

```bash
sudo mdadm --stop /dev/md0
```

---

# Reensamblar después de detener

```bash
sudo mdadm --assemble /dev/md0 /dev/sdb1 /dev/sdc1
```

Montar:

```bash
sudo mount /dev/md0 /raid/datos
```

---

# Recuperar configuración

Generar:

```bash
sudo mdadm --examine --scan
```

Guardar:

```bash
sudo mdadm --examine --scan | sudo tee /etc/mdadm.conf
```

Actualizar initramfs:

```bash
sudo dracut -f
```

---

# Superblock

Versiones comunes:

- 0.90.
- 1.0.
- 1.1.
- 1.2.

Consultar:

```bash
sudo mdadm --examine /dev/sdb1 | grep -i version
```

La versión 1.2 guarda metadatos cerca del inicio.

---

# Zero superblock

```bash
sudo mdadm --zero-superblock /dev/sdb1
```

Solo cuando se desea retirar completamente el miembro del RAID.

---

# Recuperación tras corrupción de metadata

Antes:

```bash
sudo mdadm --examine /dev/sdb1
```

No ejecutar `--create` sobre un arreglo existente sin conocimiento exacto de:

- Nivel.
- Orden de discos.
- Chunk.
- Metadata.
- Layout.
- Tamaño.

Un `--create --assume-clean` incorrecto puede destruir la estructura.

---

# Recreate con assume-clean

En recuperaciones avanzadas:

```bash
sudo mdadm --create /dev/md0 \
  --assume-clean \
  --level=5 \
  --raid-devices=3 \
  /dev/sdb1 /dev/sdc1 /dev/sdd1
```

Este procedimiento es de alto riesgo y no debe ejecutarse sin copia de seguridad de metadatos y validación.

---

# RAID 5 degradado

Con un disco fallado:

- El arreglo sigue funcionando.
- El rendimiento puede disminuir.
- No existe tolerancia adicional.
- Debe reemplazarse lo antes posible.

---

# RAID 6 degradado

Puede tolerar dos fallos.

Con un fallo:

- Sigue protegido frente a otro.

Con dos fallos:

- Sigue operativo, pero sin tolerancia adicional.

---

# RAID 10 degradado

La tolerancia depende de qué miembros fallen.

Puede sobrevivir a múltiples fallos si no pertenecen al mismo espejo.

---

# Rebuild

Durante rebuild:

- Aumenta I/O.
- Aumenta temperatura.
- Disminuye rendimiento.
- Puede revelar fallos latentes.
- Debe monitorearse SMART.

---

# Pausar reconstrucción

```bash
echo frozen | sudo tee /sys/block/md0/md/sync_action
```

Reanudar:

```bash
echo recover | sudo tee /sys/block/md0/md/sync_action
```

La compatibilidad puede variar.

---

# Reshape

Cambiar número de discos o nivel implica reshape.

Debe existir:

- Respaldo.
- Energía estable.
- Espacio de respaldo crítico.
- Monitoreo.
- Ventana controlada.

---

# Backup file durante grow

```bash
sudo mdadm --grow /dev/md5 \
  --raid-devices=4 \
  --backup-file=/root/md5-grow.backup
```

El backup file ayuda a proteger zonas críticas durante reshape.

---

# Sistema de archivos después de recuperación

Verificar:

```bash
sudo xfs_repair -n /dev/md0
```

o:

```bash
sudo e2fsck -fn /dev/md0
```

Debe estar desmontado para reparaciones reales.

---

# XFS

Comprobación:

```bash
sudo xfs_repair -n /dev/md0
```

Reparación:

```bash
sudo xfs_repair /dev/md0
```

---

# ext4

```bash
sudo e2fsck -f /dev/md0
```

---

# Recuperación con LVM sobre RAID

Después de ensamblar:

```bash
sudo pvscan
sudo vgscan
sudo vgchange -ay
```

Verificar:

```bash
sudo pvs
sudo vgs
sudo lvs
```

Montar:

```bash
sudo mount -a
```

---

# Recuperación con LUKS sobre RAID

```bash
sudo mdadm --assemble --scan
sudo cryptsetup open /dev/md0 raid_crypt
sudo vgchange -ay
sudo mount -a
```

---

# Emergency mode

Si el sistema entra en emergencia:

1. Examinar `/proc/mdstat`.
2. Ensamblar RAID.
3. Activar LUKS o LVM.
4. Corregir `/etc/mdadm.conf`.
5. Corregir `/etc/fstab`.
6. Regenerar initramfs.
7. Reiniciar.

---

# Logs

```bash
journalctl -b -k | grep -Ei "md|raid|I/O error|failed"
```

```bash
dmesg | grep -Ei "md|raid"
```

---

# Buenas prácticas

- No fuerces ensamblado sin examinar eventos.
- Identifica físicamente el disco correcto.
- Mantén respaldo antes de rebuild.
- Reemplaza por igual o mayor tamaño.
- Revisa SMART antes y durante reconstrucción.
- No reinicies innecesariamente durante reshape.
- Conserva copia de `mdadm.conf`.
- Actualiza initramfs.
- Documenta seriales.

---

# Resumen

Aprendiste procedimientos de recuperación, reemplazo, ensamblado y reconstrucción.
