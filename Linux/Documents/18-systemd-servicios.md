# 18. Systemd y Administración de Servicios

**systemd** es el sistema de inicio (**init system**) utilizado por la mayoría de las distribuciones Linux modernas, como Fedora, RHEL, Rocky Linux, AlmaLinux, Ubuntu, Debian y SUSE.

Su función principal es administrar el proceso de arranque del sistema, iniciar y detener servicios, controlar dependencias, administrar registros mediante **journald** y proporcionar herramientas para supervisar el estado general del sistema.

Hoy en día, dominar **systemd** es una habilidad indispensable para cualquier administrador Linux.

---

# Objetivos de este capítulo

Al finalizar esta unidad serás capaz de:

* Comprender qué es systemd.
* Administrar servicios mediante `systemctl`.
* Habilitar y deshabilitar servicios.
* Analizar el estado de un servicio.
* Revisar registros con `journalctl`.
* Comprender las unidades (Units) de systemd.
* Crear un servicio personalizado.
* Aplicar buenas prácticas de administración.

---

# ¿Qué es systemd?

Systemd es el primer proceso que inicia el sistema operativo.

Su PID siempre es:

```bash id="sys001"
ps -p 1
```

Resultado

```text id="sys002"
PID TTY      TIME CMD
1   ?        00:00:04 systemd
```

---

# Funciones de systemd

Systemd administra:

* Arranque del sistema.
* Servicios.
* Montajes.
* Dispositivos.
* Temporizadores.
* Registros del sistema.
* Dependencias.
* Usuarios.

---

# ¿Qué es una Unit?

Todo en systemd se administra mediante **Units**.

Las unidades representan distintos tipos de recursos del sistema.

| Tipo    | Extensión  | Descripción            |
| ------- | ---------- | ---------------------- |
| Service | `.service` | Servicios              |
| Target  | `.target`  | Estados del sistema    |
| Socket  | `.socket`  | Activación por sockets |
| Mount   | `.mount`   | Sistemas de archivos   |
| Timer   | `.timer`   | Tareas programadas     |
| Device  | `.device`  | Dispositivos           |
| Path    | `.path`    | Monitoreo de archivos  |
| Swap    | `.swap`    | Áreas de intercambio   |

---

# Ver servicios

```bash id="sys003"
systemctl list-units --type=service
```

Solo los activos

```bash id="sys004"
systemctl list-units --type=service --state=running
```

---

# Ver todos los servicios

```bash id="sys005"
systemctl list-unit-files --type=service
```

---

# Consultar un servicio

Ejemplo

```bash id="sys006"
systemctl status sshd
```

Resultado

```text id="sys007"
Active: active (running)
```

---

Otro ejemplo

```bash id="sys008"
systemctl status postgresql
```

---

SQL Server

```bash id="sys009"
systemctl status mssql-server
```

---

# Iniciar un servicio

```bash id="sys010"
sudo systemctl start httpd
```

---

# Detener un servicio

```bash id="sys011"
sudo systemctl stop httpd
```

---

# Reiniciar un servicio

```bash id="sys012"
sudo systemctl restart httpd
```

---

# Recargar configuración

Cuando el servicio soporta recarga sin reinicio:

```bash id="sys013"
sudo systemctl reload httpd
```

---

# Recargar o reiniciar

```bash id="sys014"
sudo systemctl reload-or-restart httpd
```

---

# Verificar estado

```bash id="sys015"
systemctl is-active httpd
```

Resultado

```text id="sys016"
active
```

---

# Saber si inicia con el sistema

```bash id="sys017"
systemctl is-enabled httpd
```

Resultado

```text id="sys018"
enabled
```

---

# Habilitar un servicio

```bash id="sys019"
sudo systemctl enable httpd
```

---

# Deshabilitar

```bash id="sys020"
sudo systemctl disable httpd
```

---

# Habilitar e iniciar

```bash id="sys021"
sudo systemctl enable --now httpd
```

---

# Deshabilitar y detener

```bash id="sys022"
sudo systemctl disable --now httpd
```

---

# Recargar systemd

Después de modificar archivos `.service`

```bash id="sys023"
sudo systemctl daemon-reload
```

---

# Reiniciar systemd

```bash id="sys024"
sudo systemctl daemon-reexec
```

---

# Journalctl

Systemd incluye su propio sistema de registros.

Ver todo

```bash id="sys025"
journalctl
```

---

Ver registros recientes

```bash id="sys026"
journalctl -n 50
```

---

Seguir registros en tiempo real

```bash id="sys027"
journalctl -f
```

---

Ver registros de un servicio

```bash id="sys028"
journalctl -u sshd
```

---

Últimos registros

```bash id="sys029"
journalctl -u postgresql -n 100
```

---

Seguir un servicio

```bash id="sys030"
journalctl -u mssql-server -f
```

---

Filtrar por fecha

```bash id="sys031"
journalctl --since today
```

Última hora

```bash id="sys032"
journalctl --since "1 hour ago"
```

---

Filtrar por prioridad

Errores

```bash id="sys033"
journalctl -p err
```

Advertencias

```bash id="sys034"
journalctl -p warning
```

---

# Targets

Los Targets reemplazan los antiguos Runlevels de SysV.

Ver Target actual

```bash id="sys035"
systemctl get-default
```

---

Cambiar a modo gráfico

```bash id="sys036"
sudo systemctl set-default graphical.target
```

---

Modo texto

```bash id="sys037"
sudo systemctl set-default multi-user.target
```

---

Cambiar temporalmente

```bash id="sys038"
sudo systemctl isolate multi-user.target
```

---

# Dependencias

Ver dependencias

```bash id="sys039"
systemctl list-dependencies sshd
```

---

Invertidas

```bash id="sys040"
systemctl list-dependencies --reverse sshd
```

---

# Analizar tiempo de arranque

```bash id="sys041"
systemd-analyze
```

---

Servicios más lentos

```bash id="sys042"
systemd-analyze blame
```

---

Cadena crítica

```bash id="sys043"
systemd-analyze critical-chain
```

---

# Crear un servicio personalizado

Crear archivo

```text id="sys044"
/etc/systemd/system/miapp.service
```

Contenido

```ini id="sys045"
[Unit]
Description=Mi Aplicación
After=network.target

[Service]
Type=simple
User=alejandro
ExecStart=/opt/miapp/app.sh
Restart=always

[Install]
WantedBy=multi-user.target
```

---

Recargar

```bash id="sys046"
sudo systemctl daemon-reload
```

---

Habilitar

```bash id="sys047"
sudo systemctl enable miapp
```

---

Iniciar

```bash id="sys048"
sudo systemctl start miapp
```

---

Estado

```bash id="sys049"
systemctl status miapp
```

---

# Ver propiedades

```bash id="sys050"
systemctl show sshd
```

---

Propiedad específica

```bash id="sys051"
systemctl show sshd -p MainPID
```

---

# Ver procesos del servicio

```bash id="sys052"
systemctl status sshd
```

o

```bash id="sys053"
systemctl show sshd -p MainPID
```

---

# Enmascarar un servicio

Impide que pueda iniciarse.

```bash id="sys054"
sudo systemctl mask httpd
```

Desenmascarar

```bash id="sys055"
sudo systemctl unmask httpd
```

---

# Reiniciar servicios fallidos

```bash id="sys056"
systemctl --failed
```

Reiniciar

```bash id="sys057"
sudo systemctl restart servicio
```

---

# Reiniciar todos los servicios fallidos

```bash id="sys058"
sudo systemctl reset-failed
```

---

# Ubicación de los archivos

| Directorio                 | Descripción                             |
| -------------------------- | --------------------------------------- |
| `/usr/lib/systemd/system/` | Servicios instalados por paquetes       |
| `/etc/systemd/system/`     | Servicios personalizados y sobrescritos |
| `/run/systemd/system/`     | Configuración temporal                  |

---

# Comandos más utilizados

| Comando                   | Descripción                  |
| ------------------------- | ---------------------------- |
| `systemctl status`        | Estado del servicio          |
| `systemctl start`         | Iniciar servicio             |
| `systemctl stop`          | Detener servicio             |
| `systemctl restart`       | Reiniciar servicio           |
| `systemctl reload`        | Recargar configuración       |
| `systemctl enable`        | Habilitar al inicio          |
| `systemctl disable`       | Deshabilitar al inicio       |
| `systemctl daemon-reload` | Recargar archivos `.service` |
| `journalctl`              | Ver registros                |
| `systemd-analyze`         | Analizar el arranque         |

---

# Archivos relacionados

| Archivo o directorio       | Función                                       |
| -------------------------- | --------------------------------------------- |
| `/etc/systemd/system/`     | Servicios personalizados                      |
| `/usr/lib/systemd/system/` | Servicios instalados                          |
| `/run/systemd/system/`     | Configuración temporal                        |
| `/etc/machine-id`          | Identificador único del sistema               |
| `/var/log/journal/`        | Registros persistentes (si están habilitados) |

---

# Buenas prácticas

* Utiliza siempre `systemctl` para administrar servicios.
* Después de modificar un archivo `.service`, ejecuta `systemctl daemon-reload`.
* Revisa los registros con `journalctl` antes de reiniciar un servicio.
* Configura `Restart=` únicamente cuando sea apropiado para el servicio.
* Utiliza `enable --now` para habilitar e iniciar un servicio en un solo paso.
* Guarda los servicios personalizados en `/etc/systemd/system/`.
* Utiliza `systemd-analyze blame` para identificar servicios que retrasan el arranque.
* Documenta cualquier modificación realizada a servicios críticos.

---

# Laboratorio práctico

## Ejercicio 1: Consultar el estado de un servicio

```bash id="labsys001"
systemctl status sshd
```

---

## Ejercicio 2: Reiniciar un servicio

```bash id="labsys002"
sudo systemctl restart sshd
```

Verificar

```bash id="labsys003"
systemctl status sshd
```

---

## Ejercicio 3: Ver registros

```bash id="labsys004"
journalctl -u sshd -n 20
```

---

## Ejercicio 4: Crear un servicio personalizado

Crear el archivo:

```bash id="labsys005"
sudo nano /etc/systemd/system/prueba.service
```

Contenido:

```ini id="labsys006"
[Unit]
Description=Servicio de Prueba

[Service]
ExecStart=/usr/bin/sleep infinity

[Install]
WantedBy=multi-user.target
```

Recargar y habilitar:

```bash id="labsys007"
sudo systemctl daemon-reload

sudo systemctl enable --now prueba
```

---

## Ejercicio 5: Analizar el arranque

```bash id="labsys008"
systemd-analyze

systemd-analyze blame
```

---

## Ejercicio 6: Ver servicios fallidos

```bash id="labsys009"
systemctl --failed
```

---

# Errores comunes

### Modificar un archivo `.service` y olvidar recargar systemd

Después de cualquier cambio:

```bash id="errsys001"
sudo systemctl daemon-reload
```

De lo contrario, systemd seguirá utilizando la configuración anterior.

---

### Reiniciar un servicio sin revisar los registros

Antes de reiniciar un servicio que falla, consulta sus registros:

```bash id="errsys002"
journalctl -u nombre_servicio -n 50
```

Esto suele proporcionar información más útil que reiniciarlo repetidamente.

---

### Confundir `reload` con `restart`

* `reload` recarga la configuración **sin detener el servicio**, siempre que el servicio lo soporte.
* `restart` detiene e inicia nuevamente el servicio, interrumpiendo temporalmente su funcionamiento.

---

# Resumen

En este capítulo aprendiste a:

* Comprender el funcionamiento de **systemd** como sistema de inicio y administración de servicios.
* Gestionar servicios con `systemctl`.
* Habilitar, deshabilitar, iniciar, detener y reiniciar servicios.
* Analizar registros mediante `journalctl`.
* Comprender los diferentes tipos de **Units** y **Targets**.
* Crear y administrar servicios personalizados.
* Analizar el proceso de arranque con `systemd-analyze`.
* Aplicar buenas prácticas para administrar servicios de forma segura y eficiente en sistemas Linux modernos.
