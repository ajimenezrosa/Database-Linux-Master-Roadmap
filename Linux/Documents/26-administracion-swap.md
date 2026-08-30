# 26. Administración de Memoria Swap en Linux

La memoria swap es un espacio de almacenamiento utilizado por Linux como extensión de la memoria RAM.

Cuando la memoria física comienza a llenarse, el kernel puede mover temporalmente páginas de memoria poco utilizadas hacia el área swap. Esto permite liberar RAM para procesos activos y reducir el riesgo de que el sistema finalice aplicaciones por falta de memoria.

La swap puede implementarse mediante:

* Una partición dedicada.
* Un Logical Volume de LVM.
* Un archivo swap.
* Varios dispositivos swap con diferentes prioridades.

Aunque la swap puede ayudar a mantener la estabilidad del sistema, no sustituye la memoria RAM. Los discos y dispositivos de almacenamiento son mucho más lentos que la memoria física.

Una configuración incorrecta puede provocar:

* Rendimiento deficiente.
* Uso excesivo de disco.
* Latencia.
* Bloqueos aparentes.
* Errores de aplicaciones.
* Activación del mecanismo OOM Killer.

---

# Objetivos

Al finalizar este capítulo serás capaz de:

* Comprender qué es la memoria virtual.
* Diferenciar RAM y swap.
* Identificar dispositivos y archivos swap.
* Consultar el consumo de memoria.
* Crear una partición swap.
* Crear un Logical Volume swap.
* Crear un archivo swap.
* Activar y desactivar swap.
* Configurar swap persistentemente en `/etc/fstab`.
* Administrar prioridades.
* Ajustar el parámetro `vm.swappiness`.
* Diagnosticar presión de memoria.
* Comprender el funcionamiento del OOM Killer.
* Ampliar o reemplazar áreas swap.
* Aplicar buenas prácticas orientadas al examen RHCSA.

---

# ¿Qué es la memoria virtual?

La memoria virtual es un mecanismo mediante el cual Linux proporciona a los procesos un espacio de memoria lógico que puede estar respaldado por:

* Memoria RAM.
* Espacio swap.
* Archivos mapeados.
* Bibliotecas compartidas.
* Páginas administradas por el kernel.

Cada proceso trabaja con direcciones virtuales. El kernel y el hardware se encargan de relacionarlas con memoria física o almacenamiento secundario.

---

# ¿Qué es la swap?

La swap es un área de almacenamiento utilizada para guardar páginas de memoria que el kernel decide retirar temporalmente de la RAM.

Flujo simplificado:

```text
Proceso utiliza memoria
        ↓
La RAM comienza a llenarse
        ↓
El kernel identifica páginas poco activas
        ↓
Las mueve hacia swap
        ↓
La RAM queda disponible para otras tareas
```

Cuando el proceso necesita nuevamente esa información:

```text
Página solicitada
        ↓
Se lee desde swap
        ↓
Regresa a RAM
        ↓
El proceso continúa
```

Esta operación se conoce como:

```text
Swap in / Swap out
```

---

# RAM frente a swap

| Característica | RAM                   | Swap                                                            |
| -------------- | --------------------- | --------------------------------------------------------------- |
| Velocidad      | Muy alta              | Mucho menor                                                     |
| Tecnología     | Memoria física        | Disco, SSD, NVMe o archivo                                      |
| Uso principal  | Procesos activos      | Páginas menos utilizadas                                        |
| Persistencia   | Se pierde al apagar   | El área existe, pero su contenido no se conserva como dato útil |
| Rendimiento    | Excelente             | Puede causar latencia                                           |
| Capacidad      | Limitada por hardware | Puede ampliarse más fácilmente                                  |

La swap mejora la tolerancia ante presión de memoria, pero no convierte el almacenamiento en RAM real.

---

# Razones para utilizar swap

La swap puede ser útil para:

* Evitar fallos inmediatos por falta de RAM.
* Mantener páginas poco utilizadas fuera de memoria física.
* Permitir hibernación, cuando está configurada apropiadamente.
* Dar margen temporal ante picos de consumo.
* Facilitar ciertas cargas con memoria variable.
* Permitir al kernel administrar mejor la caché.

---

# Cuándo la swap no resuelve el problema

Agregar swap no corrige necesariamente:

* Fugas de memoria.
* Consultas de bases de datos mal optimizadas.
* Aplicaciones sobredimensionadas.
* Contenedores sin límites.
* Procesos bloqueados.
* Mala configuración de memoria.
* Servidores con RAM insuficiente.
* Uso excesivo de caché por una aplicación.
* Crecimiento descontrolado de procesos.

Si el sistema utiliza swap constantemente y presenta latencia, debe investigarse la causa.

---

# Consultar memoria con `free`

```bash
free -h
```

Ejemplo:

```text
               total        used        free      shared  buff/cache   available
Mem:            31Gi        18Gi       2.1Gi       1.0Gi        11Gi        12Gi
Swap:          8.0Gi       1.2Gi       6.8Gi
```

---

# Interpretar `free`

| Columna      | Descripción                    |
| ------------ | ------------------------------ |
| `total`      | Memoria total                  |
| `used`       | Memoria utilizada              |
| `free`       | Memoria completamente libre    |
| `shared`     | Memoria compartida             |
| `buff/cache` | Buffers y caché                |
| `available`  | Memoria estimada disponible    |
| `Swap`       | Uso del espacio de intercambio |

La columna más útil para evaluar memoria disponible suele ser:

```text
available
```

No debe suponerse que un valor pequeño en `free` significa necesariamente falta de memoria, porque Linux utiliza RAM libre como caché.

---

# Mostrar resultado en megabytes

```bash
free -m
```

En gigabytes:

```bash
free -g
```

Actualizar cada dos segundos:

```bash
free -h -s 2
```

---

# Consultar swap activa

```bash
swapon --show
```

Ejemplo:

```text
NAME       TYPE      SIZE USED PRIO
/dev/dm-1  partition   8G 1.2G   -2
```

---

# Mostrar bytes exactos

```bash
swapon --show --bytes
```

Seleccionar columnas:

```bash
swapon --show=NAME,TYPE,SIZE,USED,PRIO
```

---

# Consultar `/proc/swaps`

```bash
cat /proc/swaps
```

Ejemplo:

```text
Filename        Type        Size       Used Priority
/dev/dm-1       partition   8388604    1258292 -2
```

Los tamaños se muestran normalmente en KiB.

---

# Consultar con `lsblk`

```bash
lsblk -f
```

Ejemplo:

```text
NAME            FSTYPE      FSVER LABEL UUID                                 MOUNTPOINTS
sda
├─sda1          xfs
└─sda2          LVM2_member
  ├─vg-root     xfs                                                    /
  └─vg-swap     swap        1           47111ce1-4dcc-4da2-9083-573a…
```

---

# Consultar con `blkid`

```bash
sudo blkid
```

Dispositivo específico:

```bash
sudo blkid /dev/vgserver/lvswap
```

Ejemplo:

```text
/dev/mapper/vgserver-lvswap: UUID="47111ce1-4dcc-4da2-9083-573a…" TYPE="swap"
```

---

# Consultar swap en `/etc/fstab`

```bash
grep -E '\sswap\s' /etc/fstab
```

Ejemplo:

```fstab
UUID=47111ce1-4dcc-4da2-9083-573a8a194703 none swap defaults 0 0
```

---

# Tipos de swap

Linux puede utilizar distintos formatos.

| Tipo                | Descripción                              |
| ------------------- | ---------------------------------------- |
| Partición swap      | Partición dedicada                       |
| LV swap             | Logical Volume de LVM                    |
| Archivo swap        | Archivo dentro de un sistema de archivos |
| ZRAM                | Swap comprimida en RAM                   |
| Varios dispositivos | Múltiples áreas con prioridades          |

---

# Partición swap

Una partición swap es una partición de disco dedicada exclusivamente al intercambio.

Ejemplo:

```text
/dev/sdb2
```

Ventajas:

* Configuración tradicional.
* Rendimiento predecible.
* Independencia de archivos ordinarios.
* Fácil identificación.

Desventajas:

* Menor flexibilidad.
* Puede ser más difícil redimensionarla.
* Requiere espacio contiguo en el disco.

---

# Logical Volume swap

Un LV de swap utiliza LVM.

Ejemplo:

```text
/dev/vgserver/lvswap
```

Ventajas:

* Flexible.
* Fácil de ampliar.
* Fácil de reemplazar.
* Compatible con administración LVM.

Desventajas:

* Requiere comprender LVM.
* Debe administrarse cuidadosamente al redimensionar.

---

# Archivo swap

Un archivo swap es un archivo normal preparado mediante `mkswap`.

Ejemplo:

```text
/swapfile
```

Ventajas:

* Fácil de crear.
* No necesita reparticionar.
* Fácil de ampliar o reemplazar.
* Útil en servidores virtuales.

Desventajas:

* Depende del sistema de archivos.
* Puede requerir consideraciones especiales.
* No todos los sistemas de archivos admiten el mismo comportamiento.
* Debe tener permisos estrictos.

---

# ZRAM

ZRAM crea dispositivos de bloques comprimidos dentro de la RAM.

En lugar de escribir directamente en disco, almacena páginas comprimidas en memoria.

Ventajas:

* Más rápida que swap en disco.
* Reduce escrituras.
* Útil en equipos con memoria limitada.
* Común en distribuciones modernas.

Desventajas:

* Consume RAM.
* La compresión usa CPU.
* No sustituye la planificación de memoria.
* Su capacidad efectiva depende de los datos.

---

# Consultar ZRAM

```bash
zramctl
```

Ejemplo:

```text
NAME       ALGORITHM DISKSIZE DATA COMPR TOTAL STREAMS MOUNTPOINT
/dev/zram0 zstd           8G 1.8G  620M  690M       8 [SWAP]
```

---

# Verificar servicio de ZRAM

En Fedora puede existir:

```bash
systemctl status systemd-zram-setup@zram0.service
```

Consultar configuración:

```bash
ls -l /usr/lib/systemd/zram-generator.conf
ls -l /etc/systemd/zram-generator.conf
```

La implementación puede variar según la distribución.

---

# Crear una partición swap

Supongamos que existe una partición disponible:

```text
/dev/sdb2
```

Antes de continuar:

```bash
lsblk -f
sudo fdisk -l /dev/sdb
```

Confirma que no contenga datos necesarios.

---

# Preparar la firma swap

```bash
sudo mkswap /dev/sdb2
```

Ejemplo:

```text
Setting up swapspace version 1, size = 4 GiB
no label, UUID=67f49767-7506-4090-bedd-458e3304e379
```

---

# Crear swap con etiqueta

```bash
sudo mkswap -L SWAP_DATOS /dev/sdb2
```

Consultar:

```bash
sudo blkid /dev/sdb2
```

---

# Activar la partición swap

```bash
sudo swapon /dev/sdb2
```

Verificar:

```bash
swapon --show
free -h
```

---

# Desactivar la partición swap

```bash
sudo swapoff /dev/sdb2
```

Verificar:

```bash
swapon --show
```

Antes de desactivar swap debe existir suficiente RAM disponible para recibir las páginas actualmente almacenadas.

---

# Configurar partición swap en fstab

Obtener UUID:

```bash
sudo blkid /dev/sdb2
```

Agregar:

```fstab
UUID=67f49767-7506-4090-bedd-458e3304e379 none swap defaults 0 0
```

Activar entradas de fstab:

```bash
sudo swapon -a
```

Verificar:

```bash
swapon --show
```

---

# Uso de `none` en fstab

Una entrada swap típica utiliza:

```text
none
```

como punto de montaje porque la swap no se monta en un directorio.

Ejemplo:

```fstab
UUID=... none swap defaults 0 0
```

---

# Crear un Logical Volume swap

Supongamos:

```text
VG: vgserver
LV: lvswap2
Tamaño: 4 GB
```

Crear:

```bash
sudo lvcreate -L 4G -n lvswap2 vgserver
```

Verificar:

```bash
sudo lvs
```

---

# Preparar el LV como swap

```bash
sudo mkswap /dev/vgserver/lvswap2
```

---

# Activar LV swap

```bash
sudo swapon /dev/vgserver/lvswap2
```

Verificar:

```bash
swapon --show
free -h
```

---

# Configurar LV swap persistentemente

Obtener UUID:

```bash
sudo blkid /dev/vgserver/lvswap2
```

Agregar:

```fstab
UUID=UUID_OBTENIDO none swap defaults 0 0
```

Validar:

```bash
sudo swapon -a
swapon --show
```

---

# Crear un archivo swap

Supongamos que se necesita un archivo de 4 GB:

```text
/swapfile
```

Primero verificar espacio:

```bash
df -h /
```

---

# Crear archivo swap con `fallocate`

```bash
sudo fallocate -l 4G /swapfile
```

Verificar:

```bash
ls -lh /swapfile
```

---

# Alternativa con `dd`

```bash
sudo dd if=/dev/zero of=/swapfile bs=1M count=4096 status=progress
```

Este método escribe físicamente los bloques y puede ser más lento.

---

# Permisos obligatorios

```bash
sudo chmod 600 /swapfile
```

Verificar:

```bash
ls -l /swapfile
```

Resultado esperado:

```text
-rw-------. 1 root root 4.0G Jul 28 10:00 /swapfile
```

La swap no debe quedar accesible para otros usuarios.

---

# Preparar archivo swap

```bash
sudo mkswap /swapfile
```

---

# Activar archivo swap

```bash
sudo swapon /swapfile
```

Verificar:

```bash
swapon --show
free -h
```

---

# Configurar archivo swap en fstab

Agregar:

```fstab
/swapfile none swap defaults 0 0
```

Validar:

```bash
sudo swapoff /swapfile
sudo swapon -a
swapon --show
```

---

# Flujo completo para crear un archivo swap

```bash
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

Agregar a `/etc/fstab`:

```fstab
/swapfile none swap defaults 0 0
```

Verificar:

```bash
swapon --show
free -h
```

---

# Error con archivos dispersos

No debe utilizarse simplemente:

```bash
sudo truncate -s 4G /swapfile
```

porque podría crear un archivo disperso.

La swap necesita bloques apropiadamente asignados.

Son preferibles:

```bash
fallocate
```

o:

```bash
dd
```

La compatibilidad de `fallocate` depende del sistema de archivos.

---

# Archivos swap en XFS

XFS soporta archivos swap en configuraciones modernas, pero deben cumplirse las restricciones correspondientes.

Antes de activarlo:

```bash
sudo swapon /swapfile
```

Si aparece un error relacionado con holes o copy-on-write, debe revisarse cómo fue creado el archivo y las características del sistema.

Consultar:

```bash
filefrag -v /swapfile
```

---

# Archivos swap en Btrfs

Btrfs utiliza copy-on-write, lo cual requiere consideraciones especiales.

En sistemas modernos puede crearse mediante:

```bash
sudo btrfs filesystem mkswapfile --size 4G /swapfile
```

Luego:

```bash
sudo swapon /swapfile
```

Si la herramienta no está disponible, deben seguirse los procedimientos específicos de la versión de Btrfs instalada.

No debe crearse un archivo swap genérico sobre Btrfs sin validar compatibilidad.

---

# Activar todas las áreas swap de fstab

```bash
sudo swapon -a
```

Modo detallado:

```bash
sudo swapon -av
```

---

# Desactivar todas las áreas swap

```bash
sudo swapoff -a
```

Advertencia: puede causar falta de memoria si la RAM no puede recibir todas las páginas.

Antes:

```bash
free -h
swapon --show
```

---

# Reactivar todas las áreas

```bash
sudo swapon -a
```

---

# Mostrar resumen numérico

```bash
swapon --summary
```

En versiones modernas, se recomienda:

```bash
swapon --show
```

---

# Prioridad de swap

Linux puede utilizar varias áreas swap.

Cada una puede tener una prioridad.

Consultar:

```bash
swapon --show=NAME,TYPE,SIZE,USED,PRIO
```

Ejemplo:

```text
NAME       TYPE      SIZE USED PRIO
/dev/zram0 partition   8G 1.5G  100
/swapfile  file        4G   0B   10
/dev/sdb2  partition   8G   0B   -2
```

Linux utiliza primero la swap con mayor prioridad.

---

# Configurar prioridad manual

Activar:

```bash
sudo swapon -p 50 /swapfile
```

En `/etc/fstab`:

```fstab
/swapfile none swap defaults,pri=50 0 0
```

Otra área:

```fstab
UUID=... none swap defaults,pri=10 0 0
```

---

# Swap con la misma prioridad

Si varias áreas tienen la misma prioridad, Linux puede distribuir páginas entre ellas.

Ejemplo:

```fstab
UUID=SWAP1 none swap defaults,pri=20 0 0
UUID=SWAP2 none swap defaults,pri=20 0 0
```

Esto puede mejorar el paralelismo si están en dispositivos físicos diferentes.

---

# No asumir mayor rendimiento en el mismo disco

Dos áreas swap en el mismo disco no ofrecen necesariamente mejor rendimiento.

Pueden incluso aumentar movimientos de lectura y escritura.

La distribución tiene más sentido cuando las áreas están en dispositivos independientes.

---

# ¿Qué es `vm.swappiness`?

`vm.swappiness` controla la tendencia del kernel a mover páginas hacia swap.

Consultar:

```bash
sysctl vm.swappiness
```

También:

```bash
cat /proc/sys/vm/swappiness
```

Ejemplo:

```text
vm.swappiness = 60
```

---

# Interpretar swappiness

El valor normalmente está entre:

```text
0 y 200
```

En muchos sistemas:

* Valores bajos reducen la tendencia a usar swap.
* Valores altos permiten utilizarla más activamente.
* El comportamiento exacto depende del kernel.

No significa un porcentaje directo de RAM.

---

# Cambiar temporalmente swappiness

```bash
sudo sysctl -w vm.swappiness=10
```

Verificar:

```bash
sysctl vm.swappiness
```

El cambio desaparece al reiniciar.

---

# Configurar swappiness persistentemente

Crear archivo:

```bash
sudo nano /etc/sysctl.d/99-swappiness.conf
```

Contenido:

```ini
vm.swappiness = 10
```

Aplicar:

```bash
sudo sysctl --system
```

Verificar:

```bash
sysctl vm.swappiness
```

---

# Seleccionar un valor

No existe un valor universal.

Debe evaluarse según:

* Tipo de servidor.
* Cantidad de RAM.
* Latencia del almacenamiento.
* Aplicaciones.
* Base de datos.
* Uso de contenedores.
* Comportamiento histórico.
* Recomendaciones del fabricante.

Valores bajos son comunes en servidores de bases de datos, pero no debe establecerse `0` automáticamente sin análisis.

---

# Swappiness igual a cero

```ini
vm.swappiness = 0
```

No desactiva completamente la swap.

Reduce fuertemente la tendencia a usarla y, según el kernel, puede reservarla para situaciones de alta presión.

Para desactivar swap realmente:

```bash
sudo swapoff -a
```

---

# `vm.vfs_cache_pressure`

Otro parámetro relacionado con memoria es:

```bash
sysctl vm.vfs_cache_pressure
```

Controla la tendencia del kernel a recuperar cachés de inodos y dentries.

Valor habitual:

```text
100
```

No debe modificarse sin mediciones y una razón clara.

---

# Consultar actividad de swap con `vmstat`

```bash
vmstat 1
```

Ejemplo:

```text
procs -----------memory---------- ---swap-- -----io----
 r  b   swpd   free   buff  cache   si   so    bi    bo
 1  0 524288 250000  12000 620000    0    0     4     8
 0  0 524288 248000  12000 621000    0    0     0     0
```

---

# Columnas importantes de vmstat

| Columna | Significado                         |
| ------- | ----------------------------------- |
| `swpd`  | Memoria swap utilizada              |
| `si`    | Datos leídos desde swap hacia RAM   |
| `so`    | Datos escritos desde RAM hacia swap |
| `free`  | Memoria libre                       |
| `cache` | Memoria de caché                    |
| `r`     | Procesos ejecutables                |
| `b`     | Procesos bloqueados                 |

Una actividad continua y elevada en `si` y `so` puede indicar presión de memoria.

---

# Consultar con `sar`

Si está instalado `sysstat`:

```bash
sar -W 1 10
```

Columnas:

| Columna     | Descripción                             |
| ----------- | --------------------------------------- |
| `pswpin/s`  | Páginas leídas desde swap por segundo   |
| `pswpout/s` | Páginas escritas hacia swap por segundo |

Memoria:

```bash
sar -r 1 10
```

---

# Instalar sysstat

```bash
sudo dnf install sysstat
```

Activar:

```bash
sudo systemctl enable --now sysstat
```

---

# Consultar procesos con mayor memoria

```bash
ps aux --sort=-%mem | head -20
```

Con columnas específicas:

```bash
ps -eo pid,user,comm,%mem,rss,vsz --sort=-rss | head -20
```

---

# Interpretar RSS y VSZ

| Campo   | Descripción                |
| ------- | -------------------------- |
| RSS     | Memoria física residente   |
| VSZ     | Espacio de memoria virtual |
| `%MEM`  | Porcentaje de RAM          |
| COMMAND | Proceso                    |

Un VSZ alto no significa necesariamente que toda esa memoria esté ocupando RAM.

---

# Consultar memoria de un proceso

```bash
grep -E 'VmRSS|VmSize|VmSwap' /proc/PID/status
```

Ejemplo:

```bash
grep -E 'VmRSS|VmSize|VmSwap' /proc/4215/status
```

---

# Consultar swap por proceso

Puede utilizarse:

```bash
for pid in /proc/[0-9]*; do
    if [ -r "$pid/status" ]; then
        awk '
        /^Name:/ {name=$2}
        /^Pid:/ {id=$2}
        /^VmSwap:/ {
            if ($2 > 0)
                printf "%10d KiB  PID=%-8s %s\n", $2, id, name
        }' "$pid/status"
    fi
done | sort -n
```

Los procesos con mayor swap aparecerán al final.

---

# Versión resumida para procesos con swap

```bash
for status in /proc/[0-9]*/status; do
    awk '
    /^Name:/ {name=$2}
    /^Pid:/ {pid=$2}
    /^VmSwap:/ {
        if ($2 > 0)
            print $2, pid, name
    }' "$status" 2>/dev/null
done | sort -n | tail -20
```

---

# Uso de `smem`

Instalar:

```bash
sudo dnf install smem
```

Consultar:

```bash
sudo smem -rs swap
```

Columnas habituales:

* USS.
* PSS.
* RSS.
* Swap.

La disponibilidad del paquete depende de la distribución.

---

# Presión de memoria

Un sistema puede estar bajo presión de memoria cuando:

* `available` permanece muy bajo.
* Existe actividad constante de swap.
* `si` y `so` son altos.
* Aparecen procesos bloqueados.
* Aumenta la latencia.
* Se ejecuta el OOM Killer.
* Los servicios comienzan a fallar.
* El almacenamiento muestra I/O elevado.

---

# Thrashing

Thrashing ocurre cuando el sistema pasa gran parte del tiempo moviendo páginas entre RAM y swap.

Síntomas:

* Servidor extremadamente lento.
* CPU no necesariamente al 100%.
* I/O de disco alto.
* Valores continuos de `si` y `so`.
* Aplicaciones sin respuesta.
* Latencia elevada.
* Muchos procesos bloqueados.

La solución puede requerir:

* Agregar RAM.
* Limitar procesos.
* Corregir fugas.
* Ajustar aplicaciones.
* Reducir concurrencia.
* Redistribuir servicios.
* Ajustar parámetros con pruebas.
* Reiniciar controladamente procesos defectuosos.

---

# OOM Killer

OOM significa:

```text
Out Of Memory
```

Cuando Linux no puede satisfacer una solicitud de memoria, puede ejecutar el OOM Killer.

Este mecanismo selecciona uno o más procesos para finalizarlos y liberar memoria.

---

# Consultar eventos OOM

```bash
journalctl -k | grep -Ei "out of memory|oom|killed process"
```

Desde el arranque actual:

```bash
journalctl -b -k | grep -Ei "out of memory|oom|killed process"
```

Ejemplo:

```text
Out of memory: Killed process 8421 (java) total-vm:...
```

---

# Consultar `oom_score`

```bash
cat /proc/PID/oom_score
```

Ajuste:

```bash
cat /proc/PID/oom_score_adj
```

Valores altos aumentan la posibilidad de selección.

Valores bajos reducen la posibilidad.

---

# Ajustar temporalmente OOM score

```bash
echo -500 | sudo tee /proc/PID/oom_score_adj
```

Rango habitual:

```text
-1000 a 1000
```

`-1000` puede hacer un proceso prácticamente inmune al OOM Killer.

Debe utilizarse con extrema precaución, porque proteger demasiados procesos puede empeorar la recuperación del sistema.

---

# Memoria y systemd

Systemd puede aplicar límites de memoria a servicios.

Consultar:

```bash
systemctl show nombre-servicio \
-p MemoryCurrent \
-p MemoryMax \
-p MemoryHigh
```

---

# Configurar límites en una unidad

```ini
[Service]
MemoryHigh=4G
MemoryMax=6G
```

* `MemoryHigh` aplica presión antes del límite máximo.
* `MemoryMax` establece un límite estricto.

Después:

```bash
sudo systemctl daemon-reload
sudo systemctl restart nombre-servicio
```

Debe probarse cuidadosamente para evitar finalizaciones inesperadas.

---

# Consultar límites de un proceso

```bash
cat /proc/PID/limits
```

Memoria virtual:

```bash
ulimit -v
```

La administración moderna suele realizarse mediante cgroups y systemd.

---

# Ampliar un LV swap existente

Supongamos:

```text
/dev/vgserver/lvswap
```

Antes:

```bash
swapon --show
free -h
sudo lvs
sudo vgs
```

---

# Paso 1: desactivar swap

```bash
sudo swapoff /dev/vgserver/lvswap
```

Si falla por falta de memoria, no debe forzarse.

Puede ser necesario:

* Detener aplicaciones.
* Crear una segunda swap temporal.
* Programar mantenimiento.
* Agregar RAM.
* Reiniciar controladamente.

---

# Paso 2: ampliar LV

Agregar 2 GB:

```bash
sudo lvextend -L +2G /dev/vgserver/lvswap
```

---

# Paso 3: recrear la firma

```bash
sudo mkswap /dev/vgserver/lvswap
```

Advertencia: `mkswap` genera normalmente un nuevo UUID.

---

# Paso 4: consultar UUID

```bash
sudo blkid /dev/vgserver/lvswap
```

Si `/etc/fstab` utiliza el UUID anterior, debe actualizarse.

---

# Paso 5: activar

```bash
sudo swapon /dev/vgserver/lvswap
```

Verificar:

```bash
swapon --show
free -h
```

---

# Alternativa más segura: crear otra swap

En lugar de modificar la existente:

```bash
sudo lvcreate -L 2G -n lvswap_extra vgserver
sudo mkswap /dev/vgserver/lvswap_extra
sudo swapon /dev/vgserver/lvswap_extra
```

Agregar a fstab:

```fstab
UUID=UUID_NUEVO none swap defaults 0 0
```

Esto evita desactivar la swap principal durante la creación.

---

# Ampliar un archivo swap

Un archivo swap no suele ampliarse mientras está activo.

Supongamos:

```text
/swapfile
```

---

# Paso 1: comprobar memoria

```bash
free -h
swapon --show
```

---

# Paso 2: desactivar

```bash
sudo swapoff /swapfile
```

---

# Paso 3: recrear con nuevo tamaño

Eliminar:

```bash
sudo rm /swapfile
```

Crear uno nuevo de 8 GB:

```bash
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
```

---

# Paso 4: activar

```bash
sudo swapon /swapfile
```

Verificar:

```bash
swapon --show
free -h
```

Si `/etc/fstab` usa la ruta `/swapfile`, no necesita actualización.

---

# Crear una segunda swap temporal

Cuando no se puede desactivar la swap principal por falta de memoria:

```bash
sudo fallocate -l 4G /swaptemp
sudo chmod 600 /swaptemp
sudo mkswap /swaptemp
sudo swapon /swaptemp
```

Luego puede intentarse:

```bash
sudo swapoff /swapfile
```

Al finalizar:

```bash
sudo swapoff /swaptemp
sudo rm /swaptemp
```

Solo debe ejecutarse después de confirmar que la swap definitiva está activa.

---

# Eliminar una swap

## Paso 1: identificar

```bash
swapon --show
```

## Paso 2: desactivar

```bash
sudo swapoff /dev/sdb2
```

## Paso 3: retirar de fstab

Editar:

```bash
sudo nano /etc/fstab
```

Eliminar o comentar:

```fstab
# UUID=... none swap defaults 0 0
```

## Paso 4: borrar firma si se reutilizará

```bash
sudo wipefs -a /dev/sdb2
```

Debe utilizarse únicamente si se desea reutilizar el dispositivo y se ha confirmado que es el correcto.

---

# Eliminar un archivo swap

```bash
sudo swapoff /swapfile
```

Retirar de `/etc/fstab`.

Luego:

```bash
sudo rm /swapfile
```

Verificar:

```bash
swapon --show
```

---

# Etiquetas swap

Asignar etiqueta al crear:

```bash
sudo mkswap -L SWAP_SERVIDOR /dev/sdb2
```

Consultar:

```bash
sudo blkid /dev/sdb2
```

Usar en fstab:

```fstab
LABEL=SWAP_SERVIDOR none swap defaults 0 0
```

El UUID suele ser más seguro si las etiquetas no son estrictamente únicas.

---

# Tamaño recomendado de swap

No existe una regla universal.

Depende de:

* Cantidad de RAM.
* Hibernación.
* Tipo de carga.
* Bases de datos.
* Picos de memoria.
* Aplicaciones.
* Política de la empresa.
* Tipo de almacenamiento.
* SLA.
* Contenedores.

Una tabla orientativa puede ser:

|           RAM | Swap orientativa sin hibernación |
| ------------: | -------------------------------: |
| Menos de 2 GB |               1 a 2 veces la RAM |
|      2 a 8 GB |         Similar a la RAM o menor |
|     8 a 32 GB |             4 a 8 GB según carga |
|  Más de 32 GB |        Según la carga y política |

Estos valores no son reglas obligatorias.

Para hibernación puede requerirse una swap comparable con la memoria utilizada, además de configuración adicional.

---

# Swap en servidores de bases de datos

En bases de datos debe evitarse que el proceso principal utilice swap excesivamente.

Debe revisarse:

* Memoria máxima del motor.
* Buffers.
* Caché.
* Procesos concurrentes.
* Huge Pages.
* Swappiness.
* Latencia del almacenamiento.
* Recomendaciones del fabricante.

La swap puede actuar como margen de seguridad, pero un uso constante puede afectar severamente el rendimiento.

---

# PostgreSQL y swap

En PostgreSQL deben revisarse:

* `shared_buffers`.
* `work_mem`.
* `maintenance_work_mem`.
* Número de conexiones.
* Procesos paralelos.
* Memoria del sistema operativo.
* Caché del sistema de archivos.

Un `work_mem` alto puede multiplicarse por:

* Operaciones.
* Consultas.
* Sesiones.
* Procesos paralelos.

No debe calcularse únicamente por conexión.

---

# SQL Server en Linux y swap

Para SQL Server deben revisarse:

* Memoria máxima configurada.
* Memoria reservada para Linux.
* Buffer pool.
* Servicios adicionales.
* Contenedores.
* Monitorización.
* Recomendaciones oficiales de la versión.

Consultar memoria configurada desde SQL Server:

```sql
SELECT
    name,
    value,
    value_in_use
FROM sys.configurations
WHERE name IN
(
    'min server memory (MB)',
    'max server memory (MB)'
);
```

El sistema operativo debe conservar memoria suficiente fuera del límite de SQL Server.

---

# Swap en contenedores

Los contenedores pueden tener límites de memoria y swap mediante cgroups.

Consultar un servicio:

```bash
systemctl show nombre-servicio \
-p MemoryMax \
-p MemorySwapMax
```

En cgroup v2 puede existir:

```text
memory.max
memory.swap.max
```

Debe configurarse según el runtime utilizado.

---

# Verificar cgroup v2

```bash
stat -fc %T /sys/fs/cgroup
```

Resultado esperado:

```text
cgroup2fs
```

---

# Consultar presión de memoria PSI

Linux puede publicar Pressure Stall Information.

```bash
cat /proc/pressure/memory
```

Ejemplo:

```text
some avg10=0.25 avg60=0.10 avg300=0.05 total=...
full avg10=0.03 avg60=0.01 avg300=0.00 total=...
```

* `some`: al menos una tarea espera por memoria.
* `full`: todas las tareas relevantes están detenidas por presión.

Valores sostenidos pueden indicar problemas.

---

# Consultar memoria desde `/proc/meminfo`

```bash
cat /proc/meminfo
```

Filtrar:

```bash
grep -E \
'MemTotal|MemFree|MemAvailable|Buffers|Cached|SwapTotal|SwapFree|Dirty|Writeback' \
/proc/meminfo
```

---

# Campos importantes de `/proc/meminfo`

| Campo          | Descripción                    |
| -------------- | ------------------------------ |
| `MemTotal`     | RAM total                      |
| `MemFree`      | RAM completamente libre        |
| `MemAvailable` | Estimación disponible          |
| `Cached`       | Caché de archivos              |
| `SwapTotal`    | Swap total                     |
| `SwapFree`     | Swap libre                     |
| `Dirty`        | Datos pendientes de escritura  |
| `Writeback`    | Datos que se están escribiendo |

---

# Limpiar swap sin reiniciar

En ciertos casos puede quererse mover páginas nuevamente a RAM:

```bash
sudo swapoff -a
sudo swapon -a
```

Esto solo debe ejecutarse cuando exista memoria disponible suficiente.

No debe convertirse en una tarea periódica.

El hecho de que existan páginas en swap no significa necesariamente que haya un problema. Linux puede mantener allí páginas inactivas aunque luego exista RAM libre.

---

# Swap usada con RAM disponible

Es normal que Linux no devuelva inmediatamente todas las páginas desde swap hacia RAM.

Puede mantener páginas inactivas en swap y utilizar RAM para caché.

No debe limpiarse la swap únicamente porque:

```bash
swapon --show
```

muestre uso.

Debe evaluarse:

* Actividad actual de `si` y `so`.
* Latencia.
* Memoria disponible.
* Rendimiento.
* Estado de aplicaciones.

---

# Diagnóstico de uso elevado de swap

## Paso 1: revisar memoria

```bash
free -h
```

## Paso 2: revisar swap

```bash
swapon --show
```

## Paso 3: revisar actividad

```bash
vmstat 1 10
```

## Paso 4: revisar procesos

```bash
ps -eo pid,user,comm,%mem,rss,vsz --sort=-rss | head -20
```

## Paso 5: revisar swap por proceso

```bash
for status in /proc/[0-9]*/status; do
    awk '
    /^Name:/ {name=$2}
    /^Pid:/ {pid=$2}
    /^VmSwap:/ {
        if ($2 > 0)
            print $2, pid, name
    }' "$status" 2>/dev/null
done | sort -n | tail -20
```

## Paso 6: revisar OOM

```bash
journalctl -k | grep -Ei "out of memory|oom|killed process"
```

## Paso 7: revisar I/O

```bash
iostat -xz 1
```

Puede requerir:

```bash
sudo dnf install sysstat
```

---

# Diagnóstico: `swapon failed: Operation not permitted`

Posibles causas:

* Entorno contenedor restringido.
* Permisos insuficientes.
* Política de seguridad.
* Archivo incompatible.
* Filesystem no compatible.
* Área ya activa.

Verificar:

```bash
id
swapon --show
ls -l /swapfile
findmnt -T /swapfile
```

---

# Diagnóstico: permisos inseguros

Error posible:

```text
swapon: /swapfile: insecure permissions
```

Corregir:

```bash
sudo chmod 600 /swapfile
sudo chown root:root /swapfile
```

---

# Diagnóstico: archivo con huecos

Error:

```text
swapon failed: Invalid argument
```

Consultar:

```bash
filefrag -v /swapfile
```

Recrear:

```bash
sudo swapoff /swapfile 2>/dev/null
sudo rm -f /swapfile
sudo dd if=/dev/zero of=/swapfile bs=1M count=4096 status=progress
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

---

# Diagnóstico: UUID de swap cambió

Después de ejecutar nuevamente:

```bash
sudo mkswap /dev/vgserver/lvswap
```

puede generarse otro UUID.

Consultar:

```bash
sudo blkid /dev/vgserver/lvswap
```

Comparar:

```bash
grep swap /etc/fstab
```

Actualizar fstab si es necesario.

---

# Diagnóstico: `swapoff` falla

Puede aparecer:

```text
swapoff failed: Cannot allocate memory
```

Significa que no hay RAM suficiente para recibir las páginas.

Opciones:

* Detener procesos no críticos.
* Crear swap adicional.
* Reducir carga.
* Programar mantenimiento.
* Reiniciar de forma controlada.
* Agregar memoria.

No debe forzarse la operación.

---

# Diagnóstico: swap no se activa al reiniciar

Revisar:

```bash
grep swap /etc/fstab
```

Verificar UUID:

```bash
sudo blkid
```

Probar:

```bash
sudo swapon -av
```

Revisar logs:

```bash
journalctl -b | grep -i swap
```

---

# Diagnóstico: uso alto de swap sin actividad

Si `USED` es alto pero `si` y `so` permanecen en cero, puede tratarse de páginas antiguas e inactivas.

Consultar:

```bash
vmstat 1 10
```

Si no hay actividad significativa, no necesariamente existe un problema.

---

# Diagnóstico: actividad constante de swap

Si `si` y `so` son altos de forma continua:

* Existe presión real de memoria.
* Puede haber thrashing.
* Debe analizarse el consumo.
* Revisar servicios.
* Ajustar memoria.
* Añadir RAM.
* Corregir aplicación.

---

# Monitoreo básico mediante script

```bash
#!/bin/bash

set -euo pipefail

echo "=== Fecha ==="
date

echo
echo "=== Memoria ==="
free -h

echo
echo "=== Swap activa ==="
swapon --show

echo
echo "=== Swappiness ==="
sysctl vm.swappiness

echo
echo "=== Procesos por RSS ==="
ps -eo pid,user,comm,%mem,rss,vsz --sort=-rss | head -15

echo
echo "=== Eventos OOM recientes ==="
journalctl -k --since "24 hours ago" |
grep -Ei "out of memory|oom|killed process" || true
```

---

# Script para crear un archivo swap

```bash
#!/bin/bash

set -euo pipefail

SWAPFILE="/swapfile"
SIZE="4G"

if [[ $EUID -ne 0 ]]; then
    echo "ERROR: Debe ejecutarse como root."
    exit 1
fi

if swapon --show=NAME --noheadings | grep -qx "$SWAPFILE"; then
    echo "ERROR: $SWAPFILE ya está activo."
    exit 1
fi

if [[ -e "$SWAPFILE" ]]; then
    echo "ERROR: $SWAPFILE ya existe."
    exit 1
fi

fallocate -l "$SIZE" "$SWAPFILE"
chmod 600 "$SWAPFILE"
chown root:root "$SWAPFILE"
mkswap "$SWAPFILE"
swapon "$SWAPFILE"

if ! grep -qE "^${SWAPFILE//\//\\/}[[:space:]]" /etc/fstab; then
    printf '%s\n' "$SWAPFILE none swap defaults 0 0" >> /etc/fstab
fi

echo "Swap creada correctamente:"
swapon --show
free -h
```

---

# Laboratorio 1: consultar la swap existente

```bash
free -h
swapon --show
cat /proc/swaps
grep swap /etc/fstab
```

Documenta:

* Dispositivo.
* Tipo.
* Tamaño.
* Uso.
* Prioridad.
* Persistencia.

---

# Laboratorio 2: crear una partición swap

Supongamos:

```text
/dev/sdb2
```

Crear firma:

```bash
sudo mkswap -L SWAP_LAB /dev/sdb2
```

Activar:

```bash
sudo swapon /dev/sdb2
```

Verificar:

```bash
swapon --show
free -h
```

---

# Laboratorio 3: configurar persistencia

Obtener UUID:

```bash
sudo blkid /dev/sdb2
```

Agregar:

```fstab
UUID=UUID_OBTENIDO none swap defaults 0 0
```

Probar:

```bash
sudo swapoff /dev/sdb2
sudo swapon -a
swapon --show
```

---

# Laboratorio 4: crear LV swap

```bash
sudo lvcreate -L 2G -n lvswap_lab vg_lab
sudo mkswap /dev/vg_lab/lvswap_lab
sudo swapon /dev/vg_lab/lvswap_lab
```

Verificar:

```bash
sudo lvs
swapon --show
```

---

# Laboratorio 5: crear archivo swap

```bash
sudo fallocate -l 2G /swap_lab
sudo chmod 600 /swap_lab
sudo mkswap /swap_lab
sudo swapon /swap_lab
```

Verificar:

```bash
swapon --show
free -h
```

---

# Laboratorio 6: configurar prioridad

Desactivar:

```bash
sudo swapoff /swap_lab
```

Activar con prioridad 50:

```bash
sudo swapon -p 50 /swap_lab
```

Verificar:

```bash
swapon --show=NAME,TYPE,SIZE,USED,PRIO
```

---

# Laboratorio 7: modificar swappiness

Consultar:

```bash
sysctl vm.swappiness
```

Cambiar temporalmente:

```bash
sudo sysctl -w vm.swappiness=20
```

Crear persistencia:

```bash
echo 'vm.swappiness = 20' |
sudo tee /etc/sysctl.d/99-lab-swappiness.conf
```

Aplicar:

```bash
sudo sysctl --system
```

Verificar:

```bash
sysctl vm.swappiness
```

---

# Laboratorio 8: observar actividad

```bash
vmstat 1 10
```

Identifica:

* `swpd`.
* `si`.
* `so`.
* `free`.
* `cache`.

---

# Laboratorio 9: consultar memoria por proceso

```bash
ps -eo pid,user,comm,%mem,rss,vsz --sort=-rss | head -15
```

Selecciona un PID:

```bash
grep -E 'Name|VmRSS|VmSize|VmSwap' /proc/PID/status
```

---

# Laboratorio 10: eliminar el archivo de laboratorio

```bash
sudo swapoff /swap_lab
```

Eliminar la entrada de `/etc/fstab` si fue agregada.

Luego:

```bash
sudo rm /swap_lab
```

Verificar:

```bash
swapon --show
```

---

# Práctica RHCSA: crear swap persistente

Una tarea típica puede solicitar:

```text
Cree una nueva área swap de 512 MiB y configúrela de forma persistente.
```

## Opción con LVM

```bash
sudo lvcreate -L 512M -n lvswap vg_lab
sudo mkswap /dev/vg_lab/lvswap
sudo swapon /dev/vg_lab/lvswap
sudo blkid /dev/vg_lab/lvswap
```

Agregar:

```fstab
UUID=UUID_OBTENIDO none swap defaults 0 0
```

Verificar:

```bash
sudo swapoff /dev/vg_lab/lvswap
sudo swapon -a
swapon --show
```

---

# Práctica RHCSA: crear archivo swap

```bash
sudo fallocate -l 512M /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

Agregar:

```fstab
/swapfile none swap defaults 0 0
```

Verificar:

```bash
sudo swapoff /swapfile
sudo swapon -a
swapon --show
```

---

# Comandos principales

| Comando                | Función                             |
| ---------------------- | ----------------------------------- |
| `free -h`              | Consultar RAM y swap                |
| `swapon --show`        | Mostrar áreas activas               |
| `swapon`               | Activar swap                        |
| `swapoff`              | Desactivar swap                     |
| `mkswap`               | Crear firma swap                    |
| `lsblk -f`             | Identificar dispositivos            |
| `blkid`                | Consultar UUID                      |
| `vmstat`               | Consultar actividad de memoria      |
| `sar -W`               | Estadísticas de swap                |
| `sysctl vm.swappiness` | Consultar swappiness                |
| `fallocate`            | Crear archivo con tamaño asignado   |
| `dd`                   | Crear archivo escribiendo bloques   |
| `zramctl`              | Consultar ZRAM                      |
| `ps`                   | Consultar memoria por proceso       |
| `journalctl -k`        | Revisar OOM y errores               |
| `smem`                 | Analizar memoria y swap por proceso |

---

# Archivos y rutas importantes

| Ruta                      | Función                          |
| ------------------------- | -------------------------------- |
| `/proc/swaps`             | Áreas swap activas               |
| `/proc/meminfo`           | Información de memoria           |
| `/proc/sys/vm/swappiness` | Valor actual de swappiness       |
| `/proc/PID/status`        | Memoria de un proceso            |
| `/proc/PID/oom_score`     | Puntuación OOM                   |
| `/proc/PID/oom_score_adj` | Ajuste OOM                       |
| `/proc/pressure/memory`   | Presión de memoria               |
| `/etc/fstab`              | Persistencia de swap             |
| `/etc/sysctl.conf`        | Parámetros persistentes          |
| `/etc/sysctl.d/`          | Archivos de configuración sysctl |
| `/sys/block/zram0/`       | Información ZRAM                 |

---

# Buenas prácticas

* Mantén al menos una estrategia de swap apropiada para el servidor.
* No utilices swap como sustituto de RAM.
* Revisa `MemAvailable`, no solamente `MemFree`.
* Supervisa actividad `si` y `so`.
* Investiga uso constante de swap.
* Utiliza permisos `600` para archivos swap.
* Prefiere UUID en particiones o LV swap.
* Verifica `/etc/fstab` después de crear o recrear swap.
* Recuerda que `mkswap` puede cambiar el UUID.
* No ejecutes `swapoff -a` sin verificar memoria disponible.
* Evita limpiar swap de forma rutinaria.
* Evalúa `swappiness` según la aplicación.
* Mantén memoria suficiente para el sistema operativo.
* Configura límites de memoria en servicios problemáticos.
* Revisa eventos OOM.
* Monitorea ZRAM si está habilitada.
* Documenta tamaño, tipo y prioridad.
* Prueba la persistencia con `swapoff` y `swapon -a`.
* Conserva acceso administrativo durante cambios.
* Realiza cambios importantes en una ventana controlada.

---

# Errores comunes

## Crear un archivo swap sin permisos seguros

Incorrecto:

```bash
sudo fallocate -l 4G /swapfile
sudo mkswap /swapfile
```

Correcto:

```bash
sudo chmod 600 /swapfile
sudo mkswap /swapfile
```

---

## Usar `truncate` sin asignar bloques

Incorrecto:

```bash
sudo truncate -s 4G /swapfile
```

Puede crear un archivo disperso.

Preferible:

```bash
sudo fallocate -l 4G /swapfile
```

o:

```bash
sudo dd if=/dev/zero of=/swapfile bs=1M count=4096 status=progress
```

---

## Desactivar swap sin memoria suficiente

Incorrecto:

```bash
sudo swapoff -a
```

sin revisar:

```bash
free -h
```

Puede provocar errores o finalizar procesos.

---

## Olvidar `/etc/fstab`

Activar:

```bash
sudo swapon /dev/sdb2
```

no garantiza persistencia.

Debe agregarse:

```fstab
UUID=... none swap defaults 0 0
```

---

## Mantener un UUID antiguo

Después de:

```bash
sudo mkswap /dev/vgserver/lvswap
```

debe verificarse:

```bash
sudo blkid /dev/vgserver/lvswap
```

---

## Confundir swap usada con problema activo

Una swap con páginas utilizadas no significa necesariamente presión actual.

Debe revisarse:

```bash
vmstat 1
```

Si `si` y `so` permanecen cerca de cero, puede no existir actividad problemática.

---

## Establecer swappiness en cero sin análisis

Un valor extremadamente bajo puede limitar la flexibilidad del kernel.

Debe validarse según la carga.

---

## Crear swap en Btrfs sin preparación

Los archivos swap en Btrfs requieren procedimientos compatibles con copy-on-write.

Debe utilizarse la herramienta apropiada de Btrfs o validar la versión instalada.

---

## Usar swap lenta para una aplicación crítica

Si el dispositivo tiene latencia elevada, el uso intensivo de swap puede degradar severamente la aplicación.

---

## Proteger todos los procesos contra OOM

Asignar:

```text
oom_score_adj = -1000
```

a demasiados servicios puede impedir que el sistema recupere memoria.

---

# Resumen

En este capítulo aprendiste a:

* Comprender la memoria virtual y la función de swap.
* Diferenciar RAM, swap en disco y ZRAM.
* Consultar memoria con `free`, `swapon`, `vmstat` y `/proc`.
* Crear particiones swap.
* Crear Logical Volumes swap.
* Crear archivos swap.
* Activar y desactivar áreas swap.
* Configurar persistencia en `/etc/fstab`.
* Administrar prioridades.
* Ajustar `vm.swappiness`.
* Analizar actividad de intercambio.
* Detectar presión de memoria y thrashing.
* Consultar procesos con consumo elevado.
* Comprender el OOM Killer.
* Ampliar o reemplazar una swap.
* Diagnosticar errores de permisos, UUID y memoria.
* Aplicar procedimientos seguros y orientados al examen RHCSA.
