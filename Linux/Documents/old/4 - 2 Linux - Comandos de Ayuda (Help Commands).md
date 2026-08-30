# Linux - Comandos de Ayuda (Help Commands)
## Apuntes de Clase

**Tema:** Help Commands en Linux  
**Objetivo:** Aprender a consultar la documentación y ayuda de los comandos de Linux utilizando las herramientas integradas del sistema.

---

# Introducción

Linux cuenta con diversas herramientas que permiten consultar la documentación de los comandos sin necesidad de acceder a Internet.

Estas herramientas son fundamentales para administradores de sistemas y desarrolladores, ya que permiten conocer:

- Qué hace un comando.
- Su sintaxis.
- Sus opciones.
- Ejemplos de uso.
- Información detallada del comando.

Los tres comandos de ayuda más utilizados son:

1. `whatis`
2. `--help`
3. `man`

---

# 1. Comando `whatis`

## ¿Qué es?

El comando **whatis** muestra una descripción muy breve de un comando.

Es ideal cuando solo queremos recordar rápidamente para qué sirve un comando.

## Sintaxis

```bash
whatis comando
```

---

## Ejemplo 1

```bash
whatis ls
```

Salida

```text
ls (1) - list directory contents
```

Significado:

El comando **ls** lista el contenido de un directorio.

---

## Ejemplo 2

```bash
whatis pwd
```

Salida

```text
pwd (1) - print name of current working directory
```

Significado:

Muestra el directorio actual donde nos encontramos.

---

## Ejemplo 3

```bash
whatis cd
```

Salida

```text
cd (1) - shell builtin command
```

El resultado indica que **cd** es un comando interno (Built-in) del shell Bash.

---

# ¿Por qué algunos comandos muestran varias descripciones?

Algunos comandos poseen más de una página de documentación (Manual Pages), por lo que `whatis` puede mostrar varias definiciones provenientes de distintas fuentes instaladas en el sistema.

---

# 2. Opción `--help`

## ¿Qué es?

Muchos comandos de Linux incluyen la opción:

```bash
--help
```

Esta opción muestra una ayuda más extensa que `whatis`, incluyendo:

- Sintaxis
- Opciones disponibles
- Parámetros
- Ejemplos rápidos

---

## Sintaxis

```bash
comando --help
```

---

## Ejemplo con `ls`

```bash
ls --help
```

Salida (resumida)

```text
Usage: ls [OPTION]... [FILE]...

List information about the FILEs.
```

Luego aparecen todas las opciones disponibles:

```
-a
-l
-h
-R
-t
-r
...
```

---

## Ejemplo con `chmod`

```bash
chmod --help
```

Salida (resumida)

```text
Change the mode of each FILE to MODE.
```

Luego muestra todas las opciones disponibles para cambiar permisos.

---

## Ventajas de `--help`

- Muy rápido.
- No requiere abrir el manual completo.
- Excelente para recordar opciones.

---

# 3. Comando `man`

## ¿Qué significa?

`man` significa:

**Manual**

Es la documentación oficial de prácticamente todos los comandos de Linux.

---

## Sintaxis

```bash
man comando
```

---

## Ejemplo

```bash
man ls
```

El sistema abrirá una página completa con información organizada.

---

## Secciones de un Manual

Generalmente un manual contiene:

- NAME
- SYNOPSIS
- DESCRIPTION
- OPTIONS
- EXAMPLES
- AUTHOR
- BUGS
- SEE ALSO

---

## Ejemplo

```bash
man pwd
```

Mostrará información completa acerca del comando `pwd`.

---

# Navegación dentro de `man`

Una vez abierto el manual podemos utilizar las siguientes teclas:

| Tecla | Acción |
|--------|--------|
| Espacio | Avanzar una página |
| Enter | Avanzar una línea |
| b | Retroceder una página |
| ↑ | Línea anterior |
| ↓ | Línea siguiente |
| /texto | Buscar una palabra |
| n | Buscar siguiente coincidencia |
| N | Buscar coincidencia anterior |
| q | Salir del manual |

---

## Buscar dentro del manual

Por ejemplo:

```
/permission
```

Luego presionar Enter.

Para continuar buscando:

```
n
```

---

# Comparación de los tres comandos

| Comando | Nivel de información | Uso recomendado |
|----------|----------------------|-----------------|
| `whatis` | Muy breve | Saber rápidamente qué hace un comando |
| `--help` | Intermedio | Recordar sintaxis y opciones |
| `man` | Completo | Aprender completamente un comando |

---

# Ejemplos prácticos

## Saber qué hace `cp`

```bash
whatis cp
```

---

## Ver ayuda rápida

```bash
cp --help
```

---

## Abrir el manual completo

```bash
man cp
```

---

## Consultar ayuda para `mkdir`

```bash
whatis mkdir
mkdir --help
man mkdir
```

---

## Consultar ayuda para `rm`

```bash
whatis rm
rm --help
man rm
```

---

## Consultar ayuda para `chmod`

```bash
whatis chmod
chmod --help
man chmod
```

---

## Consultar ayuda para `chown`

```bash
whatis chown
chown --help
man chown
```

---

# Buenas prácticas

Cuando no recuerdes un comando:

1. Usa primero:

```bash
whatis comando
```

Si necesitas conocer las opciones:

```bash
comando --help
```

Si deseas comprender completamente el comando:

```bash
man comando
```

---

# Ejemplo de flujo de trabajo

Supongamos que olvidamos cómo funciona `tar`.

Primero:

```bash
whatis tar
```

Luego:

```bash
tar --help
```

Finalmente:

```bash
man tar
```

Este flujo permite ir desde una explicación rápida hasta una documentación completa.

---

# Resumen

| Comando | Descripción |
|----------|-------------|
| `whatis` | Muestra una descripción breve del comando. |
| `comando --help` | Muestra ayuda rápida con las opciones disponibles. |
| `man comando` | Abre el manual completo del comando. |

---

# Conclusión

Los comandos de ayuda son herramientas esenciales para cualquier usuario de Linux. Permiten aprender nuevos comandos, recordar opciones y consultar la documentación oficial directamente desde la terminal.

El comando **`whatis`** ofrece una descripción rápida, **`--help`** proporciona una ayuda práctica con las opciones más comunes y **`man`** brinda una documentación completa y detallada. Dominar estas herramientas mejora la productividad y facilita el aprendizaje continuo del sistema operativo Linux.