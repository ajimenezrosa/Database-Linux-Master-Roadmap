# 8. Creación, Copia y Búsqueda de Archivos y Directorios

> **Objetivo del capítulo**
>
> Aprender a crear archivos y directorios, copiar archivos y estructuras completas, buscar elementos dentro del sistema de archivos y utilizar comodines para trabajar eficientemente con varios nombres desde la terminal.

---

# Introducción

La administración de Linux requiere trabajar constantemente con archivos y directorios.

Durante las operaciones diarias será necesario:

* Crear archivos vacíos.
* Crear estructuras de directorios.
* Copiar archivos.
* Respaldar directorios completos.
* Localizar archivos olvidados.
* Trabajar con grupos de archivos.
* Filtrar nombres mediante patrones.
* Verificar cada operación antes de continuar.

Las herramientas principales que estudiaremos son:

```bash
touch
mkdir
cp
find
locate
```

También aprenderemos a utilizar los siguientes patrones:

```text
*
?
[]
{}
```

Estos últimos suelen denominarse **comodines**, patrones de expansión o expresiones de globbing del shell.

---

# 1. Verificar el entorno antes de trabajar

Antes de crear o modificar archivos, es recomendable verificar dos cosas:

1. El usuario conectado.
2. El directorio actual.

Para conocer el usuario:

```bash
whoami
```

Para conocer la ubicación actual:

```bash
pwd
```

Ejemplo:

```text
ajimenez
/home/ajimenez
```

Esto evita crear archivos accidentalmente en una ubicación equivocada.

---

# 2. Crear archivos vacíos con `touch`

El comando `touch` permite crear un archivo vacío.

Sintaxis:

```bash
touch nombre_archivo
```

Ejemplo:

```bash
touch reporte.txt
```

Para comprobar que fue creado:

```bash
ls -l reporte.txt
```

Salida aproximada:

```text
-rw-r--r--. 1 ajimenez ajimenez 0 Jul 24 16:30 reporte.txt
```

El tamaño `0` indica que el archivo está vacío.

---

# ¿Qué hace realmente `touch`?

El comando `touch` tiene dos comportamientos principales:

* Si el archivo no existe, lo crea vacío.
* Si ya existe, actualiza sus marcas de tiempo.

Ejemplo:

```bash
touch archivo.txt
```

Podemos revisar sus tiempos con:

```bash
stat archivo.txt
```

---

# Crear varios archivos con un solo comando

No es necesario ejecutar `touch` repetidamente.

Podemos crear varios archivos indicando sus nombres separados por espacios:

```bash
touch archivo1 archivo2 archivo3
```

Ejemplo:

```bash
touch jerry kramer george
```

Comprobamos:

```bash
ls -ltr
```

---

# Crear archivos con extensiones

```bash
touch notas.txt
touch aplicacion.py
touch configuracion.conf
touch mantenimiento.sh
```

Linux no depende obligatoriamente de la extensión para determinar el tipo de archivo, pero las extensiones ayudan a identificar su propósito.

---

# Nombres con espacios

Aunque se recomienda evitar espacios en nombres utilizados frecuentemente desde la terminal, pueden emplearse usando comillas:

```bash
touch "reporte mensual.txt"
```

También puede escaparse el espacio:

```bash
touch reporte\ mensual.txt
```

---

# 3. Crear un archivo mediante redirección

Otra forma rápida de crear un archivo vacío es:

```bash
> archivo.txt
```

Ejemplo:

```bash
> registro.log
```

> **Advertencia:** Si el archivo ya existe, su contenido será eliminado y quedará vacío.

Para evitar sobrescribir accidentalmente un archivo, conviene verificarlo primero:

```bash
ls -l registro.log
```

---

# 4. Crear archivos con un editor

También es posible crear un archivo abriéndolo con un editor.

Ejemplo con Vim:

```bash
vim documento.txt
```

Si el archivo no existe, Vim permitirá crearlo.

Para insertar texto:

```text
i
```

Después de escribir, presiona:

```text
Esc
```

Para guardar y salir:

```text
:wq
```

Para salir sin guardar:

```text
:q!
```

En el editor `vi` o `vim`, los dos puntos forman parte del comando.

---

# Guardar y salir correctamente de Vim

| Acción                      | Comando |
| --------------------------- | ------- |
| Entrar en modo de inserción | `i`     |
| Salir del modo de inserción | `Esc`   |
| Guardar                     | `:w`    |
| Guardar y salir             | `:wq`   |
| Salir sin guardar           | `:q!`   |

El editor Vim se estudiará con mayor profundidad en un capítulo posterior.

---

# 5. Crear directorios con `mkdir`

El comando `mkdir` significa:

```text
Make Directory
```

Su función es crear directorios.

Sintaxis:

```bash
mkdir nombre_directorio
```

Ejemplo:

```bash
mkdir proyectos
```

Comprobamos:

```bash
ls -ld proyectos
```

Salida:

```text
drwxr-xr-x. 2 ajimenez ajimenez 4096 Jul 24 16:40 proyectos
```

La letra inicial `d` confirma que se trata de un directorio.

---

# Crear varios directorios

```bash
mkdir documentos respaldos scripts
```

Esto crea los tres directorios en una sola operación.

---

# Crear directorios anidados

Supongamos que queremos crear:

```text
proyecto/app/config
```

El siguiente comando puede fallar si los directorios superiores todavía no existen:

```bash
mkdir proyecto/app/config
```

Para crear toda la estructura utilizamos `-p`:

```bash
mkdir -p proyecto/app/config
```

Comprobamos con:

```bash
find proyecto -type d
```

Salida:

```text
proyecto
proyecto/app
proyecto/app/config
```

---

# Mensajes detallados con `mkdir -v`

La opción `-v` muestra lo que se está creando:

```bash
mkdir -pv laboratorio/{documentos,respaldos,scripts}
```

Salida aproximada:

```text
mkdir: created directory 'laboratorio'
mkdir: created directory 'laboratorio/documentos'
mkdir: created directory 'laboratorio/respaldos'
mkdir: created directory 'laboratorio/scripts'
```

---

# 6. Crear archivos y directorios en rutas específicas

Podemos crear un archivo sin entrar primero en el directorio de destino.

```bash
touch ~/Documents/informe.txt
```

También podemos crear un directorio:

```bash
mkdir ~/Documents/proyecto
```

Para rutas absolutas:

```bash
mkdir /tmp/laboratorio
```

Esto solo funcionará si el usuario tiene permisos sobre la ubicación.

---

# 7. Permisos al crear archivos

Un usuario normal puede crear archivos dentro de ubicaciones donde tenga permiso de escritura.

Por ejemplo:

```bash
touch ~/prueba.txt
```

Normalmente funcionará porque el usuario es propietario de su directorio personal.

Sin embargo:

```bash
touch /etc/prueba.txt
```

producirá un error similar a:

```text
touch: cannot touch '/etc/prueba.txt': Permission denied
```

Esto ocurre porque `/etc` contiene archivos de configuración del sistema y normalmente pertenece a `root`.

Para tareas administrativas autorizadas:

```bash
sudo touch /etc/prueba.txt
```

No debe utilizarse `sudo` únicamente para evitar comprender un error de permisos. Primero debe verificarse si la operación es realmente necesaria.

---

# 8. Copiar archivos con `cp`

El comando `cp` permite copiar archivos.

Sintaxis:

```bash
cp origen destino
```

Ejemplo:

```bash
cp jerry lex
```

Esto crea una copia de `jerry` llamada `lex`.

Comprobamos:

```bash
ls -l jerry lex
```

---

# Copiar un archivo dentro de un directorio

```bash
cp reporte.txt respaldos/
```

El archivo conservará su nombre:

```text
respaldos/reporte.txt
```

También podemos cambiarle el nombre durante la copia:

```bash
cp reporte.txt respaldos/reporte_2026.txt
```

---

# Copiar varios archivos

```bash
cp archivo1 archivo2 archivo3 destino/
```

El último argumento debe ser un directorio existente.

Ejemplo:

```bash
cp jerry kramer george personajes/
```

---

# Confirmar antes de sobrescribir

La opción `-i` solicita confirmación:

```bash
cp -i reporte.txt respaldos/reporte.txt
```

Salida posible:

```text
cp: overwrite 'respaldos/reporte.txt'?
```

Esto reduce el riesgo de reemplazar un archivo accidentalmente.

---

# Mostrar cada copia

La opción `-v` activa el modo detallado:

```bash
cp -v reporte.txt respaldos/
```

Salida:

```text
'reporte.txt' -> 'respaldos/reporte.txt'
```

Podemos combinar opciones:

```bash
cp -iv reporte.txt respaldos/
```

---

# Preservar atributos

La opción `-p` intenta preservar:

* Permisos.
* Propietario, cuando sea posible.
* Grupo.
* Fechas.

```bash
cp -p configuracion.conf configuracion.conf.bak
```

Para respaldos administrativos suele utilizarse:

```bash
cp -a origen destino
```

La opción `-a` activa el modo archivo y preserva gran parte de la estructura y metadatos.

---

# 9. Copiar directorios

Intentar copiar un directorio sin una opción especial produce un error:

```bash
cp config /tmp/config-backup
```

Salida:

```text
cp: -r not specified; omitting directory 'config'
```

Para copiar directorios se utiliza la opción recursiva:

```bash
cp -r config /tmp/config-backup
```

La letra `r` significa:

```text
Recursive
```

Esto copia:

* El directorio principal.
* Sus archivos.
* Sus subdirectorios.
* Los elementos contenidos en ellos.

---

# Ejemplo de copia recursiva

Creamos una estructura:

```bash
mkdir -p ~/config/aplicacion
touch ~/config/a.conf
touch ~/config/b.conf
touch ~/config/aplicacion/app.conf
```

La copiamos:

```bash
cp -r ~/config /tmp/config-backup
```

Comprobamos:

```bash
find /tmp/config-backup
```

Salida:

```text
/tmp/config-backup
/tmp/config-backup/a.conf
/tmp/config-backup/b.conf
/tmp/config-backup/aplicacion
/tmp/config-backup/aplicacion/app.conf
```

---

# `cp -r` frente a `cp -a`

Para copias sencillas:

```bash
cp -r origen destino
```

Para preservar mejor atributos y enlaces:

```bash
cp -a origen destino
```

En respaldos de configuraciones suele ser preferible:

```bash
sudo cp -a /etc/ssh /etc/ssh.backup
```

Antes de modificar un archivo específico:

```bash
sudo cp -a /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
```

---

# 10. Verificar las copias

Nunca debe asumirse que una copia es correcta únicamente porque no apareció un error.

Podemos verificarla con:

```bash
ls -l destino
```

Para mostrar toda la estructura:

```bash
find destino
```

Para comparar archivos:

```bash
cmp archivo_original archivo_copiado
```

Si son archivos de texto:

```bash
diff archivo_original archivo_copiado
```

---

# 11. Buscar archivos y directorios

Con el tiempo, es normal olvidar dónde fue guardado un archivo.

Linux proporciona principalmente dos herramientas:

```bash
find
locate
```

Aunque ambas buscan archivos, funcionan de manera diferente.

---

# 12. Buscar con `find`

El comando `find` recorre directamente el sistema de archivos.

Sintaxis básica:

```bash
find ruta condiciones
```

Ejemplo:

```bash
find . -name "kramer"
```

El punto indica que la búsqueda debe comenzar en el directorio actual.

Salida posible:

```text
./seinfeld/kramer
```

---

# Buscar desde el directorio personal

```bash
find ~ -name "reporte.txt"
```

---

# Buscar desde la raíz

```bash
sudo find / -name "sshd_config"
```

Buscar desde `/` puede tardar más porque debe recorrer gran parte del sistema de archivos.

---

# Buscar ignorando mayúsculas y minúsculas

`-name` distingue entre mayúsculas y minúsculas.

```bash
find . -name "Reporte.txt"
```

Para ignorar esa diferencia:

```bash
find . -iname "reporte.txt"
```

Esto podría encontrar:

```text
reporte.txt
Reporte.txt
REPORTE.TXT
```

---

# Buscar únicamente archivos

```bash
find . -type f
```

Buscar un archivo específico:

```bash
find . -type f -name "*.conf"
```

---

# Buscar únicamente directorios

```bash
find . -type d
```

Ejemplo:

```bash
find /etc -type d -name "ssh"
```

---

# Tipos comunes en `find`

| Condición | Elemento                  |
| --------- | ------------------------- |
| `-type f` | Archivo regular           |
| `-type d` | Directorio                |
| `-type l` | Enlace simbólico          |
| `-type b` | Dispositivo de bloques    |
| `-type c` | Dispositivo de caracteres |

---

# Buscar por tamaño

Archivos mayores de 100 MB:

```bash
find /var -type f -size +100M
```

Archivos menores de 10 KB:

```bash
find . -type f -size -10k
```

Archivos de aproximadamente 1 GB:

```bash
find . -type f -size 1G
```

---

# Buscar por fecha de modificación

Modificados durante las últimas 24 horas:

```bash
find . -type f -mtime -1
```

Modificados hace más de 30 días:

```bash
find . -type f -mtime +30
```

Modificados exactamente según el intervalo correspondiente a 7 días:

```bash
find . -type f -mtime 7
```

Para minutos:

```bash
find . -type f -mmin -60
```

Esto busca archivos modificados durante los últimos 60 minutos.

---

# Buscar por propietario

```bash
find /home -user ajimenez
```

Buscar por grupo:

```bash
find /srv -group desarrolladores
```

---

# Buscar por permisos

Archivos con permisos exactos `644`:

```bash
find . -type f -perm 0644
```

Archivos con el bit SUID:

```bash
find / -type f -perm /4000 2>/dev/null
```

---

# Ocultar mensajes de permiso denegado

Al buscar desde `/` como usuario normal pueden aparecer numerosos mensajes:

```text
Permission denied
```

Podemos redirigir los errores:

```bash
find / -name "archivo.conf" 2>/dev/null
```

Aquí:

```text
2
```

representa la salida de error estándar y:

```text
/dev/null
```

descarta esos mensajes.

Otra posibilidad es utilizar `sudo` cuando la búsqueda administrativa esté justificada:

```bash
sudo find / -name "archivo.conf"
```

---

# 13. Ejecutar acciones con `find`

`find` también puede ejecutar comandos sobre los resultados.

Ejemplo para mostrar detalles:

```bash
find . -type f -name "*.log" -exec ls -lh {} \;
```

En esta expresión:

| Elemento | Función                         |
| -------- | ------------------------------- |
| `{}`     | Se reemplaza por cada resultado |
| `\;`     | Finaliza la acción `-exec`      |

---

# Eliminar resultados con `find`

Ejemplo peligroso:

```bash
find . -type f -name "*.tmp" -delete
```

Antes de eliminar, debe ejecutarse primero la búsqueda sin `-delete`:

```bash
find . -type f -name "*.tmp"
```

Solo después de revisar cuidadosamente los resultados debería considerarse la eliminación.

Una alternativa con confirmación:

```bash
find . -type f -name "*.tmp" -ok rm {} \;
```

---

# 14. Buscar con `locate`

El comando `locate` busca nombres dentro de una base de datos previamente creada.

Ejemplo:

```bash
locate sshd_config
```

Salida posible:

```text
/etc/ssh/sshd_config
/usr/share/man/man5/sshd_config.5.gz
```

A diferencia de `find`, `locate` no recorre directamente todos los directorios durante cada consulta.

Por eso suele ser mucho más rápido.

---

# Actualizar la base de datos de `locate`

Si un archivo fue creado recientemente, es posible que `locate` todavía no lo encuentre.

La base de datos puede actualizarse mediante:

```bash
sudo updatedb
```

Después:

```bash
locate nombre_archivo
```

El paquete y la implementación concreta pueden variar según la distribución Linux utilizada.

---

# Limitar resultados de `locate`

```bash
locate -n 10 passwd
```

Esto muestra como máximo diez coincidencias.

---

# Buscar nombres base exactos

```bash
locate -b '\sshd_config'
```

La opción `-b` limita la coincidencia al nombre base del archivo, en lugar de buscar el texto en toda la ruta.

---

# 15. Diferencias entre `find` y `locate`

| Característica                  | `find`                   | `locate`                |
| ------------------------------- | ------------------------ | ----------------------- |
| Fuente de información           | Sistema de archivos real | Base de datos indexada  |
| Velocidad                       | Puede ser más lento      | Generalmente muy rápido |
| Resultados recientes            | Sí                       | Dependen de `updatedb`  |
| Búsqueda por tamaño             | Sí                       | No de forma equivalente |
| Búsqueda por permisos           | Sí                       | No                      |
| Búsqueda por propietario        | Sí                       | No                      |
| Ejecutar acciones               | Sí                       | No                      |
| Búsqueda por patrones de nombre | Sí                       | Sí                      |
| Base de datos previa            | No                       | Sí                      |

---

# ¿Cuándo utilizar cada uno?

Utiliza `find` cuando:

* Necesites resultados actuales.
* Quieras buscar por tipo.
* Necesites filtrar por tamaño.
* Quieras buscar por fecha.
* Necesites filtrar por usuario o grupo.
* Desees ejecutar una acción sobre los resultados.

Utiliza `locate` cuando:

* Solo necesitas encontrar rápidamente una ruta por su nombre.
* No necesitas condiciones complejas.
* La base de datos se encuentra actualizada.

---

# Ejemplo comparativo

Creamos un archivo:

```bash
touch ~/archivo_reciente.txt
```

Búsqueda con `find`:

```bash
find ~ -name "archivo_reciente.txt"
```

Debería encontrarlo inmediatamente.

Búsqueda con `locate`:

```bash
locate archivo_reciente.txt
```

Puede no mostrarlo hasta ejecutar:

```bash
sudo updatedb
```

---

# 16. Comodines en Linux

Los comodines permiten trabajar con nombres que siguen un patrón.

Los principales son:

| Patrón   | Significado                         |
| -------- | ----------------------------------- |
| `*`      | Cero o más caracteres               |
| `?`      | Exactamente un carácter             |
| `[abc]`  | Un carácter de la lista             |
| `[a-z]`  | Un carácter dentro del rango        |
| `[!abc]` | Un carácter que no esté en la lista |

Estos patrones son interpretados normalmente por el shell antes de ejecutar el comando.

---

# 17. El comodín `*`

El asterisco representa cero o más caracteres.

Mostrar todos los archivos que comienzan con `reporte`:

```bash
ls reporte*
```

Podría coincidir con:

```text
reporte
reporte.txt
reporte_julio.txt
reportes
```

---

# Archivos que terminan en una extensión

```bash
ls *.conf
```

Coincide con todos los nombres que terminan en `.conf`.

Ejemplos:

```text
httpd.conf
app.conf
database.conf
```

---

# Archivos que contienen una palabra

```bash
ls *backup*
```

Podría encontrar:

```text
backup.sql
config-backup
ultimo_backup.tar
```

---

# 18. El comodín `?`

El signo de interrogación representa exactamente un carácter.

```bash
ls archivo?.txt
```

Coincidiría con:

```text
archivo1.txt
archivoA.txt
archivo9.txt
```

No coincidiría con:

```text
archivo10.txt
archivo.txt
```

porque el patrón exige exactamente un carácter entre `archivo` y `.txt`.

---

# Ejemplo adicional

```bash
ls ?BCD*
```

Coincide con nombres que:

* Tienen cualquier carácter en la primera posición.
* Continúan con `BCD`.
* Pueden contener cualquier cantidad de caracteres después.

Ejemplos:

```text
ABCD1
XBCD-prueba
9BCD
```

---

# 19. Corchetes `[]`

Los corchetes representan un carácter perteneciente a una lista o rango.

```bash
ls archivo[123].txt
```

Coincide con:

```text
archivo1.txt
archivo2.txt
archivo3.txt
```

No coincide con:

```text
archivo4.txt
archivo10.txt
```

---

# Rangos

```bash
ls archivo[1-5].txt
```

Coincide con archivos numerados del 1 al 5.

Para letras:

```bash
ls reporte[a-d].txt
```

---

# Negación

```bash
ls archivo[!1-3].txt
```

Coincide con nombres donde esa posición no contenga los caracteres del 1 al 3.

En algunos shells también puede utilizarse:

```bash
ls archivo[^1-3].txt
```

---

# 20. Expansión con llaves `{}`

Las llaves permiten generar varias palabras o secuencias.

Aunque suelen estudiarse junto con los comodines, técnicamente constituyen una **expansión de llaves de Bash**, no una coincidencia contra nombres existentes.

Crear tres archivos:

```bash
touch archivo{1,2,3}.txt
```

Bash expande el comando como:

```bash
touch archivo1.txt archivo2.txt archivo3.txt
```

---

# Crear una secuencia

```bash
touch archivo{1..9}.txt
```

Esto genera:

```text
archivo1.txt
archivo2.txt
archivo3.txt
...
archivo9.txt
```

---

# Secuencia con ceros iniciales

```bash
touch reporte_{01..12}.txt
```

Genera:

```text
reporte_01.txt
reporte_02.txt
...
reporte_12.txt
```

---

# Secuencias de letras

```bash
mkdir servidor_{a..d}
```

Genera:

```text
servidor_a
servidor_b
servidor_c
servidor_d
```

---

# Crear estructuras complejas

```bash
mkdir -p proyecto/{config,logs,scripts,backups}
```

Resultado:

```text
proyecto/
├── backups
├── config
├── logs
└── scripts
```

También podemos combinar expansiones:

```bash
touch proyecto/logs/app_{01..05}.log
```

---

# 21. Diferencia entre comodines y llaves

Supongamos que no existe ningún archivo.

Este comando:

```bash
echo archivo*.txt
```

intenta coincidir con nombres existentes.

En cambio:

```bash
echo archivo{1..3}.txt
```

genera directamente:

```text
archivo1.txt archivo2.txt archivo3.txt
```

Resumen:

| Característica                    | Comodines              | Llaves             |
| --------------------------------- | ---------------------- | ------------------ |
| Coinciden con archivos existentes | Sí                     | No necesariamente  |
| Generan palabras nuevas           | Mediante coincidencias | Sí                 |
| Ejemplos                          | `*`, `?`, `[]`         | `{a,b}`, `{1..10}` |

---

# 22. Utilizar comodines con diferentes comandos

Los patrones pueden combinarse con numerosos comandos.

Listar archivos:

```bash
ls *.txt
```

Copiar archivos:

```bash
cp *.conf respaldos/
```

Mover archivos:

```bash
mv reporte_*.pdf informes/
```

Mostrar contenido:

```bash
cat notas*.txt
```

Buscar texto:

```bash
grep "ERROR" *.log
```

Obtener propiedades:

```bash
stat *.conf
```

---

# 23. Precaución con `rm` y los comodines

Los comodines pueden afectar muchos archivos en una sola operación.

Este comando:

```bash
rm *.log
```

elimina todos los archivos terminados en `.log` dentro del directorio actual.

Antes de ejecutarlo, debe comprobarse el patrón:

```bash
ls *.log
```

Después, si los resultados son correctos:

```bash
rm -i *.log
```

La opción `-i` solicita confirmación.

---

# Regla de seguridad

Antes de ejecutar:

```bash
rm patron
```

ejecuta primero:

```bash
ls patron
```

Por ejemplo:

```bash
ls app_*.log
```

Solo cuando confirmes que el resultado es correcto:

```bash
rm -i app_*.log
```

---

# Nunca utilizar patrones demasiado generales sin revisar

El siguiente comando puede ser extremadamente peligroso:

```bash
rm *
```

Puede eliminar todos los archivos no ocultos del directorio actual.

Mucho más peligroso sería combinarlo con:

* Privilegios de root.
* Rutas equivocadas.
* Operaciones recursivas.
* Ausencia de confirmación.

Por eso siempre debe verificarse:

```bash
pwd
ls
```

antes de eliminar grupos de archivos.

---

# 24. Comillas y expansión del shell

Existe una diferencia importante entre:

```bash
find . -name "*.log"
```

y:

```bash
find . -name *.log
```

En la primera forma, las comillas impiden que Bash expanda el patrón antes de entregarlo a `find`.

Esta es la forma recomendada:

```bash
find . -name "*.log"
```

Sin comillas, el shell puede expandir `*.log` utilizando los archivos del directorio actual y alterar el comportamiento del comando.

---

# 25. Ejemplo completo de trabajo

Creamos un laboratorio:

```bash
mkdir -p ~/laboratorio-archivos/{entrada,salida,respaldos}
```

Creamos archivos:

```bash
touch ~/laboratorio-archivos/entrada/reporte_{01..05}.txt
touch ~/laboratorio-archivos/entrada/app_{01..03}.log
```

Listamos:

```bash
ls -l ~/laboratorio-archivos/entrada
```

Copiamos los archivos de texto:

```bash
cp ~/laboratorio-archivos/entrada/*.txt \
   ~/laboratorio-archivos/salida/
```

Respaldamos toda la estructura:

```bash
cp -a ~/laboratorio-archivos \
      ~/laboratorio-archivos-backup
```

Buscamos archivos `.log`:

```bash
find ~/laboratorio-archivos -type f -name "*.log"
```

Verificamos el respaldo:

```bash
find ~/laboratorio-archivos-backup
```

---

# 26. Laboratorio práctico

## Ejercicio 1: crear archivos

```bash
mkdir -p ~/practica-rhcsa/archivos
cd ~/practica-rhcsa/archivos
touch jerry kramer george
touch clark lois lex
```

Comprueba:

```bash
ls -ltr
```

---

## Ejercicio 2: expansión con llaves

```bash
touch personaje_{01..09}.txt
```

Muestra únicamente estos archivos:

```bash
ls personaje_*.txt
```

---

## Ejercicio 3: crear directorios

```bash
mkdir seinfeld superman simpson
```

Crea una estructura anidada:

```bash
mkdir -p proyectos/app/{config,logs,backups}
```

Comprueba:

```bash
find proyectos -type d
```

---

## Ejercicio 4: copiar archivos

```bash
cp jerry seinfeld/
cp kramer george seinfeld/
```

Verifica:

```bash
ls -l seinfeld
```

---

## Ejercicio 5: copiar directorios

```bash
cp -a proyectos proyectos-backup
```

Compara ambas estructuras:

```bash
find proyectos
find proyectos-backup
```

---

## Ejercicio 6: buscar archivos

```bash
find ~/practica-rhcsa -type f -name "kramer"
```

Busca todos los archivos `.txt`:

```bash
find ~/practica-rhcsa -type f -name "*.txt"
```

Busca directorios llamados `config`:

```bash
find ~/practica-rhcsa -type d -name "config"
```

---

## Ejercicio 7: utilizar `locate`

Actualiza la base de datos, cuando la herramienta esté instalada:

```bash
sudo updatedb
```

Busca:

```bash
locate personaje_01.txt
```

Compara el resultado con:

```bash
find ~/practica-rhcsa -name "personaje_01.txt"
```

---

## Ejercicio 8: comodines

Lista archivos numerados del 1 al 5:

```bash
ls personaje_0[1-5].txt
```

Lista archivos cuyo nombre contenga `son`:

```bash
find ~/practica-rhcsa -name "*son*"
```

---

## Ejercicio 9: eliminación segura

Comprueba primero:

```bash
ls personaje_0[6-9].txt
```

Después elimina con confirmación:

```bash
rm -i personaje_0[6-9].txt
```

---

# 27. Desafío RHCSA

Realiza las siguientes tareas sin utilizar una interfaz gráfica:

1. Crea el directorio `~/rhcsa-lab`.
2. Dentro, crea `config`, `logs` y `backup`.
3. Crea diez archivos llamados `app_01.log` hasta `app_10.log`.
4. Crea tres archivos `.conf`.
5. Copia todos los archivos `.conf` a `backup`.
6. Copia recursivamente toda la estructura a `/tmp/rhcsa-lab-copy`.
7. Busca todos los archivos `.log`.
8. Busca los archivos modificados durante la última hora.
9. Muestra únicamente los archivos `app_01.log` a `app_05.log`.
10. Verifica toda la estructura con `find`.

Posible solución:

```bash
mkdir -p ~/rhcsa-lab/{config,logs,backup}

touch ~/rhcsa-lab/logs/app_{01..10}.log
touch ~/rhcsa-lab/config/{app,database,network}.conf

cp ~/rhcsa-lab/config/*.conf ~/rhcsa-lab/backup/

cp -a ~/rhcsa-lab /tmp/rhcsa-lab-copy

find ~/rhcsa-lab -type f -name "*.log"

find ~/rhcsa-lab -type f -mmin -60

ls ~/rhcsa-lab/logs/app_0[1-5].log

find ~/rhcsa-lab
```

---

# 28. Errores comunes

## Usar `cp` sin `-r` para un directorio

```text
cp: omitting directory
```

Solución:

```bash
cp -r origen destino
```

o preferiblemente para preservar atributos:

```bash
cp -a origen destino
```

---

## Crear un archivo en un directorio sin permisos

```text
Permission denied
```

Verifica:

```bash
ls -ld directorio
```

No utilices `sudo` automáticamente sin entender por qué se necesita.

---

## Buscar con un nombre incorrecto

Linux distingue entre mayúsculas y minúsculas:

```text
Archivo.txt
archivo.txt
ARCHIVO.TXT
```

Para ignorar la diferencia:

```bash
find . -iname "archivo.txt"
```

---

## `locate` no encuentra un archivo reciente

Actualiza su base de datos:

```bash
sudo updatedb
```

También verifica que la herramienta correspondiente esté instalada.

---

## Utilizar patrones sin comillas en `find`

Evita:

```bash
find . -name *.log
```

Utiliza:

```bash
find . -name "*.log"
```

---

## Eliminar demasiados archivos

Antes de:

```bash
rm patron
```

ejecuta:

```bash
ls patron
```

---

# 29. Buenas prácticas

* Verificar `whoami` y `pwd` antes de realizar cambios importantes.
* Crear archivos de laboratorio dentro del directorio personal.
* Utilizar `mkdir -p` para estructuras anidadas.
* Usar `cp -i` cuando exista riesgo de sobrescritura.
* Utilizar `cp -a` para respaldar configuraciones.
* Verificar las copias con `find`, `diff` o `cmp`.
* Colocar entre comillas los patrones utilizados con `find`.
* Revisar con `ls` cualquier patrón antes de usarlo con `rm`.
* Evitar trabajar permanentemente como root.
* Consultar `man` antes de ejecutar opciones desconocidas.

---

# 30. Preguntas de repaso

1. ¿Qué función realiza el comando `touch`?
2. ¿Qué ocurre si se ejecuta `touch` sobre un archivo existente?
3. ¿Cómo se crean varios archivos con un solo comando?
4. ¿Para qué sirve `mkdir`?
5. ¿Qué función cumple la opción `mkdir -p`?
6. ¿Cómo se copia un archivo?
7. ¿Por qué se necesita `-r` para copiar un directorio?
8. ¿Qué diferencia existe entre `cp -r` y `cp -a`?
9. ¿Cómo se verifica que una copia fue realizada correctamente?
10. ¿Cómo funciona el comando `find`?
11. ¿Cómo funciona `locate`?
12. ¿Cuál de los dos utiliza una base de datos?
13. ¿Cuál permite buscar por tamaño o permisos?
14. ¿Qué hace el comodín `*`?
15. ¿Qué representa `?`?
16. ¿Para qué se utilizan los corchetes?
17. ¿Cuál es la diferencia entre un comodín y la expansión con llaves?
18. ¿Por qué deben colocarse comillas alrededor de `"*.log"` en `find`?
19. ¿Qué debe hacerse antes de ejecutar `rm` con un comodín?
20. ¿Cómo se buscan únicamente directorios llamados `backup`?

---

# Resumen

Para crear archivos podemos utilizar:

```bash
touch archivo
vim archivo
> archivo
```

Para crear directorios:

```bash
mkdir directorio
mkdir -p ruta/anidada
```

Para copiar archivos:

```bash
cp origen destino
```

Para copiar directorios:

```bash
cp -r origen destino
cp -a origen destino
```

Para buscar archivos directamente:

```bash
find ruta condiciones
```

Para realizar búsquedas rápidas mediante una base de datos:

```bash
locate nombre
```

Los patrones principales son:

```text
*       Cero o más caracteres
?       Exactamente un carácter
[]      Un carácter de una lista o rango
{}      Generación de palabras o secuencias
```

Dominar estas herramientas permite administrar grandes cantidades de archivos de forma rápida, precisa y segura. Son habilidades fundamentales para trabajar profesionalmente con Linux y prepararse para RHCSA.

---

# Comandos esenciales del capítulo

```bash
whoami
pwd

touch archivo
touch archivo1 archivo2 archivo3

mkdir directorio
mkdir -p ruta/directorio
mkdir -pv proyecto/{config,logs,backup}

cp origen destino
cp -i origen destino
cp -v origen destino
cp -p origen destino
cp -r directorio destino
cp -a directorio destino

find . -name "archivo"
find /ruta -type f
find /ruta -type d
find /ruta -type f -name "*.conf"
find /ruta -type f -size +100M
find /ruta -type f -mtime -7
find /ruta -type f -mmin -60
find /ruta -user usuario

locate nombre
sudo updatedb

ls *.txt
ls archivo?.txt
ls archivo[1-5].txt
touch archivo{1..10}.txt
mkdir -p proyecto/{config,logs,backup}
```

---

# Próxima lección

➡ **9. Copiar, mover, renombrar y eliminar archivos de forma segura**
