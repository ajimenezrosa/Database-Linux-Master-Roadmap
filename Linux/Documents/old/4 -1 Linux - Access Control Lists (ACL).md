# Linux - Access Control Lists (ACL)
## Apuntes de Clase

**Tema:** Access Control Lists (ACL) en Linux  
**Objetivo:** Aprender cómo asignar permisos específicos a usuarios y grupos sin modificar los permisos tradicionales de Linux.

---

# ¿Qué es ACL?

**ACL (Access Control List)** es un mecanismo adicional de permisos que se ejecuta sobre el sistema tradicional de permisos de Linux (Owner, Group y Others).

Permite otorgar permisos específicos a un usuario o grupo determinado sin necesidad de hacerlo propietario del archivo ni miembro del grupo propietario.

En otras palabras:

> ACL proporciona un mecanismo de permisos mucho más flexible que los permisos tradicionales de Unix.

---

# ¿Por qué existe ACL?

Supongamos el siguiente escenario:

Tenemos un archivo propiedad de **root**.

```
-rw-r--r-- 1 root root 250 Jul 23 archivo.txt
```

Queremos que únicamente el usuario **juan** pueda modificar ese archivo.

## Sin ACL

La única opción sería modificar los permisos de "Others":

```
-rw-rw-rw-
```

El problema es que **todos los usuarios** podrían modificar el archivo.

Eso representa un problema de seguridad.

---

## Con ACL

Podemos permitir únicamente al usuario **juan**:

- Leer
- Escribir

Mientras que el resto de usuarios continúan sin permisos adicionales.

Eso es precisamente el propósito de ACL.

---

# Ventajas de ACL

- Permisos mucho más flexibles.
- Permisos por usuario.
- Permisos por grupo específico.
- No es necesario modificar Owner ni Group.
- No es necesario agregar usuarios a grupos.
- Ideal para ambientes empresariales.

---

# Comandos ACL

## Ver permisos ACL

```bash
getfacl archivo
```

Ejemplo:

```bash
getfacl /tmp/TX
```

Salida:

```text
# file: TX
# owner: root
# group: root

user::rw-
group::r--
other::r--
```

---

## Asignar permisos a un usuario

Sintaxis

```bash
setfacl -m u:usuario:permisos archivo
```

Ejemplo

```bash
sudo setfacl -m u:juan:rw /tmp/TX
```

Significado:

- **-m** = modificar ACL
- **u** = usuario
- **juan** = usuario
- **rw** = lectura y escritura

---

## Asignar permisos a un grupo

Sintaxis

```bash
setfacl -m g:grupo:permisos archivo
```

Ejemplo

```bash
sudo setfacl -m g:desarrollo:rwx /tmp/TX
```

---

## Permisos heredados (Default ACL)

Permite que todos los archivos creados dentro de un directorio hereden automáticamente los permisos ACL.

Sintaxis

```bash
setfacl -dm u:usuario:rwx directorio
```

Ejemplo

```bash
sudo setfacl -dm u:juan:rwx /proyectos
```

Todo archivo nuevo dentro de **/proyectos** heredará esos permisos.

---

## Eliminar una ACL específica

Sintaxis

```bash
setfacl -x u:usuario archivo
```

Ejemplo

```bash
sudo setfacl -x u:juan /tmp/TX
```

---

## Eliminar todas las ACL

Sintaxis

```bash
setfacl -b archivo
```

Ejemplo

```bash
sudo setfacl -b /tmp/TX
```

El archivo vuelve únicamente a los permisos tradicionales.

---

# Verificar permisos ACL

Siempre utilizar:

```bash
getfacl archivo
```

Ejemplo:

```bash
getfacl /tmp/TX
```

Salida

```text
user::rw-
user:juan:rw-
group::r--
mask::rw-
other::r--
```

Aquí podemos observar que **juan** posee permisos especiales.

---

# Identificar archivos con ACL

Cuando ejecutamos:

```bash
ls -l
```

Veremos algo similar a:

```text
-rw-rw-r--+ 1 root root 250 Jul 23 TX
```

Observe el signo:

```
+
```

Ese **+** indica que el archivo tiene ACL configuradas.

---

# Ejemplo práctico

## Crear archivo

```bash
touch /tmp/TX
```

---

## Ver permisos

```bash
ls -l /tmp/TX
```

Resultado

```text
-rw-r--r-- 1 root root
```

---

## Consultar ACL

```bash
getfacl /tmp/TX
```

---

## Dar permisos de lectura y escritura al usuario juan

```bash
sudo setfacl -m u:juan:rw /tmp/TX
```

---

## Verificar

```bash
getfacl /tmp/TX
```

Resultado

```text
user:juan:rw-
```

---

Ahora el usuario **juan** podrá editar el archivo sin ser propietario.

---

# Permisos para grupos

Ejemplo

```bash
sudo setfacl -m g:ventas:rwx /tmp/TX
```

Todos los usuarios pertenecientes al grupo **ventas** tendrán esos permisos.

---

# Eliminar permisos ACL del usuario

```bash
sudo setfacl -x u:juan /tmp/TX
```

---

# Eliminar todas las ACL

```bash
sudo setfacl -b /tmp/TX
```

---

# ACL NO permite eliminar archivos

Aunque un usuario tenga:

```
rw
```

sobre un archivo mediante ACL,

**NO podrá eliminarlo**.

Ejemplo

```bash
rm /tmp/TX
```

Resultado

```text
Operation not permitted
```

¿Por qué?

Porque el permiso para eliminar depende del directorio que contiene el archivo, no únicamente del archivo en sí.

ACL permite:

- Leer
- Escribir
- Modificar

Pero no convierte al usuario en propietario.

---

# Resumen de comandos

| Acción | Comando |
|---------|----------|
| Ver ACL | `getfacl archivo` |
| Agregar ACL usuario | `setfacl -m u:usuario:rw archivo` |
| Agregar ACL grupo | `setfacl -m g:grupo:rwx archivo` |
| ACL heredada | `setfacl -dm u:usuario:rwx directorio` |
| Eliminar ACL usuario | `setfacl -x u:usuario archivo` |
| Eliminar todas las ACL | `setfacl -b archivo` |

---

# Conceptos importantes

- ACL significa **Access Control List**.
- Es una capa adicional sobre los permisos tradicionales de Linux.
- Permite asignar permisos específicos a usuarios individuales.
- No requiere cambiar propietario ni grupo.
- No requiere agregar usuarios a grupos.
- Se administra mediante:
  - `setfacl`
  - `getfacl`
- Los archivos con ACL muestran un **+** al final de sus permisos en `ls -l`.
- ACL ofrece un control de acceso más granular y flexible.

---

# Conclusión

Las **Access Control Lists (ACL)** son una herramienta fundamental para administrar permisos avanzados en Linux. Permiten otorgar acceso específico a usuarios o grupos sin alterar los permisos tradicionales del sistema, facilitando una administración más segura y flexible de archivos y directorios, especialmente en entornos multiusuario y empresariales.