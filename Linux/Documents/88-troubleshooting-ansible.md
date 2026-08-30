# 88. Troubleshooting y Depuración de Playbooks (Fase 1)

> **Módulo 11 – Automatización con Ansible**
>
> **Archivo:** `88-troubleshooting-ansible.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Comprender la metodología de Troubleshooting en Ansible.
- Interpretar correctamente los mensajes de error.
- Utilizar distintos niveles de verbosidad.
- Validar la sintaxis de un Playbook.
- Localizar rápidamente el origen de un problema.
- Aplicar un proceso sistemático para resolver incidencias.
- Reducir considerablemente el tiempo de diagnóstico.

---

# Introducción

Uno de los mayores errores que cometen los administradores nuevos es ejecutar un Playbook, observar un mensaje de error y comenzar a modificar archivos al azar.

Ese enfoque rara vez funciona.

Los administradores experimentados siguen un procedimiento ordenado.

```text
Problema

↓

Analizar

↓

Reproducir

↓

Identificar

↓

Corregir

↓

Validar

↓

Documentar
```

Esta metodología evita introducir nuevos errores mientras se intenta solucionar el problema original.

---

# ¿Qué es Troubleshooting?

Troubleshooting es el proceso sistemático de identificar, analizar y corregir errores.

No consiste simplemente en "probar cosas".

Consiste en:

- Obtener información.
- Interpretarla.
- Formular una hipótesis.
- Confirmarla.
- Aplicar la solución.
- Verificar el resultado.

---

# Principio Fundamental

Nunca modifiques varias cosas al mismo tiempo.

Si cambias:

- Variables.
- Templates.
- Inventarios.
- Playbooks.

al mismo tiempo, nunca sabrás cuál cambio resolvió (o empeoró) el problema.

---

# Flujo Profesional

```text
Error

↓

Leer mensaje

↓

Identificar archivo

↓

Identificar línea

↓

Encontrar causa

↓

Aplicar solución

↓

Ejecutar nuevamente
```

---

# ¿Dónde aparecen los errores?

Los errores pueden originarse en diferentes componentes.

```text
Inventario

↓

Variables

↓

Playbook

↓

Task

↓

Template

↓

Role

↓

Módulo

↓

Servidor remoto
```

---

# Clasificación General

Los errores normalmente pertenecen a una de estas categorías.

| Categoría | Ejemplo |
|-----------|----------|
| Sintaxis | Error YAML |
| Variables | Undefined Variable |
| Inventario | Host inexistente |
| Permisos | Permission Denied |
| Conectividad | SSH Timeout |
| Templates | Error Jinja2 |
| Módulos | Parámetro inválido |
| Roles | Dependencias faltantes |

---

# Flujo de Diagnóstico

```text
Playbook

↓

¿Compila?

↓

SI

↓

¿Conecta?

↓

SI

↓

¿Ejecuta?

↓

SI

↓

¿Resultado correcto?

↓

Producción
```

---

# Primer Paso

Siempre leer completamente el mensaje.

Muchos administradores leen únicamente.

```text
FAILED
```

Pero ignoran toda la información posterior.

Generalmente el mensaje contiene exactamente la causa del problema.

---

# Ejemplo

```text
FAILED!

Permission denied
```

No significa que Ansible esté dañado.

Significa que existe un problema de permisos.

---

# Ejemplo

```text
FAILED!

No such file
```

No implica un error del módulo.

Generalmente el archivo realmente no existe.

---

# Ejemplo

```text
FAILED!

Host unreachable
```

El problema probablemente sea:

- Red.
- Firewall.
- SSH.
- DNS.
- Inventario.

No el Playbook.

---

# Validación de Sintaxis

Antes de ejecutar cualquier Playbook.

Debe verificarse la sintaxis.

Comando.

```bash
ansible-playbook playbook.yml --syntax-check
```

---

Proceso.

```text
Playbook

↓

Syntax Check

↓

Sin errores

↓

Ejecución
```

---

# Beneficios

- Detecta errores rápidamente.
- No modifica servidores.
- Es extremadamente rápido.
- Reduce tiempo de diagnóstico.

---

# Ejemplo

```bash
ansible-playbook apache.yml \
--syntax-check
```

Resultado.

```text
playbook: apache.yml
```

No significa que el Playbook funcione.

Únicamente que la sintaxis es correcta.

---

# Error de Sintaxis

Ejemplo.

```yaml
tasks

- name: Install Apache
```

Falta.

```yaml
tasks:
```

---

Resultado.

```text
Syntax Error
```

---

# Error YAML

Incorrecto.

```yaml
vars

http_port:80
```

Correcto.

```yaml
vars:

  http_port: 80
```

---

# YAML es Sensible

Errores comunes.

- Espacios.
- Indentación.
- Dos puntos.
- Guiones.
- Comillas.

---

# Diagrama

```text
YAML

↓

Parser

↓

Correcto

↓

Playbook
```

---

# Verbosidad

Ansible puede mostrar distintos niveles de información.

```text
-v

-vv

-vvv

-vvvv
```

---

# ¿Por qué existe?

Porque normalmente no necesitamos ver toda la información.

Sin embargo.

Cuando ocurre un problema.

Necesitamos muchos más detalles.

---

# Nivel 1

```bash
-v
```

Muestra información adicional.

Ideal para problemas sencillos.

---

# Nivel 2

```bash
-vv
```

Incluye mucha más información.

Por ejemplo.

- Variables.
- Tareas.
- Hosts.

---

# Nivel 3

```bash
-vvv
```

Muy utilizado por administradores Linux.

Permite observar.

- SSH.
- Ejecución.
- Variables.
- Flujo interno.

---

# Nivel 4

```bash
-vvvv
```

Nivel máximo de depuración.

Muestra prácticamente toda la comunicación.

Especialmente útil para problemas SSH.

---

# Comparación

| Nivel | Información |
|--------|-------------|
| Normal | Muy poca |
| -v | Baja |
| -vv | Media |
| -vvv | Alta |
| -vvvv | Muy detallada |

---

# Ejemplo

```bash
ansible-playbook site.yml \
-vvv
```

---

# Flujo

```text
Playbook

↓

-vvv

↓

Información detallada

↓

Diagnóstico
```

---

# Cuándo usar cada nivel

| Situación | Nivel recomendado |
|-----------|------------------|
| Ejecución normal | Sin opciones |
| Error sencillo | -v |
| Variables | -vv |
| Problemas SSH | -vvv |
| Diagnóstico avanzado | -vvvv |

---

# No abusar de -vvvv

Aunque proporciona mucha información.

También genera miles de líneas.

Por ello.

Debe utilizarse únicamente cuando sea necesario.

---

# Información que aparece

Con verbosidad elevada pueden observarse.

- Host.
- Usuario SSH.
- Puerto.
- Variables.
- Task.
- Resultado.
- Tiempo.
- Mensajes internos.

---

# Caso Empresarial

Servidor.

```text
server01
```

Error.

```text
UNREACHABLE
```

Primero.

```bash
ansible-playbook site.yml -v
```

Si continúa.

```bash
-vv
```

Luego.

```bash
-vvv
```

Finalmente.

```bash
-vvvv
```

Hasta encontrar la causa.

---

# Estrategia Recomendada

```text
Normal

↓

-v

↓

-vv

↓

-vvv

↓

-vvvv
```

No comenzar directamente con:

```text
-vvvv
```

---

# Buenas Prácticas

- Leer completamente el mensaje de error.
- Ejecutar primero `--syntax-check`.
- Corregir un problema a la vez.
- Incrementar la verbosidad gradualmente.
- Documentar el problema encontrado.
- Validar la solución antes de Producción.
- Confirmar siempre que el error fue realmente resuelto.
- Mantener una copia del Playbook antes de realizar cambios importantes.

---

# Errores Comunes

## Error 1

Ignorar el mensaje completo.

---

## Error 2

No ejecutar.

```bash
--syntax-check
```

---

## Error 3

Modificar varios archivos simultáneamente.

---

## Error 4

Comenzar directamente con.

```text
-vvvv
```

---

## Error 5

Suponer que todos los errores provienen del Playbook.

---

## Error 6

No revisar el inventario.

---

## Error 7

No verificar YAML.

---

## Error 8

No volver a ejecutar después de cada corrección.

---

## Error 9

No documentar la causa encontrada.

---

## Error 10

Declarar resuelto un problema sin validar el resultado final.

---

# Laboratorio RHCSA

## Escenario

Una empresa administra:

- 200 servidores Linux.
- 40 PostgreSQL.
- 25 Apache.

Un Playbook comienza a fallar.

---

## Laboratorio 1

Ejecutar.

```bash
ansible-playbook site.yml \
--syntax-check
```

---

## Laboratorio 2

Corregir un error de indentación.

---

## Laboratorio 3

Ejecutar.

```bash
-v
```

---

## Laboratorio 4

Ejecutar.

```bash
-vv
```

---

## Laboratorio 5

Ejecutar.

```bash
-vvv
```

---

## Laboratorio 6

Ejecutar.

```bash
-vvvv
```

---

## Laboratorio 7

Comparar la información obtenida en cada nivel de verbosidad.

---

## Laboratorio 8

Identificar si un error pertenece a:

- Inventario.
- Variables.
- SSH.
- YAML.
- Permisos.

---

## Laboratorio 9

Aplicar la metodología de diagnóstico paso a paso hasta resolver un error de conexión con un host remoto.

---

## Laboratorio 10

Documentar el problema encontrado indicando:

- Síntoma.
- Causa.
- Solución.
- Validación.
- Lecciones aprendidas.

---

# Preguntas de Repaso

1. ¿Qué es Troubleshooting?
2. ¿Por qué debe seguirse un procedimiento sistemático?
3. ¿Qué categorías de errores son las más comunes?
4. ¿Para qué sirve `--syntax-check`?
5. ¿Qué diferencia existe entre `-v` y `-vvvv`?
6. ¿Cuándo conviene utilizar cada nivel de verbosidad?
7. ¿Por qué YAML produce tantos errores?
8. ¿Qué información proporciona la verbosidad avanzada?
9. ¿Por qué no deben modificarse varios elementos simultáneamente?
10. ¿Cómo organizarías un proceso profesional de diagnóstico en Ansible?

---

# Resumen

En esta primera fase estudiamos la metodología profesional de **Troubleshooting** aplicada a Ansible. Aprendimos a clasificar errores, interpretar correctamente los mensajes mostrados por Ansible, validar la sintaxis mediante `--syntax-check` y utilizar de forma progresiva los distintos niveles de verbosidad (`-v`, `-vv`, `-vvv` y `-vvvv`) para obtener la información necesaria sin generar ruido innecesario.

En la **Fase 2** aprenderemos a utilizar el módulo **`debug`**, inspeccionar Variables, comprender la estructura de los resultados (`register`), interpretar códigos de retorno (`rc`), analizar salidas (`stdout` y `stderr`) y construir Playbooks mucho más fáciles de diagnosticar y mantener.

----

# 88. Troubleshooting y Depuración de Playbooks (Fase 2)

> **Módulo 11 – Automatización con Ansible**
>
> **Archivo:** `88-troubleshooting-ansible.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Utilizar el módulo `debug`.
- Inspeccionar Variables durante la ejecución.
- Utilizar `register`.
- Analizar `stdout`, `stderr` y `rc`.
- Comprender la estructura de los resultados de un módulo.
- Diagnosticar errores de ejecución.
- Construir Playbooks mucho más fáciles de mantener.

---

# Introducción

Una de las mayores ventajas de Ansible es que prácticamente todo lo que ejecuta puede inspeccionarse.

Esto permite responder preguntas como:

- ¿Qué Variable tiene este valor?
- ¿Qué devolvió un comando?
- ¿Falló realmente?
- ¿Cuál fue el código de retorno?
- ¿Qué escribió el programa en pantalla?

Para ello utilizaremos dos herramientas fundamentales:

- `debug`
- `register`

---

# El módulo debug

Es uno de los módulos más utilizados durante el desarrollo.

Su función consiste en mostrar información durante la ejecución.

Sintaxis.

```yaml
- name: Mostrar mensaje

  debug:

    msg: "Hola Mundo"
```

---

Resultado.

```text
ok: [server01] =>

msg:

Hola Mundo
```

---

# ¿Cuándo utilizar debug?

Cuando necesitamos conocer.

- Variables.
- Resultados.
- Decisiones.
- Flujo del Playbook.
- Información temporal.

---

# Arquitectura

```text
Playbook

↓

Variable

↓

debug

↓

Pantalla
```

---

# Mostrar una Variable

```yaml
- debug:

    var: ansible_hostname
```

---

Resultado.

```text
ansible_hostname:

server01
```

---

# Mostrar múltiples Variables

```yaml
- debug:

    msg:

      - "Host: {{ ansible_hostname }}"

      - "IP: {{ ansible_default_ipv4.address }}"

      - "Usuario: {{ ansible_user }}"
```

---

Resultado.

```text
Host: server01

IP: 192.168.1.25

Usuario: ansible
```

---

# debug con msg

```yaml
- debug:

    msg: "Instalación completada correctamente."
```

---

# debug con var

```yaml
- debug:

    var: ansible_distribution
```

---

¿Cuál utilizar?

| Opción | Uso |
|---------|-----|
| msg | Mostrar texto personalizado |
| var | Mostrar el contenido completo de una Variable |

---

# Introducción a register

Muchos módulos devuelven información.

Si no la almacenamos.

Se pierde.

Para conservarla utilizamos.

```yaml
register:
```

---

# Arquitectura

```text
Módulo

↓

Resultado

↓

register

↓

Variable
```

---

# Ejemplo

```yaml
- name: Ver Kernel

  command: uname -r

  register: kernel
```

---

Ahora.

Toda la salida quedó almacenada en.

```text
kernel
```

---

# Visualizar el Resultado

```yaml
- debug:

    var: kernel
```

---

Resultado simplificado.

```text
kernel:

changed: true

stdout: 6.14.5

stderr:

rc: 0
```

---

# ¿Qué contiene register?

Generalmente.

```text
changed

stdout

stderr

rc

failed

cmd

start

end

delta
```

Dependiendo del módulo.

---

# Diagrama

```text
Command Module

↓

register

↓

stdout

stderr

rc

changed
```

---

# stdout

Es la salida estándar.

Ejemplo.

```bash
hostname
```

Resultado.

```text
server01
```

En Ansible.

```text
stdout
```

---

Ejemplo.

```yaml
- debug:

    var: kernel.stdout
```

---

Resultado.

```text
6.14.5
```

---

# stderr

Es la salida de errores.

Ejemplo.

```text
Permission denied
```

---

Visualización.

```yaml
- debug:

    var: kernel.stderr
```

---

# rc

Significa.

```text
Return Code
```

o.

```text
Código de Retorno
```

---

En Linux.

Normalmente.

```text
0

↓

Éxito
```

---

Cualquier otro valor.

Generalmente.

```text
Error
```

---

Ejemplo.

```yaml
- debug:

    var: kernel.rc
```

---

Resultado.

```text
0
```

---

# Interpretación

```text
rc = 0

↓

Correcto
```

---

```text
rc = 1

↓

Error
```

---

Aunque algunos programas utilizan otros códigos.

Siempre debe consultarse su documentación.

---

# changed

Indica si la tarea modificó el sistema.

Ejemplo.

```text
changed: true
```

---

O.

```text
changed: false
```

---

Muy útil para comprobar la idempotencia.

---

# failed

También existe.

```text
failed
```

---

Ejemplo.

```text
failed: false
```

---

Si ocurre un error.

```text
failed: true
```

---

# Ejemplo Completo

```yaml
- name: Ver espacio

  command: df -h

  register: disco

- debug:

    var: disco.stdout
```

---

Resultado.

```text
Filesystem

Size

Used

Avail
```

---

# Otro Ejemplo

```yaml
- name: Fecha

  command: date

  register: fecha

- debug:

    var: fecha.stdout
```

---

Resultado.

```text
Mon Jul 27

14:20:35
```

---

# register con módulos

No solamente funciona con.

```text
command
```

También.

- shell
- copy
- file
- package
- service
- user
- yum
- dnf

Y prácticamente cualquier módulo.

---

# Ejemplo

```yaml
- name: Instalar Apache

  dnf:

    name: httpd

    state: present

  register: apache
```

---

Después.

```yaml
- debug:

    var: apache
```

---

# Acceso a Campos

Supongamos.

```yaml
register: resultado
```

Podemos consultar.

```text
resultado.stdout

resultado.stderr

resultado.rc

resultado.changed

resultado.failed
```

---

# Flujo Completo

```text
Task

↓

register

↓

Variable

↓

debug

↓

Pantalla
```

---

# Diagnóstico

Supongamos.

```yaml
- command: ls /tmp

  register: salida
```

Luego.

```yaml
- debug:

    var: salida.stderr
```

Si aparece.

```text
No such file
```

Ya conocemos exactamente el problema.

---

# Uso durante el Desarrollo

Es muy común agregar.

```yaml
debug
```

Mientras desarrollamos.

Y posteriormente eliminarlo.

---

# No abusar de debug

Demasiados mensajes producen.

- Salidas largas.
- Difícil lectura.
- Menor productividad.

---

Recomendación.

Utilizarlo únicamente cuando sea necesario.

---

# Caso Empresarial

Playbook.

```text
100 Tasks
```

Task 73 falla.

El administrador agrega.

```yaml
register
```

y.

```yaml
debug
```

únicamente alrededor de esa Task.

Así obtiene toda la información necesaria sin llenar la salida con cientos de mensajes.

---

# Comparación

| Herramienta | Función |
|------------|---------|
| debug | Mostrar información |
| register | Guardar resultados |
| stdout | Salida estándar |
| stderr | Mensajes de error |
| rc | Código de retorno |
| changed | Indica cambios |
| failed | Indica fallo |

---

# Buenas Prácticas

- Utilizar nombres descriptivos para `register`.
- Mostrar únicamente la información necesaria.
- Revisar `stdout` antes de asumir un error.
- Consultar `stderr` cuando falle una tarea.
- Validar `rc`.
- Eliminar `debug` innecesarios antes de Producción.
- Mantener los Playbooks limpios.
- Documentar problemas encontrados.
- Utilizar `register` para reutilizar resultados en tareas posteriores.
- Aprovechar `changed` para validar la idempotencia.

---

# Errores Comunes

## Error 1

Olvidar utilizar.

```yaml
register
```

---

## Error 2

Mostrar toda la Variable cuando sólo interesa.

```text
stdout
```

---

## Error 3

Ignorar.

```text
stderr
```

---

## Error 4

No revisar.

```text
rc
```

---

## Error 5

Nombrar Variables como.

```text
resultado1

resultado2

dato3
```

---

## Error 6

Dejar decenas de módulos `debug` en Producción.

---

## Error 7

Suponer que `changed: true` significa éxito.

Puede existir un error posteriormente.

---

## Error 8

No interpretar correctamente la salida del comando ejecutado.

---

## Error 9

No revisar la documentación del módulo para conocer todos los campos devueltos.

---

## Error 10

No reutilizar la información almacenada mediante `register` en tareas posteriores.

---

# Laboratorio RHCSA

## Escenario

Un administrador necesita diagnosticar varios Playbooks.

---

## Laboratorio 1

Crear un Playbook que ejecute.

```bash
hostname
```

y almacene el resultado mediante `register`.

---

## Laboratorio 2

Mostrar únicamente.

```text
stdout
```

---

## Laboratorio 3

Mostrar únicamente.

```text
stderr
```

---

## Laboratorio 4

Mostrar.

```text
rc
```

---

## Laboratorio 5

Ejecutar.

```bash
df -h
```

y mostrar el resultado.

---

## Laboratorio 6

Instalar Apache utilizando el módulo `dnf` y revisar el valor de `changed`.

---

## Laboratorio 7

Ejecutar un comando inexistente y analizar:

- `stderr`
- `rc`
- `failed`

---

## Laboratorio 8

Comparar los resultados devueltos por los módulos `command` y `shell`, identificando qué información es común y qué diferencias existen.

---

## Laboratorio 9

Construir un Playbook que almacene los resultados de varias tareas utilizando `register` y genere un resumen final mediante `debug`.

---

## Laboratorio 10

Desarrollar un procedimiento de diagnóstico utilizando `register`, `debug`, `stdout`, `stderr` y `rc` para localizar el origen de un error en un Playbook de instalación de servicios.

---

# Preguntas de Repaso

1. ¿Qué función cumple el módulo `debug`?
2. ¿Qué hace `register`?
3. ¿Qué información suele almacenar una Variable registrada?
4. ¿Qué representa `stdout`?
5. ¿Qué representa `stderr`?
6. ¿Qué significa `rc`?
7. ¿Qué indica `changed`?
8. ¿Qué indica `failed`?
9. ¿Cuándo conviene eliminar los módulos `debug`?
10. ¿Cómo utilizarías `register` para simplificar el diagnóstico de un Playbook complejo?

---

# Resumen

En esta segunda fase aprendimos a utilizar el módulo **`debug`** para mostrar información durante la ejecución de un Playbook y el mecanismo **`register`** para almacenar los resultados devueltos por cualquier módulo. También estudiamos cómo interpretar los campos más importantes de un resultado (`stdout`, `stderr`, `rc`, `changed` y `failed`) y cómo utilizarlos para comprender exactamente qué ocurrió durante la ejecución de una tarea.

En la **Fase 3** estudiaremos el manejo avanzado de errores mediante `failed_when`, `changed_when`, `ignore_errors`, bloques `block`, `rescue` y `always`, construyendo Playbooks mucho más robustos y preparados para recuperarse automáticamente ante fallos.

----

# 88. Troubleshooting y Depuración de Playbooks (Fase 3)

> **Módulo 11 – Automatización con Ansible**
>
> **Archivo:** `88-troubleshooting-ansible.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Controlar cuándo una tarea debe considerarse exitosa o fallida.
- Utilizar `failed_when`.
- Utilizar `changed_when`.
- Comprender `ignore_errors`.
- Implementar bloques `block`, `rescue` y `always`.
- Diseñar Playbooks resistentes a fallos.
- Automatizar procesos de recuperación.

---

# Introducción

Hasta ahora hemos aprendido a:

- Leer mensajes de error.
- Utilizar distintos niveles de verbosidad.
- Analizar variables mediante `debug`.
- Utilizar `register`.

Pero aún existe un problema.

Por defecto Ansible decide automáticamente cuándo una tarea:

- Fue exitosa.
- Falló.
- Modificó el sistema.

Sin embargo.

En muchas ocasiones el administrador necesita modificar ese comportamiento.

Para ello existen herramientas muy poderosas.

- `failed_when`
- `changed_when`
- `ignore_errors`
- `block`
- `rescue`
- `always`

---

# Flujo Tradicional

```text
Task

↓

Resultado

↓

Ansible decide

↓

Success o Failed
```

---

# Flujo Avanzado

```text
Task

↓

Resultado

↓

Administrador decide

↓

Success o Failed
```

---

# ¿Qué es failed_when?

Permite definir manualmente cuándo una tarea debe considerarse un error.

---

# Sintaxis

```yaml
failed_when:
```

---

# Ejemplo

Supongamos.

```yaml
- command: df -h

  register: disco

  failed_when: false
```

Aunque el comando devuelva un código inesperado.

Nunca marcará error.

---

# Otro Ejemplo

```yaml
- command: cat archivo.txt

  register: resultado

  failed_when:

    resultado.rc != 0
```

---

Representación.

```text
Task

↓

Register

↓

Evaluar condición

↓

FAILED
```

---

# Ventajas

- Mayor control.
- Automatización inteligente.
- Mejor diagnóstico.
- Menos falsos positivos.

---

# failed_when con Texto

Supongamos.

El comando devuelve.

```text
Database Offline
```

---

Podemos detectar.

```yaml
failed_when:

  "'Database Offline' in resultado.stdout"
```

---

Proceso.

```text
stdout

↓

Buscar texto

↓

Encontrado

↓

FAILED
```

---

# Múltiples Condiciones

```yaml
failed_when:

  - resultado.rc != 0

  - "'ERROR' in resultado.stdout"
```

---

También.

```yaml
failed_when:

  resultado.rc != 0 or

  "ERROR" in resultado.stdout
```

---

# ¿Qué es changed_when?

Permite decidir cuándo una tarea realmente modificó el sistema.

---

Sintaxis.

```yaml
changed_when:
```

---

Ejemplo

```yaml
- command: uptime

  changed_when: false
```

---

¿Por qué?

Porque.

```bash
uptime
```

No modifica absolutamente nada.

---

Resultado.

```text
changed: false
```

---

# Caso Real

Muchos comandos administrativos únicamente consultan información.

No deberían aparecer como.

```text
changed: true
```

---

Ejemplo

```yaml
- command: hostname

  changed_when: false
```

---

Otro ejemplo.

```yaml
- command: cat /etc/os-release

  changed_when: false
```

---

# Beneficios

- Reportes más precisos.
- Mayor claridad.
- Idempotencia.
- Mejor auditoría.

---

# ignore_errors

Algunas veces deseamos continuar.

Aunque ocurra un error.

---

Sintaxis.

```yaml
ignore_errors: true
```

---

Ejemplo.

```yaml
- command: cat archivo_que_no_existe

  ignore_errors: true
```

---

Resultado.

```text
FAILED

↓

Continuar
```

---

Diagrama.

```text
Task

↓

FAILED

↓

ignore_errors

↓

Próxima Task
```

---

# ¿Cuándo utilizarlo?

Por ejemplo.

Consultar un archivo opcional.

Si existe.

Excelente.

Si no.

Continuar.

---

# Mala Práctica

Utilizar.

```yaml
ignore_errors: true
```

En todas las Tasks.

---

Eso oculta problemas reales.

---

# Block

Permite agrupar tareas relacionadas.

---

Sintaxis.

```yaml
block:
```

---

Ejemplo.

```yaml
- block:

    - name: Instalar Apache

      dnf:

        name: httpd

        state: present

    - name: Iniciar Servicio

      service:

        name: httpd

        state: started
```

---

Representación.

```text
Task 1

↓

Task 2

↓

Task 3
```

---

# Ventajas

- Organización.
- Legibilidad.
- Recuperación.
- Reutilización.

---

# Rescue

Se ejecuta únicamente si falla el bloque.

---

Arquitectura.

```text
Block

↓

Error

↓

Rescue
```

---

Ejemplo.

```yaml
- block:

    - command: false

  rescue:

    - debug:

        msg: "Ocurrió un error."
```

---

Flujo.

```text
Block

↓

FAILED

↓

Rescue

↓

Continuar
```

---

# Caso Empresarial

Instalación.

```text
Apache

↓

Error

↓

Eliminar paquetes

↓

Registrar incidente

↓

Continuar
```

---

# Ejemplo

```yaml
- block:

    - name: Instalar PostgreSQL

      dnf:

        name: postgresql-server

        state: present

  rescue:

    - debug:

        msg: "Instalación fallida."
```

---

# Always

Existe un tercer bloque.

```yaml
always:
```

---

Se ejecuta siempre.

Sin importar el resultado.

---

Diagrama.

```text
Block

↓

Success

↓

Always
```

---

También.

```text
Block

↓

FAILED

↓

Rescue

↓

Always
```

---

# Ejemplo

```yaml
- block:

    - command: hostname

  always:

    - debug:

        msg: "Finalizó la ejecución."
```

---

# Caso Empresarial

Siempre registrar.

- Fecha.
- Hora.
- Servidor.
- Resultado.

Aunque ocurra un error.

---

# Arquitectura Completa

```text
Block

↓

Success

↓

Always
```

---

```text
Block

↓

FAILED

↓

Rescue

↓

Always
```

---

# Recuperación Automática

Supongamos.

```text
Instalar Apache

↓

Error

↓

Eliminar archivos temporales

↓

Registrar Log

↓

Continuar
```

---

# Ejemplo Completo

```yaml
- block:

    - name: Instalar Apache

      dnf:

        name: httpd

        state: present

  rescue:

    - name: Mostrar error

      debug:

        msg: "Error instalando Apache."

  always:

    - name: Finalizar

      debug:

        msg: "Proceso terminado."
```

---

# Comparación

| Elemento | Función |
|----------|---------|
| failed_when | Define cuándo una tarea falla |
| changed_when | Define cuándo hubo cambios |
| ignore_errors | Continúa tras un error |
| block | Agrupa tareas |
| rescue | Recuperación tras un fallo |
| always | Siempre se ejecuta |

---

# Flujo Empresarial

```text
Playbook

↓

Block

↓

Task

↓

Error

↓

Rescue

↓

Registrar

↓

Always

↓

Finalizar
```

---

# Casos de Uso

| Situación | Herramienta recomendada |
|-----------|-------------------------|
| Personalizar un fallo | `failed_when` |
| Evitar falsos cambios | `changed_when` |
| Continuar tras un error no crítico | `ignore_errors` |
| Agrupar tareas | `block` |
| Recuperarse automáticamente | `rescue` |
| Ejecutar limpieza o auditoría | `always` |

---

# Buenas Prácticas

- Utilizar `failed_when` únicamente cuando sea realmente necesario.
- Mantener las condiciones simples y fáciles de leer.
- Utilizar `changed_when` para comandos de consulta.
- Evitar abusar de `ignore_errors`.
- Agrupar tareas relacionadas mediante `block`.
- Implementar acciones de recuperación con `rescue`.
- Registrar información importante mediante `always`.
- Probar todos los escenarios posibles.
- Documentar las condiciones personalizadas.
- Validar la idempotencia después de implementar cambios.

---

# Errores Comunes

## Error 1

Utilizar.

```yaml
ignore_errors: true
```

en todo el Playbook.

---

## Error 2

Crear condiciones demasiado complejas en `failed_when`.

---

## Error 3

Olvidar utilizar `changed_when` para tareas que únicamente consultan información.

---

## Error 4

No implementar un bloque `rescue` cuando existe una posible recuperación.

---

## Error 5

Duplicar código en varios bloques.

---

## Error 6

No registrar el resultado final mediante `always`.

---

## Error 7

No probar el flujo de recuperación.

---

## Error 8

Ocultar errores críticos mediante `ignore_errors`.

---

## Error 9

No documentar la lógica utilizada en condiciones personalizadas.

---

## Error 10

Suponer que `rescue` reemplaza una estrategia adecuada de monitoreo y registro.

---

# Laboratorio RHCSA

## Escenario

Una empresa administra más de 500 servidores Linux mediante Ansible.

Desea que sus Playbooks sean capaces de recuperarse automáticamente ante fallos.

---

## Laboratorio 1

Crear un Playbook que utilice.

```yaml
failed_when
```

---

## Laboratorio 2

Modificar una tarea de consulta utilizando.

```yaml
changed_when: false
```

---

## Laboratorio 3

Ejecutar un comando inexistente utilizando.

```yaml
ignore_errors: true
```

---

## Laboratorio 4

Crear un.

```yaml
block
```

con tres tareas.

---

## Laboratorio 5

Agregar un.

```yaml
rescue
```

que muestre un mensaje mediante `debug`.

---

## Laboratorio 6

Agregar un.

```yaml
always
```

que registre el final de la ejecución.

---

## Laboratorio 7

Diseñar un proceso de instalación de Apache que, ante un fallo, elimine archivos temporales y registre el incidente antes de finalizar.

---

## Laboratorio 8

Crear una condición `failed_when` basada en el contenido de `stdout` en lugar del código de retorno.

---

## Laboratorio 9

Construir un bloque que instale PostgreSQL, valide el servicio y, si alguna tarea falla, ejecute automáticamente un procedimiento de recuperación.

---

## Laboratorio 10

Desarrollar un Playbook robusto que combine:

- `register`
- `debug`
- `failed_when`
- `changed_when`
- `block`
- `rescue`
- `always`

y documentar el flujo completo de recuperación.

---

# Preguntas de Repaso

1. ¿Qué función cumple `failed_when`?
2. ¿Cuándo utilizarías `changed_when`?
3. ¿Qué riesgos existen al abusar de `ignore_errors`?
4. ¿Qué ventajas ofrece `block`?
5. ¿En qué momento se ejecuta `rescue`?
6. ¿Cuál es la diferencia entre `rescue` y `always`?
7. ¿Por qué es importante registrar el resultado final de un proceso?
8. ¿Cómo mejora `changed_when` la idempotencia?
9. ¿Qué escenarios justifican una recuperación automática?
10. ¿Cómo diseñarías un Playbook resistente a fallos para un entorno empresarial?

---

# Resumen

En esta tercera fase aprendimos a controlar el comportamiento de Ansible frente a errores mediante **`failed_when`**, **`changed_when`** e **`ignore_errors`**, personalizando cuándo una tarea debe considerarse exitosa, fallida o modificada. También estudiamos los bloques **`block`**, **`rescue`** y **`always`**, que permiten agrupar tareas, implementar mecanismos de recuperación automática y ejecutar acciones obligatorias como auditorías o limpieza de recursos.

En la **Fase 4** integraremos todas estas técnicas en metodologías profesionales de troubleshooting, analizaremos casos reales de fallos en inventarios, Variables, Roles, Templates y conexiones SSH, y construiremos un laboratorio integral de diagnóstico para entornos empresariales con Ansible.

----

# 88. Troubleshooting y Depuración de Playbooks (Fase 4)

> **Módulo 11 – Automatización con Ansible**
>
> **Archivo:** `88-troubleshooting-ansible.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Aplicar una metodología profesional de Troubleshooting.
- Diagnosticar problemas reales en Playbooks.
- Resolver errores relacionados con Inventarios, Variables, Roles, Templates y SSH.
- Construir procedimientos repetibles de diagnóstico.
- Implementar buenas prácticas de soporte empresarial.
- Diseñar Playbooks fáciles de mantener y depurar.

---

# Introducción

Durante este capítulo hemos aprendido:

- Verbosidad (`-v`, `-vv`, `-vvv`, `-vvvv`)
- `debug`
- `register`
- `failed_when`
- `changed_when`
- `block`
- `rescue`
- `always`

Ahora integraremos todos estos conocimientos para crear una metodología profesional de resolución de problemas.

---

# Filosofía del Troubleshooting

Cuando un Playbook falla, el objetivo no es corregirlo lo más rápido posible.

El objetivo es encontrar la causa raíz.

Una solución temporal puede ocultar el problema durante días o semanas.

La solución definitiva elimina la causa.

---

# Metodología Profesional

```text
Problema

↓

Reproducir

↓

Recolectar Evidencia

↓

Analizar

↓

Identificar Causa

↓

Aplicar Solución

↓

Validar

↓

Documentar

↓

Cerrar Incidente
```

---

# Regla Número Uno

Nunca asumir.

Siempre verificar.

Ejemplo.

Incorrecto.

```text
Debe ser un problema SSH.
```

Correcto.

```text
Voy a comprobar SSH.
```

---

# Regla Número Dos

Modificar únicamente una variable a la vez.

Incorrecto.

```text
Editar Inventario

Editar Variables

Editar Templates

Editar Roles

Editar SSH

Todo al mismo tiempo
```

Nunca sabrás qué solucionó el problema.

---

# Procedimiento General

## Paso 1

Validar sintaxis.

```bash
ansible-playbook site.yml \
--syntax-check
```

---

## Paso 2

Verificar Inventario.

```bash
ansible-inventory \
--list
```

---

## Paso 3

Comprobar conectividad.

```bash
ansible all \
-m ping
```

---

## Paso 4

Incrementar verbosidad.

```bash
-v
```

↓

```bash
-vv
```

↓

```bash
-vvv
```

↓

```bash
-vvvv
```

---

## Paso 5

Agregar.

```yaml
debug
```

si es necesario.

---

## Paso 6

Aplicar solución.

---

## Paso 7

Ejecutar nuevamente.

---

## Paso 8

Documentar.

---

# Árbol de Diagnóstico

```text
                Playbook Falla

                      │

          ┌───────────┴────────────┐

          ▼                        ▼

   Error de Sintaxis         Sintaxis Correcta

                                    │

                                    ▼

                         ¿Conecta al Host?

                              │

                ┌─────────────┴─────────────┐

                ▼                           ▼

              NO                            SI

                │                           │

         Revisar SSH                 Revisar Variables

         Revisar Firewall            Revisar Tasks

         Revisar Inventario          Revisar Roles

                                     Revisar Templates
```

---

# Caso 1

## Error de Inventario

Mensaje.

```text
Host not found
```

Posibles causas.

- Nombre incorrecto.
- Grupo inexistente.
- Archivo equivocado.
- Inventario no cargado.

---

Diagnóstico.

```bash
ansible-inventory --graph
```

---

# Caso 2

## Error SSH

Mensaje.

```text
UNREACHABLE
```

Verificar.

- Puerto.
- Usuario.
- Firewall.
- Llaves SSH.
- DNS.
- Red.

---

Comando.

```bash
ansible all \
-m ping \
-vvv
```

---

# Caso 3

## Variable no definida

Mensaje.

```text
Undefined Variable
```

Posibles causas.

- Error tipográfico.
- Variable inexistente.
- group_vars.
- host_vars.
- defaults.
- vars.

---

Diagnóstico.

```yaml
- debug:

    var: variable
```

---

# Caso 4

## Error Template

Mensaje.

```text
Template Error
```

Generalmente.

- Error Jinja2.
- Variable inexistente.
- Llaves incorrectas.
- Filtro inexistente.

---

Ejemplo.

Incorrecto.

```jinja2
{{ usuario
```

Correcto.

```jinja2
{{ usuario }}
```

---

# Caso 5

## Error de Permisos

Mensaje.

```text
Permission denied
```

Posibles causas.

- Usuario incorrecto.
- sudo.
- become.
- SELinux.
- Permisos Linux.

---

Verificar.

```yaml
become: true
```

---

# Caso 6

## Role no encontrado

Mensaje.

```text
Role not found
```

Comprobar.

```bash
ansible-galaxy role list
```

---

También.

```text
roles/
```

---

# Caso 7

## Collection inexistente

Mensaje.

```text
Module not found
```

Verificar.

```bash
ansible-galaxy collection list
```

---

# Caso 8

## YAML Incorrecto

Mensaje.

```text
Syntax Error
```

Generalmente.

- Espacios.
- Dos puntos.
- Indentación.
- Tabulaciones.

---

# Caso 9

## Servicio no inicia

Task.

```yaml
service:

  name: httpd

  state: started
```

---

Diagnóstico.

```yaml
register: servicio
```

Después.

```yaml
debug:

  var: servicio
```

---

# Caso 10

## Comando devuelve Error

```yaml
command: systemctl status httpd
```

---

Registrar.

```yaml
register: resultado
```

---

Mostrar.

```yaml
stdout

stderr

rc
```

---

# Flujo Completo

```text
Task

↓

register

↓

stdout

↓

stderr

↓

debug

↓

Diagnóstico
```

---

# Checklist Profesional

Antes de escalar un incidente.

```text
□ Inventario

□ SSH

□ Firewall

□ Variables

□ Templates

□ Roles

□ Collections

□ YAML

□ Verbosidad

□ Logs

□ register

□ debug
```

---

# Documentación del Incidente

Toda incidencia debería registrar.

| Campo | Descripción |
|--------|-------------|
| Fecha | Momento del incidente |
| Servidor | Host afectado |
| Playbook | Archivo ejecutado |
| Error | Mensaje recibido |
| Causa | Problema identificado |
| Solución | Acción aplicada |
| Validación | Evidencia de éxito |
| Responsable | Administrador que resolvió |

---

# Estrategia Empresarial

```text
Incidente

↓

Diagnóstico

↓

Corrección

↓

Pruebas

↓

Documentación

↓

Base de Conocimiento
```

---

# Base de Conocimiento

Las organizaciones maduras documentan todos los incidentes.

Ejemplo.

```text
Error

↓

Causa

↓

Solución

↓

Fecha

↓

Administrador
```

Esto evita investigar el mismo problema repetidamente.

---

# Integración con CI/CD

Antes de Producción.

```text
Commit

↓

Lint

↓

Syntax Check

↓

Tests

↓

Playbook

↓

Producción
```

Muchos problemas pueden detectarse antes de llegar a los servidores.

---

# Monitoreo

Después del despliegue.

```text
Playbook

↓

Servicios

↓

Logs

↓

Alertas

↓

Administrador
```

El Troubleshooting no termina cuando finaliza el Playbook.

También incluye validar que el servicio funcione correctamente.

---

# Estrategia para Grandes Infraestructuras

```text
Miles de Servidores

↓

Playbooks

↓

Logs Centralizados

↓

Monitoreo

↓

Alertas

↓

Troubleshooting

↓

Corrección
```

---

# Caso Empresarial

Una empresa administra.

- 4.000 servidores Linux.
- 700 PostgreSQL.
- 500 SQL Server.
- 300 Kubernetes.

Todos los Playbooks siguen exactamente la misma metodología.

```text
Validar

↓

Diagnosticar

↓

Corregir

↓

Documentar
```

Esto permite que cualquier administrador pueda continuar el trabajo iniciado por otro miembro del equipo.

---

# Buenas Prácticas

- Leer completamente el mensaje de error.
- Confirmar siempre la causa antes de modificar el código.
- Validar primero la sintaxis.
- Utilizar `register` para conservar resultados importantes.
- Incrementar la verbosidad de forma gradual.
- Utilizar `debug` únicamente durante el diagnóstico.
- Documentar todas las incidencias relevantes.
- Mantener una base de conocimiento compartida.
- Probar las soluciones en un laboratorio antes de Producción.
- Automatizar verificaciones mediante CI/CD.
- Diseñar Playbooks pequeños y fáciles de depurar.
- Revisar la documentación oficial del módulo cuando exista alguna duda.

---

# Errores Comunes

## Error 1

Suponer la causa del problema sin comprobarla.

---

## Error 2

Modificar múltiples archivos simultáneamente.

---

## Error 3

Ignorar el mensaje completo mostrado por Ansible.

---

## Error 4

No utilizar `--syntax-check`.

---

## Error 5

No revisar el inventario.

---

## Error 6

No comprobar la conectividad SSH antes de investigar el Playbook.

---

## Error 7

No registrar evidencia del incidente.

---

## Error 8

Eliminar los mensajes de `debug` antes de finalizar el diagnóstico.

---

## Error 9

Cerrar un incidente sin validar completamente el servicio.

---

## Error 10

Resolver el problema pero no documentarlo para futuras incidencias.

---

# Laboratorio Integral RHCSA

## Escenario

Una empresa administra una infraestructura compuesta por:

- 600 servidores Linux.
- 120 PostgreSQL.
- 90 Apache.
- 50 Nginx.

Durante un despliegue comienzan a aparecer múltiples errores.

---

## Laboratorio 1

Ejecutar.

```bash
ansible-playbook site.yml \
--syntax-check
```

---

## Laboratorio 2

Validar el inventario utilizando.

```bash
ansible-inventory --graph
```

---

## Laboratorio 3

Comprobar conectividad.

```bash
ansible all -m ping
```

---

## Laboratorio 4

Incrementar gradualmente la verbosidad hasta localizar el origen del problema.

---

## Laboratorio 5

Agregar módulos `debug` únicamente en las tareas críticas.

---

## Laboratorio 6

Utilizar `register` para analizar un servicio que no inicia correctamente.

---

## Laboratorio 7

Implementar un bloque `block/rescue/always` que registre el incidente y ejecute tareas de recuperación.

---

## Laboratorio 8

Resolver un error provocado por una Variable inexistente dentro de un Template Jinja2.

---

## Laboratorio 9

Crear un procedimiento estándar de Troubleshooting que pueda seguir cualquier administrador del equipo.

---

## Laboratorio 10 (Proyecto Final)

Diseñar una metodología corporativa de resolución de incidencias que incluya:

- Validación de sintaxis.
- Verificación de Inventarios.
- Diagnóstico SSH.
- Uso de `debug`.
- Uso de `register`.
- Verbosidad.
- Manejo de errores.
- Recuperación automática.
- Documentación.
- Validación posterior al despliegue.
- Incorporación de la solución a una base de conocimiento.

---

# Preguntas de Repaso

1. ¿Cuál es la primera acción que debe realizarse cuando falla un Playbook?
2. ¿Por qué no deben modificarse varios componentes simultáneamente?
3. ¿Cómo diagnosticarías un error de Inventario?
4. ¿Cómo identificarías un problema relacionado con SSH?
5. ¿Qué información proporciona `register` durante una investigación?
6. ¿Qué ventajas ofrece documentar los incidentes?
7. ¿Por qué es importante validar el servicio después de ejecutar un Playbook?
8. ¿Cómo ayuda CI/CD a reducir problemas en Producción?
9. ¿Qué elementos debería contener una base de conocimiento de incidencias?
10. ¿Cómo diseñarías un procedimiento estándar de Troubleshooting para un equipo de administradores Linux?

---

# Resumen del Capítulo

En este capítulo estudiamos las técnicas fundamentales y avanzadas para el **Troubleshooting de Playbooks en Ansible**. Aprendimos a validar la sintaxis, interpretar mensajes de error, utilizar distintos niveles de verbosidad, inspeccionar Variables mediante `debug`, analizar resultados con `register` y controlar el comportamiento de las tareas mediante `failed_when`, `changed_when`, `ignore_errors`, `block`, `rescue` y `always`.

También desarrollamos una metodología profesional de diagnóstico basada en la identificación de la causa raíz, la validación sistemática, la documentación de incidencias y la automatización de procesos de recuperación. Estas prácticas permiten construir Playbooks más robustos, fáciles de mantener y adecuados para infraestructuras empresariales de gran escala.

---

# Próximo Capítulo

## **89. Proyecto Final de Automatización con Ansible**

En el siguiente capítulo integraremos todos los conocimientos del módulo para construir una solución empresarial completa. Diseñaremos una infraestructura automatizada utilizando Inventarios, Variables, Roles, Templates, Handlers, Vault, Collections, Galaxy, manejo de errores, buenas prácticas, CI/CD y procedimientos de Troubleshooting, siguiendo una arquitectura similar a la utilizada por equipos profesionales de administración Linux y DevOps.

-----






