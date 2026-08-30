# 27.5 RAID avanzado, troubleshooting y laboratorios RHCSA

Esta sección reúne administración avanzada, diagnóstico y prácticas.

---

# Objetivos

- Diagnosticar fallos complejos.
- Integrar RAID con servicios.
- Automatizar monitoreo.
- Realizar laboratorios.
- Prepararse para tareas prácticas.

---

# Troubleshooting sistemático

## Paso 1: identificar arreglo

```bash
cat /proc/mdstat
```

## Paso 2: detalle

```bash
sudo mdadm --detail /dev/md0
```

## Paso 3: miembros

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS,SERIAL
```

## Paso 4: metadatos

```bash
sudo mdadm --examine /dev/sdb1
```

## Paso 5: logs

```bash
journalctl -k | grep -Ei "md0|raid|I/O error|failed"
```

## Paso 6: SMART

```bash
sudo smartctl -a /dev/sdb
```

## Paso 7: filesystem

```bash
findmnt /raid/datos
df -Th /raid/datos
```

---

# Error: no superblock

```text
No md superblock detected
```

Posibles causas:

- Dispositivo incorrecto.
- Superblock borrado.
- Partición equivocada.
- Arreglo creado sobre disco completo.
- Metadata dañada.

---

# Error: device busy

```text
mdadm: Cannot open /dev/sdb1: Device or resource busy
```

Revisar:

```bash
findmnt -S /dev/sdb1
lsblk
sudo lsof /dev/sdb1
```

---

# Error: not enough devices

```text
not enough to start the array
```

Puede requerir:

```bash
sudo mdadm --assemble --run /dev/md0 /dev/sdb1
```

Solo si el nivel permite operación degradada.

---

# Error: wrong UUID

Examinar:

```bash
sudo mdadm --examine /dev/sdb1 | grep -i uuid
```

Comparar con:

```bash
sudo mdadm --detail /dev/md0 | grep -i uuid
```

---

# Error: stale member

Un miembro con eventos antiguos no debe agregarse sin análisis.

Comparar:

```bash
sudo mdadm --examine /dev/sd[bcd]1 | grep -E "Events|Array UUID"
```

---

# Error: filesystem no monta

Verificar:

```bash
sudo blkid /dev/md0
sudo file -s /dev/md0
```

XFS:

```bash
sudo xfs_repair -n /dev/md0
```

ext4:

```bash
sudo e2fsck -fn /dev/md0
```

---

# Error: md0 cambia a md127

Puede ocurrir si falta configuración persistente.

Generar:

```bash
sudo mdadm --detail --scan
```

Guardar en:

```text
/etc/mdadm.conf
```

Actualizar initramfs:

```bash
sudo dracut -f
```

---

# Error: arreglo no inicia al arrancar

Revisar:

```bash
cat /etc/mdadm.conf
```

```bash
journalctl -b | grep -Ei "mdadm|raid|md0"
```

```bash
lsinitrd | grep mdadm
```

Regenerar:

```bash
sudo dracut -f
```

---

# Error: reconstrucción muy lenta

Revisar:

```bash
cat /proc/mdstat
iostat -xz 1
```

Ajustar temporalmente:

```bash
echo 50000 | sudo tee /proc/sys/dev/raid/speed_limit_min
```

No aumentar sin evaluar impacto.

---

# Error: mismatch_cnt elevado

Consultar:

```bash
cat /sys/block/md0/md/mismatch_cnt
```

Ejecutar check:

```bash
echo check | sudo tee /sys/block/md0/md/sync_action
```

Si persiste, investigar:

- Hardware.
- Cableado.
- Controladora.
- RAM.
- Corrupción.
- Errores de energía.

---

# RAID y systemd dependencies

Un servicio puede requerir el punto montado.

```ini
[Unit]
RequiresMountsFor=/raid/datos
After=local-fs.target
```

---

# Script de alerta

```bash
#!/bin/bash

set -euo pipefail

LOG="/var/log/raid-health.log"

{
    echo "===== $(date) ====="
    cat /proc/mdstat

    for md in /dev/md[0-9]*; do
        [ -b "$md" ] || continue
        mdadm --detail "$md"
    done
} >> "$LOG" 2>&1

if grep -q '_' /proc/mdstat; then
    echo "RAID degradado detectado" >&2
    exit 1
fi
```

---

# Timer systemd

Unidad:

```ini
[Unit]
Description=Verificación RAID

[Service]
Type=oneshot
ExecStart=/usr/local/sbin/check-raid.sh
```

Timer:

```ini
[Unit]
Description=Ejecutar verificación RAID cada 15 minutos

[Timer]
OnBootSec=5min
OnUnitActiveSec=15min
Persistent=true

[Install]
WantedBy=timers.target
```

Habilitar:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now check-raid.timer
```

---

# Laboratorio 1: RAID 0

```bash
sudo mdadm --create /dev/md0 \
  --level=0 \
  --raid-devices=2 \
  /dev/sdb1 /dev/sdc1

sudo mkfs.xfs /dev/md0
sudo mkdir -p /raid0
sudo mount /dev/md0 /raid0
```

Verificar:

```bash
cat /proc/mdstat
sudo mdadm --detail /dev/md0
df -Th /raid0
```

---

# Laboratorio 2: RAID 1

```bash
sudo mdadm --create /dev/md1 \
  --level=1 \
  --raid-devices=2 \
  /dev/sdd1 /dev/sde1
```

Crear archivo:

```bash
sudo mkfs.ext4 /dev/md1
sudo mkdir -p /raid1
sudo mount /dev/md1 /raid1
sudo touch /raid1/prueba.txt
```

---

# Laboratorio 3: simular fallo

```bash
sudo mdadm /dev/md1 --fail /dev/sdd1
sudo mdadm /dev/md1 --remove /dev/sdd1
```

Verificar:

```bash
cat /proc/mdstat
sudo mdadm --detail /dev/md1
ls -l /raid1/prueba.txt
```

---

# Laboratorio 4: reemplazo

```bash
sudo mdadm /dev/md1 --add /dev/sdf1
watch -n 2 cat /proc/mdstat
```

---

# Laboratorio 5: spare

```bash
sudo mdadm /dev/md1 --add /dev/sdg1
```

Verificar:

```bash
sudo mdadm --detail /dev/md1
```

---

# Laboratorio 6: RAID 5

```bash
sudo mdadm --create /dev/md5 \
  --level=5 \
  --raid-devices=3 \
  /dev/sdb1 /dev/sdc1 /dev/sdd1
```

---

# Laboratorio 7: scrubbing

```bash
echo check | sudo tee /sys/block/md5/md/sync_action
watch -n 2 cat /proc/mdstat
cat /sys/block/md5/md/mismatch_cnt
```

---

# Laboratorio 8: RAID + LVM

```bash
sudo pvcreate /dev/md5
sudo vgcreate vg_raid /dev/md5
sudo lvcreate -L 2G -n lv_data vg_raid
sudo mkfs.xfs /dev/vg_raid/lv_data
sudo mkdir -p /raidlvm
sudo mount /dev/vg_raid/lv_data /raidlvm
```

---

# Laboratorio 9: desmontar y reensamblar

```bash
sudo umount /raid1
sudo mdadm --stop /dev/md1
sudo mdadm --assemble /dev/md1 /dev/sde1 /dev/sdf1
sudo mount /dev/md1 /raid1
```

---

# Laboratorio 10: eliminar entorno

```bash
sudo umount /raid1
sudo mdadm --stop /dev/md1
sudo mdadm --remove /dev/md1
sudo mdadm --zero-superblock /dev/sde1 /dev/sdf1 /dev/sdg1
```

---

# Preguntas tipo examen

1. ¿Qué archivo muestra el estado de todos los arreglos?
2. ¿Cómo se marca un disco como fallado?
3. ¿Cómo se agrega un spare?
4. ¿Cómo se guarda la configuración?
5. ¿Qué comando regenera initramfs en RHEL?
6. ¿Qué nivel tolera dos fallos?
7. ¿Qué nivel ofrece mejor rendimiento sin redundancia?
8. ¿Qué nivel suele recomendarse para bases de datos?
9. ¿Cómo se ejecuta un scrub?
10. ¿Qué significa `[U_]`?

---

# Respuestas

1. `/proc/mdstat`.
2. `mdadm /dev/md0 --fail /dev/sdX1`.
3. `mdadm /dev/md0 --add /dev/sdX1`.
4. `mdadm --detail --scan`.
5. `dracut -f`.
6. RAID 6.
7. RAID 0.
8. RAID 10.
9. `echo check > /sys/block/md0/md/sync_action`.
10. Un miembro está ausente o fallado.

---

# Checklist operativo

Antes:

- Confirmar dispositivos.
- Confirmar respaldos.
- Confirmar nivel RAID.
- Confirmar ventana.
- Revisar SMART.
- Identificar seriales.

Durante:

- Monitorear `/proc/mdstat`.
- Revisar I/O.
- Revisar temperatura.
- Evitar reinicios.
- Registrar comandos.

Después:

- Confirmar `[UU...]`.
- Revisar `mdadm --detail`.
- Validar filesystem.
- Probar aplicación.
- Actualizar documentación.
- Verificar alertas.

---

# Tabla de comandos

| Comando | Función |
|---|---|
| `mdadm --create` | Crear arreglo |
| `mdadm --detail` | Mostrar detalle |
| `mdadm --examine` | Leer superblock |
| `mdadm --assemble` | Ensamblar |
| `mdadm --stop` | Detener |
| `mdadm --add` | Agregar miembro |
| `mdadm --fail` | Marcar fallado |
| `mdadm --remove` | Remover |
| `mdadm --grow` | Ampliar o cambiar |
| `mdadm --zero-superblock` | Borrar metadata |
| `cat /proc/mdstat` | Estado |
| `smartctl` | Salud de discos |
| `iostat` | Rendimiento |
| `fio` | Benchmark |
| `dracut -f` | Regenerar initramfs |

---

# Resumen general

En este capítulo aprendiste:

- Fundamentos de RAID.
- Diferencias entre niveles.
- Creación con mdadm.
- Persistencia.
- Monitoreo.
- Spares.
- Scrubbing.
- Recuperación.
- Reemplazo de discos.
- Integración con LVM, LUKS, swap y filesystem.
- Troubleshooting.
- Laboratorios prácticos.
