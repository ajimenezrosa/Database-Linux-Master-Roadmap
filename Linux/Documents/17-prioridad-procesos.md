# 17. Prioridad de Procesos en Linux

En Linux, cada proceso tiene una **prioridad de ejecución** que determina cuánto tiempo de CPU recibe en comparación con los demás procesos. Una correcta administración de prioridades permite optimizar el rendimiento del sistema, garantizar que los servicios críticos reciban recursos suficientes y evitar que procesos de baja importancia afecten el funcionamiento general del servidor.

En entornos empresariales es común ajustar la prioridad de procesos como **bases de datos**, **servidores web**, **respaldos**, **procesos ETL** y **tareas de mantenimiento**.

---

# Objetivos de este capítulo

Al finalizar esta unidad serás capaz de:

* Comprender cómo Linux asigna prioridades.
* Entender los conceptos de **Nice** y **Niceness**.
* Modificar la prioridad al iniciar un proceso.
* Cambiar la prioridad de procesos en ejecución.
* Interpretar los valores NI y PR.
* Monitorear prioridades con diferentes herramientas.
* Aplicar buenas prácticas en servidores Linux.

---

# ¿Qué es la prioridad de un proceso?

La prioridad determina la frecuencia con la que el planificador (Scheduler) asigna tiempo de CPU a un proceso.

Una prioridad alta significa que el proceso tendrá mayor oportunidad de ejecutarse.

Una prioridad baja implica que el proceso cederá más tiempo de CPU a otros procesos.

---

# El Scheduler de Linux

Linux utiliza un planificador de procesos (Scheduler) que decide:

* Qué proceso ejecutar.
* Durante cuánto tiempo.
* Cuándo cambiar entre procesos.
* Cómo distribuir la CPU entre todos los procesos.

El Scheduler busca un equilibrio entre:

* Rendimiento.
* Interactividad.
* Equidad.
* Tiempo de respuesta.

---

# Prioridad vs Nice

En Linux existen dos conceptos importantes:

* **PR (Priority)** → Prioridad interna utilizada por el kernel.
* **NI (Nice Value)** → Valor que el administrador puede modificar.

Normalmente el usuario administra únicamente el valor **Nice**.

---

# Valor Nice

El valor Nice controla la prioridad relativa de un proceso.

Rango:

```text id="prio001"
-20  -------------------->  19

Mayor prioridad        Menor prioridad
```

| Valor Nice | Prioridad |
| ---------- | --------- |
| -20        | Máxima    |
| -10        | Muy alta  |
| 0          | Normal    |
| 10         | Baja      |
| 19         | Muy baja  |

---

# Regla importante

Mientras menor sea el valor Nice:

Mayor prioridad tendrá el proceso.

Mientras mayor sea el valor Nice:

Menor prioridad tendrá el proceso.

---

# Ver prioridades

```bash id="prio002"
ps -eo pid,ni,pri,comm
```

Ejemplo

```text id="prio003"
PID   NI PRI COMMAND
2456   0  20 bash
3245   0  20 postgres
```

---

# Ver con top

```bash id="prio004"
top
```

Columnas importantes

```text id="prio005"
PR

NI
```

| Columna | Significado          |
| ------- | -------------------- |
| PR      | Prioridad del kernel |
| NI      | Valor Nice           |

---

# Ejecutar con prioridad diferente

Sintaxis

```bash id="prio006"
nice -n valor comando
```

Ejemplo

```bash id="prio007"
nice -n 10 backup.sh
```

---

Otro ejemplo

```bash id="prio008"
nice -n 19 tar -czf respaldo.tar.gz /datos
```

El proceso consumirá menos CPU.

---

# Ejecutar con mayor prioridad

Solo **root** puede utilizar valores negativos.

Ejemplo

```bash id="prio009"
sudo nice -n -10 proceso
```

---

# Cambiar prioridad de un proceso existente

```bash id="prio010"
sudo renice 10 -p 2541
```

---

Disminuir Nice (mayor prioridad)

```bash id="prio011"
sudo renice -5 -p 2541
```

---

# Verificar

```bash id="prio012"
ps -o pid,ni,pri,comm -p 2541
```

---

# Cambiar varios procesos

```bash id="prio013"
sudo renice 15 -p 1200 1201 1202
```

---

# Cambiar prioridad por usuario

```bash id="prio014"
sudo renice 5 -u postgres
```

---

# Cambiar prioridad por grupo

```bash id="prio015"
sudo renice 10 -g dba
```

---

# Ver únicamente NI

```bash id="prio016"
ps -eo pid,ni,comm
```

---

# Ver únicamente PR

```bash id="prio017"
ps -eo pid,pri,comm
```

---

# Monitorear en tiempo real

```bash id="prio018"
top
```

Ordenar por CPU

Presionar

```text id="prio019"
Shift + P
```

Ordenar por memoria

```text id="prio020"
Shift + M
```

---

# htop

Ejecutar

```bash id="prio021"
htop
```

Cambiar prioridad

Seleccionar el proceso.

Presionar

```text id="prio022"
F7
```

Reducir Nice (más prioridad).

```text id="prio023"
F8
```

Incrementar Nice (menos prioridad).

---

# Casos prácticos

## Respaldo nocturno

```bash id="prio024"
nice -n 19 respaldo.sh
```

No afecta a los usuarios.

---

## Compresión

```bash id="prio025"
nice -n 15 tar -czf respaldo.tar.gz datos/
```

---

## Consulta intensiva

```bash id="prio026"
nice -n 12 python analisis.py
```

---

## SQL Server

Generalmente **no** se recomienda modificar la prioridad del proceso principal.

Debe dejarse que el Scheduler del sistema operativo administre la CPU.

---

## PostgreSQL

Tampoco suele ser recomendable modificar el Nice del proceso principal del servidor, salvo indicaciones específicas del fabricante o necesidades muy particulares.

---

# Tiempo real (Real-Time)

Linux también soporta prioridades de tiempo real.

Políticas disponibles

* SCHED_FIFO
* SCHED_RR
* SCHED_DEADLINE

Ver política

```bash id="prio027"
chrt -p PID
```

---

Ejecutar en tiempo real

```bash id="prio028"
sudo chrt -f 50 programa
```

Debe utilizarse únicamente cuando sea estrictamente necesario.

---

# ionice

Controla la prioridad de acceso al disco.

Ver clases

```text id="prio029"
0 Ninguna

1 Tiempo Real

2 Mejor Esfuerzo

3 Idle
```

Ejemplo

```bash id="prio030"
ionice -c3 respaldo.sh
```

El respaldo utilizará el disco únicamente cuando el sistema esté inactivo.

---

# Combinar nice e ionice

```bash id="prio031"
nice -n 19 ionice -c3 respaldo.sh
```

Ideal para:

* Backups.
* Limpieza de logs.
* Compresión.
* Indexaciones.

---

# Ver el Scheduler

```bash id="prio032"
chrt -p $$
```

---

# Archivos relacionados

| Archivo           | Función                   |
| ----------------- | ------------------------- |
| `/proc/PID/stat`  | Información del proceso   |
| `/proc/PID/sched` | Información del Scheduler |
| `/proc/loadavg`   | Carga del sistema         |
| `/proc/cpuinfo`   | Información de CPU        |

---

# Comandos más utilizados

| Comando  | Descripción                        |
| -------- | ---------------------------------- |
| `nice`   | Iniciar proceso con otra prioridad |
| `renice` | Cambiar prioridad de un proceso    |
| `top`    | Ver prioridades                    |
| `htop`   | Administrar prioridades            |
| `ps`     | Mostrar NI y PR                    |
| `ionice` | Prioridad de disco                 |
| `chrt`   | Prioridad en tiempo real           |

---

# Diferencias entre Nice e ionice

| Nice                          | ionice                        |
| ----------------------------- | ----------------------------- |
| Controla el uso de CPU        | Controla el acceso al disco   |
| Utiliza valores de -20 a 19   | Utiliza clases de prioridad   |
| Se aplica al Scheduler de CPU | Se aplica al Scheduler de E/S |

---

# Buenas prácticas

* Mantén el valor **Nice** en `0` para la mayoría de los servicios del sistema.
* Incrementa el valor Nice (por ejemplo, `10` o `19`) para procesos de mantenimiento, respaldos y tareas no críticas.
* Evita asignar valores negativos salvo cuando exista una necesidad claramente justificada.
* No modifiques la prioridad de servicios críticos como SQL Server, PostgreSQL o el kernel sin comprender el impacto.
* Combina `nice` con `ionice` para reducir el impacto de tareas intensivas de CPU y disco.
* Supervisa el sistema con `top`, `htop` y `ps` antes y después de cambiar prioridades.

---

# Laboratorio práctico

## Ejercicio 1: Ver prioridades

```bash id="labprio001"
ps -eo pid,ni,pri,comm
```

---

## Ejercicio 2: Ejecutar un proceso con baja prioridad

```bash id="labprio002"
nice -n 15 sleep 300
```

---

## Ejercicio 3: Cambiar prioridad

Abrir otra terminal y localizar el PID:

```bash id="labprio003"
pgrep sleep
```

Modificar el valor Nice:

```bash id="labprio004"
sudo renice 5 -p PID
```

Verificar:

```bash id="labprio005"
ps -o pid,ni,pri,comm -p PID
```

---

## Ejercicio 4: Ejecutar un respaldo con poca prioridad

```bash id="labprio006"
nice -n 19 ionice -c3 tar -czf respaldo.tar.gz /home
```

---

## Ejercicio 5: Ver la política de planificación

```bash id="labprio007"
chrt -p $$
```

---

## Ejercicio 6: Monitorear prioridades

```bash id="labprio008"
top
```

Observa las columnas **PR** y **NI** mientras cambias la prioridad de un proceso.

---

# Errores comunes

### Asignar prioridad máxima a procesos innecesarios

Incorrecto:

```bash id="errprio001"
sudo nice -n -20 respaldo.sh
```

Esto puede afectar el rendimiento de otros procesos importantes.

---

### Pensar que Nice garantiza más CPU

El valor **Nice** influye en la prioridad del Scheduler, pero no garantiza un porcentaje fijo de CPU. Otros factores, como la carga del sistema y la política de planificación, también intervienen.

---

### Confundir `nice` con `ionice`

`nice` modifica la prioridad de uso de la **CPU**, mientras que `ionice` controla la prioridad de acceso al **disco**. Para tareas intensivas como respaldos o compresión, suele ser conveniente usar ambos.

---

# Resumen

En este capítulo aprendiste a:

* Comprender cómo Linux asigna prioridades mediante el Scheduler.
* Diferenciar entre **PR** (Priority) y **NI** (Nice).
* Ejecutar procesos con diferentes niveles de prioridad usando `nice`.
* Modificar la prioridad de procesos en ejecución con `renice`.
* Supervisar prioridades mediante `ps`, `top` y `htop`.
* Utilizar `ionice` para controlar la prioridad de acceso al disco.
* Conocer las políticas de tiempo real con `chrt`.
* Aplicar buenas prácticas para optimizar el rendimiento del sistema sin afectar la estabilidad del servidor.
