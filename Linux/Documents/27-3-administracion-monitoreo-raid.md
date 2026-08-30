# 27.3 Administración y monitoreo de arreglos RAID

Una vez creado un arreglo RAID, debe monitorearse de forma continua.

---

# Objetivos

- Consultar estado.
- Interpretar `/proc/mdstat`.
- Usar `mdadm --detail`.
- Configurar alertas.
- Agregar y retirar discos.
- Administrar spares.
- Ejecutar scrubbing.
- Medir rendimiento.

---

# Consultar estado general

```bash
cat /proc/mdstat
```

Ejemplo saludable:

```text
md0 : active raid1 sdc1[1] sdb1[0]
      1047552 blocks super 1.2 [2/2] [UU]
```

Ejemplo degradado:

```text
md0 : active raid1 sdc1[1]
      1047552 blocks super 1.2 [2/1] [_U]
```

---

# Ver detalle

```bash
sudo mdadm --detail /dev/md0
```

Campos importantes:

- Raid Level.
- Array Size.
- Raid Devices.
- Active Devices.
- Working Devices.
- Failed Devices.
- Spare Devices.
- State.
- UUID.
- Events.

---

# Examinar miembros

```bash
sudo mdadm --examine /dev/sdb1
```

Comparar eventos:

```bash
sudo mdadm --examine /dev/sdb1 /dev/sdc1 | grep -E "Device Role|Array UUID|Events"
```

---

# Escanear arreglos

```bash
sudo mdadm --assemble --scan
```

```bash
sudo mdadm --detail --scan
```

---

# Agregar un spare

```bash
sudo mdadm /dev/md0 --add /dev/sdd1
```

Verificar:

```bash
sudo mdadm --detail /dev/md0
```

---

# Marcar miembro como fallado

```bash
sudo mdadm /dev/md0 --fail /dev/sdb1
```

Verificar:

```bash
cat /proc/mdstat
```

---

# Remover miembro fallado

```bash
sudo mdadm /dev/md0 --remove /dev/sdb1
```

Secuencia combinada:

```bash
sudo mdadm /dev/md0 --fail /dev/sdb1 --remove /dev/sdb1
```

---

# Reagregar disco

Después de preparar reemplazo:

```bash
sudo mdadm /dev/md0 --add /dev/sdb1
```

Monitorear:

```bash
watch -n 2 cat /proc/mdstat
```

---

# Cambiar un spare a activo

Normalmente mdadm lo gestiona automáticamente cuando falta un miembro.

Verificar:

```bash
sudo mdadm --detail /dev/md0
```

---

# Aumentar número de discos

Ejemplo RAID 5 de 3 a 4 discos:

```bash
sudo mdadm /dev/md5 --add /dev/sde1
sudo mdadm --grow /dev/md5 --raid-devices=4
```

Monitorear:

```bash
watch -n 2 cat /proc/mdstat
```

Después debe ampliarse:

- Sistema de archivos.
- PV de LVM, si aplica.
- LV, si aplica.

---

# Cambiar tamaño del arreglo

Si los miembros fueron ampliados:

```bash
sudo mdadm --grow /dev/md0 --size=max
```

Después:

```bash
sudo pvresize /dev/md0
```

o ampliar directamente el filesystem.

---

# Scrubbing

Scrubbing verifica consistencia.

Iniciar check:

```bash
echo check | sudo tee /sys/block/md0/md/sync_action
```

Monitorear:

```bash
cat /proc/mdstat
```

Consultar errores:

```bash
cat /sys/block/md0/md/mismatch_cnt
```

---

# Repair

```bash
echo repair | sudo tee /sys/block/md0/md/sync_action
```

Debe ejecutarse con precaución y después de analizar las inconsistencias.

---

# Detener scrubbing

```bash
echo idle | sudo tee /sys/block/md0/md/sync_action
```

---

# Velocidad de reconstrucción

Consultar:

```bash
cat /proc/sys/dev/raid/speed_limit_min
cat /proc/sys/dev/raid/speed_limit_max
```

Modificar temporalmente:

```bash
echo 50000 | sudo tee /proc/sys/dev/raid/speed_limit_min
echo 200000 | sudo tee /proc/sys/dev/raid/speed_limit_max
```

Persistencia:

```ini
dev.raid.speed_limit_min = 50000
dev.raid.speed_limit_max = 200000
```

en:

```text
/etc/sysctl.d/99-raid.conf
```

---

# Bitmap

Consultar:

```bash
sudo mdadm --detail /dev/md0 | grep -i bitmap
```

Agregar:

```bash
sudo mdadm --grow /dev/md0 --bitmap=internal
```

Eliminar:

```bash
sudo mdadm --grow /dev/md0 --bitmap=none
```

---

# Monitoreo con systemd

Consultar unidades:

```bash
systemctl list-units | grep -i md
```

En algunas distribuciones:

```bash
systemctl status mdmonitor
```

Habilitar:

```bash
sudo systemctl enable --now mdmonitor
```

El nombre puede variar.

---

# Configurar correo

En `/etc/mdadm.conf`:

```text
MAILADDR admin@example.com
```

Ejecutar monitoreo:

```bash
sudo mdadm --monitor --scan --daemonise
```

Para probar:

```bash
sudo mdadm --monitor --scan --test
```

Requiere un sistema de correo funcional.

---

# Script de monitoreo

```bash
#!/bin/bash

set -euo pipefail

STATUS=$(cat /proc/mdstat)

echo "=== RAID STATUS ==="
echo "$STATUS"

if echo "$STATUS" | grep -Eq '\[[U_]+\]' && echo "$STATUS" | grep -q '_'; then
    echo "ALERTA: RAID degradado"
    exit 1
fi

for md in /dev/md*; do
    [ -b "$md" ] || continue
    mdadm --detail "$md"
done
```

---

# Monitoreo con cron

Ejemplo:

```cron
*/10 * * * * /usr/local/sbin/check_raid.sh
```

Es preferible integrar alertas con:

- Prometheus.
- Nagios.
- Zabbix.
- Grafana.
- Icinga.
- systemd timers.

---

# Benchmark con dd

Escritura:

```bash
dd if=/dev/zero of=/raid/datos/testfile bs=1G count=1 oflag=direct status=progress
```

Lectura:

```bash
dd if=/raid/datos/testfile of=/dev/null bs=1G iflag=direct status=progress
```

Eliminar:

```bash
rm /raid/datos/testfile
```

`dd` ofrece una prueba simple, no un benchmark completo.

---

# Benchmark con fio

Instalar:

```bash
sudo dnf install fio
```

Prueba secuencial:

```bash
fio --name=seqwrite \
    --filename=/raid/datos/fio.test \
    --size=4G \
    --rw=write \
    --bs=1M \
    --direct=1
```

Lectura aleatoria:

```bash
fio --name=randread \
    --filename=/raid/datos/fio.test \
    --size=4G \
    --rw=randread \
    --bs=4k \
    --iodepth=32 \
    --direct=1
```

---

# Monitoreo con iostat

```bash
iostat -xz 1
```

Campos importantes:

- `%util`.
- `await`.
- `r/s`.
- `w/s`.
- `rkB/s`.
- `wkB/s`.

---

# SMART

RAID no sustituye monitoreo SMART.

```bash
sudo smartctl -a /dev/sdb
```

Instalar:

```bash
sudo dnf install smartmontools
```

Habilitar:

```bash
sudo systemctl enable --now smartd
```

---

# Temperatura

```bash
sudo smartctl -A /dev/sdb | grep -i temperature
```

También:

```bash
sudo nvme smart-log /dev/nvme0
```

para NVMe.

---

# Capacidad y uso

```bash
df -Th
```

```bash
lsblk
```

```bash
sudo mdadm --detail /dev/md0
```

---

# Buenas prácticas

- Monitorea estado y SMART.
- Ejecuta scrub periódico.
- Verifica alertas.
- Mantén un spare compatible.
- Documenta el reemplazo.
- No retires discos sin marcarlos y removerlos.
- Evita benchmarks en horas críticas.
- Supervisa temperatura y latencia.
- Comprueba reconstrucción hasta finalizar.

---

# Resumen

Aprendiste a administrar miembros, spares, scrubbing, alertas y rendimiento.
