# 61. Targets y Modos de systemd

> **Módulo 9: Arranque y Recuperación**  
> **Página 61 del Manual RHCSA**

---

# Objetivos de aprendizaje

Al finalizar esta lección serás capaz de:

- Comprender qué es un target de `systemd`.
- Relacionar los targets con los antiguos runlevels.
- Identificar el target predeterminado.
- Cambiar temporalmente el estado operativo del sistema.
- Cambiar permanentemente el target de arranque.
- Diferenciar `multi-user.target` y `graphical.target`.
- Comprender los modos `rescue` y `emergency`.
- Consultar dependencias entre targets y servicios.
- Interpretar targets activos.
- Diagnosticar problemas relacionados con targets.
- Utilizar targets como parte de tareas de recuperación.
- Aplicar buenas prácticas similares a las evaluadas en RHCSA.

---

# Introducción

En sistemas modernos basados en Red Hat Enterprise Linux, `systemd` administra el arranque, los servicios, los montajes, los sockets, los dispositivos y el estado operativo general del sistema.

Uno de sus conceptos principales es el:

```text
target
```

Un target representa un estado operativo del sistema.

Por ejemplo, un sistema puede estar preparado para:

- Trabajar en modo multiusuario.
- Iniciar una interfaz gráfica.
- Entrar en modo de rescate.
- Entrar en modo de emergencia.
- Reiniciarse.
- Apagarse.

---

# ¿Qué es un target?

Un target es una unidad de `systemd` cuya función principal es agrupar otras unidades.

Puede incluir:

- Servicios.
- Montajes.
- Sockets.
- Dispositivos.
- Otros targets.
- Dependencias necesarias para alcanzar un estado.

Ejemplo conceptual:

```text
graphical.target
      │
      ├── multi-user.target
      │      ├── sshd.service
      │      ├── crond.service
      │      ├── NetworkManager.service
      │      └── firewalld.service
      │
      └── display-manager.service
```

---

# Extensión de un target

Los targets utilizan la extensión:

```text
.target
```

Ejemplos:

```text
multi-user.target
graphical.target
rescue.target
emergency.target
reboot.target
poweroff.target
```

---

# Función de los targets

Los targets permiten que `systemd` lleve el sistema a un estado concreto.

```text
Estado deseado
      │
      ▼
Target
      │
      ▼
Dependencias
      │
      ▼
Servicios, montajes y dispositivos
      │
      ▼
Sistema operativo disponible
```

---

# Targets principales

| Target | Función |
|---|---|
| `poweroff.target` | Apaga el sistema |
| `rescue.target` | Inicia un entorno mínimo de rescate |
| `multi-user.target` | Modo multiusuario sin entorno gráfico |
| `graphical.target` | Modo multiusuario con interfaz gráfica |
| `reboot.target` | Reinicia el sistema |
| `emergency.target` | Inicia una shell mínima de emergencia |
| `default.target` | Enlace al target de arranque predeterminado |

---

# Targets y antiguos runlevels

Antes de `systemd`, los sistemas Linux utilizaban runlevels de SysV init.

`systemd` mantiene equivalencias para facilitar la compatibilidad.

| Runlevel | Target equivalente | Descripción |
|---:|---|---|
| 0 | `poweroff.target` | Apagado |
| 1 | `rescue.target` | Modo monousuario |
| 2 | `multi-user.target` | Multiusuario |
| 3 | `multi-user.target` | Multiusuario sin interfaz gráfica |
| 4 | `multi-user.target` | Personalizable |
| 5 | `graphical.target` | Multiusuario con interfaz gráfica |
| 6 | `reboot.target` | Reinicio |

---

# Alias de runlevels

`systemd` puede incluir alias como:

```text
runlevel0.target
runlevel1.target
runlevel2.target
runlevel3.target
runlevel4.target
runlevel5.target
runlevel6.target
```

Consultar:

```bash
ls -l /usr/lib/systemd/system/runlevel*.target
```

Ejemplo:

```text
runlevel3.target -> multi-user.target
runlevel5.target -> graphical.target
```

---

# Diferencia conceptual entre runlevels y targets

Los runlevels eran estados numéricos y relativamente rígidos.

Los targets son más flexibles y pueden:

- Depender de otros targets.
- Iniciarse en paralelo.
- Agrupar múltiples tipos de unidades.
- Crear estructuras complejas de dependencias.
- Ser personalizados.

```text
Runlevel
   │
   └── Estado numérico

Target
   │
   ├── Dependencias
   ├── Paralelización
   ├── Servicios
   ├── Montajes
   └── Otros targets
```

---

# Consultar el target predeterminado

```bash
systemctl get-default
```

Ejemplo en un servidor:

```text
multi-user.target
```

Ejemplo en una estación gráfica:

```text
graphical.target
```

---

# ¿Qué es `default.target`?

`default.target` normalmente es un enlace simbólico al target que se utilizará durante el arranque.

Consultar:

```bash
ls -l /etc/systemd/system/default.target
```

Ejemplo:

```text
/etc/systemd/system/default.target \
-> /usr/lib/systemd/system/graphical.target
```

También:

```bash
readlink -f \
/etc/systemd/system/default.target
```

---

# Establecer el target predeterminado

Para iniciar en modo texto:

```bash
sudo systemctl set-default \
multi-user.target
```

Para iniciar con interfaz gráfica:

```bash
sudo systemctl set-default \
graphical.target
```

Verificar:

```bash
systemctl get-default
```

---

# Efecto de `set-default`

`systemctl set-default` modifica el estado de futuros arranques.

```text
systemctl set-default
          │
          ▼
Actualiza default.target
          │
          ▼
No cambia necesariamente la sesión actual
          │
          ▼
Se aplica en el próximo arranque
```

---

# Cambiar el estado actual con `isolate`

Para cambiar inmediatamente al modo multiusuario:

```bash
sudo systemctl isolate \
multi-user.target
```

Para cambiar inmediatamente al modo gráfico:

```bash
sudo systemctl isolate \
graphical.target
```

---

# Efecto de `isolate`

`isolate` detiene unidades que no son necesarias y activa las requeridas por el target solicitado.

```text
Estado actual
    │
    ▼
systemctl isolate TARGET
    │
    ├── Inicia dependencias necesarias
    └── Detiene unidades no requeridas
            │
            ▼
      Nuevo estado operativo
```

---

# Diferencia entre `set-default` e `isolate`

| Comando | Efecto |
|---|---|
| `systemctl set-default TARGET` | Cambia el target de futuros arranques |
| `systemctl isolate TARGET` | Cambia inmediatamente el estado actual |
| `systemctl start TARGET` | Inicia el target sin aislar completamente |
| `systemctl get-default` | Muestra el target predeterminado |

---

# Ejemplo práctico

Cambiar el próximo arranque a modo texto:

```bash
sudo systemctl set-default \
multi-user.target
```

Esto no necesariamente cierra la interfaz gráfica actual.

Para hacerlo inmediatamente:

```bash
sudo systemctl isolate \
multi-user.target
```

---

# Precaución con `isolate`

`systemctl isolate` puede detener:

- La interfaz gráfica.
- Sesiones de usuario.
- Servicios no requeridos.
- Componentes de red.
- La sesión SSH desde la cual se ejecutó.

Antes de utilizarlo:

- Asegura acceso a consola.
- Comprueba las dependencias.
- Evita ejecutarlo remotamente sin un plan de recuperación.
- Guarda el trabajo abierto.
- Verifica que el target admita aislamiento.

---

# Targets que permiten aislamiento

No todos los targets deben aislarse.

Consultar si un target permite aislamiento:

```bash
systemctl show \
multi-user.target \
-p AllowIsolate
```

Ejemplo:

```text
AllowIsolate=yes
```

Para `graphical.target`:

```bash
systemctl show \
graphical.target \
-p AllowIsolate
```

---

# Consultar targets activos

```bash
systemctl list-units \
--type=target
```

Solo activos:

```bash
systemctl list-units \
--type=target \
--state=active
```

---

# Consultar todos los targets disponibles

```bash
systemctl list-unit-files \
--type=target
```

Este comando muestra targets:

- Habilitados.
- Deshabilitados.
- Estáticos.
- Enmascarados.
- Alias.

---

# Consultar un target específico

```bash
systemctl status \
multi-user.target
```

También:

```bash
systemctl status \
graphical.target
```

---

# Ver el contenido de una unidad target

```bash
systemctl cat \
multi-user.target
```

Ejemplo conceptual:

```ini
[Unit]
Description=Multi-User System
Documentation=man:systemd.special(7)
Requires=basic.target
Conflicts=rescue.service rescue.target
After=basic.target rescue.service rescue.target
AllowIsolate=yes
```

---

# Ubicaciones de unidades

Las unidades pueden almacenarse en:

```text
/usr/lib/systemd/system/
```

```text
/etc/systemd/system/
```

```text
/run/systemd/system/
```

---

# Prioridad de configuración

En términos generales:

```text
/etc/systemd/system
        │
        ▼
/run/systemd/system
        │
        ▼
/usr/lib/systemd/system
```

Las configuraciones del administrador en `/etc` tienen prioridad sobre las proporcionadas por paquetes en `/usr/lib`.

---

# No modificar directamente `/usr/lib/systemd/system`

Los archivos en:

```text
/usr/lib/systemd/system/
```

son administrados por paquetes.

Los cambios pueden perderse durante una actualización.

Para personalizaciones se recomienda utilizar:

```bash
systemctl edit \
nombre.target
```

o crear unidades en:

```text
/etc/systemd/system/
```

---

# Dependencias de targets

Consultar las dependencias de `multi-user.target`:

```bash
systemctl list-dependencies \
multi-user.target
```

Para `graphical.target`:

```bash
systemctl list-dependencies \
graphical.target
```

---

# Mostrar dependencias inversas

```bash
systemctl list-dependencies \
--reverse \
multi-user.target
```

Esto muestra qué unidades dependen de ese target.

---

# Mostrar dependencias completas

```bash
systemctl list-dependencies \
--all \
graphical.target
```

---

# Tipos de relaciones

Las unidades de `systemd` pueden utilizar relaciones como:

```text
Requires=
Wants=
After=
Before=
Conflicts=
PartOf=
```

---

# `Requires=`

Indica una dependencia fuerte.

Si la unidad requerida no puede iniciarse, la unidad principal puede fallar.

Ejemplo conceptual:

```ini
Requires=basic.target
```

---

# `Wants=`

Indica una dependencia más débil.

La unidad principal puede continuar incluso si la unidad deseada falla.

Ejemplo:

```ini
Wants=network-online.target
```

---

# `After=`

Define orden de inicio, no una dependencia automática.

```ini
After=network.target
```

Significa que la unidad debe iniciarse después de `network.target`, pero no obliga a iniciar ese target por sí sola.

---

# `Before=`

Indica que una unidad debe iniciarse antes que otra.

```ini
Before=multi-user.target
```

---

# `Conflicts=`

Indica que dos unidades no deben estar activas simultáneamente.

Ejemplo:

```ini
Conflicts=rescue.target
```

---

# Dependencia frente a orden

Es importante distinguir:

```text
Requires= o Wants=
```

de:

```text
After= o Before=
```

Ejemplo:

```ini
Wants=network-online.target
After=network-online.target
```

- `Wants=` solicita la unidad.
- `After=` define el orden.

---

# `multi-user.target`

Representa un sistema multiusuario completamente funcional sin requerir interfaz gráfica.

Normalmente incluye:

- Red.
- Servicios de sistema.
- Consolas.
- SSH.
- Cron.
- Firewall.
- Servicios de servidor.
- Montajes necesarios.

---

# Casos de uso de `multi-user.target`

- Servidores sin entorno gráfico.
- Administración por consola.
- Reducción de consumo de recursos.
- Diagnóstico de problemas gráficos.
- Mantenimiento.
- Entornos de producción.

---

# Consultar dependencias

```bash
systemctl list-dependencies \
multi-user.target
```

---

# Cambiar permanentemente a modo texto

```bash
sudo systemctl set-default \
multi-user.target
```

Verificar:

```bash
systemctl get-default
```

---

# Cambiar inmediatamente a modo texto

```bash
sudo systemctl isolate \
multi-user.target
```

---

# `graphical.target`

Representa un sistema multiusuario con interfaz gráfica.

Normalmente depende de:

```text
multi-user.target
```

y añade un administrador de pantalla.

Ejemplo:

```text
graphical.target
       │
       ├── multi-user.target
       └── display-manager.service
```

---

# Consultar el display manager

```bash
systemctl status \
display-manager.service
```

En sistemas GNOME puede apuntar a:

```text
gdm.service
```

Consultar:

```bash
readlink -f \
/etc/systemd/system/display-manager.service
```

---

# Cambiar permanentemente a modo gráfico

```bash
sudo systemctl set-default \
graphical.target
```

---

# Cambiar inmediatamente

```bash
sudo systemctl isolate \
graphical.target
```

Esto solo funcionará correctamente si están instalados:

- Entorno gráfico.
- Display manager.
- Controladores requeridos.
- Paquetes relacionados.

---

# Verificar si existe entorno gráfico

```bash
rpm -q gdm
```

También:

```bash
systemctl status \
display-manager.service
```

---

# `rescue.target`

`rescue.target` proporciona un entorno de mantenimiento con servicios mínimos.

Es similar al antiguo:

```text
runlevel 1
```

Normalmente:

- Monta sistemas de archivos locales.
- Inicia servicios básicos.
- Proporciona una shell administrativa.
- No inicia el entorno multiusuario completo.
- No suele iniciar red completa.
- Requiere autenticación de `root` en muchos casos.

---

# Flujo de rescue target

```text
Kernel
   │
   ▼
systemd
   │
   ▼
rescue.target
   │
   ├── Sistemas de archivos locales
   ├── Servicios mínimos
   └── Shell de root
```

---

# Entrar en rescue target desde el sistema activo

```bash
sudo systemctl isolate \
rescue.target
```

También puede utilizarse:

```bash
sudo systemctl rescue
```

---

# Diferencia entre ambos comandos

```bash
systemctl isolate rescue.target
```

y:

```bash
systemctl rescue
```

llevan al sistema a un estado de rescate.

`systemctl rescue` es una forma especializada y puede solicitar confirmación según el entorno.

---

# Iniciar rescue target desde GRUB

Editar la entrada y agregar:

```text
systemd.unit=rescue.target
```

Después iniciar con:

```text
Ctrl+x
```

Este cambio es temporal.

---

# Casos de uso de rescue target

- Corregir configuraciones.
- Reparar permisos.
- Revisar servicios.
- Modificar `/etc/fstab`.
- Deshabilitar una unidad problemática.
- Recuperar configuraciones.
- Realizar mantenimiento con pocos servicios activos.

---

# `emergency.target`

`emergency.target` proporciona un entorno aún más mínimo que rescue.

Normalmente:

- Inicia una shell de emergencia.
- Activa muy pocos servicios.
- Puede mantener `/` en solo lectura.
- No monta automáticamente todos los sistemas de archivos.
- No inicia red.
- No activa el entorno multiusuario.

---

# Flujo de emergency target

```text
Kernel
   │
   ▼
systemd
   │
   ▼
emergency.target
   │
   └── Shell mínima
```

---

# Entrar en emergency target desde el sistema activo

```bash
sudo systemctl isolate \
emergency.target
```

También:

```bash
sudo systemctl emergency
```

---

# Iniciar emergency target desde GRUB

Agregar temporalmente:

```text
systemd.unit=emergency.target
```

---

# Sistema raíz en solo lectura

En modo de emergencia puede ser necesario remontar `/`:

```bash
mount -o remount,rw /
```

Verificar:

```bash
findmnt /
```

---

# Casos de uso de emergency target

- Errores graves de montaje.
- Daños en `/etc/fstab`.
- Problemas con el sistema raíz.
- Servicios básicos incapaces de iniciar.
- Recuperación cuando rescue no funciona.
- Diagnóstico con el mínimo de unidades activas.

---

# Rescue frente a Emergency

| Característica | `rescue.target` | `emergency.target` |
|---|---|---|
| Nivel de servicios | Mínimo | Extremadamente mínimo |
| Sistemas locales | Normalmente montados | Pueden no estar montados |
| Sistema raíz | Generalmente disponible | Puede estar en solo lectura |
| Red | Normalmente no | No |
| Shell administrativa | Sí | Sí |
| Uso | Mantenimiento general | Fallos graves |
| Dependencias | Más unidades | Muy pocas unidades |

---

# `basic.target`

`basic.target` representa una etapa básica del arranque.

Incluye componentes esenciales como:

- Sockets.
- Temporizadores.
- Rutas.
- Servicios básicos.
- Montajes fundamentales.

Normalmente sirve de base para targets más avanzados.

```text
sysinit.target
      │
      ▼
basic.target
      │
      ▼
multi-user.target
      │
      ▼
graphical.target
```

---

# `sysinit.target`

`sysinit.target` agrupa unidades necesarias durante la inicialización temprana.

Puede incluir:

- Montajes API.
- Dispositivos.
- Swap.
- Configuración de sistema.
- Servicios de inicialización.
- Reglas tempranas.

---

# `local-fs.target`

Representa la disponibilidad de sistemas de archivos locales.

Consultar:

```bash
systemctl status \
local-fs.target
```

Dependencias:

```bash
systemctl list-dependencies \
local-fs.target
```

---

# `remote-fs.target`

Representa la disponibilidad de sistemas de archivos remotos.

Ejemplos:

- NFS.
- CIFS.
- Montajes de red.

Consultar:

```bash
systemctl status \
remote-fs.target
```

---

# `network.target`

Indica que la pila de red básica fue inicializada.

No garantiza necesariamente que exista conectividad completa.

---

# `network-online.target`

Representa un estado en el que la red debería estar configurada y disponible.

Puede depender de un servicio de espera como:

```text
NetworkManager-wait-online.service
```

---

# Diferencia entre `network.target` y `network-online.target`

| Target | Significado |
|---|---|
| `network.target` | La infraestructura de red básica fue iniciada |
| `network-online.target` | La red se considera configurada y operativa |

Un servicio que necesita conectividad real puede requerir:

```ini
Wants=network-online.target
After=network-online.target
```

---

# `shutdown.target`

Agrupa unidades relacionadas con el proceso de apagado.

No suele utilizarse directamente para la administración diaria.

---

# `reboot.target`

Reinicia el sistema.

```bash
sudo systemctl isolate \
reboot.target
```

Sin embargo, se recomienda utilizar:

```bash
sudo systemctl reboot
```

---

# `poweroff.target`

Apaga el sistema.

Puede alcanzarse con:

```bash
sudo systemctl isolate \
poweroff.target
```

La forma recomendada es:

```bash
sudo systemctl poweroff
```

---

# `halt.target`

Detiene el sistema, pero su comportamiento puede variar según la plataforma.

La opción más clara para apagar es:

```bash
sudo systemctl poweroff
```

---

# `suspend.target`

Suspende el sistema en memoria.

```bash
sudo systemctl suspend
```

---

# `hibernate.target`

Guarda el contenido de memoria en disco y apaga el sistema, si está configurado.

```bash
sudo systemctl hibernate
```

---

# `hybrid-sleep.target`

Combina suspensión e hibernación.

```bash
sudo systemctl hybrid-sleep
```

---

# Targets activos durante el arranque

Consultar:

```bash
systemctl list-units \
--type=target \
--state=active
```

Salida conceptual:

```text
basic.target
cryptsetup.target
getty.target
local-fs.target
multi-user.target
network.target
paths.target
slices.target
sockets.target
swap.target
sysinit.target
timers.target
```

---

# Determinar si un target está activo

```bash
systemctl is-active \
multi-user.target
```

Resultado posible:

```text
active
```

---

# Determinar si está habilitado

Los targets no siempre se habilitan igual que un servicio.

Consultar:

```bash
systemctl is-enabled \
graphical.target
```

Puede devolver:

```text
static
```

Esto no implica necesariamente un problema.

---

# ¿Qué significa `static`?

Una unidad estática:

- No tiene una sección `[Install]` para habilitación directa.
- Se inicia como dependencia de otra unidad.
- No necesita un enlace creado mediante `enable`.

---

# Estados comunes de unit files

| Estado | Significado |
|---|---|
| `enabled` | Se inicia mediante enlaces configurados |
| `disabled` | No está habilitada |
| `static` | Se inicia como dependencia |
| `masked` | No puede iniciarse |
| `alias` | Es un alias de otra unidad |
| `indirect` | Se habilita mediante otra unidad |
| `generated` | Fue creada dinámicamente |

---

# Iniciar un target sin aislar

```bash
sudo systemctl start \
multi-user.target
```

Esto inicia sus dependencias, pero no necesariamente detiene unidades que no pertenecen al target.

Por eso no es igual a:

```bash
sudo systemctl isolate \
multi-user.target
```

---

# Detener un target

```bash
sudo systemctl stop \
nombre.target
```

Esto puede afectar múltiples unidades y debe utilizarse con cuidado.

---

# Enmascarar un target

```bash
sudo systemctl mask \
nombre.target
```

Esto impide que se inicie manualmente o como dependencia.

Es una operación peligrosa para targets críticos.

---

# Desenmascarar

```bash
sudo systemctl unmask \
nombre.target
```

---

# No enmascarar targets esenciales

Evita enmascarar:

```text
basic.target
sysinit.target
multi-user.target
graphical.target
local-fs.target
```

Un target crítico enmascarado puede impedir que el sistema complete el arranque.

---

# Consultar propiedades de un target

```bash
systemctl show \
multi-user.target
```

Propiedades específicas:

```bash
systemctl show \
multi-user.target \
-p Id \
-p Description \
-p ActiveState \
-p SubState \
-p LoadState \
-p AllowIsolate
```

---

# Consultar dependencias requeridas

```bash
systemctl show \
multi-user.target \
-p Requires
```

---

# Consultar dependencias deseadas

```bash
systemctl show \
multi-user.target \
-p Wants
```

---

# Consultar orden

```bash
systemctl show \
multi-user.target \
-p After \
-p Before
```

---

# Modificación temporal desde GRUB

Se puede indicar un target para un único arranque.

Ejemplos:

```text
systemd.unit=multi-user.target
```

```text
systemd.unit=rescue.target
```

```text
systemd.unit=emergency.target
```

---

# Procedimiento desde GRUB

1. Reinicia el sistema.
2. Muestra el menú de GRUB.
3. Selecciona una entrada.
4. Presiona:

```text
e
```

5. Localiza la línea del kernel.
6. Agrega:

```text
systemd.unit=TARGET
```

7. Inicia con:

```text
Ctrl+x
```

---

# Verificar el target usado

Después de iniciar:

```bash
systemctl list-units \
--type=target \
--state=active
```

También:

```bash
systemctl get-default
```

Recuerda que el target activo temporal puede ser diferente del predeterminado permanente.

---

# Target temporal frente a predeterminado

```text
systemd.unit=rescue.target
          │
          ▼
Solo este arranque
          │
          ▼
systemctl get-default
          │
          ▼
Conserva el valor permanente
```

---

# Cambiar el target por argumento persistente

Aunque es posible agregar:

```text
systemd.unit=multi-user.target
```

de forma permanente a la línea del kernel, no es la forma recomendada para establecer el modo normal.

Debe utilizarse:

```bash
sudo systemctl set-default \
multi-user.target
```

---

# Diagnóstico: el sistema inicia en modo incorrecto

Consulta:

```bash
systemctl get-default
```

Después:

```bash
readlink -f \
/etc/systemd/system/default.target
```

Verifica targets activos:

```bash
systemctl list-units \
--type=target \
--state=active
```

Consulta parámetros:

```bash
cat /proc/cmdline
```

Busca si existe:

```text
systemd.unit=
```

---

# Diagnóstico: no inicia la interfaz gráfica

Verifica el target:

```bash
systemctl get-default
```

Si muestra:

```text
multi-user.target
```

cámbialo:

```bash
sudo systemctl set-default \
graphical.target
```

Prueba temporalmente:

```bash
sudo systemctl isolate \
graphical.target
```

---

# Verificar display manager

```bash
systemctl status \
display-manager.service
```

También:

```bash
systemctl --failed
```

```bash
journalctl \
-u display-manager.service \
-b
```

---

# Diagnóstico: `graphical.target` activo sin interfaz

Esto puede ocurrir si:

- No existe display manager.
- El servicio gráfico falló.
- Faltan paquetes.
- Existe un problema de vídeo.
- El enlace `display-manager.service` es incorrecto.

Consultar:

```bash
systemctl list-dependencies \
graphical.target
```

```bash
readlink -f \
/etc/systemd/system/display-manager.service
```

---

# Diagnóstico: el sistema entra en rescue target

Revisa:

```bash
journalctl -b -p err
```

```bash
systemctl --failed
```

```bash
cat /etc/fstab
```

```bash
mount -av
```

Posibles causas:

- Montaje fallido.
- UUID incorrecto.
- Sistema de archivos dañado.
- Unidad crítica fallida.
- Configuración incorrecta.

---

# Diagnóstico: el sistema entra en emergency target

Consulta:

```bash
journalctl -xb
```

También:

```bash
systemctl --failed
```

Revisa el sistema raíz:

```bash
findmnt /
```

Si está en solo lectura:

```bash
mount -o remount,rw /
```

Después revisa:

```bash
cat /etc/fstab
```

```bash
lsblk -f
```

```bash
blkid
```

---

# `journalctl -xb`

La combinación:

```bash
journalctl -xb
```

muestra mensajes del arranque actual y suele utilizarse durante recuperación.

- `-x`: agrega explicaciones cuando están disponibles.
- `-b`: limita al arranque actual.

---

# Regresar desde rescue o emergency

Para intentar continuar al target predeterminado:

```bash
systemctl default
```

También puede utilizarse:

```bash
systemctl isolate \
default.target
```

---

# Salir de una shell de recuperación

En algunos casos puede utilizarse:

```bash
exit
```

Esto permite que `systemd` intente continuar el arranque.

---

# Recargar configuración de systemd

Después de modificar unidades:

```bash
sudo systemctl daemon-reload
```

Este comando:

- Vuelve a leer archivos de unidades.
- Regenera dependencias internas.
- No reinicia servicios automáticamente.

---

# `daemon-reexec`

```bash
sudo systemctl daemon-reexec
```

Reejecuta el proceso de `systemd`.

No suele ser necesario para cambios normales de unidades.

---

# Crear un target personalizado

Se puede crear un target para agrupar servicios propios.

Ejemplo:

```bash
sudo vi \
/etc/systemd/system/aplicaciones.target
```

Contenido:

```ini
[Unit]
Description=Servicios de aplicaciones corporativas
Requires=multi-user.target
After=multi-user.target
AllowIsolate=yes
```

---

# Recargar

```bash
sudo systemctl daemon-reload
```

Consultar:

```bash
systemctl status \
aplicaciones.target
```

---

# Asociar servicios mediante `Wants`

Crear el directorio:

```bash
sudo mkdir -p \
/etc/systemd/system/aplicaciones.target.wants
```

Crear un enlace:

```bash
sudo ln -s \
/usr/lib/systemd/system/httpd.service \
/etc/systemd/system/aplicaciones.target.wants/httpd.service
```

Sin embargo, es preferible utilizar:

```bash
sudo systemctl add-wants \
aplicaciones.target \
httpd.service
```

---

# `systemctl add-wants`

```bash
sudo systemctl add-wants \
aplicaciones.target \
httpd.service
```

Verificar:

```bash
systemctl list-dependencies \
aplicaciones.target
```

---

# Eliminar una relación

```bash
sudo systemctl revert \
aplicaciones.target
```

o eliminar cuidadosamente el enlace creado.

Para targets personalizados, revisa siempre:

```bash
find \
/etc/systemd/system \
-maxdepth 2 \
-type l \
-ls
```

---

# Establecer un target personalizado como predeterminado

Solo debe hacerse si el target fue probado correctamente.

```bash
sudo systemctl set-default \
aplicaciones.target
```

Antes de reiniciar:

```bash
systemctl status \
aplicaciones.target
```

```bash
systemctl list-dependencies \
aplicaciones.target
```

---

# Riesgo de un target personalizado incorrecto

Un error puede provocar:

- Servicios faltantes.
- Ausencia de consola.
- Red no disponible.
- Arranque incompleto.
- Dependencias circulares.
- Entrada en modo de recuperación.

Por ello, debe conservarse acceso a consola y un método de recuperación mediante GRUB.

---

# Flujo recomendado para cambiar el target

```text
Consultar target actual
        │
        ▼
Revisar dependencias
        │
        ▼
Probar con isolate
        │
        ▼
Validar servicios y acceso
        │
        ▼
Aplicar con set-default
        │
        ▼
Reiniciar
        │
        ▼
Verificar
```

---

# Ejemplo: convertir una estación en servidor sin GUI

Consultar:

```bash
systemctl get-default
```

Probar temporalmente:

```bash
sudo systemctl isolate \
multi-user.target
```

Verificar red y SSH:

```bash
systemctl status \
NetworkManager
```

```bash
systemctl status \
sshd
```

Aplicar permanentemente:

```bash
sudo systemctl set-default \
multi-user.target
```

---

# Ejemplo: restaurar la interfaz gráfica

Verificar paquetes y display manager:

```bash
systemctl status \
display-manager.service
```

Establecer:

```bash
sudo systemctl set-default \
graphical.target
```

Probar:

```bash
sudo systemctl isolate \
graphical.target
```

---

# Ejemplo: recuperar un sistema con error en `/etc/fstab`

1. Iniciar con:

```text
systemd.unit=emergency.target
```

2. Remontar raíz:

```bash
mount -o remount,rw /
```

3. Crear respaldo:

```bash
cp /etc/fstab \
/etc/fstab.bak
```

4. Revisar dispositivos:

```bash
lsblk -f
```

```bash
blkid
```

5. Corregir `/etc/fstab`.

6. Validar:

```bash
mount -av
```

7. Recargar:

```bash
systemctl daemon-reload
```

8. Continuar:

```bash
systemctl default
```

---

# Verificar el estado general

```bash
systemctl status
```

Puede mostrar:

- Estado del sistema.
- Unidades cargadas.
- Unidades fallidas.
- Trabajos pendientes.
- Estado degradado.

---

# Estado degradado

Consultar:

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

---

# Diagnosticar estado degradado

```bash
systemctl --failed
```

Después:

```bash
systemctl status \
nombre-unidad
```

```bash
journalctl \
-u nombre-unidad \
-b
```

---

# Buenas prácticas RHCSA

✔ Utilizar `systemctl get-default` antes de realizar cambios.

✔ Diferenciar claramente `set-default`, `start` e `isolate`.

✔ Probar un target temporalmente antes de establecerlo como predeterminado.

✔ Evitar `isolate` mediante SSH sin acceso alternativo.

✔ Mantener una consola disponible durante cambios críticos.

✔ Utilizar `systemd.unit=` para pruebas temporales desde GRUB.

✔ Revisar las dependencias con `list-dependencies`.

✔ No modificar directamente unidades de `/usr/lib/systemd/system`.

✔ Utilizar `/etc/systemd/system` para personalizaciones.

✔ Ejecutar `daemon-reload` después de cambiar unidades.

✔ Validar `/etc/fstab` antes de reiniciar.

✔ Comprender la diferencia entre rescue y emergency.

✔ No enmascarar targets críticos.

✔ Documentar el target predeterminado y cualquier cambio realizado.

---

# Errores comunes

## Confundir target activo con target predeterminado

El target activo puede cambiar temporalmente mediante `isolate`.

El predeterminado se consulta con:

```bash
systemctl get-default
```

---

## Utilizar `start` pensando que es igual a `isolate`

```bash
systemctl start TARGET
```

no detiene necesariamente las unidades ajenas al target.

---

## Ejecutar `isolate` desde una conexión remota

Puede detener la red o la sesión SSH.

---

## Cambiar a graphical sin tener paquetes gráficos

`graphical.target` puede activarse sin mostrar una interfaz si falta el display manager.

---

## Confundir rescue y emergency

Emergency ofrece un entorno más mínimo y puede dejar `/` en solo lectura.

---

## Editar unidades en `/usr/lib`

Los cambios pueden perderse tras una actualización.

---

## Olvidar `daemon-reload`

`systemd` puede continuar utilizando la configuración anterior.

---

## Establecer un target personalizado sin probarlo

Puede provocar un arranque incompleto.

---

## Enmascarar `multi-user.target`

Puede impedir alcanzar un estado operativo normal.

---

# Comandos importantes RHCSA

| Comando | Descripción |
|---|---|
| `systemctl get-default` | Mostrar target predeterminado |
| `systemctl set-default TARGET` | Cambiar target de futuros arranques |
| `systemctl isolate TARGET` | Cambiar inmediatamente de estado |
| `systemctl start TARGET` | Iniciar un target sin aislar |
| `systemctl status TARGET` | Consultar estado |
| `systemctl list-units --type=target` | Mostrar targets activos |
| `systemctl list-unit-files --type=target` | Mostrar todos los targets |
| `systemctl list-dependencies TARGET` | Mostrar dependencias |
| `systemctl list-dependencies --reverse TARGET` | Mostrar dependencias inversas |
| `systemctl cat TARGET` | Mostrar archivo de unidad |
| `systemctl show TARGET` | Mostrar propiedades |
| `systemctl rescue` | Entrar en modo rescue |
| `systemctl emergency` | Entrar en modo emergency |
| `systemctl default` | Intentar alcanzar el target predeterminado |
| `systemctl daemon-reload` | Recargar unidades |
| `systemctl --failed` | Mostrar unidades fallidas |
| `systemctl is-system-running` | Mostrar estado general |
| `systemctl add-wants TARGET UNIT` | Agregar dependencia débil |
| `readlink -f /etc/systemd/system/default.target` | Mostrar destino real |
| `journalctl -xb` | Analizar el arranque actual |

---

# Resumen rápido

```text
Targets de systemd
      │
      ├── Predeterminado
      │     ├── get-default
      │     └── set-default
      │
      ├── Cambio inmediato
      │     └── isolate
      │
      ├── Operación normal
      │     ├── multi-user.target
      │     └── graphical.target
      │
      ├── Recuperación
      │     ├── rescue.target
      │     └── emergency.target
      │
      ├── Energía
      │     ├── poweroff.target
      │     └── reboot.target
      │
      └── Dependencias
            ├── Requires
            ├── Wants
            ├── After
            └── Before
```

---

# Resumen

En esta lección aprendiste a:

- Comprender qué es un target de `systemd`.
- Relacionar targets con runlevels.
- Consultar y modificar el target predeterminado.
- Cambiar inmediatamente el estado con `isolate`.
- Diferenciar `multi-user.target` y `graphical.target`.
- Comprender `rescue.target` y `emergency.target`.
- Consultar dependencias y propiedades.
- Utilizar targets temporales desde GRUB.
- Diagnosticar arranques en modos incorrectos.
- Crear conceptos básicos de targets personalizados.
- Aplicar buenas prácticas de administración y recuperación.

---

# Laboratorio práctico RHCSA

> Realiza las tareas en una máquina virtual o en un sistema con acceso a consola. Evita cambios de aislamiento críticos únicamente desde SSH.

## Escenario 1: Consultar el target predeterminado

```bash
systemctl get-default
```

Después:

```bash
readlink -f \
/etc/systemd/system/default.target
```

Registra ambos resultados.

---

## Escenario 2: Listar targets activos

```bash
systemctl list-units \
--type=target \
--state=active
```

Identifica:

- `basic.target`
- `local-fs.target`
- `network.target`
- `multi-user.target`
- `graphical.target`

No todos estarán activos en todos los sistemas.

---

## Escenario 3: Consultar todos los targets

```bash
systemctl list-unit-files \
--type=target
```

Identifica cuáles son:

- `static`
- `alias`
- `disabled`
- `masked`

---

## Escenario 4: Examinar `multi-user.target`

```bash
systemctl status \
multi-user.target
```

```bash
systemctl cat \
multi-user.target
```

```bash
systemctl show \
multi-user.target \
-p Description \
-p ActiveState \
-p AllowIsolate
```

---

## Escenario 5: Consultar dependencias

```bash
systemctl list-dependencies \
multi-user.target
```

Después:

```bash
systemctl list-dependencies \
graphical.target
```

Compara ambas salidas.

---

## Escenario 6: Probar modo texto

Primero consulta:

```bash
systemctl get-default
```

Después, desde consola:

```bash
sudo systemctl isolate \
multi-user.target
```

Verifica:

```bash
systemctl is-active \
multi-user.target
```

---

## Escenario 7: Restaurar modo gráfico

Si el sistema dispone de interfaz gráfica:

```bash
sudo systemctl isolate \
graphical.target
```

Verifica:

```bash
systemctl is-active \
graphical.target
```

---

## Escenario 8: Cambiar el target predeterminado

Guarda el valor actual:

```bash
systemctl get-default \
> ~/target-original.txt
```

Cambia:

```bash
sudo systemctl set-default \
multi-user.target
```

Verifica:

```bash
systemctl get-default
```

---

## Escenario 9: Restaurar el target original

Consulta:

```bash
cat ~/target-original.txt
```

Si era `graphical.target`:

```bash
sudo systemctl set-default \
graphical.target
```

Si era `multi-user.target`:

```bash
sudo systemctl set-default \
multi-user.target
```

---

## Escenario 10: Arranque temporal con target

Desde GRUB:

1. Selecciona la entrada.
2. Presiona `e`.
3. Agrega:

```text
systemd.unit=multi-user.target
```

4. Inicia con `Ctrl+x`.

Después verifica:

```bash
systemctl list-units \
--type=target \
--state=active
```

```bash
systemctl get-default
```

El target predeterminado no debe haber cambiado.

---

## Escenario 11: Examinar rescue target

Sin aislar todavía:

```bash
systemctl cat \
rescue.target
```

```bash
systemctl list-dependencies \
rescue.target
```

```bash
systemctl show \
rescue.target \
-p AllowIsolate
```

---

## Escenario 12: Examinar emergency target

```bash
systemctl cat \
emergency.target
```

```bash
systemctl list-dependencies \
emergency.target
```

Compara con `rescue.target`.

---

## Escenario 13: Consultar red y targets

```bash
systemctl status \
network.target
```

```bash
systemctl status \
network-online.target
```

```bash
systemctl list-dependencies \
network-online.target
```

---

## Escenario 14: Estado general del sistema

```bash
systemctl is-system-running
```

```bash
systemctl --failed
```

Si existe una unidad fallida:

```bash
systemctl status \
nombre-unidad
```

---

## Escenario 15: Crear un target de laboratorio

Crear:

```bash
sudo vi \
/etc/systemd/system/laboratorio.target
```

Contenido:

```ini
[Unit]
Description=Target de laboratorio RHCSA
Requires=multi-user.target
After=multi-user.target
AllowIsolate=yes
```

Recargar:

```bash
sudo systemctl daemon-reload
```

Verificar:

```bash
systemctl status \
laboratorio.target
```

---

## Escenario 16: Agregar una dependencia

Utiliza un servicio pequeño instalado en el sistema.

Ejemplo:

```bash
sudo systemctl add-wants \
laboratorio.target \
chronyd.service
```

Verifica:

```bash
systemctl list-dependencies \
laboratorio.target
```

---

## Escenario 17: Iniciar el target personalizado

```bash
sudo systemctl start \
laboratorio.target
```

Verifica:

```bash
systemctl is-active \
laboratorio.target
```

---

## Escenario 18: Limpiar el laboratorio

Detener:

```bash
sudo systemctl stop \
laboratorio.target
```

Eliminar enlaces:

```bash
sudo rm -rf \
/etc/systemd/system/laboratorio.target.wants
```

Eliminar unidad:

```bash
sudo rm -f \
/etc/systemd/system/laboratorio.target
```

Recargar:

```bash
sudo systemctl daemon-reload
```

Verificar:

```bash
systemctl status \
laboratorio.target
```

---

# Script opcional de auditoría

```bash
#!/bin/bash

REPORTE="$HOME/reporte-targets-systemd.txt"

{
    echo "=================================================="
    echo "REPORTE DE TARGETS DE SYSTEMD"
    echo "=================================================="
    echo

    echo "Fecha:"
    date
    echo

    echo "Hostname:"
    hostname
    echo

    echo "Target predeterminado:"
    systemctl get-default
    echo

    echo "Enlace default.target:"
    readlink -f /etc/systemd/system/default.target
    echo

    echo "Targets activos:"
    systemctl list-units \
    --type=target \
    --state=active \
    --no-pager
    echo

    echo "Targets disponibles:"
    systemctl list-unit-files \
    --type=target \
    --no-pager
    echo

    echo "Estado general:"
    systemctl is-system-running
    echo

    echo "Unidades fallidas:"
    systemctl --failed --no-pager
    echo

    echo "Dependencias de multi-user.target:"
    systemctl list-dependencies \
    multi-user.target \
    --no-pager
    echo

    echo "Dependencias de graphical.target:"
    systemctl list-dependencies \
    graphical.target \
    --no-pager
    echo

    echo "Propiedades de rescue.target:"
    systemctl show \
    rescue.target \
    -p Description \
    -p ActiveState \
    -p AllowIsolate
    echo

    echo "Propiedades de emergency.target:"
    systemctl show \
    emergency.target \
    -p Description \
    -p ActiveState \
    -p AllowIsolate
    echo

    echo "=================================================="
    echo "FIN DEL REPORTE"
    echo "=================================================="

} > "$REPORTE" 2>&1

echo "Reporte generado en: $REPORTE"
```

Guardar como:

```text
~/auditar-targets.sh
```

Asignar permisos:

```bash
chmod +x \
~/auditar-targets.sh
```

Ejecutar:

```bash
~/auditar-targets.sh
```

---

# Preguntas de repaso

1. ¿Qué es un target de `systemd`?
2. ¿Qué extensión utiliza?
3. ¿Qué target equivale aproximadamente al runlevel 3?
4. ¿Qué target equivale al runlevel 5?
5. ¿Cómo se consulta el target predeterminado?
6. ¿Qué función cumple `default.target`?
7. ¿Cómo se cambia permanentemente el target?
8. ¿Qué función cumple `systemctl isolate`?
9. ¿Cuál es la diferencia entre `start` e `isolate`?
10. ¿Qué riesgo tiene ejecutar `isolate` por SSH?
11. ¿Qué servicios suele incluir `multi-user.target`?
12. ¿Qué añade `graphical.target`?
13. ¿Qué diferencia existe entre rescue y emergency?
14. ¿Cómo se entra en rescue desde GRUB?
15. ¿Cómo se entra en emergency desde GRUB?
16. ¿Qué comando permite regresar al target predeterminado?
17. ¿Qué diferencia existe entre `Requires=` y `Wants=`?
18. ¿Qué diferencia existe entre `After=` y `Requires=`?
19. ¿Qué significa que una unidad sea `static`?
20. ¿Dónde deben almacenarse las personalizaciones?
21. ¿Cuándo debe ejecutarse `daemon-reload`?
22. ¿Qué comando muestra las dependencias?
23. ¿Cómo se consulta si un target permite aislamiento?
24. ¿Qué diferencia existe entre `network.target` y `network-online.target`?
25. ¿Por qué no debe enmascararse un target crítico?

---

# Desafío final

Realiza las siguientes tareas:

1. Consulta el target predeterminado.
2. Identifica el destino de `default.target`.
3. Lista los targets activos.
4. Lista todos los archivos target.
5. Consulta las dependencias de `multi-user.target`.
6. Consulta las dependencias de `graphical.target`.
7. Identifica el display manager.
8. Cambia temporalmente a modo texto.
9. Regresa al modo gráfico.
10. Cambia permanentemente el target.
11. Restaura el valor original.
12. Inicia temporalmente con un target desde GRUB.
13. Compara rescue y emergency.
14. Consulta el estado general del sistema.
15. Identifica unidades fallidas.
16. Crea un target personalizado.
17. Agrega una dependencia con `add-wants`.
18. Inicia y valida el target.
19. Elimina completamente el target de laboratorio.
20. Genera un reporte de auditoría.

> **Objetivo general:** administrar de forma segura los estados operativos del sistema mediante targets de `systemd`, comprender su relación con los antiguos runlevels y utilizar `multi-user`, `graphical`, `rescue` y `emergency` para operación normal, mantenimiento y recuperación en sistemas Red Hat Enterprise Linux.