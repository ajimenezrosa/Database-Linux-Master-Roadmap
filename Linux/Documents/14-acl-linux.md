# 14. ACL (Access Control Lists) en Linux

Las **ACL (Access Control Lists)** permiten asignar permisos adicionales a usuarios y grupos específicos sin modificar el propietario ni el grupo principal de un archivo o directorio.

Mientras que los permisos tradicionales de Linux solo permiten definir permisos para:

* Propietario (Owner)
* Grupo (Group)
* Otros (Others)

Las ACL permiten otorgar permisos individuales a múltiples usuarios y grupos, ofreciendo un control de acceso mucho más flexible.

---

# Objetivos de este capítulo

Al finalizar esta unidad serás capaz de:

* Comprender qué son las ACL.
* Identificar cuándo utilizar ACL.
* Consultar permisos ACL existentes.
* Agregar, modificar y eliminar ACL.
* Configurar ACL por defecto en directorios.
* Comprender la máscara (Mask) de ACL.
* Aplicar buenas prácticas de administración.

---

# ¿Qué son las ACL?

Las ACL son una extensión del sistema tradicional de permisos de Linux.

Permiten asignar permisos específicos a:

* Usuarios adicionales.
* Grupos adicionales.
* Directorios compartidos.

Sin modificar:

* Propietario.
* Grupo principal.
* Permisos tradicionales.

---

# ¿Cuándo utilizar ACL?

Las ACL son ideales cuando:

* Un único usuario necesita acceso especial.
* Varios departamentos comparten información.
* Se requiere un acceso temporal.
* No es conveniente cambiar el propietario del archivo.
* No se desea crear nuevos grupos para un caso puntual.

---

# Ejemplo sin ACL

Archivo

```text id="acl001"
reporte.xlsx
```

Permisos

```text id="acl002"
Owner
Alejandro

Group
DBA

Others
Sin acceso
```

El usuario:

```text id="acl003"
Juan
```

necesita leer el archivo.

Sin ACL habría que:

* Cambiar el grupo.
* Cambiar el propietario.
* Modificar permisos generales.

Con ACL simplemente se agrega un permiso específico para Juan.

---

# Verificar si el sistema soporta ACL

En la mayoría de las distribuciones modernas (Fedora, RHEL, Rocky Linux, AlmaLinux, Ubuntu y Debian), las ACL están habilitadas por defecto.

Puedes verificar el tipo de sistema de archivos:

```bash id="acl004"
mount | grep " / "
```

o

```bash id="acl005"
tune2fs -l /dev/sdaX | grep "Default mount options"
```

---

# Instalar herramientas ACL

Fedora / RHEL

```bash id="acl006"
sudo dnf install acl
```

Ubuntu / Debian

```bash id="acl007"
sudo apt install acl
```

Verificar

```bash id="acl008"
which getfacl

which setfacl
```

---

# Ver ACL

```bash id="acl009"
getfacl archivo.txt
```

Ejemplo

```text id="acl010"
# file: archivo.txt
# owner: alejandro
# group: dba

user::rw-
user:juan:r--
group::r--
mask::r--
other::---
```

---

# Interpretación

```text id="acl011"
user::
```

Permisos del propietario.

---

```text id="acl012"
user:juan:
```

Permisos adicionales para Juan.

---

```text id="acl013"
group::
```

Permisos del grupo.

---

```text id="acl014"
mask::
```

Límite máximo de permisos ACL.

---

```text id="acl015"
other::
```

Permisos para los demás usuarios.

---

# Agregar una ACL

Dar lectura al usuario Juan

```bash id="acl016"
setfacl -m u:juan:r archivo.txt
```

Dar lectura y escritura

```bash id="acl017"
setfacl -m u:juan:rw archivo.txt
```

Dar todos los permisos

```bash id="acl018"
setfacl -m u:juan:rwx archivo.txt
```

---

# Agregar permisos a un grupo

```bash id="acl019"
setfacl -m g:dba:rwx proyecto.sql
```

---

# Verificar

```bash id="acl020"
getfacl proyecto.sql
```

---

# Modificar una ACL

Simplemente vuelve a ejecutar:

```bash id="acl021"
setfacl -m u:juan:rw archivo.txt
```

---

# Eliminar una ACL específica

```bash id="acl022"
setfacl -x u:juan archivo.txt
```

Eliminar ACL de un grupo

```bash id="acl023"
setfacl -x g:dba archivo.txt
```

---

# Eliminar todas las ACL

```bash id="acl024"
setfacl -b archivo.txt
```

---

# ACL por defecto

Las ACL por defecto se aplican automáticamente a todos los archivos nuevos creados dentro de un directorio.

Crear directorio

```bash id="acl025"
mkdir compartido
```

Asignar ACL por defecto

```bash id="acl026"
setfacl -d -m u:juan:rwx compartido
```

Verificar

```bash id="acl027"
getfacl compartido
```

Resultado

```text id="acl028"
default:user::rwx
default:user:juan:rwx
default:group::rwx
default:mask::rwx
default:other::r-x
```

---

# Diferencia entre ACL normal y ACL por defecto

ACL normal

```text id="acl029"
Afecta únicamente el archivo actual.
```

ACL por defecto

```text id="acl030"
Se hereda automáticamente por los archivos nuevos.
```

---

# Aplicar ACL recursivamente

```bash id="acl031"
setfacl -R -m u:juan:rwx /proyectos
```

---

# Aplicar ACL por defecto recursivamente

```bash id="acl032"
setfacl -R -d -m u:juan:rwx /proyectos
```

---

# Copiar ACL

Guardar ACL

```bash id="acl033"
getfacl -R /proyecto > permisos.acl
```

Restaurar

```bash id="acl034"
setfacl --restore=permisos.acl
```

Muy útil antes de migraciones.

---

# La máscara (Mask)

La máscara limita los permisos máximos que pueden recibir los usuarios y grupos definidos mediante ACL.

Ejemplo

```text id="acl035"
user:juan:rwx

mask:r--
```

Resultado efectivo

```text id="acl036"
Juan solo podrá leer.
```

Modificar la máscara

```bash id="acl037"
setfacl -m m:rwx archivo.txt
```

---

# Ver permisos ACL en ls

```bash id="acl038"
ls -l archivo.txt
```

Resultado

```text id="acl039"
-rw-rw-r--+
```

El símbolo:

```text id="acl040"
+
```

indica que el archivo posee ACL adicionales.

---

# Buscar archivos con ACL

```bash id="acl041"
find /home -exec getfacl {} \; 2>/dev/null | grep "^user:"
```

---

# Comparación

| Permisos tradicionales | ACL                |
| ---------------------- | ------------------ |
| Owner                  | Múltiples usuarios |
| Group                  | Múltiples grupos   |
| Others                 | ACL por defecto    |
| Simple                 | Flexible           |
| Muy rápido             | Más detallado      |

---

# Casos de uso

## Compartir un archivo

Dar acceso únicamente a un usuario.

```bash id="acl042"
setfacl -m u:juan:r informe.pdf
```

---

## Equipo DBA

Todos los DBA pueden modificar respaldos.

```bash id="acl043"
setfacl -m g:dba:rwx /respaldos
```

---

## Directorio colaborativo

```bash id="acl044"
setfacl -d -m g:desarrollo:rwx /proyecto
```

---

## Acceso temporal

```bash id="acl045"
setfacl -m u:consultor:r backup.sql
```

Cuando finalice el trabajo:

```bash id="acl046"
setfacl -x u:consultor backup.sql
```

---

# Comandos más utilizados

| Comando             | Descripción                 |
| ------------------- | --------------------------- |
| `getfacl`           | Ver ACL                     |
| `setfacl`           | Configurar ACL              |
| `setfacl -m`        | Agregar o modificar ACL     |
| `setfacl -x`        | Eliminar una ACL específica |
| `setfacl -b`        | Eliminar todas las ACL      |
| `setfacl -d`        | Configurar ACL por defecto  |
| `setfacl -R`        | Aplicar recursivamente      |
| `setfacl --restore` | Restaurar ACL               |

---

# Archivos relacionados

| Archivo       | Función                                     |
| ------------- | ------------------------------------------- |
| `/etc/passwd` | Usuarios                                    |
| `/etc/group`  | Grupos                                      |
| `/etc/fstab`  | Opciones de montaje del sistema de archivos |
| `/etc/mtab`   | Sistemas de archivos montados               |

---

# Buenas prácticas

* Utiliza ACL únicamente cuando los permisos tradicionales no sean suficientes.
* Documenta las ACL aplicadas en servidores de producción.
* Revisa periódicamente los archivos con ACL mediante `getfacl`.
* Utiliza ACL por defecto en directorios colaborativos.
* Antes de migrar información, respalda las ACL con `getfacl -R`.
* Evita asignar permisos excesivos (`rwx`) si no son necesarios.
* Verifica la máscara (`mask`) cuando los permisos efectivos no coincidan con lo esperado.

---

# Laboratorio práctico

## Ejercicio 1: Crear un archivo

```bash id="labacl001"
touch informe.txt
```

---

## Ejercicio 2: Dar acceso a un usuario

```bash id="labacl002"
setfacl -m u:juan:r informe.txt
```

Verificar

```bash id="labacl003"
getfacl informe.txt
```

---

## Ejercicio 3: Crear un directorio colaborativo

```bash id="labacl004"
mkdir proyectos

setfacl -d -m g:dba:rwx proyectos
```

Comprobar

```bash id="labacl005"
getfacl proyectos
```

---

## Ejercicio 4: Eliminar una ACL

```bash id="labacl006"
setfacl -x u:juan informe.txt
```

---

## Ejercicio 5: Respaldar y restaurar ACL

Guardar

```bash id="labacl007"
getfacl -R proyectos > respaldo_acl.txt
```

Restaurar

```bash id="labacl008"
setfacl --restore=respaldo_acl.txt
```

---

## Ejercicio 6: Eliminar todas las ACL de un archivo

```bash id="labacl009"
setfacl -b informe.txt
```

---

# Errores comunes

### El usuario tiene una ACL pero no puede escribir

Verifica la máscara:

```bash id="erracl001"
getfacl archivo.txt
```

Si la línea `mask::` restringe los permisos, ajusta la máscara:

```bash id="erracl002"
setfacl -m m:rwx archivo.txt
```

---

### El archivo no muestra el símbolo `+`

Si `ls -l` no muestra un `+` al final de los permisos, el archivo no tiene ACL adicionales o el sistema de archivos no las soporta.

Compruébalo con:

```bash id="erracl003"
getfacl archivo.txt
```

---

### Las ACL no se heredan en archivos nuevos

Recuerda que las ACL normales no se heredan. Para ello debes configurar **ACL por defecto**:

```bash id="erracl004"
setfacl -d -m u:juan:rwx directorio
```

---

# Resumen

En este capítulo aprendiste a:

* Comprender el funcionamiento de las **Access Control Lists (ACL)**.
* Consultar, agregar, modificar y eliminar permisos ACL.
* Configurar ACL para usuarios y grupos específicos.
* Crear ACL por defecto para directorios compartidos.
* Entender el papel de la **máscara (Mask)** en los permisos efectivos.
* Respaldar y restaurar ACL durante migraciones.
* Aplicar buenas prácticas para administrar permisos avanzados en entornos Linux multiusuario.
