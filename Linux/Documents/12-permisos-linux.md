# 12. Permisos en Linux

Los permisos son uno de los pilares de la seguridad en Linux. Determinan **quién puede leer, modificar o ejecutar** archivos y directorios, permitiendo proteger la información y controlar el acceso a los recursos del sistema.

Comprender cómo funcionan los permisos es fundamental para cualquier administrador Linux, ya que prácticamente todas las tareas relacionadas con usuarios, aplicaciones, servicios y servidores dependen de una correcta administración de permisos.

---

# Objetivos de este capítulo

Al finalizar esta unidad serás capaz de:

* Comprender cómo funciona el modelo de permisos de Linux.
* Interpretar permisos simbólicos y numéricos.
* Modificar permisos utilizando `chmod`.
* Cambiar propietarios con `chown`.
* Cambiar grupos con `chgrp`.
* Comprender los permisos especiales (SUID, SGID y Sticky Bit).
* Aplicar permisos de forma recursiva.
* Implementar buenas prácticas de seguridad.

---

# ¿Qué son los permisos?

Cada archivo y directorio posee tres tipos de permisos:

* Lectura (Read)
* Escritura (Write)
* Ejecución (Execute)

Y dichos permisos pueden asignarse a tres tipos de usuarios:

* Propietario (Owner)
* Grupo (Group)
* Otros usuarios (Others)

Visualmente:

```text id="3whv7r"
                 Permisos

             r   w   x
Owner        ✓   ✓   ✓

Group        ✓   ✓   -

Others       ✓   -   -
```

---

# Ver permisos

```bash id="kdm7q3"
ls -l
```

Ejemplo

```text id="w9q7lm"
-rwxr-xr-- 1 alejandro dba 2540 Jul 25 respaldo.sh
```

---

# Interpretando la salida

```text id="mbdlyw"
-rwxr-xr--
```

Se divide en cuatro partes.

```text id="p87jpf"
-

rwx

r-x

r--
```

---

## Primer carácter

Indica el tipo de archivo.

| Símbolo | Significado               |
| ------- | ------------------------- |
| -       | Archivo                   |
| d       | Directorio                |
| l       | Enlace simbólico          |
| c       | Dispositivo de caracteres |
| b       | Dispositivo de bloques    |
| s       | Socket                    |
| p       | Pipe                      |

Ejemplos

```text id="ubv0c8"
-rw-r--r--
```

Archivo

```text id="vfrl2z"
drwxr-xr-x
```

Directorio

```text id="b3rvp4"
lrwxrwxrwx
```

Enlace simbólico

---

# Permisos

Los siguientes nueve caracteres representan los permisos.

```text id="nqarf0"
rwx

r-x

r--
```

---

## Owner

```text id="kwv5n0"
rwx
```

El propietario puede:

* Leer
* Escribir
* Ejecutar

---

## Group

```text id="j6mjlwm"
r-x
```

Puede:

* Leer
* Ejecutar

---

## Others

```text id="0kzomk"
r--
```

Puede:

* Leer únicamente

---

# Significado de cada permiso

## Read (r)

Valor

```text id="l2x5g8"
4
```

Permite:

* Leer archivos.
* Ver contenido de directorios.

---

## Write (w)

Valor

```text id="8o7wz2"
2
```

Permite:

* Modificar archivos.
* Crear y eliminar archivos dentro de un directorio (junto con el permiso de ejecución sobre el directorio).

---

## Execute (x)

Valor

```text id="phwocv"
1
```

Permite:

* Ejecutar programas.
* Acceder (entrar) a directorios.

---

# Permisos numéricos

Cada permiso posee un valor.

| Permiso | Valor |
| ------- | ----- |
| Read    | 4     |
| Write   | 2     |
| Execute | 1     |

Se suman.

Ejemplo

```text id="5rjlwm"
Read
4

Write
2

Execute
1

Total
7
```

---

# Tabla de permisos

| Número | Permiso | Significado         |
| ------ | ------- | ------------------- |
| 0      | ---     | Ningún permiso      |
| 1      | --x     | Ejecutar            |
| 2      | -w-     | Escribir            |
| 3      | -wx     | Escribir y ejecutar |
| 4      | r--     | Leer                |
| 5      | r-x     | Leer y ejecutar     |
| 6      | rw-     | Leer y escribir     |
| 7      | rwx     | Control total       |

---

# Ejemplo práctico

```text id="rk18jv"
754
```

Equivale a

```text id="jsz0tq"
Owner
7

Group
5

Others
4
```

Resultado

```text id="w1emxu"
rwxr-xr--
```

---

# Cambiar permisos con chmod

Sintaxis

```bash id="n7vgcq"
chmod permisos archivo
```

Ejemplo

```bash id="xk87j2"
chmod 755 script.sh
```

Verificar

```bash id="p1zh1o"
ls -l script.sh
```

---

# Cambiar permisos simbólicos

Agregar ejecución

```bash id="3ax8y9"
chmod +x script.sh
```

Quitar escritura

```bash id="l3j68x"
chmod -w archivo.txt
```

Agregar lectura al grupo

```bash id="2i6m0d"
chmod g+r archivo.txt
```

Agregar ejecución al propietario

```bash id="e0kx3g"
chmod u+x script.sh
```

Quitar permisos a otros

```bash id="x7zjlwm"
chmod o-rwx archivo.txt
```

---

# Letras utilizadas

| Letra | Significado |
| ----- | ----------- |
| u     | User        |
| g     | Group       |
| o     | Others      |
| a     | All         |

Ejemplos

```bash id="sxpvh3"
chmod a+r archivo.txt
```

```bash id="2rvyoh"
chmod g-w archivo.txt
```

```bash id="9j2d4b"
chmod u+x respaldo.sh
```

---

# Cambiar permisos recursivamente

```bash id="5z8l31"
chmod -R 755 /proyecto
```

Con cuidado.

---

# Cambiar propietario

```bash id="qp6d0g"
sudo chown juan archivo.txt
```

Cambiar propietario y grupo

```bash id="bjlwm9"
sudo chown juan:dba archivo.txt
```

Recursivamente

```bash id="0gqfvs"
sudo chown -R juan:dba /proyecto
```

---

# Cambiar grupo

```bash id="5a4jgl"
sudo chgrp dba respaldo.sql
```

---

# Permisos de directorios

Para directorios los permisos significan algo diferente.

| Permiso | Directorios               |
| ------- | ------------------------- |
| Read    | Ver contenido             |
| Write   | Crear y eliminar archivos |
| Execute | Entrar al directorio      |

---

# Ejemplo

```bash id="gxvjlwm"
mkdir datos
chmod 700 datos
```

Solo el propietario puede acceder.

---

# Permisos recomendados

## Scripts

```text id="jlwm84"
755
```

---

## Archivos normales

```text id="jlwm51"
644
```

---

## Directorios

```text id="jlwm73"
755
```

---

## Archivos privados

```text id="jlwm64"
600
```

---

# Umask

La **umask** determina los permisos predeterminados para archivos y directorios recién creados.

Ver la umask actual:

```bash id="8g1h3r"
umask
```

Ejemplo de salida:

```text id="jlwm95"
0022
```

Con una umask de `022`:

* Archivos nuevos → `644`
* Directorios nuevos → `755`

Cambiar temporalmente la umask:

```bash id="jlwm96"
umask 027
```

---

# Permisos especiales

Linux posee tres permisos especiales.

* SUID
* SGID
* Sticky Bit

---

# SUID

Permite ejecutar un programa con los permisos del propietario.

Aplicarlo

```bash id="jlwm97"
chmod u+s programa
```

Verificar

```bash id="jlwm98"
ls -l programa
```

Resultado

```text id="jlwm99"
-rwsr-xr-x
```

---

# SGID

En archivos:

Ejecuta con los permisos del grupo.

En directorios:

Los archivos nuevos heredan el grupo.

Aplicarlo

```bash id="jlwm100"
chmod g+s proyecto
```

Resultado

```text id="jlwm101"
drwxrwsr-x
```

---

# Sticky Bit

Evita que usuarios eliminen archivos de otros dentro de un directorio compartido.

Ejemplo típico

```text id="jlwm102"
/tmp
```

Aplicarlo

```bash id="jlwm103"
chmod +t compartido
```

Resultado

```text id="jlwm104"
drwxrwxrwt
```

---

# Permisos especiales en formato numérico

| Valor | Permiso    |
| ----- | ---------- |
| 4     | SUID       |
| 2     | SGID       |
| 1     | Sticky Bit |

Ejemplos

```bash id="jlwm105"
chmod 4755 programa
```

```bash id="jlwm106"
chmod 2775 proyecto
```

```bash id="jlwm107"
chmod 1777 /compartido
```

---

# Buscar archivos con permisos especiales

Buscar SUID

```bash id="jlwm108"
find / -perm -4000
```

Buscar SGID

```bash id="jlwm109"
find / -perm -2000
```

Buscar Sticky Bit

```bash id="jlwm110"
find / -perm -1000
```

---

# Permisos por defecto mediante ACL (Introducción)

Además de los permisos tradicionales, Linux permite usar **ACL (Access Control Lists)** para asignar permisos específicos a usuarios o grupos adicionales.

Ver ACL:

```bash id="jlwm111"
getfacl archivo.txt
```

Asignar permiso de lectura a un usuario:

```bash id="jlwm112"
setfacl -m u:juan:r archivo.txt
```

> Las ACL se estudiarán con mayor profundidad en un capítulo posterior.

---

# Archivos relacionados

| Archivo           | Función                         |
| ----------------- | ------------------------------- |
| `/etc/passwd`     | Usuarios                        |
| `/etc/group`      | Grupos                          |
| `/etc/login.defs` | Configuración de cuentas        |
| `/etc/skel`       | Plantillas para nuevos usuarios |

---

# Comandos más utilizados

| Comando   | Descripción                             |
| --------- | --------------------------------------- |
| `ls -l`   | Ver permisos                            |
| `chmod`   | Cambiar permisos                        |
| `chown`   | Cambiar propietario                     |
| `chgrp`   | Cambiar grupo                           |
| `umask`   | Ver o cambiar permisos por defecto      |
| `stat`    | Ver información detallada de un archivo |
| `getfacl` | Consultar ACL                           |
| `setfacl` | Configurar ACL                          |

---

# Buenas prácticas

* Asigna siempre el principio del **mínimo privilegio**.
* Evita utilizar `777`, salvo casos muy específicos y controlados.
* Usa `755` para directorios y scripts públicos.
* Usa `644` para archivos normales.
* Protege claves privadas y credenciales con `600`.
* Revisa periódicamente los archivos con permisos SUID y SGID.
* Utiliza grupos para compartir recursos en lugar de otorgar permisos a todos los usuarios.
* Comprueba los permisos con `ls -l` antes de modificar archivos críticos.

---

# Laboratorio práctico

## Ejercicio 1: Crear un archivo y modificar permisos

```bash id="jlwm113"
touch reporte.txt

chmod 644 reporte.txt

ls -l reporte.txt
```

---

## Ejercicio 2: Crear un script ejecutable

```bash id="jlwm114"
touch respaldo.sh

chmod +x respaldo.sh

ls -l respaldo.sh
```

---

## Ejercicio 3: Cambiar propietario y grupo

```bash id="jlwm115"
sudo chown juan:dba reporte.txt

ls -l reporte.txt
```

---

## Ejercicio 4: Crear un directorio compartido con SGID

```bash id="jlwm116"
sudo mkdir /proyecto

sudo chgrp dba /proyecto

sudo chmod 2775 /proyecto

ls -ld /proyecto
```

---

## Ejercicio 5: Configurar Sticky Bit

```bash id="jlwm117"
sudo mkdir /compartido

sudo chmod 1777 /compartido

ls -ld /compartido
```

---

## Ejercicio 6: Buscar archivos con SUID

```bash id="jlwm118"
find / -perm -4000 2>/dev/null
```

---

# Resumen

En este capítulo aprendiste a:

* Comprender el modelo de permisos de Linux.
* Interpretar permisos simbólicos y numéricos.
* Modificar permisos con `chmod`.
* Cambiar propietarios y grupos mediante `chown` y `chgrp`.
* Entender el funcionamiento de la `umask`.
* Utilizar los permisos especiales **SUID**, **SGID** y **Sticky Bit**.
* Aplicar permisos de forma recursiva y segura.
* Implementar buenas prácticas para proteger archivos, directorios y recursos del sistema.
