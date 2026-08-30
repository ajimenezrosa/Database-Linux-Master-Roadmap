# 11. Administración de Grupos

En Linux, los **grupos** permiten organizar usuarios y administrar permisos de forma eficiente. En lugar de asignar permisos individualmente a cada usuario, es posible otorgarlos a un grupo, facilitando la administración en servidores con múltiples usuarios.

---

# Objetivos de este capítulo

Al finalizar esta unidad serás capaz de:

* Comprender qué son los grupos en Linux.
* Diferenciar entre grupos primarios y secundarios.
* Crear, modificar y eliminar grupos.
* Agregar y eliminar usuarios de grupos.
* Cambiar el grupo principal de un usuario.
* Administrar permisos mediante grupos.
* Aplicar buenas prácticas de administración.

---

# ¿Qué es un grupo?

Un grupo es una colección de usuarios que comparten permisos sobre archivos, directorios y recursos del sistema.

Un usuario puede pertenecer a:

* Un grupo principal (Primary Group)
* Uno o varios grupos secundarios (Supplementary Groups)

Ejemplo:

```text
Usuario: alejandro

Grupo principal:
    developers

Grupos secundarios:
    dba
    docker
    wheel
```

---

# ¿Por qué utilizar grupos?

Los grupos facilitan:

* Administración de permisos.
* Compartir archivos entre usuarios.
* Administración de aplicaciones.
* Seguridad.
* Escalabilidad.

En lugar de:

```text
Permiso → Usuario 1
Permiso → Usuario 2
Permiso → Usuario 3
```

Se utiliza:

```text
Permiso → Grupo DBA

Usuarios:
Juan
Pedro
María
Alejandro
```

---

# El archivo /etc/group

Todos los grupos se almacenan en:

```text
/ etc/group
```

Visualizar

```bash
cat /etc/group
```

Ejemplo

```text
wheel:x:10:root,alejandro
```

Cada línea contiene:

```text
NombreGrupo
Contraseña
GID
Miembros
```

---

# El archivo /etc/gshadow

Contiene la información protegida de los grupos.

```bash
sudo cat /etc/gshadow
```

Ejemplo

```text
wheel:!::
```

---

# Ver los grupos existentes

```bash
cat /etc/group
```

o

```bash
getent group
```

---

# Ver grupos de un usuario

```bash
groups alejandro
```

También

```bash
id alejandro
```

Resultado

```text
uid=1001(alejandro)
gid=1001(alejandro)
groups=1001(alejandro),10(wheel),995(docker)
```

---

# Crear un grupo

Sintaxis

```bash
sudo groupadd nombre_grupo
```

Ejemplo

```bash
sudo groupadd desarrolladores
```

Verificar

```bash
getent group desarrolladores
```

---

# Crear un grupo con GID específico

```bash
sudo groupadd -g 2000 dba
```

Verificar

```bash
getent group dba
```

---

# Modificar un grupo

Cambiar nombre

```bash
sudo groupmod -n administradores desarrolladores
```

---

# Cambiar el GID

```bash
sudo groupmod -g 2500 administradores
```

---

# Eliminar un grupo

```bash
sudo groupdel administradores
```

---

# Agregar un usuario a un grupo

Forma recomendada

```bash
sudo usermod -aG dba alejandro
```

Explicación

* **-a** → Append (agregar)
* **-G** → Grupo secundario

Verificar

```bash
groups alejandro
```

---

# Agregar un usuario a varios grupos

```bash
sudo usermod -aG docker,dba,wheel alejandro
```

---

# Cambiar el grupo principal

```bash
sudo usermod -g dba alejandro
```

Verificar

```bash
id alejandro
```

---

# Eliminar un usuario de un grupo

En distribuciones modernas:

```bash
sudo gpasswd -d alejandro docker
```

Resultado

```text
Removing user alejandro from group docker
```

---

# Agregar usuarios usando gpasswd

```bash
sudo gpasswd -a alejandro docker
```

---

# Administrar miembros de un grupo

Ver miembros

```bash
getent group docker
```

Resultado

```text
docker:x:995:alejandro,juan,maria
```

---

# Crear un directorio compartido

```bash
sudo mkdir /proyectos
```

Asignar grupo

```bash
sudo chgrp desarrolladores /proyectos
```

Dar permisos

```bash
sudo chmod 775 /proyectos
```

Verificar

```bash
ls -ld /proyectos
```

Resultado

```text
drwxrwxr-x
```

---

# Habilitar el bit SGID

El bit SGID hace que todos los archivos nuevos hereden automáticamente el grupo del directorio.

Aplicarlo

```bash
sudo chmod 2775 /proyectos
```

Verificar

```bash
ls -ld /proyectos
```

Resultado

```text
drwxrwsr-x
```

La letra **s** indica que el SGID está activo.

---

# Cambiar el grupo de un archivo

```bash
sudo chgrp dba respaldo.sql
```

También

```bash
sudo chown alejandro:dba respaldo.sql
```

---

# Cambiar grupo de forma recursiva

```bash
sudo chgrp -R dba /respaldos
```

---

# Buscar archivos pertenecientes a un grupo

```bash
find / -group dba
```

Buscar por GID

```bash
find / -gid 2000
```

---

# Crear un entorno colaborativo

Crear grupo

```bash
sudo groupadd proyecto
```

Agregar usuarios

```bash
sudo usermod -aG proyecto juan

sudo usermod -aG proyecto maria

sudo usermod -aG proyecto alejandro
```

Crear directorio

```bash
sudo mkdir /proyecto
```

Asignar grupo

```bash
sudo chgrp proyecto /proyecto
```

Permisos

```bash
sudo chmod 2775 /proyecto
```

Ahora todos los usuarios podrán colaborar dentro del directorio.

---

# Ver el GID de un grupo

```bash
getent group docker
```

Resultado

```text
docker:x:995:
```

Solo el GID

```bash
getent group docker | cut -d: -f3
```

---

# Cambiar temporalmente el grupo activo

```bash
newgrp dba
```

Verificar

```bash
id
```

---

# Archivos relacionados

| Archivo           | Función                     |
| ----------------- | --------------------------- |
| `/etc/group`      | Información de grupos       |
| `/etc/gshadow`    | Contraseñas de grupos       |
| `/etc/passwd`     | Grupo principal del usuario |
| `/etc/login.defs` | Configuración del sistema   |

---

# Comandos más utilizados

| Comando        | Descripción                 |
| -------------- | --------------------------- |
| `groupadd`     | Crear grupo                 |
| `groupmod`     | Modificar grupo             |
| `groupdel`     | Eliminar grupo              |
| `groups`       | Ver grupos del usuario      |
| `id`           | Ver UID y GID               |
| `gpasswd`      | Administrar miembros        |
| `getent group` | Consultar grupos            |
| `newgrp`       | Cambiar grupo activo        |
| `chgrp`        | Cambiar grupo de archivos   |
| `chown`        | Cambiar propietario y grupo |

---

# Buenas prácticas

* Utiliza grupos para administrar permisos en lugar de asignarlos usuario por usuario.
* Evita agregar usuarios innecesariamente al grupo `wheel` o `sudo`.
* Usa nombres descriptivos para los grupos (`dba`, `developers`, `backup`, `docker`).
* Implementa el bit **SGID** en directorios compartidos.
* Revisa periódicamente los miembros de cada grupo.
* Elimina grupos que ya no sean utilizados.
* Mantén una estructura de grupos consistente en todos los servidores.

---

# Laboratorio práctico

## Ejercicio 1: Crear un grupo

```bash
sudo groupadd dba
```

Verificar

```bash
getent group dba
```

---

## Ejercicio 2: Crear dos usuarios

```bash
sudo useradd -m juan
sudo passwd juan

sudo useradd -m maria
sudo passwd maria
```

---

## Ejercicio 3: Agregarlos al grupo

```bash
sudo usermod -aG dba juan

sudo usermod -aG dba maria
```

Verificar

```bash
groups juan

groups maria
```

---

## Ejercicio 4: Crear un directorio compartido

```bash
sudo mkdir /datos_compartidos

sudo chgrp dba /datos_compartidos

sudo chmod 2775 /datos_compartidos
```

Comprobar

```bash
ls -ld /datos_compartidos
```

---

## Ejercicio 5: Eliminar un usuario del grupo

```bash
sudo gpasswd -d juan dba
```

Verificar

```bash
groups juan
```

---

## Ejercicio 6: Eliminar el grupo

Primero asegúrate de que ningún usuario lo tenga como grupo principal y luego ejecuta:

```bash
sudo groupdel dba
```

---

# Errores comunes

### Error al eliminar un grupo

```text
groupdel: cannot remove the primary group of user 'alejandro'
```

**Solución:** Cambia el grupo principal del usuario antes de eliminar el grupo.

```bash
sudo usermod -g users alejandro
sudo groupdel dba
```

---

### Se reemplazaron los grupos secundarios

Si ejecutas:

```bash
sudo usermod -G docker alejandro
```

Se eliminarán los demás grupos secundarios.

**Correcto:**

```bash
sudo usermod -aG docker alejandro
```

El parámetro `-a` conserva los grupos existentes.

---

# Resumen

En este capítulo aprendiste a:

* Comprender el funcionamiento de los grupos en Linux.
* Diferenciar entre grupos primarios y secundarios.
* Crear, modificar y eliminar grupos.
* Agregar y quitar usuarios de grupos de forma segura.
* Administrar directorios compartidos utilizando permisos de grupo.
* Utilizar el bit **SGID** para facilitar el trabajo colaborativo.
* Consultar información de grupos mediante `groups`, `id` y `getent`.
* Aplicar buenas prácticas para una administración de grupos segura, organizada y escalable.
