# 22. Sistemas de Archivos en Linux

Un sistema de archivos define la manera en que Linux organiza, almacena, identifica y recupera los datos dentro de un dispositivo de almacenamiento.

Cada partición, volumen lógico, dispositivo USB o disco utilizado por Linux necesita una estructura que permita administrar archivos, directorios, permisos, metadatos, espacio libre y recuperación ante errores.

Los sistemas de archivos más utilizados en Linux incluyen:

* XFS.
* ext4.
* Btrfs.
* VFAT.
* NTFS.
* tmpfs.
* NFS.
* ISO 9660.

Comprender sus características, límites y herramientas de administración es esencial para seleccionar la opción adecuada y mantener la integridad de los datos.

---

# Objetivos de este capítulo

Al finalizar esta unidad serás capaz de:

* Comprender qué es un sistema de archivos.
* Diferenciar entre sistemas de archivos locales, temporales y de red.
* Identificar el sistema de archivos utilizado por cada dispositivo.
* Crear sistemas de archivos XFS y ext4.
* Consultar sus propiedades y metadatos.
* Verificar y reparar sistemas de archivos.
* Administrar etiquetas y UUID.
* Comprender el funcionamiento del journaling.
* Analizar espacio, bloques e inodos.
* Expandir sistemas de archivos.
* Seleccionar el sistema de archivos adecuado.
* Aplicar buenas prácticas de administración y recuperación.

---

# ¿Qué es un sistema de archivos?

Un sistema de archivos es una estructura lógica que permite almacenar y organizar información dentro de un dispositivo.

Administra elementos como:

* Archivos.
* Directorios.
* Nombres.
* Permisos.
* Propietarios.
* Fechas.
* Enlaces.
* Bloques de datos.
* Inodos.
* Espacio libre.
* Metadatos.

Sin un sistema de archivos, el sistema operativo solamente vería una colección de bloques de almacenamiento sin organización.

---

# Flujo básico de almacenamiento

El flujo habitual es:

```text
Disco físico o virtual
        ↓
Tabla de particiones
        ↓
Partición o volumen lógico
        ↓
Sistema de archivos
        ↓
Punto de montaje
        ↓
Archivos y directorios
```

Ejemplo:

```text
/dev/sdb
   ↓
/dev/sdb1
   ↓
XFS
   ↓
/datos
   ↓
/datos/archivos
```

---

# Sistemas de archivos locales

Los sistemas de archivos locales se almacenan directamente en discos conectados al servidor.

Ejemplos:

* XFS.
* ext4.
* Btrfs.
* VFAT.
* NTFS.
* exFAT.

---

# Sistemas de archivos temporales

Se almacenan principalmente en memoria y su contenido puede desaparecer después de reiniciar.

Ejemplos:

* tmpfs.
* devtmpfs.
* ramfs.

---

# Sistemas de archivos de red

Permiten acceder a archivos almacenados en otro servidor.

Ejemplos:

* NFS.
* CIFS o SMB.
* SSHFS.

---

# Sistemas de archivos virtuales

No representan almacenamiento físico tradicional. Exponen información del kernel, procesos y dispositivos.

Ejemplos:

| Sistema    | Punto habitual         | Función                              |
| ---------- | ---------------------- | ------------------------------------ |
| proc       | `/proc`                | Información de procesos y kernel     |
| sysfs      | `/sys`                 | Información de dispositivos y kernel |
| devtmpfs   | `/dev`                 | Dispositivos del sistema             |
| cgroup2    | `/sys/fs/cgroup`       | Control de recursos                  |
| securityfs | `/sys/kernel/security` | Funciones de seguridad               |

---

# Sistemas de archivos comunes en Linux

| Sistema  | Uso principal              | Características                        |
| -------- | -------------------------- | -------------------------------------- |
| XFS      | Servidores empresariales   | Alto rendimiento y escalabilidad       |
| ext4     | Uso general                | Estable, compatible y flexible         |
| Btrfs    | Sistemas modernos          | Snapshots, checksums y subvolúmenes    |
| VFAT     | USB y particiones EFI      | Alta compatibilidad                    |
| NTFS     | Compatibilidad con Windows | Uso compartido con Windows             |
| exFAT    | Memorias USB grandes       | Compatible con varios sistemas         |
| tmpfs    | Archivos temporales        | Utiliza memoria                        |
| NFS      | Almacenamiento de red      | Compartición entre sistemas Unix/Linux |
| ISO 9660 | Medios ópticos             | CD y DVD                               |

---

# Identificar sistemas de archivos

## lsblk

```bash id="fs001"
lsblk -f
```

Ejemplo:

```text id="fs002"
NAME   FSTYPE FSVER LABEL UUID                                 MOUNTPOINTS
sda
├─sda1 vfat   FAT32       7C2A-11D0                            /boot/efi
├─sda2 xfs                90afee9b-d3c7-4ec1-97fa-3df52abb3181 /boot
└─sda3 xfs                f9c74e36-e235-481d-b473-f6d78de52211 /
sdb
└─sdb1 ext4   1.0   DATOS 4cb0c22c-129d-45a6-a19e-c1f78285a113 /datos
```

---

# Consultar con blkid

```bash id="fs003"
sudo blkid
```

Dispositivo específico:

```bash id="fs004"
sudo blkid /dev/sdb1
```

Ejemplo:

```text id="fs005"
/dev/sdb1: LABEL="DATOS" UUID="4cb0c22c-129d-45a6-a19e-c1f78285a113" BLOCK_SIZE="4096" TYPE="ext4"
```

---

# Consultar sistemas montados

```bash id="fs006"
findmnt
```

Mostrar columnas específicas:

```bash id="fs007"
findmnt -o SOURCE,TARGET,FSTYPE,OPTIONS
```

Consultar un punto:

```bash id="fs008"
findmnt /datos
```

---

# Consultar con df

```bash id="fs009"
df -Th
```

Ejemplo:

```text id="fs010"
Filesystem     Type   Size  Used Avail Use% Mounted on
/dev/sda3      xfs     80G   35G   45G  44% /
/dev/sdb1      ext4   500G  120G  355G  26% /datos
tmpfs          tmpfs   16G     0   16G   0% /dev/shm
```

---

# Consultar el tipo con file

```bash id="fs011"
sudo file -s /dev/sdb1
```

Ejemplo:

```text id="fs012"
/dev/sdb1: Linux rev 1.0 ext4 filesystem data, UUID=...
```

---

# Firmas de sistemas de archivos

Los dispositivos contienen firmas que identifican el sistema de archivos.

Consultar firmas:

```bash id="fs013"
sudo wipefs /dev/sdb1
```

Ejemplo:

```text id="fs014"
DEVICE OFFSET TYPE UUID                                 LABEL
sdb1   0x438  ext4 4cb0c22c-129d-45a6-a19e-c1f78285a113 DATOS
```

Eliminar firmas:

```bash id="fs015"
sudo wipefs -a /dev/sdb1
```

Este comando puede hacer que los datos sean inaccesibles y debe utilizarse con extremo cuidado.

---

# ¿Qué es un bloque?

Los sistemas de archivos dividen el almacenamiento en unidades llamadas bloques.

Un bloque puede almacenar:

* Datos de archivos.
* Directorios.
* Metadatos.
* Información de control.

Tamaños comunes:

```text
1024 bytes
2048 bytes
4096 bytes
```

El tamaño de bloque más común es:

```text
4096 bytes
```

---

# Consultar tamaño de bloque

Con `stat`:

```bash id="fs016"
stat -f /datos
```

Con `tune2fs` para ext4:

```bash id="fs017"
sudo tune2fs -l /dev/sdb1 | grep "Block size"
```

Con XFS:

```bash id="fs018"
xfs_info /datos
```

---

# ¿Qué es un inodo?

Un inodo almacena metadatos relacionados con un archivo.

Incluye:

* Tipo de archivo.
* Permisos.
* UID.
* GID.
* Tamaño.
* Fechas.
* Número de enlaces.
* Ubicación de los bloques.

El nombre del archivo no se almacena directamente dentro del inodo. El directorio relaciona el nombre con el número de inodo.

---

# Ver el número de inodo

```bash id="fs019"
ls -li archivo.txt
```

Ejemplo:

```text id="fs020"
1048577 -rw-r--r--. 1 alejandro usuarios 1250 Jul 28 10:00 archivo.txt
```

El primer número es el inodo.

---

# Consultar información con stat

```bash id="fs021"
stat archivo.txt
```

Ejemplo:

```text id="fs022"
File: archivo.txt
Size: 1250
Blocks: 8
IO Block: 4096 regular file
Device: 8,17
Inode: 1048577
Links: 1
Access: (0644/-rw-r--r--)
Uid: (1000/alejandro)
Gid: (1000/alejandro)
```

---

# Consultar uso de inodos

```bash id="fs023"
df -i
```

Para un punto específico:

```bash id="fs024"
df -i /datos
```

Ejemplo:

```text id="fs025"
Filesystem      Inodes  IUsed   IFree IUse% Mounted on
/dev/sdb1      32768000 25000 32743000    1% /datos
```

---

# Agotamiento de inodos

Un sistema puede tener espacio disponible y aun así no permitir crear archivos si se agotaron los inodos.

Síntoma:

```text
No space left on device
```

Aunque `df -h` muestre espacio libre.

Verificar:

```bash id="fs026"
df -h /datos
df -i /datos
```

---

# Buscar directorios con muchos archivos

```bash id="fs027"
sudo find /var -xdev -type f | wc -l
```

Contar elementos por directorio:

```bash id="fs028"
sudo du --inodes -d 1 /var 2>/dev/null | sort -n
```

---

# ¿Qué es journaling?

El journaling es una técnica utilizada para registrar cambios pendientes antes de aplicarlos completamente al sistema de archivos.

Su objetivo es facilitar la recuperación después de:

* Apagones.
* Reinicios inesperados.
* Fallos del kernel.
* Caídas del servidor.
* Desconexión accidental del almacenamiento.

XFS y ext4 utilizan journaling.

---

# Funcionamiento simplificado

```text
1. Se registra la operación en el journal.
2. Se aplican los cambios al sistema de archivos.
3. La operación se marca como completada.
```

Si el servidor falla, el sistema puede revisar el journal y recuperar la consistencia con mayor rapidez.

---

# Journaling no es un respaldo

El journal ayuda a mantener la consistencia estructural.

No protege contra:

* Eliminación accidental.
* Corrupción de la aplicación.
* Malware.
* Sobrescritura de datos.
* Errores humanos.
* Fallo físico del disco.

Siempre debe existir una estrategia de respaldo.

---

# XFS

XFS es un sistema de archivos de alto rendimiento diseñado para manejar grandes volúmenes y cargas intensivas.

Es utilizado ampliamente en:

* RHEL.
* Rocky Linux.
* AlmaLinux.
* Fedora.
* Servidores de bases de datos.
* Sistemas con archivos grandes.
* Almacenamiento empresarial.

---

# Características de XFS

* Journaling.
* Alto rendimiento.
* Escalabilidad.
* Soporte para archivos grandes.
* Crecimiento en línea.
* Desfragmentación en línea.
* Cuotas.
* Copias y restauración con herramientas específicas.
* Allocation Groups para paralelismo.

---

# Limitación principal de XFS

XFS puede crecer, pero no puede reducirse directamente.

Si se necesita un sistema XFS más pequeño, normalmente se debe:

1. Crear un nuevo sistema de archivos.
2. Copiar los datos.
3. Verificar la copia.
4. Modificar el montaje.
5. Retirar el sistema anterior.

---

# Crear XFS

```bash id="fs029"
sudo mkfs.xfs /dev/sdb1
```

Con etiqueta:

```bash id="fs030"
sudo mkfs.xfs -L DATOS /dev/sdb1
```

Forzar si existe una firma:

```bash id="fs031"
sudo mkfs.xfs -f /dev/sdb1
```

Debe utilizarse solamente después de verificar el dispositivo.

---

# Consultar XFS

Con el sistema montado:

```bash id="fs032"
xfs_info /datos
```

Ejemplo:

```text id="fs033"
meta-data=/dev/sdb1              isize=512    agcount=4, agsize=...
data     =                       bsize=4096   blocks=...
naming   =version 2              bsize=4096
log      =internal               bsize=4096
realtime =none
```

---

# Consultar etiqueta de XFS

```bash id="fs034"
sudo xfs_admin -l /dev/sdb1
```

Asignar etiqueta:

```bash id="fs035"
sudo xfs_admin -L DATOS /dev/sdb1
```

Eliminar etiqueta:

```bash id="fs036"
sudo xfs_admin -L "" /dev/sdb1
```

---

# Verificar XFS

La comprobación sin cambios se realiza con:

```bash id="fs037"
sudo xfs_repair -n /dev/sdb1
```

La opción `-n` realiza una revisión sin modificar.

La partición debe estar desmontada.

---

# Reparar XFS

```bash id="fs038"
sudo umount /datos
sudo xfs_repair /dev/sdb1
```

---

# Journal dañado en XFS

En ciertos casos, `xfs_repair` puede solicitar la opción:

```bash id="fs039"
sudo xfs_repair -L /dev/sdb1
```

La opción `-L` elimina el log de XFS y puede provocar pérdida de las últimas operaciones pendientes.

Debe utilizarse únicamente como último recurso y después de revisar el diagnóstico.

---

# Expandir XFS

Primero debe ampliarse el dispositivo subyacente, partición o volumen lógico.

Luego:

```bash id="fs040"
sudo xfs_growfs /datos
```

Utilizar todo el espacio disponible:

```bash id="fs041"
sudo xfs_growfs -d /datos
```

Verificar:

```bash id="fs042"
df -h /datos
```

---

# Desfragmentar XFS

Consultar fragmentación:

```bash id="fs043"
sudo xfs_db -c frag -r /dev/sdb1
```

Desfragmentar un archivo:

```bash id="fs044"
sudo xfs_fsr /datos/archivo_grande
```

Desfragmentar un punto de montaje:

```bash id="fs045"
sudo xfs_fsr /datos
```

La desfragmentación no debe ejecutarse rutinariamente sin evidencia de necesidad.

---

# Congelar un sistema XFS

Congelar temporalmente:

```bash id="fs046"
sudo xfs_freeze -f /datos
```

Descongelar:

```bash id="fs047"
sudo xfs_freeze -u /datos
```

Se utiliza en ciertos escenarios de snapshots o respaldos consistentes.

No debe dejarse congelado durante largos periodos.

---

# Copias con xfsdump

Instalar herramientas:

```bash id="fs048"
sudo dnf install xfsdump
```

Realizar respaldo:

```bash id="fs049"
sudo xfsdump -f /respaldos/datos.dump /datos
```

Restaurar:

```bash id="fs050"
sudo xfsrestore -f /respaldos/datos.dump /restauracion
```

---

# ext4

ext4 significa Fourth Extended Filesystem.

Es uno de los sistemas de archivos más utilizados en Linux debido a su estabilidad, compatibilidad y facilidad de administración.

---

# Características de ext4

* Journaling.
* Compatibilidad amplia.
* Buen rendimiento.
* Soporte para archivos grandes.
* Crecimiento en línea.
* Posibilidad de reducción fuera de línea.
* Extents.
* Checksum del journal.
* Herramientas maduras de recuperación.

---

# Crear ext4

```bash id="fs051"
sudo mkfs.ext4 /dev/sdb1
```

Con etiqueta:

```bash id="fs052"
sudo mkfs.ext4 -L DATOS /dev/sdb1
```

Mostrar información durante el formateo:

```bash id="fs053"
sudo mkfs.ext4 -v /dev/sdb1
```

---

# Crear ext4 con porcentaje reservado

Por defecto, ext4 puede reservar una parte del espacio para root.

Crear con 1% reservado:

```bash id="fs054"
sudo mkfs.ext4 -m 1 /dev/sdb1
```

Para un volumen grande de datos no destinado al sistema operativo, puede reducirse la reserva.

---

# Consultar información de ext4

```bash id="fs055"
sudo tune2fs -l /dev/sdb1
```

Información resumida:

```bash id="fs056"
sudo dumpe2fs -h /dev/sdb1
```

---

# Campos importantes de tune2fs

```bash id="fs057"
sudo tune2fs -l /dev/sdb1 | grep -E \
"Filesystem volume name|Filesystem UUID|Filesystem state|Block count|Block size|Inode count|Reserved block count|Mount count"
```

---

# Consultar etiqueta de ext4

```bash id="fs058"
sudo e2label /dev/sdb1
```

Asignar etiqueta:

```bash id="fs059"
sudo e2label /dev/sdb1 DATOS
```

También:

```bash id="fs060"
sudo tune2fs -L DATOS /dev/sdb1
```

---

# Cambiar porcentaje reservado en ext4

Consultar bloques reservados:

```bash id="fs061"
sudo tune2fs -l /dev/sdb1 | grep "Reserved block"
```

Cambiar a 1%:

```bash id="fs062"
sudo tune2fs -m 1 /dev/sdb1
```

Cambiar a 0%:

```bash id="fs063"
sudo tune2fs -m 0 /dev/sdb1
```

No se recomienda eliminar la reserva del sistema de archivos raíz sin analizar el impacto.

---

# Verificar ext4

La partición debe estar desmontada.

```bash id="fs064"
sudo umount /datos
sudo fsck.ext4 -n /dev/sdb1
```

La opción `-n` no realiza cambios.

---

# Reparar ext4

Modo interactivo:

```bash id="fs065"
sudo fsck.ext4 /dev/sdb1
```

Aceptar automáticamente:

```bash id="fs066"
sudo fsck.ext4 -y /dev/sdb1
```

La opción `-y` debe utilizarse con precaución.

---

# Forzar una comprobación

```bash id="fs067"
sudo e2fsck -f /dev/sdb1
```

Comprobación sin modificar:

```bash id="fs068"
sudo e2fsck -fn /dev/sdb1
```

---

# Superbloques de respaldo

ext4 mantiene copias de respaldo del superbloque.

Mostrar ubicaciones posibles sin formatear:

```bash id="fs069"
sudo mke2fs -n /dev/sdb1
```

Reparar utilizando un superbloque alternativo:

```bash id="fs070"
sudo e2fsck -b 32768 /dev/sdb1
```

El número debe obtenerse del resultado correspondiente al dispositivo.

---

# Expandir ext4

Después de ampliar la partición o volumen:

```bash id="fs071"
sudo resize2fs /dev/sdb1
```

Verificar:

```bash id="fs072"
df -h /datos
```

---

# Reducir ext4

La reducción normalmente requiere desmontar el sistema de archivos.

Primero verificar:

```bash id="fs073"
sudo umount /datos
sudo e2fsck -f /dev/sdb1
```

Reducir:

```bash id="fs074"
sudo resize2fs /dev/sdb1 20G
```

Luego debe reducirse cuidadosamente el dispositivo subyacente.

Reducir una partición o volumen antes del sistema de archivos puede destruir datos.

---

# Orden correcto para redimensionar

## Para ampliar

```text
1. Ampliar partición o volumen.
2. Ampliar sistema de archivos.
```

## Para reducir ext4

```text
1. Desmontar.
2. Verificar el sistema de archivos.
3. Reducir el sistema de archivos.
4. Reducir el volumen o partición.
```

---

# XFS vs ext4

| Característica            | XFS          | ext4                       |
| ------------------------- | ------------ | -------------------------- |
| Journaling                | Sí           | Sí                         |
| Expansión en línea        | Sí           | Sí                         |
| Reducción                 | No           | Sí, normalmente desmontado |
| Archivos grandes          | Excelente    | Muy bueno                  |
| Uso empresarial           | Muy alto     | Muy alto                   |
| Herramienta de reparación | `xfs_repair` | `e2fsck`                   |
| Información               | `xfs_info`   | `tune2fs`                  |
| Etiqueta                  | `xfs_admin`  | `e2label`                  |
| Desfragmentación          | `xfs_fsr`    | `e4defrag`                 |

---

# Btrfs

Btrfs es un sistema de archivos moderno con funciones avanzadas.

Incluye:

* Copy-on-write.
* Snapshots.
* Subvolúmenes.
* Checksums.
* Compresión.
* RAID integrado.
* Detección de corrupción.
* Balanceo de datos.

---

# Crear Btrfs

```bash id="fs075"
sudo mkfs.btrfs -L DATOS /dev/sdb1
```

Montar:

```bash id="fs076"
sudo mount /dev/sdb1 /datos
```

---

# Consultar Btrfs

```bash id="fs077"
sudo btrfs filesystem show
```

Uso:

```bash id="fs078"
sudo btrfs filesystem usage /datos
```

---

# Crear subvolumen

```bash id="fs079"
sudo btrfs subvolume create /datos/proyectos
```

Listar:

```bash id="fs080"
sudo btrfs subvolume list /datos
```

---

# Crear snapshot

```bash id="fs081"
sudo btrfs subvolume snapshot \
/datos/proyectos \
/datos/snapshots/proyectos_antes_cambio
```

Snapshot de solo lectura:

```bash id="fs082"
sudo btrfs subvolume snapshot -r \
/datos/proyectos \
/datos/snapshots/proyectos_lectura
```

---

# Verificar Btrfs

Scrub:

```bash id="fs083"
sudo btrfs scrub start /datos
```

Estado:

```bash id="fs084"
sudo btrfs scrub status /datos
```

---

# VFAT

VFAT es utilizado principalmente para:

* Particiones EFI.
* Memorias USB.
* Compatibilidad con Windows.
* Dispositivos embebidos.

No soporta permisos Linux tradicionales de la misma forma que XFS o ext4.

---

# Crear VFAT

```bash id="fs085"
sudo mkfs.vfat /dev/sdb1
```

FAT32 con etiqueta:

```bash id="fs086"
sudo mkfs.vfat -F 32 -n DATOSUSB /dev/sdb1
```

---

# Partición EFI

Una partición EFI normalmente utiliza:

```text
FAT32
```

Punto de montaje habitual:

```text
/boot/efi
```

---

# exFAT

exFAT es adecuado para memorias USB y discos externos con archivos grandes.

Crear:

```bash id="fs087"
sudo mkfs.exfat -n DATOSUSB /dev/sdb1
```

Puede ser necesario instalar herramientas:

```bash id="fs088"
sudo dnf install exfatprogs
```

---

# NTFS

NTFS es el sistema de archivos principal de Windows.

Linux puede montarlo para lectura y escritura mediante controladores apropiados.

Consultar:

```bash id="fs089"
lsblk -f
```

Montar:

```bash id="fs090"
sudo mount -t ntfs3 /dev/sdb1 /mnt/windows
```

El soporte depende del kernel y de los paquetes instalados.

---

# tmpfs

`tmpfs` almacena datos en memoria y puede utilizar swap.

Su contenido normalmente desaparece al reiniciar.

Puntos comunes:

```text
/dev/shm
/run
/tmp
```

según la configuración de la distribución.

---

# Consultar tmpfs

```bash id="fs091"
df -hT | grep tmpfs
```

Ejemplo:

```text id="fs092"
tmpfs tmpfs 16G 0 16G 0% /dev/shm
tmpfs tmpfs 6.2G 9M 6.2G 1% /run
```

---

# Crear un montaje tmpfs

```bash id="fs093"
sudo mkdir -p /mnt/ram
sudo mount -t tmpfs -o size=1G tmpfs /mnt/ram
```

Verificar:

```bash id="fs094"
df -hT /mnt/ram
```

---

# tmpfs persistente en fstab

```fstab id="fs095"
tmpfs /mnt/ram tmpfs defaults,size=1G,mode=1777 0 0
```

Persistente significa que el montaje se recreará; los datos continuarán siendo temporales.

---

# Sistemas de archivos de solo lectura

Montar en modo de solo lectura:

```bash id="fs096"
sudo mount -o ro /dev/sdb1 /datos
```

Remontar:

```bash id="fs097"
sudo mount -o remount,ro /datos
```

Volver a lectura y escritura:

```bash id="fs098"
sudo mount -o remount,rw /datos
```

---

# Consultar opciones activas

```bash id="fs099"
findmnt -no SOURCE,FSTYPE,OPTIONS /datos
```

---

# Opciones de seguridad

| Opción     | Función                               |
| ---------- | ------------------------------------- |
| `ro`       | Solo lectura                          |
| `rw`       | Lectura y escritura                   |
| `noexec`   | Impide ejecución directa              |
| `nosuid`   | Ignora SUID y SGID                    |
| `nodev`    | No interpreta archivos de dispositivo |
| `noatime`  | No actualiza tiempo de acceso         |
| `relatime` | Reduce actualizaciones de acceso      |
| `sync`     | Escrituras síncronas                  |
| `async`    | Escrituras asíncronas                 |

---

# Ejemplo en fstab

```fstab id="fs100"
UUID=4cb0c22c-129d-45a6-a19e-c1f78285a113 /datos ext4 defaults,nodev,nosuid 0 2
```

---

# Fechas de los archivos

Linux mantiene varias marcas de tiempo.

| Fecha | Significado                               |
| ----- | ----------------------------------------- |
| atime | Último acceso                             |
| mtime | Última modificación del contenido         |
| ctime | Último cambio de metadatos                |
| btime | Fecha de creación, cuando está disponible |

---

# Consultar fechas

```bash id="fs101"
stat archivo.txt
```

Ejemplo:

```text id="fs102"
Access: 2026-07-28 09:00:00
Modify: 2026-07-28 08:30:00
Change: 2026-07-28 08:45:00
Birth:  2026-07-27 17:00:00
```

---

# noatime y relatime

Actualizar `atime` en cada lectura puede generar escrituras adicionales.

Opciones:

* `strictatime`: actualiza siempre.
* `noatime`: no actualiza el acceso.
* `relatime`: actualiza de forma reducida.

La mayoría de las distribuciones modernas utiliza `relatime`.

Consultar:

```bash id="fs103"
findmnt -no OPTIONS /
```

---

# Enlaces físicos y sistemas de archivos

Un enlace físico comparte el mismo inodo.

```bash id="fs104"
ln archivo.txt enlace_fisico.txt
```

Ver inodos:

```bash id="fs105"
ls -li archivo.txt enlace_fisico.txt
```

Los enlaces físicos no pueden cruzar sistemas de archivos distintos.

---

# Enlaces simbólicos

Un enlace simbólico contiene una ruta.

```bash id="fs106"
ln -s /datos/archivo.txt acceso.txt
```

Puede apuntar a otro sistema de archivos.

---

# Archivos dispersos

Un archivo disperso o sparse file puede tener un tamaño lógico mayor que el espacio físico utilizado.

Crear:

```bash id="fs107"
truncate -s 10G archivo_sparse.img
```

Comparar:

```bash id="fs108"
ls -lh archivo_sparse.img
du -h archivo_sparse.img
```

---

# Copiar archivos dispersos

Preservar estructura dispersa:

```bash id="fs109"
cp --sparse=always archivo_sparse.img copia.img
```

---

# Espacio reservado en ext4

ext4 puede reservar bloques para root.

Ventajas:

* Permite iniciar sesión y reparar el sistema cuando está casi lleno.
* Reduce fragmentación.
* Protege servicios críticos.

Consultar:

```bash id="fs110"
sudo tune2fs -l /dev/sda3 | grep -E "Reserved block count|Reserved block percentage"
```

---

# Archivos eliminados que siguen ocupando espacio

Cuando un proceso mantiene abierto un archivo eliminado, el espacio no se libera.

Buscar:

```bash id="fs111"
sudo lsof +L1
```

Ejemplo:

```text id="fs112"
COMMAND PID USER FD TYPE DEVICE SIZE/OFF NLINK NAME
java   4521 app  10w REG  253,0 15G      0 /var/log/app.log (deleted)
```

La solución apropiada suele ser:

* Recargar el servicio.
* Reiniciar el proceso.
* Cerrar el descriptor.

No es necesario reiniciar todo el servidor en la mayoría de los casos.

---

# Espacio usado por directorios

```bash id="fs113"
du -sh /var/*
```

Primer nivel:

```bash id="fs114"
sudo du -xhd1 /var | sort -h
```

Archivos mayores de 1 GB:

```bash id="fs115"
sudo find /var -xdev -type f -size +1G \
-printf '%s %p\n' | sort -n
```

---

# Diferencia entre df y du

`df` consulta bloques utilizados por el sistema de archivos.

`du` suma el espacio visible de archivos.

Pueden diferir por:

* Archivos eliminados abiertos.
* Bloques reservados.
* Metadatos.
* Directorios ocultos debajo de montajes.
* Permisos insuficientes.
* Archivos dispersos.

---

# Revisar un sistema de archivos antes de repararlo

Antes de ejecutar una herramienta de reparación:

1. Identifica el sistema de archivos.
2. Confirma el dispositivo.
3. Revisa los logs.
4. Realiza un respaldo si es posible.
5. Desmonta el sistema de archivos.
6. Ejecuta primero un modo de solo diagnóstico.
7. Documenta el resultado.

---

# Identificar el tipo antes de reparar

```bash id="fs116"
lsblk -f
```

O:

```bash id="fs117"
sudo blkid /dev/sdb1
```

No debe ejecutarse `fsck.ext4` sobre XFS ni `xfs_repair` sobre ext4.

---

# Reparación desde modo de emergencia

El sistema de archivos raíz no puede desmontarse durante la operación normal.

Para repararlo puede ser necesario:

* Arrancar desde un medio de rescate.
* Utilizar el modo de emergencia.
* Utilizar un entorno Live.
* Activar un volumen sin montarlo.
* Ejecutar la herramienta apropiada.

---

# Ver eventos relacionados con sistemas de archivos

```bash id="fs118"
journalctl -k | grep -Ei \
"xfs|ext4|btrfs|filesystem|I/O error|corrupt"
```

Desde el arranque actual:

```bash id="fs119"
journalctl -b -k -p warning
```

---

# Mensajes comunes de error

## Sistema de archivos en solo lectura

```text
Read-only file system
```

Puede indicar:

* Montaje configurado como `ro`.
* Error de almacenamiento.
* Corrupción.
* Protección automática del kernel.

Consultar:

```bash id="fs120"
findmnt -no OPTIONS /datos
journalctl -k -p err
```

---

## Error de entrada y salida

```text
Input/output error
```

Puede relacionarse con:

* Disco defectuoso.
* Cableado.
* Controladora.
* Almacenamiento remoto.
* Corrupción.
* Error del dispositivo virtual.

Consultar:

```bash id="fs121"
journalctl -k | grep -i "I/O error"
```

---

## Sistema de archivos desconocido

```text
unknown filesystem type
```

Puede deberse a:

* Paquete o módulo faltante.
* Tipo incorrecto.
* Firma dañada.
* Sistema no soportado.

Verificar:

```bash id="fs122"
sudo blkid /dev/sdb1
```

---

# Seleccionar un sistema de archivos

La elección depende de:

* Distribución.
* Tipo de carga.
* Tamaño.
* Necesidad de reducción.
* Snapshots.
* Compatibilidad.
* Soporte empresarial.
* Herramientas de recuperación.
* Tipo de archivos.
* Requisitos de rendimiento.

---

# Recomendaciones generales

| Escenario                              | Sistema sugerido |
| -------------------------------------- | ---------------- |
| Servidor RHEL/Fedora de uso general    | XFS              |
| Volumen que podría necesitar reducción | ext4             |
| Snapshots y subvolúmenes               | Btrfs            |
| Partición EFI                          | VFAT             |
| USB entre Linux, Windows y macOS       | exFAT            |
| Intercambio con Windows                | NTFS o exFAT     |
| Archivos temporales rápidos            | tmpfs            |

La selección debe validar siempre el soporte oficial de la distribución y de la aplicación.

---

# Sistemas de archivos para bases de datos

Para bases de datos se deben considerar:

* Recomendaciones del fabricante.
* Tamaño de bloque.
* Opciones de montaje.
* Latencia.
* Durabilidad.
* Caché.
* Alineación.
* Tipo de almacenamiento.
* Comportamiento ante fallos.
* Herramientas de respaldo.

No debe cambiarse el sistema de archivos de una base de datos únicamente por resultados genéricos de rendimiento.

---

# SQL Server en Linux

SQL Server en Linux suele utilizar sistemas compatibles y soportados por Microsoft, de acuerdo con la distribución y versión.

Los directorios comunes incluyen:

```text
/var/opt/mssql/data
/var/opt/mssql/log
/var/opt/mssql/backups
```

Antes de mover archivos a otro sistema de archivos se deben revisar:

* Soporte oficial.
* Propietario y grupo.
* Permisos.
* Contextos SELinux.
* Rendimiento.
* Persistencia en `/etc/fstab`.
* Disponibilidad durante el arranque.

---

# PostgreSQL

PostgreSQL puede trabajar sobre sistemas de archivos Linux comunes.

Debe prestarse atención a:

* Durabilidad de `fsync`.
* Espacio de WAL.
* Inodos.
* Permisos.
* Propietario `postgres`.
* Opciones de montaje.
* Snapshots consistentes.
* Comportamiento de caché.

Nunca debe desactivarse `fsync` en producción únicamente para obtener más velocidad sin comprender el riesgo.

---

# Flujo completo para crear un sistema de archivos

## Paso 1: Identificar la partición

```bash id="fs123"
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
```

## Paso 2: Confirmar que no está montada

```bash id="fs124"
findmnt /dev/sdb1
```

## Paso 3: Revisar firmas

```bash id="fs125"
sudo wipefs /dev/sdb1
```

## Paso 4: Crear XFS

```bash id="fs126"
sudo mkfs.xfs -L DATOS /dev/sdb1
```

O ext4:

```bash id="fs127"
sudo mkfs.ext4 -L DATOS /dev/sdb1
```

## Paso 5: Crear el punto de montaje

```bash id="fs128"
sudo mkdir -p /datos
```

## Paso 6: Montar

```bash id="fs129"
sudo mount /dev/sdb1 /datos
```

## Paso 7: Verificar

```bash id="fs130"
findmnt /datos
df -Th /datos
```

## Paso 8: Obtener UUID

```bash id="fs131"
sudo blkid /dev/sdb1
```

## Paso 9: Configurar fstab

XFS:

```fstab id="fs132"
UUID=UUID_OBTENIDO /datos xfs defaults 0 0
```

ext4:

```fstab id="fs133"
UUID=UUID_OBTENIDO /datos ext4 defaults 0 2
```

## Paso 10: Validar

```bash id="fs134"
sudo mount -a
sudo findmnt --verify
```

---

# Laboratorio práctico

## Ejercicio 1: Identificar sistemas de archivos

```bash id="labfs001"
lsblk -f
```

```bash id="labfs002"
df -Th
```

---

## Ejercicio 2: Consultar UUID

```bash id="labfs003"
sudo blkid
```

---

## Ejercicio 3: Crear XFS

En una partición de laboratorio:

```bash id="labfs004"
sudo mkfs.xfs -L LABXFS /dev/sdb1
```

---

## Ejercicio 4: Montar XFS

```bash id="labfs005"
sudo mkdir -p /mnt/labxfs
sudo mount /dev/sdb1 /mnt/labxfs
```

Verificar:

```bash id="labfs006"
xfs_info /mnt/labxfs
```

---

## Ejercicio 5: Crear un archivo y consultar su inodo

```bash id="labfs007"
sudo touch /mnt/labxfs/prueba.txt
ls -li /mnt/labxfs/prueba.txt
stat /mnt/labxfs/prueba.txt
```

---

## Ejercicio 6: Consultar bloques e inodos

```bash id="labfs008"
df -h /mnt/labxfs
df -i /mnt/labxfs
```

---

## Ejercicio 7: Desmontar y verificar XFS

```bash id="labfs009"
sudo umount /mnt/labxfs
sudo xfs_repair -n /dev/sdb1
```

---

## Ejercicio 8: Crear ext4

En otra partición de laboratorio:

```bash id="labfs010"
sudo mkfs.ext4 -L LABEXT4 /dev/sdc1
```

---

## Ejercicio 9: Consultar ext4

```bash id="labfs011"
sudo tune2fs -l /dev/sdc1
```

---

## Ejercicio 10: Verificar ext4

```bash id="labfs012"
sudo e2fsck -fn /dev/sdc1
```

---

## Ejercicio 11: Crear tmpfs

```bash id="labfs013"
sudo mkdir -p /mnt/labram
sudo mount -t tmpfs -o size=256M tmpfs /mnt/labram
```

Verificar:

```bash id="labfs014"
df -hT /mnt/labram
```

---

## Ejercicio 12: Comparar tamaño lógico y físico

```bash id="labfs015"
truncate -s 1G /mnt/labxfs/archivo_sparse.img
ls -lh /mnt/labxfs/archivo_sparse.img
du -h /mnt/labxfs/archivo_sparse.img
```

---

## Ejercicio 13: Buscar archivos eliminados abiertos

```bash id="labfs016"
sudo lsof +L1
```

---

## Ejercicio 14: Consultar opciones de montaje

```bash id="labfs017"
findmnt -o SOURCE,TARGET,FSTYPE,OPTIONS
```

---

# Diagnóstico de un sistema de archivos lleno

## Paso 1: Verificar bloques

```bash id="diagfs001"
df -h
```

## Paso 2: Verificar inodos

```bash id="diagfs002"
df -i
```

## Paso 3: Identificar directorios grandes

```bash id="diagfs003"
sudo du -xhd1 / | sort -h
```

## Paso 4: Buscar archivos grandes

```bash id="diagfs004"
sudo find / -xdev -type f -size +1G \
-printf '%s %p\n' 2>/dev/null | sort -n
```

## Paso 5: Buscar archivos eliminados abiertos

```bash id="diagfs005"
sudo lsof +L1
```

## Paso 6: Revisar logs

```bash id="diagfs006"
journalctl -k -p warning
```

## Paso 7: Determinar la solución

Las opciones pueden incluir:

* Eliminar archivos innecesarios.
* Aplicar retención.
* Rotar logs.
* Mover datos.
* Ampliar el sistema de archivos.
* Corregir una aplicación que genera archivos excesivos.
* Reiniciar de forma controlada el proceso que mantiene archivos eliminados.

---

# Diagnóstico de un sistema de archivos en solo lectura

## Paso 1: Confirmar opciones

```bash id="diagfs007"
findmnt -no SOURCE,FSTYPE,OPTIONS /datos
```

## Paso 2: Revisar kernel

```bash id="diagfs008"
journalctl -k --since "1 hour ago"
```

## Paso 3: Buscar errores

```bash id="diagfs009"
journalctl -k | grep -Ei \
"I/O error|read-only|xfs|ext4|corrupt"
```

## Paso 4: Realizar respaldo

Copiar los datos accesibles antes de intentar reparaciones cuando sea posible.

## Paso 5: Desmontar

```bash id="diagfs010"
sudo umount /datos
```

## Paso 6: Verificar según el tipo

XFS:

```bash id="diagfs011"
sudo xfs_repair -n /dev/sdb1
```

ext4:

```bash id="diagfs012"
sudo e2fsck -fn /dev/sdb1
```

---

# Comandos más utilizados

| Comando      | Descripción                         |
| ------------ | ----------------------------------- |
| `lsblk -f`   | Identificar sistemas de archivos    |
| `blkid`      | Consultar UUID, etiqueta y tipo     |
| `findmnt`    | Consultar montajes                  |
| `df -Th`     | Ver tipo y uso                      |
| `df -i`      | Ver uso de inodos                   |
| `stat`       | Mostrar metadatos                   |
| `mkfs.xfs`   | Crear XFS                           |
| `mkfs.ext4`  | Crear ext4                          |
| `xfs_info`   | Consultar XFS                       |
| `xfs_repair` | Verificar o reparar XFS             |
| `xfs_growfs` | Expandir XFS                        |
| `xfs_admin`  | Administrar etiqueta XFS            |
| `tune2fs`    | Administrar ext4                    |
| `e2fsck`     | Verificar y reparar ext4            |
| `resize2fs`  | Redimensionar ext4                  |
| `e2label`    | Administrar etiqueta ext4           |
| `wipefs`     | Consultar o borrar firmas           |
| `lsof +L1`   | Buscar archivos eliminados abiertos |
| `du`         | Analizar espacio por directorio     |

---

# Archivos y rutas importantes

| Ruta                  | Función                             |
| --------------------- | ----------------------------------- |
| `/etc/fstab`          | Montajes persistentes               |
| `/proc/filesystems`   | Sistemas soportados por el kernel   |
| `/proc/mounts`        | Montajes actuales                   |
| `/dev/disk/by-uuid/`  | Dispositivos por UUID               |
| `/dev/disk/by-label/` | Dispositivos por etiqueta           |
| `/sys/fs/`            | Información de sistemas de archivos |
| `/lost+found/`        | Recuperación en ext                 |
| `/proc/sys/fs/`       | Parámetros del kernel relacionados  |

---

# Ver sistemas soportados

```bash id="fs135"
cat /proc/filesystems
```

Ejemplo:

```text id="fs136"
nodev   sysfs
nodev   tmpfs
nodev   proc
        ext4
        xfs
        vfat
```

La palabra `nodev` indica que el sistema de archivos no depende directamente de un dispositivo de bloques.

---

# Ver módulos relacionados

```bash id="fs137"
lsmod | grep -E "xfs|btrfs|ntfs"
```

---

# Buenas prácticas

* Identifica siempre el tipo de sistema de archivos antes de repararlo.
* Confirma cuidadosamente el dispositivo antes de ejecutar `mkfs`.
* Realiza respaldos antes de reparar, redimensionar o convertir.
* Utiliza UUID para montajes persistentes.
* Ejecuta `mount -a` y `findmnt --verify` antes de reiniciar.
* No ejecutes herramientas de reparación sobre sistemas montados en lectura y escritura.
* Utiliza primero las opciones de diagnóstico sin cambios.
* Monitorea bloques e inodos.
* Revisa archivos eliminados que permanecen abiertos.
* Mantén espacio libre suficiente para operaciones normales.
* No utilices el journaling como sustituto de un respaldo.
* Selecciona el sistema de archivos según la aplicación y el soporte oficial.
* Documenta etiquetas, UUID, puntos de montaje y propósito.
* Revisa los mensajes del kernel ante errores de entrada y salida.
* No uses `xfs_repair -L` salvo que sea estrictamente necesario.
* No reduzcas una partición antes de reducir el sistema de archivos ext4.
* Recuerda que XFS no puede reducirse directamente.
* Prueba procedimientos de recuperación en un entorno de laboratorio.

---

# Errores comunes

## Formatear el dispositivo equivocado

Un comando como:

```bash id="errfs001"
sudo mkfs.xfs /dev/sda3
```

puede destruir un sistema operativo si el dispositivo es incorrecto.

Verifica antes:

```bash id="errfs002"
lsblk -o NAME,SIZE,MODEL,SERIAL,FSTYPE,MOUNTPOINTS
```

---

## Ejecutar una herramienta incorrecta

Incorrecto para XFS:

```bash id="errfs003"
sudo fsck.ext4 /dev/sdb1
```

Correcto:

```bash id="errfs004"
sudo xfs_repair -n /dev/sdb1
```

---

## Reparar un sistema montado

Incorrecto:

```bash id="errfs005"
sudo e2fsck /dev/sdb1
```

mientras está montado en `/datos`.

Correcto:

```bash id="errfs006"
sudo umount /datos
sudo e2fsck -fn /dev/sdb1
```

---

## Confundir espacio con inodos

`df -h` puede mostrar espacio libre, pero `df -i` puede mostrar:

```text
IUse% 100%
```

En ese caso no será posible crear nuevos archivos.

---

## Pensar que eliminar un archivo siempre libera espacio

Si el archivo permanece abierto:

```bash id="errfs007"
sudo lsof +L1
```

El espacio solamente se liberará cuando el proceso cierre el descriptor.

---

## Reducir el dispositivo antes que ext4

Esto puede cortar datos y destruir el sistema de archivos.

El orden correcto es:

```text
e2fsck
resize2fs
reducción del volumen o partición
```

---

## Intentar reducir XFS

XFS no admite reducción directa.

Debe migrarse el contenido a otro sistema de archivos de menor tamaño.

---

## Usar xfs_repair -L sin analizar

Este comando borra el journal de XFS.

Puede eliminar operaciones recientes que todavía no se habían aplicado completamente.

---

## Crear tmpfs demasiado grande

Aunque `tmpfs` no utiliza necesariamente toda la memoria inmediatamente, puede consumir RAM y swap hasta el límite establecido.

Debe dimensionarse según la memoria disponible y la carga del servidor.

---

## Utilizar VFAT para datos Linux sensibles

VFAT no soporta de forma nativa:

* Propietarios Unix.
* Permisos tradicionales.
* ACL de Linux.
* Enlaces simbólicos normales.

No es adecuado para directorios de aplicaciones Linux que dependan de estas funciones.

---

# Resumen

En este capítulo aprendiste a:

* Comprender la función de un sistema de archivos.
* Diferenciar sistemas locales, temporales, virtuales y de red.
* Identificar sistemas con `lsblk`, `blkid`, `df` y `findmnt`.
* Comprender bloques, inodos y metadatos.
* Analizar el funcionamiento del journaling.
* Crear y administrar XFS.
* Crear y administrar ext4.
* Verificar y reparar sistemas de archivos.
* Expandir XFS y redimensionar ext4.
* Comprender las funciones básicas de Btrfs.
* Trabajar con VFAT, exFAT, NTFS y tmpfs.
* Analizar espacio, inodos y archivos eliminados abiertos.
* Seleccionar un sistema de archivos según la carga.
* Diagnosticar sistemas llenos, dañados o montados en solo lectura.
* Aplicar procedimientos seguros de creación, reparación y mantenimiento.
