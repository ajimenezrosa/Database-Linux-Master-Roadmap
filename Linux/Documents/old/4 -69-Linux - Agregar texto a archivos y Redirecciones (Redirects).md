# Linux - Agregar texto a archivos y Redirecciones (Redirects)
## Apuntes de Clase

**Tema:** Agregar contenido a archivos utilizando `echo` y redirecciones (`>` y `>>`)

**Objetivo:** Aprender diferentes formas de escribir información dentro de un archivo en Linux utilizando comandos desde la terminal.

---

# Introducción

En Linux, el comando `touch` únicamente crea archivos vacíos.

Ejemplo:

```bash
touch archivo.txt
```

Verificamos su tamaño:

```bash
ls -l archivo.txt
```

Salida:

```text
-rw-r--r-- 1 usuario usuario 0 Jul 23 archivo.txt
```

El archivo existe, pero tiene **0 bytes**, es decir, está vacío.

Ahora aprenderemos cómo agregar contenido a esos archivos.

---

# Formas de agregar texto a un archivo

Existen tres formas principales:

1. Utilizando el editor **vi** (o vim).
2. Utilizando el comando **echo** con redirecciones.
3. Redireccionando la salida de cualquier comando hacia un archivo.

En esta lección nos enfocaremos en las dos últimas.

---

# El comando `echo`

El comando `echo` simplemente imprime en pantalla el texto que escribimos.

## Sintaxis

```bash
echo "texto"
```

Ejemplo:

```bash
echo "Hola Mundo"
```

Salida:

```text
Hola Mundo
```

---

# Redirección con `>`

El símbolo:

```text
>
```

permite enviar la salida de un comando hacia un archivo.

## Sintaxis

```bash
echo "Texto" > archivo.txt
```

Ejemplo

```bash
echo "Hola Mundo" > notas.txt
```

No veremos nada en pantalla porque la salida fue enviada al archivo.

---

## Verificar el contenido

```bash
cat notas.txt
```

Salida

```text
Hola Mundo
```

---

## Verificar el tamaño

```bash
ls -l notas.txt
```

Ahora el archivo ya no tendrá cero bytes.

---

# Comando `cat`

El comando `cat` permite visualizar el contenido de un archivo.

## Sintaxis

```bash
cat archivo
```

Ejemplo

```bash
cat notas.txt
```

---

# Redirección con `>>`

Los símbolos

```text
>>
```

permiten **agregar** contenido al final del archivo sin borrar lo que ya existe.

## Sintaxis

```bash
echo "Nueva línea" >> archivo.txt
```

Ejemplo

```bash
echo "Esta es la segunda línea." >> notas.txt
```

Ahora verificamos:

```bash
cat notas.txt
```

Salida

```text
Hola Mundo
Esta es la segunda línea.
```

---

# Diferencia entre `>` y `>>`

## `>` (Sobrescribe)

```bash
echo "Nuevo contenido" > notas.txt
```

Resultado

```text
Nuevo contenido
```

Todo el contenido anterior desaparece.

---

## `>>` (Agrega)

```bash
echo "Otra línea" >> notas.txt
```

Resultado

```text
Nuevo contenido
Otra línea
```

Conserva el contenido existente y agrega una nueva línea al final.

---

# Ejemplo completo

Crear archivo

```bash
touch jerry
```

Agregar primera línea

```bash
echo "Jerry Seinfeld es el personaje principal." > jerry
```

Ver contenido

```bash
cat jerry
```

Salida

```text
Jerry Seinfeld es el personaje principal.
```

Agregar otra línea

```bash
echo "La serie comenzó en 1989." >> jerry
```

Ver nuevamente

```bash
cat jerry
```

Resultado

```text
Jerry Seinfeld es el personaje principal.
La serie comenzó en 1989.
```

---

# Sobrescribir un archivo

Supongamos que ejecutamos

```bash
echo "Nuevo contenido." > jerry
```

Ahora

```bash
cat jerry
```

Resultado

```text
Nuevo contenido.
```

Todo el contenido anterior fue eliminado.

---

# Redireccionar la salida de comandos

No solamente podemos guardar texto.

También podemos guardar el resultado de cualquier comando.

---

## Ejemplo con `ls`

```bash
ls -l > listado.txt
```

El archivo contendrá exactamente la salida de `ls -l`.

Verificar:

```bash
cat listado.txt
```

---

## Ejemplo con `ls -ltr`

```bash
ls -ltr > listado_directorio.txt
```

Luego:

```bash
cat listado_directorio.txt
```

Obtendremos el listado completo del directorio.

---

## Ejemplo con `date`

Mostrar la fecha

```bash
date
```

Guardar la fecha

```bash
date > fecha.txt
```

Agregar una nueva fecha

```bash
date >> fecha.txt
```

Verificar

```bash
cat fecha.txt
```

Salida

```text
Thu Jul 23 20:15:18 EDT 2026
Thu Jul 23 20:16:10 EDT 2026
```

---

# Autocompletar nombres de archivos

Linux permite completar automáticamente el nombre de un archivo utilizando la tecla **TAB**.

Ejemplo

```bash
cat list
```

Presionar:

```
TAB
```

Resultado

```bash
cat listado_directorio.txt
```

Esto facilita el trabajo y evita errores de escritura.

---

# Ejemplos prácticos

## Crear archivo

```bash
touch prueba.txt
```

---

## Escribir contenido

```bash
echo "Linux es un excelente sistema operativo." > prueba.txt
```

---

## Agregar otra línea

```bash
echo "Es ampliamente utilizado en servidores." >> prueba.txt
```

---

## Ver contenido

```bash
cat prueba.txt
```

---

## Guardar usuarios conectados

```bash
who > usuarios.txt
```

---

## Guardar directorio actual

```bash
pwd > ubicacion.txt
```

---

## Guardar procesos activos

```bash
ps -ef > procesos.txt
```

---

## Guardar uso del disco

```bash
df -h > discos.txt
```

---

## Guardar memoria disponible

```bash
free -h > memoria.txt
```

---

## Guardar información del sistema

```bash
uname -a > sistema.txt
```

---

# Resumen de comandos

| Comando | Descripción |
|----------|-------------|
| `touch archivo` | Crear un archivo vacío |
| `echo "texto"` | Mostrar texto en pantalla |
| `echo "texto" > archivo` | Escribir texto reemplazando el contenido |
| `echo "texto" >> archivo` | Agregar texto al final del archivo |
| `cat archivo` | Mostrar el contenido del archivo |
| `ls -l > archivo` | Guardar la salida del comando en un archivo |
| `date >> archivo` | Agregar la fecha al archivo |

---

# Diferencias entre `>` y `>>`

| Operador | Acción |
|-----------|--------|
| `>` | Sobrescribe el archivo existente |
| `>>` | Agrega contenido al final del archivo |

---

# Buenas prácticas

- Utiliza `>` únicamente cuando desees reemplazar completamente el contenido de un archivo.
- Utiliza `>>` cuando quieras conservar la información existente y agregar nuevos datos.
- Usa `cat` para verificar siempre el contenido después de escribir.
- Aprovecha la redirección para guardar resultados de comandos y generar reportes automáticamente.

---

# Conclusión

La redirección de salida es una de las características más poderosas del shell de Linux. Gracias a los operadores `>` y `>>`, es posible almacenar texto o la salida de cualquier comando en archivos, facilitando la creación de registros (logs), reportes, scripts y documentación. Comprender la diferencia entre sobrescribir (`>`) y agregar (`>>`) es fundamental para trabajar de forma segura y eficiente en la línea de comandos.