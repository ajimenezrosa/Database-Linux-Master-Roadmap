# 13. Permisos Especiales en Linux

Los permisos especiales son una extensión del modelo tradicional de permisos de Linux y permiten modificar el comportamiento de archivos y directorios en situaciones específicas.

Existen tres permisos especiales:

* **SUID (Set User ID)**
* **SGID (Set Group ID)**
* **Sticky Bit**

Estos permisos son ampliamente utilizados en sistemas Linux para mejorar la administración de recursos compartidos y permitir que determinados programas se ejecuten con privilegios específicos.

---

# Objetivos de este capítulo

Al finalizar esta unidad serás capaz de:

* Comprender qué son los permisos especiales.
* Diferenciar entre SUID, SGID y Sticky Bit.
* Configurar cada permiso especial.
* Interpretar su representación simbólica y numérica.
* Identificar riesgos de seguridad asociados.
* Localizar archivos con permisos especiales.
* Aplicar buenas prácticas de administración.

---

# ¿Qué son los permisos especiales?

Los permisos tradicionales controlan:

* Lectura
* Escritura
* Ejecución

Los permisos especiales modifican ese comportamiento.

Los tres permisos especiales son:

| Permiso    | Función                                    |
| ---------- | ------------------------------------------ |
| SUID       | Ejecutar como propietario                  |
| SGID       | Ejecutar como grupo o heredar grupo        |
| Sticky Bit | Evitar eliminar archivos de otros usuarios |

---

# Representación numérica

Los permisos especiales utilizan un cuarto dígito.

| Valor | Permiso    |
| ----- | ---------- |
| 4     | SUID       |
| 2     | SGID       |
| 1     | Sticky Bit |

Ejemplos

```text id="suid001"
4755
2755
1777
6755
```

---

# Permiso SUID

## ¿Qué hace?

Cuando un usuario ejecuta un programa con SUID, dicho programa se ejecuta utilizando los permisos del propietario del archivo y no del usuario que lo ejecuta.

Es útil cuando un programa necesita realizar tareas privilegiadas sin que el usuario tenga permisos administrativos.

---

# Ejemplo clásico

El comando:

```bash id="suid002"
passwd
```

Permite cambiar la contraseña del usuario.

Sin SUID no podría modificar:

```text id="suid003"
/etc/shadow
```

porque dicho archivo solo puede ser modificado por **root**.

---

# Verificar SUID

```bash id="suid004"
ls -l /usr/bin/passwd
```

Resultado

```text id="suid005"
-rwsr-xr-x
```

Observa la letra:

```text id="suid006"
s
```

en lugar de:

```text id="suid007"
x
```

---

# Aplicar SUID

```bash id="suid008"
chmod u+s programa
```

O mediante notación numérica

```bash id="suid009"
chmod 4755 programa
```

---

# Eliminar SUID

```bash id="suid010"
chmod u-s programa
```

o

```bash id="suid011"
chmod 0755 programa
```

---

# Riesgos del SUID

Un programa mal diseñado con SUID puede permitir:

* Escalada de privilegios.
* Ejecución de código como root.
* Acceso a información sensible.

Nunca debe asignarse SUID a programas desconocidos.

---

# Buscar archivos con SUID

```bash id="suid012"
find / -perm -4000 2>/dev/null
```

Ejemplo de salida

```text id="suid013"
/usr/bin/passwd
/usr/bin/su
/usr/bin/chsh
/usr/bin/newgrp
```

---

# Permiso SGID

El comportamiento cambia dependiendo del tipo de archivo.

---

# SGID en archivos

Cuando un archivo ejecutable posee SGID, se ejecuta utilizando los permisos del grupo propietario.

Aplicarlo

```bash id="sgid001"
chmod g+s programa
```

---

# SGID en directorios

Es el uso más frecuente.

Todos los archivos creados dentro del directorio heredarán automáticamente el grupo del directorio.

---

# Ejemplo

Crear directorio

```bash id="sgid002"
sudo mkdir /proyecto
```

Asignar grupo

```bash id="sgid003"
sudo chgrp desarrolladores /proyecto
```

Aplicar SGID

```bash id="sgid004"
sudo chmod 2775 /proyecto
```

Verificar

```bash id="sgid005"
ls -ld /proyecto
```

Resultado

```text id="sgid006"
drwxrwsr-x
```

La letra:

```text id="sgid007"
s
```

indica que SGID está activo.

---

# Ejemplo práctico

Sin SGID

```text id="sgid008"
Usuario Juan crea archivo

Grupo:
juan
```

Con SGID

```text id="sgid009"
Usuario Juan crea archivo

Grupo:
desarrolladores
```

Todos los archivos pertenecerán al grupo compartido.

---

# Quitar SGID

```bash id="sgid010"
chmod g-s directorio
```

---

# Buscar archivos SGID

```bash id="sgid011"
find / -perm -2000 2>/dev/null
```

---

# Sticky Bit

Es utilizado principalmente en directorios compartidos.

Impide que un usuario elimine archivos pertenecientes a otro usuario.

---

# Ejemplo

El directorio:

```text id="sticky001"
/tmp
```

posee Sticky Bit.

Todos pueden crear archivos.

Pero únicamente el propietario puede eliminarlos.

---

# Verificar

```bash id="sticky002"
ls -ld /tmp
```

Resultado

```text id="sticky003"
drwxrwxrwt
```

La letra:

```text id="sticky004"
t
```

indica Sticky Bit.

---

# Crear un directorio compartido

```bash id="sticky005"
sudo mkdir /compartido
```

Permisos

```bash id="sticky006"
sudo chmod 1777 /compartido
```

Verificar

```bash id="sticky007"
ls -ld /compartido
```

Resultado

```text id="sticky008"
drwxrwxrwt
```

---

# Eliminar Sticky Bit

```bash id="sticky009"
chmod -t /compartido
```

---

# Comparación

| Permiso    | Archivos                 | Directorios                    |
| ---------- | ------------------------ | ------------------------------ |
| SUID       | Ejecuta como propietario | No suele utilizarse            |
| SGID       | Ejecuta como grupo       | Hereda el grupo                |
| Sticky Bit | Sin uso práctico         | Evita eliminar archivos ajenos |

---

# Representación simbólica

Ejemplo

```text id="tabla001"
-rwsr-xr-x
```

Significa

```text id="tabla002"
Owner:
rws

Group:
r-x

Others:
r-x
```

---

Otro ejemplo

```text id="tabla003"
drwxrwsr-x
```

SGID activo.

---

Otro ejemplo

```text id="tabla004"
drwxrwxrwt
```

Sticky Bit activo.

---

# Representación numérica

| Permisos | Significado                  |
| -------- | ---------------------------- |
| 4755     | SUID                         |
| 2755     | SGID                         |
| 1755     | Sticky Bit                   |
| 6755     | SUID + SGID                  |
| 7755     | Los tres permisos especiales |

---

# Ver permisos especiales

```bash id="tabla005"
stat archivo
```

También

```bash id="tabla006"
ls -l archivo
```

---

# Buscar todos los permisos especiales

```bash id="tabla007"
find / \( -perm -4000 -o -perm -2000 -o -perm -1000 \) 2>/dev/null
```

---

# Riesgos de seguridad

Evita asignar permisos especiales a:

* Scripts Bash.
* Programas descargados de Internet.
* Archivos desconocidos.
* Binarios sin verificar.

Revisa periódicamente:

```bash id="tabla008"
find / -perm -4000 2>/dev/null
```

---

# Casos de uso

## SUID

* `passwd`
* `su`
* `mount`
* `umount`

---

## SGID

* Directorios compartidos.
* Equipos de desarrollo.
* Equipos DBA.
* Repositorios compartidos.

---

## Sticky Bit

* `/tmp`
* Directorios públicos.
* Carpetas compartidas entre usuarios.

---

# Archivos relacionados

| Archivo       | Función                              |
| ------------- | ------------------------------------ |
| `/etc/passwd` | Usuarios                             |
| `/etc/group`  | Grupos                               |
| `/etc/shadow` | Contraseñas                          |
| `/tmp`        | Directorio compartido con Sticky Bit |

---

# Comandos más utilizados

| Comando            | Descripción           |
| ------------------ | --------------------- |
| `chmod u+s`        | Aplicar SUID          |
| `chmod g+s`        | Aplicar SGID          |
| `chmod +t`         | Aplicar Sticky Bit    |
| `find -perm -4000` | Buscar SUID           |
| `find -perm -2000` | Buscar SGID           |
| `find -perm -1000` | Buscar Sticky Bit     |
| `ls -l`            | Ver permisos          |
| `stat`             | Información detallada |

---

# Buenas prácticas

* Aplica **SUID** únicamente a binarios confiables y necesarios.
* Utiliza **SGID** en directorios donde varios usuarios colaboran.
* Emplea **Sticky Bit** en directorios públicos para evitar eliminaciones accidentales o maliciosas.
* Revisa periódicamente los binarios con permisos especiales.
* Evita asignar permisos especiales a scripts o aplicaciones de origen desconocido.
* Documenta cualquier cambio relacionado con permisos especiales en servidores de producción.

---

# Laboratorio práctico

## Ejercicio 1: Identificar programas con SUID

```bash id="lab001"
find / -perm -4000 2>/dev/null
```

---

## Ejercicio 2: Crear un directorio con SGID

```bash id="lab002"
sudo groupadd desarrollo

sudo mkdir /desarrollo

sudo chgrp desarrollo /desarrollo

sudo chmod 2775 /desarrollo

ls -ld /desarrollo
```

---

## Ejercicio 3: Crear un directorio con Sticky Bit

```bash id="lab003"
sudo mkdir /publico

sudo chmod 1777 /publico

ls -ld /publico
```

---

## Ejercicio 4: Quitar un permiso especial

```bash id="lab004"
chmod g-s /desarrollo

chmod -t /publico
```

---

## Ejercicio 5: Buscar todos los archivos con permisos especiales

```bash id="lab005"
find / \( -perm -4000 -o -perm -2000 -o -perm -1000 \) 2>/dev/null
```

---

# Errores comunes

### Aplicar SUID a un script

```bash id="err001"
chmod 4755 respaldo.sh
```

En la mayoría de las distribuciones Linux, el bit **SUID no tiene efecto sobre scripts interpretados** (como Bash o Python) por motivos de seguridad.

---

### Olvidar el Sticky Bit en un directorio compartido

Si un directorio tiene permisos `777` pero no tiene Sticky Bit:

```bash id="err002"
chmod 777 /compartido
```

Cualquier usuario podrá eliminar archivos de otros usuarios.

La configuración correcta es:

```bash id="err003"
chmod 1777 /compartido
```

---

# Resumen

En este capítulo aprendiste a:

* Comprender el propósito de los permisos especiales en Linux.
* Configurar y administrar **SUID**, **SGID** y **Sticky Bit**.
* Interpretar sus representaciones simbólicas y numéricas.
* Localizar archivos y directorios con permisos especiales mediante `find`.
* Identificar los riesgos de seguridad asociados con un uso inadecuado.
* Implementar buenas prácticas para proteger sistemas multiusuario y entornos colaborativos.
