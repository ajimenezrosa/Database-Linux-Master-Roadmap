# 54. Laboratorio de Administración de LVM (Logical Volume Manager) — Fase 1

> **Módulo 5 — Administración de Almacenamiento**
>
> **Archivo:** `54-laboratorio-lvm.md`
>
> **Nivel:** RHCSA
>
> **Objetivo:** Aprender a implementar, administrar y solucionar problemas relacionados con LVM en entornos empresariales Linux mediante ejercicios prácticos similares al examen RHCSA.

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Comprender la arquitectura de LVM.
- Identificar discos y particiones.
- Crear Physical Volumes (PV).
- Crear Volume Groups (VG).
- Crear Logical Volumes (LV).
- Formatear volúmenes.
- Montarlos correctamente.
- Configurar montajes permanentes mediante `/etc/fstab`.
- Consultar el estado completo de LVM.
- Resolver errores comunes.
- Aplicar buenas prácticas empresariales.

---

# Introducción

En servidores Linux modernos, el almacenamiento rara vez se administra utilizando particiones tradicionales.

Las empresas requieren:

- Crecimiento dinámico.
- Flexibilidad.
- Facilidad para agregar discos.
- Administración centralizada.
- Escalabilidad.

Para resolver estas necesidades existe **LVM (Logical Volume Manager)**.

---

# Escenario empresarial

La empresa **TechData Solutions** ha adquirido un nuevo servidor para alojar aplicaciones internas.

El servidor dispone inicialmente de:

| Disco | Tamaño | Uso |
|---------|---------|----------------|
| /dev/sda | 100 GB | Sistema Operativo |
| /dev/sdb | 50 GB | Disponible |
| /dev/sdc | 80 GB | Disponible |

El administrador debe implementar LVM para crear un almacenamiento flexible.

---

# Arquitectura de LVM

```text
                 Discos Físicos

          /dev/sdb        /dev/sdc

               │              │

               ▼              ▼

       Physical Volume   Physical Volume

               │              │

               └──────┬───────┘

                      ▼

              Volume Group (VGDATA)

                      │

      ┌───────────────┼─────────────────┐

      ▼               ▼                 ▼

    LV_APP         LV_DB           LV_BACKUP

      │               │                 │

      ▼               ▼                 ▼

   XFS Filesystem  XFS Filesystem   XFS Filesystem

      │               │                 │

      ▼               ▼                 ▼

 /appdata        /database        /backups
```

---

# Flujo de trabajo

```text
Disco

↓

Physical Volume

↓

Volume Group

↓

Logical Volume

↓

Filesystem

↓

Mount Point

↓

Producción
```

---

# Verificar discos disponibles

```bash
lsblk
```

Resultado esperado:

```text
NAME   SIZE TYPE

sda    100G disk
├─sda1
├─sda2

sdb     50G disk

sdc     80G disk
```

---

# Consultar dispositivos de bloques

```bash
blkid
```

---

También:

```bash
fdisk -l
```

---

# Verificar si existen Physical Volumes

```bash
pvs
```

O bien:

```bash
pvdisplay
```

Resultado inicial:

```text
No physical volumes found
```

---

# Verificar Volume Groups

```bash
vgs
```

---

```bash
vgdisplay
```

---

# Verificar Logical Volumes

```bash
lvs
```

---

```bash
lvdisplay
```

---

# Paso 1: Inicializar un Physical Volume

Convertir `/dev/sdb` en un PV.

```bash
sudo pvcreate /dev/sdb
```

Resultado:

```text
Physical volume "/dev/sdb" successfully created.
```

---

# Consultar información del PV

```bash
pvs
```

---

```bash
pvdisplay
```

Ejemplo:

```text
PV Name               /dev/sdb

VG Name

PV Size               50.00 GiB

Allocatable           yes
```

---

# Inicializar un segundo disco

```bash
sudo pvcreate /dev/sdc
```

---

Consultar nuevamente:

```bash
pvs
```

Resultado esperado:

```text
PV

/dev/sdb

/dev/sdc
```

---

# Arquitectura actual

```text
/dev/sdb

↓

PV

/dev/sdc

↓

PV
```

---

# Paso 2: Crear un Volume Group

Crear:

```text
VGDATA
```

utilizando ambos discos.

```bash
sudo vgcreate VGDATA /dev/sdb /dev/sdc
```

Resultado:

```text
Volume group "VGDATA" successfully created
```

---

# Consultar Volume Groups

```bash
vgs
```

---

```bash
vgdisplay
```

Ejemplo:

```text
VG Name

VGDATA

VG Size

129.99 GiB
```

---

# Arquitectura actual

```text
/dev/sdb

       \

        \

         VGDATA

        /

       /

/dev/sdc
```

---

# Paso 3: Crear un Logical Volume

Crear un volumen de:

```text
30G
```

```bash
sudo lvcreate \
-L 30G \
-n LV_APP \
VGDATA
```

---

Consultar:

```bash
lvs
```

Resultado:

```text
LV_APP
```

---

# Crear segundo volumen

```bash
sudo lvcreate \
-L 40G \
-n LV_DB \
VGDATA
```

---

Consultar:

```bash
lvs
```

Resultado:

```text
LV_APP

LV_DB
```

---

# Crear tercer volumen

Utilizando el espacio restante.

```bash
sudo lvcreate \
-l 100%FREE \
-n LV_BACKUP \
VGDATA
```

---

Consultar nuevamente.

```bash
lvs
```

---

# Arquitectura

```text
VGDATA

├── LV_APP

├── LV_DB

└── LV_BACKUP
```

---

# Consultar información completa

```bash
lvdisplay
```

---

# Crear sistemas de archivos

Formatear cada volumen con XFS.

```bash
sudo mkfs.xfs /dev/VGDATA/LV_APP
```

---

```bash
sudo mkfs.xfs /dev/VGDATA/LV_DB
```

---

```bash
sudo mkfs.xfs /dev/VGDATA/LV_BACKUP
```

---

# Crear puntos de montaje

```bash
sudo mkdir /appdata
```

---

```bash
sudo mkdir /database
```

---

```bash
sudo mkdir /backups
```

---

# Montar manualmente

```bash
sudo mount /dev/VGDATA/LV_APP /appdata
```

---

```bash
sudo mount /dev/VGDATA/LV_DB /database
```

---

```bash
sudo mount /dev/VGDATA/LV_BACKUP /backups
```

---

# Verificar montaje

```bash
df -h
```

Resultado:

```text
/appdata

/database

/backups
```

---

# Verificar tipo de sistema

```bash
mount | grep VGDATA
```

---

# Obtener UUID

```bash
blkid
```

---

# Configurar montaje permanente

Editar:

```text
/etc/fstab
```

Agregar:

```text
UUID=xxxxxxxx /appdata xfs defaults 0 0

UUID=xxxxxxxx /database xfs defaults 0 0

UUID=xxxxxxxx /backups xfs defaults 0 0
```

---

# Validar fstab

```bash
sudo mount -a
```

Si no muestra errores:

```text
fstab válido
```

---

# Verificar reinicio

```bash
reboot
```

Luego:

```bash
df -h
```

Los tres volúmenes deben aparecer montados.

---

# Consultar espacio

```bash
df -h
```

---

```bash
vgs
```

---

```bash
lvs
```

---

```bash
pvs
```

---

# Buenas prácticas

- Utilizar nombres descriptivos.
- Separar aplicaciones por Logical Volumes.
- Utilizar XFS para servidores Red Hat.
- Documentar todos los VG.
- Mantener espacio libre en el VG.
- Utilizar UUID en `/etc/fstab`.
- Verificar siempre con `mount -a`.

---

# Errores comunes

## Error 1

Intentar crear un VG sobre un disco sin inicializar.

```text
Debe ejecutarse pvcreate primero.
```

---

## Error 2

Olvidar crear el sistema de archivos.

El volumen existe pero no puede montarse.

---

## Error 3

Agregar una entrada incorrecta en `/etc/fstab`.

Resultado:

El servidor puede no iniciar correctamente.

---

## Error 4

Montar utilizando nombres incorrectos.

Verificar siempre:

```bash
ls /dev/VGDATA/
```

---

## Error 5

Consumir el 100% del Volume Group.

Es recomendable dejar espacio libre para futuras ampliaciones.

---

# Laboratorios RHCSA

## Laboratorio 1

Crear un PV utilizando `/dev/sdb`.

---

## Laboratorio 2

Crear un VG llamado:

```text
VGDATA
```

---

## Laboratorio 3

Crear un LV llamado:

```text
LV_APP
```

de 20 GB.

---

## Laboratorio 4

Crear un segundo LV llamado:

```text
LV_LOGS
```

---

## Laboratorio 5

Formatear ambos con XFS.

---

## Laboratorio 6

Montarlos en:

```text
/application

/logs
```

---

## Laboratorio 7

Configurar `/etc/fstab`.

---

## Laboratorio 8

Reiniciar el servidor y verificar.

---

## Laboratorio 9

Mostrar toda la información de:

- PV
- VG
- LV

---

## Laboratorio 10

Documentar toda la infraestructura creada.

---

# Preguntas de repaso

1. ¿Qué significa PV?
2. ¿Qué significa VG?
3. ¿Qué significa LV?
4. ¿Cuál es el orden correcto para crear LVM?
5. ¿Qué comando crea un Physical Volume?
6. ¿Qué comando crea un Volume Group?
7. ¿Qué comando crea un Logical Volume?
8. ¿Por qué utilizar UUID en `/etc/fstab`?
9. ¿Cómo se verifica un archivo `/etc/fstab`?
10. ¿Qué comando muestra todos los Logical Volumes?

---

# Resumen

En esta primera fase aprendimos a construir completamente una infraestructura LVM desde cero.

Se implementó:

- Physical Volumes.
- Volume Groups.
- Logical Volumes.
- Sistemas de archivos XFS.
- Puntos de montaje.
- Configuración permanente mediante `/etc/fstab`.
- Validaciones.
- Buenas prácticas.
- Laboratorios similares al examen RHCSA.

---

# Próxima fase

## Fase 2 — Administración avanzada de LVM

En la siguiente fase aprenderás:

- Extender Logical Volumes.
- Reduccir Logical Volumes (ext4).
- Agregar nuevos discos al VG.
- Extender Volume Groups.
- Snapshots.
- Migración de datos.
- Reemplazo de discos.
- Eliminación segura de LVM.
- Troubleshooting avanzado.
- Laboratorio empresarial completo.