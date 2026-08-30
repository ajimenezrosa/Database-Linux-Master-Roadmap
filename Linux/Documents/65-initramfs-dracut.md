# **65. Initramfs y Dracut**

Siguiendo el mismo nivel de profundidad de las páginas anteriores (59-64), este será uno de los capítulos más técnicos del manual RHCSA, ya que explica todo lo que ocurre **entre GRUB2 y systemd**.

## Contenido propuesto

### 65-initramfs-dracut.md

### 1. Objetivos

* Comprender qué es initramfs.
* Diferenciar initramfs de initrd.
* Entender el papel de dracut.
* Diagnosticar fallos del initramfs.
* Regenerar imágenes initramfs.
* Personalizar módulos.
* Recuperar sistemas cuando initramfs falla.

---

## 2. Introducción

* ¿Qué ocurre después de GRUB?
* ¿Por qué el kernel necesita initramfs?
* Problema "el kernel no conoce el hardware"
* Necesidad de un sistema de archivos temporal.

---

## 3. Arquitectura completa del arranque

Diagrama completo:

```text
BIOS / UEFI
      │
      ▼
GRUB2
      │
      ▼
Kernel Linux
      │
      ▼
initramfs
      │
      ▼
dracut init
      │
      ▼
Carga de módulos
      │
      ▼
Detección de discos
      │
      ▼
LVM
      │
      ▼
RAID
      │
      ▼
LUKS
      │
      ▼
Montaje de /
      │
      ▼
switch_root
      │
      ▼
systemd
```

---

## 4. ¿Qué es initramfs?

* definición
* memoria RAM
* root temporal
* cpio archive
* compresión
* contenido interno

---

## 5. initrd vs initramfs

Comparación histórica completa.

---

## 6. ¿Qué contiene initramfs?

Explicación de:

```text
/bin
/sbin
/etc
/usr
/init
/lib
/lib64
/proc
/sys
/dev
/run
```

---

## 7. El archivo init

Cómo funciona.

---

## 8. dracut

* historia
* filosofía
* funcionamiento
* módulos
* detección automática

---

## 9. Arquitectura de módulos dracut

```
90lvm
90crypt
90kernel-modules
95udev
98systemd
```

---

## 10. Directorios importantes

```
/usr/lib/dracut/modules.d
/etc/dracut.conf
/etc/dracut.conf.d/
/boot/initramfs*
```

---

## 11. Archivo dracut.conf

Explicación completa.

---

## 12. dracut.conf.d

Configuraciones personalizadas.

---

## 13. Crear initramfs

```
dracut
```

```
dracut -f
```

---

## 14. Regenerar para un kernel específico

```
dracut -f initramfs...
```

---

## 15. dracut con verbose

```
dracut -v
```

```
dracut -f -v
```

---

## 16. Forzar módulos

```
add_drivers
```

```
force_drivers
```

---

## 17. Omitir módulos

```
omit_dracutmodules
```

---

## 18. Listar módulos disponibles

```
dracut --list-modules
```

---

## 19. Ver contenido de initramfs

```
lsinitrd
```

Explicación completa.

---

## 20. Buscar archivos dentro del initramfs

```
lsinitrd | grep
```

---

## 21. Extraer initramfs

```
cpio
```

```
gzip
```

---

## 22. Personalizar initramfs

Agregar scripts.

---

## 23. Hooks de dracut

```
pre-mount
pre-pivot
cleanup
initqueue
```

---

## 24. switch_root

Explicación profunda.

---

## 25. initqueue

Cómo funciona.

---

## 26. dracut shell

Cuando aparece:

```
dracut:/#
```

---

## 27. Recuperación desde dracut shell

```
lvm
```

```
blkid
```

```
lsblk
```

```
mount
```

```
switch_root
```

---

## 28. Errores comunes

```
dracut-initqueue timeout
```

```
Cannot find root device
```

```
Cannot mount root
```

```
Warning: /dev/mapper not found
```

---

## 29. LVM durante initramfs

Activación manual.

---

## 30. LUKS

Desbloqueo durante initramfs.

---

## 31. RAID

Reconocimiento.

---

## 32. Drivers

Cómo se cargan.

---

## 33. Kernel modules

```
modprobe
```

```
lsmod
```

---

## 34. journalctl relacionado

Logs del initramfs.

---

## 35. Parámetros del kernel

```
rd.break
```

```
rd.shell
```

```
rd.debug
```

```
rd.lvm.lv
```

```
rd.neednet
```

```
rd.driver.blacklist
```

---

## 36. Diagnóstico completo

Flujo tipo RHCSA.

---

## 37. Diagramas ASCII

Más de 10 diagramas.

---

## 38. Buenas prácticas

---

## 39. Errores frecuentes

---

## 40. Script de auditoría

Revisión automática de:

* initramfs
* dracut
* módulos
* kernel
* imágenes
* espacio en `/boot`
* versiones
* módulos faltantes

---

## 41. Laboratorio RHCSA

Alrededor de **25 prácticas** que incluyan:

* regenerar initramfs
* inspeccionar imágenes
* simular errores
* entrar a dracut shell
* recuperar root
* analizar módulos
* recuperar LVM
* validar imágenes

---

## 42. Preguntas de repaso

Aproximadamente **50 preguntas**.

---

## 43. Desafío final RHCSA

Escenario completo donde el estudiante deba recuperar un servidor cuyo **initramfs no contiene el controlador necesario para acceder al volumen raíz**, regenerar la imagen con `dracut`, verificar el contenido con `lsinitrd` y restaurar el arranque normal.

---

Este capítulo será probablemente el **más largo y técnico de todo el Módulo 9**, por lo que, al igual que las páginas 64 y anteriores, convendrá dividirlo en **4 fases** para mantener el mismo nivel de detalle y calidad:

* **Fase 1:** Fundamentos de initramfs, initrd y dracut.
* **Fase 2:** Arquitectura interna, módulos, configuración y personalización.
* **Fase 3:** Diagnóstico, recuperación desde `dracut` shell y casos reales.
* **Fase 4:** Script de auditoría, laboratorio RHCSA, preguntas de repaso y desafío final.
