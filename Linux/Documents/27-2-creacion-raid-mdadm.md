# 27.2 Creación de RAID con `mdadm`

`mdadm` es la herramienta principal para crear y administrar RAID por software en Linux.

---

# Objetivos

- Instalar `mdadm`.
- Preparar discos.
- Crear RAID 0, 1, 5, 6 y 10.
- Crear sistemas de archivos.
- Montar arreglos.
- Configurar persistencia.
- Integrar RAID con LVM.

---

# Instalar mdadm

Fedora/RHEL:

```bash
sudo dnf install mdadm
```

Debian/Ubuntu:

```bash
sudo apt install mdadm
```

Verificar:

```bash
mdadm --version
```

---

# Identificar discos

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
```

```bash
sudo fdisk -l
```

```bash
sudo blkid
```

Nunca ejecutes comandos destructivos sin confirmar el dispositivo.

---

# Limpiar firmas antiguas

Consultar:

```bash
sudo wipefs /dev/sdb
```

Eliminar firmas:

```bash
sudo wipefs -a /dev/sdb
```

Advertencia: este comando puede destruir acceso a datos existentes.

---

# Crear particiones RAID

Con `fdisk`:

```bash
sudo fdisk /dev/sdb
```

Crear partición:

```text
n
```

Cambiar tipo:

```text
t
```

Seleccionar:

```text
Linux RAID
```

Guardar:

```text
w
```

Repetir en los demás discos.

Actualizar:

```bash
sudo partprobe
```

---

# Crear RAID 0

Ejemplo con dos particiones:

```bash
sudo mdadm --create /dev/md0 \
  --level=0 \
  --raid-devices=2 \
  /dev/sdb1 /dev/sdc1
```

Verificar:

```bash
cat /proc/mdstat
```

```bash
sudo mdadm --detail /dev/md0
```

---

# Crear RAID 1

```bash
sudo mdadm --create /dev/md1 \
  --level=1 \
  --raid-devices=2 \
  /dev/sdd1 /dev/sde1
```

Con spare:

```bash
sudo mdadm --create /dev/md1 \
  --level=1 \
  --raid-devices=2 \
  --spare-devices=1 \
  /dev/sdd1 /dev/sde1 /dev/sdf1
```

---

# Crear RAID 5

```bash
sudo mdadm --create /dev/md5 \
  --level=5 \
  --raid-devices=3 \
  /dev/sdb1 /dev/sdc1 /dev/sdd1
```

Con chunk de 512 KiB:

```bash
sudo mdadm --create /dev/md5 \
  --level=5 \
  --raid-devices=3 \
  --chunk=512 \
  /dev/sdb1 /dev/sdc1 /dev/sdd1
```

---

# Crear RAID 6

```bash
sudo mdadm --create /dev/md6 \
  --level=6 \
  --raid-devices=4 \
  /dev/sdb1 /dev/sdc1 /dev/sdd1 /dev/sde1
```

---

# Crear RAID 10

```bash
sudo mdadm --create /dev/md10 \
  --level=10 \
  --raid-devices=4 \
  /dev/sdb1 /dev/sdc1 /dev/sdd1 /dev/sde1
```

---

# Crear arreglo degradado

Ejemplo RAID 1 con un miembro temporalmente ausente:

```bash
sudo mdadm --create /dev/md1 \
  --level=1 \
  --raid-devices=2 \
  /dev/sdb1 missing
```

Debe utilizarse solo cuando existe una razón operativa clara.

---

# Supervisar creación

```bash
watch -n 2 cat /proc/mdstat
```

Ejemplo:

```text
md5 : active raid5 sdd1[3] sdc1[1] sdb1[0]
      2095104 blocks super 1.2 level 5
      [3/3] [UUU]
      [====>................] resync = 25.0%
```

---

# Interpretar indicadores

```text
[UUU]
```

Todos activos.

```text
[U_U]
```

Un miembro ausente o fallado.

---

# Crear sistema de archivos

XFS:

```bash
sudo mkfs.xfs /dev/md0
```

ext4:

```bash
sudo mkfs.ext4 /dev/md0
```

---

# Crear punto de montaje

```bash
sudo mkdir -p /raid/datos
```

Montar:

```bash
sudo mount /dev/md0 /raid/datos
```

Verificar:

```bash
findmnt /raid/datos
df -Th /raid/datos
```

---

# Obtener UUID

```bash
sudo blkid /dev/md0
```

Agregar a `/etc/fstab`:

```fstab
UUID=UUID_OBTENIDO /raid/datos xfs defaults 0 0
```

Validar:

```bash
sudo findmnt --verify
sudo mount -a
```

---

# Guardar configuración de mdadm

Fedora/RHEL:

```bash
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm.conf
```

Debian/Ubuntu:

```bash
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
```

Verificar:

```bash
cat /etc/mdadm.conf
```

Evita duplicar entradas.

---

# Actualizar initramfs

Fedora/RHEL:

```bash
sudo dracut -f
```

Debian/Ubuntu:

```bash
sudo update-initramfs -u
```

Esto es especialmente importante cuando el RAID participa en el arranque.

---

# Examinar metadatos

```bash
sudo mdadm --examine /dev/sdb1
```

```bash
sudo mdadm --examine --scan
```

---

# Crear RAID sobre discos completos

También puede hacerse:

```bash
sudo mdadm --create /dev/md0 \
  --level=1 \
  --raid-devices=2 \
  /dev/sdb /dev/sdc
```

Sin embargo, usar particiones puede facilitar documentación y compatibilidad.

---

# RAID y LVM

Flujo:

```text
Discos
  ↓
RAID /dev/md0
  ↓
pvcreate
  ↓
vgcreate
  ↓
lvcreate
  ↓
mkfs
  ↓
mount
```

Ejemplo:

```bash
sudo pvcreate /dev/md0
sudo vgcreate vgraid /dev/md0
sudo lvcreate -L 100G -n lvdatos vgraid
sudo mkfs.xfs /dev/vgraid/lvdatos
sudo mkdir -p /datos
sudo mount /dev/vgraid/lvdatos /datos
```

---

# RAID y swap

Crear swap sobre RAID 1:

```bash
sudo mkswap /dev/md1
sudo swapon /dev/md1
```

Persistencia:

```fstab
UUID=UUID_SWAP none swap defaults 0 0
```

No se recomienda RAID 0 para swap crítica.

---

# RAID y LUKS

Flujo recomendado:

```text
RAID
  ↓
LUKS
  ↓
LVM
  ↓
Filesystem
```

Ejemplo conceptual:

```bash
sudo cryptsetup luksFormat /dev/md0
sudo cryptsetup open /dev/md0 raid_crypt
sudo pvcreate /dev/mapper/raid_crypt
```

---

# Crear bitmap interno

Durante creación:

```bash
sudo mdadm --create /dev/md1 \
  --level=1 \
  --raid-devices=2 \
  --bitmap=internal \
  /dev/sdb1 /dev/sdc1
```

Agregar después:

```bash
sudo mdadm --grow /dev/md1 --bitmap=internal
```

---

# Configurar nombre

```bash
sudo mdadm --create /dev/md/datos \
  --name=datos \
  --level=1 \
  --raid-devices=2 \
  /dev/sdb1 /dev/sdc1
```

Verificar:

```bash
ls -l /dev/md/
```

---

# Eliminar un arreglo de laboratorio

Desmontar:

```bash
sudo umount /raid/datos
```

Detener:

```bash
sudo mdadm --stop /dev/md0
```

Eliminar definición:

```bash
sudo mdadm --remove /dev/md0
```

Borrar superblocks:

```bash
sudo mdadm --zero-superblock /dev/sdb1 /dev/sdc1
```

Eliminar firmas:

```bash
sudo wipefs -a /dev/sdb1 /dev/sdc1
```

---

# Laboratorio: RAID 1 completo

```bash
sudo mdadm --create /dev/md1 \
  --level=1 \
  --raid-devices=2 \
  /dev/sdb1 /dev/sdc1

watch -n 2 cat /proc/mdstat

sudo mkfs.xfs /dev/md1
sudo mkdir -p /raid1
sudo mount /dev/md1 /raid1
sudo blkid /dev/md1
```

Agregar a `/etc/fstab`:

```fstab
UUID=UUID_OBTENIDO /raid1 xfs defaults 0 0
```

Guardar configuración:

```bash
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm.conf
sudo dracut -f
```

---

# Errores comunes

- Usar discos con datos existentes.
- Confundir `/dev/sdb` y `/dev/sdb1`.
- Crear RAID sobre discos montados.
- No guardar configuración.
- No actualizar initramfs.
- No validar `/etc/fstab`.
- Usar discos de tamaños muy diferentes.
- Crear RAID 0 para datos críticos.

---

# Resumen

Aprendiste a:

- Instalar mdadm.
- Preparar discos.
- Crear varios niveles RAID.
- Crear filesystem.
- Montar.
- Configurar persistencia.
- Integrar RAID con LVM, swap y LUKS.
