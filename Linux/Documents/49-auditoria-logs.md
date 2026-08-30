# 49. Auditoría y Logs en Red Hat Enterprise Linux

> **Módulo 7: Seguridad del Sistema**  
> **Página 49 del Manual RHCSA**

---

# Objetivos de aprendizaje

Al finalizar esta lección serás capaz de:

- Comprender la importancia de la auditoría en Linux.
- Conocer los principales archivos de logs del sistema.
- Utilizar `journalctl` para consultar registros.
- Comprender el funcionamiento de `rsyslog` y `systemd-journald`.
- Consultar eventos de autenticación.
- Utilizar `auditd` y `ausearch` para auditorías de seguridad.
- Diagnosticar incidentes mediante los registros del sistema.

---

# Introducción

Todo administrador de sistemas debe ser capaz de responder preguntas como:

- ¿Quién inició sesión?
- ¿Cuándo falló un servicio?
- ¿Qué usuario ejecutó un comando?
- ¿Quién modificó un archivo?
- ¿Cuándo reinició el servidor?
- ¿Por qué una aplicación dejó de funcionar?

La respuesta normalmente se encuentra en los **logs** y en el sistema de **auditoría**.

En Red Hat Enterprise Linux existen dos mecanismos principales:

- **System Logs** (journalctl / rsyslog)
- **Linux Audit Framework (auditd)**

---

# Arquitectura de los registros

```
Aplicaciones

↓

Kernel

↓

systemd-journald

↓

journalctl

↓

rsyslog (opcional)

↓

/var/log/
```

---

# ¿Qué es systemd-journald?

Es el servicio responsable de recopilar los eventos del sistema.

Consultar su estado:

```bash
systemctl status systemd-journald
```

---

# ¿Qué es journalctl?

Es la herramienta utilizada para consultar los registros almacenados por `systemd-journald`.

Es el método recomendado en versiones modernas de Red Hat Enterprise Linux.

---

# Consultar todos los registros

```bash
journalctl
```

---

# Ver únicamente los registros recientes

```bash
journalctl -n 50
```

---

# Seguir los registros en tiempo real

```bash
journalctl -f
```

Equivalente al comportamiento de:

```bash
tail -f
```

---

# Mostrar eventos desde el arranque actual

```bash
journalctl -b
```

---

# Mostrar eventos del arranque anterior

```bash
journalctl -b -1
```

---

# Buscar eventos por fecha

```bash
journalctl \
--since "2026-07-28 08:00:00"
```

---

Entre dos fechas:

```bash
journalctl \
--since yesterday \
--until today
```

---

# Consultar registros de un servicio

Ejemplo:

```bash
journalctl -u sshd
```

Últimos eventos:

```bash
journalctl -u sshd -n 30
```

Tiempo real:

```bash
journalctl -fu sshd
```

---

# Consultar el servicio de red

```bash
journalctl -u NetworkManager
```

---

# Consultar Firewalld

```bash
journalctl -u firewalld
```

---

# Consultar SELinux

Buscar mensajes relacionados:

```bash
journalctl | grep AVC
```

---

# Filtrar por prioridad

Errores:

```bash
journalctl -p err
```

Advertencias:

```bash
journalctl -p warning
```

Solo mensajes críticos:

```bash
journalctl -p crit
```

---

# Consultar registros del Kernel

```bash
journalctl -k
```

Últimos eventos:

```bash
journalctl -k -n 50
```

---

# Archivos clásicos de logs

Aunque `journalctl` es la herramienta principal, muchos sistemas también almacenan registros en:

```
/var/log/
```

---

# Directorios importantes

| Archivo | Contenido |
|----------|-----------|
| `/var/log/messages` | Eventos generales del sistema (si rsyslog está configurado) |
| `/var/log/secure` | Autenticación y seguridad |
| `/var/log/maillog` | Correo electrónico |
| `/var/log/cron` | Cron y tareas programadas |
| `/var/log/boot.log` | Arranque |
| `/var/log/dmesg` | Mensajes del kernel |
| `/var/log/audit/audit.log` | Auditoría del sistema |

> **Nota:** Dependiendo de la distribución o configuración, algunos archivos pueden no existir y toda la información estará disponible únicamente mediante `journalctl`.

---

# Consultar un archivo de log

```bash
less /var/log/messages
```

Buscar texto:

```
/error
```

Salir:

```
q
```

---

# Seguir un archivo

```bash
tail -f /var/log/messages
```

---

# ¿Qué es auditd?

`auditd` es el servicio encargado de registrar eventos relacionados con la seguridad.

Registrar:

- Accesos.
- Cambios en archivos.
- Ejecución de programas.
- Eventos del kernel.
- Actividad de usuarios.

---

# Verificar auditd

```bash
systemctl status auditd
```

---

# Consultar eventos de auditoría

```bash
ausearch
```

---

# Buscar autenticaciones

```bash
ausearch -m USER_LOGIN
```

---

# Buscar eventos AVC

```bash
ausearch -m AVC
```

---

# Buscar eventos recientes

```bash
ausearch -ts recent
```

---

# Buscar por usuario

```bash
ausearch -ua root
```

---

# Resumen de auditoría

```bash
aureport
```

---

# Resumen de autenticaciones

```bash
aureport -au
```

---

# Resumen de inicios de sesión

```bash
aureport -l
```

---

# Consultar usuarios conectados

```bash
who
```

---

Información adicional:

```bash
w
```

---

# Historial de conexiones

```bash
last
```

Últimos reinicios:

```bash
last reboot
```

---

# Historial de intentos fallidos

```bash
lastb
```

> Requiere privilegios de administrador.

---

# Flujo de auditoría

```
Usuario

↓

Servicio

↓

Kernel

↓

auditd

↓

audit.log

↓

ausearch

↓

Administrador
```

---

# Ejemplo práctico

Un usuario informa que no pudo iniciar sesión por SSH.

Procedimiento:

Verificar el servicio:

```bash
systemctl status sshd
```

Consultar registros:

```bash
journalctl -u sshd
```

Buscar autenticaciones:

```bash
ausearch -m USER_LOGIN
```

Consultar intentos recientes:

```bash
last

lastb
```

---

# Rotación de logs

Para evitar que los registros ocupen todo el espacio en disco se utiliza:

```
logrotate
```

Verificar configuración:

```bash
ls /etc/logrotate.d
```

Configuración principal:

```text
/etc/logrotate.conf
```

---

# Herramientas relacionadas

| Herramienta | Función |
|-------------|----------|
| `journalctl` | Consultar registros |
| `systemctl` | Administrar servicios |
| `tail` | Ver el final de un archivo |
| `less` | Leer archivos grandes |
| `grep` | Filtrar texto |
| `auditd` | Servicio de auditoría |
| `ausearch` | Buscar eventos |
| `aureport` | Generar informes |
| `last` | Historial de conexiones |
| `lastb` | Intentos fallidos |
| `who` | Usuarios conectados |
| `w` | Información detallada de usuarios |

---

# Buenas prácticas RHCSA

✔ Revisar periódicamente los registros del sistema.

✔ Supervisar los intentos fallidos de autenticación.

✔ Mantener el servicio `auditd` habilitado.

✔ Verificar regularmente el espacio utilizado por los logs.

✔ Utilizar `journalctl` como herramienta principal de consulta.

✔ Conservar los registros según la política de retención de la organización.

✔ Documentar los eventos relevantes durante la resolución de incidentes.

---

# Errores comunes

## Ignorar los registros

La mayoría de los problemas dejan evidencia en los logs.

---

## Revisar únicamente journalctl

Algunas aplicaciones aún escriben directamente en:

```
/var/log/
```

---

## No revisar audit.log

Muchos eventos de seguridad únicamente aparecen en la auditoría.

---

## Eliminar registros manualmente

La gestión debe realizarse mediante `logrotate` y las políticas definidas por la organización.

---

## No comprobar el espacio en disco

Una partición llena puede impedir que se escriban nuevos registros.

Verificar:

```bash
df -h
```

---

# Comandos importantes RHCSA

| Comando | Descripción |
|----------|-------------|
| `journalctl` | Mostrar todos los registros |
| `journalctl -b` | Registros del arranque actual |
| `journalctl -u servicio` | Registros de un servicio |
| `journalctl -f` | Seguir registros en tiempo real |
| `journalctl -p err` | Mostrar errores |
| `journalctl -k` | Registros del kernel |
| `ausearch` | Buscar eventos de auditoría |
| `aureport` | Generar informes |
| `last` | Historial de conexiones |
| `lastb` | Intentos fallidos |
| `who` | Usuarios conectados |
| `w` | Información detallada de usuarios |

---

# Resumen

En esta lección aprendiste a:

- Comprender el funcionamiento de `systemd-journald`.
- Utilizar `journalctl` para consultar registros.
- Conocer los principales archivos de log del sistema.
- Utilizar `auditd`, `ausearch` y `aureport`.
- Consultar eventos de autenticación y seguridad.
- Diagnosticar problemas mediante los registros del sistema.

---

# Laboratorio práctico RHCSA

## Escenario 1

Consulta los últimos 20 registros del sistema.

```bash
journalctl -n 20
```

---

## Escenario 2

Visualiza los registros del servicio SSH.

```bash
journalctl -u sshd -n 30
```

Después, sigue los registros en tiempo real mientras realizas una conexión SSH desde otro equipo.

```bash
journalctl -fu sshd
```

---

## Escenario 3

Consulta los eventos del arranque actual y del arranque anterior.

```bash
journalctl -b

journalctl -b -1
```

---

## Escenario 4

Verifica el estado del servicio de auditoría y genera un informe de autenticaciones.

```bash
systemctl status auditd

aureport -au
```

Busca también eventos recientes de inicio de sesión.

```bash
ausearch -m USER_LOGIN -ts recent
```

---

## Escenario 5

Consulta el historial de conexiones y los intentos fallidos de autenticación.

```bash
last

sudo lastb
```

Finalmente, verifica el espacio disponible para los registros del sistema.

```bash
df -h
```

> **Objetivo general:** dominar el uso de **`journalctl`**, **`auditd`** y las herramientas de auditoría de Red Hat Enterprise Linux para analizar eventos del sistema, investigar incidentes de seguridad y diagnosticar problemas de forma eficiente. Estas competencias son esenciales para el examen **RHCSA** y para la administración profesional de servidores Linux.