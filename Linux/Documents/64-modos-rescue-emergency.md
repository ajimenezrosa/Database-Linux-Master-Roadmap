# 64. Modos Rescue y Emergency — Fase 1

> **Módulo 9: Arranque y Recuperación**  
> **Página 64 del Manual RHCSA**  
> **Fase 1: Fundamentos, arquitectura y diferencias**

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Comprender qué son los modos `rescue` y `emergency`.
- Identificar su función dentro del proceso de arranque.
- Diferenciar ambos modos de recuperación.
- Relacionarlos con los antiguos runlevels.
- Comprender qué servicios y sistemas de archivos están disponibles en cada modo.
- Identificar los targets de `systemd` relacionados.
- Consultar las dependencias de cada target.
- Reconocer cuándo utilizar `rescue.target`.
- Reconocer cuándo utilizar `emergency.target`.
- Diferenciar estos modos de `rd.break`.
- Comprender los riesgos de cambiar de modo desde una sesión remota.
- Prepararse para realizar tareas de recuperación RHCSA.

---

# Introducción

Durante el arranque de un sistema Linux pueden aparecer problemas que impidan alcanzar el modo operativo normal.

Algunos ejemplos son:

- Errores en `/etc/fstab`.
- Sistemas de archivos que no pueden montarse.
- Servicios críticos que fallan.
- Configuraciones incorrectas.
- Problemas de almacenamiento.
- Permisos dañados.
- Errores en unidades de `systemd`.
- Fallos de red.
- Contraseñas olvidadas.
- Configuraciones de seguridad incorrectas.

Para atender estos problemas, `systemd` ofrece modos especiales de recuperación.

Los dos principales son:

```text
rescue.target
```

y:

```text
emergency.target
```

Ambos proporcionan acceso administrativo con una cantidad reducida de servicios activos.

Sin embargo, no son equivalentes.

---

# Visión general

```text
Arranque normal
      │
      ▼
systemd
      │
      ├── graphical.target
      │
      ├── multi-user.target
      │
      ├── rescue.target
      │
      └── emergency.target
```

Los targets representan estados operativos del sistema.

Los modos `rescue` y `emergency` permiten trabajar con el sistema cuando el arranque normal no es posible o no resulta conveniente.

---

# ¿Qué es el modo Rescue?

El modo Rescue proporciona un entorno de mantenimiento con servicios mínimos.

Normalmente incluye:

- El sistema raíz montado.
- Sistemas de archivos locales disponibles.
- Servicios básicos.
- Una shell administrativa.
- Acceso a herramientas del sistema.
- Una cantidad limitada de procesos.

No suele incluir:

- Interfaz gráfica.
- Sesiones multiusuario normales.
- Servicios de aplicaciones.
- Todos los servicios de red.
- Servicios no esenciales.

---

# Target asociado al modo Rescue

El modo Rescue está representado por:

```text
rescue.target
```

Consultar su estado:

```bash
systemctl status rescue.target
```

Mostrar su archivo de unidad:

```bash
systemctl cat rescue.target
```

Consultar sus dependencias:

```bash
systemctl list-dependencies rescue.target
```

---

# Concepto de Rescue

```text
Sistema iniciado parcialmente
          │
          ▼
Sistemas locales disponibles
          │
          ▼
Servicios básicos activos
          │
          ▼
Shell administrativa
          │
          ▼
Mantenimiento y corrección
```

---

# ¿Qué es el modo Emergency?

El modo Emergency proporciona un entorno todavía más reducido que Rescue.

Normalmente ofrece:

- Una shell administrativa mínima.
- El sistema raíz.
- Muy pocos servicios.
- Ningún entorno gráfico.
- Ninguna sesión multiusuario.
- Normalmente ninguna red.
- Montajes muy limitados.

En algunos casos, el sistema raíz puede estar montado en modo de solo lectura.

---

# Target asociado al modo Emergency

El modo Emergency está representado por:

```text
emergency.target
```

Consultar su estado:

```bash
systemctl status emergency.target
```

Mostrar su archivo de unidad:

```bash
systemctl cat emergency.target
```

Consultar sus dependencias:

```bash
systemctl list-dependencies emergency.target
```

---

# Concepto de Emergency

```text
Kernel iniciado
      │
      ▼
systemd mínimo
      │
      ▼
Sistema raíz básico
      │
      ▼
Shell administrativa
      │
      ▼
Reparación crítica
```

---

# Rescue frente a Emergency

| Característica | `rescue.target` | `emergency.target` |
|---|---|---|
| Nivel de recuperación | Mantenimiento general | Recuperación crítica |
| Servicios activos | Algunos servicios básicos | Muy pocos servicios |
| Sistemas locales | Normalmente montados | Pueden no estar montados |
| Sistema raíz | Generalmente disponible | Puede estar en solo lectura |
| Red | Normalmente no disponible | No disponible |
| Interfaz gráfica | No | No |
| Multiusuario | No | No |
| Shell administrativa | Sí | Sí |
| Uso principal | Reparaciones generales | Fallos graves de arranque |
| Dependencias | Más numerosas | Muy limitadas |
| Equivalencia histórica | Runlevel 1 | Sin equivalencia exacta |

---

# Diferencia esencial

La diferencia principal puede resumirse así:

```text
Rescue
   │
   └── Sistema mínimo, pero relativamente preparado

Emergency
   │
   └── Sistema extremadamente mínimo
```

En Rescue, el sistema ya ha completado más etapas del arranque.

En Emergency, el sistema se detiene antes y activa menos componentes.

---

# Cuándo utilizar Rescue

El modo Rescue resulta apropiado para:

- Deshabilitar un servicio problemático.
- Corregir configuraciones de servicios.
- Modificar permisos.
- Revisar archivos de configuración.
- Reparar una configuración de red.
- Cambiar el target predeterminado.
- Analizar unidades fallidas.
- Corregir errores que no afectan el montaje de `/`.
- Realizar mantenimiento con pocos servicios activos.
- Desinstalar software problemático.
- Restaurar archivos de configuración.
- Revisar registros del sistema.

---

# Ejemplos de uso de Rescue

```text
El servidor inicia, pero un servicio crítico falla
```

```text
La interfaz gráfica impide iniciar correctamente
```

```text
Una aplicación consume demasiados recursos
```

```text
Se necesita modificar una unidad de systemd
```

```text
Se requiere mantenimiento sin usuarios conectados
```

---

# Cuándo utilizar Emergency

El modo Emergency resulta más apropiado para:

- Corregir un `/etc/fstab` incorrecto.
- Reparar montajes que bloquean el arranque.
- Revisar el sistema raíz.
- Trabajar cuando Rescue no puede iniciarse.
- Corregir problemas críticos de almacenamiento.
- Montar manualmente sistemas de archivos.
- Reparar volúmenes LVM.
- Atender problemas con UUID.
- Recuperar un sistema con montajes esenciales dañados.
- Analizar fallos ocurridos muy temprano durante el arranque.

---

# Ejemplos de uso de Emergency

```text
El sistema entra automáticamente en emergency mode
```

```text
Un UUID incorrecto impide montar /var
```

```text
El sistema raíz está en solo lectura
```

```text
Un volumen lógico no fue activado
```

```text
Existe un error grave en /etc/fstab
```

---

# Relación con los runlevels

Antes de `systemd`, Linux utilizaba niveles de ejecución denominados:

```text
runlevels
```

El modo Rescue se relaciona con:

```text
runlevel 1
```

o:

```text
single-user mode
```

---

# Tabla de equivalencias

| Runlevel tradicional | Target de systemd | Función |
|---:|---|---|
| 0 | `poweroff.target` | Apagar |
| 1 | `rescue.target` | Modo monousuario |
| 2 | `multi-user.target` | Multiusuario |
| 3 | `multi-user.target` | Multiusuario sin GUI |
| 4 | `multi-user.target` | Personalizable |
| 5 | `graphical.target` | Multiusuario con GUI |
| 6 | `reboot.target` | Reiniciar |

Emergency no tiene una equivalencia directa con un runlevel clásico.

Es un estado más mínimo que el antiguo runlevel 1.

---

# Alias de runlevel

Consultar los alias:

```bash
ls -l /usr/lib/systemd/system/runlevel*.target
```

Puede aparecer:

```text
runlevel1.target -> rescue.target
```

Consultar específicamente:

```bash
readlink -f \
/usr/lib/systemd/system/runlevel1.target
```

---

# Rescue como modo monousuario

En Rescue, el sistema está diseñado para que exista una única sesión administrativa principal.

Por esa razón se relaciona con:

```text
single-user mode
```

Características:

- No se ofrecen sesiones normales a otros usuarios.
- No se inicia el entorno multiusuario completo.
- Se reducen las aplicaciones activas.
- Se facilita el mantenimiento exclusivo.

---

# Arquitectura de systemd durante el arranque

El proceso general puede representarse así:

```text
Firmware
   │
   ▼
GRUB2
   │
   ▼
Kernel
   │
   ▼
initramfs
   │
   ▼
systemd
   │
   ├── sysinit.target
   │
   ├── basic.target
   │
   ├── multi-user.target
   │
   └── graphical.target
```

Los modos de recuperación modifican el punto al que debe llegar `systemd`.

---

# Flujo hacia Rescue

```text
Kernel
   │
   ▼
initramfs
   │
   ▼
systemd
   │
   ▼
sysinit.target
   │
   ▼
basic.target
   │
   ▼
rescue.target
   │
   ▼
rescue.service
   │
   ▼
Shell administrativa
```

El flujo exacto puede variar ligeramente según la versión y la distribución.

---

# Flujo hacia Emergency

```text
Kernel
   │
   ▼
initramfs
   │
   ▼
systemd
   │
   ▼
emergency.target
   │
   ▼
emergency.service
   │
   ▼
Shell mínima
```

Emergency evita activar muchas de las dependencias normales.

---

# Unidades relacionadas

Entre las unidades que pueden intervenir se encuentran:

```text
rescue.target
rescue.service
emergency.target
emergency.service
basic.target
sysinit.target
local-fs.target
default.target
```

---

# Examinar `rescue.target`

```bash
systemctl cat rescue.target
```

Ejemplo conceptual:

```ini
[Unit]
Description=Rescue Mode
Documentation=man:systemd.special(7)
Requires=sysinit.target rescue.service
After=sysinit.target rescue.service
AllowIsolate=yes
```

La configuración exacta depende de la versión del sistema.

---

# Interpretar `rescue.target`

## `Description=`

Describe la función de la unidad.

```ini
Description=Rescue Mode
```

---

## `Requires=`

Indica dependencias obligatorias.

```ini
Requires=sysinit.target rescue.service
```

Si una dependencia crítica falla, el target puede no alcanzarse correctamente.

---

## `After=`

Define el orden.

```ini
After=sysinit.target rescue.service
```

El target debe activarse después de esas unidades.

---

## `AllowIsolate=yes`

Permite utilizar:

```bash
systemctl isolate rescue.target
```

---

# Examinar `emergency.target`

```bash
systemctl cat emergency.target
```

Ejemplo conceptual:

```ini
[Unit]
Description=Emergency Mode
Documentation=man:systemd.special(7)
Requires=emergency.service
After=emergency.service
AllowIsolate=yes
```

---

# Interpretar `emergency.target`

Emergency suele tener menos dependencias que Rescue.

Esto le permite iniciar incluso cuando:

- Los sistemas de archivos locales fallan.
- El target básico no puede alcanzarse.
- Existen errores críticos de montaje.
- Algunos servicios esenciales fallan.

---

# Comparación de dependencias

```bash
systemctl list-dependencies \
rescue.target
```

```bash
systemctl list-dependencies \
emergency.target
```

Resultado conceptual:

```text
rescue.target
├─rescue.service
├─sysinit.target
├─local-fs.target
├─swap.target
└─...

emergency.target
└─emergency.service
```

La salida exacta depende del sistema.

---

# Visualizar dependencias inversas

Para Rescue:

```bash
systemctl list-dependencies \
--reverse \
rescue.target
```

Para Emergency:

```bash
systemctl list-dependencies \
--reverse \
emergency.target
```

---

# Consultar propiedades

```bash
systemctl show \
rescue.target \
-p Id \
-p Description \
-p ActiveState \
-p SubState \
-p Requires \
-p Wants \
-p After \
-p Before \
-p AllowIsolate
```

Para Emergency:

```bash
systemctl show \
emergency.target \
-p Id \
-p Description \
-p ActiveState \
-p SubState \
-p Requires \
-p Wants \
-p After \
-p Before \
-p AllowIsolate
```

---

# Estado de aislamiento

Consultar si Rescue permite aislamiento:

```bash
systemctl show \
rescue.target \
-p AllowIsolate
```

Resultado:

```text
AllowIsolate=yes
```

Consultar Emergency:

```bash
systemctl show \
emergency.target \
-p AllowIsolate
```

---

# ¿Qué significa aislar un target?

Aislar un target significa:

- Activar las unidades necesarias para ese target.
- Detener unidades que no sean necesarias.
- Cambiar el estado operativo actual.
- Reducir el sistema al entorno solicitado.

Comando general:

```bash
systemctl isolate TARGET
```

Ejemplo:

```bash
sudo systemctl isolate rescue.target
```

---

# Inicio frente a aislamiento

Existe una diferencia entre:

```bash
systemctl start rescue.target
```

y:

```bash
systemctl isolate rescue.target
```

---

# `start`

```bash
systemctl start rescue.target
```

Intenta iniciar el target y sus dependencias.

No necesariamente detiene todas las unidades ajenas al target.

---

# `isolate`

```bash
systemctl isolate rescue.target
```

Cambia el sistema al estado de Rescue y detiene unidades que no son necesarias.

---

# Comparación

| Comando | Resultado |
|---|---|
| `systemctl start TARGET` | Inicia el target |
| `systemctl isolate TARGET` | Cambia completamente al estado del target |
| `systemctl status TARGET` | Consulta el estado |
| `systemctl stop TARGET` | Detiene el target |
| `systemctl show TARGET` | Muestra propiedades |

---

# Riesgos de `systemctl isolate`

Utilizar `isolate` puede detener:

- Sesiones gráficas.
- Servicios de aplicaciones.
- Servicios de red.
- SSH.
- Bases de datos.
- Procesos de usuarios.
- Servicios de monitoreo.
- Montajes no requeridos.

Por esta razón debe utilizarse con mucho cuidado.

---

# Advertencia para sesiones SSH

No se recomienda ejecutar:

```bash
sudo systemctl isolate rescue.target
```

desde una conexión SSH sin acceso alternativo.

La red o `sshd` pueden detenerse y la sesión puede cerrarse.

---

# Recomendación de consola

Antes de entrar en Rescue o Emergency, asegúrate de tener:

- Consola física.
- Consola del hipervisor.
- iDRAC.
- iLO.
- IPMI.
- KVM remoto.
- Consola serial.
- Acceso local a la máquina virtual.

---

# Servicios disponibles en Rescue

Los servicios disponibles pueden variar, pero normalmente Rescue intenta ofrecer:

- Shell administrativa.
- Montajes locales.
- Dispositivos esenciales.
- Servicios básicos del sistema.
- Journald.
- Udev.
- Componentes de inicialización.

No debe asumirse que la red estará disponible.

---

# Servicios disponibles en Emergency

Emergency activa la menor cantidad posible de componentes.

Puede incluir únicamente:

- `systemd`.
- Una shell.
- El sistema raíz.
- Dispositivos mínimos.
- Registro básico.

No debe esperarse:

- Red.
- SSH.
- Montajes secundarios.
- Servicios de aplicaciones.
- Entorno gráfico.
- Cron.
- Bases de datos.

---

# Sistemas de archivos en Rescue

En Rescue normalmente se intentan montar los sistemas de archivos locales definidos correctamente.

Consultar:

```bash
findmnt
```

También:

```bash
mount
```

Consultar sistemas locales:

```bash
systemctl status local-fs.target
```

---

# Sistemas de archivos en Emergency

En Emergency algunos sistemas de archivos pueden no estar montados.

Consultar:

```bash
findmnt
```

```bash
lsblk -f
```

```bash
mount
```

Puede ser necesario montar manualmente:

```bash
mount /dev/dispositivo /punto
```

---

# Sistema raíz en Rescue

Normalmente `/` está disponible para lectura y escritura.

Verificar:

```bash
findmnt /
```

Salida conceptual:

```text
TARGET SOURCE                FSTYPE OPTIONS
/      /dev/mapper/rhel-root xfs    rw,relatime,seclabel
```

---

# Sistema raíz en Emergency

Puede encontrarse en modo:

```text
ro
```

Verificar:

```bash
findmnt -o TARGET,SOURCE,FSTYPE,OPTIONS /
```

Si aparece:

```text
ro
```

debe remontarse antes de modificar archivos.

---

# Remontar la raíz

```bash
mount -o remount,rw /
```

Verificar:

```bash
findmnt /
```

La opción debe mostrar:

```text
rw
```

---

# Por qué la raíz puede estar en solo lectura

Esto puede ocurrir para:

- Evitar daños adicionales.
- Permitir diagnóstico seguro.
- Proteger un sistema de archivos con errores.
- Detener modificaciones mientras se investiga.
- Mantener un entorno mínimo.

---

# Red en Rescue

No debe asumirse que la red está activa.

Consultar:

```bash
ip address
```

```bash
nmcli device status
```

```bash
systemctl status NetworkManager
```

```bash
systemctl status network.target
```

---

# Red en Emergency

Normalmente no está disponible.

Puede no existir:

- Dirección IP.
- DNS.
- Ruta predeterminada.
- NetworkManager activo.
- Conectividad SSH.

---

# Consultar interfaces

```bash
ip link
```

```bash
ip address
```

Incluso si una interfaz existe, eso no significa que tenga conectividad configurada.

---

# Autenticación en Rescue y Emergency

Dependiendo de la versión y de la configuración, el sistema puede solicitar la contraseña de `root`.

Puede aparecer un mensaje similar a:

```text
Give root password for maintenance
```

o:

```text
Press Enter for maintenance
```

El comportamiento puede variar según:

- Versión de RHEL.
- Configuración de `sulogin`.
- Estado de la cuenta `root`.
- Método utilizado para entrar.
- Políticas de seguridad.

---

# Servicio `rescue.service`

Consultar:

```bash
systemctl cat rescue.service
```

Puede utilizar:

```text
systemd-sulogin-shell
```

o:

```text
sulogin
```

para proporcionar la shell administrativa.

---

# Servicio `emergency.service`

Consultar:

```bash
systemctl cat emergency.service
```

También puede invocar una shell de mantenimiento mediante `sulogin`.

---

# ¿Qué es `sulogin`?

`sulogin` permite iniciar una shell administrativa durante mantenimiento.

Consultar manual:

```bash
man sulogin
```

Características:

- Solicita autenticación de `root`.
- Proporciona una shell administrativa.
- Se utiliza en situaciones de mantenimiento.
- Reduce el acceso no autorizado.

---

# Rescue, Emergency y `rd.break`

Estos tres métodos no son equivalentes.

| Característica | Rescue | Emergency | `rd.break` |
|---|---|---|---|
| Administrado por | `systemd` | `systemd` | `dracut` |
| Etapa | Arranque de systemd | Arranque mínimo de systemd | Dentro de initramfs |
| Raíz real | `/` | `/` | `/sysroot` |
| `chroot` necesario | No normalmente | No normalmente | Sí normalmente |
| Servicios | Algunos | Muy pocos | Prácticamente ninguno |
| Red | Normalmente no | No | No |
| Uso | Mantenimiento general | Fallos críticos | Recuperación temprana |
| Cambio de contraseña | Posible con autenticación | Posible con autenticación | Método habitual de recuperación |

---

# Flujo comparativo

```text
GRUB2
  │
  ▼
Kernel
  │
  ▼
initramfs
  │
  ├── rd.break
  │      └── Shell con raíz real en /sysroot
  │
  ▼
systemd
  │
  ├── emergency.target
  │      └── Shell mínima
  │
  ├── rescue.target
  │      └── Sistema básico de mantenimiento
  │
  ├── multi-user.target
  │      └── Servidor operativo
  │
  └── graphical.target
         └── Sistema gráfico
```

---

# ¿Cuándo usar `rd.break`?

Utiliza `rd.break` principalmente cuando:

- Necesitas recuperar la contraseña de `root`.
- El sistema no llega a iniciar `systemd`.
- Debes modificar archivos antes de montar completamente la raíz.
- Rescue y Emergency no son suficientes.
- Necesitas intervenir desde `initramfs`.
- Debes trabajar con `/sysroot`.

---

# ¿Cuándo usar Emergency?

Utiliza Emergency cuando:

- `systemd` inicia, pero el sistema no puede completar montajes.
- Existe un error grave en `/etc/fstab`.
- Se necesita una shell mínima.
- El sistema raíz requiere revisión.
- Se desea evitar iniciar la mayoría de las dependencias.
- Rescue no puede alcanzarse correctamente.

---

# ¿Cuándo usar Rescue?

Utiliza Rescue cuando:

- El sistema puede montar sus sistemas locales.
- Se necesita acceder a herramientas normales.
- Deben corregirse servicios.
- Se requiere un entorno de mantenimiento más completo.
- Se desea detener aplicaciones y sesiones de usuarios.
- El problema no ocurre en una etapa extremadamente temprana.

---

# Selección del modo adecuado

```text
¿El sistema inicia normalmente?
        │
        ├── Sí
        │    └── Puede usarse systemctl rescue
        │
        └── No
             │
             ▼
¿systemd alcanza una shell mínima?
        │
        ├── Sí
        │    ├── Rescue para mantenimiento general
        │    └── Emergency para fallos críticos
        │
        └── No
             │
             └── rd.break o medio de rescate
```

---

# Ejemplo 1: servicio gráfico defectuoso

Problema:

```text
El sistema queda con pantalla negra después de iniciar GDM
```

Modo recomendado:

```text
rescue.target
```

o:

```text
multi-user.target
```

Razón:

- El almacenamiento funciona.
- El problema ocurre en un servicio superior.
- Se necesitan herramientas normales del sistema.

---

# Ejemplo 2: UUID incorrecto en `/etc/fstab`

Problema:

```text
El sistema no puede montar /datos
```

Modo recomendado:

```text
emergency.target
```

Razón:

- El fallo está relacionado con montajes.
- Puede ser necesario corregir `/etc/fstab`.
- El sistema debe iniciar con dependencias mínimas.

---

# Ejemplo 3: contraseña root olvidada

Problema:

```text
No existe otro usuario con sudo
```

Método recomendado:

```text
rd.break
```

Razón:

- Permite modificar la contraseña antes del arranque completo.
- No depende de autenticación normal de `root`.

---

# Ejemplo 4: servicio de base de datos bloquea el servidor

Problema:

```text
La base de datos consume todos los recursos durante el inicio
```

Modo recomendado:

```text
rescue.target
```

Razón:

- Permite deshabilitar el servicio.
- El sistema de archivos funciona.
- Se dispone de herramientas administrativas.

---

# Ejemplo 5: sistema raíz en solo lectura

Problema:

```text
No pueden modificarse archivos de configuración
```

Modo recomendado:

```text
emergency.target
```

Acción inicial:

```bash
findmnt /
```

Después:

```bash
mount -o remount,rw /
```

Siempre debe investigarse por qué el sistema raíz quedó en solo lectura.

---

# Posibles causas de entrada automática en Emergency

El sistema puede entrar automáticamente en Emergency debido a:

- UUID incorrecto.
- Sistema de archivos ausente.
- Disco desconectado.
- Error en `/etc/fstab`.
- Montaje requerido que excede el tiempo de espera.
- Volumen LVM no activado.
- Dispositivo cifrado no disponible.
- Sistema de archivos dañado.
- Dependencia crítica fallida.
- Unidad `.mount` incorrecta.

---

# Mensaje típico de Emergency

Puede aparecer:

```text
You are in emergency mode.
After logging in, type "journalctl -xb" to view system logs,
"systemctl reboot" to reboot,
or "exit" to continue bootup.
```

Este mensaje recomienda revisar:

```bash
journalctl -xb
```

---

# Primeros comandos en Emergency

```bash
journalctl -xb
```

```bash
systemctl --failed
```

```bash
findmnt
```

```bash
lsblk -f
```

```bash
cat /etc/fstab
```

```bash
mount -av
```

---

# Primeros comandos en Rescue

```bash
systemctl --failed
```

```bash
journalctl -b -p err
```

```bash
systemctl list-units \
--state=failed
```

```bash
findmnt
```

```bash
df -h
```

---

# Comando `journalctl -xb`

```bash
journalctl -xb
```

Opciones:

```text
-x
```

agrega explicaciones cuando están disponibles.

```text
-b
```

muestra registros del arranque actual.

---

# Consultar errores

```bash
journalctl -b -p err
```

Mostrar advertencias y errores:

```bash
journalctl -b -p warning
```

---

# Unidades fallidas

```bash
systemctl --failed
```

Salida conceptual:

```text
UNIT                    LOAD   ACTIVE SUB    DESCRIPTION
mnt-datos.mount         loaded failed failed /mnt/datos
```

Esto puede indicar directamente qué montaje causó el problema.

---

# Estado general del sistema

```bash
systemctl is-system-running
```

Resultados posibles:

```text
running
degraded
maintenance
starting
stopping
offline
unknown
```

En Rescue o Emergency puede aparecer:

```text
maintenance
```

---

# Consultar el target activo

```bash
systemctl list-units \
--type=target \
--state=active
```

En Rescue debería aparecer:

```text
rescue.target
```

En Emergency:

```text
emergency.target
```

---

# Verificar un target

```bash
systemctl is-active rescue.target
```

```bash
systemctl is-active emergency.target
```

---

# Consultar el target predeterminado

```bash
systemctl get-default
```

Estar en Rescue temporalmente no significa que el target predeterminado haya cambiado.

---

# Target activo frente a target predeterminado

```text
Target predeterminado
        │
        └── Se usa en arranques normales

Target activo
        │
        └── Estado actual del sistema
```

Ejemplo:

```text
Predeterminado: graphical.target
Activo: rescue.target
```

Esto puede ocurrir durante mantenimiento.

---

# No establecer Rescue como target normal

Aunque técnicamente se podría ejecutar:

```bash
systemctl set-default rescue.target
```

no debe utilizarse como configuración normal.

El sistema iniciaría siempre en modo de mantenimiento.

---

# No establecer Emergency como target normal

Tampoco debe utilizarse:

```bash
systemctl set-default emergency.target
```

como configuración permanente normal.

Puede dejar el sistema iniciando siempre en una shell mínima.

---

# Restaurar un target normal

Para servidores:

```bash
sudo systemctl set-default \
multi-user.target
```

Para estaciones gráficas:

```bash
sudo systemctl set-default \
graphical.target
```

Verificar:

```bash
systemctl get-default
```

---

# Salir del modo de mantenimiento

En algunos casos se puede escribir:

```bash
exit
```

Esto solicita continuar el arranque.

También:

```bash
systemctl default
```

---

# Comando `systemctl default`

```bash
systemctl default
```

Intenta alcanzar:

```text
default.target
```

Es equivalente conceptualmente a intentar continuar hacia el target predeterminado.

---

# Aislar el target predeterminado

```bash
systemctl isolate default.target
```

También puede utilizarse, aunque:

```bash
systemctl default
```

es más claro.

---

# Reiniciar desde recuperación

```bash
systemctl reboot
```

Apagar:

```bash
systemctl poweroff
```

---

# No utilizar reinicio forzado sin necesidad

Evita:

```bash
reboot -f
```

salvo que el sistema no permita un reinicio normal.

Un reinicio forzado puede:

- Omitir sincronización.
- No desmontar sistemas correctamente.
- Aumentar el riesgo de corrupción.
- Perder cambios pendientes.

---

# Sincronizar cambios

Antes de reiniciar:

```bash
sync
```

Especialmente después de:

- Editar `/etc/fstab`.
- Cambiar archivos de configuración.
- Reparar permisos.
- Modificar unidades.
- Cambiar contraseñas.

---

# Buenas prácticas de esta fase

✔ Comprender la diferencia entre Rescue y Emergency.

✔ Utilizar Rescue para mantenimiento general.

✔ Utilizar Emergency para fallos críticos.

✔ Utilizar `rd.break` para recuperación temprana.

✔ Mantener acceso a consola.

✔ No aislar targets de recuperación únicamente desde SSH.

✔ Consultar las dependencias antes de cambiar de modo.

✔ Verificar si `/` está en lectura o escritura.

✔ Revisar `journalctl -xb`.

✔ Consultar unidades fallidas.

✔ No establecer targets de recuperación como predeterminados.

✔ Ejecutar `sync` antes de reiniciar.

✔ Documentar el motivo de la recuperación.

---

# Errores comunes

## Pensar que Rescue y Emergency son iguales

Emergency activa menos unidades y puede dejar sistemas sin montar.

---

## Confundir Emergency con `rd.break`

Emergency es administrado por `systemd`.

`rd.break` se ejecuta dentro de `initramfs`.

---

## Ejecutar `isolate` desde SSH

La red puede detenerse y perderse la sesión.

---

## Suponer que la red estará disponible

Ni Rescue ni Emergency garantizan conectividad.

---

## Modificar archivos con `/` en solo lectura

Los cambios no podrán guardarse.

---

## Reiniciar sin revisar los registros

Puede repetirse exactamente el mismo error.

---

## Establecer Rescue como predeterminado

El sistema iniciará siempre en mantenimiento.

---

## Utilizar Emergency cuando Rescue es suficiente

Emergency dispone de menos herramientas y servicios.

---

## Utilizar Rescue cuando el sistema raíz no puede montarse

En ese caso puede ser necesario Emergency, `rd.break` o un medio de rescate.

---

# Comandos importantes de la Fase 1

| Comando | Descripción |
|---|---|
| `systemctl status rescue.target` | Consultar Rescue |
| `systemctl status emergency.target` | Consultar Emergency |
| `systemctl cat rescue.target` | Mostrar definición de Rescue |
| `systemctl cat emergency.target` | Mostrar definición de Emergency |
| `systemctl list-dependencies rescue.target` | Mostrar dependencias |
| `systemctl list-dependencies emergency.target` | Mostrar dependencias |
| `systemctl show TARGET` | Mostrar propiedades |
| `systemctl isolate TARGET` | Cambiar al target |
| `systemctl start TARGET` | Iniciar sin aislar |
| `systemctl get-default` | Consultar target normal |
| `systemctl default` | Continuar hacia target predeterminado |
| `systemctl --failed` | Mostrar unidades fallidas |
| `systemctl is-system-running` | Mostrar estado general |
| `journalctl -xb` | Revisar arranque actual |
| `findmnt` | Consultar montajes |
| `findmnt /` | Consultar sistema raíz |
| `mount -o remount,rw /` | Remontar raíz en escritura |
| `lsblk -f` | Consultar discos y UUID |
| `sync` | Escribir cambios pendientes |
| `systemctl reboot` | Reiniciar normalmente |

---

# Resumen rápido

```text
Modos de recuperación
        │
        ├── rescue.target
        │     ├── Servicios básicos
        │     ├── Sistemas locales
        │     ├── Shell administrativa
        │     └── Mantenimiento general
        │
        ├── emergency.target
        │     ├── Servicios mínimos
        │     ├── Posible raíz en ro
        │     ├── Sin red
        │     └── Reparación crítica
        │
        └── rd.break
              ├── Entorno initramfs
              ├── Raíz en /sysroot
              ├── chroot necesario
              └── Recuperación temprana
```

---

# Resumen de la Fase 1

En esta fase aprendiste a:

- Comprender los modos Rescue y Emergency.
- Relacionar Rescue con el antiguo runlevel 1.
- Identificar los targets correspondientes.
- Consultar sus dependencias y propiedades.
- Diferenciar `start` e `isolate`.
- Reconocer los riesgos de cambiar de modo remotamente.
- Identificar los servicios disponibles.
- Comprender el estado de los sistemas de archivos.
- Diferenciar Rescue, Emergency y `rd.break`.
- Seleccionar el método adecuado según el problema.
- Utilizar comandos iniciales de diagnóstico.
- Comprender cómo continuar o reiniciar el sistema.

---

# Laboratorio de fundamentos

> Este laboratorio no requiere modificar configuraciones críticas. Puede realizarse desde un sistema normal.

## Ejercicio 1: Consultar Rescue

```bash
systemctl status \
rescue.target
```

---

## Ejercicio 2: Consultar Emergency

```bash
systemctl status \
emergency.target
```

---

## Ejercicio 3: Mostrar definiciones

```bash
systemctl cat \
rescue.target
```

```bash
systemctl cat \
emergency.target
```

---

## Ejercicio 4: Comparar dependencias

```bash
systemctl list-dependencies \
rescue.target
```

```bash
systemctl list-dependencies \
emergency.target
```

Identifica cuál posee menos dependencias.

---

## Ejercicio 5: Consultar aislamiento

```bash
systemctl show \
rescue.target \
-p AllowIsolate
```

```bash
systemctl show \
emergency.target \
-p AllowIsolate
```

---

## Ejercicio 6: Consultar el target predeterminado

```bash
systemctl get-default
```

```bash
readlink -f \
/etc/systemd/system/default.target
```

---

## Ejercicio 7: Consultar targets activos

```bash
systemctl list-units \
--type=target \
--state=active
```

---

## Ejercicio 8: Consultar estado general

```bash
systemctl is-system-running
```

---

## Ejercicio 9: Consultar unidades fallidas

```bash
systemctl --failed
```

---

## Ejercicio 10: Verificar la raíz

```bash
findmnt -o \
TARGET,SOURCE,FSTYPE,OPTIONS /
```

Identifica si aparece:

```text
rw
```

o:

```text
ro
```

---

# Preguntas de repaso

1. ¿Qué es `rescue.target`?
2. ¿Qué es `emergency.target`?
3. ¿Cuál de los dos inicia más servicios?
4. ¿Cuál es equivalente aproximadamente al runlevel 1?
5. ¿Emergency tiene una equivalencia exacta con los runlevels?
6. ¿Qué comando muestra las dependencias de un target?
7. ¿Qué significa `AllowIsolate=yes`?
8. ¿Cuál es la diferencia entre `start` e `isolate`?
9. ¿Por qué es peligroso utilizar `isolate` mediante SSH?
10. ¿La red está garantizada en Rescue?
11. ¿La red está disponible normalmente en Emergency?
12. ¿Cómo se comprueba si `/` está en solo lectura?
13. ¿Cómo se remonta `/` en escritura?
14. ¿Qué diferencia existe entre Emergency y `rd.break`?
15. ¿Dónde se encuentra la raíz real con `rd.break`?
16. ¿Qué modo utilizarías para corregir un servicio defectuoso?
17. ¿Qué modo utilizarías para corregir un UUID incorrecto?
18. ¿Qué comando muestra los errores del arranque actual?
19. ¿Qué comando intenta continuar hacia el target predeterminado?
20. ¿Por qué no debe configurarse Rescue como target normal?

---

# Fin de la Fase 1

La siguiente fase cubrirá:

- Cómo iniciar Rescue desde GRUB2.
- Cómo iniciar Emergency desde GRUB2.
- Uso de `systemctl rescue`.
- Uso de `systemctl emergency`.
- Uso seguro de `systemctl isolate`.
- Continuación del arranque.
- Recuperación de `/etc/fstab`.
- Validación de montajes.
- Diagnóstico con `journalctl`.
- Casos prácticos de recuperación.