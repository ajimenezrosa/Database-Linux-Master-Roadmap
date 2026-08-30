# 9. Navegación por el Sistema de Archivos y Propiedades de Archivos

![Administrador navegando por el sistema de archivos Linux](images/07-navegacion-portada.jpg)

> **Objetivo del capítulo**
>
> Aprender a desplazarse por el sistema de archivos Linux mediante los comandos `cd`, `pwd` y `ls`, interpretar rutas absolutas y relativas, regresar a directorios anteriores y reconocer las propiedades de archivos y directorios mostradas por `ls -l`.

---

# Introducción

En un entorno gráfico, navegar por carpetas normalmente consiste en hacer doble clic, utilizar el botón Atrás y observar la ruta en la parte superior del explorador de archivos.

En Linux, especialmente en servidores sin interfaz gráfica, esas mismas operaciones se realizan desde la terminal.

Los tres comandos fundamentales para navegar son:

```bash
cd
pwd
ls
```

| Comando | Significado             | Función                               |
| ------- | ----------------------- | ------------------------------------- |
| `cd`    | Change directory        | Cambiar de directorio                 |
| `pwd`   | Print working directory | Mostrar el directorio actual          |
| `ls`    | List                    | Mostrar el contenido de un directorio |

Estos comandos se utilizan constantemente durante la administración de sistemas Linux y son esenciales para RHCSA.

---

# 1. Comprender la estructura jerárquica de Linux

Linux organiza sus archivos y directorios en una estructura jerárquica que comienza en:

```text
/
```

Este directorio se denomina **directorio raíz**.

Ejemplo simplificado:

```text
/
├── boot
├── dev
├── etc
├── home
│   ├── ana
│   └── juan
├── root
├── tmp
├── usr
└── var
    └── log
```

A diferencia de Windows, Linux no organiza el sistema mediante letras de unidad como:

```text
C:
D:
E:
```

Todos los sistemas de archivos se integran bajo la misma estructura que comienza en `/`.

---

# 2. Mostrar el directorio actual con `pwd`

Cuando trabajamos desde una terminal, debemos saber en qué ubicación nos encontramos.

Para ello se utiliza:

```bash
pwd
```

El comando significa:

```text
Print Working Directory
```

Ejemplo:

```bash
pwd
```

Salida:

```text
/home/ajimenez
```

Esto indica que el usuario se encuentra dentro de su directorio personal.

---

## Ejemplo adicional

```bash
cd /var/log
pwd
```

Salida:

```text
/var/log
```

Siempre que tengas dudas sobre tu ubicación, ejecuta:

```bash
pwd
```

---

# 3. Mostrar el contenido con `ls`

El comando `ls` muestra los archivos y directorios contenidos en una ubicación.

```bash
ls
```

Ejemplo:

```text
Desktop  Documents  Downloads  Pictures
```

También podemos consultar directamente otra ruta sin entrar en ella:

```bash
ls /etc
```

Esto muestra el contenido de `/etc`, aunque el usuario permanezca en su directorio actual.

---

# 4. Cambiar de directorio con `cd`

El comando `cd` permite desplazarse de una ubicación a otra.

Sintaxis:

```bash
cd ruta
```

Ejemplo:

```bash
cd /var
```

Para confirmar el cambio:

```bash
pwd
```

Salida:

```text
/var
```

---

# Acceder a un subdirectorio

Supongamos que estamos en:

```text
/var
```

y queremos entrar en:

```text
/var/log
```

Podemos ejecutar:

```bash
cd log
```

Luego:

```bash
pwd
```

Salida:

```text
/var/log
```

Como `log` se encuentra dentro del directorio actual, no fue necesario escribir la ruta completa.

---

# 5. Rutas absolutas y relativas

Existen dos maneras principales de indicar una ubicación en Linux.

## Ruta absoluta

Una ruta absoluta comienza en `/`.

Ejemplo:

```bash
cd /var/log
```

Esta ruta funcionará independientemente del directorio en el que se encuentre el usuario.

Otros ejemplos:

```text
/etc/ssh/sshd_config
/home/ajimenez/Documents
/var/log/messages
```

---

## Ruta relativa

Una ruta relativa se interpreta a partir del directorio actual.

Si estamos en:

```text
/var
```

podemos entrar en `/var/log` usando:

```bash
cd log
```

No comienza con `/`, porque su punto de partida es el directorio actual.

---

## Comparación

| Ruta        | Tipo     |
| ----------- | -------- |
| `/etc/ssh`  | Absoluta |
| `/var/log`  | Absoluta |
| `Documents` | Relativa |
| `log`       | Relativa |
| `../log`    | Relativa |

---

# 6. Regresar al directorio anterior con `..`

Dos puntos representan el directorio padre:

```text
..
```

Ejemplo:

```bash
cd /var/log
pwd
```

Salida:

```text
/var/log
```

Para regresar a `/var`:

```bash
cd ..
```

Comprobamos:

```bash
pwd
```

Salida:

```text
/var
```

Para regresar nuevamente a `/`:

```bash
cd ..
```

---

# Subir varios niveles

Podemos combinar varios directorios padre:

```bash
cd ../..
```

Por ejemplo, desde:

```text
/etc/sysconfig/network-scripts
```

el comando:

```bash
cd ../..
```

podría llevarnos a:

```text
/etc
```

El resultado depende de la ubicación inicial.

---

# 7. El directorio actual representado por `.`

Un solo punto representa el directorio actual:

```text
.
```

Ejemplo:

```bash
ls .
```

Esto muestra el contenido del directorio en el que estamos.

Otro ejemplo:

```bash
cp archivo.txt .
```

En este caso, `.` significa “copiar al directorio actual”.

---

# 8. Regresar al directorio personal

Ejecutar `cd` sin argumentos lleva al usuario a su directorio personal:

```bash
cd
```

También puede utilizarse:

```bash
cd ~
```

Ejemplo para un usuario normal:

```text
/home/ajimenez
```

Ejemplo para root:

```text
/root
```

Podemos verificarlo con:

```bash
pwd
```

---

# El símbolo `~`

La virgulilla:

```text
~
```

representa el directorio personal del usuario actual.

Ejemplos:

```bash
cd ~
ls ~/Documents
mkdir ~/pruebas
```

Para el usuario `ajimenez`, `~` normalmente representa:

```text
/home/ajimenez
```

Para el usuario root representa:

```text
/root
```

---

# 9. Regresar al directorio visitado anteriormente

El comando:

```bash
cd -
```

permite regresar al directorio anterior.

Ejemplo:

```bash
cd /etc
cd /var/log
cd -
```

Salida:

```text
/etc
```

Si volvemos a ejecutar:

```bash
cd -
```

regresaremos a:

```text
/var/log
```

Esta función es útil para alternar entre dos ubicaciones.

---

# 10. Navegar hasta el directorio raíz

Para ir directamente a la parte superior del sistema de archivos:

```bash
cd /
```

Verificamos:

```bash
pwd
```

Salida:

```text
/
```

Listamos su contenido:

```bash
ls
```

o con información detallada:

```bash
ls -l
```

---

# 11. Opciones importantes de `ls`

El comando `ls` dispone de numerosas opciones.

## Listado largo

```bash
ls -l
```

Muestra propiedades como:

* Tipo de archivo.
* Permisos.
* Número de enlaces.
* Propietario.
* Grupo.
* Tamaño.
* Fecha de modificación.
* Nombre.

---

## Mostrar archivos ocultos

```bash
ls -a
```

Los archivos ocultos comienzan con un punto:

```text
.bashrc
.bash_profile
.config
.ssh
```

---

## Listado largo con archivos ocultos

```bash
ls -la
```

Esta es una de las combinaciones más utilizadas.

También puede escribirse:

```bash
ls -al
```

---

## Tamaños legibles

```bash
ls -lh
```

En lugar de mostrar tamaños únicamente en bytes, puede mostrar:

```text
4.0K
25M
2.1G
```

---

## Mostrar el tipo de cada elemento

```bash
ls -F
```

Puede agregar indicadores al nombre:

| Indicador | Significado        |                    |
| --------- | ------------------ | ------------------ |
| `/`       | Directorio         |                    |
| `*`       | Archivo ejecutable |                    |
| `@`       | Enlace simbólico   |                    |
| `=`       | Socket             |                    |
| `         | `                  | Tubería con nombre |

Ejemplo:

```text
Documents/
script.sh*
ultimo_log@
```

---

## Ordenar por fecha

```bash
ls -lt
```

Muestra primero los archivos modificados más recientemente.

Para invertir el orden:

```bash
ls -ltr
```

---

# 12. Interpretar la salida de `ls -l`

Ejemplo:

```bash
ls -l
```

Salida:

```text
-rw-r--r--. 1 ajimenez usuarios 2450 Jul 24 14:30 informe.txt
drwxr-xr-x. 2 ajimenez usuarios 4096 Jul 24 13:15 proyectos
lrwxrwxrwx. 1 root      root       7 Jul 20 09:00 bin -> usr/bin
```

Cada columna proporciona información importante.

---

# Desglose de las columnas

```text
-rw-r--r--. 1 ajimenez usuarios 2450 Jul 24 14:30 informe.txt
│           │ │        │        │    │            │
│           │ │        │        │    │            └── Nombre
│           │ │        │        │    └── Fecha y hora
│           │ │        │        └── Tamaño
│           │ │        └── Grupo propietario
│           │ └── Usuario propietario
│           └── Número de enlaces
└── Tipo y permisos
```

---

# 13. Tipo de archivo

El primer carácter de la salida indica el tipo de elemento.

| Carácter | Tipo                      |
| -------- | ------------------------- |
| `-`      | Archivo regular           |
| `d`      | Directorio                |
| `l`      | Enlace simbólico          |
| `b`      | Dispositivo de bloques    |
| `c`      | Dispositivo de caracteres |
| `p`      | Tubería con nombre        |
| `s`      | Socket                    |

Ejemplos:

```text
-rw-r--r-- archivo.txt
drwxr-xr-x documentos
lrwxrwxrwx enlace
```

---

## Archivo regular

Cuando comienza con un guion:

```text
-
```

significa que es un archivo regular.

Ejemplo:

```text
-rw-r--r-- informe.txt
```

---

## Directorio

Cuando comienza con:

```text
d
```

significa que es un directorio.

Ejemplo:

```text
drwxr-xr-x proyectos
```

---

## Enlace simbólico

Cuando comienza con:

```text
l
```

significa que se trata de un enlace simbólico.

Ejemplo:

```text
lrwxrwxrwx bin -> usr/bin
```

La flecha indica el destino del enlace.

---

# 14. Permisos

Después del carácter correspondiente al tipo aparecen nueve caracteres de permisos:

```text
rw-r--r--
```

Se dividen en tres grupos:

```text
rw-  r--  r--
│    │    │
│    │    └── Otros usuarios
│    └── Grupo
└── Propietario
```

Los símbolos principales son:

| Símbolo | Significado        |
| ------- | ------------------ |
| `r`     | Lectura            |
| `w`     | Escritura          |
| `x`     | Ejecución o acceso |
| `-`     | Permiso ausente    |

Los permisos se estudiarán detalladamente en un capítulo posterior.

---

# 15. Número de enlaces

La segunda columna muestra el número de enlaces asociados al archivo o directorio.

Ejemplo:

```text
drwxr-xr-x. 4 root root 4096 Jul 24 10:00 proyecto
             ^
```

En los archivos regulares representa el número de enlaces físicos.

En los directorios está relacionado con:

* El propio directorio.
* Su referencia `.`.
* La referencia `..` de sus subdirectorios inmediatos.

Este tema se ampliará al estudiar enlaces físicos y simbólicos.

---

# 16. Propietario y grupo

Ejemplo:

```text
-rw-r--r--. 1 ajimenez desarrolladores 2450 Jul 24 informe.txt
                │         │
                │         └── Grupo propietario
                └── Usuario propietario
```

En este ejemplo:

* Propietario: `ajimenez`
* Grupo: `desarrolladores`

Linux utiliza usuarios y grupos para controlar el acceso a los recursos.

---

# 17. Tamaño

La columna de tamaño normalmente se expresa en bytes:

```text
2450
```

Para mostrar un formato más legible:

```bash
ls -lh
```

Ejemplo:

```text
2.4K
18M
1.5G
```

> **Importante:** En un directorio, el tamaño mostrado por `ls -l` no representa la suma total de todo lo almacenado en su interior.

Para calcular el espacio utilizado por un directorio se emplea normalmente:

```bash
du -sh directorio
```

Ejemplo:

```bash
du -sh /var/log
```

---

# 18. Fecha y hora

Las siguientes columnas muestran la fecha de última modificación.

Ejemplo:

```text
Jul 24 14:30
```

Esto no necesariamente indica cuándo fue creado el archivo.

En Linux, `ls -l` muestra normalmente el tiempo de modificación del contenido.

Para obtener información más detallada se puede utilizar:

```bash
stat archivo.txt
```

---

# 19. Nombre del archivo o directorio

La última columna muestra el nombre:

```text
informe.txt
```

Cuando se trata de un enlace simbólico también se muestra el destino:

```text
bin -> usr/bin
```

---

# 20. Ver propiedades de un archivo específico

En lugar de listar todo el directorio:

```bash
ls -l informe.txt
```

Para un directorio:

```bash
ls -ld proyectos
```

La opción `-d` evita que `ls` muestre el contenido interno y permite observar las propiedades del directorio mismo.

Comparación:

```bash
ls -l proyectos
```

Muestra el contenido de `proyectos`.

```bash
ls -ld proyectos
```

Muestra las propiedades del directorio `proyectos`.

---

# 21. El comando `stat`

Para ver información más detallada:

```bash
stat archivo.txt
```

Salida aproximada:

```text
File: archivo.txt
Size: 2450
Blocks: 8
IO Block: 4096 regular file
Device: 253,0
Inode: 123456
Links: 1
Access: (0644/-rw-r--r--)
Uid: (1000/ajimenez)
Gid: (1000/ajimenez)
Access: 2026-07-24 14:35:00
Modify: 2026-07-24 14:30:00
Change: 2026-07-24 14:31:00
```

`stat` puede mostrar:

* Tamaño.
* Inodo.
* Permisos.
* Usuario.
* Grupo.
* Número de enlaces.
* Último acceso.
* Última modificación.
* Último cambio de metadatos.

---

# 22. Nombres con espacios

Cuando una ruta contiene espacios, debe colocarse entre comillas:

```bash
cd "Mis Documentos"
```

También se puede escapar el espacio:

```bash
cd Mis\ Documentos
```

Ejemplo de ruta completa:

```bash
cd "/home/ajimenez/Mis Documentos"
```

---

# 23. Autocompletado con Tab

No es necesario escribir siempre los nombres completos.

Podemos comenzar a escribir:

```bash
cd Doc
```

y presionar:

```text
Tab
```

Bash intentará completar el nombre:

```bash
cd Documents/
```

Si existen varias coincidencias, presionar `Tab` dos veces puede mostrar las opciones disponibles.

El autocompletado:

* Reduce errores.
* Acelera la navegación.
* Ayuda con nombres largos.
* Facilita el trabajo con rutas complejas.

---

# 24. Historial de comandos

Para reutilizar comandos anteriores podemos utilizar:

```text
Flecha arriba
```

También podemos consultar el historial:

```bash
history
```

Ejemplo:

```text
101  cd /var/log
102  ls -lh
103  pwd
```

Podemos repetir un comando por su número:

```bash
!102
```

---

# 25. Comparación con Windows

| Acción            | Windows gráfico            | Linux desde terminal |
| ----------------- | -------------------------- | -------------------- |
| Entrar en carpeta | Doble clic                 | `cd directorio`      |
| Ver ubicación     | Barra superior             | `pwd`                |
| Ver contenido     | Automático                 | `ls`                 |
| Regresar          | Botón Atrás                | `cd ..` o `cd -`     |
| Ir al inicio      | Unidad o acceso rápido     | `cd /` o `cd ~`      |
| Ver propiedades   | Clic derecho → Propiedades | `ls -l` o `stat`     |

En Windows, al abrir una carpeta el explorador realiza visualmente varias operaciones al mismo tiempo.

En la terminal Linux, el administrador tiene control explícito sobre cada acción.

---

# 26. Ejemplo completo de navegación

Comenzamos en el directorio personal:

```bash
cd
pwd
```

Salida:

```text
/home/ajimenez
```

Vamos al directorio raíz:

```bash
cd /
```

Mostramos su contenido:

```bash
ls -l
```

Entramos en `/var`:

```bash
cd var
```

Entramos en `/var/log`:

```bash
cd log
```

Confirmamos:

```bash
pwd
```

Salida:

```text
/var/log
```

Mostramos archivos con tamaño legible:

```bash
ls -lh
```

Regresamos a `/var`:

```bash
cd ..
```

Regresamos directamente al directorio personal:

```bash
cd
```

---

# 27. No es necesario convertirse en root para navegar

Para aprender navegación no es necesario iniciar una sesión permanente como root.

Un usuario normal puede ejecutar:

```bash
cd /
ls
pwd
```

Algunos directorios restringidos producirán errores de permisos, lo cual es normal.

Ejemplo:

```bash
cd /root
```

Posible resultado:

```text
bash: cd: /root: Permission denied
```

Cuando una tarea administrativa requiera privilegios, se recomienda utilizar:

```bash
sudo comando
```

o, cuando sea necesario abrir una sesión administrativa:

```bash
sudo -i
```

Trabajar permanentemente como root aumenta el riesgo de cometer errores graves.

---

# 28. Errores frecuentes

## No existe el archivo o directorio

```text
bash: cd: proyecto: No such file or directory
```

Posibles causas:

* Nombre incorrecto.
* Diferencia entre mayúsculas y minúsculas.
* Ruta equivocada.
* Directorio inexistente.

Linux distingue entre:

```text
Documents
documents
DOCUMENTS
```

Son nombres diferentes.

---

## No es un directorio

```text
bash: cd: informe.txt: Not a directory
```

Esto ocurre al intentar utilizar `cd` sobre un archivo regular.

Para confirmar el tipo:

```bash
ls -l informe.txt
```

o:

```bash
file informe.txt
```

---

## Permiso denegado

```text
bash: cd: /root: Permission denied
```

El usuario no tiene permisos para acceder a esa ubicación.

---

# 29. Buenas prácticas

* Ejecutar `pwd` antes de realizar operaciones destructivas.
* Utilizar autocompletado con `Tab`.
* Evitar trabajar permanentemente como root.
* Utilizar rutas absolutas en scripts.
* Colocar entre comillas las rutas con espacios.
* Revisar cuidadosamente el contenido antes de ejecutar `rm`, `mv` o `cp`.
* Utilizar `ls -la` para incluir archivos ocultos.
* Utilizar `ls -ld` para revisar las propiedades de un directorio.
* Utilizar `stat` cuando se necesite información detallada.

---

# 30. Laboratorio práctico

## Preparación

Crea una estructura de prueba:

```bash
mkdir -p ~/laboratorio-linux/proyectos/app1
mkdir -p ~/laboratorio-linux/documentos
touch ~/laboratorio-linux/documentos/notas.txt
touch ~/laboratorio-linux/proyectos/app1/config.conf
```

---

## Ejercicio 1: ubicación actual

```bash
cd
pwd
```

Responde:

1. ¿Cuál es tu directorio personal?
2. ¿La ruta mostrada es absoluta o relativa?

---

## Ejercicio 2: navegación

```bash
cd ~/laboratorio-linux
pwd
ls
```

Entra en:

```bash
cd proyectos/app1
```

Comprueba:

```bash
pwd
```

---

## Ejercicio 3: directorio padre

Regresa un nivel:

```bash
cd ..
pwd
```

Regresa dos niveles:

```bash
cd ../..
pwd
```

---

## Ejercicio 4: directorio anterior

```bash
cd /etc
cd /var/log
cd -
pwd
```

Comprueba a qué ubicación regresaste.

---

## Ejercicio 5: propiedades

```bash
ls -l ~/laboratorio-linux
ls -la ~/laboratorio-linux
ls -lh ~/laboratorio-linux/documentos
```

Identifica:

* Directorios.
* Archivos regulares.
* Propietarios.
* Grupos.
* Tamaños.
* Fechas.

---

## Ejercicio 6: `ls -ld`

Compara:

```bash
ls -l ~/laboratorio-linux/documentos
```

con:

```bash
ls -ld ~/laboratorio-linux/documentos
```

Explica la diferencia.

---

## Ejercicio 7: información detallada

```bash
stat ~/laboratorio-linux/documentos/notas.txt
```

Localiza:

* Tamaño.
* Inodo.
* Permisos.
* Propietario.
* Grupo.
* Fechas.

---

# 31. Desafío RHCSA

Realiza las siguientes tareas únicamente desde la terminal:

1. Ve al directorio raíz.
2. Lista su contenido en formato largo.
3. Entra en `/var/log`.
4. Confirma la ubicación.
5. Muestra los archivos ocultos.
6. Ordena el listado por fecha.
7. Muestra los tamaños en formato legible.
8. Regresa al directorio anterior.
9. Regresa al directorio personal.
10. Muestra las propiedades de tu directorio personal sin listar su contenido.

Posible secuencia:

```bash
cd /
ls -l
cd /var/log
pwd
ls -la
ls -lt
ls -lh
cd -
cd
ls -ld ~
```

---

# 32. Preguntas de repaso

1. ¿Para qué sirve el comando `pwd`?
2. ¿Qué función realiza `cd`?
3. ¿Qué hace el comando `ls`?
4. ¿Cuál es la diferencia entre una ruta absoluta y una relativa?
5. ¿Qué representa `.`?
6. ¿Qué representa `..`?
7. ¿Qué sucede al ejecutar `cd` sin argumentos?
8. ¿Qué representa `~`?
9. ¿Para qué sirve `cd -`?
10. ¿Qué opción de `ls` muestra archivos ocultos?
11. ¿Qué opción muestra un listado detallado?
12. ¿Qué carácter identifica un directorio en `ls -l`?
13. ¿Qué carácter identifica un archivo regular?
14. ¿Qué carácter identifica un enlace simbólico?
15. ¿Qué diferencia existe entre `ls -l directorio` y `ls -ld directorio`?
16. ¿Por qué el tamaño mostrado para un directorio no representa todo su contenido?
17. ¿Qué comando proporciona más detalles que `ls -l`?
18. ¿Es necesario convertirse en root para navegar por el sistema?

---

# Resumen

Los comandos fundamentales para navegar por Linux son:

```bash
cd
pwd
ls
```

Con ellos podemos:

* Cambiar de ubicación.
* Confirmar dónde estamos.
* Mostrar archivos y directorios.
* Navegar mediante rutas absolutas y relativas.
* Regresar al directorio padre.
* Volver al directorio personal.
* Alternar entre dos ubicaciones.

El comando:

```bash
ls -l
```

permite además analizar las propiedades de archivos y directorios, incluyendo:

* Tipo.
* Permisos.
* Enlaces.
* Propietario.
* Grupo.
* Tamaño.
* Fecha de modificación.
* Nombre.

Dominar estos comandos es indispensable para administrar sistemas Linux y constituye una base esencial para la certificación RHCSA.

---

# Comandos esenciales del capítulo

```bash
pwd
ls
ls -l
ls -la
ls -lh
ls -lt
ls -ltr
ls -ld directorio
cd ruta
cd /
cd ..
cd ../..
cd
cd ~
cd -
stat archivo
file archivo
du -sh directorio
```

---

# Próxima lección

