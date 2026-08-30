# 4. Uso de la Ayuda y Documentación

![Administrador consultando la documentación de Linux desde la terminal](images/04-ayuda-documentacion-portada.jpg)

> **Objetivo del capítulo**
>
> Aprender a utilizar las herramientas de ayuda y documentación integradas en Linux para consultar comandos, opciones, archivos de configuración y manuales sin depender de Internet.

---

# Introducción

Es imposible memorizar todos los comandos, opciones y archivos de configuración de Linux.

Incluso los administradores con muchos años de experiencia consultan la documentación diariamente.

Linux incorpora una amplia colección de herramientas que permiten obtener ayuda directamente desde la terminal, incluso cuando el servidor no tiene acceso a Internet.

Aprender a utilizar estas herramientas es una habilidad fundamental para cualquier administrador de sistemas y constituye un conocimiento indispensable para la certificación **RHCSA**.

---

# ¿Por qué es importante conocer la documentación?

Durante la administración de un servidor surgirán preguntas como:

- ¿Qué hace este comando?
- ¿Qué opciones admite?
- ¿Cuál es la sintaxis correcta?
- ¿Dónde se encuentra un archivo de configuración?
- ¿Qué significa un mensaje de error?

En lugar de buscar siempre en Internet, Linux proporciona documentación integrada que responde a la mayoría de estas preguntas.

---

# Herramientas de ayuda en Linux

Las principales herramientas de documentación son:

| Herramienta | Propósito |
|-------------|-----------|
| `man` | Manual completo de comandos |
| `--help` | Ayuda rápida de un comando |
| `info` | Manuales detallados GNU |
| `whatis` | Descripción breve de un comando |
| `apropos` | Buscar comandos por palabras clave |
| `which` | Ubicación de un ejecutable |
| `whereis` | Localizar binarios, código fuente y páginas man |
| `type` | Identificar el tipo de comando |
| `help` | Ayuda para comandos internos de Bash |

---

# 1. Manuales con `man`

![Consulta de una página man](images/04-man-pages.jpg)

El comando **man** (manual) muestra la documentación oficial de un programa.

Sintaxis:

```bash
man comando
```

Ejemplo:

```bash
man ls
```

Se abrirá una página con información detallada sobre el comando.

---

## ¿Qué información contiene?

Una página del manual normalmente incluye:

- Nombre
- Descripción
- Sintaxis
- Opciones
- Ejemplos
- Variables de entorno
- Archivos relacionados
- Errores conocidos
- Autor

---

## Navegación dentro de `man`

| Tecla | Acción |
|--------|--------|
| Flecha abajo | Avanzar una línea |
| Flecha arriba | Retroceder |
| Page Down | Página siguiente |
| Page Up | Página anterior |
| Espacio | Avanzar una página |
| / | Buscar texto |
| n | Siguiente coincidencia |
| N | Coincidencia anterior |
| q | Salir |

---

# Buscar dentro del manual

Ejemplo:

```text
/
permission
```

Esto buscará todas las apariciones de la palabra **permission**.

---

# 2. Ayuda rápida con `--help`

![Uso de --help](images/04-help-option.jpg)

Muchos comandos incorporan una ayuda resumida.

Ejemplo:

```bash
ls --help
```

o

```bash
cp --help
```

Produce una salida corta con:

- Sintaxis
- Opciones principales
- Descripción rápida

Es ideal cuando solo necesitamos recordar una opción específica.

---

# ¿Cuándo usar `man` y cuándo `--help`?

| `man` | `--help` |
|--------|-----------|
| Información completa | Información resumida |
| Explicaciones detalladas | Uso rápido |
| Incluye ejemplos | Lista de opciones |
| Mejor para estudiar | Mejor para trabajar |

---

# 3. Documentación GNU con `info`

![Manual GNU Info](images/04-info-command.jpg)

Algunos programas incluyen documentación mediante:

```bash
info comando
```

Ejemplo:

```bash
info coreutils
```

Los manuales **info** suelen ser más extensos que las páginas man.

---

# 4. Descripción rápida con `whatis`

Cuando solo queremos saber qué hace un comando:

```bash
whatis passwd
```

Resultado:

```text
passwd (1) - change user password
```

---

# 5. Buscar comandos con `apropos`

![Búsqueda con apropos](images/04-apropos.jpg)

Si no recuerdas el nombre de un comando, puedes buscar por palabras clave.

Ejemplo:

```bash
apropos password
```

Resultado aproximado:

```text
passwd
chpasswd
gpasswd
```

Muy útil durante la administración diaria.

---

# 6. Localizar un ejecutable con `which`

```bash
which ls
```

Salida:

```text
/usr/bin/ls
```

Permite conocer qué programa se ejecutará cuando escribimos un comando.

---

# 7. Localizar archivos relacionados con `whereis`

Ejemplo:

```bash
whereis ssh
```

Salida aproximada:

```text
ssh:
/usr/bin/ssh
/usr/share/man/man1/ssh.1.gz
```

También puede mostrar:

- Binarios
- Código fuente
- Manuales

---

# 8. Identificar el tipo de comando

El comando:

```bash
type cd
```

permite saber si un comando es:

- Alias
- Función
- Ejecutable
- Comando interno (builtin)

Ejemplo:

```text
cd is a shell builtin
```

---

# 9. Ayuda para comandos internos de Bash

Algunos comandos no tienen página man porque pertenecen al propio Bash.

Ejemplo:

```bash
help cd
```

Otros ejemplos:

```bash
help echo
help pwd
help history
```

---

# Secciones de las páginas man

Las páginas del manual están organizadas por categorías.

| Sección | Contenido |
|----------|-----------|
| 1 | Comandos de usuario |
| 2 | Llamadas al sistema |
| 3 | Funciones de bibliotecas |
| 4 | Dispositivos |
| 5 | Formatos de archivos |
| 6 | Juegos |
| 7 | Información general |
| 8 | Administración del sistema |

Ejemplo:

```bash
man 5 passwd
```

Mostrará el formato del archivo `/etc/passwd`.

Mientras que:

```bash
man 1 passwd
```

mostrará el comando `passwd`.

---

# Documentación instalada

En muchos paquetes existe documentación adicional.

Normalmente puede encontrarse en:

```text
/usr/share/doc
```

Ejemplo:

```bash
ls /usr/share/doc
```

---

# Ejemplos prácticos

Consultar el manual de `cp`:

```bash
man cp
```

Ayuda rápida:

```bash
cp --help
```

Buscar comandos relacionados con usuarios:

```bash
apropos user
```

Conocer la ubicación de Bash:

```bash
which bash
```

Ver el tipo de `cd`:

```bash
type cd
```

Consultar ayuda de Bash:

```bash
help history
```

---

# Buenas prácticas

✅ Consultar primero `--help`.

✅ Si necesitas más información, utilizar `man`.

✅ Usar `apropos` cuando no recuerdes el nombre de un comando.

✅ Leer completamente las páginas del manual antes de ejecutar comandos críticos.

---

# Errores comunes

❌ Memorizar comandos sin comprender su funcionamiento.

❌ Buscar siempre en Internet antes de consultar la documentación local.

❌ Ignorar las páginas del manual.

❌ Ejecutar comandos encontrados en foros sin leer previamente su documentación.

---

# Laboratorio práctico

Realiza las siguientes actividades.

Consultar el manual de:

```bash
man ls
```

Consultar ayuda rápida:

```bash
ls --help
```

Buscar comandos relacionados con red:

```bash
apropos network
```

Consultar el tipo del comando:

```bash
type pwd
```

Obtener ayuda de Bash:

```bash
help alias
```

Localizar el ejecutable:

```bash
which ssh
```

---

# Preguntas de repaso

1. ¿Qué hace el comando `man`?
2. ¿Cuál es la diferencia entre `man` y `--help`?
3. ¿Para qué sirve `apropos`?
4. ¿Qué información devuelve `whatis`?
5. ¿Qué diferencia existe entre `which` y `whereis`?
6. ¿Qué comando identifica si una instrucción es un builtin de Bash?
7. ¿Qué comando muestra ayuda para los builtins de Bash?
8. ¿Dónde se encuentra normalmente la documentación instalada por los paquetes?

---

# Resumen

Linux incorpora un completo sistema de documentación integrado que permite consultar información sin necesidad de conexión a Internet.

Las herramientas **man**, **--help**, **info**, **apropos**, **whatis**, **which**, **whereis**, **type** y **help** forman parte del conjunto de utilidades que todo administrador Linux debe dominar.

Saber utilizar correctamente estas herramientas es una habilidad esencial para trabajar con sistemas Linux y una competencia evaluada durante la certificación RHCSA.

---

# Próxima lección

➡ **5. Navegación por el sistema de archivos**