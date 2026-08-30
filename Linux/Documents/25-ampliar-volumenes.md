# 25. Ampliación de Volúmenes, Particiones y Sistemas de Archivos en Linux

La ampliación de almacenamiento es una de las tareas más frecuentes en la administración de servidores Linux.

Con el tiempo, los sistemas de archivos pueden quedarse sin espacio debido al crecimiento de:

* Bases de datos.
* Registros del sistema.
* Respaldos.
* Archivos de aplicaciones.
* Contenedores.
* Directorios de usuarios.
* Datos temporales.
* Repositorios.
* Máquinas virtuales.

En Linux, ampliar almacenamiento no consiste únicamente en aumentar el tamaño de un disco. Es necesario comprender todas las capas involucradas.

Una estructura típica puede incluir:

```text
Disco físico o virtual
        ↓
Tabla de particiones
        ↓
Partición
        ↓
Physical Volume de LVM
        ↓
Volume Group
        ↓
Logical Volume
        ↓
Sistema de archivos
        ↓
Punto de montaje
```

Para que el nuevo espacio aparezca disponible en el sistema, debe ampliarse correctamente cada capa necesaria.

---

# Objetivos

Al finalizar este capítulo serás capaz de:

* Analizar la estructura actual del almacenamiento.
* Identificar qué capa necesita ampliación.
* Detectar espacio libre en discos, particiones y grupos LVM.
* Ampliar particiones tradicionales.
* Ampliar Physical Volumes.
* Ampliar Volume Groups.
* Ampliar Logical Volumes.
* Expandir sistemas de archivos XFS.
* Expandir sistemas de archivos ext4.
* Utilizar `lvextend`, `growpart`, `parted`, `pvresize`, `xfs_growfs` y `resize2fs`.
* Aplicar ampliaciones en línea cuando sean compatibles.
* Verificar el resultado después de cada operación.
* Reconocer riesgos y errores comunes.
* Ejecutar procedimientos orientados al examen RHCSA.

---

# Concepto general de ampliación

Una ampliación consiste en proporcionar capacidad adicional a un volumen existente.

El procedimiento depende del tipo de almacenamiento.

## Escenario 1: LVM con espacio libre en el Volume Group

```text
Volume Group con espacio libre
        ↓
Ampliar Logical Volume
        ↓
Ampliar sistema de archivos
```

## Escenario 2: LVM sin espacio libre en el Volume Group

```text
Agregar disco o partición
        ↓
Crear Physical Volume
        ↓
Ampliar Volume Group
        ↓
Ampliar Logical Volume
        ↓
Ampliar sistema de archivos
```

## Escenario 3: disco virtual ampliado

```text
Ampliar disco desde hipervisor
        ↓
Solicitar relectura del disco
        ↓
Ampliar partición
        ↓
Ampliar PV de LVM
        ↓
Ampliar LV
        ↓
Ampliar sistema de archivos
```

## Escenario 4: partición tradicional sin LVM

```text
Ampliar disco
        ↓
Ampliar partición
        ↓
Ampliar sistema de archivos
```

---

# Regla fundamental

Ampliar una capa inferior no amplía automáticamente las capas superiores.

Por ejemplo:

```bash
sudo lvextend -L +10G /dev/vgdatos/lvdatos
```

amplía el Logical Volume, pero el sistema de archivos podría continuar mostrando el tamaño anterior.

Debe ampliarse también el sistema de archivos.

Para XFS:

```bash
sudo xfs_growfs /datos
```

Para ext4:

```bash
sudo resize2fs /dev/vgdatos/lvdatos
```

También puede utilizarse:

```bash
sudo lvextend -r -L +10G /dev/vgdatos/lvdatos
```

La opción `-r` intenta ampliar el Logical Volume y el sistema de archivos automáticamente.

---

# Antes de ampliar

Antes de realizar cambios, debe recopilarse información del entorno.

## Ver sistemas de archivos

```bash
df -Th
```

## Ver discos y particiones

```bash
lsblk
```

Con más detalle:

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
```

## Ver sistemas de archivos y UUID

```bash
lsblk -f
```

## Ver estructura LVM

```bash
sudo pvs
sudo vgs
sudo lvs
```

## Ver árbol completo

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS,PKNAME
```

---

# Ejemplo de estructura

```text
NAME                 SIZE TYPE FSTYPE      MOUNTPOINTS
sda                  100G disk
├─sda1                 1G part xfs         /boot
└─sda2                99G part LVM2_member
  ├─vgserver-root      50G lvm  xfs         /
  ├─vgserver-var       20G lvm  xfs         /var
  └─vgserver-home      10G lvm  xfs         /home
```

Consultar el VG:

```bash
sudo vgs
```

Ejemplo:

```text
VG        #PV #LV #SN Attr   VSize   VFree
vgserver    1   3   0 wz--n- 98.99g 18.99g
```

En este caso existen aproximadamente 19 GB libres en el Volume Group.

Por tanto, puede ampliarse un Logical Volume sin agregar otro disco.

---

# Identificar el origen de un punto de montaje

Para conocer qué dispositivo contiene una ruta:

```bash
findmnt /var
```

Ejemplo:

```text
TARGET SOURCE                 FSTYPE OPTIONS
/var   /dev/mapper/vg-var     xfs    rw,relatime
```

También:

```bash
findmnt -no SOURCE /var
```

Resultado:

```text
/dev/mapper/vg-var
```

---

# Consultar el sistema de archivos

```bash
findmnt -no FSTYPE /var
```

Resultado posible:

```text
xfs
```

Esto es importante porque el procedimiento final depende del tipo de sistema de archivos.

---

# Comprobar espacio disponible en el VG

```bash
sudo vgs
```

Salida:

```text
VG       #PV #LV #SN Attr   VSize   VFree
vgdatos    2   3   0 wz--n- 500.00g 80.00g
```

La columna `VFree` indica cuánto espacio libre puede asignarse.

Detalle:

```bash
sudo vgdisplay vgdatos
```

Buscar:

```text
Free PE / Size
```

---

# Consultar Logical Volumes

```bash
sudo lvs
```

Salida:

```text
LV       VG       Attr       LSize
lvdatos  vgdatos  -wi-ao---- 200.00g
lvbackup vgdatos  -wi-ao---- 100.00g
```

Salida detallada:

```bash
sudo lvs -o lv_name,vg_name,lv_size,lv_path,devices
```

---

# Diferencia entre `-L` y `-l`

En comandos LVM se utilizan dos opciones importantes.

## `-L`

Trabaja con tamaños expresados en unidades.

Ejemplos:

```bash
sudo lvextend -L +10G /dev/vgdatos/lvdatos
```

```bash
sudo lvextend -L 100G /dev/vgdatos/lvdatos
```

El signo `+` significa agregar.

Sin `+`, el valor representa el tamaño final.

---

## `-l`

Trabaja con extents o porcentajes.

Ejemplo usando todo el espacio libre:

```bash
sudo lvextend -l +100%FREE /dev/vgdatos/lvdatos
```

Agregar la mitad del espacio libre:

```bash
sudo lvextend -l +50%FREE /dev/vgdatos/lvdatos
```

Asignar un número de extents:

```bash
sudo lvextend -l +2560 /dev/vgdatos/lvdatos
```

---

# Cuidado con el signo más

Estos comandos no significan lo mismo.

## Agregar 10 GB

```bash
sudo lvextend -L +10G /dev/vgdatos/lvdatos
```

## Establecer tamaño final en 10 GB

```bash
sudo lvextend -L 10G /dev/vgdatos/lvdatos
```

Si el volumen ya supera los 10 GB, el segundo comando no lo ampliará.

En tareas de expansión suele utilizarse:

```text
+10G
```

---

# Ampliar un Logical Volume XFS

Supongamos:

```text
Logical Volume: /dev/vgdatos/lvdatos
Punto de montaje: /datos
Sistema de archivos: XFS
Espacio adicional: 20 GB
```

## Paso 1: verificar

```bash
sudo lvs
sudo vgs
df -Th /datos
findmnt /datos
```

## Paso 2: ampliar el LV

```bash
sudo lvextend -L +20G /dev/vgdatos/lvdatos
```

## Paso 3: ampliar XFS

```bash
sudo xfs_growfs /datos
```

## Paso 4: verificar

```bash
sudo lvs
df -Th /datos
```

---

# Ampliar XFS automáticamente con `-r`

```bash
sudo lvextend -r -L +20G /dev/vgdatos/lvdatos
```

La opción `-r` ejecuta el redimensionamiento del sistema de archivos mediante `fsadm` o la herramienta apropiada.

Verificar:

```bash
df -Th /datos
```

---

# Ampliar XFS usando todo el espacio libre

```bash
sudo lvextend -r -l +100%FREE /dev/vgdatos/lvdatos
```

Esto asigna todo el espacio disponible del VG al volumen.

Debe utilizarse con criterio, ya que dejar el VG sin espacio libre reduce la flexibilidad para futuras ampliaciones o snapshots.

---

# Ampliar ext4

Supongamos:

```text
Logical Volume: /dev/vgdatos/lvarchivos
Punto de montaje: /archivos
Sistema de archivos: ext4
Espacio adicional: 15 GB
```

## Paso 1: ampliar LV

```bash
sudo lvextend -L +15G /dev/vgdatos/lvarchivos
```

## Paso 2: ampliar ext4

```bash
sudo resize2fs /dev/vgdatos/lvarchivos
```

## Paso 3: verificar

```bash
df -Th /archivos
sudo lvs
```

---

# Ampliar ext4 automáticamente

```bash
sudo lvextend -r -L +15G /dev/vgdatos/lvarchivos
```

En la mayoría de los casos ext4 puede ampliarse en línea mientras está montado.

---

# XFS utiliza el punto de montaje

Para XFS se utiliza normalmente:

```bash
sudo xfs_growfs /datos
```

No:

```bash
sudo xfs_growfs /dev/vgdatos/lvdatos
```

`xfs_growfs` trabaja sobre el sistema de archivos montado.

---

# ext4 utiliza el dispositivo

Para ext4 se utiliza:

```bash
sudo resize2fs /dev/vgdatos/lvarchivos
```

Aunque el sistema esté montado, se especifica normalmente el dispositivo.

---

# Ampliar solamente una parte del espacio libre

Supongamos que el VG tiene 100 GB libres y se desean agregar 40 GB.

```bash
sudo lvextend -r -L +40G /dev/vgdatos/lvdatos
```

Verificar espacio restante:

```bash
sudo vgs
```

---

# Agregar un nuevo disco a LVM

Supongamos que se agregó:

```text
/dev/sdb
```

## Paso 1: verificar el disco

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
```

## Paso 2: crear una partición LVM

```bash
sudo fdisk /dev/sdb
```

Dentro de `fdisk`:

```text
n
t
Linux LVM
w
```

Actualizar kernel:

```bash
sudo partprobe /dev/sdb
```

Verificar:

```bash
lsblk
```

---

# Crear PV sobre el nuevo disco

```bash
sudo pvcreate /dev/sdb1
```

Verificar:

```bash
sudo pvs
```

---

# Agregar PV al Volume Group

```bash
sudo vgextend vgdatos /dev/sdb1
```

Verificar:

```bash
sudo vgs
sudo pvs
```

---

# Ampliar el LV después de agregar el disco

Agregar 100 GB:

```bash
sudo lvextend -r -L +100G /dev/vgdatos/lvdatos
```

Usar todo:

```bash
sudo lvextend -r -l +100%FREE /dev/vgdatos/lvdatos
```

Verificar:

```bash
df -Th /datos
sudo lvs
sudo vgs
```

---

# Utilizar un disco completo como PV

LVM puede utilizar directamente un disco completo:

```bash
sudo pvcreate /dev/sdb
```

Luego:

```bash
sudo vgextend vgdatos /dev/sdb
```

Sin embargo, en muchos entornos se prefiere crear una partición para facilitar identificación, documentación y compatibilidad con herramientas.

Debe mantenerse un estándar consistente.

---

# Disco virtual ampliado desde un hipervisor

En entornos VMware, VirtualBox, Hyper-V, KVM o plataformas cloud, puede ampliarse el disco virtual desde la plataforma.

Ejemplo:

```text
Tamaño anterior: 100 GB
Tamaño nuevo: 150 GB
```

Linux puede continuar mostrando el tamaño anterior hasta solicitar una nueva lectura.

---

# Comprobar el tamaño detectado

```bash
lsblk
```

También:

```bash
sudo fdisk -l /dev/sda
```

Consultar kernel:

```bash
cat /sys/class/block/sda/size
```

---

# Solicitar relectura del disco

Para discos SCSI:

```bash
echo 1 | sudo tee /sys/class/block/sda/device/rescan
```

Verificar:

```bash
lsblk
```

Otra opción:

```bash
sudo partprobe /dev/sda
```

También:

```bash
sudo blockdev --rereadpt /dev/sda
```

Cada comando tiene un propósito diferente. El rescan actualiza el tamaño detectado y `partprobe` solicita releer la tabla de particiones.

---

# Escanear hosts SCSI

```bash
for host in /sys/class/scsi_host/host*; do
    echo "- - -" | sudo tee "$host/scan"
done
```

Verificar:

```bash
lsblk
```

---

# Ampliar una partición con `growpart`

Supongamos que el disco `/dev/sda` fue ampliado y LVM utiliza `/dev/sda3`.

Instalar herramienta:

```bash
sudo dnf install cloud-utils-growpart
```

Simular o comprobar:

```bash
sudo growpart -N /dev/sda 3
```

Ampliar:

```bash
sudo growpart /dev/sda 3
```

Verificar:

```bash
lsblk
```

---

# Sintaxis de growpart

```bash
sudo growpart DISCO NUMERO_PARTICION
```

Ejemplo:

```bash
sudo growpart /dev/sda 3
```

No debe escribirse:

```bash
sudo growpart /dev/sda3
```

El disco y el número se proporcionan por separado.

---

# NVMe con growpart

Para:

```text
/dev/nvme0n1p3
```

se ejecuta:

```bash
sudo growpart /dev/nvme0n1 3
```

Verificar:

```bash
lsblk
```

---

# Ampliar una partición con `parted`

Consultar:

```bash
sudo parted /dev/sda print free
```

Ampliar partición 3 hasta el final:

```bash
sudo parted /dev/sda resizepart 3 100%
```

Actualizar:

```bash
sudo partprobe /dev/sda
```

Verificar:

```bash
lsblk
```

---

# Advertencia sobre particiones montadas

Ampliar el final de una partición suele ser posible sin desmontarla, dependiendo de:

* Tipo de partición.
* Tabla GPT o MBR.
* Kernel.
* Herramienta utilizada.
* Posición del espacio libre.
* Dispositivo subyacente.

No debe modificarse el inicio de una partición activa.

Antes de trabajar en producción:

* Realiza respaldo.
* Confirma la partición correcta.
* Verifica que el espacio libre esté contiguo.
* Programa ventana de mantenimiento si existe riesgo.

---

# Ampliar el Physical Volume con `pvresize`

Después de ampliar la partición que contiene LVM:

```bash
sudo pvresize /dev/sda3
```

Verificar:

```bash
sudo pvs
sudo vgs
```

Sin `pvresize`, el PV podría continuar utilizando el tamaño anterior.

---

# Flujo completo después de ampliar disco virtual

Supongamos:

```text
Disco: /dev/sda
Partición LVM: /dev/sda3
VG: vgserver
LV: /dev/vgserver/lvvar
Punto: /var
Sistema: XFS
```

## Paso 1: detectar nuevo tamaño

```bash
echo 1 | sudo tee /sys/class/block/sda/device/rescan
lsblk
```

## Paso 2: ampliar partición

```bash
sudo growpart /dev/sda 3
```

## Paso 3: ampliar PV

```bash
sudo pvresize /dev/sda3
```

## Paso 4: comprobar VG

```bash
sudo vgs
```

## Paso 5: ampliar LV y sistema

```bash
sudo lvextend -r -L +20G /dev/vgserver/lvvar
```

## Paso 6: verificar

```bash
df -Th /var
sudo lvs
sudo vgs
```

---

# Ampliar una partición ext4 sin LVM

Supongamos:

```text
Disco: /dev/sdb
Partición: /dev/sdb1
Sistema: ext4
Punto: /datos
```

El disco fue ampliado.

## Paso 1: ampliar partición

```bash
sudo growpart /dev/sdb 1
```

## Paso 2: ampliar ext4

```bash
sudo resize2fs /dev/sdb1
```

## Paso 3: verificar

```bash
df -Th /datos
lsblk
```

---

# Ampliar una partición XFS sin LVM

Supongamos:

```text
Disco: /dev/sdb
Partición: /dev/sdb1
Sistema: XFS
Punto: /datos
```

## Paso 1: ampliar partición

```bash
sudo growpart /dev/sdb 1
```

## Paso 2: ampliar XFS

```bash
sudo xfs_growfs /datos
```

## Paso 3: verificar

```bash
df -Th /datos
```

---

# Ampliar un LV a un tamaño final

Supongamos que el LV mide 50 GB y debe terminar con 80 GB.

```bash
sudo lvextend -r -L 80G /dev/vgdatos/lvdatos
```

Esto establece un tamaño final de 80 GB.

No agrega 80 GB.

Para agregar 80 GB:

```bash
sudo lvextend -r -L +80G /dev/vgdatos/lvdatos
```

---

# Consultar tamaño exacto

```bash
sudo lvs --units g
```

Más precisión:

```bash
sudo lvs --units g --nosuffix
```

Consultar un LV:

```bash
sudo lvs /dev/vgdatos/lvdatos
```

---

# Usar extents para tareas RHCSA

En algunos ejercicios puede solicitarse ampliar por un número de extents.

Consultar tamaño de extent:

```bash
sudo vgdisplay vgdatos | grep "PE Size"
```

Ejemplo:

```text
PE Size 4.00 MiB
```

Agregar 512 extents:

```bash
sudo lvextend -l +512 /dev/vgdatos/lvdatos
```

Espacio aproximado:

```text
512 × 4 MiB = 2048 MiB
```

Luego ampliar el sistema:

```bash
sudo xfs_growfs /datos
```

o:

```bash
sudo resize2fs /dev/vgdatos/lvdatos
```

---

# Calcular extents necesarios

Supongamos:

```text
Tamaño de PE: 4 MiB
Espacio adicional: 8 GiB
```

Convertir:

```text
8 GiB = 8192 MiB
```

Dividir:

```text
8192 / 4 = 2048 extents
```

Comando:

```bash
sudo lvextend -l +2048 /dev/vgdatos/lvdatos
```

---

# Uso de `fsadm`

LVM puede utilizar `fsadm` para redimensionar ciertos sistemas de archivos.

Ejemplo:

```bash
sudo fsadm resize /dev/vgdatos/lvarchivos
```

Sin embargo, en la práctica se utilizan con más frecuencia:

```bash
xfs_growfs
```

y:

```bash
resize2fs
```

La opción `lvextend -r` automatiza parte de este proceso.

---

# Verificar antes y después

Antes:

```bash
df -Th /datos
sudo lvs
sudo vgs
```

Después:

```bash
df -Th /datos
sudo lvs
sudo vgs
lsblk
```

Debe confirmarse que:

* El LV tiene el nuevo tamaño.
* El sistema de archivos tiene el nuevo tamaño.
* El punto continúa montado.
* No hay errores en logs.
* La aplicación puede acceder al volumen.

---

# Revisar logs después de ampliar

```bash
journalctl -k --since "10 minutes ago"
```

Filtrar:

```bash
journalctl -k | grep -Ei "xfs|ext4|lvm|I/O error|filesystem"
```

---

# Ampliación de `/`

La raíz puede ampliarse en línea si utiliza LVM y un sistema compatible.

Identificar:

```bash
findmnt /
```

Ejemplo:

```text
/ /dev/mapper/vgserver-root xfs rw,relatime
```

Consultar espacio:

```bash
sudo vgs
```

Ampliar:

```bash
sudo lvextend -r -L +10G /dev/vgserver/root
```

Verificar:

```bash
df -Th /
```

Debe confirmarse cuidadosamente la ruta real del LV.

---

# Ampliación de `/var`

```bash
findmnt /var
sudo vgs
sudo lvs
```

Ampliar 10 GB:

```bash
sudo lvextend -r -L +10G /dev/vgserver/lvvar
```

Verificar:

```bash
df -Th /var
```

Esto puede realizarse en línea si el sistema de archivos lo permite.

---

# Ampliación de `/home`

```bash
sudo lvextend -r -L +5G /dev/vgserver/lvhome
```

Verificar:

```bash
df -Th /home
```

---

# Ampliar almacenamiento de PostgreSQL

Antes de ampliar:

```bash
df -Th /var/lib/pgsql
findmnt -T /var/lib/pgsql
sudo lvs
sudo vgs
```

Si el volumen es XFS:

```bash
sudo lvextend -r -L +50G /dev/vgdata/lvpostgres
```

Verificar:

```bash
df -Th /var/lib/pgsql
```

En una ampliación normal en línea no suele ser necesario detener PostgreSQL, pero debe validarse:

* Sistema de archivos.
* Tipo de almacenamiento.
* Política operativa.
* Riesgo de la plataforma.
* Respaldo disponible.

---

# Ampliar almacenamiento de SQL Server

Consultar:

```bash
df -Th /var/opt/mssql
findmnt -T /var/opt/mssql
sudo lvs
sudo vgs
```

Ampliar:

```bash
sudo lvextend -r -L +50G /dev/vgsql/lvmssql
```

Verificar:

```bash
df -Th /var/opt/mssql
```

Revisar el servicio:

```bash
systemctl status mssql-server
```

---

# Uso de todo el espacio libre: ventajas y riesgos

Comando:

```bash
sudo lvextend -r -l +100%FREE /dev/vgdatos/lvdatos
```

Ventajas:

* Aprovecha todo el espacio disponible.
* Simplifica una expansión urgente.
* Evita cálculos manuales.

Riesgos:

* El VG queda sin espacio libre.
* No habrá margen para otros LV.
* Se limita la creación de snapshots tradicionales.
* Futuras ampliaciones requerirán otro PV.
* Puede afectar planes de contingencia.

Es recomendable conservar una reserva cuando sea posible.

---

# Ampliar Thin Pools

LVM thin provisioning utiliza thin pools.

Consultar:

```bash
sudo lvs -a
```

Ejemplo:

```text
LV       VG     Attr       LSize  Pool Data% Meta%
thinpool vgdata twi-aotz-- 500.00g      80.00 12.00
```

Ampliar el thin pool:

```bash
sudo lvextend -L +100G /dev/vgdata/thinpool
```

Ampliar metadatos:

```bash
sudo lvextend --poolmetadatasize +1G /dev/vgdata/thinpool
```

Los thin pools requieren monitoreo constante de:

* `Data%`.
* `Meta%`.
* Espacio libre del VG.
* Estado de autoextensión.

Un thin pool lleno puede provocar indisponibilidad o corrupción.

---

# Autoextensión de thin pools

Consultar configuración:

```bash
grep -E "thin_pool_autoextend_threshold|thin_pool_autoextend_percent" \
/etc/lvm/lvm.conf
```

Valores posibles:

```text
thin_pool_autoextend_threshold = 80
thin_pool_autoextend_percent = 20
```

Esto permite ampliar automáticamente cuando se alcanza un umbral, siempre que exista espacio libre en el VG.

---

# Ampliar swap en LVM

La swap no se amplía como un sistema de archivos normal.

Supongamos:

```text
/dev/vgserver/lvswap
```

Desactivar:

```bash
sudo swapoff /dev/vgserver/lvswap
```

Ampliar:

```bash
sudo lvextend -L +2G /dev/vgserver/lvswap
```

Recrear firma:

```bash
sudo mkswap /dev/vgserver/lvswap
```

Activar:

```bash
sudo swapon /dev/vgserver/lvswap
```

Verificar:

```bash
swapon --show
free -h
```

Debe confirmarse que exista suficiente memoria disponible antes de ejecutar `swapoff`.

---

# Crear otro volumen swap en lugar de ampliar

Puede crearse un segundo LV:

```bash
sudo lvcreate -L 2G -n lvswap2 vgserver
sudo mkswap /dev/vgserver/lvswap2
sudo swapon /dev/vgserver/lvswap2
```

Agregar a `/etc/fstab`:

```fstab
UUID=UUID_SWAP none swap defaults 0 0
```

Consultar UUID:

```bash
sudo blkid /dev/vgserver/lvswap2
```

---

# Ampliar una partición cuando no hay espacio contiguo

Una partición tradicional solamente puede ampliarse hacia espacio libre contiguo, normalmente ubicado después de la partición.

Ejemplo problemático:

```text
/dev/sda1  20G
/dev/sda2  30G
Espacio libre 50G
```

No puede ampliarse `/dev/sda1` directamente porque `/dev/sda2` está en medio.

Alternativas:

* Agregar otro disco.
* Migrar a LVM.
* Mover particiones desde un entorno offline.
* Crear una nueva partición y migrar datos.
* Utilizar otro volumen.
* Reorganizar almacenamiento durante mantenimiento.

---

# MBR y particiones extendidas

En discos MBR pueden existir limitaciones relacionadas con:

* Máximo de cuatro particiones primarias.
* Particiones extendidas.
* Límite tradicional de 2 TB.
* Posición del espacio libre.

Para almacenamiento moderno se recomienda GPT.

---

# GPT y ampliación

GPT soporta discos grandes y numerosas particiones.

Después de ampliar un disco GPT, algunas herramientas pueden advertir que la copia secundaria de la tabla no está al final.

Puede corregirse con herramientas como:

```bash
sudo parted /dev/sda print
```

o:

```bash
sudo gdisk /dev/sda
```

Debe revisarse cuidadosamente antes de guardar cambios.

---

# Ampliar discos NVMe

Ejemplo:

```text
Disco: /dev/nvme0n1
Partición LVM: /dev/nvme0n1p3
```

Verificar:

```bash
lsblk
```

Ampliar partición:

```bash
sudo growpart /dev/nvme0n1 3
```

Ampliar PV:

```bash
sudo pvresize /dev/nvme0n1p3
```

Ampliar LV:

```bash
sudo lvextend -r -L +20G /dev/vgserver/lvdata
```

---

# Utilizar `partprobe`

Después de cambiar una tabla de particiones:

```bash
sudo partprobe /dev/sda
```

Verificar:

```bash
lsblk
```

Si el kernel no puede releer la tabla porque alguna partición está en uso, puede ser necesario reiniciar.

No debe forzarse una relectura sin evaluar el impacto.

---

# Uso de `udevadm settle`

```bash
sudo udevadm settle
```

Este comando espera a que terminen eventos pendientes de udev.

Puede ejecutarse después de detectar o modificar dispositivos.

---

# Comprobar espacio sin asignar en un disco

```bash
sudo parted /dev/sda print free
```

Ejemplo:

```text
Number Start   End     Size    File system Name Flags
       1049kB  2097kB 1049kB  Free Space
1      2097kB  1076MB 1074MB  xfs
2      1076MB  80.0GB 78.9GB
       80.0GB  120GB   40.0GB  Free Space
```

---

# Comprobar espacio libre de un PV

```bash
sudo pvs
```

Salida:

```text
PV         VG       Fmt Attr PSize   PFree
/dev/sda3  vgserver lvm2 a-- 149.00g 50.00g
```

---

# Comprobar espacio libre de todos los niveles

```bash
lsblk
df -Th
sudo pvs
sudo vgs
sudo lvs
sudo parted /dev/sda print free
```

Estos comandos permiten determinar dónde existe realmente el espacio.

---

# Diferencia entre PFree y VFree

`PFree` muestra espacio libre dentro de un Physical Volume.

`VFree` muestra espacio libre total dentro del Volume Group.

Ejemplo:

```bash
sudo pvs
sudo vgs
```

Si hay varios PV, el `VFree` representa la suma utilizable por el VG.

---

# Prueba previa con `--test`

Muchos comandos LVM aceptan:

```bash
--test
```

Ejemplo:

```bash
sudo lvextend --test -L +20G /dev/vgdatos/lvdatos
```

Esto simula parte de la operación sin escribir los metadatos finales.

También puede utilizarse modo detallado:

```bash
sudo lvextend -v --test -L +20G /dev/vgdatos/lvdatos
```

Una prueba no sustituye el respaldo ni la validación completa.

---

# Respaldo de metadatos LVM

LVM guarda copias automáticas de sus metadatos en:

```text
/etc/lvm/backup/
```

y:

```text
/etc/lvm/archive/
```

Consultar:

```bash
sudo ls -l /etc/lvm/backup/
sudo ls -l /etc/lvm/archive/
```

Crear respaldo manual:

```bash
sudo vgcfgbackup vgdatos
```

---

# Respaldo antes de ampliar

Una ampliación es generalmente menos riesgosa que una reducción, pero debe existir respaldo especialmente cuando:

* Se modifica la tabla de particiones.
* Se trabaja sobre discos de producción.
* Se amplía el disco del sistema.
* Se utiliza almacenamiento remoto.
* Se trabaja con bases de datos críticas.
* Se modifica GPT o MBR.
* No existe acceso físico o consola.

---

# Validación de aplicaciones después de ampliar

Después de ampliar un volumen que utiliza una aplicación:

```bash
systemctl status nombre-servicio
```

Revisar logs:

```bash
journalctl -u nombre-servicio --since "10 minutes ago"
```

Verificar escritura:

```bash
sudo -u usuario_aplicacion touch /ruta/prueba_ampliacion
```

Eliminar prueba:

```bash
sudo rm /ruta/prueba_ampliacion
```

Debe utilizarse el usuario correcto y una ruta segura.

---

# Automatización segura

Ejemplo de script para ampliar un LV XFS:

```bash
#!/bin/bash

set -euo pipefail

LV="/dev/vgdatos/lvdatos"
MOUNT="/datos"
SIZE="+10G"

echo "Estado antes:"
lvs "$LV"
df -Th "$MOUNT"
vgs

if ! mountpoint -q "$MOUNT"; then
    echo "ERROR: $MOUNT no está montado."
    exit 1
fi

FSTYPE=$(findmnt -no FSTYPE "$MOUNT")

if [[ "$FSTYPE" != "xfs" ]]; then
    echo "ERROR: Se esperaba XFS y se encontró $FSTYPE."
    exit 1
fi

lvextend -L "$SIZE" "$LV"
xfs_growfs "$MOUNT"

echo "Estado después:"
lvs "$LV"
df -Th "$MOUNT"
vgs
```

---

# Script genérico con `lvextend -r`

```bash
#!/bin/bash

set -euo pipefail

LV="/dev/vgdatos/lvdatos"
SIZE="+10G"

if [[ $EUID -ne 0 ]]; then
    echo "Debe ejecutarse como root."
    exit 1
fi

if ! lvs "$LV" >/dev/null 2>&1; then
    echo "No existe el Logical Volume: $LV"
    exit 1
fi

echo "Antes:"
lvs "$LV"
vgs

lvextend -r -L "$SIZE" "$LV"

echo "Después:"
lvs "$LV"
vgs
```

---

# Laboratorio 1: ampliar XFS con espacio libre en el VG

## Escenario

```text
VG: vg_lab
LV: lv_datos
Punto: /labdatos
Sistema: XFS
Aumento: 2 GB
```

## Paso 1

```bash
sudo vgs
sudo lvs
df -Th /labdatos
```

## Paso 2

```bash
sudo lvextend -L +2G /dev/vg_lab/lv_datos
```

## Paso 3

```bash
sudo xfs_growfs /labdatos
```

## Paso 4

```bash
sudo lvs
df -Th /labdatos
```

---

# Laboratorio 2: ampliar ext4

## Escenario

```text
VG: vg_lab
LV: lv_archivos
Punto: /labarchivos
Sistema: ext4
Aumento: 1 GB
```

## Procedimiento

```bash
sudo lvextend -L +1G /dev/vg_lab/lv_archivos
sudo resize2fs /dev/vg_lab/lv_archivos
df -Th /labarchivos
```

---

# Laboratorio 3: ampliar automáticamente

```bash
sudo lvextend -r -L +1G /dev/vg_lab/lv_datos
```

Verificar:

```bash
sudo lvs
df -Th /labdatos
```

---

# Laboratorio 4: usar porcentaje libre

Consultar:

```bash
sudo vgs
```

Asignar 50% del espacio libre:

```bash
sudo lvextend -r -l +50%FREE /dev/vg_lab/lv_datos
```

Verificar:

```bash
sudo vgs
sudo lvs
df -Th /labdatos
```

---

# Laboratorio 5: agregar un nuevo PV

Crear PV:

```bash
sudo pvcreate /dev/sdc1
```

Agregar al VG:

```bash
sudo vgextend vg_lab /dev/sdc1
```

Verificar:

```bash
sudo pvs
sudo vgs
```

Ampliar LV:

```bash
sudo lvextend -r -L +5G /dev/vg_lab/lv_datos
```

---

# Laboratorio 6: ampliar disco virtual y partición

Supongamos que `/dev/sdb` fue ampliado y la partición es `/dev/sdb1`.

```bash
lsblk
sudo growpart /dev/sdb 1
sudo pvresize /dev/sdb1
sudo vgs
sudo lvextend -r -l +100%FREE /dev/vg_lab/lv_datos
df -Th /labdatos
```

---

# Laboratorio 7: ampliar partición ext4 sin LVM

```bash
sudo growpart /dev/sdd 1
sudo resize2fs /dev/sdd1
df -Th /mnt/ext4lab
```

---

# Laboratorio 8: ampliar partición XFS sin LVM

```bash
sudo growpart /dev/sde 1
sudo xfs_growfs /mnt/xfslab
df -Th /mnt/xfslab
```

---

# Laboratorio 9: cálculo de extents

Consultar:

```bash
sudo vgdisplay vg_lab | grep "PE Size"
```

Si el PE es 4 MiB y se desean agregar 1 GiB:

```text
1024 MiB / 4 MiB = 256 extents
```

Ejecutar:

```bash
sudo lvextend -l +256 /dev/vg_lab/lv_datos
sudo xfs_growfs /labdatos
```

---

# Laboratorio 10: verificar todas las capas

```bash
lsblk
findmnt /labdatos
df -Th /labdatos
sudo pvs
sudo vgs
sudo lvs
```

Documentar:

* Disco.
* Partición.
* PV.
* VG.
* LV.
* Sistema de archivos.
* Punto de montaje.
* Tamaño inicial.
* Tamaño final.

---

# Práctica RHCSA

Una tarea típica puede indicar:

```text
Amplíe el volumen lógico lvdatos en 500 MiB y ajuste el sistema de archivos.
```

## Paso 1: identificar

```bash
sudo lvs
sudo vgs
findmnt /datos
```

## Paso 2: ampliar

```bash
sudo lvextend -r -L +500M /dev/vgdatos/lvdatos
```

## Paso 3: verificar

```bash
sudo lvs
df -Th /datos
```

---

# Práctica RHCSA con extents

Tarea:

```text
Amplíe el volumen lógico utilizando 100 extents adicionales.
```

Ejecutar:

```bash
sudo lvextend -l +100 /dev/vgdatos/lvdatos
```

Luego, según el sistema:

XFS:

```bash
sudo xfs_growfs /datos
```

ext4:

```bash
sudo resize2fs /dev/vgdatos/lvdatos
```

Verificar:

```bash
df -Th /datos
```

---

# Diagnóstico: el LV creció, pero df no

Síntoma:

```bash
sudo lvs
```

muestra el nuevo tamaño, pero:

```bash
df -h
```

muestra el tamaño anterior.

Causa:

El sistema de archivos no fue ampliado.

Para XFS:

```bash
sudo xfs_growfs /punto
```

Para ext4:

```bash
sudo resize2fs /dev/vg/lv
```

---

# Diagnóstico: insufficient free space

Error:

```text
Insufficient free space: 2560 extents needed, but only 1000 available
```

Consultar:

```bash
sudo vgs
```

Opciones:

* Ampliar menos.
* Agregar otro PV.
* Liberar espacio eliminando otro LV.
* Reducir otro volumen compatible después de respaldo.
* Ampliar el disco y ejecutar `pvresize`.

---

# Diagnóstico: growpart no cambia nada

Posibles causas:

* La partición ya ocupa todo el disco.
* El disco no fue ampliado.
* El kernel no detectó el nuevo tamaño.
* No existe espacio contiguo.
* Se indicó un número incorrecto.
* La tabla de particiones presenta errores.

Consultar:

```bash
lsblk
sudo fdisk -l /dev/sda
sudo parted /dev/sda print free
```

---

# Diagnóstico: pvresize no encuentra espacio

Consultar:

```bash
lsblk
sudo pvs
sudo parted /dev/sda print free
```

Posibles causas:

* La partición no fue ampliada.
* El kernel no releyó la tabla.
* Se ejecutó sobre el dispositivo equivocado.
* El PV ya utiliza todo el espacio.
* El disco no fue realmente ampliado.

---

# Diagnóstico: xfs_growfs muestra data size unchanged

Ejemplo:

```text
data size unchanged, skipping
```

Significa que XFS no detecta espacio adicional en el dispositivo subyacente.

Verificar:

```bash
sudo lvs
lsblk
df -Th /datos
```

Si el LV no fue ampliado, ejecutar:

```bash
sudo lvextend -L +10G /dev/vgdatos/lvdatos
```

Luego:

```bash
sudo xfs_growfs /datos
```

---

# Diagnóstico: resize2fs no encuentra espacio nuevo

```text
The filesystem is already ... blocks long. Nothing to do!
```

Verificar tamaño del dispositivo:

```bash
sudo blockdev --getsize64 /dev/vgdatos/lvdatos
```

Verificar LV:

```bash
sudo lvs
```

Puede indicar que:

* El LV no fue ampliado.
* Se usó el dispositivo incorrecto.
* El sistema ya ocupa todo el dispositivo.

---

# Diagnóstico: dispositivo ocupado

Al ampliar una partición, puede aparecer:

```text
Device or resource busy
```

Posibles soluciones:

* Utilizar `growpart`.
* Ejecutar `partprobe`.
* Revisar si el kernel admite relectura en línea.
* Programar reinicio.
* Ejecutar desde modo de rescate.

No debe desmontarse el sistema raíz de forma improvisada.

---

# Diagnóstico: no se reconoce el nuevo disco

Consultar:

```bash
lsblk
lsscsi
journalctl -k | tail -100
```

Escanear:

```bash
for host in /sys/class/scsi_host/host*; do
    echo "- - -" | sudo tee "$host/scan"
done
```

Verificar nuevamente:

```bash
lsblk
```

---

# Errores comunes

## Ampliar solo el LV

Incorrecto:

```bash
sudo lvextend -L +10G /dev/vgdatos/lvdatos
```

sin ampliar el sistema de archivos.

Correcto:

```bash
sudo lvextend -r -L +10G /dev/vgdatos/lvdatos
```

o ejecutar la herramienta correspondiente después.

---

## Utilizar `xfs_growfs` sobre el dispositivo

Incorrecto:

```bash
sudo xfs_growfs /dev/vgdatos/lvdatos
```

Correcto:

```bash
sudo xfs_growfs /datos
```

---

## Utilizar `resize2fs` sobre XFS

Incorrecto:

```bash
sudo resize2fs /dev/vgdatos/lvdatos
```

cuando el sistema es XFS.

Verificar primero:

```bash
findmnt -no FSTYPE /datos
```

---

## Utilizar todo el espacio libre sin planificar

```bash
sudo lvextend -r -l +100%FREE /dev/vgdatos/lvdatos
```

puede dejar el VG sin margen para otros volúmenes.

---

## Confundir tamaño adicional con tamaño final

```bash
-L +20G
```

agrega 20 GB.

```bash
-L 20G
```

establece un tamaño final de 20 GB.

---

## Ejecutar growpart con sintaxis incorrecta

Incorrecto:

```bash
sudo growpart /dev/sda3
```

Correcto:

```bash
sudo growpart /dev/sda 3
```

---

## Olvidar `pvresize`

Después de ampliar una partición LVM:

```bash
sudo growpart /dev/sda 3
```

debe ejecutarse:

```bash
sudo pvresize /dev/sda3
```

---

## Ampliar la partición equivocada

Antes:

```bash
lsblk -f
findmnt /
sudo pvs
```

Debe confirmarse la relación entre partición, PV, VG y LV.

---

## Modificar una tabla de particiones sin respaldo

Aunque la ampliación suele conservar datos, un error de dispositivo o número puede causar pérdida total.

---

## No comprobar espacio contiguo

Una partición tradicional no puede crecer si otra partición ocupa el espacio siguiente.

---

## No validar la aplicación

Después de ampliar almacenamiento de una base de datos debe comprobarse:

```bash
systemctl status servicio
df -Th ruta
journalctl -u servicio --since "10 minutes ago"
```

---

# Buenas prácticas

* Realiza un respaldo antes de modificar particiones.
* Identifica todas las capas antes de ampliar.
* Utiliza `findmnt` para ubicar el dispositivo correcto.
* Verifica el tipo de sistema de archivos.
* Consulta `VFree` antes de usar `lvextend`.
* Conserva espacio libre en el VG cuando sea posible.
* Utiliza `lvextend -r` para simplificar ampliaciones compatibles.
* Verifica siempre con `df`, `lvs`, `vgs` y `lsblk`.
* Utiliza `growpart -N` para revisar cuando esté disponible.
* Ejecuta `pvresize` después de ampliar una partición LVM.
* No confundas `-L +10G` con `-L 10G`.
* Documenta tamaños antes y después.
* Revisa logs del kernel.
* Mantén respaldos de metadatos LVM.
* Evita cambios en particiones críticas sin acceso a consola.
* Prueba el procedimiento en laboratorio.
* No reduzcas volúmenes durante una tarea de ampliación.
* No utilices comandos destructivos como `mkfs`, `pvcreate` o `wipefs` sobre volúmenes existentes.
* Valida servicios y aplicaciones después del cambio.

---

# Comandos principales

| Comando             | Función                                   |
| ------------------- | ----------------------------------------- |
| `df -Th`            | Consultar espacio del sistema de archivos |
| `lsblk`             | Mostrar discos, particiones y volúmenes   |
| `findmnt`           | Identificar origen y tipo                 |
| `pvs`               | Consultar Physical Volumes                |
| `vgs`               | Consultar espacio libre del VG            |
| `lvs`               | Consultar Logical Volumes                 |
| `lvextend`          | Ampliar Logical Volume                    |
| `xfs_growfs`        | Ampliar XFS                               |
| `resize2fs`         | Ampliar ext4                              |
| `growpart`          | Ampliar una partición                     |
| `parted resizepart` | Cambiar final de partición                |
| `pvresize`          | Ajustar tamaño del PV                     |
| `vgextend`          | Agregar PV a un VG                        |
| `partprobe`         | Solicitar relectura de particiones        |
| `blockdev`          | Consultar o releer dispositivos           |
| `vgcfgbackup`       | Respaldar metadatos LVM                   |
| `journalctl -k`     | Revisar eventos del kernel                |

---

# Tabla de procedimientos

| Escenario              | Procedimiento                                 |
| ---------------------- | --------------------------------------------- |
| VG tiene espacio libre | `lvextend` + ampliar filesystem               |
| Nuevo disco            | `pvcreate` + `vgextend` + `lvextend`          |
| Disco virtual creció   | rescan + `growpart` + `pvresize` + `lvextend` |
| Partición ext4 sin LVM | `growpart` + `resize2fs`                      |
| Partición XFS sin LVM  | `growpart` + `xfs_growfs`                     |
| XFS sobre LVM          | `lvextend` + `xfs_growfs`                     |
| ext4 sobre LVM         | `lvextend` + `resize2fs`                      |
| Ampliación automática  | `lvextend -r`                                 |

---

# Resumen

En este capítulo aprendiste a:

* Comprender las capas involucradas en una ampliación.
* Identificar discos, particiones, PV, VG, LV y sistemas de archivos.
* Determinar dónde se encuentra el espacio libre.
* Ampliar Logical Volumes con `lvextend`.
* Diferenciar tamaños finales y tamaños adicionales.
* Utilizar porcentajes y extents.
* Ampliar XFS con `xfs_growfs`.
* Ampliar ext4 con `resize2fs`.
* Automatizar la ampliación con `lvextend -r`.
* Agregar nuevos discos a un Volume Group.
* Ampliar discos virtuales, particiones y Physical Volumes.
* Utilizar `growpart`, `parted` y `pvresize`.
* Ampliar sistemas sin LVM.
* Trabajar con discos SCSI y NVMe.
* Ampliar swap y thin pools.
* Diagnosticar fallos comunes.
* Aplicar procedimientos seguros orientados a RHCSA.
