# 56. Historial y Administración Avanzada de DNF

> **Módulo 8: Administración del Software y Repositorios**  
> **Página 56 del Manual RHCSA**

---

# Objetivos de aprendizaje

Al finalizar esta lección serás capaz de:

- Consultar el historial de operaciones realizadas con DNF.
- Identificar instalaciones, actualizaciones y eliminaciones anteriores.
- Obtener detalles de una transacción.
- Revertir operaciones mediante `undo`.
- Repetir operaciones mediante `redo`.
- Sincronizar paquetes con los repositorios.
- Administrar dependencias innecesarias.
- Excluir paquetes de una actualización.
- Descargar paquetes sin instalarlos.
- Trabajar con repositorios de manera temporal.
- Aplicar buenas prácticas en tareas avanzadas de administración.

---

# Introducción

DNF no solo permite instalar, actualizar y eliminar software.

También registra las operaciones realizadas en el sistema.

Este historial resulta útil para:

- Investigar cambios.
- Saber quién instaló o eliminó un paquete.
- Identificar cuándo se actualizó una aplicación.
- Revertir una transacción.
- Repetir una operación.
- Diagnosticar problemas después de una actualización.

En servidores de producción, estas funciones son especialmente importantes porque permiten reconstruir el historial de cambios del software.

---

# ¿Qué es una transacción DNF?

Una transacción es un conjunto de operaciones ejecutadas por DNF como una sola acción lógica.

Ejemplo:

```bash
sudo dnf install httpd
```

Aunque el usuario solicita un solo paquete, DNF puede:

- Instalar `httpd`.
- Instalar dependencias.
- Actualizar bibliotecas.
- Ejecutar scripts.
- Registrar todos los cambios.

Todo esto queda almacenado como una transacción.

---

# Flujo de una transacción

```text
Comando DNF
    │
    ▼
Resolución de dependencias
    │
    ▼
Descarga de paquetes
    │
    ▼
Verificación GPG
    │
    ▼
Instalación, actualización o eliminación
    │
    ▼
Registro en el historial
```

---

# Consultar el historial

```bash
dnf history
```

También:

```bash
dnf history list
```

---

# Ejemplo de salida

```text
ID     Command line                 Date and time       Action(s)      Altered
25     install httpd                2026-07-20 10:32    Install           6
24     update                       2026-07-19 22:00    Upgrade          48
23     remove nginx                 2026-07-18 14:10    Removed           4
```

---

# Interpretar las columnas

| Columna | Descripción |
|---------|-------------|
| ID | Identificador de la transacción |
| Command line | Comando ejecutado |
| Date and time | Fecha y hora |
| Action(s) | Tipo de operación |
| Altered | Cantidad de paquetes afectados |

---

# Tipos de acciones

DNF puede registrar acciones como:

| Acción | Significado |
|--------|-------------|
| Install | Paquetes instalados |
| Upgrade | Paquetes actualizados |
| Downgrade | Paquetes degradados |
| Removed | Paquetes eliminados |
| Reinstall | Paquetes reinstalados |
| Erase | Paquetes borrados |
| Obsoleted | Paquetes reemplazados |
| Reason change | Cambio en el motivo de instalación |

---

# Consultar una transacción específica

```bash
dnf history info ID
```

Ejemplo:

```bash
dnf history info 25
```

---

# Información disponible

La salida puede incluir:

- Fecha y hora.
- Usuario.
- Comando ejecutado.
- Versión de DNF.
- Paquetes instalados.
- Paquetes actualizados.
- Paquetes eliminados.
- Estado de la transacción.
- Código de retorno.

---

# Consultar varias transacciones

```bash
dnf history info 20..25
```

Esto muestra información desde la transacción 20 hasta la 25.

---

# Consultar el historial de un paquete

```bash
dnf history list httpd
```

También:

```bash
dnf history package-list httpd
```

Dependiendo de la versión de DNF, el subcomando disponible puede variar.

---

# Buscar operaciones relacionadas con un paquete

```bash
dnf history | grep -i httpd
```

Después consulta la transacción correspondiente:

```bash
dnf history info ID
```

---

# Resumen general del historial

```bash
dnf history summary
```

Esta salida resume el número de operaciones realizadas.

---

# Consultar paquetes modificados

```bash
dnf history package-list
```

Puede mostrar paquetes:

- Instalados.
- Actualizados.
- Eliminados.
- Reinstalados.
- Degradados.

---

# Revertir una transacción con `undo`

Sintaxis:

```bash
sudo dnf history undo ID
```

Ejemplo:

```bash
sudo dnf history undo 25
```

DNF intentará realizar la operación opuesta.

---

# Ejemplo conceptual

Transacción original:

```text
install httpd
```

Operación `undo`:

```text
remove httpd
```

Otro ejemplo:

```text
remove nginx
```

Operación `undo`:

```text
install nginx
```

---

# Limitaciones de `undo`

`undo` no garantiza que siempre pueda restaurarse exactamente el estado anterior.

Puede fallar si:

- El paquete anterior ya no existe en los repositorios.
- Cambiaron las dependencias.
- El repositorio fue eliminado.
- La versión anterior dejó de estar disponible.
- Se modificaron archivos de configuración manualmente.
- La operación afectó datos de una aplicación.

---

# Advertencia importante

DNF administra paquetes, pero no siempre puede revertir:

- Datos de bases de datos.
- Archivos creados por usuarios.
- Cambios manuales.
- Configuraciones modificadas después de la instalación.
- Migraciones de aplicaciones.

Por ejemplo, desinstalar PostgreSQL no elimina necesariamente todos sus datos, pero revertir paquetes tampoco restaura automáticamente una base de datos.

---

# Repetir una transacción con `redo`

Sintaxis:

```bash
sudo dnf history redo ID
```

Ejemplo:

```bash
sudo dnf history redo 25
```

DNF intentará repetir la transacción original.

---

# Ejemplo

Transacción original:

```text
install httpd
```

Con:

```bash
sudo dnf history redo 25
```

DNF intentará instalar nuevamente los mismos paquetes.

---

# Cuándo utilizar `redo`

Puede ser útil cuando:

- Una transacción fue revertida.
- Se desea repetir una instalación.
- Se necesita reproducir una operación.
- Se está practicando en un laboratorio.
- Se desea reconstruir un entorno similar.

---

# Revertir hasta una transacción anterior

```bash
sudo dnf history rollback ID
```

Ejemplo:

```bash
sudo dnf history rollback 20
```

Esto intenta revertir todas las transacciones posteriores hasta dejar el sistema en el estado correspondiente a la transacción indicada.

---

# Diferencia entre `undo` y `rollback`

| Comando | Función |
|---------|---------|
| `undo ID` | Revierte una sola transacción |
| `rollback ID` | Revierte varias transacciones posteriores |
| `redo ID` | Repite una transacción |

---

# Precaución con `rollback`

`rollback` puede afectar una gran cantidad de paquetes.

Antes de confirmar:

- Revisa la lista completa.
- Verifica versiones disponibles.
- Confirma que los repositorios estén activos.
- Realiza respaldo.
- Evita utilizarlo sin pruebas en producción.

---

# Verificar el estado de una transacción

```bash
dnf history info ID
```

Busca campos como:

```text
Return-Code
```

Una transacción puede aparecer como:

- Correcta.
- Incompleta.
- Interrumpida.
- Fallida.

---

# Transacciones interrumpidas

Una operación puede quedar incompleta por:

- Reinicio.
- Corte eléctrico.
- Pérdida de conexión.
- Terminación del proceso.
- Error del repositorio.
- Falta de espacio.

Consultar:

```bash
dnf history
```

Después:

```bash
dnf history info ID
```

---

# Comprobar el sistema después de una interrupción

```bash
sudo dnf check
```

También puede utilizarse:

```bash
sudo rpm -Va
```

Y para revisar dependencias:

```bash
sudo dnf repoquery --unsatisfied
```

La disponibilidad exacta de algunos subcomandos depende de los complementos instalados.

---

# Sincronizar paquetes con los repositorios

```bash
sudo dnf distro-sync
```

Este comando ajusta las versiones instaladas para que coincidan con las disponibles en los repositorios habilitados.

Puede:

- Actualizar paquetes.
- Degradar paquetes.
- Corregir inconsistencias de versiones.

---

# Ejemplo de uso

```bash
sudo dnf distro-sync
```

Para un paquete específico:

```bash
sudo dnf distro-sync httpd
```

---

# Cuándo utilizar `distro-sync`

- Después de cambiar repositorios.
- Después de cambiar un stream modular.
- Cuando existen paquetes con versiones incompatibles.
- Después de migrar entre fuentes de software.
- Cuando el sistema tiene paquetes más nuevos que el repositorio actual.

---

# Precaución con `distro-sync`

Puede degradar paquetes.

Siempre revisa la transacción antes de confirmar.

---

# Degradar un paquete

```bash
sudo dnf downgrade paquete
```

Ejemplo:

```bash
sudo dnf downgrade httpd
```

DNF intentará instalar una versión anterior disponible en los repositorios.

---

# Consultar versiones disponibles

```bash
dnf list --showduplicates httpd
```

También:

```bash
dnf repoquery --show-duplicates httpd
```

---

# Instalar una versión específica

Sintaxis:

```bash
sudo dnf install paquete-version
```

Ejemplo conceptual:

```bash
sudo dnf install httpd-2.4.57
```

La versión debe existir en un repositorio habilitado o estar disponible localmente.

---

# Reinstalar un paquete

```bash
sudo dnf reinstall httpd
```

Útil cuando:

- Se eliminó un archivo.
- Se dañó una biblioteca.
- Se modificó un ejecutable.
- Se desea restaurar archivos originales.

---

# Verificar antes de reinstalar

```bash
rpm -V httpd
```

Si aparecen diferencias, la reinstalación puede restaurar los archivos proporcionados por el paquete.

---

# Dependencias innecesarias

Cuando se instala software, DNF puede instalar dependencias adicionales.

Si el paquete principal se elimina, algunas dependencias pueden quedar sin uso.

---

# Detectar paquetes innecesarios

```bash
dnf repoquery --unneeded
```

---

# Eliminar dependencias innecesarias

```bash
sudo dnf autoremove
```

---

# Precaución con `autoremove`

Antes de confirmar, revisa cuidadosamente los paquetes que serán eliminados.

No debe ejecutarse automáticamente sin revisar la transacción en servidores críticos.

---

# Motivo de instalación

DNF registra por qué se instaló un paquete.

Puede ser:

- Instalado directamente por el usuario.
- Instalado como dependencia.
- Instalado como parte de un grupo.
- Instalado como parte de un módulo.

---

# Consultar motivo de instalación

En versiones que lo soporten:

```bash
dnf repoquery --installed --qf '%{name} %{reason}'
```

También puede consultarse con:

```bash
dnf history userinstalled
```

---

# Mostrar paquetes instalados por el usuario

```bash
dnf history userinstalled
```

Esto ayuda a diferenciar:

- Paquetes solicitados directamente.
- Dependencias instaladas automáticamente.

---

# Marcar un paquete como instalado por el usuario

```bash
sudo dnf mark install paquete
```

Ejemplo:

```bash
sudo dnf mark install httpd
```

Esto evita que pueda considerarse una dependencia innecesaria.

---

# Marcar un paquete como dependencia

```bash
sudo dnf mark remove paquete
```

En algunas versiones también puede utilizarse:

```bash
sudo dnf mark dependency paquete
```

Consulta la sintaxis disponible:

```bash
dnf mark --help
```

---

# Actualizar únicamente paquetes de seguridad

```bash
sudo dnf upgrade --security
```

---

# Consultar actualizaciones de seguridad

```bash
dnf updateinfo list security
```

También:

```bash
dnf updateinfo info security
```

---

# Aplicar una recomendación específica

```bash
sudo dnf update \
--advisory=RHSA-2026:1234
```

El identificador es solo un ejemplo.

---

# Tipos de avisos

| Tipo | Descripción |
|------|-------------|
| RHSA | Red Hat Security Advisory |
| RHBA | Red Hat Bug Advisory |
| RHEA | Red Hat Enhancement Advisory |

---

# Consultar todos los avisos

```bash
dnf updateinfo list
```

---

# Mostrar resumen de avisos

```bash
dnf updateinfo summary
```

---

# Excluir paquetes de una actualización

Excluir temporalmente:

```bash
sudo dnf update \
--exclude=kernel*
```

Excluir varios:

```bash
sudo dnf update \
--exclude=kernel* \
--exclude=postgresql*
```

---

# Exclusión permanente

Editar:

```text
/etc/dnf/dnf.conf
```

Agregar:

```ini
exclude=kernel* postgresql*
```

---

# Precaución con exclusiones

Excluir paquetes durante mucho tiempo puede provocar:

- Vulnerabilidades.
- Dependencias incompatibles.
- Software desactualizado.
- Problemas de soporte.

Toda exclusión debe estar documentada.

---

# Habilitar o deshabilitar repositorios temporalmente

Habilitar:

```bash
sudo dnf \
--enablerepo=repo-laboratorio \
install paquete
```

Deshabilitar:

```bash
sudo dnf \
--disablerepo=repo-laboratorio \
update
```

---

# Usar únicamente un repositorio

```bash
sudo dnf \
--disablerepo="*" \
--enablerepo="repo-laboratorio" \
install paquete
```

---

# Ignorar temporalmente la verificación GPG

```bash
sudo dnf install paquete \
--nogpgcheck
```

⚠ Esta opción debe utilizarse únicamente en laboratorios controlados.

No se recomienda en producción.

---

# Descargar paquetes sin instalarlos

Primero instala:

```bash
sudo dnf install dnf-plugins-core
```

Después:

```bash
dnf download httpd
```

---

# Descargar dependencias

```bash
dnf download \
--resolve \
httpd
```

---

# Descargar en un directorio específico

```bash
dnf download \
--destdir=/tmp/paquetes \
httpd
```

---

# Descargar el paquete fuente

```bash
dnf download \
--source \
httpd
```

Puede requerir repositorios de código fuente habilitados.

---

# Descargar únicamente

Con `dnf install` puede utilizarse:

```bash
sudo dnf install \
--downloadonly \
--downloaddir=/tmp/paquetes \
httpd
```

---

# Consultar dependencias

```bash
dnf repoquery --requires httpd
```

Mostrar dependencias resueltas:

```bash
dnf repoquery \
--requires \
--resolve \
httpd
```

---

# Consultar dependencias inversas

```bash
dnf repoquery --whatrequires openssl
```

---

# Identificar qué paquete proporciona un archivo

```bash
dnf provides /usr/bin/htop
```

Con comodín:

```bash
dnf provides "*/htop"
```

---

# Consultar archivos de un paquete disponible

```bash
dnf repoquery --list httpd
```

---

# Consultar paquetes duplicados

```bash
dnf repoquery --duplicates
```

Puede ayudar a detectar varias versiones instaladas.

---

# Consultar paquetes extras

```bash
dnf repoquery --extras
```

Muestra paquetes instalados que no existen en los repositorios habilitados.

---

# Consultar paquetes más recientes

```bash
dnf repoquery --latest-limit 1
```

Para un paquete:

```bash
dnf repoquery \
--latest-limit 1 \
httpd
```

---

# Limpiar la caché

```bash
sudo dnf clean all
```

---

# Tipos de limpieza

```bash
sudo dnf clean metadata
```

```bash
sudo dnf clean packages
```

```bash
sudo dnf clean expire-cache
```

```bash
sudo dnf clean dbcache
```

---

# Regenerar la caché

```bash
sudo dnf makecache
```

Forzar actualización:

```bash
sudo dnf makecache --refresh
```

---

# Verificar el espacio utilizado por la caché

```bash
du -sh /var/cache/dnf
```

---

# Consultar configuración de DNF

```bash
dnf config-manager --dump
```

También:

```bash
cat /etc/dnf/dnf.conf
```

---

# Parámetros útiles de DNF

| Opción | Función |
|--------|---------|
| `-y` | Confirmar automáticamente |
| `--assumeno` | Responder no automáticamente |
| `--refresh` | Forzar actualización de metadatos |
| `--downloadonly` | Descargar sin instalar |
| `--exclude` | Excluir paquetes |
| `--enablerepo` | Habilitar repositorio temporal |
| `--disablerepo` | Deshabilitar repositorio temporal |
| `--security` | Limitar a seguridad |
| `--best` | Seleccionar la mejor versión |
| `--allowerasing` | Permitir eliminar paquetes en conflictos |
| `--nogpgcheck` | Omitir validación GPG |
| `--setopt` | Modificar una opción temporalmente |

---

# Simular una operación

DNF no tiene un modo de simulación perfecto para todas las operaciones, pero puede utilizarse:

```bash
sudo dnf install httpd --assumeno
```

DNF resolverá la transacción y mostrará los cambios sin confirmarlos.

---

# Ejemplo

```bash
sudo dnf remove httpd --assumeno
```

Esto permite revisar:

- Paquetes que se eliminarían.
- Dependencias afectadas.
- Tamaño de la operación.

---

# Utilizar `--setopt`

Modificar una opción solo para un comando:

```bash
sudo dnf \
--setopt=install_weak_deps=False \
install paquete
```

---

# Dependencias débiles

DNF puede instalar dependencias recomendadas que no son estrictamente obligatorias.

Para evitarlo temporalmente:

```bash
sudo dnf install paquete \
--setopt=install_weak_deps=False
```

---

# Configuración permanente

Editar:

```text
/etc/dnf/dnf.conf
```

Agregar:

```ini
install_weak_deps=False
```

Debe utilizarse con cuidado, porque algunas funcionalidades opcionales podrían no instalarse.

---

# Permitir borrado para resolver conflictos

```bash
sudo dnf upgrade --allowerasing
```

Esta opción permite eliminar paquetes conflictivos durante la resolución.

⚠ Debe revisarse cuidadosamente la transacción.

---

# Seleccionar la mejor versión

```bash
sudo dnf upgrade --best
```

En algunos sistemas `best=True` ya está configurado.

---

# Saltar paquetes no disponibles

```bash
sudo dnf upgrade --skip-broken
```

Esta opción omite paquetes con dependencias no resolubles.

No debe convertirse en una solución permanente.

---

# Diagnóstico después de `--skip-broken`

Consulta:

```bash
sudo dnf check
```

```bash
dnf repoquery --unsatisfied
```

```bash
dnf repolist
```

```bash
sudo dnf makecache --refresh
```

---

# Bloqueo de versiones

DNF puede bloquear paquetes para impedir su actualización.

Esta función suele requerir:

```bash
sudo dnf install python3-dnf-plugin-versionlock
```

En algunas versiones el paquete puede formar parte de:

```bash
dnf-plugins-core
```

---

# Listar bloqueos

```bash
dnf versionlock list
```

---

# Bloquear un paquete

```bash
sudo dnf versionlock add httpd
```

---

# Eliminar un bloqueo

```bash
sudo dnf versionlock delete httpd
```

---

# Eliminar todos los bloqueos

```bash
sudo dnf versionlock clear
```

---

# Cuándo utilizar `versionlock`

Puede utilizarse cuando:

- Una aplicación requiere una versión específica.
- Existe un problema conocido con una actualización.
- Se está esperando validación del fabricante.
- Se necesita estabilidad temporal.

---

# Riesgos de bloquear versiones

- Falta de parches de seguridad.
- Dependencias incompatibles.
- Pérdida de soporte.
- Dificultad para actualizar más adelante.

Todo bloqueo debe:

- Tener una justificación.
- Tener una fecha de revisión.
- Estar documentado.
- Ser eliminado cuando deje de ser necesario.

---

# Consultar el log de DNF

Archivos comunes:

```text
/var/log/dnf.log
```

```text
/var/log/dnf.rpm.log
```

```text
/var/log/dnf.librepo.log
```

---

# Ver el log

```bash
sudo less /var/log/dnf.log
```

Buscar errores:

```bash
sudo grep -i error /var/log/dnf.log
```

---

# Consultar DNF con Journal

Dependiendo del sistema:

```bash
journalctl | grep -i dnf
```

También puede revisarse el historial de sudo:

```bash
journalctl _COMM=sudo
```

---

# Diagnóstico de dependencias rotas

Ejecutar:

```bash
sudo dnf check
```

Consultar dependencias incumplidas:

```bash
dnf repoquery --unsatisfied
```

Sincronizar:

```bash
sudo dnf distro-sync
```

Reinstalar paquetes dañados:

```bash
sudo dnf reinstall paquete
```

---

# Error: paquete duplicado

Consultar:

```bash
dnf repoquery --duplicates
```

Después revisar versiones:

```bash
rpm -qa | grep nombre_paquete
```

La eliminación de duplicados debe realizarse con precaución.

---

# Error: paquete extra

Consultar:

```bash
dnf repoquery --extras
```

Esto puede ocurrir cuando:

- Se instaló un RPM manualmente.
- Se eliminó un repositorio.
- El paquete ya no existe en los repositorios actuales.

---

# Error: no se puede revertir una transacción

Posibles causas:

- La versión anterior ya no existe.
- El repositorio no está disponible.
- Los metadatos fueron eliminados.
- Existen conflictos de dependencias.
- El paquete fue reemplazado.

Posibles acciones:

```bash
dnf list --showduplicates paquete
```

```bash
dnf repolist all
```

```bash
sudo dnf makecache --refresh
```

---

# Error: another app is currently holding the dnf lock

Puede ocurrir cuando otro proceso DNF está en ejecución.

Verificar:

```bash
ps aux | grep -E '[d]nf|[r]pm'
```

No elimines archivos de bloqueo sin confirmar que no existe otro proceso activo.

---

# Ver procesos relacionados

```bash
pgrep -a dnf
```

```bash
pgrep -a rpm
```

---

# Integridad de la base de datos RPM

En caso de problemas graves puede verificarse:

```bash
sudo rpm --verifydb
```

La reconstrucción de la base de datos debe realizarse solo cuando exista evidencia clara de corrupción.

Comando:

```bash
sudo rpm --rebuilddb
```

⚠ No debe ejecutarse como mantenimiento rutinario.

---

# Flujo de diagnóstico recomendado

```text
Revisar error
    │
    ▼
Consultar dnf history
    │
    ▼
Revisar logs
    │
    ▼
Validar repositorios
    │
    ▼
Actualizar metadatos
    │
    ▼
Ejecutar dnf check
    │
    ▼
Consultar dependencias
    │
    ▼
Sincronizar o reinstalar
```

---

# Ejemplo práctico de auditoría

Supongamos que Apache dejó de funcionar después de una actualización.

## Paso 1: Revisar el historial

```bash
dnf history
```

---

## Paso 2: Consultar la transacción

```bash
dnf history info 25
```

---

## Paso 3: Ver paquetes modificados

Busca:

```text
httpd
mod_ssl
apr
openssl
```

---

## Paso 4: Revisar el servicio

```bash
systemctl status httpd
```

---

## Paso 5: Revisar logs

```bash
journalctl -u httpd -n 100
```

---

## Paso 6: Consultar versiones

```bash
dnf list installed httpd
```

```bash
dnf list --showduplicates httpd
```

---

## Paso 7: Evaluar reversión

```bash
sudo dnf history undo 25 --assumeno
```

Primero revisa la transacción.

Después, si es seguro:

```bash
sudo dnf history undo 25
```

---

# Ejemplo de actualización controlada

Consultar actualizaciones:

```bash
dnf check-update
```

Consultar avisos de seguridad:

```bash
dnf updateinfo summary
```

Simular:

```bash
sudo dnf upgrade --security --assumeno
```

Aplicar:

```bash
sudo dnf upgrade --security
```

Registrar el resultado:

```bash
dnf history
```

---

# Buenas prácticas RHCSA

✔ Consultar `dnf history` antes de revertir cambios.

✔ Revisar siempre la transacción antes de confirmar.

✔ Utilizar `--assumeno` para estudiar el impacto.

✔ Mantener repositorios oficiales habilitados.

✔ Documentar exclusiones y bloqueos de versión.

✔ No utilizar `--nogpgcheck` en producción.

✔ No ejecutar `autoremove` sin revisar los paquetes.

✔ Realizar respaldos antes de cambios importantes.

✔ Utilizar `distro-sync` con precaución.

✔ Consultar logs cuando una transacción falle.

✔ No reconstruir la base RPM como mantenimiento rutinario.

✔ Verificar servicios después de actualizaciones.

---

# Errores comunes

## Revertir una transacción sin leer sus detalles

Puede eliminar paquetes necesarios.

---

## Pensar que `undo` restaura datos

DNF revierte paquetes, no necesariamente datos ni configuraciones.

---

## Utilizar `rollback` sin respaldo

Puede afectar múltiples transacciones y paquetes.

---

## Ejecutar `autoremove -y` automáticamente

Puede eliminar software considerado innecesario, pero utilizado indirectamente por el administrador.

---

## Mantener exclusiones permanentes

Puede dejar el sistema vulnerable.

---

## Utilizar `--skip-broken` como solución definitiva

Solo oculta temporalmente un problema de dependencias.

---

## Bloquear paquetes sin fecha de revisión

El paquete puede quedar desactualizado indefinidamente.

---

## Eliminar el bloqueo manualmente

Primero verifica si existe otro proceso DNF o RPM activo.

---

# Comandos importantes RHCSA

| Comando | Descripción |
|---------|-------------|
| `dnf history` | Mostrar historial |
| `dnf history info ID` | Detalles de una transacción |
| `dnf history undo ID` | Revertir una transacción |
| `dnf history redo ID` | Repetir una transacción |
| `dnf history rollback ID` | Revertir hasta una transacción |
| `dnf history userinstalled` | Mostrar paquetes instalados por el usuario |
| `dnf distro-sync` | Sincronizar versiones |
| `dnf downgrade paquete` | Instalar una versión anterior |
| `dnf reinstall paquete` | Reinstalar un paquete |
| `dnf autoremove` | Eliminar dependencias innecesarias |
| `dnf updateinfo summary` | Resumir avisos |
| `dnf upgrade --security` | Aplicar actualizaciones de seguridad |
| `dnf repoquery --unneeded` | Mostrar paquetes innecesarios |
| `dnf repoquery --extras` | Mostrar paquetes sin repositorio |
| `dnf repoquery --duplicates` | Mostrar duplicados |
| `dnf repoquery --unsatisfied` | Mostrar dependencias incumplidas |
| `dnf download paquete` | Descargar un paquete |
| `dnf clean all` | Limpiar la caché |
| `dnf makecache --refresh` | Actualizar metadatos |
| `dnf check` | Comprobar la base de paquetes |
| `dnf versionlock add paquete` | Bloquear una versión |

---

# Resumen rápido

```text
DNF avanzado
    │
    ├── Historial
    │     ├── info
    │     ├── undo
    │     ├── redo
    │     └── rollback
    │
    ├── Control de versiones
    │     ├── downgrade
    │     ├── distro-sync
    │     └── versionlock
    │
    ├── Mantenimiento
    │     ├── autoremove
    │     ├── clean
    │     └── makecache
    │
    ├── Seguridad
    │     ├── updateinfo
    │     └── upgrade --security
    │
    └── Diagnóstico
          ├── dnf check
          ├── repoquery
          └── logs
```

---

# Resumen

En esta lección aprendiste a:

- Consultar el historial de DNF.
- Analizar transacciones.
- Revertir y repetir operaciones.
- Sincronizar versiones con los repositorios.
- Degradar y reinstalar paquetes.
- Administrar dependencias innecesarias.
- Aplicar actualizaciones de seguridad.
- Excluir o bloquear paquetes.
- Descargar software sin instalarlo.
- Diagnosticar conflictos y dependencias rotas.
- Aplicar buenas prácticas en servidores RHEL.

---

# Laboratorio práctico RHCSA

## Escenario 1: Crear una transacción

Instala un paquete pequeño:

```bash
sudo dnf install tree
```

Consulta el historial:

```bash
dnf history
```

Identifica el ID de la instalación.

---

## Escenario 2: Consultar detalles

```bash
dnf history info ID
```

Responde:

- ¿Qué comando se ejecutó?
- ¿Qué paquetes fueron afectados?
- ¿Qué usuario ejecutó la operación?
- ¿Cuál fue el resultado?

---

## Escenario 3: Revertir la operación

Primero simula:

```bash
sudo dnf history undo ID --assumeno
```

Después revierte:

```bash
sudo dnf history undo ID
```

Comprueba:

```bash
rpm -q tree
```

---

## Escenario 4: Repetir la transacción

```bash
sudo dnf history redo ID
```

Verifica:

```bash
rpm -q tree
```

---

## Escenario 5: Consultar paquetes innecesarios

```bash
dnf repoquery --unneeded
```

Simula la limpieza:

```bash
sudo dnf autoremove --assumeno
```

No confirmes hasta revisar la lista.

---

## Escenario 6: Consultar actualizaciones de seguridad

```bash
dnf updateinfo summary
```

```bash
dnf updateinfo list security
```

Simula:

```bash
sudo dnf upgrade --security --assumeno
```

---

## Escenario 7: Descargar un paquete

Instala los complementos si no existen:

```bash
sudo dnf install dnf-plugins-core
```

Crea un directorio:

```bash
mkdir -p ~/paquetes
```

Descarga:

```bash
dnf download \
--destdir=~/paquetes \
tree
```

Comprueba:

```bash
ls -lh ~/paquetes
```

---

## Escenario 8: Consultar versiones

```bash
dnf list --showduplicates bash
```

Consulta la versión instalada:

```bash
rpm -q bash
```

---

## Escenario 9: Consultar integridad

```bash
sudo dnf check
```

```bash
dnf repoquery --unsatisfied
```

```bash
dnf repoquery --extras
```

```bash
dnf repoquery --duplicates
```

---

## Escenario 10: Revisar logs

```bash
sudo tail -n 50 /var/log/dnf.log
```

Busca operaciones de instalación:

```bash
sudo grep -i install /var/log/dnf.log | tail
```

---

# Preguntas de repaso

1. ¿Qué información registra `dnf history`?
2. ¿Cuál es la diferencia entre `undo`, `redo` y `rollback`?
3. ¿Por qué `undo` no restaura necesariamente los datos de una aplicación?
4. ¿Qué función cumple `distro-sync`?
5. ¿Cómo se instala una versión anterior de un paquete?
6. ¿Qué comando muestra dependencias innecesarias?
7. ¿Qué riesgo tiene ejecutar `autoremove` sin revisar?
8. ¿Cómo se aplican únicamente actualizaciones de seguridad?
9. ¿Qué función cumple `versionlock`?
10. ¿Qué comando permite simular una operación sin confirmarla?
11. ¿Qué muestra `dnf repoquery --extras`?
12. ¿Cuándo debe utilizarse `rpm --rebuilddb`?

---

# Desafío final

Realiza las siguientes tareas:

1. Instala un paquete de prueba.
2. Identifica la transacción.
3. Consulta sus detalles.
4. Revierte la instalación.
5. Repite la transacción.
6. Consulta las versiones disponibles.
7. Descarga el paquete sin instalarlo.
8. Consulta paquetes innecesarios.
9. Revisa las actualizaciones de seguridad.
10. Examina los logs de DNF.
11. Ejecuta una simulación con `--assumeno`.
12. Documenta todas las transacciones realizadas.

> **Objetivo general:** dominar el historial y las funciones avanzadas de **DNF** para auditar, revertir, repetir, sincronizar y diagnosticar operaciones de software en Red Hat Enterprise Linux. Estas competencias permiten administrar cambios de paquetes de forma segura y resolver problemas comunes en entornos reales y en el examen **RHCSA**.