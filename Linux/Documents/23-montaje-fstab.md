# 23. Montaje de Sistemas de Archivos y Configuración de `/etc/fstab`

En Linux, los sistemas de archivos no se acceden mediante letras de unidad como ocurre en otros sistemas operativos. En su lugar, cada dispositivo, partición, volumen lógico o recurso de red se integra dentro de una única estructura de directorios.

El proceso de conectar un sistema de archivos a un directorio se denomina **montaje**.

Por ejemplo, una partición como:

```text id="mount001"
/dev/sdb1
```

puede montarse en:

```text id="mount002"
/datos
```

Después del montaje, todos los archivos almacenados en `/dev/sdb1` estarán disponibles dentro del directorio `/datos`.

Los montajes realizados manualmente con el comando `mount` normalmente desaparecen al reiniciar el sistema. Para hacerlos permanentes se utiliza el archivo:

```text id="mount003"
/etc/fstab
```

Una configuración incorrecta en `/etc/fstab` puede impedir que el servidor arranque correctamente. Por esta razón, su administración debe realizarse de forma cuidadosa y verificarse antes de reiniciar.

---

# Objetivos de este capítulo

Al finalizar esta unidad serás capaz de:

* Comprender el concepto de montaje en Linux.
* Identificar dispositivos y sistemas de archivos.
* Crear puntos de montaje.
* Montar y desmontar sistemas de archivos.
* Consultar los montajes activos.
* Comprender la estructura de `/etc/fstab`.
* Configurar montajes persistentes.
* Utilizar UUID, etiquetas y rutas persistentes.
* Aplicar opciones de seguridad.
* Configurar montajes locales, de red y temporales.
* Diagnosticar errores de montaje.
* Recuperar un sistema con errores en `/etc/fstab`.
* Aplicar buenas prácticas orientadas al examen RHCSA.

---

# ¿Qué significa montar un sistema de archivos?

Montar significa asociar un sistema de archivos con un directorio existente.

El dispositivo:

```text id="mount004"
/dev/sdb1
```

puede contener un sistema de archivos XFS, ext4, VFAT u otro formato.

Para acceder a sus datos se crea un directorio:

```text id="mount005"
/datos
```

Luego se realiza el montaje:

```bash id="mount006"
sudo mount /dev/sdb1 /datos
```

A partir de ese momento, `/datos` representa el contenido almacenado en `/dev/sdb1`.

---

# Elementos de un montaje

Un montaje normalmente incluye:

| Elemento            | Descripción                                |
| ------------------- | ------------------------------------------ |
| Dispositivo         | Partición, volumen lógico o recurso remoto |
| Sistema de archivos | XFS, ext4, NFS, VFAT, entre otros          |
| Punto de montaje    | Directorio donde se accede al contenido    |
| Opciones            | Parámetros de seguridad y funcionamiento   |
| Persistencia        | Define si se conserva después del reinicio |

Ejemplo:

```text id="mount007"
/dev/sdb1 → XFS → /datos
```

---

# La jerarquía única de Linux

Linux utiliza un único árbol de directorios cuyo punto inicial es:

```text id="mount008"
/
```

Dentro de esta estructura pueden integrarse múltiples sistemas de archivos.

Ejemplo:

```text id="mount009"
/
├── boot
├── home
├── var
├── datos
└── respaldos
```

Aunque todos aparecen dentro del mismo árbol, pueden estar almacenados en dispositivos diferentes.

Ejemplo:

```text id="mount010"
/              → /dev/mapper/rhel-root
/boot          → /dev/sda2
/boot/efi      → /dev/sda1
/datos         → /dev/sdb1
/respaldos     → /dev/mapper/vgbackup-lvbackup
```

---

# ¿Qué es un punto de montaje?

Un punto de montaje es un directorio utilizado para acceder a un sistema de archivos.

Crear un punto de montaje:

```bash id="mount011"
sudo mkdir -p /datos
```

Verificar:

```bash id="mount012"
ls -ld /datos
```

Resultado posible:

```text id="mount013"
drwxr-xr-x. 2 root root 6 Jul 28 08:00 /datos
```

El punto de montaje debe existir antes de realizar el montaje.

---

# El punto de montaje debe estar vacío

Es recomendable utilizar un directorio vacío.

Verificar:

```bash id="mount014"
ls -la /datos
```

Si el directorio contiene archivos y luego se monta otro sistema de archivos encima, esos archivos no se eliminan, pero quedan temporalmente ocultos.

Ejemplo:

```text id="mount015"
/datos contiene archivo_local.txt
```

Después de montar `/dev/sdb1` en `/datos`, el archivo puede dejar de ser visible.

Al desmontar, aparecerá nuevamente.

---

# Identificar dispositivos antes de montar

El primer paso consiste en identificar el dispositivo correcto.

```bash id="mount016"
lsblk
```

Mostrar sistemas de archivos:

```bash id="mount017"
lsblk -f
```

Ejemplo:

```text id="mount018"
NAME   FSTYPE FSVER LABEL UUID                                 MOUNTPOINTS
sda
├─sda1 vfat   FAT32       7A9C-11F0                            /boot/efi
├─sda2 xfs                749aa5f1-6bbc-45e4-8d35-97a2e842f500 /boot
└─sda3 xfs                b1445d30-2413-4cc4-9588-c3395f65920b /
sdb
└─sdb1 xfs          DATOS 4d9e9e85-c690-4668-83cc-833c5baab1c4
```

En este ejemplo, `/dev/sdb1` tiene XFS, una etiqueta `DATOS` y todavía no está montado.

---

# Consultar UUID y tipo

```bash id="mount019"
sudo blkid
```

Dispositivo específico:

```bash id="mount020"
sudo blkid /dev/sdb1
```

Ejemplo:

```text id="mount021"
/dev/sdb1: LABEL="DATOS" UUID="4d9e9e85-c690-4668-83cc-833c5baab1c4" TYPE="xfs"
```

---

# Consultar firmas con wipefs

```bash id="mount022"
sudo wipefs /dev/sdb1
```

Este comando permite identificar firmas sin montar el dispositivo.

No debe utilizarse `wipefs -a` salvo que se quiera eliminar deliberadamente las firmas existentes.

---

# Montaje manual básico

Sintaxis:

```bash id="mount023"
sudo mount dispositivo punto_de_montaje
```

Ejemplo:

```bash id="mount024"
sudo mount /dev/sdb1 /datos
```

Verificar:

```bash id="mount025"
findmnt /datos
```

También:

```bash id="mount026"
df -h /datos
```

---

# Detectar automáticamente el sistema de archivos

Cuando se ejecuta:

```bash id="mount027"
sudo mount /dev/sdb1 /datos
```

Linux normalmente detecta el tipo mediante la firma del sistema de archivos.

También puede especificarse explícitamente.

XFS:

```bash id="mount028"
sudo mount -t xfs /dev/sdb1 /datos
```

ext4:

```bash id="mount029"
sudo mount -t ext4 /dev/sdb1 /datos
```

VFAT:

```bash id="mount030"
sudo mount -t vfat /dev/sdb1 /mnt/usb
```

---

# Montar utilizando UUID

```bash id="mount031"
sudo mount UUID=4d9e9e85-c690-4668-83cc-833c5baab1c4 /datos
```

El UUID identifica de manera única el sistema de archivos.

---

# Montar utilizando etiqueta

```bash id="mount032"
sudo mount LABEL=DATOS /datos
```

La etiqueta debe existir y ser única.

---

# Consultar montajes actuales con findmnt

```bash id="mount033"
findmnt
```

Ejemplo:

```text id="mount034"
TARGET      SOURCE                        FSTYPE OPTIONS
/           /dev/mapper/fedora-root       xfs    rw,relatime
├─/boot     /dev/sda2                     xfs    rw,relatime
├─/boot/efi /dev/sda1                     vfat   rw,relatime
└─/datos    /dev/sdb1                     xfs    rw,relatime
```

---

# Consultar un punto específico

```bash id="mount035"
findmnt /datos
```

Consultar un dispositivo:

```bash id="mount036"
findmnt /dev/sdb1
```

Mostrar columnas seleccionadas:

```bash id="mount037"
findmnt -o SOURCE,TARGET,FSTYPE,OPTIONS
```

---

# Consultar el sistema de archivos padre

Para determinar en qué sistema está almacenado un archivo:

```bash id="mount038"
findmnt -T /datos/archivo.txt
```

La opción `-T` busca el sistema de archivos correspondiente a una ruta.

---

# Consultar con df

```bash id="mount039"
df -h
```

Mostrar tipo:

```bash id="mount040"
df -Th
```

Punto específico:

```bash id="mount041"
df -Th /datos
```

---

# Consultar `/proc/mounts`

El kernel publica los montajes activos en:

```text id="mount042"
/proc/mounts
```

Consultar:

```bash id="mount043"
cat /proc/mounts
```

Filtrar:

```bash id="mount044"
grep " /datos " /proc/mounts
```

También existe:

```text id="mount045"
/proc/self/mounts
```

---

# Comando mount sin argumentos

```bash id="mount046"
mount
```

Este comando muestra todos los sistemas montados.

Filtrar:

```bash id="mount047"
mount | grep /datos
```

Para una salida más estructurada se recomienda `findmnt`.

---

# Desmontar un sistema de archivos

El comando utilizado es:

```text id="mount048"
umount
```

No es `unmount`.

Desmontar por punto:

```bash id="mount049"
sudo umount /datos
```

Desmontar por dispositivo:

```bash id="mount050"
sudo umount /dev/sdb1
```

Verificar:

```bash id="mount051"
findmnt /datos
```

Si no muestra salida, ya no está montado.

---

# No desmontar desde dentro del directorio

Si la terminal se encuentra dentro del punto de montaje, el sistema puede considerarlo ocupado.

Ejemplo problemático:

```bash id="mount052"
cd /datos
sudo umount /datos
```

Primero salir:

```bash id="mount053"
cd /
sudo umount /datos
```

---

# Error: target is busy

Ejemplo:

```text id="mount054"
umount: /datos: target is busy
```

Significa que uno o más procesos están utilizando archivos o directorios dentro del punto de montaje.

---

# Identificar procesos con fuser

```bash id="mount055"
sudo fuser -vm /datos
```

Ejemplo:

```text id="mount056"
                     USER        PID ACCESS COMMAND
/datos:              alejandro  5321 ..c.. bash
                     postgres    6211 F.... postgres
```

---

# Identificar procesos con lsof

```bash id="mount057"
sudo lsof +D /datos
```

En directorios grandes puede ser lento.

Alternativa:

```bash id="mount058"
sudo lsof /datos
```

---

# Finalizar procesos con fuser

```bash id="mount059"
sudo fuser -km /datos
```

La opción `-k` finaliza los procesos.

Debe utilizarse con extrema precaución, especialmente en servidores de bases de datos.

Es preferible detener correctamente el servicio:

```bash id="mount060"
sudo systemctl stop nombre-servicio
```

Luego:

```bash id="mount061"
sudo umount /datos
```

---

# Desmontaje diferido

```bash id="mount062"
sudo umount -l /datos
```

La opción `-l` significa **lazy**.

El punto se separa de la jerarquía inmediatamente, pero la limpieza final ocurre cuando deja de estar ocupado.

No debe utilizarse como solución habitual para ocultar un problema.

---

# Desmontaje forzado

```bash id="mount063"
sudo umount -f /datos
```

Se utiliza principalmente en algunos sistemas de archivos de red.

Puede provocar pérdida de datos o inconsistencias.

---

# Sincronizar antes de desmontar

```bash id="mount064"
sync
```

Después:

```bash id="mount065"
sudo umount /datos
```

El comando `sync` solicita que los datos pendientes se escriban en el almacenamiento.

---

# Montaje de solo lectura

```bash id="mount066"
sudo mount -o ro /dev/sdb1 /datos
```

Verificar:

```bash id="mount067"
findmnt -no OPTIONS /datos
```

Resultado posible:

```text id="mount068"
ro,relatime
```

---

# Montaje en lectura y escritura

```bash id="mount069"
sudo mount -o rw /dev/sdb1 /datos
```

---

# Remontar sin desmontar

Cambiar un montaje existente a solo lectura:

```bash id="mount070"
sudo mount -o remount,ro /datos
```

Volver a lectura y escritura:

```bash id="mount071"
sudo mount -o remount,rw /datos
```

Verificar:

```bash id="mount072"
findmnt -no SOURCE,FSTYPE,OPTIONS /datos
```

---

# Opciones comunes del comando mount

| Opción           | Descripción                       |
| ---------------- | --------------------------------- |
| `-t`             | Especifica el tipo                |
| `-o`             | Indica opciones                   |
| `-a`             | Monta todas las entradas de fstab |
| `-v`             | Modo detallado                    |
| `-r`             | Montaje de solo lectura           |
| `-w`             | Lectura y escritura               |
| `--bind`         | Montaje enlazado                  |
| `--move`         | Mueve un montaje                  |
| `--make-shared`  | Cambia propagación                |
| `--make-private` | Aísla propagación                 |

---

# ¿Qué es `/etc/fstab`?

`/etc/fstab` significa:

```text id="mount073"
File Systems Table
```

Es el archivo utilizado para definir sistemas de archivos que deben montarse automáticamente.

Ruta:

```text id="mount074"
/etc/fstab
```

Ver su contenido:

```bash id="mount075"
cat /etc/fstab
```

Editar:

```bash id="mount076"
sudo nano /etc/fstab
```

Antes de modificarlo:

```bash id="mount077"
sudo cp -a /etc/fstab /etc/fstab.bak
```

---

# Ejemplo de `/etc/fstab`

```fstab id="mount078"
#
# /etc/fstab
#
UUID=b1445d30-2413-4cc4-9588-c3395f65920b /         xfs  defaults 0 0
UUID=749aa5f1-6bbc-45e4-8d35-97a2e842f500 /boot     xfs  defaults 0 0
UUID=7A9C-11F0                            /boot/efi vfat umask=0077 0 2
UUID=4d9e9e85-c690-4668-83cc-833c5baab1c4 /datos   xfs  defaults 0 0
```

Las líneas que comienzan con `#` son comentarios.

---

# Estructura de una entrada fstab

Cada línea posee seis campos:

```text id="mount079"
dispositivo punto_montaje tipo opciones dump fsck
```

Ejemplo:

```fstab id="mount080"
UUID=4d9e9e85-c690-4668-83cc-833c5baab1c4 /datos xfs defaults 0 0
```

---

# Campo 1: dispositivo

Puede especificarse mediante:

* Nombre del dispositivo.
* UUID.
* Etiqueta.
* Identificador persistente.
* Recurso de red.
* Archivo de imagen.
* Volumen lógico.

Ejemplos:

```text id="mount081"
/dev/sdb1
```

```text id="mount082"
UUID=4d9e9e85-c690-4668-83cc-833c5baab1c4
```

```text id="mount083"
LABEL=DATOS
```

```text id="mount084"
/dev/mapper/vgdatos-lvdatos
```

---

# Campo 2: punto de montaje

Es el directorio donde se presentará el sistema.

Ejemplos:

```text id="mount085"
/
```

```text id="mount086"
/home
```

```text id="mount087"
/datos
```

```text id="mount088"
/mnt/backup
```

```text id="mount089"
none
```

`none` puede aparecer en swap o en ciertos montajes especiales.

---

# Campo 3: tipo de sistema de archivos

Ejemplos:

```text id="mount090"
xfs
```

```text id="mount091"
ext4
```

```text id="mount092"
vfat
```

```text id="mount093"
nfs
```

```text id="mount094"
cifs
```

```text id="mount095"
swap
```

```text id="mount096"
tmpfs
```

También puede utilizarse:

```text id="mount097"
auto
```

aunque normalmente es preferible especificar el tipo correcto.

---

# Campo 4: opciones

Controla la forma en que se realiza el montaje.

Ejemplo:

```text id="mount098"
defaults
```

Varias opciones se separan por comas:

```text id="mount099"
defaults,nodev,nosuid,noexec
```

No deben incluirse espacios entre las opciones.

---

# Campo 5: dump

Este valor se relaciona con la herramienta histórica `dump`.

Valores:

| Valor | Significado |
| ----: | ----------- |
|   `0` | No incluir  |
|   `1` | Incluir     |

En la mayoría de los sistemas modernos se utiliza:

```text id="mount100"
0
```

---

# Campo 6: orden de fsck

Define el orden de comprobación durante el arranque.

Valores habituales:

| Valor | Significado       |
| ----: | ----------------- |
|   `0` | No comprobar      |
|   `1` | Comprobar primero |
|   `2` | Comprobar después |

Generalmente:

* `/` utiliza `1` en ext4.
* Otros sistemas ext4 utilizan `2`.
* XFS normalmente utiliza `0`.
* Recursos de red utilizan `0`.

Ejemplo ext4:

```fstab id="mount101"
UUID=... /datos ext4 defaults 0 2
```

Ejemplo XFS:

```fstab id="mount102"
UUID=... /datos xfs defaults 0 0
```

---

# Diferencia de fsck entre ext4 y XFS

ext4 utiliza comprobaciones mediante:

```text id="mount103"
e2fsck
```

Por eso una entrada típica puede terminar en:

```text id="mount104"
0 2
```

XFS utiliza herramientas como:

```text id="mount105"
xfs_repair
```

y normalmente se configura:

```text id="mount106"
0 0
```

---

# Uso de nombres de dispositivo

Ejemplo:

```fstab id="mount107"
/dev/sdb1 /datos xfs defaults 0 0
```

Aunque funciona, los nombres como `/dev/sdb1` pueden cambiar si se agregan discos o cambia el orden de detección.

Por ello se recomienda utilizar UUID.

---

# Uso de UUID

Obtener UUID:

```bash id="mount108"
sudo blkid /dev/sdb1
```

Agregar a fstab:

```fstab id="mount109"
UUID=4d9e9e85-c690-4668-83cc-833c5baab1c4 /datos xfs defaults 0 0
```

Ventajas:

* Identificación estable.
* Independencia del orden de detección.
* Menor riesgo al agregar discos.
* Recomendado para servidores.

---

# Uso de etiquetas

Asignar etiqueta XFS:

```bash id="mount110"
sudo xfs_admin -L DATOS /dev/sdb1
```

Asignar etiqueta ext4:

```bash id="mount111"
sudo e2label /dev/sdb1 DATOS
```

Entrada:

```fstab id="mount112"
LABEL=DATOS /datos xfs defaults 0 0
```

Las etiquetas deben ser únicas para evitar ambigüedades.

---

# Rutas persistentes en `/dev/disk`

Linux crea enlaces persistentes.

Por UUID:

```bash id="mount113"
ls -l /dev/disk/by-uuid/
```

Por etiqueta:

```bash id="mount114"
ls -l /dev/disk/by-label/
```

Por identificador:

```bash id="mount115"
ls -l /dev/disk/by-id/
```

Ejemplo de fstab:

```fstab id="mount116"
/dev/disk/by-id/scsi-3600508b400105e210000900000490000-part1 /datos xfs defaults 0 0
```

---

# Crear un montaje persistente paso a paso

## Paso 1: identificar la partición

```bash id="mount117"
lsblk -f
```

## Paso 2: obtener UUID

```bash id="mount118"
sudo blkid /dev/sdb1
```

## Paso 3: crear punto de montaje

```bash id="mount119"
sudo mkdir -p /datos
```

## Paso 4: respaldar fstab

```bash id="mount120"
sudo cp -a /etc/fstab /etc/fstab.bak
```

## Paso 5: editar fstab

```bash id="mount121"
sudo nano /etc/fstab
```

## Paso 6: agregar entrada

```fstab id="mount122"
UUID=4d9e9e85-c690-4668-83cc-833c5baab1c4 /datos xfs defaults 0 0
```

## Paso 7: validar sintaxis

```bash id="mount123"
sudo findmnt --verify
```

## Paso 8: probar sin reiniciar

```bash id="mount124"
sudo mount -a
```

## Paso 9: verificar

```bash id="mount125"
findmnt /datos
df -Th /datos
```

---

# Comando mount -a

```bash id="mount126"
sudo mount -a
```

Este comando intenta montar todas las entradas de `/etc/fstab`, excepto aquellas configuradas con `noauto`.

Es una de las comprobaciones más importantes antes de reiniciar.

Modo detallado:

```bash id="mount127"
sudo mount -av
```

---

# Validar fstab con findmnt

```bash id="mount128"
sudo findmnt --verify
```

Modo detallado:

```bash id="mount129"
sudo findmnt --verify --verbose
```

Puede detectar:

* Puntos inexistentes.
* UUID incorrectos.
* Tipos inválidos.
* Opciones no reconocidas.
* Entradas duplicadas.
* Errores de sintaxis.

---

# Recargar unidades generadas por systemd

Systemd convierte entradas de fstab en unidades de montaje.

Después de modificar `/etc/fstab`, puede ejecutarse:

```bash id="mount130"
sudo systemctl daemon-reload
```

Luego:

```bash id="mount131"
sudo mount -a
```

En algunas versiones aparece una advertencia indicando que systemd todavía utiliza una versión anterior hasta ejecutar `daemon-reload`.

---

# Unidades de montaje systemd

Un punto como:

```text id="mount132"
/datos
```

se representa como:

```text id="mount133"
datos.mount
```

Consultar:

```bash id="mount134"
systemctl status datos.mount
```

Para:

```text id="mount135"
/mnt/respaldos
```

la unidad sería:

```text id="mount136"
mnt-respaldos.mount
```

---

# Convertir rutas a nombres systemd

```bash id="mount137"
systemd-escape -p --suffix=mount /mnt/respaldos
```

Resultado:

```text id="mount138"
mnt-respaldos.mount
```

---

# Listar unidades de montaje

```bash id="mount139"
systemctl list-units --type=mount
```

Mostrar también las inactivas:

```bash id="mount140"
systemctl list-units --type=mount --all
```

---

# Consultar dependencias

```bash id="mount141"
systemctl list-dependencies local-fs.target
```

Para sistemas remotos:

```bash id="mount142"
systemctl list-dependencies remote-fs.target
```

---

# Opción defaults

La opción:

```text id="mount143"
defaults
```

representa un conjunto habitual de opciones.

Generalmente incluye opciones equivalentes a:

```text id="mount144"
rw,suid,dev,exec,auto,nouser,async
```

El conjunto exacto puede depender del sistema de archivos y del kernel.

---

# Opciones de montaje comunes

| Opción                      | Descripción                                |
| --------------------------- | ------------------------------------------ |
| `rw`                        | Lectura y escritura                        |
| `ro`                        | Solo lectura                               |
| `auto`                      | Montar automáticamente                     |
| `noauto`                    | No montar automáticamente                  |
| `exec`                      | Permite ejecución                          |
| `noexec`                    | Impide ejecución directa                   |
| `suid`                      | Permite SUID y SGID                        |
| `nosuid`                    | Ignora SUID y SGID                         |
| `dev`                       | Permite archivos de dispositivo            |
| `nodev`                     | No interpreta dispositivos                 |
| `user`                      | Permite montaje por usuario                |
| `nouser`                    | Solo root puede montar                     |
| `users`                     | Cualquier usuario puede montar y desmontar |
| `sync`                      | Escritura síncrona                         |
| `async`                     | Escritura asíncrona                        |
| `noatime`                   | No actualiza fecha de acceso               |
| `relatime`                  | Reduce actualizaciones de atime            |
| `nofail`                    | No detener arranque si falla               |
| `x-systemd.automount`       | Montar bajo demanda                        |
| `x-systemd.device-timeout=` | Limitar espera del dispositivo             |
| `_netdev`                   | Depende de red                             |

---

# Opción noexec

```fstab id="mount145"
UUID=... /datos xfs defaults,noexec 0 0
```

Impide ejecutar directamente archivos binarios desde ese sistema.

Ejemplo:

```bash id="mount146"
/datos/script.sh
```

puede fallar con permiso denegado.

Sin embargo, todavía podría ejecutarse mediante un intérprete:

```bash id="mount147"
bash /datos/script.sh
```

Por tanto, `noexec` es una medida de endurecimiento, pero no sustituye controles de permisos y seguridad.

---

# Opción nodev

```fstab id="mount148"
UUID=... /datos xfs defaults,nodev 0 0
```

Impide que archivos especiales dentro del sistema sean interpretados como dispositivos.

Es recomendable para:

* Directorios de datos.
* Sistemas compartidos.
* Medios removibles.
* Directorios de usuarios cuando sea compatible.

---

# Opción nosuid

```fstab id="mount149"
UUID=... /datos xfs defaults,nosuid 0 0
```

Ignora bits SUID y SGID en archivos ejecutables.

Puede mejorar la seguridad en sistemas compartidos.

---

# Combinación de seguridad

```fstab id="mount150"
UUID=... /datos xfs defaults,nodev,nosuid,noexec 0 0
```

Antes de aplicar estas opciones debe verificarse que la aplicación no necesite ejecutar archivos o utilizar características bloqueadas.

---

# Opción noatime

```fstab id="mount151"
UUID=... /datos xfs defaults,noatime 0 0
```

Evita actualizar el tiempo de acceso cada vez que se lee un archivo.

Puede reducir escrituras, pero algunas aplicaciones pueden depender de `atime`.

---

# Opción relatime

```fstab id="mount152"
UUID=... /datos xfs defaults,relatime 0 0
```

Actualiza `atime` de forma reducida.

Es un equilibrio habitual entre compatibilidad y rendimiento.

---

# Opción nofail

```fstab id="mount153"
UUID=... /respaldos xfs defaults,nofail 0 0
```

Permite que el sistema continúe el arranque aunque el dispositivo no esté disponible.

Es útil para:

* Discos externos.
* Unidades secundarias.
* Recursos no críticos.
* Montajes opcionales.

No debe utilizarse para ocultar fallos de almacenamiento crítico.

---

# Tiempo de espera del dispositivo

```fstab id="mount154"
UUID=... /respaldos xfs defaults,nofail,x-systemd.device-timeout=10s 0 0
```

Esto limita cuánto tiempo esperará systemd por el dispositivo.

---

# Montaje bajo demanda

```fstab id="mount155"
UUID=... /datos xfs defaults,x-systemd.automount 0 0
```

Systemd crea un punto de automontaje y monta el sistema cuando se accede.

Ver unidades automount:

```bash id="mount156"
systemctl list-units --type=automount
```

---

# idle timeout con systemd

```fstab id="mount157"
UUID=... /datos xfs defaults,x-systemd.automount,x-systemd.idle-timeout=5min 0 0
```

Puede desmontar el recurso después de un tiempo sin uso, dependiendo de la configuración y tipo de montaje.

---

# Montajes opcionales con noauto

```fstab id="mount158"
UUID=... /mnt/usb ext4 noauto,user 0 0
```

No se montará durante el arranque.

Podrá montarse con:

```bash id="mount159"
mount /mnt/usb
```

Cuando una entrada está en fstab, no es necesario escribir dispositivo y punto simultáneamente.

---

# Opción user

```fstab id="mount160"
UUID=... /mnt/usb vfat noauto,user 0 0
```

Permite que un usuario normal monte el recurso.

Normalmente solo el usuario que lo montó puede desmontarlo.

---

# Opción users

```fstab id="mount161"
UUID=... /mnt/usb vfat noauto,users 0 0
```

Permite que distintos usuarios lo monten o desmonten.

Debe aplicarse con criterio de seguridad.

---

# Montaje bind

Un montaje bind presenta un directorio existente en otra ubicación.

Ejemplo:

```bash id="mount162"
sudo mkdir -p /srv/aplicacion/datos
sudo mkdir -p /datos/app
sudo mount --bind /datos/app /srv/aplicacion/datos
```

Verificar:

```bash id="mount163"
findmnt /srv/aplicacion/datos
```

---

# Bind persistente en fstab

```fstab id="mount164"
/datos/app /srv/aplicacion/datos none bind 0 0
```

También:

```fstab id="mount165"
/datos/app /srv/aplicacion/datos none bind,nodev 0 0
```

---

# Bind de solo lectura

Primero:

```bash id="mount166"
sudo mount --bind /datos/app /srv/aplicacion/datos
```

Luego:

```bash id="mount167"
sudo mount -o remount,bind,ro /srv/aplicacion/datos
```

En fstab puede requerir una configuración adecuada según la versión de util-linux y systemd.

---

# Montar tmpfs

Manual:

```bash id="mount168"
sudo mkdir -p /mnt/ram
sudo mount -t tmpfs -o size=512M tmpfs /mnt/ram
```

Verificar:

```bash id="mount169"
df -hT /mnt/ram
```

---

# tmpfs persistente

```fstab id="mount170"
tmpfs /mnt/ram tmpfs defaults,size=512M,mode=1777 0 0
```

El montaje se recrea después del reinicio, pero su contenido no se conserva.

---

# Montar swap mediante fstab

Crear swap:

```bash id="mount171"
sudo mkswap /dev/sdb2
```

Obtener UUID:

```bash id="mount172"
sudo blkid /dev/sdb2
```

Entrada:

```fstab id="mount173"
UUID=UUID_SWAP none swap defaults 0 0
```

Activar sin reiniciar:

```bash id="mount174"
sudo swapon -a
```

Verificar:

```bash id="mount175"
swapon --show
```

---

# Montar imagen ISO

Crear punto:

```bash id="mount176"
sudo mkdir -p /mnt/iso
```

Montar:

```bash id="mount177"
sudo mount -o loop imagen.iso /mnt/iso
```

Verificar:

```bash id="mount178"
findmnt /mnt/iso
```

Desmontar:

```bash id="mount179"
sudo umount /mnt/iso
```

---

# Entrada fstab para una imagen

```fstab id="mount180"
/ruta/imagen.iso /mnt/iso iso9660 loop,ro,noauto 0 0
```

---

# Montar dispositivos USB

Identificar:

```bash id="mount181"
lsblk -f
```

Crear punto:

```bash id="mount182"
sudo mkdir -p /mnt/usb
```

Montar:

```bash id="mount183"
sudo mount /dev/sdc1 /mnt/usb
```

Desmontar antes de retirar:

```bash id="mount184"
sync
sudo umount /mnt/usb
```

---

# Opciones para VFAT

VFAT no almacena permisos Unix tradicionales.

Ejemplo:

```fstab id="mount185"
UUID=7A9C-11F0 /mnt/usb vfat defaults,uid=1000,gid=1000,umask=0022 0 0
```

Opciones comunes:

| Opción   | Función                |
| -------- | ---------------------- |
| `uid=`   | Propietario aparente   |
| `gid=`   | Grupo aparente         |
| `umask=` | Permisos a retirar     |
| `dmask=` | Máscara de directorios |
| `fmask=` | Máscara de archivos    |
| `utf8`   | Nombres UTF-8          |

---

# Ejemplo con dmask y fmask

```fstab id="mount186"
UUID=7A9C-11F0 /mnt/usb vfat uid=1000,gid=1000,dmask=0022,fmask=0133 0 0
```

---

# Montaje NFS manual

Instalar cliente:

```bash id="mount187"
sudo dnf install nfs-utils
```

Crear punto:

```bash id="mount188"
sudo mkdir -p /mnt/nfs
```

Montar:

```bash id="mount189"
sudo mount -t nfs 192.168.100.20:/export/datos /mnt/nfs
```

Verificar:

```bash id="mount190"
findmnt /mnt/nfs
```

---

# NFS persistente

```fstab id="mount191"
192.168.100.20:/export/datos /mnt/nfs nfs defaults,_netdev 0 0
```

`_netdev` indica que el montaje requiere conectividad de red.

---

# NFS con automount

```fstab id="mount192"
192.168.100.20:/export/datos /mnt/nfs nfs defaults,_netdev,x-systemd.automount,nofail 0 0
```

Esta configuración evita algunos bloqueos durante el arranque si el servidor NFS no está disponible.

---

# Opciones NFS habituales

| Opción     | Descripción                          |
| ---------- | ------------------------------------ |
| `_netdev`  | Requiere red                         |
| `hard`     | Reintenta operaciones                |
| `soft`     | Devuelve error después de reintentos |
| `timeo=`   | Tiempo de espera                     |
| `retrans=` | Número de retransmisiones            |
| `vers=4`   | Utiliza NFSv4                        |
| `ro`       | Solo lectura                         |
| `rw`       | Lectura y escritura                  |
| `noexec`   | Impide ejecución                     |
| `nosuid`   | Ignora SUID                          |

En datos críticos normalmente se debe analizar cuidadosamente el uso de `soft`, ya que puede producir errores inesperados de aplicación.

---

# Montaje CIFS o SMB

Instalar herramientas:

```bash id="mount193"
sudo dnf install cifs-utils
```

Crear punto:

```bash id="mount194"
sudo mkdir -p /mnt/compartido
```

Montar:

```bash id="mount195"
sudo mount -t cifs //servidor/recurso /mnt/compartido \
-o username=usuario
```

El comando solicitará la contraseña.

---

# CIFS persistente

No es recomendable escribir la contraseña directamente en `/etc/fstab`.

Crear archivo:

```bash id="mount196"
sudo nano /root/.smbcredentials
```

Contenido:

```text id="mount197"
username=usuario
password=CONTRASEÑA
domain=DOMINIO
```

Proteger:

```bash id="mount198"
sudo chmod 600 /root/.smbcredentials
```

Entrada:

```fstab id="mount199"
//servidor/recurso /mnt/compartido cifs credentials=/root/.smbcredentials,_netdev,vers=3.0 0 0
```

---

# CIFS con nofail y automount

```fstab id="mount200"
//servidor/recurso /mnt/compartido cifs credentials=/root/.smbcredentials,_netdev,nofail,x-systemd.automount,vers=3.0 0 0
```

---

# Montaje mediante archivo loop

Crear archivo:

```bash id="mount201"
truncate -s 1G /var/lib/disco.img
```

Crear sistema ext4:

```bash id="mount202"
sudo mkfs.ext4 /var/lib/disco.img
```

Crear punto:

```bash id="mount203"
sudo mkdir -p /mnt/disco_img
```

Montar:

```bash id="mount204"
sudo mount -o loop /var/lib/disco.img /mnt/disco_img
```

Entrada:

```fstab id="mount205"
/var/lib/disco.img /mnt/disco_img ext4 loop,defaults 0 0
```

---

# Permisos del punto de montaje

Después de montar, los permisos visibles corresponden a la raíz del sistema de archivos montado.

Verificar:

```bash id="mount206"
ls -ld /datos
```

Cambiar propietario:

```bash id="mount207"
sudo chown alejandro:dba /datos
```

Permisos compartidos:

```bash id="mount208"
sudo chmod 2770 /datos
```

El bit SGID facilita la herencia del grupo.

---

# Cambios de permisos antes y después del montaje

Si se cambia el propietario del directorio antes de montar:

```bash id="mount209"
sudo chown alejandro:dba /datos
```

y luego se monta un sistema encima, los permisos visibles pueden cambiar porque ahora corresponden al directorio raíz del sistema montado.

Por ello, normalmente los permisos se configuran después del montaje.

---

# SELinux y puntos de montaje

En Fedora y RHEL, SELinux puede impedir que una aplicación utilice un nuevo punto de montaje aunque los permisos tradicionales sean correctos.

Consultar:

```bash id="mount210"
ls -Zd /datos
```

Asignar contexto persistente:

```bash id="mount211"
sudo semanage fcontext -a -t var_t "/datos(/.*)?"
```

Aplicar:

```bash id="mount212"
sudo restorecon -Rv /datos
```

El tipo debe seleccionarse según la aplicación.

---

# Mantener contextos con context=

Algunos sistemas permiten especificar contexto en el montaje:

```fstab id="mount213"
UUID=... /datos xfs defaults,context="system_u:object_r:var_t:s0" 0 0
```

No debe utilizarse un contexto genérico sin conocer la política SELinux necesaria.

---

# Montaje de almacenamiento para aplicaciones

Antes de utilizar un nuevo punto para una aplicación deben revisarse:

* Propietario.
* Grupo.
* Permisos.
* ACL.
* Contexto SELinux.
* Opciones de montaje.
* Dependencias systemd.
* Disponibilidad durante el arranque.
* Espacio e inodos.
* Rendimiento.
* Respaldo.

---

# Ejemplo para PostgreSQL

Supongamos un volumen montado en:

```text id="mount214"
/pgdata
```

Entrada:

```fstab id="mount215"
UUID=... /pgdata xfs defaults 0 0
```

Montar:

```bash id="mount216"
sudo mount -a
```

Configurar propietario:

```bash id="mount217"
sudo chown postgres:postgres /pgdata
sudo chmod 700 /pgdata
```

Verificar SELinux según la política:

```bash id="mount218"
sudo semanage fcontext -a -t postgresql_db_t "/pgdata(/.*)?"
sudo restorecon -Rv /pgdata
```

---

# Ejemplo para SQL Server

Supongamos un volumen en:

```text id="mount219"
/var/opt/mssql/backups
```

Entrada:

```fstab id="mount220"
UUID=... /var/opt/mssql/backups xfs defaults,nodev,nosuid 0 0
```

Montar:

```bash id="mount221"
sudo mount -a
```

Configurar propietario:

```bash id="mount222"
sudo chown mssql:mssql /var/opt/mssql/backups
sudo chmod 750 /var/opt/mssql/backups
```

Debe verificarse el contexto SELinux y el soporte oficial de la plataforma.

---

# Dependencias de servicios y montajes

Un servicio puede requerir que un sistema esté montado antes de iniciar.

En una unidad systemd puede utilizarse:

```ini id="mount223"
[Unit]
RequiresMountsFor=/datos
```

Ejemplo completo:

```ini id="mount224"
[Unit]
Description=Aplicación interna
RequiresMountsFor=/datos
After=network.target

[Service]
Type=simple
ExecStart=/opt/aplicacion/iniciar.sh

[Install]
WantedBy=multi-user.target
```

---

# x-systemd.requires

En fstab puede declararse una dependencia:

```fstab id="mount225"
UUID=... /datos xfs defaults,x-systemd.requires=servicio.service 0 0
```

Debe utilizarse con cuidado para evitar ciclos de dependencias.

---

# x-systemd.before y x-systemd.after

Ejemplo:

```fstab id="mount226"
UUID=... /datos xfs defaults,x-systemd.before=miaplicacion.service 0 0
```

Estas opciones permiten ajustar el orden de systemd.

---

# Cómo se representan espacios en fstab

Los espacios dentro de rutas deben escaparse.

Un espacio se representa como:

```text id="mount227"
\040
```

Ejemplo:

```fstab id="mount228"
UUID=... /mnt/Mis\040Datos ext4 defaults 0 2
```

Es preferible evitar espacios en puntos de montaje de servidores.

---

# Otros caracteres escapados

| Carácter        | Representación |
| --------------- | -------------- |
| Espacio         | `\040`         |
| Tabulación      | `\011`         |
| Salto de línea  | `\012`         |
| Barra invertida | `\134`         |

---

# Comentarios en fstab

```fstab id="mount229"
# Volumen principal de datos
UUID=... /datos xfs defaults 0 0
```

Es recomendable documentar:

* Propósito.
* Aplicación.
* Fecha.
* Responsable.
* Dispositivo.
* Requisitos especiales.

---

# Comprobar si un punto está montado

```bash id="mount230"
mountpoint /datos
```

Resultado si está montado:

```text id="mount231"
/datos is a mountpoint
```

Código de salida:

```bash id="mount232"
echo $?
```

También:

```bash id="mount233"
findmnt -M /datos
```

---

# Utilizar mountpoint en scripts

```bash id="mount234"
#!/bin/bash

PUNTO="/datos"

if mountpoint -q "$PUNTO"; then
    echo "$PUNTO está montado."
else
    echo "$PUNTO no está montado."
    exit 1
fi
```

---

# Verificar origen esperado

No basta con confirmar que `/datos` está montado. Debe comprobarse que el origen es el correcto.

```bash id="mount235"
findmnt -no SOURCE /datos
```

Ejemplo de validación:

```bash id="mount236"
#!/bin/bash

PUNTO="/datos"
ESPERADO="/dev/sdb1"
ACTUAL=$(findmnt -no SOURCE "$PUNTO")

if [ "$ACTUAL" = "$ESPERADO" ]; then
    echo "Montaje correcto."
else
    echo "Origen inesperado: $ACTUAL"
    exit 1
fi
```

---

# Diagnóstico de mount -a

Modo detallado:

```bash id="mount237"
sudo mount -av
```

Ejemplo correcto:

```text id="mount238"
/                        : ignored
/boot                    : already mounted
/datos                   : successfully mounted
```

Ejemplo de error:

```text id="mount239"
mount: /datos: can't find UUID=...
```

---

# Error de UUID incorrecto

Consultar:

```bash id="mount240"
sudo blkid
```

Comparar con:

```bash id="mount241"
grep -v '^\s*#' /etc/fstab
```

Corregir la entrada.

Luego:

```bash id="mount242"
sudo systemctl daemon-reload
sudo mount -a
```

---

# Error: mount point does not exist

Ejemplo:

```text id="mount243"
mount: /datos: mount point does not exist
```

Solución:

```bash id="mount244"
sudo mkdir -p /datos
sudo mount -a
```

---

# Error: wrong fs type

Ejemplo:

```text id="mount245"
wrong fs type, bad option, bad superblock
```

Posibles causas:

* Tipo incorrecto en fstab.
* Sistema de archivos dañado.
* Firma inexistente.
* Herramienta faltante.
* Opción no soportada.
* Dispositivo incorrecto.

Verificar:

```bash id="mount246"
lsblk -f
sudo blkid /dev/sdb1
sudo file -s /dev/sdb1
```

---

# Error: special device does not exist

Ejemplo:

```text id="mount247"
mount: special device /dev/sdb1 does not exist
```

Verificar:

```bash id="mount248"
lsblk
```

Posibles causas:

* Disco no detectado.
* Nombre cambió.
* Volumen lógico no activado.
* Dispositivo remoto no disponible.
* Error de hardware.

---

# Error: already mounted

Ejemplo:

```text id="mount249"
/dev/sdb1 is already mounted or mount point busy
```

Verificar:

```bash id="mount250"
findmnt /dev/sdb1
```

También:

```bash id="mount251"
findmnt /datos
```

---

# Error: permission denied

Puede estar relacionado con:

* Permisos tradicionales.
* SELinux.
* Opciones `ro`.
* Sistema de archivos de red.
* Credenciales.
* Restricciones de montaje.
* Usuario sin privilegios.

Consultar:

```bash id="mount252"
ls -ld /datos
ls -Zd /datos
findmnt -no OPTIONS /datos
```

---

# Error de montaje NFS

Probar conectividad:

```bash id="mount253"
ping -c 3 192.168.100.20
```

Consultar exportaciones:

```bash id="mount254"
showmount -e 192.168.100.20
```

Revisar logs:

```bash id="mount255"
journalctl -b | grep -i nfs
```

---

# Error de montaje CIFS

Probar recurso:

```bash id="mount256"
smbclient -L //servidor -U usuario
```

Revisar kernel:

```bash id="mount257"
journalctl -k | grep -i cifs
```

Verificar permisos del archivo de credenciales:

```bash id="mount258"
ls -l /root/.smbcredentials
```

Resultado recomendado:

```text id="mount259"
-rw-------. 1 root root ... /root/.smbcredentials
```

---

# Consultar logs de montaje

```bash id="mount260"
journalctl -b | grep -Ei "mount|fstab"
```

Errores:

```bash id="mount261"
journalctl -b -p err
```

Unidad específica:

```bash id="mount262"
journalctl -u datos.mount
```

---

# Recuperación por error en fstab

Un error grave puede llevar el sistema a:

```text id="mount263"
emergency mode
```

Procedimiento general:

1. Iniciar sesión como root.
2. Remontar `/` en lectura y escritura si es necesario.
3. Editar `/etc/fstab`.
4. Comentar o corregir la entrada.
5. Validar.
6. Reiniciar o continuar el arranque.

---

# Remontar raíz en lectura y escritura

```bash id="mount264"
mount -o remount,rw /
```

Editar:

```bash id="mount265"
nano /etc/fstab
```

Comentar la entrada problemática:

```fstab id="mount266"
# UUID=UUID_INCORRECTO /datos xfs defaults 0 0
```

Validar:

```bash id="mount267"
findmnt --verify
```

Reiniciar:

```bash id="mount268"
systemctl reboot
```

---

# Restaurar respaldo de fstab

Si existe una copia:

```bash id="mount269"
sudo cp -a /etc/fstab.bak /etc/fstab
```

Luego:

```bash id="mount270"
sudo findmnt --verify
sudo mount -a
```

---

# Herramienta systemd-analyze verify

Para revisar unidades relacionadas:

```bash id="mount271"
systemd-analyze verify /etc/systemd/system/*.mount
```

Las entradas de fstab son generadas dinámicamente por systemd, por lo que la herramienta principal continúa siendo:

```bash id="mount272"
findmnt --verify
```

---

# Archivos y rutas importantes

| Ruta                      | Función                                     |
| ------------------------- | ------------------------------------------- |
| `/etc/fstab`              | Montajes persistentes                       |
| `/etc/mtab`               | Información de montajes, normalmente enlace |
| `/proc/mounts`            | Montajes conocidos por el kernel            |
| `/proc/self/mountinfo`    | Información detallada                       |
| `/dev/disk/by-uuid/`      | Dispositivos por UUID                       |
| `/dev/disk/by-label/`     | Dispositivos por etiqueta                   |
| `/dev/disk/by-id/`        | Identificadores persistentes                |
| `/run/systemd/generator/` | Unidades generadas desde fstab              |
| `/mnt/`                   | Montajes administrativos temporales         |
| `/media/`                 | Medios removibles                           |

---

# Consultar unidades generadas desde fstab

```bash id="mount273"
ls -l /run/systemd/generator/
```

Buscar montajes:

```bash id="mount274"
find /run/systemd/generator -name "*.mount" -o -name "*.automount"
```

---

# Comandos más utilizados

| Comando                             | Descripción                          |
| ----------------------------------- | ------------------------------------ |
| `mount`                             | Montar un sistema de archivos        |
| `umount`                            | Desmontar                            |
| `findmnt`                           | Consultar montajes                   |
| `mountpoint`                        | Verificar un punto                   |
| `lsblk -f`                          | Mostrar dispositivos y sistemas      |
| `blkid`                             | Consultar UUID y etiquetas           |
| `df -Th`                            | Mostrar uso y tipo                   |
| `mount -a`                          | Montar entradas de fstab             |
| `mount -av`                         | Montar con detalle                   |
| `findmnt --verify`                  | Validar fstab                        |
| `fuser`                             | Identificar procesos                 |
| `lsof`                              | Ver archivos abiertos                |
| `systemctl daemon-reload`           | Recargar configuración               |
| `systemctl list-units --type=mount` | Listar unidades                      |
| `systemd-escape`                    | Convertir rutas en nombres de unidad |
| `swapon -a`                         | Activar swap de fstab                |

---

# Flujo recomendado para agregar un montaje

## Paso 1: identificar

```bash id="mount275"
lsblk -f
```

## Paso 2: confirmar sistema de archivos

```bash id="mount276"
sudo blkid /dev/sdb1
```

## Paso 3: crear punto

```bash id="mount277"
sudo mkdir -p /datos
```

## Paso 4: montar manualmente

```bash id="mount278"
sudo mount /dev/sdb1 /datos
```

## Paso 5: verificar acceso

```bash id="mount279"
findmnt /datos
df -Th /datos
```

## Paso 6: configurar permisos

```bash id="mount280"
sudo chown root:dba /datos
sudo chmod 2770 /datos
```

## Paso 7: obtener UUID

```bash id="mount281"
sudo blkid -s UUID -o value /dev/sdb1
```

## Paso 8: respaldar fstab

```bash id="mount282"
sudo cp -a /etc/fstab /etc/fstab.bak
```

## Paso 9: agregar entrada

```fstab id="mount283"
UUID=UUID_OBTENIDO /datos xfs defaults 0 0
```

## Paso 10: recargar y validar

```bash id="mount284"
sudo systemctl daemon-reload
sudo findmnt --verify
sudo mount -a
```

## Paso 11: confirmar

```bash id="mount285"
findmnt /datos
mountpoint /datos
```

---

# Laboratorio práctico

## Ejercicio 1: identificar sistemas montados

```bash id="labmount001"
findmnt
```

Mostrar columnas:

```bash id="labmount002"
findmnt -o SOURCE,TARGET,FSTYPE,OPTIONS
```

---

## Ejercicio 2: identificar una partición disponible

```bash id="labmount003"
lsblk -f
```

Selecciona una partición de laboratorio que no contenga información importante.

---

## Ejercicio 3: crear punto de montaje

```bash id="labmount004"
sudo mkdir -p /mnt/labdatos
```

---

## Ejercicio 4: montar manualmente

```bash id="labmount005"
sudo mount /dev/sdb1 /mnt/labdatos
```

Verificar:

```bash id="labmount006"
findmnt /mnt/labdatos
```

---

## Ejercicio 5: crear archivo de prueba

```bash id="labmount007"
sudo touch /mnt/labdatos/prueba.txt
ls -l /mnt/labdatos
```

---

## Ejercicio 6: desmontar

```bash id="labmount008"
cd /
sudo umount /mnt/labdatos
```

Verificar:

```bash id="labmount009"
mountpoint /mnt/labdatos
```

---

## Ejercicio 7: obtener UUID

```bash id="labmount010"
sudo blkid /dev/sdb1
```

---

## Ejercicio 8: respaldar fstab

```bash id="labmount011"
sudo cp -a /etc/fstab /etc/fstab.lab.bak
```

---

## Ejercicio 9: crear entrada persistente

Editar:

```bash id="labmount012"
sudo nano /etc/fstab
```

Agregar:

```fstab id="labmount013"
UUID=UUID_OBTENIDO /mnt/labdatos xfs defaults 0 0
```

Utiliza `ext4` y `0 2` si ese es el sistema de archivos correspondiente.

---

## Ejercicio 10: validar

```bash id="labmount014"
sudo systemctl daemon-reload
sudo findmnt --verify
sudo mount -av
```

---

## Ejercicio 11: confirmar persistencia lógica

```bash id="labmount015"
findmnt /mnt/labdatos
df -Th /mnt/labdatos
```

---

## Ejercicio 12: aplicar opciones de seguridad

Modificar:

```fstab id="labmount016"
UUID=UUID_OBTENIDO /mnt/labdatos xfs defaults,nodev,nosuid 0 0
```

Remontar:

```bash id="labmount017"
sudo umount /mnt/labdatos
sudo mount -a
```

Verificar:

```bash id="labmount018"
findmnt -no OPTIONS /mnt/labdatos
```

---

## Ejercicio 13: crear montaje bind

```bash id="labmount019"
sudo mkdir -p /datos/origen
sudo mkdir -p /srv/destino
sudo mount --bind /datos/origen /srv/destino
```

Verificar:

```bash id="labmount020"
findmnt /srv/destino
```

---

## Ejercicio 14: crear tmpfs

```bash id="labmount021"
sudo mkdir -p /mnt/labram
sudo mount -t tmpfs -o size=128M tmpfs /mnt/labram
```

Verificar:

```bash id="labmount022"
df -Th /mnt/labram
```

---

## Ejercicio 15: generar error controlado

Agregar temporalmente una entrada con UUID incorrecto en un entorno de laboratorio:

```fstab id="labmount023"
UUID=UUID_INEXISTENTE /mnt/error xfs defaults,nofail 0 0
```

Validar:

```bash id="labmount024"
sudo findmnt --verify
sudo mount -av
```

Eliminar la entrada al finalizar.

---

# Práctica RHCSA: montaje persistente

En el examen puede solicitarse una tarea similar a:

```text id="mount286"
Configure un sistema de archivos para que se monte persistentemente en /datos.
```

Procedimiento:

```bash id="mount287"
lsblk -f
```

```bash id="mount288"
sudo blkid /dev/sdb1
```

```bash id="mount289"
sudo mkdir -p /datos
```

Agregar:

```fstab id="mount290"
UUID=UUID_OBTENIDO /datos xfs defaults 0 0
```

Validar:

```bash id="mount291"
sudo systemctl daemon-reload
sudo findmnt --verify
sudo mount -a
findmnt /datos
```

No debe reiniciarse sin validar.

---

# Diagnóstico de un montaje que no funciona

## Paso 1: confirmar dispositivo

```bash id="diagmount001"
lsblk -f
```

## Paso 2: confirmar UUID

```bash id="diagmount002"
sudo blkid
```

## Paso 3: revisar punto

```bash id="diagmount003"
ls -ld /datos
```

## Paso 4: revisar fstab

```bash id="diagmount004"
grep -Ev '^\s*(#|$)' /etc/fstab
```

## Paso 5: validar

```bash id="diagmount005"
sudo findmnt --verify
```

## Paso 6: probar detalladamente

```bash id="diagmount006"
sudo mount -av
```

## Paso 7: revisar systemd

```bash id="diagmount007"
systemctl status datos.mount
```

## Paso 8: revisar logs

```bash id="diagmount008"
journalctl -u datos.mount
journalctl -b -p err
```

## Paso 9: comprobar sistema de archivos

XFS:

```bash id="diagmount009"
sudo xfs_repair -n /dev/sdb1
```

ext4:

```bash id="diagmount010"
sudo e2fsck -fn /dev/sdb1
```

La partición debe estar desmontada antes de la comprobación.

---

# Buenas prácticas

* Identifica el dispositivo con `lsblk -f` y `blkid`.
* Utiliza UUID para montajes permanentes.
* Crea una copia de `/etc/fstab` antes de editarlo.
* Verifica que el punto de montaje exista.
* Utiliza puntos de montaje vacíos.
* Prueba primero el montaje manual.
* Ejecuta `findmnt --verify` después de editar fstab.
* Ejecuta `mount -a` antes de reiniciar.
* Revisa la salida de `mount -av`.
* Utiliza `systemctl daemon-reload` después de modificar fstab.
* Aplica `nodev`, `nosuid` y `noexec` cuando sean compatibles.
* Utiliza `_netdev` para sistemas de archivos de red.
* Considera `nofail` solo para montajes no críticos.
* Evita contraseñas directamente en `/etc/fstab`.
* Protege archivos de credenciales.
* Configura correctamente permisos y SELinux.
* Detén aplicaciones antes de desmontar almacenamiento activo.
* No utilices `umount -f` como primera solución.
* Documenta cada entrada.
* Conserva acceso de recuperación antes de reiniciar servidores remotos.

---

# Errores comunes

## Utilizar un UUID incorrecto

Incorrecto:

```fstab id="errmount001"
UUID=1234-INVALIDO /datos xfs defaults 0 0
```

Verificar:

```bash id="errmount002"
sudo blkid
```

---

## Especificar un sistema de archivos equivocado

Incorrecto:

```fstab id="errmount003"
UUID=... /datos ext4 defaults 0 2
```

cuando el dispositivo es XFS.

Verificar:

```bash id="errmount004"
lsblk -f
```

---

## No crear el punto de montaje

Error:

```text id="errmount005"
mount point does not exist
```

Solución:

```bash id="errmount006"
sudo mkdir -p /datos
```

---

## Reiniciar sin probar fstab

Procedimiento incorrecto:

```text id="errmount007"
Editar fstab y reiniciar inmediatamente.
```

Procedimiento correcto:

```bash id="errmount008"
sudo systemctl daemon-reload
sudo findmnt --verify
sudo mount -a
```

---

## Utilizar espacios entre opciones

Incorrecto:

```fstab id="errmount009"
UUID=... /datos xfs defaults, nodev, nosuid 0 0
```

Correcto:

```fstab id="errmount010"
UUID=... /datos xfs defaults,nodev,nosuid 0 0
```

---

## Agregar campos incompletos

Incorrecto:

```fstab id="errmount011"
UUID=... /datos xfs
```

Correcto:

```fstab id="errmount012"
UUID=... /datos xfs defaults 0 0
```

---

## Utilizar fsck incorrectamente para XFS

No debe suponerse que el último campo `2` es apropiado para todos los sistemas.

Entrada típica XFS:

```fstab id="errmount013"
UUID=... /datos xfs defaults 0 0
```

---

## Desmontar un sistema usado por una base de datos

Incorrecto:

```bash id="errmount014"
sudo umount /var/lib/pgsql
```

sin detener PostgreSQL.

Correcto:

```bash id="errmount015"
sudo systemctl stop postgresql
sudo umount /var/lib/pgsql
```

Debe verificarse el nombre real del servicio.

---

## Ocultar archivos bajo un montaje

Si el directorio contiene datos antes del montaje, estos quedan ocultos.

Verifica antes:

```bash id="errmount016"
ls -la /datos
```

---

## Utilizar nofail en almacenamiento crítico

`nofail` puede permitir que el sistema arranque sin el volumen.

Esto podría provocar que una aplicación escriba en el directorio vacío del sistema raíz en lugar del volumen esperado.

Por ello, antes de iniciar aplicaciones debe verificarse:

```bash id="errmount017"
mountpoint -q /datos
```

---

## No verificar el origen montado

Un punto puede estar montado desde un dispositivo incorrecto.

Verificar:

```bash id="errmount018"
findmnt -no SOURCE,TARGET,FSTYPE /datos
```

---

## Guardar credenciales CIFS en texto dentro de fstab

Incorrecto:

```fstab id="errmount019"
//servidor/recurso /mnt/cifs cifs username=usuario,password=secreto 0 0
```

Preferible:

```fstab id="errmount020"
//servidor/recurso /mnt/cifs cifs credentials=/root/.smbcredentials,_netdev 0 0
```

Proteger:

```bash id="errmount021"
sudo chmod 600 /root/.smbcredentials
```

---

# Resumen

En este capítulo aprendiste a:

* Comprender el concepto de montaje en Linux.
* Identificar dispositivos, sistemas de archivos y puntos de montaje.
* Montar y desmontar sistemas manualmente.
* Consultar montajes con `findmnt`, `mount`, `df` y `mountpoint`.
* Comprender los seis campos de `/etc/fstab`.
* Configurar montajes persistentes mediante UUID y etiquetas.
* Validar fstab con `findmnt --verify`.
* Probar configuraciones mediante `mount -a`.
* Integrar las entradas con systemd.
* Aplicar opciones como `nodev`, `nosuid`, `noexec`, `nofail` y `_netdev`.
* Crear montajes bind, tmpfs, swap y loop.
* Configurar recursos NFS y CIFS.
* Proteger archivos de credenciales.
* Administrar permisos y contextos SELinux.
* Diagnosticar dispositivos ocupados, UUID incorrectos y errores de sistema de archivos.
* Recuperar un sistema que entra en modo de emergencia por un error en fstab.
* Aplicar procedimientos seguros y orientados al examen RHCSA.
