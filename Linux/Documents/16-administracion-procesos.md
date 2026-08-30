# 16. Administración de Procesos en Linux

Todo programa que se ejecuta en Linux se convierte en un **proceso**. La administración de procesos es una de las tareas fundamentales de cualquier administrador de sistemas, ya que permite supervisar el consumo de recursos, identificar problemas de rendimiento, finalizar procesos bloqueados y controlar la prioridad con la que el sistema ejecuta las aplicaciones.

---

# Objetivos de este capítulo

Al finalizar esta unidad serás capaz de:

* Comprender qué es un proceso.
* Diferenciar entre procesos en primer plano y segundo plano.
* Visualizar procesos activos.
* Administrar procesos mediante señales.
* Cambiar la prioridad de ejecución.
* Controlar trabajos (Jobs).
* Monitorear el consumo de CPU y memoria.
* Aplicar buenas prácticas de administración.

---

# ¿Qué es un proceso?

Un proceso es una instancia de un programa que se encuentra en ejecución.

Ejemplos:

* Bash
* Firefox
* SQL Server
* PostgreSQL
* Apache
* Nginx
* Python

Cada proceso posee información como:

* PID (Process ID)
* PPID (Parent Process ID)
* Usuario propietario
* Prioridad
* Estado
* Consumo de CPU
* Consumo de memoria

---

# Ciclo de vida de un proceso

```text id="proc001"
Nuevo

↓

Listo

↓

Ejecutándose

↓

Esperando

↓

Finalizado
```

---

# PID

Cada proceso posee un identificador único.

Visualizar el proceso actual

```bash id="proc002"
echo $$
```

Ejemplo

```text id="proc003"
5421
```

---

# PPID

Ver el proceso padre

```bash id="proc004"
echo $PPID
```

---

# Ver procesos

## ps

Mostrar procesos del usuario

```bash id="proc005"
ps
```

---

Mostrar todos los procesos

```bash id="proc006"
ps -ef
```

o

```bash id="proc007"
ps aux
```

---

# Interpretar ps aux

```bash id="proc008"
ps aux
```

Ejemplo

```text id="proc009"
USER   PID %CPU %MEM VSZ RSS TTY STAT START TIME COMMAND
```

| Campo   | Descripción               |
| ------- | ------------------------- |
| USER    | Usuario propietario       |
| PID     | Identificador del proceso |
| %CPU    | Uso de CPU                |
| %MEM    | Uso de memoria            |
| VSZ     | Memoria virtual           |
| RSS     | Memoria residente         |
| TTY     | Terminal asociada         |
| STAT    | Estado                    |
| START   | Hora de inicio            |
| TIME    | Tiempo de CPU utilizado   |
| COMMAND | Comando ejecutado         |

---

# Buscar procesos

```bash id="proc010"
ps -ef | grep postgres
```

También

```bash id="proc011"
pgrep postgres
```

Ver el comando completo

```bash id="proc012"
pgrep -a postgres
```

---

# pstree

Muestra los procesos en forma de árbol.

```bash id="proc013"
pstree
```

Mostrar PID

```bash id="proc014"
pstree -p
```

---

# top

Monitor de procesos en tiempo real.

```bash id="proc015"
top
```

Información mostrada:

* CPU
* Memoria
* Swap
* Procesos
* Carga del sistema

Salir

```text id="proc016"
q
```

---

# htop

Versión mejorada de `top`.

Instalar

Fedora

```bash id="proc017"
sudo dnf install htop
```

Ubuntu

```bash id="proc018"
sudo apt install htop
```

Ejecutar

```bash id="proc019"
htop
```

---

# Monitorear procesos específicos

```bash id="proc020"
top -p 2541
```

---

# pgrep

Buscar por nombre

```bash id="proc021"
pgrep sqlservr
```

Buscar usuario

```bash id="proc022"
pgrep -u postgres
```

---

# pidof

```bash id="proc023"
pidof sshd
```

---

# Estado de los procesos

| Estado | Significado   |
| ------ | ------------- |
| R      | Ejecutándose  |
| S      | Durmiendo     |
| D      | Espera de I/O |
| T      | Detenido      |
| Z      | Zombie        |
| I      | Idle (Kernel) |

---

# Finalizar procesos

## kill

```bash id="proc024"
kill PID
```

Ejemplo

```bash id="proc025"
kill 2541
```

---

# Señales más utilizadas

| Señal   | Número | Descripción            |
| ------- | ------ | ---------------------- |
| SIGTERM | 15     | Finalización normal    |
| SIGKILL | 9      | Finalización inmediata |
| SIGHUP  | 1      | Recargar configuración |
| SIGSTOP | 19     | Detener proceso        |
| SIGCONT | 18     | Reanudar proceso       |

---

# Finalizar forzosamente

```bash id="proc026"
kill -9 2541
```

---

# killall

Finalizar por nombre

```bash id="proc027"
killall firefox
```

---

# pkill

```bash id="proc028"
pkill postgres
```

Por usuario

```bash id="proc029"
pkill -u juan
```

---

# nice

Modificar prioridad al iniciar.

Valor

```text id="proc030"
-20

↓

0

↓

19
```

-20 = mayor prioridad

19 = menor prioridad

Ejemplo

```bash id="proc031"
nice -n 10 respaldo.sh
```

---

# renice

Modificar prioridad de un proceso existente.

```bash id="proc032"
sudo renice 5 -p 2541
```

---

# Ver prioridad

```bash id="proc033"
ps -o pid,ni,comm
```

---

# Procesos en Background

Ejecutar

```bash id="proc034"
firefox &
```

---

Ver trabajos

```bash id="proc035"
jobs
```

---

Enviar al fondo

```bash id="proc036"
bg
```

---

Traer al frente

```bash id="proc037"
fg
```

---

Detener temporalmente

Presionar

```text id="proc038"
Ctrl + Z
```

---

Finalizar

```text id="proc039"
Ctrl + C
```

---

# nohup

Mantener un proceso ejecutándose después de cerrar la sesión.

```bash id="proc040"
nohup respaldo.sh &
```

Salida

```text id="proc041"
nohup.out
```

---

# disown

Eliminar un trabajo del control del shell.

```bash id="proc042"
disown %1
```

---

# uptime

Carga del sistema

```bash id="proc043"
uptime
```

Ejemplo

```text id="proc044"
load average: 0.25 0.18 0.11
```

---

# vmstat

Uso de memoria

```bash id="proc045"
vmstat 2
```

---

# free

Ver memoria

```bash id="proc046"
free -h
```

---

# iostat

Uso de discos

```bash id="proc047"
iostat
```

Instalar

Fedora

```bash id="proc048"
sudo dnf install sysstat
```

Ubuntu

```bash id="proc049"
sudo apt install sysstat
```

---

# sar

Histórico de rendimiento

```bash id="proc050"
sar -u 1 5
```

---

# mpstat

Uso por CPU

```bash id="proc051"
mpstat -P ALL
```

---

# lsof

Ver archivos abiertos

```bash id="proc052"
lsof
```

Buscar un archivo

```bash id="proc053"
lsof archivo.txt
```

Puerto

```bash id="proc054"
lsof -i :5432
```

---

# Comandos más utilizados

| Comando   | Descripción                  |
| --------- | ---------------------------- |
| `ps`      | Ver procesos                 |
| `top`     | Monitor en tiempo real       |
| `htop`    | Monitor interactivo          |
| `pstree`  | Árbol de procesos            |
| `pgrep`   | Buscar procesos              |
| `pidof`   | Obtener PID                  |
| `kill`    | Finalizar proceso            |
| `killall` | Finalizar por nombre         |
| `pkill`   | Finalizar por patrón         |
| `nice`    | Cambiar prioridad al iniciar |
| `renice`  | Cambiar prioridad            |
| `jobs`    | Ver trabajos                 |
| `bg`      | Ejecutar en segundo plano    |
| `fg`      | Volver al primer plano       |
| `nohup`   | Mantener proceso activo      |
| `lsof`    | Archivos abiertos            |

---

# Archivos relacionados

| Archivo         | Función                   |
| --------------- | ------------------------- |
| `/proc/`        | Información de procesos   |
| `/proc/cpuinfo` | Información de CPU        |
| `/proc/meminfo` | Información de memoria    |
| `/proc/loadavg` | Carga del sistema         |
| `/proc/PID/`    | Información de un proceso |

---

# Buenas prácticas

* Finaliza procesos con `SIGTERM` (`kill`) antes de usar `SIGKILL` (`kill -9`).
* Utiliza `top` o `htop` para identificar procesos con alto consumo antes de intervenir.
* Evita modificar la prioridad de procesos críticos sin conocer su impacto.
* Usa `nohup` o un gestor de servicios (`systemd`) para procesos que deban seguir ejecutándose tras cerrar la sesión.
* Revisa periódicamente procesos en estado **Zombie (Z)** o con consumo excesivo de recursos.
* Utiliza `pgrep`, `pidof` o `ps` para localizar el proceso correcto antes de finalizarlo.

---

# Laboratorio práctico

## Ejercicio 1: Visualizar procesos

```bash id="labproc001"
ps -ef

top
```

---

## Ejercicio 2: Ejecutar un proceso en segundo plano

```bash id="labproc002"
sleep 300 &
```

Verificar

```bash id="labproc003"
jobs
```

---

## Ejercicio 3: Buscar un proceso

```bash id="labproc004"
pgrep sleep

ps -ef | grep sleep
```

---

## Ejercicio 4: Cambiar la prioridad

```bash id="labproc005"
nice -n 10 sleep 300 &
```

Verificar

```bash id="labproc006"
ps -o pid,ni,comm -C sleep
```

---

## Ejercicio 5: Finalizar un proceso

```bash id="labproc007"
kill $(pgrep sleep)
```

Si no finaliza correctamente:

```bash id="labproc008"
kill -9 $(pgrep sleep)
```

---

## Ejercicio 6: Ver archivos abiertos por PostgreSQL

```bash id="labproc009"
sudo lsof -c postgres
```

O para SQL Server:

```bash id="labproc010"
sudo lsof -c sqlservr
```

---

# Errores comunes

### Usar `kill -9` como primera opción

Incorrecto:

```bash id="errproc001"
kill -9 2541
```

Lo recomendable es intentar primero:

```bash id="errproc002"
kill 2541
```

y utilizar `kill -9` solo si el proceso no responde.

---

### Confundir `jobs` con `ps`

El comando:

```bash id="errproc003"
jobs
```

Solo muestra los trabajos iniciados desde la terminal actual.

Para ver todos los procesos del sistema utiliza:

```bash id="errproc004"
ps -ef
```

o

```bash id="errproc005"
top
```

---

### Cerrar la terminal y perder un proceso

Si ejecutas un proceso largo sin `nohup`, `screen`, `tmux` o un servicio de `systemd`, este finalizará al cerrar la sesión.

Ejemplo recomendado:

```bash id="errproc006"
nohup ./backup.sh &
```

---

# Resumen

En este capítulo aprendiste a:

* Comprender el ciclo de vida de los procesos en Linux.
* Visualizar y monitorear procesos con `ps`, `top`, `htop` y `pstree`.
* Buscar procesos utilizando `pgrep`, `pidof` y `ps`.
* Finalizar procesos mediante `kill`, `killall` y `pkill`.
* Administrar prioridades con `nice` y `renice`.
* Controlar procesos en primer y segundo plano con `jobs`, `bg`, `fg` y `nohup`.
* Analizar el uso de CPU, memoria y archivos abiertos con `vmstat`, `free`, `iostat`, `sar`, `mpstat` y `lsof`.
* Aplicar buenas prácticas para una administración segura y eficiente de los procesos en Linux.
