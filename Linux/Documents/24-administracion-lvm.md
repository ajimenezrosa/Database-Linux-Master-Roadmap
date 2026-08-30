# 24. Administración de LVM (Logical Volume Manager)

El **Logical Volume Manager (LVM)** es una tecnología de administración de almacenamiento que permite abstraer los discos físicos y administrarlos como un conjunto flexible de recursos.

A diferencia del particionado tradicional, donde una partición tiene un tamaño fijo, LVM permite ampliar, reducir, mover y reorganizar el almacenamiento con mucha mayor facilidad, incluso mientras el sistema está en funcionamiento en muchos casos.

En servidores Linux empresariales, LVM es ampliamente utilizado porque facilita la administración del crecimiento del almacenamiento, la creación de snapshots y la migración de datos.

En distribuciones como **Red Hat Enterprise Linux (RHEL)**, **Rocky Linux**, **AlmaLinux** y **Fedora**, LVM es el esquema predeterminado de instalación.

---

# Objetivos

Al finalizar este capítulo serás capaz de:

* Comprender la arquitectura de LVM.
* Identificar Physical Volumes (PV).
* Crear Volume Groups (VG).
* Crear Logical Volumes (LV).
* Expandir y reducir volúmenes.
* Crear snapshots.
* Consultar información de LVM.
* Reparar problemas comunes.
* Integrar LVM con `/etc/fstab`.
* Aplicar buenas prácticas de administración.

---

# ¿Qué es LVM?

LVM agrega una capa lógica entre los discos físicos y los sistemas de archivos.

En lugar de depender directamente de particiones, Linux administra:

```text
Disco
      ↓
Physical Volume (PV)
      ↓
Volume Group (VG)
      ↓
Logical Volume (LV)
      ↓
Sistema de archivos
      ↓
Punto de montaje
```

---

# Arquitectura de LVM

```text
             +----------------------+
             |   Logical Volume     |
             |      (LV)            |
             +----------+-----------+
                        |
             +----------+-----------+
             |     Volume Group     |
             |        (VG)          |
             +----+------------+----+
                  |            |
          +-------+            +--------+
          |                            |
   +------+-----+               +------+-----+
   | Physical   |               | Physical   |
   | Volume PV1 |               | Volume PV2 |
   +------------+               +------------+
          |                            |
      /dev/sdb1                    /dev/sdc1
```

---

# Componentes principales

| Componente | Función         |
| ---------- | --------------- |
| PV         | Physical Volume |
| VG         | Volume Group    |
| LV         | Logical Volume  |
| PE         | Physical Extent |
| LE         | Logical Extent  |

---

# Flujo completo

```text
Disco físico
      ↓
Partición tipo LVM
      ↓
pvcreate
      ↓
Volume Group
      ↓
vgcreate
      ↓
Logical Volume
      ↓
lvcreate
      ↓
mkfs
      ↓
mount
```

---

# Terminología

## Physical Volume (PV)

Es un disco o partición preparada para LVM.

Ejemplos:

```text
/dev/sdb1
/dev/sdc1
/dev/nvme0n1p2
```

---

## Volume Group (VG)

Es un grupo compuesto por uno o más Physical Volumes.

Ejemplo:

```text
VG_DATOS
```

Puede estar formado por:

```text
/dev/sdb1
/dev/sdc1
/dev/sdd1
```

---

## Logical Volume (LV)

Es el volumen que utilizará el sistema operativo.

Ejemplo:

```text
/dev/VG_DATOS/LV_BACKUP
```

---

## Physical Extents

LVM divide cada PV en pequeños bloques llamados:

```text
Physical Extents
```

Generalmente de:

```text
4 MB
```

aunque el tamaño puede variar.

---

## Logical Extents

Los Logical Volumes están formados por:

```text
Logical Extents
```

Los LE se asignan sobre los PE disponibles.

---

# Verificar si LVM está instalado

```bash
rpm -q lvm2
```

En Debian:

```bash
dpkg -l | grep lvm2
```

---

# Servicio LVM

```bash
systemctl status lvm2-monitor
```

Habilitar:

```bash
sudo systemctl enable --now lvm2-monitor
```

---

# Identificar discos

```bash
lsblk
```

Más información:

```bash
lsblk -f
```

---

# Ver discos disponibles

```bash
sudo fdisk -l
```

---

# Crear una partición LVM

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

En GPT:

```text
Linux LVM
```

Guardar:

```text
w
```

Actualizar:

```bash
sudo partprobe
```

---

# Crear un Physical Volume

```bash
sudo pvcreate /dev/sdb1
```

Ejemplo:

```text
Physical volume "/dev/sdb1" successfully created.
```

---

# Crear varios PV

```bash
sudo pvcreate /dev/sdb1 /dev/sdc1
```

---

# Consultar PV

Resumen:

```bash
pvs
```

Detalle:

```bash
pvdisplay
```

Ejemplo:

```text
PV Name               /dev/sdb1
VG Name
PV Size               100.00 GiB
Allocatable           yes
PE Size               4.00 MiB
```

---

# Crear un Volume Group

```bash
sudo vgcreate VG_DATOS /dev/sdb1
```

Con varios discos:

```bash
sudo vgcreate VG_DATOS /dev/sdb1 /dev/sdc1
```

---

# Consultar VG

Resumen:

```bash
vgs
```

Detalle:

```bash
vgdisplay
```

---

# Ejemplo

```text
VG Name        VG_DATOS
VG Size        200.00 GiB
PE Size        4.00 MiB
Total PE       51199
Alloc PE       0
Free PE        51199
```

---

# Crear un Logical Volume

20 GB:

```bash
sudo lvcreate -L 20G -n LV_BACKUP VG_DATOS
```

Usando porcentaje:

```bash
sudo lvcreate -l 100%FREE -n LV_DATOS VG_DATOS
```

---

# Consultar LV

Resumen:

```bash
lvs
```

Detalle:

```bash
lvdisplay
```

---

# Ubicación del LV

Ejemplo:

```text
/dev/VG_DATOS/LV_BACKUP
```

También:

```text
/dev/mapper/VG_DATOS-LV_BACKUP
```

---

# Crear sistema de archivos

XFS:

```bash
sudo mkfs.xfs /dev/VG_DATOS/LV_BACKUP
```

ext4:

```bash
sudo mkfs.ext4 /dev/VG_DATOS/LV_BACKUP
```

---

# Crear punto de montaje

```bash
sudo mkdir -p /backup
```

---

# Montar

```bash
sudo mount /dev/VG_DATOS/LV_BACKUP /backup
```

Verificar:

```bash
findmnt /backup
```

---

# Obtener UUID

```bash
sudo blkid /dev/VG_DATOS/LV_BACKUP
```

---

# Agregar a fstab

XFS:

```fstab
UUID=UUID_OBTENIDO /backup xfs defaults 0 0
```

ext4:

```fstab
UUID=UUID_OBTENIDO /backup ext4 defaults 0 2
```

Validar:

```bash
sudo mount -a
```

---

# Flujo completo

```bash
pvcreate /dev/sdb1
vgcreate VG_DATOS /dev/sdb1
lvcreate -L 50G -n LV_BACKUP VG_DATOS
mkfs.xfs /dev/VG_DATOS/LV_BACKUP
mkdir /backup
mount /dev/VG_DATOS/LV_BACKUP /backup
```

---

# Agregar otro disco

Inicializar:

```bash
sudo pvcreate /dev/sdc1
```

Agregar:

```bash
sudo vgextend VG_DATOS /dev/sdc1
```

Verificar:

```bash
vgs
```

---

# Reducir VG

Mover datos:

```bash
sudo pvmove /dev/sdc1
```

Eliminar:

```bash
sudo vgreduce VG_DATOS /dev/sdc1
```

---

# Eliminar PV

```bash
sudo pvremove /dev/sdc1
```

---

# Expandir Logical Volume

Agregar 20 GB:

```bash
sudo lvextend -L +20G /dev/VG_DATOS/LV_BACKUP
```

Usar todo el espacio libre:

```bash
sudo lvextend -l +100%FREE /dev/VG_DATOS/LV_BACKUP
```

---

# Expandir XFS

```bash
sudo xfs_growfs /backup
```

---

# Expandir ext4

```bash
sudo resize2fs /dev/VG_DATOS/LV_BACKUP
```

---

# Expandir automáticamente

```bash
sudo lvextend -r -L +20G /dev/VG_DATOS/LV_BACKUP
```

La opción `-r` intenta redimensionar también el sistema de archivos compatible.

---

# Reducir ext4

Desmontar:

```bash
sudo umount /backup
```

Verificar:

```bash
sudo e2fsck -f /dev/VG_DATOS/LV_BACKUP
```

Reducir:

```bash
sudo resize2fs /dev/VG_DATOS/LV_BACKUP 10G
```

Reducir LV:

```bash
sudo lvreduce -L 10G /dev/VG_DATOS/LV_BACKUP
```

---

# XFS no puede reducirse

Debe:

1. Crear nuevo LV.
2. Copiar datos.
3. Cambiar montaje.
4. Eliminar volumen anterior.

---

# Renombrar VG

```bash
sudo vgrename VG_DATOS VG_STORAGE
```

---

# Renombrar LV

```bash
sudo lvrename VG_STORAGE LV_BACKUP LV_ARCHIVO
```

---

# Eliminar LV

Desmontar:

```bash
sudo umount /backup
```

Eliminar:

```bash
sudo lvremove /dev/VG_DATOS/LV_BACKUP
```

---

# Eliminar VG

```bash
sudo vgremove VG_DATOS
```

---

# Eliminar PV

```bash
sudo pvremove /dev/sdb1
```

---

# Snapshots

LVM permite crear snapshots para respaldos consistentes.

Crear snapshot de 5 GB:

```bash
sudo lvcreate -L 5G -s \
-n SNAP_BACKUP \
/dev/VG_DATOS/LV_BACKUP
```

---

# Consultar snapshots

```bash
lvs
```

Ejemplo:

```text
LV           VG         Attr
LV_BACKUP    VG_DATOS   -wi-ao----
SNAP_BACKUP  VG_DATOS   swi-a-s---
```

---

# Montar snapshot

```bash
sudo mkdir /snapshot
sudo mount -o ro \
/dev/VG_DATOS/SNAP_BACKUP \
/snapshot
```

---

# Eliminar snapshot

```bash
sudo lvremove /dev/VG_DATOS/SNAP_BACKUP
```

---

# Consultar espacio

```bash
pvs
vgs
lvs
```

---

# Mostrar árbol

```bash
lsblk
```

Ejemplo:

```text
sdb
└─sdb1
  └─VG_DATOS-LV_BACKUP
```

---

# Mostrar UUID

```bash
blkid
```

---

# Comandos importantes

| Comando   | Descripción |
| --------- | ----------- |
| pvcreate  | Crear PV    |
| pvdisplay | Mostrar PV  |
| pvs       | Resumen PV  |
| vgcreate  | Crear VG    |
| vgdisplay | Mostrar VG  |
| vgs       | Resumen VG  |
| lvcreate  | Crear LV    |
| lvdisplay | Mostrar LV  |
| lvs       | Resumen LV  |
| lvextend  | Expandir LV |
| lvreduce  | Reducir LV  |
| lvremove  | Eliminar LV |
| vgextend  | Agregar PV  |
| vgreduce  | Eliminar PV |
| pvmove    | Mover datos |
| pvremove  | Eliminar PV |

---

# Laboratorio práctico

## Ejercicio 1

Crear dos discos virtuales.

Verificar:

```bash
lsblk
```

---

## Ejercicio 2

Inicializar:

```bash
pvcreate /dev/sdb1
pvcreate /dev/sdc1
```

---

## Ejercicio 3

Crear VG:

```bash
vgcreate VG_LAB /dev/sdb1 /dev/sdc1
```

---

## Ejercicio 4

Crear LV:

```bash
lvcreate -L 20G -n LV_DATOS VG_LAB
```

---

## Ejercicio 5

Formatear:

```bash
mkfs.xfs /dev/VG_LAB/LV_DATOS
```

---

## Ejercicio 6

Montar:

```bash
mkdir /lab
mount /dev/VG_LAB/LV_DATOS /lab
```

---

## Ejercicio 7

Expandir:

```bash
lvextend -L +10G /dev/VG_LAB/LV_DATOS
xfs_growfs /lab
```

---

## Ejercicio 8

Crear snapshot:

```bash
lvcreate -L 2G -s \
-n SNAP1 \
/dev/VG_LAB/LV_DATOS
```

---

## Ejercicio 9

Eliminar snapshot:

```bash
lvremove /dev/VG_LAB/SNAP1
```

---

## Ejercicio 10

Eliminar laboratorio:

```bash
umount /lab
lvremove /dev/VG_LAB/LV_DATOS
vgremove VG_LAB
pvremove /dev/sdb1
pvremove /dev/sdc1
```

---

# Buenas prácticas

* Utiliza LVM en servidores empresariales.
* Usa nombres descriptivos para VG y LV.
* Mantén espacio libre dentro del VG para futuras expansiones.
* Prefiere UUID en `/etc/fstab`.
* Antes de reducir un volumen, realiza un respaldo completo.
* Utiliza snapshots únicamente durante el tiempo necesario.
* Supervisa el espacio libre del VG regularmente.
* Documenta la estructura de almacenamiento.
* Prueba procedimientos de expansión y recuperación en entornos de laboratorio antes de aplicarlos en producción.

---

# Errores comunes

## Expandir el LV pero olvidar el sistema de archivos

Incorrecto:

```bash
lvextend -L +20G /dev/VG_DATOS/LV_BACKUP
```

Correcto:

```bash
lvextend -L +20G /dev/VG_DATOS/LV_BACKUP
xfs_growfs /backup
```

o:

```bash
resize2fs /dev/VG_DATOS/LV_BACKUP
```

según el sistema de archivos.

---

## Reducir XFS

XFS no admite reducción directa.

---

## Crear snapshots demasiado pequeños

Si un snapshot se llena, dejará de ser utilizable.

---

## Eliminar un PV sin mover los datos

Debe utilizarse:

```bash
pvmove
```

antes de ejecutar:

```bash
vgreduce
```

---

## Eliminar un LV montado

Siempre desmonta el volumen antes de eliminarlo.

---

# Resumen

En este capítulo aprendiste a:

* Comprender la arquitectura de LVM.
* Crear Physical Volumes.
* Crear Volume Groups.
* Crear Logical Volumes.
* Formatear y montar volúmenes lógicos.
* Configurar montajes persistentes.
* Expandir volúmenes y sistemas de archivos.
* Reducir volúmenes ext4 de forma segura.
* Comprender las limitaciones de XFS.
* Crear y administrar snapshots.
* Agregar y retirar discos de un Volume Group.
* Consultar el estado de LVM.
* Aplicar buenas prácticas para la administración de almacenamiento empresarial.

El siguiente capítulo recomendado es **25-raid-software.md**, donde se estudiará la implementación y administración de **RAID por software con `mdadm`**, incluyendo RAID 0, RAID 1, RAID 5, RAID 6, RAID 10, reemplazo de discos, reconstrucción y monitoreo.
