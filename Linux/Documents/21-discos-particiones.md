# 21. Administración de Discos y Particiones en Linux

La administración de discos es una de las tareas más importantes de un administrador Linux. Permite identificar dispositivos de almacenamiento, crear particiones, formatearlas, montarlas y garantizar que estén disponibles después de reiniciar el sistema.

En entornos de servidores, una mala administración de discos puede provocar pérdida de datos, indisponibilidad de servicios, llenado de sistemas de archivos y problemas de rendimiento.

Este capítulo cubre las herramientas fundamentales para trabajar con discos y particiones en Linux de forma segura.

---

# Objetivos de este capítulo

Al finalizar esta unidad serás capaz de:

* Identificar discos y particiones.
* Interpretar los nombres de dispositivos.
* Consultar información de almacenamiento.
* Crear y eliminar particiones.
* Formatear sistemas de archivos.
* Montar y desmontar particiones.
* Configurar montajes persistentes.
* Utilizar UUID y etiquetas.
* Analizar espacio disponible y uso de disco.
* Detectar errores comunes.
* Aplicar buenas prácticas de administración.

---

# Conceptos fundamentales

Antes de trabajar con discos, es importante diferenciar varios conceptos.

| Concepto            | Descripción                                        |
| ------------------- | -------------------------------------------------- |
| Disco               | Dispositivo físico o virtual de almacenamiento     |
| Partición           | División lógica dentro de un disco                 |
| Sistema de archivos | Estructura utilizada para organizar datos          |
| Punto de montaje    | Directorio donde se accede al sistema de archivos  |
| UUID                | Identificador único del sistema de archivos        |
| Etiqueta            | Nombre descriptivo asignado al sistema de archivos |
| Sector              | Unidad mínima de almacenamiento en el disco        |

---

# Nombres de dispositivos en Linux

Los discos aparecen dentro del directorio:

```text
/dev/
```

Ejemplos comunes:

```text
/dev/sda
/dev/sdb
/dev/vda
/dev/nvme0n1
```

---

# Discos SATA, SAS y USB

Generalmente se identifican como:

```text
/dev/sda
/dev/sdb
/dev/sdc
```

Las particiones se representan con números:

```text
/dev/sda1
/dev/sda2
/dev/sda3
```

---

# Discos virtuales

En entornos virtualizados es común encontrar:

```text
/dev/vda
/dev/vdb
```

Particiones:

```text
/dev/vda1
/dev/vda2
```

---

# Discos NVMe

Los discos NVMe utilizan una nomenclatura diferente:

```text
/dev/nvme0n1
/dev/nvme1n1
```

Particiones:

```text
/dev/nvme0n1p1
/dev/nvme0n1p2
```

La letra `p` separa el nombre del disco del número de la partición.

---

# Identificar discos con lsblk

El comando principal para visualizar discos es:

```bash id="disk001"
lsblk
```

Ejemplo:

```text id="disk002"
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda           8:0    0   500G  0 disk
├─sda1        8:1    0     1G  0 part /boot
├─sda2        8:2    0    50G  0 part /
└─sda3        8:3    0   449G  0 part /datos
```

---

# Columnas de lsblk

| Columna     | Descripción            |
| ----------- | ---------------------- |
| NAME        | Nombre del dispositivo |
| MAJ:MIN     | Número mayor y menor   |
| RM          | Dispositivo removible  |
| SIZE        | Tamaño                 |
| RO          | Solo lectura           |
| TYPE        | Tipo de dispositivo    |
| MOUNTPOINTS | Puntos de montaje      |

---

# Mostrar sistemas de archivos

```bash id="disk003"
lsblk -f
```

Ejemplo:

```text id="disk004"
NAME   FSTYPE FSVER LABEL UUID                                 MOUNTPOINTS
sda
├─sda1 xfs          boot  a1b2-c3d4                            /boot
├─sda2 xfs          root  62bfe9f1-8891-41c1-8f50-123456789abc /
└─sda3 ext4         datos 911d212a-77d6-45c1-9fd9-987654321abc /datos
```

---

# Mostrar columnas específicas

```bash id="disk005"
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
```

También:

```bash id="disk006"
lsblk -o NAME,MODEL,SERIAL,SIZE,TYPE
```

---

# Identificar discos con fdisk

```bash id="disk007"
sudo fdisk -l
```

Este comando muestra:

* Discos.
* Tamaños.
* Sectores.
* Particiones.
* Tipos de partición.
* Tabla de particiones.

---

# Consultar un disco específico

```bash id="disk008"
sudo fdisk -l /dev/sdb
```

---

# Ver espacio usado con df

```bash id="disk009"
df -h
```

Ejemplo:

```text id="disk010"
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2        50G   28G   22G  57% /
/dev/sda3       449G  210G  239G  47% /datos
```

---

# Columnas de df

| Columna    | Descripción          |
| ---------- | -------------------- |
| Filesystem | Sistema de archivos  |
| Size       | Tamaño total         |
| Used       | Espacio utilizado    |
| Avail      | Espacio disponible   |
| Use%       | Porcentaje utilizado |
| Mounted on | Punto de montaje     |

---

# Mostrar tipo de sistema de archivos

```bash id="disk011"
df -Th
```

---

# Consultar un punto específico

```bash id="disk012"
df -h /datos
```

---

# Ver uso de directorios con du

```bash id="disk013"
du -sh /var
```

Ver subdirectorios:

```bash id="disk014"
du -sh /var/*
```

Ordenar por tamaño:

```bash id="disk015"
du -sh /var/* 2>/dev/null | sort -h
```

---

# Ver los directorios más grandes

```bash id="disk016"
sudo du -xah / | sort -h | tail -20
```

La opción `-x` evita cruzar a otros sistemas de archivos.

---

# Tabla de particiones MBR y GPT

Los discos utilizan una tabla de particiones.

Los formatos principales son:

| Característica         | MBR                | GPT                   |
| ---------------------- | ------------------ | --------------------- |
| Nombre completo        | Master Boot Record | GUID Partition Table  |
| Tamaño máximo habitual | 2 TB               | Mucho mayor           |
| Particiones primarias  | Máximo 4           | Normalmente hasta 128 |
| Redundancia            | No                 | Sí                    |
| Uso moderno            | Legado             | Recomendado           |
| Compatibilidad UEFI    | Limitada           | Completa              |

Para discos modernos se recomienda GPT.

---

# Herramientas de particionado

Linux incluye varias herramientas.

| Herramienta | Uso                             |
| ----------- | ------------------------------- |
| `fdisk`     | MBR y GPT en versiones modernas |
| `gdisk`     | Especializada en GPT            |
| `parted`    | Particionado avanzado           |
| `cfdisk`    | Interfaz visual en terminal     |
| `sfdisk`    | Automatización mediante scripts |

---

# Advertencia antes de particionar

Crear, eliminar o modificar particiones puede provocar pérdida de datos.

Antes de continuar:

* Confirma el disco correcto.
* Realiza una copia de seguridad.
* Verifica que la partición no esté montada.
* Evita modificar discos de producción sin una ventana de mantenimiento.
* No trabajes sobre el disco raíz sin comprender completamente el impacto.

---

# Verificar el disco correcto

```bash id="disk017"
lsblk -o NAME,SIZE,MODEL,SERIAL,MOUNTPOINTS
```

También:

```bash id="disk018"
sudo fdisk -l
```

---

# Crear una partición con fdisk

Abrir el disco:

```bash id="disk019"
sudo fdisk /dev/sdb
```

Dentro de `fdisk`, los comandos principales son:

| Comando | Acción              |
| ------- | ------------------- |
| `m`     | Mostrar ayuda       |
| `p`     | Mostrar particiones |
| `n`     | Crear partición     |
| `d`     | Eliminar partición  |
| `t`     | Cambiar tipo        |
| `g`     | Crear tabla GPT     |
| `o`     | Crear tabla DOS/MBR |
| `w`     | Guardar cambios     |
| `q`     | Salir sin guardar   |

---

# Ejemplo: crear una partición

```text id="disk020"
Command (m for help): n
Partition number: 1
First sector: Enter
Last sector: +20G
```

Mostrar resultado:

```text id="disk021"
Command (m for help): p
```

Guardar:

```text id="disk022"
Command (m for help): w
```

---

# Informar al kernel de los cambios

Después de modificar la tabla de particiones:

```bash id="disk023"
sudo partprobe /dev/sdb
```

También puede utilizarse:

```bash id="disk024"
sudo udevadm settle
```

Verificar:

```bash id="disk025"
lsblk
```

---

# Crear una tabla GPT

```bash id="disk026"
sudo fdisk /dev/sdb
```

Dentro de `fdisk`:

```text id="disk027"
g
```

Luego crear la partición con:

```text id="disk028"
n
```

Finalmente guardar:

```text id="disk029"
w
```

---

# Eliminar una partición

Abrir el disco:

```bash id="disk030"
sudo fdisk /dev/sdb
```

Dentro:

```text id="disk031"
d
```

Seleccionar el número de partición y guardar:

```text id="disk032"
w
```

Esto elimina la definición de la partición. Los datos pueden quedar inaccesibles.

---

# Crear particiones con parted

Abrir el disco:

```bash id="disk033"
sudo parted /dev/sdb
```

Mostrar información:

```text id="disk034"
print
```

Crear tabla GPT:

```text id="disk035"
mklabel gpt
```

Crear una partición:

```text id="disk036"
mkpart primary ext4 1MiB 20GiB
```

Salir:

```text id="disk037"
quit
```

---

# Usar parted sin modo interactivo

```bash id="disk038"
sudo parted -s /dev/sdb mklabel gpt
```

Crear una partición:

```bash id="disk039"
sudo parted -s /dev/sdb mkpart primary ext4 1MiB 20GiB
```

---

# Verificar alineación

```bash id="disk040"
sudo parted /dev/sdb align-check optimal 1
```

Resultado esperado:

```text id="disk041"
1 aligned
```

---

# Crear un sistema de archivos

Después de crear una partición, debe formatearse.

Advertencia: formatear destruye los datos existentes de la partición.

---

# Crear sistema de archivos XFS

```bash id="disk042"
sudo mkfs.xfs /dev/sdb1
```

Forzar si existe una firma anterior:

```bash id="disk043"
sudo mkfs.xfs -f /dev/sdb1
```

Debe utilizarse con mucho cuidado.

---

# Crear sistema de archivos ext4

```bash id="disk044"
sudo mkfs.ext4 /dev/sdb1
```

---

# Crear sistema de archivos VFAT

```bash id="disk045"
sudo mkfs.vfat /dev/sdb1
```

Útil para compatibilidad con otros sistemas y particiones EFI.

---

# Crear sistema de archivos swap

```bash id="disk046"
sudo mkswap /dev/sdb1
```

Activar:

```bash id="disk047"
sudo swapon /dev/sdb1
```

---

# Sistemas de archivos comunes

| Sistema | Características                     |
| ------- | ----------------------------------- |
| XFS     | Predeterminado en RHEL y Fedora     |
| ext4    | Muy estable y ampliamente utilizado |
| Btrfs   | Snapshots y funciones avanzadas     |
| VFAT    | Compatibilidad amplia               |
| NTFS    | Sistemas Windows                    |
| Swap    | Memoria de intercambio              |

---

# Diferencias entre XFS y ext4

| Característica                    | XFS          | ext4                       |
| --------------------------------- | ------------ | -------------------------- |
| Rendimiento con archivos grandes  | Excelente    | Muy bueno                  |
| Crecimiento en línea              | Sí           | Sí                         |
| Reducción del sistema de archivos | No           | Sí, normalmente desmontado |
| Uso empresarial                   | Muy común    | Muy común                  |
| Herramienta de reparación         | `xfs_repair` | `fsck.ext4`                |

---

# Asignar una etiqueta

## XFS

```bash id="disk048"
sudo xfs_admin -L DATOS /dev/sdb1
```

## ext4

```bash id="disk049"
sudo e2label /dev/sdb1 DATOS
```

También puede asignarse al formatear:

```bash id="disk050"
sudo mkfs.ext4 -L DATOS /dev/sdb1
```

---

# Consultar UUID y etiquetas

```bash id="disk051"
sudo blkid
```

Ejemplo:

```text id="disk052"
/dev/sdb1: LABEL="DATOS" UUID="e22b34c9-6df4-4f24-a493-f0d7ba132687" TYPE="xfs"
```

Consultar un dispositivo específico:

```bash id="disk053"
sudo blkid /dev/sdb1
```

---

# Crear un punto de montaje

```bash id="disk054"
sudo mkdir -p /datos
```

Verificar:

```bash id="disk055"
ls -ld /datos
```

---

# Montar una partición

```bash id="disk056"
sudo mount /dev/sdb1 /datos
```

Verificar:

```bash id="disk057"
findmnt /datos
```

También:

```bash id="disk058"
df -h /datos
```

---

# Mostrar montajes actuales

```bash id="disk059"
findmnt
```

Mostrar un dispositivo:

```bash id="disk060"
findmnt /dev/sdb1
```

Mostrar un punto:

```bash id="disk061"
findmnt /datos
```

---

# Desmontar una partición

Por punto de montaje:

```bash id="disk062"
sudo umount /datos
```

Por dispositivo:

```bash id="disk063"
sudo umount /dev/sdb1
```

El comando correcto es `umount`, no `unmount`.

---

# Error: target is busy

Ejemplo:

```text id="disk064"
umount: /datos: target is busy
```

Esto significa que un proceso está utilizando el sistema de archivos.

---

# Identificar procesos que usan el punto de montaje

```bash id="disk065"
sudo lsof +D /datos
```

Para directorios muy grandes puede ser lento.

Alternativa:

```bash id="disk066"
sudo fuser -vm /datos
```

---

# Finalizar procesos que utilizan el montaje

```bash id="disk067"
sudo fuser -km /datos
```

Debe utilizarse con extremo cuidado, ya que finaliza procesos.

Luego:

```bash id="disk068"
sudo umount /datos
```

---

# Montaje diferido y forzado

Desmontaje diferido:

```bash id="disk069"
sudo umount -l /datos
```

La opción `-l` significa lazy.

Desmontaje forzado:

```bash id="disk070"
sudo umount -f /datos
```

No debe usarse como primera opción, especialmente en sistemas de archivos locales.

---

# Montaje persistente con /etc/fstab

Los montajes manuales desaparecen después de reiniciar.

Para que sean permanentes se utiliza:

```text
/etc/fstab
```

Antes de editar:

```bash id="disk071"
sudo cp /etc/fstab /etc/fstab.bak
```

Editar:

```bash id="disk072"
sudo nano /etc/fstab
```

---

# Estructura de /etc/fstab

```text id="disk073"
dispositivo  punto_montaje  tipo  opciones  dump  fsck
```

Ejemplo:

```fstab id="disk074"
/dev/sdb1  /datos  xfs  defaults  0  0
```

---

# Campos de fstab

| Campo            | Descripción                |
| ---------------- | -------------------------- |
| Dispositivo      | Partición, UUID o etiqueta |
| Punto de montaje | Directorio de acceso       |
| Tipo             | Sistema de archivos        |
| Opciones         | Opciones de montaje        |
| Dump             | Copias con dump            |
| Fsck             | Orden de verificación      |

---

# Usar UUID en fstab

Obtener UUID:

```bash id="disk075"
sudo blkid /dev/sdb1
```

Agregar:

```fstab id="disk076"
UUID=e22b34c9-6df4-4f24-a493-f0d7ba132687 /datos xfs defaults 0 0
```

El uso de UUID es preferible a `/dev/sdb1`, ya que el nombre del dispositivo puede cambiar.

---

# Usar etiqueta en fstab

```fstab id="disk077"
LABEL=DATOS /datos xfs defaults 0 0
```

---

# Probar fstab sin reiniciar

Después de editar:

```bash id="disk078"
sudo mount -a
```

Si no muestra errores, la configuración normalmente es válida.

Verificar:

```bash id="disk079"
findmnt --verify
```

También:

```bash id="disk080"
findmnt /datos
```

---

# Riesgo de un error en fstab

Un error en `/etc/fstab` puede provocar:

* Fallos durante el arranque.
* Entrada en modo de emergencia.
* Montajes incorrectos.
* Servicios que no inician.
* Pérdida temporal de acceso a datos.

Siempre debe ejecutarse:

```bash id="disk081"
sudo mount -a
```

antes de reiniciar.

---

# Opciones comunes de montaje

| Opción     | Descripción                     |
| ---------- | ------------------------------- |
| `defaults` | Opciones predeterminadas        |
| `rw`       | Lectura y escritura             |
| `ro`       | Solo lectura                    |
| `noexec`   | Impide ejecutar binarios        |
| `nosuid`   | Ignora SUID y SGID              |
| `nodev`    | No interpreta dispositivos      |
| `noatime`  | No actualiza tiempo de acceso   |
| `nofail`   | No bloquea el arranque si falla |
| `_netdev`  | Indica que depende de la red    |
| `user`     | Permite montaje por usuario     |
| `auto`     | Montar automáticamente          |
| `noauto`   | No montar automáticamente       |

---

# Ejemplo con opciones de seguridad

```fstab id="disk082"
UUID=e22b34c9-6df4-4f24-a493-f0d7ba132687 /datos xfs defaults,nodev,nosuid 0 0
```

---

# Montaje de solo lectura

Manual:

```bash id="disk083"
sudo mount -o ro /dev/sdb1 /datos
```

En fstab:

```fstab id="disk084"
UUID=e22b34c9-6df4-4f24-a493-f0d7ba132687 /datos xfs ro 0 0
```

---

# Remontar un sistema de archivos

Cambiar a solo lectura:

```bash id="disk085"
sudo mount -o remount,ro /datos
```

Cambiar a lectura y escritura:

```bash id="disk086"
sudo mount -o remount,rw /datos
```

---

# Ver opciones activas de montaje

```bash id="disk087"
findmnt -o TARGET,SOURCE,FSTYPE,OPTIONS
```

Para un punto específico:

```bash id="disk088"
findmnt -no OPTIONS /datos
```

---

# Revisar sistemas de archivos

## ext4

La herramienta general es:

```bash id="disk089"
sudo fsck /dev/sdb1
```

Específica para ext4:

```bash id="disk090"
sudo fsck.ext4 /dev/sdb1
```

La partición debe estar desmontada.

---

# Verificación sin realizar cambios

```bash id="disk091"
sudo fsck -n /dev/sdb1
```

---

# Reparar automáticamente

```bash id="disk092"
sudo fsck -y /dev/sdb1
```

La opción `-y` acepta todas las reparaciones. Debe utilizarse con precaución.

---

# Revisar XFS

XFS no se repara con el `fsck` tradicional.

Primero desmontar:

```bash id="disk093"
sudo umount /datos
```

Comprobar sin modificar:

```bash id="disk094"
sudo xfs_repair -n /dev/sdb1
```

Reparar:

```bash id="disk095"
sudo xfs_repair /dev/sdb1
```

---

# Consultar información de XFS

```bash id="disk096"
sudo xfs_info /datos
```

---

# Consultar información de ext4

```bash id="disk097"
sudo tune2fs -l /dev/sdb1
```

Mostrar información resumida:

```bash id="disk098"
sudo dumpe2fs -h /dev/sdb1
```

---

# Expandir un sistema de archivos XFS

Después de aumentar el tamaño del dispositivo o volumen:

```bash id="disk099"
sudo xfs_growfs /datos
```

Verificar:

```bash id="disk100"
df -h /datos
```

XFS puede crecer, pero no puede reducirse.

---

# Expandir ext4

Primero ampliar la partición o volumen.

Luego:

```bash id="disk101"
sudo resize2fs /dev/sdb1
```

---

# Expandir una partición con growpart

Instalar herramientas si es necesario:

```bash id="disk102"
sudo dnf install cloud-utils-growpart
```

Expandir la primera partición de `/dev/sdb`:

```bash id="disk103"
sudo growpart /dev/sdb 1
```

Después ampliar el sistema de archivos.

XFS:

```bash id="disk104"
sudo xfs_growfs /datos
```

ext4:

```bash id="disk105"
sudo resize2fs /dev/sdb1
```

---

# Ver discos detectados por el kernel

```bash id="disk106"
journalctl -k | grep -Ei "sd[a-z]|nvme|disk"
```

También:

```bash id="disk107"
dmesg | grep -Ei "sd[a-z]|nvme|disk"
```

---

# Escanear nuevos discos SCSI

En servidores o máquinas virtuales puede ser necesario solicitar un nuevo escaneo.

Identificar hosts:

```bash id="disk108"
ls /sys/class/scsi_host/
```

Escanear:

```bash id="disk109"
for host in /sys/class/scsi_host/host*; do
    echo "- - -" | sudo tee "$host/scan"
done
```

Verificar:

```bash id="disk110"
lsblk
```

---

# Ver información SMART

Instalar herramientas:

```bash id="disk111"
sudo dnf install smartmontools
```

Ver estado general:

```bash id="disk112"
sudo smartctl -H /dev/sda
```

Información completa:

```bash id="disk113"
sudo smartctl -a /dev/sda
```

---

# Pruebas SMART

Prueba corta:

```bash id="disk114"
sudo smartctl -t short /dev/sda
```

Ver resultado:

```bash id="disk115"
sudo smartctl -l selftest /dev/sda
```

Prueba larga:

```bash id="disk116"
sudo smartctl -t long /dev/sda
```

---

# Consultar inodos

El espacio disponible no es el único límite. También pueden agotarse los inodos.

```bash id="disk117"
df -i
```

Ejemplo:

```text id="disk118"
Filesystem      Inodes  IUsed   IFree IUse% Mounted on
/dev/sda2      2621440 250000 2371440   10% /
```

---

# Disco con espacio libre pero sin poder crear archivos

Una causa posible es el agotamiento de inodos.

Verificar:

```bash id="disk119"
df -i /var
```

Buscar directorios con muchos archivos:

```bash id="disk120"
sudo find /var -xdev -type f | wc -l
```

---

# Detectar archivos eliminados que siguen ocupando espacio

Un archivo eliminado puede seguir ocupando espacio si un proceso lo mantiene abierto.

```bash id="disk121"
sudo lsof +L1
```

También:

```bash id="disk122"
sudo lsof | grep deleted
```

La solución normalmente consiste en reiniciar o recargar el proceso responsable, no el servidor completo.

---

# Sincronizar datos pendientes

```bash id="disk123"
sync
```

Este comando solicita que los datos pendientes se escriban en disco.

Es útil antes de:

* Desconectar almacenamiento externo.
* Apagar de forma controlada.
* Realizar ciertas operaciones de mantenimiento.

---

# Limpiar firmas de sistemas de archivos

Ver firmas:

```bash id="disk124"
sudo wipefs /dev/sdb1
```

Eliminar firmas:

```bash id="disk125"
sudo wipefs -a /dev/sdb1
```

Advertencia: esto puede hacer que los datos sean inaccesibles.

---

# Copiar una tabla de particiones

Con `sfdisk`:

```bash id="disk126"
sudo sfdisk -d /dev/sda > particiones_sda.txt
```

Restaurar en otro disco:

```bash id="disk127"
sudo sfdisk /dev/sdb < particiones_sda.txt
```

El disco de destino debe tener capacidad suficiente.

---

# Ver dispositivos por UUID

```bash id="disk128"
ls -l /dev/disk/by-uuid/
```

Por etiqueta:

```bash id="disk129"
ls -l /dev/disk/by-label/
```

Por identificador persistente:

```bash id="disk130"
ls -l /dev/disk/by-id/
```

---

# Permisos del punto de montaje

Después de montar, el propietario del directorio raíz del sistema de archivos puede ser `root`.

Verificar:

```bash id="disk131"
ls -ld /datos
```

Cambiar propietario:

```bash id="disk132"
sudo chown alejandro:dba /datos
```

Permisos compartidos:

```bash id="disk133"
sudo chmod 2770 /datos
```

El bit SGID permite que los archivos hereden el grupo del directorio.

---

# SELinux y nuevos puntos de montaje

En Fedora y RHEL, los permisos tradicionales pueden ser correctos, pero SELinux puede impedir el acceso.

Ver contexto:

```bash id="disk134"
ls -Zd /datos
```

Asignar un contexto persistente, según la aplicación:

```bash id="disk135"
sudo semanage fcontext -a -t var_t "/datos(/.*)?"
```

Aplicar:

```bash id="disk136"
sudo restorecon -Rv /datos
```

El tipo de contexto debe elegirse de acuerdo con el servicio que utilizará el directorio.

---

# Ejemplo para base de datos

No debe utilizarse un contexto genérico sin validar la aplicación.

Para PostgreSQL, podría requerirse un tipo específico relacionado con PostgreSQL.

Para servicios web, puede requerirse:

```text
httpd_sys_content_t
```

Siempre debe consultarse la política correspondiente.

---

# Flujo completo para agregar un disco

## Paso 1: Identificar el disco

```bash id="disk137"
lsblk -o NAME,SIZE,MODEL,SERIAL,TYPE,MOUNTPOINTS
```

## Paso 2: Crear tabla GPT y partición

```bash id="disk138"
sudo parted -s /dev/sdb mklabel gpt
sudo parted -s /dev/sdb mkpart primary xfs 1MiB 100%
```

## Paso 3: Informar al kernel

```bash id="disk139"
sudo partprobe /dev/sdb
```

## Paso 4: Formatear

```bash id="disk140"
sudo mkfs.xfs -L DATOS /dev/sdb1
```

## Paso 5: Crear punto de montaje

```bash id="disk141"
sudo mkdir -p /datos
```

## Paso 6: Obtener UUID

```bash id="disk142"
sudo blkid /dev/sdb1
```

## Paso 7: Agregar a fstab

```fstab id="disk143"
UUID=UUID_OBTENIDO /datos xfs defaults 0 0
```

## Paso 8: Probar

```bash id="disk144"
sudo mount -a
sudo findmnt --verify
```

## Paso 9: Verificar

```bash id="disk145"
findmnt /datos
df -h /datos
```

## Paso 10: Configurar permisos

```bash id="disk146"
sudo chown alejandro:dba /datos
sudo chmod 2770 /datos
```

---

# Laboratorio práctico

## Ejercicio 1: Identificar discos

```bash id="labdisk001"
lsblk
```

Mostrar sistemas de archivos:

```bash id="labdisk002"
lsblk -f
```

---

## Ejercicio 2: Consultar tabla de particiones

```bash id="labdisk003"
sudo fdisk -l
```

---

## Ejercicio 3: Crear una partición

En un disco de laboratorio:

```bash id="labdisk004"
sudo fdisk /dev/sdb
```

Crear una partición de 5 GB usando:

```text id="labdisk005"
n
```

Guardar:

```text id="labdisk006"
w
```

---

## Ejercicio 4: Formatear la partición

```bash id="labdisk007"
sudo mkfs.xfs -L LABDATOS /dev/sdb1
```

---

## Ejercicio 5: Crear punto de montaje

```bash id="labdisk008"
sudo mkdir -p /mnt/labdatos
```

---

## Ejercicio 6: Montar manualmente

```bash id="labdisk009"
sudo mount /dev/sdb1 /mnt/labdatos
```

Verificar:

```bash id="labdisk010"
findmnt /mnt/labdatos
```

---

## Ejercicio 7: Crear un archivo

```bash id="labdisk011"
sudo touch /mnt/labdatos/prueba.txt
```

Verificar:

```bash id="labdisk012"
ls -l /mnt/labdatos
```

---

## Ejercicio 8: Desmontar

```bash id="labdisk013"
sudo umount /mnt/labdatos
```

---

## Ejercicio 9: Configurar montaje persistente

Obtener UUID:

```bash id="labdisk014"
sudo blkid /dev/sdb1
```

Editar:

```bash id="labdisk015"
sudo nano /etc/fstab
```

Agregar:

```fstab id="labdisk016"
UUID=UUID_OBTENIDO /mnt/labdatos xfs defaults 0 0
```

Probar:

```bash id="labdisk017"
sudo mount -a
```

---

## Ejercicio 10: Validar fstab

```bash id="labdisk018"
sudo findmnt --verify
```

Verificar:

```bash id="labdisk019"
findmnt /mnt/labdatos
```

---

## Ejercicio 11: Analizar uso de disco

```bash id="labdisk020"
df -h
```

Directorios más grandes de `/var`:

```bash id="labdisk021"
sudo du -xhd1 /var | sort -h
```

---

## Ejercicio 12: Consultar inodos

```bash id="labdisk022"
df -i
```

---

# Diagnóstico de disco lleno

Cuando un sistema de archivos está lleno, sigue este procedimiento.

## Paso 1: Verificar uso

```bash id="diagdisk001"
df -h
```

## Paso 2: Revisar inodos

```bash id="diagdisk002"
df -i
```

## Paso 3: Identificar directorios grandes

```bash id="diagdisk003"
sudo du -xhd1 / | sort -h
```

## Paso 4: Profundizar en el directorio

```bash id="diagdisk004"
sudo du -xhd1 /var | sort -h
```

## Paso 5: Buscar archivos grandes

```bash id="diagdisk005"
sudo find /var -xdev -type f -size +1G -printf '%s %p\n' | sort -n
```

## Paso 6: Buscar archivos eliminados abiertos

```bash id="diagdisk006"
sudo lsof +L1
```

## Paso 7: Revisar logs

```bash id="diagdisk007"
sudo du -sh /var/log/*
```

## Paso 8: Limpiar de forma segura

No eliminar archivos sin confirmar su función.

Debe aplicarse:

* Rotación de logs.
* Retención.
* Limpieza de respaldos antiguos.
* Depuración de archivos temporales.
* Expansión del almacenamiento si corresponde.

---

# Diagnóstico de una partición que no monta

## Paso 1: Verificar que existe

```bash id="diagdisk008"
lsblk -f
```

## Paso 2: Confirmar UUID

```bash id="diagdisk009"
sudo blkid
```

## Paso 3: Revisar fstab

```bash id="diagdisk010"
cat /etc/fstab
```

## Paso 4: Validar configuración

```bash id="diagdisk011"
sudo findmnt --verify
```

## Paso 5: Probar montaje

```bash id="diagdisk012"
sudo mount -av
```

## Paso 6: Consultar logs

```bash id="diagdisk013"
journalctl -b | grep -i mount
```

## Paso 7: Verificar sistema de archivos

Para ext4:

```bash id="diagdisk014"
sudo fsck -n /dev/sdb1
```

Para XFS:

```bash id="diagdisk015"
sudo xfs_repair -n /dev/sdb1
```

---

# Comandos más utilizados

| Comando      | Descripción                     |
| ------------ | ------------------------------- |
| `lsblk`      | Listar discos y particiones     |
| `blkid`      | Mostrar UUID y tipo             |
| `fdisk`      | Administrar particiones         |
| `parted`     | Particionado avanzado           |
| `mkfs`       | Crear sistema de archivos       |
| `mount`      | Montar sistema de archivos      |
| `umount`     | Desmontar                       |
| `findmnt`    | Consultar montajes              |
| `df`         | Ver uso de sistemas de archivos |
| `du`         | Ver uso de directorios          |
| `fsck`       | Verificar sistemas de archivos  |
| `xfs_repair` | Reparar XFS                     |
| `resize2fs`  | Redimensionar ext4              |
| `xfs_growfs` | Expandir XFS                    |
| `partprobe`  | Informar cambios de particiones |
| `smartctl`   | Consultar salud del disco       |
| `wipefs`     | Consultar o borrar firmas       |

---

# Archivos y directorios importantes

| Ruta                  | Función                          |
| --------------------- | -------------------------------- |
| `/dev/`               | Dispositivos                     |
| `/etc/fstab`          | Montajes persistentes            |
| `/proc/mounts`        | Montajes conocidos por el kernel |
| `/dev/disk/by-uuid/`  | Enlaces por UUID                 |
| `/dev/disk/by-label/` | Enlaces por etiqueta             |
| `/dev/disk/by-id/`    | Identificadores persistentes     |
| `/sys/block/`         | Información de bloques           |
| `/mnt/`               | Montajes temporales              |
| `/media/`             | Montajes removibles              |

---

# Buenas prácticas

* Realiza respaldos antes de modificar particiones.
* Confirma siempre el dispositivo correcto.
* Utiliza `lsblk`, `blkid` y `fdisk -l` antes de actuar.
* Prefiere GPT en discos modernos.
* Utiliza UUID en `/etc/fstab`.
* Crea una copia de `/etc/fstab` antes de modificarlo.
* Ejecuta `mount -a` y `findmnt --verify` antes de reiniciar.
* No ejecutes `fsck` sobre una partición montada en lectura y escritura.
* No utilices `mkfs`, `wipefs` o `fdisk` sin confirmar el disco.
* Monitorea tanto espacio como inodos.
* Utiliza `lsof +L1` cuando el uso real no coincide con `du`.
* Configura permisos y contextos SELinux después de montar.
* Documenta la finalidad de cada disco y punto de montaje.
* Utiliza etiquetas descriptivas.
* Supervisa la salud de los discos con SMART.
* Mantén una estrategia de respaldo independiente del almacenamiento local.

---

# Errores comunes

## Confundir el disco correcto

Ejecutar:

```bash id="errdisk001"
sudo mkfs.xfs /dev/sda1
```

sobre el dispositivo equivocado puede destruir el sistema.

Antes de formatear:

```bash id="errdisk002"
lsblk -o NAME,SIZE,MODEL,SERIAL,FSTYPE,MOUNTPOINTS
```

---

## Editar fstab y reiniciar sin probar

Incorrecto:

```text
Editar /etc/fstab y reiniciar inmediatamente.
```

Correcto:

```bash id="errdisk003"
sudo mount -a
sudo findmnt --verify
```

---

## Utilizar el nombre del dispositivo en lugar del UUID

Una entrada como:

```fstab id="errdisk004"
/dev/sdb1 /datos xfs defaults 0 0
```

puede fallar si el nombre cambia.

Es preferible:

```fstab id="errdisk005"
UUID=e22b34c9-6df4-4f24-a493-f0d7ba132687 /datos xfs defaults 0 0
```

---

## Ejecutar fsck sobre una partición montada

Incorrecto:

```bash id="errdisk006"
sudo fsck /dev/sdb1
```

mientras `/dev/sdb1` está montada.

Debe desmontarse primero:

```bash id="errdisk007"
sudo umount /datos
sudo fsck /dev/sdb1
```

---

## Pensar que df y du siempre coinciden

`df` muestra el uso del sistema de archivos.

`du` suma el espacio visible de archivos y directorios.

Pueden diferir por:

* Archivos eliminados todavía abiertos.
* Bloques reservados.
* Montajes ocultos.
* Permisos de lectura.
* Metadatos del sistema de archivos.

Buscar archivos eliminados abiertos:

```bash id="errdisk008"
sudo lsof +L1
```

---

## El punto de montaje contiene archivos previos

Si `/datos` contiene archivos y luego se monta una partición encima, esos archivos quedan temporalmente ocultos.

Antes de montar:

```bash id="errdisk009"
ls -la /datos
```

El punto de montaje debería estar vacío.

---

## Utilizar xfs_growfs sobre el dispositivo

Incorrecto:

```bash id="errdisk010"
sudo xfs_growfs /dev/sdb1
```

Normalmente debe utilizarse el punto de montaje:

```bash id="errdisk011"
sudo xfs_growfs /datos
```

---

## Intentar reducir XFS

XFS no admite reducción directa.

La alternativa habitual es:

1. Crear un sistema de archivos más pequeño.
2. Copiar los datos.
3. Verificar la copia.
4. Cambiar el montaje.
5. Retirar el volumen anterior.

---

# Resumen

En este capítulo aprendiste a:

* Identificar discos, particiones y sistemas de archivos.
* Interpretar dispositivos SATA, virtuales y NVMe.
* Consultar almacenamiento con `lsblk`, `fdisk`, `df`, `du` y `blkid`.
* Comprender las diferencias entre MBR y GPT.
* Crear y eliminar particiones con `fdisk` y `parted`.
* Formatear sistemas de archivos XFS y ext4.
* Montar y desmontar particiones.
* Configurar montajes persistentes en `/etc/fstab`.
* Utilizar UUID y etiquetas.
* Verificar sistemas de archivos.
* Expandir XFS y ext4.
* Analizar discos llenos, inodos y archivos eliminados abiertos.
* Consultar la salud de discos con SMART.
* Configurar permisos y contextos SELinux.
* Aplicar buenas prácticas para proteger la integridad y disponibilidad de los datos.
