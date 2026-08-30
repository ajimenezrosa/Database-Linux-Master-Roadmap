Aquí tienes la página completa para tu libro web de Linux:

# 19. Journalctl y Administración de Logs en Linux

Los registros o **logs** son una de las principales fuentes de información para diagnosticar problemas en Linux. Permiten conocer qué ocurrió en el sistema, cuándo ocurrió, qué usuario o servicio estuvo involucrado y cuál fue el error generado.

En sistemas que utilizan **systemd**, la herramienta principal para consultar estos registros es `journalctl`.

`journalctl` permite visualizar los eventos recopilados por el servicio **systemd-journald**, incluyendo mensajes del kernel, servicios, autenticación, procesos de arranque y errores del sistema.

---

# Objetivos de este capítulo

Al finalizar esta unidad serás capaz de:

* Comprender cómo funciona `systemd-journald`.
* Consultar registros con `journalctl`.
* Filtrar logs por servicio, fecha, prioridad y proceso.
* Visualizar registros en tiempo real.
* Analizar eventos de arranque.
* Configurar registros persistentes.
* Administrar el espacio utilizado por el Journal.
* Exportar registros para análisis.
* Aplicar buenas prácticas de diagnóstico.

---

# ¿Qué son los logs?

Los logs son registros generados por:

* El kernel de Linux.
* Servicios del sistema.
* Aplicaciones.
* Usuarios.
* Procesos de autenticación.
* Tareas programadas.
* Dispositivos de hardware.
* Firewalls y herramientas de seguridad.

Ejemplos de información registrada:

```text
Servicio iniciado correctamente
```

```text
Intento fallido de autenticación
```

```text
Disco sin espacio disponible
```

```text
Error al conectar con una base de datos
```

```text
Proceso finalizado inesperadamente
```

---

# ¿Qué es systemd-journald?

`systemd-journald` es el servicio responsable de recopilar y almacenar los mensajes del sistema.

Ver su estado:

```bash
systemctl status systemd-journald
```

Verificar si está activo:

```bash
systemctl is-active systemd-journald
```

Resultado esperado:

```text
active
```

---

# Fuentes de información del Journal

`systemd-journald` recopila mensajes procedentes de:

* Kernel.
* Servicios de systemd.
* Salida estándar de procesos.
* Salida de error de procesos.
* Syslog.
* Auditoría.
* Aplicaciones.

---

# Consultar todos los registros

```bash
journalctl
```

Este comando muestra los registros desde el más antiguo hasta el más reciente.

Utiliza las teclas:

```text
Flecha arriba
Flecha abajo
Page Up
Page Down
```

Salir:

```text
q
```

---

# Ver los registros más recientes

Mostrar los últimos 10 registros:

```bash
journalctl -n 10
```

Mostrar los últimos 50:

```bash
journalctl -n 50
```

Mostrar los últimos 100:

```bash
journalctl -n 100
```

También puede utilizarse:

```bash
journalctl --lines=100
```

---

# Seguir logs en tiempo real

```bash
journalctl -f
```

La opción `-f` significa **follow**.

Funciona de forma similar a:

```bash
tail -f archivo.log
```

Detener la visualización:

```text
Ctrl + C
```

---

# Ver registros de un servicio

Sintaxis:

```bash
journalctl -u nombre-servicio
```

Ejemplo con SSH:

```bash
journalctl -u sshd
```

Ejemplo con PostgreSQL:

```bash
journalctl -u postgresql
```

En Fedora o RHEL puede existir un nombre con versión:

```bash
journalctl -u postgresql-17
```

SQL Server:

```bash
journalctl -u mssql-server
```

Servidor web Apache:

```bash
journalctl -u httpd
```

---

# Ver los últimos registros de un servicio

```bash
journalctl -u sshd -n 50
```

SQL Server:

```bash
journalctl -u mssql-server -n 100
```

PostgreSQL:

```bash
journalctl -u postgresql-17 -n 100
```

---

# Seguir un servicio en tiempo real

```bash
journalctl -u sshd -f
```

SQL Server:

```bash
journalctl -u mssql-server -f
```

PostgreSQL:

```bash
journalctl -u postgresql-17 -f
```

Esta opción es muy útil mientras se reproduce un error.

---

# Filtrar por fecha

## Registros de hoy

```bash
journalctl --since today
```

## Registros de ayer

```bash
journalctl --since yesterday
```

## Desde una fecha específica

```bash
journalctl --since "2026-07-27"
```

## Desde una fecha y hora

```bash
journalctl --since "2026-07-27 08:00:00"
```

## Hasta una fecha determinada

```bash
journalctl --until "2026-07-27 17:00:00"
```

---

# Filtrar por rango de fechas

```bash
journalctl \
  --since "2026-07-27 08:00:00" \
  --until "2026-07-27 17:00:00"
```

Aplicado a un servicio:

```bash
journalctl -u mssql-server \
  --since "2026-07-27 08:00:00" \
  --until "2026-07-27 17:00:00"
```

---

# Utilizar fechas relativas

Última hora:

```bash
journalctl --since "1 hour ago"
```

Últimos 30 minutos:

```bash
journalctl --since "30 minutes ago"
```

Últimas 24 horas:

```bash
journalctl --since "24 hours ago"
```

Últimos 7 días:

```bash
journalctl --since "7 days ago"
```

---

# Filtrar por prioridad

Los mensajes del Journal se clasifican por niveles de prioridad.

| Número | Prioridad | Descripción |
| -----: | --------- | ----------- |
|      0 | emerg     | Emergencia  |
|      1 | alert     | Alerta      |
|      2 | crit      | Crítico     |
|      3 | err       | Error       |
|      4 | warning   | Advertencia |
|      5 | notice    | Aviso       |
|      6 | info      | Información |
|      7 | debug     | Depuración  |

---

# Ver solamente errores

```bash
journalctl -p err
```

También puede utilizarse el número:

```bash
journalctl -p 3
```

---

# Ver advertencias y errores

```bash
journalctl -p warning
```

Este comando incluye las prioridades desde `emerg` hasta `warning`.

---

# Ver errores de un servicio

```bash
journalctl -u sshd -p err
```

SQL Server:

```bash
journalctl -u mssql-server -p err
```

PostgreSQL:

```bash
journalctl -u postgresql-17 -p err
```

---

# Ver registros del kernel

```bash
journalctl -k
```

También:

```bash
journalctl --dmesg
```

Últimos mensajes del kernel:

```bash
journalctl -k -n 50
```

Errores del kernel:

```bash
journalctl -k -p err
```

---

# Ver registros del arranque actual

```bash
journalctl -b
```

La opción `-b` representa el arranque actual.

Últimos registros del arranque:

```bash
journalctl -b -n 100
```

Errores del arranque actual:

```bash
journalctl -b -p err
```

---

# Listar arranques disponibles

```bash
journalctl --list-boots
```

Ejemplo:

```text
-2  4b19e7... Sun 2026-07-25 08:00:01 -04—Sun 2026-07-25 22:30:10 -04
-1  9c42a1... Mon 2026-07-26 07:55:03 -04—Mon 2026-07-26 21:18:40 -04
 0  f10a5d... Tue 2026-07-27 08:02:20 -04—Tue 2026-07-27 16:30:00 -04
```

El valor `0` representa el arranque actual.

---

# Ver el arranque anterior

```bash
journalctl -b -1
```

Ver dos arranques atrás:

```bash
journalctl -b -2
```

Errores del arranque anterior:

```bash
journalctl -b -1 -p err
```

---

# Filtrar por proceso

Utilizando PID:

```bash
journalctl _PID=2541
```

Encontrar el PID:

```bash
pgrep -a sshd
```

Luego:

```bash
journalctl _PID=PID
```

---

# Filtrar por usuario

Utilizando UID:

```bash
journalctl _UID=1000
```

Obtener el UID:

```bash
id -u alejandro
```

También:

```bash
journalctl _UID=$(id -u alejandro)
```

---

# Filtrar por comando

```bash
journalctl _COMM=sshd
```

PostgreSQL:

```bash
journalctl _COMM=postgres
```

SQL Server:

```bash
journalctl _COMM=sqlservr
```

---

# Filtrar por archivo ejecutable

```bash
journalctl /usr/sbin/sshd
```

Otro ejemplo:

```bash
journalctl /usr/bin/sudo
```

---

# Filtrar por identificador Syslog

```bash
journalctl SYSLOG_IDENTIFIER=sudo
```

Ejemplo:

```bash
journalctl SYSLOG_IDENTIFIER=sshd
```

---

# Combinar filtros

Ver errores de SQL Server durante la última hora:

```bash
journalctl -u mssql-server \
  --since "1 hour ago" \
  -p err
```

Ver los últimos 100 mensajes de PostgreSQL desde hoy:

```bash
journalctl -u postgresql-17 \
  --since today \
  -n 100
```

Ver mensajes del kernel relacionados con errores:

```bash
journalctl -k -p err --since today
```

---

# Buscar texto dentro de los logs

Con `grep`:

```bash
journalctl -u sshd | grep -i error
```

Buscar conexiones fallidas:

```bash
journalctl -u sshd | grep -i failed
```

Buscar mensajes de memoria:

```bash
journalctl -k | grep -i memory
```

Buscar eventos relacionados con discos:

```bash
journalctl -k | grep -Ei "disk|I/O|nvme|sda"
```

---

# Buscar directamente con grep integrado

En versiones modernas puede utilizarse `--grep`:

```bash
journalctl --grep="error"
```

Búsqueda sin distinguir mayúsculas:

```bash
journalctl --grep="failed" --case-sensitive=no
```

Buscar en un servicio:

```bash
journalctl -u sshd --grep="failed"
```

---

# Formatos de salida

## Formato corto

```bash
journalctl -o short
```

## Fecha completa

```bash
journalctl -o short-full
```

## Fecha ISO

```bash
journalctl -o short-iso
```

## Formato detallado

```bash
journalctl -o verbose
```

## Solo el mensaje

```bash
journalctl -o cat
```

## Formato JSON

```bash
journalctl -o json
```

## JSON legible

```bash
journalctl -o json-pretty
```

---

# Mostrar logs sin paginador

Por defecto, `journalctl` puede abrir la salida con `less`.

Para mostrar todo directamente:

```bash
journalctl --no-pager
```

Ejemplo:

```bash
journalctl -u sshd -n 50 --no-pager
```

---

# Mostrar fechas locales

```bash
journalctl --local
```

Ver registros en UTC:

```bash
journalctl --utc
```

---

# Mostrar los registros en orden inverso

Los más recientes primero:

```bash
journalctl -r
```

También:

```bash
journalctl --reverse
```

Ejemplo:

```bash
journalctl -u mssql-server -r -n 50
```

---

# Ver el espacio utilizado por el Journal

```bash
journalctl --disk-usage
```

Ejemplo:

```text
Archived and active journals take up 1.2G in the file system.
```

---

# Limpiar logs antiguos por tiempo

Eliminar registros de más de 7 días:

```bash
sudo journalctl --vacuum-time=7d
```

Mantener solo los últimos 30 días:

```bash
sudo journalctl --vacuum-time=30d
```

Mantener solo el último año:

```bash
sudo journalctl --vacuum-time=1year
```

---

# Limpiar logs por tamaño

Mantener un máximo de 500 MB:

```bash
sudo journalctl --vacuum-size=500M
```

Mantener un máximo de 1 GB:

```bash
sudo journalctl --vacuum-size=1G
```

---

# Limpiar por cantidad de archivos

```bash
sudo journalctl --vacuum-files=10
```

Esto mantiene un máximo de 10 archivos archivados del Journal.

---

# Rotar el Journal antes de limpiar

En algunos casos es recomendable rotar primero:

```bash
sudo journalctl --rotate
```

Luego limpiar:

```bash
sudo journalctl --vacuum-time=7d
```

Combinación:

```bash
sudo journalctl --rotate
sudo journalctl --vacuum-size=500M
```

---

# Configuración de journald

Archivo principal:

```text
/etc/systemd/journald.conf
```

Editar:

```bash
sudo nano /etc/systemd/journald.conf
```

Después de modificarlo:

```bash
sudo systemctl restart systemd-journald
```

---

# Opciones importantes de journald.conf

Ejemplo:

```ini
[Journal]
Storage=persistent
SystemMaxUse=1G
SystemKeepFree=500M
SystemMaxFileSize=100M
MaxRetentionSec=30day
Compress=yes
```

---

# Storage

La opción `Storage` determina dónde se almacenan los registros.

Valores disponibles:

| Valor        | Descripción                                  |
| ------------ | -------------------------------------------- |
| `volatile`   | Guarda logs únicamente en memoria            |
| `persistent` | Guarda logs en disco                         |
| `auto`       | Utiliza persistencia si existe el directorio |
| `none`       | No almacena logs                             |

Configuración recomendada para persistencia:

```ini
Storage=persistent
```

---

# Activar registros persistentes

Crear el directorio:

```bash
sudo mkdir -p /var/log/journal
```

Asignar permisos correctos:

```bash
sudo systemd-tmpfiles --create --prefix /var/log/journal
```

Reiniciar journald:

```bash
sudo systemctl restart systemd-journald
```

Verificar:

```bash
ls -ld /var/log/journal
```

---

# Ver configuración efectiva

```bash
systemd-analyze cat-config systemd/journald.conf
```

También puede verificarse el archivo manualmente:

```bash
grep -Ev '^\s*(#|$)' /etc/systemd/journald.conf
```

---

# Exportar registros a un archivo

Guardar los últimos 100 mensajes:

```bash
journalctl -n 100 > registros.txt
```

Guardar logs de un servicio:

```bash
journalctl -u sshd > sshd.log
```

Guardar errores desde hoy:

```bash
journalctl --since today -p err > errores_hoy.log
```

---

# Exportar sin formato adicional

```bash
journalctl -u mssql-server -o cat > mssql-server.log
```

---

# Exportar en formato JSON

```bash
journalctl -u sshd -o json-pretty > sshd.json
```

---

# Comprimir registros exportados

```bash
journalctl --since "7 days ago" > journal_7_dias.log
```

Comprimir:

```bash
gzip journal_7_dias.log
```

Resultado:

```text
journal_7_dias.log.gz
```

---

# Ver autenticaciones con sudo

```bash
journalctl _COMM=sudo
```

También:

```bash
journalctl SYSLOG_IDENTIFIER=sudo
```

Buscar comandos ejecutados con sudo:

```bash
journalctl _COMM=sudo --since today
```

---

# Ver intentos fallidos de SSH

```bash
journalctl -u sshd | grep -i failed
```

Desde hoy:

```bash
journalctl -u sshd --since today | grep -i failed
```

Utilizando `--grep`:

```bash
journalctl -u sshd --since today --grep="Failed password"
```

---

# Ver reinicios y apagados

Reinicios:

```bash
journalctl --list-boots
```

Eventos de apagado:

```bash
journalctl | grep -i shutdown
```

Eventos de reinicio:

```bash
journalctl | grep -i reboot
```

También puede utilizarse:

```bash
last reboot
```

---

# Ver errores de un servicio fallido

Primero:

```bash
systemctl status nombre-servicio
```

Luego:

```bash
journalctl -u nombre-servicio -n 100
```

Con información adicional:

```bash
journalctl -xeu nombre-servicio
```

Ejemplo:

```bash
journalctl -xeu mssql-server
```

---

# Significado de journalctl -xeu

```text
-x
```

Agrega explicaciones adicionales cuando están disponibles.

```text
-e
```

Muestra el final de los registros.

```text
-u
```

Filtra por una unidad o servicio.

Ejemplo completo:

```bash
journalctl -xeu postgresql-17
```

---

# Logs tradicionales en /var/log

Aunque systemd utiliza el Journal, muchas aplicaciones todavía escriben en archivos tradicionales.

Directorio principal:

```text
/var/log
```

Listar contenido:

```bash
ls -lah /var/log
```

---

# Archivos comunes de logs

| Archivo                    | Descripción                           |
| -------------------------- | ------------------------------------- |
| `/var/log/messages`        | Mensajes generales en RHEL y Fedora   |
| `/var/log/syslog`          | Mensajes generales en Debian y Ubuntu |
| `/var/log/secure`          | Autenticación en RHEL y Fedora        |
| `/var/log/auth.log`        | Autenticación en Debian y Ubuntu      |
| `/var/log/cron`            | Eventos de cron                       |
| `/var/log/dnf.log`         | Actividad de DNF                      |
| `/var/log/apt/`            | Actividad de APT                      |
| `/var/log/audit/audit.log` | Eventos de auditoría                  |
| `/var/log/boot.log`        | Mensajes de arranque                  |
| `/var/log/wtmp`            | Historial de sesiones                 |
| `/var/log/btmp`            | Intentos fallidos de inicio           |

---

# Consultar logs tradicionales

Últimas líneas:

```bash
tail -n 50 /var/log/messages
```

En Ubuntu:

```bash
tail -n 50 /var/log/syslog
```

Seguir en tiempo real:

```bash
tail -f /var/log/messages
```

Buscar errores:

```bash
grep -i error /var/log/messages
```

---

# Ver logs de cron

Fedora o RHEL:

```bash
journalctl -u crond
```

Últimos 100 eventos:

```bash
journalctl -u crond -n 100
```

Desde hoy:

```bash
journalctl -u crond --since today
```

Seguir en tiempo real:

```bash
journalctl -u crond -f
```

En algunos sistemas también existe:

```bash
sudo tail -f /var/log/cron
```

---

# Ver ejecuciones específicas de cron

```bash
journalctl -u crond --since today | grep CMD
```

Buscar un script:

```bash
journalctl -u crond | grep mantenimiento.sh
```

Buscar ejecuciones de root:

```bash
journalctl -u crond | grep "(root)"
```

---

# Rotación de logs con logrotate

Los logs tradicionales pueden crecer rápidamente.

Linux utiliza `logrotate` para:

* Rotar archivos.
* Comprimir registros antiguos.
* Eliminar archivos vencidos.
* Mantener un número limitado de copias.

Archivo principal:

```text
/etc/logrotate.conf
```

Configuraciones adicionales:

```text
/etc/logrotate.d/
```

Ver configuraciones:

```bash
ls -l /etc/logrotate.d/
```

---

# Ejemplo de configuración logrotate

```text
/var/log/miaplicacion.log {
    daily
    rotate 7
    compress
    missingok
    notifempty
}
```

Significado:

| Opción       | Descripción                       |
| ------------ | --------------------------------- |
| `daily`      | Rotar diariamente                 |
| `rotate 7`   | Mantener siete copias             |
| `compress`   | Comprimir logs antiguos           |
| `missingok`  | No fallar si el archivo no existe |
| `notifempty` | No rotar archivos vacíos          |

---

# Probar logrotate

Modo de depuración:

```bash
sudo logrotate -d /etc/logrotate.conf
```

Forzar rotación:

```bash
sudo logrotate -f /etc/logrotate.conf
```

Debe utilizarse con cuidado en servidores de producción.

---

# Comandos más utilizados

| Comando                    | Descripción                  |
| -------------------------- | ---------------------------- |
| `journalctl`               | Ver todos los registros      |
| `journalctl -f`            | Seguir logs en tiempo real   |
| `journalctl -u`            | Filtrar por servicio         |
| `journalctl -b`            | Ver logs del arranque        |
| `journalctl -k`            | Ver mensajes del kernel      |
| `journalctl -p`            | Filtrar por prioridad        |
| `journalctl --since`       | Filtrar desde una fecha      |
| `journalctl --until`       | Filtrar hasta una fecha      |
| `journalctl --disk-usage`  | Ver espacio utilizado        |
| `journalctl --vacuum-time` | Limpiar por antigüedad       |
| `journalctl --vacuum-size` | Limpiar por tamaño           |
| `journalctl --list-boots`  | Listar arranques             |
| `tail -f`                  | Seguir un archivo de log     |
| `grep`                     | Buscar texto en logs         |
| `logrotate`                | Administrar rotación de logs |

---

# Buenas prácticas

* Consulta los logs antes de reiniciar un servicio.
* Filtra por servicio, fecha y prioridad para reducir la cantidad de información.
* Utiliza `journalctl -xeu servicio` para diagnosticar servicios fallidos.
* Habilita persistencia cuando necesites conservar registros después de reinicios.
* Controla regularmente el espacio utilizado con `journalctl --disk-usage`.
* No elimines manualmente archivos del Journal.
* Utiliza las opciones `--vacuum-time` y `--vacuum-size`.
* Exporta los registros importantes antes de limpiarlos.
* Protege los logs, ya que pueden contener información sensible.
* Configura rotación y retención según las necesidades del servidor.
* Sincroniza correctamente la hora del sistema para facilitar el análisis.

---

# Laboratorio práctico

## Ejercicio 1: Ver registros recientes

```bash
journalctl -n 50
```

---

## Ejercicio 2: Ver registros del arranque actual

```bash
journalctl -b
```

Mostrar solamente errores:

```bash
journalctl -b -p err
```

---

## Ejercicio 3: Analizar un servicio

```bash
systemctl status sshd
```

Luego:

```bash
journalctl -u sshd -n 50
```

---

## Ejercicio 4: Seguir logs en tiempo real

```bash
sudo journalctl -u sshd -f
```

Desde otra terminal, realiza una conexión SSH para observar los eventos.

---

## Ejercicio 5: Filtrar por fecha

```bash
journalctl \
  --since "1 hour ago" \
  --until now
```

---

## Ejercicio 6: Ver errores del sistema

```bash
journalctl -p err --since today
```

---

## Ejercicio 7: Consultar espacio utilizado

```bash
journalctl --disk-usage
```

---

## Ejercicio 8: Limpiar logs antiguos

Primero rotar:

```bash
sudo journalctl --rotate
```

Mantener solamente 30 días:

```bash
sudo journalctl --vacuum-time=30d
```

---

## Ejercicio 9: Exportar registros

```bash
journalctl -u sshd --since today > sshd_hoy.log
```

Verificar:

```bash
ls -lh sshd_hoy.log
```

---

## Ejercicio 10: Consultar el arranque anterior

```bash
journalctl --list-boots
```

Luego:

```bash
journalctl -b -1 -p err
```

---

# Diagnóstico práctico de servicios

Cuando un servicio no inicia, sigue este orden:

## Paso 1: Consultar su estado

```bash
systemctl status nombre-servicio
```

## Paso 2: Consultar sus logs recientes

```bash
journalctl -u nombre-servicio -n 100
```

## Paso 3: Ver errores detallados

```bash
journalctl -xeu nombre-servicio
```

## Paso 4: Revisar la configuración

```bash
systemctl cat nombre-servicio
```

## Paso 5: Corregir y reiniciar

```bash
sudo systemctl restart nombre-servicio
```

## Paso 6: Confirmar el resultado

```bash
systemctl is-active nombre-servicio
```

---

# Errores comunes

## No aparecen registros de arranques anteriores

Posible causa:

```text
El Journal está configurado como volatile.
```

Solución:

```bash
sudo mkdir -p /var/log/journal
sudo systemd-tmpfiles --create --prefix /var/log/journal
sudo systemctl restart systemd-journald
```

---

## El Journal ocupa demasiado espacio

Verificar:

```bash
journalctl --disk-usage
```

Limpiar:

```bash
sudo journalctl --rotate
sudo journalctl --vacuum-size=500M
```

También puede configurarse:

```ini
SystemMaxUse=500M
```

en:

```text
/etc/systemd/journald.conf
```

---

## No se encuentran logs de un servicio

Verifica el nombre correcto:

```bash
systemctl list-units --type=service | grep -i postgres
```

Luego utiliza el nombre exacto:

```bash
journalctl -u postgresql-17.service
```

---

## journalctl muestra demasiada información

Aplica varios filtros:

```bash
journalctl -u mssql-server \
  --since "1 hour ago" \
  -p warning \
  -n 100
```

---

## El comando no muestra colores ni formato

Cuando la salida se redirige a un archivo, `journalctl` modifica el formato.

Puedes elegir uno explícitamente:

```bash
journalctl -o short-iso
```

---

## El servicio falla y los logs desaparecen al reiniciar

Habilita el almacenamiento persistente:

```ini
Storage=persistent
```

Después:

```bash
sudo systemctl restart systemd-journald
```

---

# Resumen

En este capítulo aprendiste a:

* Comprender el funcionamiento de `systemd-journald`.
* Consultar registros con `journalctl`.
* Filtrar logs por servicio, proceso, usuario, fecha y prioridad.
* Analizar eventos del kernel y del arranque.
* Consultar arranques anteriores.
* Seguir registros en tiempo real.
* Exportar logs para investigaciones y soporte.
* Configurar registros persistentes.
* Controlar el espacio utilizado por el Journal.
* Administrar logs tradicionales mediante `tail`, `grep` y `logrotate`.
* Diagnosticar servicios fallidos de forma organizada.
* Aplicar buenas prácticas para conservar registros útiles sin llenar el almacenamiento.

Esta página puede guardarse como `19-journalctl-logs.md`.
