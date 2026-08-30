# 10. Administración de Usuarios

La administración de usuarios es una de las tareas más importantes en Linux. Un administrador del sistema debe controlar quién puede acceder al servidor, qué permisos tiene cada usuario y cómo proteger la información mediante una correcta gestión de cuentas, grupos y políticas de seguridad.

---

# Objetivos de este capítulo

Al finalizar esta unidad serás capaz de:

* Comprender cómo funciona el sistema de usuarios y grupos en Linux.
* Crear, modificar y eliminar usuarios.
* Administrar grupos.
* Cambiar contraseñas y políticas de expiración.
* Otorgar privilegios administrativos.
* Bloquear y desbloquear cuentas.
* Entender los archivos relacionados con usuarios y autenticación.
* Aplicar buenas prácticas de seguridad.

---

# ¿Qué es un usuario?

Un usuario representa una identidad dentro del sistema operativo.

Cada usuario posee:

* Un nombre (username)
* Un identificador único (UID)
* Un grupo principal (GID)
* Un directorio personal (Home Directory)
* Un Shell por defecto
* Una contraseña (almacenada de forma cifrada)

Ejemplo:

```
Usuario:
    ajimenez

UID:
    1001

Grupo:
    users

Home:
    /home/ajimenez

Shell:
    /bin/bash
```

---

# Tipos de usuarios

## 1. Usuario Root

Es el administrador absoluto del sistema.

Características:

* UID = 0
* Puede realizar cualquier acción
* No tiene restricciones

Ver el usuario actual

```bash
whoami
```

Resultado

```
root
```

---

## 2. Usuarios normales

Son los usuarios utilizados para trabajar diariamente.

Ejemplo

```
alejandro
juan
maria
developer
```

Generalmente poseen UID mayores que 1000.

---

## 3. Usuarios del sistema

Son utilizados por servicios.

Ejemplos

```
apache
mysql
postgres
sshd
nginx
chrony
```

Normalmente poseen UID menores que 1000.

Ver usuarios del sistema

```bash
cat /etc/passwd
```

---

# El archivo /etc/passwd

Contiene todos los usuarios del sistema.

Visualizar

```bash
cat /etc/passwd
```

Ejemplo

```
root:x:0:0:root:/root:/bin/bash
```

Cada línea posee siete campos.

```
usuario
contraseña
UID
GID
comentario
home
shell
```

Ejemplo explicado

```
alejandro:x:1001:1001:Alejandro:/home/alejandro:/bin/bash
```

| Campo           | Significado          |
| --------------- | -------------------- |
| alejandro       | Usuario              |
| x               | Contraseña en shadow |
| 1001            | UID                  |
| 1001            | GID                  |
| Alejandro       | Información          |
| /home/alejandro | Directorio Home      |
| /bin/bash       | Shell                |

---

# El archivo /etc/shadow

Aquí se almacenan las contraseñas cifradas.

Solo Root puede leerlo.

Visualizar

```bash
sudo cat /etc/shadow
```

Ejemplo

```
alejandro:$y$j9T...
```

Contiene:

* contraseña cifrada
* fecha del último cambio
* expiración
* advertencias
* bloqueo

Ver permisos

```bash
ls -l /etc/shadow
```

---

# El archivo /etc/group

Contiene los grupos.

Ver

```bash
cat /etc/group
```

Ejemplo

```
sudo:x:27:alejandro
```

Campos

```
grupo
contraseña
GID
miembros
```

---

# Crear usuarios

Sintaxis

```bash
sudo useradd usuario
```

Ejemplo

```bash
sudo useradd juan
```

Crear con Home

```bash
sudo useradd -m juan
```

Crear con Bash

```bash
sudo useradd -m -s /bin/bash juan
```

Crear con comentario

```bash
sudo useradd -m -c "Juan Perez" juan
```

---

# Establecer contraseña

```bash
sudo passwd juan
```

Ejemplo

```
New password:
Retype new password:
passwd: password updated successfully
```

---

# Crear un usuario completamente

```bash
sudo useradd -m \
-s /bin/bash \
-c "Administrador SQL" \
alejandro

sudo passwd alejandro
```

---

# Ver información del usuario

```bash
id alejandro
```

Resultado

```
uid=1001
gid=1001
groups=1001,10
```

También

```bash
finger alejandro
```

(si está instalado)

---

# Cambiar información

Modificar nombre completo

```bash
sudo usermod -c "Administrador Linux" alejandro
```

Cambiar Shell

```bash
sudo usermod -s /bin/zsh alejandro
```

Cambiar Home

```bash
sudo usermod -d /home/admin alejandro
```

Mover el Home

```bash
sudo usermod -m -d /home/admin alejandro
```

---

# Cambiar nombre del usuario

```bash
sudo usermod -l administrador alejandro
```

---

# Bloquear un usuario

```bash
sudo passwd -l juan
```

o

```bash
sudo usermod -L juan
```

Verificar

```bash
sudo passwd -S juan
```

Resultado

```
juan L
```

---

# Desbloquear

```bash
sudo passwd -u juan
```

o

```bash
sudo usermod -U juan
```

---

# Eliminar usuarios

Eliminar únicamente la cuenta

```bash
sudo userdel juan
```

Eliminar cuenta y Home

```bash
sudo userdel -r juan
```

---

# Crear grupos

```bash
sudo groupadd desarrolladores
```

Verificar

```bash
getent group desarrolladores
```

---

# Agregar usuarios a grupos

```bash
sudo usermod -aG desarrolladores juan
```

Ver grupos

```bash
groups juan
```

o

```bash
id juan
```

---

# Cambiar grupo principal

```bash
sudo usermod -g desarrolladores juan
```

---

# Eliminar grupos

```bash
sudo groupdel desarrolladores
```

---

# Cambiar contraseña de otro usuario

```bash
sudo passwd juan
```

---

# Expiración de contraseña

Ver política

```bash
sudo chage -l juan
```

Forzar cambio al próximo inicio

```bash
sudo chage -d 0 juan
```

Expirar en 90 días

```bash
sudo chage -M 90 juan
```

Advertir 7 días antes

```bash
sudo chage -W 7 juan
```

---

# Deshabilitar acceso mediante Shell

Muy útil para cuentas de servicios.

```bash
sudo usermod -s /sbin/nologin usuario
```

o

```bash
sudo usermod -s /usr/sbin/nologin usuario
```

Según la distribución.

---

# Cambiar propietario de archivos

```bash
sudo chown juan archivo.txt
```

Cambiar propietario y grupo

```bash
sudo chown juan:desarrolladores archivo.txt
```

Recursivamente

```bash
sudo chown -R juan:desarrolladores carpeta/
```

---

# Permisos administrativos (sudo)

Agregar al grupo wheel (Fedora, RHEL, Rocky, AlmaLinux)

```bash
sudo usermod -aG wheel juan
```

Agregar al grupo sudo (Ubuntu, Debian)

```bash
sudo usermod -aG sudo juan
```

Verificar

```bash
groups juan
```

---

# Cambiar de usuario

Cambiar a otro usuario

```bash
su juan
```

Abrir sesión completa

```bash
su - juan
```

Volver

```bash
exit
```

---

# Información de usuarios conectados

Usuarios activos

```bash
who
```

Información detallada

```bash
w
```

Últimos inicios

```bash
last
```

Último inicio de un usuario

```bash
lastlog
```

---

# Encontrar el UID del usuario

```bash
id
```

Solo UID

```bash
id -u
```

Solo GID

```bash
id -g
```

---

# Archivos importantes

| Archivo           | Función                         |
| ----------------- | ------------------------------- |
| `/etc/passwd`     | Información de usuarios         |
| `/etc/shadow`     | Contraseñas cifradas            |
| `/etc/group`      | Información de grupos           |
| `/etc/gshadow`    | Contraseñas de grupos           |
| `/etc/login.defs` | Configuración de cuentas        |
| `/etc/skel/`      | Plantillas para nuevos usuarios |
| `/home/`          | Directorios personales          |
| `/root/`          | Directorio del usuario root     |

---

# Buenas prácticas

* Nunca trabajes como **root** para tareas cotidianas.
* Utiliza **sudo** siempre que sea posible.
* Asigna el principio de **mínimo privilegio**.
* Elimina cuentas que ya no se utilicen.
* Revisa periódicamente los grupos de usuarios.
* Obliga a cambiar contraseñas periódicamente.
* Usa contraseñas robustas y autenticación multifactor cuando esté disponible.
* Mantén los directorios **Home** con permisos adecuados.
* Audita regularmente los accesos y usuarios con privilegios.

---

# Laboratorio práctico

## Ejercicio 1: Crear un usuario

```bash
sudo useradd -m laboratorio
sudo passwd laboratorio
id laboratorio
```

---

## Ejercicio 2: Crear un grupo y agregar el usuario

```bash
sudo groupadd dba
sudo usermod -aG dba laboratorio
groups laboratorio
```

---

## Ejercicio 3: Forzar cambio de contraseña

```bash
sudo chage -d 0 laboratorio
```

Cerrar sesión e iniciar nuevamente para comprobar el cambio.

---

## Ejercicio 4: Bloquear y desbloquear la cuenta

```bash
sudo usermod -L laboratorio
sudo passwd -S laboratorio

sudo usermod -U laboratorio
sudo passwd -S laboratorio
```

---

## Ejercicio 5: Eliminar el usuario

```bash
sudo userdel -r laboratorio
```

---

# Resumen

En este capítulo aprendiste a:

* Comprender la estructura de usuarios y grupos en Linux.
* Administrar cuentas mediante `useradd`, `usermod` y `userdel`.
* Gestionar grupos con `groupadd` y `groupdel`.
* Configurar contraseñas y políticas de expiración con `passwd` y `chage`.
* Asignar privilegios administrativos mediante `sudo` o `wheel`.
* Consultar información de usuarios con `id`, `who`, `w` y `last`.
* Identificar los archivos críticos relacionados con la autenticación (`/etc/passwd`, `/etc/shadow`, `/etc/group` y `/etc/gshadow`).
* Aplicar buenas prácticas de seguridad para una administración de usuarios eficiente y segura.
