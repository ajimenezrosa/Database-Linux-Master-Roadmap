# 15. Configuración de Sudo

**sudo (SuperUser DO)** es una de las herramientas de seguridad más importantes en Linux. Permite ejecutar comandos con privilegios elevados sin necesidad de iniciar sesión como **root**, registrando además todas las acciones realizadas por los usuarios autorizados.

En entornos empresariales, la recomendación es **deshabilitar el acceso directo al usuario root** y administrar el sistema utilizando `sudo`.

---

# Objetivos de este capítulo

Al finalizar esta unidad serás capaz de:

* Comprender qué es `sudo` y cómo funciona.
* Configurar usuarios con privilegios administrativos.
* Administrar el archivo `sudoers`.
* Crear reglas específicas para usuarios y grupos.
* Configurar comandos permitidos sin contraseña.
* Comprender los alias de `sudo`.
* Aplicar buenas prácticas de seguridad.

---

# ¿Qué es sudo?

`sudo` permite ejecutar un comando con privilegios de otro usuario (normalmente **root**).

Ejemplo:

```bash id="sudo001"
sudo dnf update
```

El usuario introduce **su propia contraseña**, no la de root.

---

# ¿Por qué utilizar sudo?

Ventajas:

* Mayor seguridad.
* Registro de auditoría.
* Menor riesgo de errores.
* Control granular de permisos.
* No es necesario compartir la contraseña de root.

---

# ¿Cómo funciona sudo?

Flujo básico:

```text id="sudo002"
Usuario

↓

sudo comando

↓

Validación de permisos

↓

Solicitud de contraseña

↓

Ejecución del comando
```

---

# Verificar si un usuario tiene sudo

```bash id="sudo003"
sudo -l
```

Ejemplo

```text id="sudo004"
User alejandro may run the following commands...
```

---

# Verificar grupos del usuario

```bash id="sudo005"
groups
```

o

```bash id="sudo006"
id
```

---

# Agregar un usuario al grupo administrador

## Fedora / RHEL / Rocky Linux / AlmaLinux

```bash id="sudo007"
sudo usermod -aG wheel alejandro
```

---

## Ubuntu / Debian

```bash id="sudo008"
sudo usermod -aG sudo alejandro
```

---

# Verificar pertenencia

```bash id="sudo009"
groups alejandro
```

Resultado

```text id="sudo010"
alejandro wheel docker
```

---

# Archivo sudoers

Toda la configuración se almacena en:

```text id="sudo011"
/etc/sudoers
```

Nunca debe editarse directamente con un editor de texto.

La herramienta recomendada es:

```bash id="sudo012"
sudo visudo
```

`visudo` valida la sintaxis antes de guardar los cambios.

---

# Sintaxis del archivo sudoers

Formato general

```text id="sudo013"
usuario    hosts=(usuarios) comandos
```

Ejemplo

```text id="sudo014"
alejandro ALL=(ALL) ALL
```

Significado

| Campo     | Descripción                         |
| --------- | ----------------------------------- |
| alejandro | Usuario                             |
| ALL       | Cualquier host                      |
| (ALL)     | Puede actuar como cualquier usuario |
| ALL       | Puede ejecutar cualquier comando    |

---

# Dar acceso completo

```text id="sudo015"
alejandro ALL=(ALL) ALL
```

---

# Permitir ejecutar un comando específico

Ejemplo

```text id="sudo016"
alejandro ALL=(ALL) /usr/bin/systemctl restart httpd
```

Solo podrá ejecutar ese comando.

---

# Permitir varios comandos

```text id="sudo017"
alejandro ALL=(ALL) /usr/bin/systemctl,/usr/bin/journalctl
```

---

# Ejecutar sin contraseña

```text id="sudo018"
alejandro ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart httpd
```

---

# Requerir contraseña nuevamente

```text id="sudo019"
Defaults authenticate
```

---

# Tiempo de expiración de sudo

Por defecto, la autenticación permanece válida durante unos minutos (habitualmente 5).

Modificar a 15 minutos

```text id="sudo020"
Defaults timestamp_timeout=15
```

Nunca expirar

```text id="sudo021"
Defaults timestamp_timeout=-1
```

Solicitar contraseña siempre

```text id="sudo022"
Defaults timestamp_timeout=0
```

---

# Cerrar la sesión de sudo

```bash id="sudo023"
sudo -k
```

Eliminar completamente la caché

```bash id="sudo024"
sudo -K
```

---

# Ejecutar como otro usuario

```bash id="sudo025"
sudo -u postgres psql
```

También

```bash id="sudo026"
sudo -u apache whoami
```

---

# Ejecutar una Shell como root

```bash id="sudo027"
sudo -i
```

o

```bash id="sudo028"
sudo su -
```

Se recomienda utilizar `sudo -i`, ya que conserva un mejor registro de auditoría.

---

# Alias en sudoers

## User Alias

```text id="sudo029"
User_Alias DBA = alejandro,juan,maria
```

---

## Command Alias

```text id="sudo030"
Cmnd_Alias SERVICES = /usr/bin/systemctl
```

---

## Host Alias

```text id="sudo031"
Host_Alias SERVERS = servidor1,servidor2
```

---

## Runas Alias

```text id="sudo032"
Runas_Alias ADMINS = root,postgres
```

---

# Ejemplo completo

```text id="sudo033"
User_Alias DBA = alejandro,juan

Cmnd_Alias BACKUPS = /usr/bin/rsync,/usr/bin/tar

DBA ALL=(root) BACKUPS
```

---

# Directorio sudoers.d

Además del archivo principal, pueden agregarse configuraciones independientes.

Ubicación

```text id="sudo034"
/etc/sudoers.d/
```

Crear archivo

```bash id="sudo035"
sudo visudo -f /etc/sudoers.d/dba
```

Ejemplo

```text id="sudo036"
alejandro ALL=(ALL) ALL
```

Esta práctica facilita la administración y evita modificar el archivo principal.

---

# Ver registros de sudo

Fedora / RHEL

```bash id="sudo037"
sudo journalctl | grep sudo
```

Ubuntu / Debian

```bash id="sudo038"
sudo grep sudo /var/log/auth.log
```

---

# Verificar configuración

```bash id="sudo039"
sudo visudo -c
```

Resultado

```text id="sudo040"
/etc/sudoers: parsed OK
```

---

# Variables de entorno

Ejecutar preservando variables

```bash id="sudo041"
sudo -E comando
```

Ver usuario efectivo

```bash id="sudo042"
sudo whoami
```

Resultado

```text id="sudo043"
root
```

---

# Comandos útiles

Actualizar credenciales

```bash id="sudo044"
sudo -v
```

Ver versión

```bash id="sudo045"
sudo -V
```

Listar privilegios

```bash id="sudo046"
sudo -l
```

---

# Archivos relacionados

| Archivo             | Función                                   |
| ------------------- | ----------------------------------------- |
| `/etc/sudoers`      | Configuración principal de sudo           |
| `/etc/sudoers.d/`   | Configuraciones adicionales               |
| `/etc/group`        | Grupos de usuarios                        |
| `/var/log/auth.log` | Registro de autenticación (Debian/Ubuntu) |
| `journalctl`        | Registros del sistema (systemd)           |

---

# Comandos más utilizados

| Comando           | Descripción                                      |
| ----------------- | ------------------------------------------------ |
| `sudo comando`    | Ejecutar un comando como root                    |
| `sudo -l`         | Mostrar privilegios del usuario                  |
| `sudo -k`         | Invalidar la autenticación actual                |
| `sudo -K`         | Eliminar completamente la caché de autenticación |
| `sudo -u usuario` | Ejecutar como otro usuario                       |
| `sudo -i`         | Abrir una shell de root                          |
| `visudo`          | Editar el archivo sudoers de forma segura        |
| `visudo -c`       | Validar la configuración                         |

---

# Buenas prácticas

* Utiliza siempre `visudo` para editar la configuración.
* Prefiere archivos individuales dentro de `/etc/sudoers.d/`.
* Otorga únicamente los privilegios necesarios (principio de mínimo privilegio).
* Evita el uso indiscriminado de `NOPASSWD`.
* Revisa periódicamente quién pertenece a los grupos `wheel` o `sudo`.
* Audita los registros de `sudo` regularmente.
* Deshabilita el acceso directo al usuario `root` cuando sea posible.
* Documenta cualquier cambio en la configuración de privilegios.

---

# Laboratorio práctico

## Ejercicio 1: Agregar un usuario al grupo administrador

Fedora / RHEL

```bash id="labsudo001"
sudo usermod -aG wheel juan
```

Ubuntu / Debian

```bash id="labsudo002"
sudo usermod -aG sudo juan
```

Verificar

```bash id="labsudo003"
groups juan
```

---

## Ejercicio 2: Ver privilegios de sudo

```bash id="labsudo004"
sudo -l
```

---

## Ejercicio 3: Crear una regla personalizada

Crear un archivo en `/etc/sudoers.d/`:

```bash id="labsudo005"
sudo visudo -f /etc/sudoers.d/backups
```

Contenido:

```text id="labsudo006"
juan ALL=(root) /usr/bin/tar,/usr/bin/rsync
```

---

## Ejercicio 4: Validar la configuración

```bash id="labsudo007"
sudo visudo -c
```

---

## Ejercicio 5: Ejecutar un comando como otro usuario

```bash id="labsudo008"
sudo -u postgres psql
```

---

## Ejercicio 6: Invalidar la sesión de sudo

```bash id="labsudo009"
sudo -k
```

Luego intenta ejecutar:

```bash id="labsudo010"
sudo whoami
```

Se solicitará nuevamente la contraseña.

---

# Errores comunes

### Editar `/etc/sudoers` con un editor de texto

```bash id="errsudo001"
sudo nano /etc/sudoers
```

Esto puede introducir errores de sintaxis y dejar el sistema sin acceso administrativo.

**Correcto:**

```bash id="errsudo002"
sudo visudo
```

---

### Olvidar la opción `-a` al agregar un grupo

Incorrecto:

```bash id="errsudo003"
sudo usermod -G wheel juan
```

Esto reemplaza todos los grupos secundarios del usuario.

Correcto:

```bash id="errsudo004"
sudo usermod -aG wheel juan
```

---

### Conceder privilegios excesivos

Evita reglas como:

```text id="errsudo005"
ALL ALL=(ALL) NOPASSWD: ALL
```

Salvo en entornos de laboratorio muy controlados.

---

# Resumen

En este capítulo aprendiste a:

* Comprender el funcionamiento de `sudo`.
* Agregar usuarios con privilegios administrativos.
* Configurar reglas mediante `sudoers` y `sudoers.d`.
* Permitir comandos específicos y controlar el uso de contraseñas.
* Utilizar alias para simplificar configuraciones complejas.
* Auditar y validar la configuración de `sudo`.
* Aplicar buenas prácticas para administrar privilegios de forma segura en sistemas Linux.
