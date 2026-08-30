Aquí tienes el capítulo completo para guardarlo como `20-cron-at.md`:

# 20. Programación de Tareas con Cron y At

Linux permite automatizar tareas mediante herramientas de planificación. Las dos más utilizadas son:

* `cron`, para tareas repetitivas.
* `at`, para tareas que deben ejecutarse una sola vez.

Estas herramientas son fundamentales para automatizar respaldos, mantenimientos, limpieza de archivos, ejecución de scripts, generación de reportes, sincronizaciones y tareas administrativas.

---

# Objetivos de este capítulo

Al finalizar esta unidad serás capaz de:

* Comprender cómo funciona `cron`.
* Crear tareas programadas con `crontab`.
* Interpretar la sintaxis de una expresión cron.
* Administrar tareas de usuarios y del sistema.
* Consultar ejecuciones mediante logs.
* Configurar tareas de una sola ejecución con `at`.
* Utilizar `anacron` para tareas periódicas.
* Evitar errores frecuentes en scripts automatizados.
* Aplicar buenas prácticas de seguridad y mantenimiento.

---

# ¿Qué es cron?

`cron` es un servicio de Linux que ejecuta comandos o scripts automáticamente según una programación definida.

El servicio normalmente se llama:

```text
crond
```

en Fedora, RHEL, Rocky Linux y AlmaLinux.

En Debian y Ubuntu suele llamarse:

```text
cron
```

---

# Verificar el servicio cron

## Fedora, RHEL, Rocky Linux y AlmaLinux

```bash
systemctl status crond
```

Verificar si está activo:

```bash
systemctl is-active crond
```

Verificar si inicia automáticamente:

```bash
systemctl is-enabled crond
```

---

## Ubuntu y Debian

```bash
systemctl status cron
```

---

# Iniciar cron

Fedora y RHEL:

```bash
sudo systemctl start crond
```

Ubuntu y Debian:

```bash
sudo systemctl start cron
```

---

# Habilitar cron al iniciar

Fedora y RHEL:

```bash
sudo systemctl enable --now crond
```

Ubuntu y Debian:

```bash
sudo systemctl enable --now cron
```

---

# ¿Qué es crontab?

`crontab` es el comando utilizado para administrar las tareas programadas de un usuario.

Cada usuario puede tener su propio archivo crontab.

Editar el crontab actual:

```bash
crontab -e
```

Listar las tareas:

```bash
crontab -l
```

Eliminar todas las tareas:

```bash
crontab -r
```

Debe utilizarse con mucho cuidado porque elimina el crontab completo sin editarlo.

---

# Editar el crontab de otro usuario

Como root:

```bash
sudo crontab -u usuario -e
```

Ejemplo:

```bash
sudo crontab -u postgres -e
```

Listar el crontab de otro usuario:

```bash
sudo crontab -u postgres -l
```

---

# Sintaxis de cron

Una entrada de cron está formada por cinco campos de tiempo y el comando que se ejecutará.

```text
┌──────────── minuto (0-59)
│ ┌────────── hora (0-23)
│ │ ┌──────── día del mes (1-31)
│ │ │ ┌────── mes (1-12)
│ │ │ │ ┌──── día de la semana (0-7)
│ │ │ │ │
* * * * * comando
```

Los valores `0` y `7` representan el domingo.

---

# Campos de cron

| Campo            | Valores permitidos                |
| ---------------- | --------------------------------- |
| Minuto           | 0-59                              |
| Hora             | 0-23                              |
| Día del mes      | 1-31                              |
| Mes              | 1-12                              |
| Día de la semana | 0-7                               |
| Comando          | Comando o script que se ejecutará |

---

# Ejemplo básico

Ejecutar un script todos los días a las 2:00 a. m.:

```cron
0 2 * * * /opt/scripts/respaldo.sh
```

Interpretación:

```text
Minuto: 0
Hora: 2
Día del mes: cualquiera
Mes: cualquiera
Día de la semana: cualquiera
```

---

# Caracteres especiales

## Asterisco

El asterisco representa cualquier valor.

```cron
* * * * * comando
```

Este comando se ejecutaría cada minuto.

---

## Coma

Permite especificar varios valores.

```cron
0 8,12,18 * * * comando
```

Se ejecuta a las:

```text
08:00
12:00
18:00
```

---

## Guion

Permite definir un rango.

```cron
0 8 * * 1-5 comando
```

Se ejecuta de lunes a viernes a las 8:00 a. m.

---

## Barra inclinada

Permite establecer intervalos.

```cron
*/15 * * * * comando
```

Se ejecuta cada 15 minutos.

---

# Ejemplos comunes de cron

## Cada minuto

```cron
* * * * * /opt/scripts/proceso.sh
```

---

## Cada cinco minutos

```cron
*/5 * * * * /opt/scripts/proceso.sh
```

---

## Cada quince minutos

```cron
*/15 * * * * /opt/scripts/proceso.sh
```

---

## Cada hora

```cron
0 * * * * /opt/scripts/proceso.sh
```

---

## Cada dos horas

```cron
0 */2 * * * /opt/scripts/proceso.sh
```

---

## Todos los días a medianoche

```cron
0 0 * * * /opt/scripts/respaldo.sh
```

---

## Todos los días a las 11:00 p. m.

```cron
0 23 * * * /opt/scripts/mantenimiento.sh
```

---

## Todos los lunes

```cron
0 8 * * 1 /opt/scripts/reporte.sh
```

---

## De lunes a viernes

```cron
0 8 * * 1-5 /opt/scripts/reporte.sh
```

---

## Todos los domingos

```cron
0 3 * * 0 /opt/scripts/respaldo_completo.sh
```

También puede utilizarse:

```cron
0 3 * * 7 /opt/scripts/respaldo_completo.sh
```

---

## El primer día de cada mes

```cron
0 0 1 * * /opt/scripts/reporte_mensual.sh
```

---

## Cada 1 de enero

```cron
0 0 1 1 * /opt/scripts/reporte_anual.sh
```

---

## Los días 1 y 15 de cada mes

```cron
0 6 1,15 * * /opt/scripts/proceso.sh
```

---

## Durante una franja horaria

Cada 10 minutos entre las 8:00 a. m. y las 5:59 p. m.:

```cron
*/10 8-17 * * * /opt/scripts/proceso.sh
```

---

# Palabras especiales de cron

Cron admite expresiones especiales.

| Expresión   | Significado           |
| ----------- | --------------------- |
| `@reboot`   | Al iniciar el sistema |
| `@yearly`   | Una vez al año        |
| `@annually` | Una vez al año        |
| `@monthly`  | Una vez al mes        |
| `@weekly`   | Una vez por semana    |
| `@daily`    | Una vez al día        |
| `@midnight` | A medianoche          |
| `@hourly`   | Una vez por hora      |

---

# Ejecutar al iniciar el sistema

```cron
@reboot /opt/scripts/iniciar_aplicacion.sh
```

Es recomendable utilizar una ruta absoluta y redirigir la salida.

```cron
@reboot /opt/scripts/iniciar_aplicacion.sh >> /var/log/iniciar_aplicacion.log 2>&1
```

Para servicios de larga duración, normalmente es mejor utilizar `systemd`.

---

# Variables dentro de crontab

Es posible definir variables al inicio del archivo.

```cron
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
MAILTO=administrador@example.com
```

Ejemplo completo:

```cron
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

0 2 * * * /opt/scripts/respaldo.sh
```

---

# Importancia de PATH en cron

Cron utiliza un entorno más limitado que una terminal interactiva.

Un comando que funciona manualmente puede fallar desde cron si no se especifica su ruta completa.

En lugar de:

```cron
0 2 * * * python script.py
```

Es preferible:

```cron
0 2 * * * /usr/bin/python3 /opt/scripts/script.py
```

Encontrar la ruta de un comando:

```bash
which python3
```

También:

```bash
command -v python3
```

---

# Utilizar rutas absolutas

Incorrecto:

```cron
0 2 * * * ./respaldo.sh
```

Correcto:

```cron
0 2 * * * /opt/scripts/respaldo.sh
```

Dentro del script también deben utilizarse rutas absolutas cuando sea posible.

---

# Permisos del script

El script debe tener permisos de ejecución:

```bash
chmod +x /opt/scripts/respaldo.sh
```

Verificar:

```bash
ls -l /opt/scripts/respaldo.sh
```

Ejemplo:

```text
-rwxr-xr-x. 1 root root 1250 Jul 27 10:00 respaldo.sh
```

---

# Shebang

Todo script debe comenzar con el intérprete correcto.

Bash:

```bash
#!/bin/bash
```

Python:

```python
#!/usr/bin/env python3
```

Ejemplo:

```bash
#!/bin/bash

date
echo "Ejecutando respaldo"
```

---

# Redirección de salida

Cron puede generar salida estándar y errores.

Guardar la salida:

```cron
0 2 * * * /opt/scripts/respaldo.sh >> /var/log/respaldo.log
```

Guardar salida y errores:

```cron
0 2 * * * /opt/scripts/respaldo.sh >> /var/log/respaldo.log 2>&1
```

---

# Significado de 2>&1

```text
1 = salida estándar
2 = salida de error
```

La expresión:

```bash
2>&1
```

envía los errores al mismo destino que la salida estándar.

---

# Descartar toda la salida

```cron
0 2 * * * /opt/scripts/proceso.sh > /dev/null 2>&1
```

Esto debe utilizarse solamente cuando no se necesiten registros.

Para tareas importantes es preferible conservar un log.

---

# Generar logs con fecha

Desde el crontab:

```cron
0 2 * * * /opt/scripts/respaldo.sh >> /var/log/respaldo_$(date +\%Y\%m\%d).log 2>&1
```

En un crontab, el carácter `%` debe escaparse:

```text
\%
```

Por esta razón suele ser más seguro manejar el nombre del log dentro del script.

Ejemplo:

```bash
#!/bin/bash

FECHA=$(date +%Y%m%d_%H%M%S)
LOG="/var/log/respaldo_${FECHA}.log"

/opt/scripts/realizar_respaldo.sh >> "$LOG" 2>&1
```

---

# Directorios de cron del sistema

Linux incluye directorios para ejecutar tareas según una frecuencia determinada.

```text
/etc/cron.hourly/
/etc/cron.daily/
/etc/cron.weekly/
/etc/cron.monthly/
```

---

# Ver tareas del sistema

```bash
ls -l /etc/cron.hourly/
```

```bash
ls -l /etc/cron.daily/
```

```bash
ls -l /etc/cron.weekly/
```

```bash
ls -l /etc/cron.monthly/
```

---

# Archivo /etc/crontab

El archivo global del sistema es:

```text
/etc/crontab
```

Ver su contenido:

```bash
cat /etc/crontab
```

Su formato incluye un campo adicional para el usuario:

```text
minuto hora día mes semana usuario comando
```

Ejemplo:

```cron
0 2 * * * root /opt/scripts/respaldo.sh
```

---

# Diferencia entre crontab de usuario y /etc/crontab

En el crontab de un usuario:

```cron
0 2 * * * /opt/scripts/respaldo.sh
```

En `/etc/crontab`:

```cron
0 2 * * * root /opt/scripts/respaldo.sh
```

El archivo `/etc/crontab` requiere especificar el usuario que ejecutará la tarea.

---

# Directorio /etc/cron.d

Las aplicaciones y administradores pueden crear archivos individuales en:

```text
/etc/cron.d/
```

Ejemplo:

```bash
sudo nano /etc/cron.d/respaldo
```

Contenido:

```cron
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

0 2 * * * root /opt/scripts/respaldo.sh >> /var/log/respaldo.log 2>&1
```

Asignar permisos adecuados:

```bash
sudo chmod 644 /etc/cron.d/respaldo
```

---

# Archivos donde se guardan los crontabs

En Fedora y RHEL:

```text
/var/spool/cron/
```

Ejemplo:

```text
/var/spool/cron/root
```

En algunas distribuciones Debian:

```text
/var/spool/cron/crontabs/
```

Estos archivos no deben editarse manualmente.

Debe utilizarse:

```bash
crontab -e
```

---

# Controlar quién puede usar cron

Los archivos relacionados son:

```text
/etc/cron.allow
/etc/cron.deny
```

Si existe `/etc/cron.allow`, solamente los usuarios incluidos pueden utilizar cron.

Ejemplo:

```bash
sudo nano /etc/cron.allow
```

Contenido:

```text
alejandro
postgres
```

---

# Bloquear usuarios

Agregar un usuario a:

```text
/etc/cron.deny
```

Ejemplo:

```bash
echo "invitado" | sudo tee -a /etc/cron.deny
```

---

# Ver logs de cron

## Fedora y RHEL

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

En tiempo real:

```bash
journalctl -u crond -f
```

---

## Ubuntu y Debian

```bash
journalctl -u cron
```

También puede revisarse:

```bash
grep CRON /var/log/syslog
```

---

# Ver comandos ejecutados por cron

Fedora y RHEL:

```bash
journalctl -u crond --since today | grep CMD
```

Ver inicio y finalización:

```bash
journalctl -u crond --since today | grep -E "CMD|CMDEND"
```

Buscar un script:

```bash
journalctl -u crond | grep respaldo.sh
```

Buscar ejecuciones de root:

```bash
journalctl -u crond | grep "(root)"
```

---

# Ejemplo de log de cron

```text
Jul 27 02:00:01 servidor CROND[5124]: (root) CMD (/opt/scripts/respaldo.sh)
Jul 27 02:10:12 servidor CROND[5123]: (root) CMDEND (/opt/scripts/respaldo.sh)
```

Esto confirma que la tarea comenzó y terminó.

Sin embargo, no garantiza que el script haya realizado correctamente todas sus operaciones internas.

Por ello es importante que el script genere su propio log y código de salida.

---

# Validar si un script terminó correctamente

Ejemplo:

```bash
#!/bin/bash

LOG="/var/log/respaldo.log"

echo "Inicio: $(date)" >> "$LOG"

/opt/scripts/realizar_respaldo.sh >> "$LOG" 2>&1
CODIGO=$?

if [ "$CODIGO" -eq 0 ]; then
    echo "Resultado: correcto" >> "$LOG"
else
    echo "Resultado: error. Código: $CODIGO" >> "$LOG"
fi

echo "Fin: $(date)" >> "$LOG"
exit "$CODIGO"
```

---

# Evitar ejecuciones simultáneas

Una tarea puede comenzar nuevamente antes de que termine la ejecución anterior.

Para evitarlo puede utilizarse `flock`.

Ejemplo:

```cron
*/15 * * * * /usr/bin/flock -n /run/proceso.lock /opt/scripts/proceso.sh
```

La opción `-n` evita esperar si el bloqueo ya está ocupado.

---

# Ejemplo con archivo de bloqueo

```cron
0 2 * * * /usr/bin/flock -n /run/respaldo.lock /opt/scripts/respaldo.sh >> /var/log/respaldo.log 2>&1
```

Esto evita dos respaldos simultáneos.

---

# Utilizar timeout

Para evitar que una tarea permanezca bloqueada indefinidamente:

```cron
0 2 * * * /usr/bin/timeout 2h /opt/scripts/respaldo.sh
```

El proceso será detenido si supera dos horas.

---

# Combinar flock y timeout

```cron
0 2 * * * /usr/bin/flock -n /run/respaldo.lock /usr/bin/timeout 2h /opt/scripts/respaldo.sh >> /var/log/respaldo.log 2>&1
```

---

# Ejecutar con baja prioridad

Para reducir el impacto sobre el servidor:

```cron
0 2 * * * /usr/bin/nice -n 15 /usr/bin/ionice -c3 /opt/scripts/respaldo.sh
```

Combinación completa:

```cron
0 2 * * * /usr/bin/flock -n /run/respaldo.lock /usr/bin/timeout 2h /usr/bin/nice -n 15 /usr/bin/ionice -c3 /opt/scripts/respaldo.sh >> /var/log/respaldo.log 2>&1
```

---

# Probar un script antes de agregarlo a cron

Ejecutarlo manualmente con el mismo usuario:

```bash
sudo -u usuario /opt/scripts/proceso.sh
```

Ejemplo:

```bash
sudo -u postgres /opt/scripts/respaldo_postgres.sh
```

Verificar el código de salida:

```bash
echo $?
```

Un valor de:

```text
0
```

normalmente significa ejecución correcta.

---

# Simular un entorno reducido

Cron no carga el mismo entorno de la terminal.

Puede probarse con:

```bash
env -i \
HOME="$HOME" \
PATH=/usr/bin:/bin \
SHELL=/bin/bash \
/opt/scripts/proceso.sh
```

Esto ayuda a detectar dependencias de variables de entorno.

---

# Variables de entorno en scripts de base de datos

PostgreSQL puede requerir:

```bash
export PGHOST=127.0.0.1
export PGPORT=5432
export PGUSER=postgres
export PGDATABASE=basedatos
```

SQL Server puede requerir rutas completas:

```bash
/opt/mssql-tools18/bin/sqlcmd
```

No debe suponerse que cron encontrará automáticamente el comando.

---

# Correos generados por cron

Si la tarea produce salida y el sistema tiene un agente de correo configurado, cron puede enviarla al usuario.

Definir destinatario:

```cron
MAILTO=administrador@example.com
```

Deshabilitar correos:

```cron
MAILTO=""
```

Aunque se deshabilite el correo, es recomendable registrar la salida en archivos de log.

---

# ¿Qué es at?

`at` permite programar un comando para ejecutarse una sola vez en el futuro.

A diferencia de cron, no se repite automáticamente.

---

# Instalar at

## Fedora, RHEL, Rocky Linux y AlmaLinux

```bash
sudo dnf install at
```

## Ubuntu y Debian

```bash
sudo apt install at
```

---

# Servicio atd

Verificar:

```bash
systemctl status atd
```

Habilitar e iniciar:

```bash
sudo systemctl enable --now atd
```

---

# Programar una tarea con at

Ejecutar hoy a las 10:00 p. m.:

```bash
at 22:00
```

Luego escribir:

```text
/opt/scripts/respaldo.sh
```

Finalizar con:

```text
Ctrl + D
```

---

# Ejemplo completo

```bash
at 22:00
```

Dentro del prompt:

```text
/opt/scripts/respaldo.sh >> /var/log/respaldo_at.log 2>&1
```

Presionar:

```text
Ctrl + D
```

---

# Expresiones de tiempo con at

## Dentro de una hora

```bash
at now + 1 hour
```

## Dentro de 30 minutos

```bash
at now + 30 minutes
```

## Mañana

```bash
at tomorrow
```

## Mañana a las 8:00 a. m.

```bash
at 08:00 tomorrow
```

## El próximo lunes

```bash
at 09:00 next Monday
```

## Fecha específica

```bash
at 14:30 07/30/2026
```

El formato de fecha puede variar según la configuración regional del sistema.

---

# Programar at con echo

```bash
echo "/opt/scripts/respaldo.sh" | at 22:00
```

Con redirección:

```bash
echo "/opt/scripts/respaldo.sh >> /var/log/respaldo_at.log 2>&1" | at now + 1 hour
```

---

# Listar tareas pendientes de at

```bash
atq
```

También:

```bash
at -l
```

Ejemplo:

```text
3   Mon Jul 27 22:00:00 2026 a alejandro
```

---

# Ver el contenido de una tarea at

```bash
at -c 3
```

Donde `3` es el número de trabajo.

---

# Eliminar una tarea at

```bash
atrm 3
```

También:

```bash
at -r 3
```

Verificar:

```bash
atq
```

---

# Controlar quién puede usar at

Archivos:

```text
/etc/at.allow
/etc/at.deny
```

Permitir únicamente ciertos usuarios:

```bash
sudo nano /etc/at.allow
```

Contenido:

```text
alejandro
postgres
```

---

# ¿Qué es batch?

`batch` programa una tarea para ejecutarse cuando la carga del sistema sea suficientemente baja.

Ejemplo:

```bash
batch
```

Luego escribir:

```text
/opt/scripts/proceso_intensivo.sh
```

Finalizar con:

```text
Ctrl + D
```

Es útil para tareas que consumen muchos recursos y no necesitan ejecutarse a una hora exacta.

---

# ¿Qué es anacron?

`anacron` ejecuta tareas periódicas que pudieron no haberse ejecutado porque el equipo estaba apagado.

Es útil en:

* Computadoras personales.
* Laptops.
* Servidores que no permanecen encendidos todo el tiempo.
* Equipos que se suspenden.

Cron ejecuta una tarea solamente si el sistema está encendido en el momento programado.

Anacron puede ejecutarla posteriormente.

---

# Archivo de configuración de anacron

```text
/etc/anacrontab
```

Ver contenido:

```bash
cat /etc/anacrontab
```

---

# Sintaxis de anacrontab

```text
periodo retraso identificador comando
```

Ejemplo:

```text
1 5 respaldo_diario /opt/scripts/respaldo.sh
```

Interpretación:

| Campo         | Valor                      |
| ------------- | -------------------------- |
| Periodo       | Cada 1 día                 |
| Retraso       | Esperar 5 minutos          |
| Identificador | respaldo_diario            |
| Comando       | `/opt/scripts/respaldo.sh` |

---

# Ejemplos de anacron

Tarea diaria:

```text
1 5 mantenimiento_diario /opt/scripts/mantenimiento.sh
```

Tarea semanal:

```text
7 10 respaldo_semanal /opt/scripts/respaldo.sh
```

Tarea mensual:

```text
30 15 reporte_mensual /opt/scripts/reporte.sh
```

---

# Cron vs at vs anacron

| Herramienta | Tipo de tarea | Repetición | Requiere sistema encendido a la hora exacta |
| ----------- | ------------- | ---------- | ------------------------------------------- |
| `cron`      | Programada    | Sí         | Sí                                          |
| `at`        | Una sola vez  | No         | Sí                                          |
| `anacron`   | Periódica     | Sí         | No necesariamente                           |
| `batch`     | Una sola vez  | No         | Se ejecuta cuando baja la carga             |

---

# Cron vs systemd timers

Systemd también permite programar tareas mediante unidades `.timer`.

Ventajas de systemd timers:

* Mejor integración con logs.
* Dependencias entre servicios.
* Ejecución después de reinicios.
* Control mediante `systemctl`.
* Opciones avanzadas de precisión y persistencia.

Cron continúa siendo ampliamente utilizado por su simplicidad.

---

# Ejemplo de systemd timer equivalente

Servicio:

```ini
[Unit]
Description=Ejecutar respaldo

[Service]
Type=oneshot
ExecStart=/opt/scripts/respaldo.sh
```

Temporizador:

```ini
[Unit]
Description=Respaldo diario

[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

Aunque se estudia cron en este capítulo, systemd timers son una alternativa moderna.

---

# Comandos más utilizados

| Comando               | Descripción                            |
| --------------------- | -------------------------------------- |
| `crontab -e`          | Editar tareas del usuario              |
| `crontab -l`          | Listar tareas                          |
| `crontab -r`          | Eliminar todas las tareas              |
| `crontab -u`          | Administrar el crontab de otro usuario |
| `journalctl -u crond` | Ver logs de cron                       |
| `at`                  | Programar una tarea única              |
| `atq`                 | Listar tareas de at                    |
| `atrm`                | Eliminar una tarea de at               |
| `at -c`               | Mostrar una tarea de at                |
| `batch`               | Ejecutar cuando la carga sea baja      |
| `flock`               | Evitar ejecuciones simultáneas         |
| `timeout`             | Limitar duración                       |
| `anacron`             | Ejecutar tareas periódicas pendientes  |

---

# Archivos importantes

| Archivo o directorio | Función                         |
| -------------------- | ------------------------------- |
| `/etc/crontab`       | Crontab global                  |
| `/etc/cron.d/`       | Tareas individuales del sistema |
| `/etc/cron.hourly/`  | Tareas por hora                 |
| `/etc/cron.daily/`   | Tareas diarias                  |
| `/etc/cron.weekly/`  | Tareas semanales                |
| `/etc/cron.monthly/` | Tareas mensuales                |
| `/var/spool/cron/`   | Crontabs de usuarios            |
| `/etc/cron.allow`    | Usuarios autorizados            |
| `/etc/cron.deny`     | Usuarios bloqueados             |
| `/etc/at.allow`      | Usuarios autorizados para at    |
| `/etc/at.deny`       | Usuarios bloqueados para at     |
| `/etc/anacrontab`    | Configuración de anacron        |

---

# Buenas prácticas

* Utiliza rutas absolutas para comandos, scripts y archivos.
* Prueba los scripts manualmente antes de programarlos.
* Ejecuta la prueba con el mismo usuario que utilizará cron.
* Define explícitamente `PATH` y `SHELL`.
* Agrega un shebang válido al script.
* Redirige la salida y los errores a un archivo de log.
* Utiliza `flock` para impedir ejecuciones simultáneas.
* Utiliza `timeout` para controlar procesos bloqueados.
* Evita ejecutar todo como root.
* Aplica el principio de mínimo privilegio.
* No almacenes contraseñas visibles dentro del crontab.
* Protege los archivos de configuración con permisos apropiados.
* Revisa regularmente los logs de cron.
* Documenta el propósito de cada tarea.
* Elimina tareas antiguas o que ya no se utilizan.
* Considera systemd timers para tareas críticas o complejas.

---

# Laboratorio práctico

## Ejercicio 1: Crear una tarea cada minuto

Crear un script:

```bash
nano /tmp/prueba_cron.sh
```

Contenido:

```bash
#!/bin/bash

echo "Cron ejecutado: $(date)" >> /tmp/prueba_cron.log
```

Asignar permisos:

```bash
chmod +x /tmp/prueba_cron.sh
```

Agregar al crontab:

```bash
crontab -e
```

Contenido:

```cron
* * * * * /tmp/prueba_cron.sh
```

Verificar:

```bash
tail -f /tmp/prueba_cron.log
```

---

## Ejercicio 2: Listar tareas

```bash
crontab -l
```

---

## Ejercicio 3: Ejecutar cada cinco minutos

```cron
*/5 * * * * /opt/scripts/proceso.sh >> /var/log/proceso.log 2>&1
```

---

## Ejercicio 4: Evitar ejecuciones simultáneas

```cron
*/5 * * * * /usr/bin/flock -n /run/proceso.lock /opt/scripts/proceso.sh >> /var/log/proceso.log 2>&1
```

---

## Ejercicio 5: Revisar logs

Fedora y RHEL:

```bash
journalctl -u crond --since today
```

Buscar la tarea:

```bash
journalctl -u crond --since today | grep proceso.sh
```

---

## Ejercicio 6: Programar una tarea con at

```bash
echo "date >> /tmp/ejecucion_at.log" | at now + 5 minutes
```

Listar:

```bash
atq
```

---

## Ejercicio 7: Ver el contenido de una tarea

Primero obtener el número:

```bash
atq
```

Luego:

```bash
at -c NUMERO
```

---

## Ejercicio 8: Eliminar una tarea de at

```bash
atrm NUMERO
```

---

## Ejercicio 9: Crear una tarea global

Crear:

```bash
sudo nano /etc/cron.d/mantenimiento
```

Contenido:

```cron
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

0 3 * * * root /opt/scripts/mantenimiento.sh >> /var/log/mantenimiento.log 2>&1
```

Permisos:

```bash
sudo chmod 644 /etc/cron.d/mantenimiento
```

---

## Ejercicio 10: Crear una tarea al reiniciar

```cron
@reboot /opt/scripts/iniciar_app.sh >> /var/log/iniciar_app.log 2>&1
```

Reinicia el equipo únicamente en un entorno de laboratorio y verifica el log.

---

# Diagnóstico de una tarea cron que no se ejecuta

Cuando una tarea no funciona, sigue este orden.

## Paso 1: Confirmar que cron está activo

Fedora:

```bash
systemctl status crond
```

Ubuntu:

```bash
systemctl status cron
```

---

## Paso 2: Confirmar que la tarea existe

```bash
crontab -l
```

Para root:

```bash
sudo crontab -l
```

---

## Paso 3: Probar el script manualmente

```bash
/opt/scripts/proceso.sh
```

---

## Paso 4: Probar con el usuario correcto

```bash
sudo -u usuario /opt/scripts/proceso.sh
```

---

## Paso 5: Verificar permisos

```bash
ls -l /opt/scripts/proceso.sh
```

---

## Paso 6: Verificar el shebang

```bash
head -n 1 /opt/scripts/proceso.sh
```

Resultado esperado:

```text
#!/bin/bash
```

---

## Paso 7: Revisar rutas absolutas

Incorrecto:

```bash
python3 script.py
```

Correcto:

```bash
/usr/bin/python3 /opt/scripts/script.py
```

---

## Paso 8: Consultar logs

```bash
journalctl -u crond --since "1 hour ago"
```

---

## Paso 9: Registrar salida y errores

```cron
* * * * * /opt/scripts/proceso.sh >> /tmp/proceso_cron.log 2>&1
```

---

## Paso 10: Revisar SELinux

En Fedora o RHEL, SELinux puede bloquear una operación.

Buscar denegaciones recientes:

```bash
sudo ausearch -m AVC -ts recent
```

También:

```bash
sudo journalctl | grep -i denied
```

---

# Errores comunes

## Utilizar rutas relativas

Incorrecto:

```cron
0 2 * * * ./respaldo.sh
```

Correcto:

```cron
0 2 * * * /opt/scripts/respaldo.sh
```

---

## El script no tiene permisos de ejecución

Verificar:

```bash
ls -l /opt/scripts/respaldo.sh
```

Corregir:

```bash
chmod +x /opt/scripts/respaldo.sh
```

---

## El comando existe en la terminal, pero cron no lo encuentra

Causa probable:

```text
PATH limitado
```

Solución:

```cron
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

O utilizar la ruta completa:

```cron
0 2 * * * /usr/bin/python3 /opt/scripts/proceso.py
```

---

## Agregar el usuario dentro de un crontab personal

Incorrecto:

```cron
0 2 * * * root /opt/scripts/respaldo.sh
```

Esto es válido en `/etc/crontab` o `/etc/cron.d/`, pero no en `crontab -e`.

Correcto para un crontab personal:

```cron
0 2 * * * /opt/scripts/respaldo.sh
```

---

## Usar porcentajes sin escapar

Incorrecto:

```cron
0 2 * * * echo $(date +%Y%m%d)
```

Correcto:

```cron
0 2 * * * echo $(date +\%Y\%m\%d)
```

---

## Dos ejecuciones corren al mismo tiempo

Solución:

```cron
*/5 * * * * /usr/bin/flock -n /run/proceso.lock /opt/scripts/proceso.sh
```

---

## El script funciona como root, pero falla con otro usuario

Esto puede deberse a:

* Permisos de archivos.
* Acceso a directorios.
* Variables de entorno.
* Credenciales.
* SELinux.
* Rutas diferentes.

Debe probarse con el usuario real:

```bash
sudo -u usuario /opt/scripts/proceso.sh
```

---

## El equipo estaba apagado

Cron no recupera automáticamente una ejecución perdida.

Para ese caso puede utilizarse:

* `anacron`.
* Un `systemd timer` con `Persistent=true`.

---

## El crontab desapareció

Puede ocurrir si se ejecutó:

```bash
crontab -r
```

Antes de realizar cambios, crear una copia:

```bash
crontab -l > crontab_backup.txt
```

Restaurar:

```bash
crontab crontab_backup.txt
```

---

# Respaldo del crontab

Guardar:

```bash
crontab -l > "$HOME/crontab_$(date +%Y%m%d).bak"
```

Para root:

```bash
sudo crontab -l > root_crontab_backup.txt
```

Restaurar:

```bash
crontab archivo_backup.txt
```

---

# Ejemplo de tarea robusta

```cron
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

0 2 * * * /usr/bin/flock -n /run/respaldo.lock /usr/bin/timeout 2h /usr/bin/nice -n 15 /usr/bin/ionice -c3 /opt/scripts/respaldo.sh >> /var/log/respaldo.log 2>&1
```

Esta tarea:

* Se ejecuta diariamente a las 2:00 a. m.
* Evita ejecuciones simultáneas.
* Limita la duración a dos horas.
* Reduce la prioridad de CPU.
* Reduce la prioridad de disco.
* Registra salida y errores.

---

# Resumen

En este capítulo aprendiste a:

* Comprender el funcionamiento de `cron` y `crond`.
* Crear tareas periódicas mediante `crontab`.
* Interpretar los cinco campos de una expresión cron.
* Utilizar rangos, listas e intervalos.
* Administrar tareas de usuarios y del sistema.
* Configurar variables como `PATH`, `SHELL` y `MAILTO`.
* Utilizar rutas absolutas y registrar la salida.
* Consultar ejecuciones mediante `journalctl`.
* Evitar ejecuciones simultáneas con `flock`.
* Limitar tareas con `timeout`.
* Programar ejecuciones únicas mediante `at`.
* Administrar tareas pendientes con `atq` y `atrm`.
* Utilizar `batch` cuando la carga sea baja.
* Comprender la utilidad de `anacron`.
* Diagnosticar tareas que no se ejecutan correctamente.
* Aplicar buenas prácticas para automatizar tareas de forma segura y confiable.

Este capítulo conecta directamente con administración de servicios, procesos, logs, respaldos y mantenimiento automatizado.
