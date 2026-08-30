# 81. Comandos Ad-Hoc en Ansible (Fase 1)

> **Módulo 11 – Automatización con Ansible**
>
> **Archivo:** `81-comandos-ad-hoc.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Comprender qué son los Comandos Ad-Hoc.
- Entender cuándo utilizarlos y cuándo preferir un Playbook.
- Conocer la arquitectura de ejecución de Ansible.
- Aprender la sintaxis general de los comandos Ad-Hoc.
- Utilizar correctamente los módulos `ping`, `command`, `shell` y `raw`.
- Comprender las diferencias entre estos módulos.
- Ejecutar comandos sobre uno o cientos de servidores.
- Aplicar buenas prácticas utilizadas en entornos empresariales.

---

# Introducción

Hasta ahora hemos aprendido:

- Cómo instalar Ansible.
- Cómo configurar el Control Node.
- Cómo organizar un proyecto.
- Cómo crear Inventarios.

El siguiente paso consiste en comenzar a administrar servidores.

La forma más rápida de hacerlo es mediante los **Comandos Ad-Hoc**.

---

# ¿Qué es un Comando Ad-Hoc?

Un Comando Ad-Hoc es una instrucción que ejecuta una única tarea sobre uno o varios servidores sin necesidad de crear un Playbook.

Podemos compararlo con ejecutar un comando directamente por SSH, pero de manera simultánea sobre decenas o cientos de equipos.

---

# Analogía

Supongamos que debemos verificar el espacio en disco de 100 servidores.

### Sin Ansible

```text
SSH servidor1

Ejecutar df -h

Salir

SSH servidor2

Ejecutar df -h

Salir

...

Repetir 100 veces
```

---

### Con Ansible

```bash
ansible all -m command -a "df -h"
```

Una única instrucción ejecutará el comando en todos los servidores.

---

# ¿Cuándo utilizar Comandos Ad-Hoc?

Son ideales para tareas rápidas.

Ejemplos:

- Verificar conectividad.
- Reiniciar un servicio.
- Consultar memoria.
- Revisar espacio en disco.
- Instalar un paquete.
- Crear un usuario.
- Reiniciar un servidor.
- Obtener información.

---

# ¿Cuándo NO utilizarlos?

No son la mejor opción cuando:

- Existen muchas tareas.
- Se requiere reutilización.
- Hay condiciones complejas.
- Se necesita control de versiones.
- La automatización será recurrente.

En esos casos es recomendable utilizar un **Playbook**.

---

# Comparación

| Comandos Ad-Hoc | Playbooks |
|-----------------|-----------|
| Una tarea | Muchas tareas |
| Rápidos | Más estructurados |
| No reutilizables | Reutilizables |
| Ideales para soporte | Ideales para automatización |
| Poco código | Automatización completa |

---

# Arquitectura

```text
             Administrador

                    │

                    ▼

             Comando Ad-Hoc

                    │

                    ▼

               Inventario

                    │

          SSH hacia los Hosts

                    │

      ┌─────────────┼─────────────┐

      ▼             ▼             ▼

    web01         web02         db01
```

---

# Flujo de Ejecución

```text
Comando

↓

Inventory

↓

Grupo

↓

SSH

↓

Python

↓

Módulo

↓

Resultado
```

---

# Sintaxis General

```bash
ansible <HOSTS> -m <MODULO> -a "<ARGUMENTOS>"
```

---

# Componentes

```text
ansible

↓

Hosts

↓

Módulo

↓

Argumentos
```

---

# Ejemplo

```bash
ansible web -m ping
```

---

Significado

| Elemento | Descripción |
|----------|-------------|
| ansible | Ejecutable |
| web | Grupo del Inventory |
| ping | Módulo |

---

# Selección de Hosts

Podemos ejecutar comandos sobre:

Un Host

```bash
ansible web01 -m ping
```

---

Un Grupo

```bash
ansible web -m ping
```

---

Todos los servidores

```bash
ansible all -m ping
```

---

Varios grupos

```bash
ansible web:database -m ping
```

---

Excluir grupos

```bash
ansible all:!database -m ping
```

---

# Selección mediante patrones

Ansible soporta patrones.

Ejemplo

```bash
ansible "web*" -m ping
```

---

También

```bash
ansible "db*" -m ping
```

---

# Inventario específico

```bash
ansible all -i inventory.ini -m ping
```

---

# Usuario remoto

```bash
ansible all -u ansible -m ping
```

---

# Become (sudo)

```bash
ansible all -b -m command -a "dnf update"
```

---

# Solicitar contraseña sudo

```bash
ansible all -K -m command -a "dnf update"
```

---

# El módulo ping

Es el módulo más utilizado para comprobar conectividad.

Importante:

No utiliza ICMP.

No ejecuta el comando Linux `ping`.

---

# ¿Qué verifica?

Comprueba que:

- SSH funciona.
- Python está instalado.
- El módulo puede ejecutarse.
- El servidor responde correctamente.

---

# Ejemplo

```bash
ansible all -m ping
```

---

Resultado esperado

```text
web01 | SUCCESS

{

    "changed": false,

    "ping": "pong"

}
```

---

# Interpretación

```text
pong

↓

SSH correcto

↓

Python correcto

↓

Ansible operativo
```

---

# Error típico

```text
UNREACHABLE
```

Puede indicar:

- SSH detenido.
- Firewall.
- Usuario incorrecto.
- Clave SSH inválida.

---

# Módulo command

Permite ejecutar comandos directamente.

Sintaxis

```bash
ansible all -m command -a "hostname"
```

---

Ejemplo

```bash
ansible all -m command -a "uptime"
```

---

Otro ejemplo

```bash
ansible all -m command -a "df -h"
```

---

Consultar memoria

```bash
ansible all -m command -a "free -m"
```

---

Consultar Kernel

```bash
ansible all -m command -a "uname -r"
```

---

Consultar versión

```bash
ansible all -m command -a "cat /etc/os-release"
```

---

# Características

El módulo `command`:

- No utiliza un Shell.
- Es más seguro.
- No interpreta operadores.
- Es el recomendado cuando solo se ejecuta un comando.

---

# Módulo shell

Permite ejecutar comandos utilizando el Shell del sistema.

Ejemplo

```bash
ansible all -m shell -a "df -h | grep /"
```

---

Otro ejemplo

```bash
ansible all -m shell -a "echo Hola > /tmp/prueba.txt"
```

---

También

```bash
ansible all -m shell -a "ls /tmp/*.log"
```

---

# ¿Por qué utilizar shell?

Porque soporta:

- Pipes.
- Redirecciones.
- Variables.
- Operadores.
- Expansión del Shell.

---

Ejemplo

```bash
echo Hola > archivo.txt
```

Con `command`

❌ No funciona.

Con `shell`

✅ Funciona.

---

# Pipes

```bash
ps aux | grep httpd
```

Requiere `shell`.

---

# Redirecciones

```bash
echo prueba > archivo.txt
```

Requiere `shell`.

---

# Variables

```bash
echo $HOME
```

Requiere `shell`.

---

# Wildcards

```bash
ls *.log
```

Requiere `shell`.

---

# Módulo raw

Existe un tercer módulo.

```text
raw
```

---

# ¿Qué hace?

Ejecuta comandos sin utilizar Python.

---

# Arquitectura

```text
Ansible

↓

SSH

↓

Comando

↓

Servidor
```

---

# ¿Cuándo utilizar raw?

Principalmente cuando:

- Python no está instalado.
- El servidor acaba de instalarse.
- Se necesita instalar Python.

---

Ejemplo

```bash
ansible all -m raw -a "dnf install python3 -y"
```

---

# Comparación

| Módulo | Usa Shell | Requiere Python | Uso recomendado |
|---------|-----------|-----------------|-----------------|
| command | No | Sí | Comandos simples |
| shell | Sí | Sí | Pipes y redirecciones |
| raw | Sí | No | Bootstrap de servidores |

---

# ¿Cuál utilizar?

```text
¿Necesita Pipes?

↓

Sí

↓

shell

↓

No

↓

¿Python instalado?

↓

Sí

↓

command

↓

No

↓

raw
```

---

# Ejemplos Empresariales

Consultar espacio libre

```bash
ansible all -m command -a "df -h"
```

---

Consultar memoria

```bash
ansible all -m command -a "free -m"
```

---

Consultar usuarios

```bash
ansible all -m command -a "who"
```

---

Consultar procesos

```bash
ansible all -m shell -a "ps -ef | grep postgres"
```

---

Buscar archivos

```bash
ansible all -m shell -a "find /var/log -name '*.log'"
```

---

Verificar versión de Python

```bash
ansible all -m command -a "python3 --version"
```

---

Verificar SELinux

```bash
ansible all -m command -a "getenforce"
```

---

Verificar Firewall

```bash
ansible all -m command -a "firewall-cmd --state"
```

---

Consultar tiempo activo

```bash
ansible all -m command -a "uptime"
```

---

# Buenas Prácticas

- Utilizar `command` siempre que sea posible.
- Utilizar `shell` únicamente cuando realmente sea necesario.
- Utilizar `raw` solo para inicializar servidores sin Python.
- Ejecutar primero sobre un servidor de prueba.
- Verificar siempre el resultado antes de continuar.
- Utilizar grupos en lugar de Hosts individuales cuando corresponda.
- Documentar comandos utilizados frecuentemente.

---

# Errores Comunes

## Error 1

Utilizar `shell` cuando `command` es suficiente.

---

## Error 2

Intentar utilizar Pipes con `command`.

```bash
ps aux | grep ssh
```

No funcionará correctamente.

---

## Error 3

Intentar utilizar variables del Shell con `command`.

---

## Error 4

Ejecutar comandos peligrosos sobre `all` sin verificar el alcance.

---

## Error 5

Utilizar `raw` cuando Python ya está instalado.

---

## Error 6

No utilizar `become` cuando el comando requiere privilegios administrativos.

---

## Error 7

No comprobar previamente la conectividad con `ansible all -m ping`.

---

# Laboratorio RHCSA

## Laboratorio 1

Comprobar la conectividad de todos los servidores utilizando:

```bash
ansible all -m ping
```

---

## Laboratorio 2

Mostrar el nombre del Host en todos los servidores.

---

## Laboratorio 3

Consultar el tiempo de actividad (`uptime`) de todos los equipos.

---

## Laboratorio 4

Mostrar el uso del disco mediante `df -h`.

---

## Laboratorio 5

Consultar la memoria utilizando `free -m`.

---

## Laboratorio 6

Buscar procesos relacionados con `sshd` utilizando el módulo `shell`.

---

## Laboratorio 7

Mostrar la versión del Kernel en todos los servidores.

---

## Laboratorio 8

Consultar el estado de SELinux.

---

## Laboratorio 9

Ejecutar un comando utilizando `become`.

---

## Laboratorio 10

Comparar el comportamiento de los módulos `command`, `shell` y `raw`, documentando:

- Casos de uso.
- Ventajas.
- Limitaciones.
- Riesgos.
- Recomendaciones para producción.

---

# Resumen

En esta primera fase aprendimos qué son los **Comandos Ad-Hoc**, cómo se ejecutan, cuál es su arquitectura y cuándo utilizarlos. Estudiamos la sintaxis general de Ansible y los módulos fundamentales `ping`, `command`, `shell` y `raw`, comprendiendo sus diferencias, ventajas y escenarios de uso. Estos comandos constituyen la base para la administración rápida de servidores Linux antes de avanzar hacia automatizaciones más complejas mediante Playbooks.

En la **Fase 2** exploraremos los módulos especializados en la administración de archivos, como `copy`, `fetch`, `file`, `stat`, `template`, `lineinfile`, `replace` y `blockinfile`, aplicándolos a escenarios reales de administración de sistemas.

----

# 81. Comandos Ad-Hoc en Ansible (Fase 2)

> **Módulo 11 – Automatización con Ansible**
>
> **Archivo:** `81-comandos-ad-hoc.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Administrar archivos remotamente utilizando Ansible.
- Copiar archivos hacia los servidores.
- Descargar archivos desde servidores remotos.
- Crear, eliminar y modificar archivos y directorios.
- Obtener información detallada de archivos.
- Administrar archivos de configuración.
- Modificar archivos de forma segura.
- Comprender cuándo utilizar cada módulo relacionado con archivos.

---

# Introducción

Una de las tareas más comunes de un administrador Linux consiste en administrar archivos.

Ejemplos:

- Copiar configuraciones.
- Respaldar archivos.
- Crear directorios.
- Cambiar permisos.
- Modificar configuraciones.
- Distribuir certificados.
- Actualizar banners.
- Instalar archivos de configuración.

Ansible dispone de varios módulos especializados para realizar estas tareas de forma segura e idempotente.

---

# Arquitectura

```text
                 Control Node

                      │

                      ▼

             Comando Ad-Hoc

                      │

                      ▼

                 Módulo File

                      │

      ┌───────────────┼───────────────┐

      ▼               ▼               ▼

    Web01           Web02           DB01
```

---

# Módulos estudiados

Durante esta fase aprenderemos:

| Módulo | Función |
|---------|----------|
| copy | Copiar archivos al servidor remoto |
| fetch | Descargar archivos desde el servidor remoto |
| file | Administrar archivos y directorios |
| stat | Obtener información de archivos |
| template | Copiar plantillas Jinja2 |
| lineinfile | Modificar una línea |
| replace | Reemplazar texto |
| blockinfile | Insertar bloques completos |

---

# Módulo copy

Permite copiar archivos desde el Control Node hacia uno o varios servidores.

Es uno de los módulos más utilizados.

---

# Sintaxis

```bash
ansible all -m copy -a "src=archivo dest=/tmp/"
```

---

# Flujo

```text
Control Node

↓

archivo

↓

copy

↓

Servidor Linux
```

---

# Ejemplo

Copiar un archivo.

```bash
ansible web -m copy -a "src=index.html dest=/var/www/html/index.html"
```

---

# Cambiar propietario

```bash
ansible web -m copy \
-a "src=index.html dest=/var/www/html/index.html owner=apache group=apache"
```

---

# Cambiar permisos

```bash
ansible web -m copy \
-a "src=index.html dest=/var/www/html/index.html mode=0644"
```

---

# Crear respaldo automáticamente

```bash
ansible web -m copy \
-a "src=httpd.conf dest=/etc/httpd/conf/httpd.conf backup=yes"
```

---

# Forzar reemplazo

```bash
force=yes
```

---

# Evitar reemplazo

```bash
force=no
```

---

# Beneficios

- Idempotente.
- Conserva permisos.
- Puede realizar respaldos.
- Muy rápido.
- Seguro.

---

# Módulo fetch

Realiza el proceso inverso.

Descarga archivos desde los servidores hacia el Control Node.

---

# Arquitectura

```text
Servidor Linux

↓

Archivo

↓

fetch

↓

Control Node
```

---

# Ejemplo

```bash
ansible web -m fetch \
-a "src=/etc/hosts dest=backup/"
```

---

# Resultado

```text
backup/

├── web01/

│   └── etc/

├── web02/

│   └── etc/

└── web03/

    └── etc/
```

Cada servidor mantiene su propia copia.

---

# Casos de uso

- Respaldar configuraciones.
- Descargar logs.
- Obtener archivos de auditoría.
- Copiar certificados.
- Recolectar evidencias.

---

# Módulo file

Permite administrar:

- Archivos.
- Directorios.
- Enlaces simbólicos.
- Permisos.
- Propietarios.

---

# Crear un directorio

```bash
ansible all -m file \
-a "path=/opt/scripts state=directory"
```

---

# Crear un archivo vacío

```bash
ansible all -m file \
-a "path=/tmp/prueba.txt state=touch"
```

---

# Eliminar un archivo

```bash
ansible all -m file \
-a "path=/tmp/prueba.txt state=absent"
```

---

# Cambiar permisos

```bash
ansible all -m file \
-a "path=/opt/scripts mode=0755"
```

---

# Cambiar propietario

```bash
ansible all -m file \
-a "path=/opt/scripts owner=root group=root"
```

---

# Crear enlace simbólico

```bash
ansible all -m file \
-a "src=/var/www/html dest=/web state=link"
```

---

# Estados soportados

| Estado | Descripción |
|----------|-------------|
| file | Archivo |
| directory | Directorio |
| touch | Archivo vacío |
| absent | Eliminar |
| link | Enlace simbólico |
| hard | Enlace duro |

---

# Módulo stat

Obtiene información detallada de un archivo.

No modifica nada.

---

# Ejemplo

```bash
ansible web -m stat \
-a "path=/etc/passwd"
```

---

# Información obtenida

```text
Existe

↓

Permisos

↓

Propietario

↓

Grupo

↓

Checksum

↓

Tamaño

↓

Fecha modificación
```

---

# Casos de uso

- Verificar existencia.
- Comparar archivos.
- Auditoría.
- Validaciones.

---

# Módulo template

Similar a `copy`.

La diferencia es que utiliza Jinja2.

Permite generar archivos dinámicos.

---

# Flujo

```text
Template

↓

Variables

↓

Jinja2

↓

Archivo Final

↓

Servidor
```

---

# Ejemplo

```bash
ansible web -m template \
-a "src=httpd.conf.j2 dest=/etc/httpd/conf/httpd.conf"
```

---

# ¿Cuándo utilizar template?

Cuando el archivo cambia dependiendo del servidor.

Ejemplo

```text
Hostname

↓

Puerto

↓

IP

↓

Variables
```

Cada servidor recibirá un archivo diferente.

---

# Módulo lineinfile

Permite modificar una única línea.

Muy utilizado.

---

# Ejemplo

```bash
ansible all -m lineinfile \
-a "path=/etc/ssh/sshd_config line='PermitRootLogin no'"
```

---

# Buscar una línea

```bash
regexp="^PermitRootLogin"
```

---

# Reemplazar

```text
PermitRootLogin yes

↓

PermitRootLogin no
```

---

# Beneficios

- No reemplaza el archivo completo.
- Seguro.
- Idempotente.
- Muy rápido.

---

# Módulo replace

Permite reemplazar texto utilizando expresiones regulares.

---

# Ejemplo

```bash
ansible all -m replace \
-a "path=/etc/httpd/conf/httpd.conf regexp='Listen 80' replace='Listen 8080'"
```

---

# Arquitectura

```text
Archivo

↓

Buscar

↓

Regex

↓

Reemplazar

↓

Guardar
```

---

# Casos de uso

- Cambiar puertos.
- Cambiar rutas.
- Cambiar dominios.
- Sustituir configuraciones.

---

# Módulo blockinfile

Inserta bloques completos.

Muy útil cuando una configuración requiere varias líneas.

---

# Ejemplo

```bash
ansible all -m blockinfile \
-a "path=/etc/hosts block='
192.168.100.10 dns01
192.168.100.20 web01
'"
```

---

# Resultado

```text
# BEGIN ANSIBLE MANAGED BLOCK

192.168.100.10 dns01

192.168.100.20 web01

# END ANSIBLE MANAGED BLOCK
```

---

# Ventajas

- Fácil eliminación.
- Fácil actualización.
- Claramente identificable.
- Seguro.

---

# Comparación de módulos

| Módulo | Uso principal |
|---------|---------------|
| copy | Copiar archivos |
| fetch | Descargar archivos |
| file | Crear y administrar archivos |
| stat | Consultar información |
| template | Archivos dinámicos |
| lineinfile | Modificar una línea |
| replace | Sustituir texto |
| blockinfile | Insertar bloques |

---

# Escenario Empresarial

Una empresa necesita:

- Actualizar `/etc/motd`.
- Copiar certificados SSL.
- Cambiar permisos.
- Crear directorios.
- Descargar logs.
- Modificar `sshd_config`.

Todos estos cambios pueden realizarse mediante los módulos estudiados.

---

# Flujo Empresarial

```text
Administrador

↓

Ansible

↓

copy

↓

lineinfile

↓

template

↓

file

↓

150 servidores
```

---

# Buenas prácticas

- Utilizar `template` para archivos dinámicos.
- Utilizar `copy` únicamente para archivos estáticos.
- Utilizar `lineinfile` para cambios pequeños.
- Utilizar `replace` cuando existan múltiples coincidencias.
- Utilizar `blockinfile` para bloques completos.
- Verificar permisos después de copiar archivos.
- Crear respaldos antes de modificar configuraciones críticas.

---

# Errores comunes

## Error 1

Utilizar `copy` para archivos que contienen variables.

Debe utilizarse `template`.

---

## Error 2

Reemplazar un archivo completo cuando solo debe modificarse una línea.

---

## Error 3

No conservar permisos originales.

---

## Error 4

Modificar archivos críticos sin crear respaldos.

---

## Error 5

Utilizar expresiones regulares demasiado amplias en `replace`.

---

## Error 6

Insertar manualmente bloques que luego serán administrados por Ansible.

---

## Error 7

No verificar la existencia del archivo antes de modificarlo.

El módulo `stat` puede ayudar a evitar este problema.

---

# Laboratorio RHCSA

## Laboratorio 1

Copiar un archivo desde el Control Node hacia todos los servidores utilizando `copy`.

---

## Laboratorio 2

Crear el directorio:

```text
/opt/empresa
```

utilizando el módulo `file`.

---

## Laboratorio 3

Crear un archivo vacío llamado:

```text
/opt/empresa/control.txt
```

---

## Laboratorio 4

Modificar los permisos del directorio a:

```text
0755
```

---

## Laboratorio 5

Descargar `/etc/hosts` de todos los servidores mediante `fetch`.

---

## Laboratorio 6

Verificar si existe el archivo:

```text
/etc/ssh/sshd_config
```

utilizando `stat`.

---

## Laboratorio 7

Modificar la línea `PermitRootLogin` utilizando `lineinfile`.

---

## Laboratorio 8

Cambiar el puerto `Listen 80` por `Listen 8080` mediante `replace`.

---

## Laboratorio 9

Insertar un bloque administrado por Ansible en `/etc/hosts` utilizando `blockinfile`.

---

## Laboratorio 10

Comparar los módulos `copy`, `template`, `lineinfile`, `replace` y `blockinfile`, indicando:

- Cuándo utilizar cada uno.
- Ventajas.
- Limitaciones.
- Riesgos.
- Escenarios recomendados para producción.

---

# Resumen

En esta segunda fase estudiamos los principales módulos de Ansible para la administración de archivos. Aprendimos a copiar archivos (`copy`), descargarlos (`fetch`), crear y administrar archivos y directorios (`file`), obtener información (`stat`), generar configuraciones dinámicas (`template`) y modificar archivos de forma segura mediante `lineinfile`, `replace` y `blockinfile`. Estos módulos constituyen la base para la gestión de configuraciones y archivos en entornos Linux administrados con Ansible.

En la **Fase 3** exploraremos los módulos orientados a la administración del sistema, como `dnf`, `package`, `service`, `systemd`, `user`, `group`, `cron` y `reboot`, aplicándolos a escenarios reales de administración masiva de servidores Linux.

-----

# 81. Comandos Ad-Hoc en Ansible (Fase 3)

> **Módulo 11 – Automatización con Ansible**
>
> **Archivo:** `81-comandos-ad-hoc.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Administrar paquetes remotamente.
- Instalar, actualizar y eliminar software.
- Administrar servicios del sistema.
- Controlar servicios mediante systemd.
- Crear y administrar usuarios y grupos.
- Automatizar tareas programadas (Cron).
- Reiniciar servidores de forma controlada.
- Ejecutar tareas administrativas sobre cientos de servidores simultáneamente.

---

# Introducción

En la administración diaria de servidores Linux, las tareas más frecuentes suelen ser:

- Instalar software.
- Actualizar paquetes.
- Reiniciar servicios.
- Crear usuarios.
- Agregar grupos.
- Programar tareas.
- Reiniciar servidores.

Todas estas operaciones pueden ejecutarse mediante **Comandos Ad-Hoc**, permitiendo administrar cientos o miles de servidores desde un único Control Node.

---

# Arquitectura

```text
               Administrador

                     │

                     ▼

             Comando Ad-Hoc

                     │

                     ▼

              Módulo Ansible

                     │

     ┌───────────────┼───────────────┐

     ▼               ▼               ▼

   Web01           DB01          Backup01
```

---

# Módulos estudiados

Durante esta fase aprenderemos:

| Módulo | Función |
|---------|----------|
| dnf | Administrar paquetes DNF |
| yum | Compatibilidad con YUM |
| package | Administrador genérico |
| service | Administrar servicios |
| systemd | Administrar servicios systemd |
| user | Administrar usuarios |
| group | Administrar grupos |
| cron | Administrar tareas programadas |
| reboot | Reiniciar servidores |

---

# Módulo dnf

Permite administrar paquetes en:

- Fedora
- Red Hat Enterprise Linux 8+
- Rocky Linux
- AlmaLinux
- CentOS Stream

Es el módulo recomendado para sistemas modernos basados en RPM.

---

# Instalar un paquete

```bash
ansible all -m dnf \
-a "name=httpd state=present" -b
```

---

# Instalar varios paquetes

```bash
ansible all -m dnf \
-a "name=httpd,git,wget state=present" -b
```

---

# Instalar la última versión

```bash
ansible all -m dnf \
-a "name=httpd state=latest" -b
```

---

# Eliminar un paquete

```bash
ansible all -m dnf \
-a "name=httpd state=absent" -b
```

---

# Actualizar todos los paquetes

```bash
ansible all -m dnf \
-a "name=* state=latest" -b
```

---

# Limpiar caché

```bash
ansible all -m command \
-a "dnf clean all" -b
```

---

# Estados soportados

| Estado | Descripción |
|----------|-------------|
| present | Instalar si no existe |
| latest | Actualizar a la versión más reciente |
| absent | Desinstalar |

---

# Idempotencia

Supongamos que Apache ya está instalado.

```bash
state=present
```

Resultado

```text
changed = false
```

Ansible no realizará ningún cambio innecesario.

---

# Módulo yum

Históricamente utilizado en:

- RHEL 7
- CentOS 7

Actualmente continúa existiendo por compatibilidad.

Ejemplo

```bash
ansible all -m yum \
-a "name=httpd state=present" -b
```

---

# ¿Cuál utilizar?

| Sistema | Recomendado |
|----------|-------------|
| Fedora | dnf |
| RHEL 8/9 | dnf |
| Rocky | dnf |
| AlmaLinux | dnf |
| CentOS 7 | yum |

---

# Módulo package

Es un módulo genérico.

Ansible detecta automáticamente el administrador de paquetes.

```bash
ansible all -m package \
-a "name=httpd state=present" -b
```

---

# Ventajas

```text
Playbook

↓

Package

↓

DNF

↓

YUM

↓

APT

↓

Zypper
```

El mismo código funciona en distintas distribuciones.

---

# Módulo service

Permite administrar servicios.

Ejemplo

```bash
ansible all -m service \
-a "name=httpd state=started" -b
```

---

# Iniciar un servicio

```bash
state=started
```

---

# Detener un servicio

```bash
state=stopped
```

---

# Reiniciar

```bash
state=restarted
```

---

# Recargar configuración

```bash
state=reloaded
```

---

# Habilitar al iniciar

```bash
enabled=yes
```

---

# Deshabilitar

```bash
enabled=no
```

---

# Ejemplo

```bash
ansible web -m service \
-a "name=httpd state=started enabled=yes" -b
```

---

# Módulo systemd

Especializado para sistemas modernos.

Permite utilizar características avanzadas de systemd.

---

# Ejemplo

```bash
ansible all -m systemd \
-a "name=sshd state=restarted" -b
```

---

# Recargar daemon

```bash
ansible all -m systemd \
-a "daemon_reload=yes" -b
```

---

# Enmascarar un servicio

```bash
masked=yes
```

---

# Desenmascarar

```bash
masked=no
```

---

# Diferencias

| service | systemd |
|-----------|----------|
| Compatible | Específico de systemd |
| Simple | Más completo |
| Portátil | Funciones avanzadas |

---

# Módulo user

Permite administrar usuarios.

---

# Crear usuario

```bash
ansible all -m user \
-a "name=alejandro state=present" -b
```

---

# Crear usuario con Shell

```bash
ansible all -m user \
-a "name=alejandro shell=/bin/bash" -b
```

---

# Crear Home

```bash
create_home=yes
```

---

# Agregar a grupos

```bash
groups=wheel,docker
```

---

# Eliminar usuario

```bash
ansible all -m user \
-a "name=alejandro state=absent" -b
```

---

# Eliminar Home

```bash
remove=yes
```

---

# Bloquear usuario

```bash
password_lock=yes
```

---

# Desbloquear

```bash
password_lock=no
```

---

# Casos de uso

- Nuevos empleados.
- Cuentas de servicio.
- Automatización.
- Auditorías.

---

# Módulo group

Permite administrar grupos.

---

# Crear grupo

```bash
ansible all -m group \
-a "name=dba state=present" -b
```

---

# Eliminar grupo

```bash
ansible all -m group \
-a "name=dba state=absent" -b
```

---

# Arquitectura

```text
Grupo

↓

Usuarios

↓

Permisos

↓

Recursos
```

---

# Módulo cron

Permite administrar tareas programadas.

---

# Crear tarea

```bash
ansible all -m cron \
-a "name='Backup Diario' minute=0 hour=2 job='/opt/backup.sh'" -b
```

---

# Resultado

```text
0 2 * * * /opt/backup.sh
```

---

# Ejecutar cada hora

```bash
minute=0

hour=*
```

---

# Ejecutar cada 15 minutos

```bash
minute=*/15
```

---

# Eliminar tarea

```bash
state=absent
```

---

# Casos de uso

- Backups.
- Limpieza.
- Monitoreo.
- Reportes.
- Sincronización.

---

# Módulo reboot

Permite reiniciar servidores de forma controlada.

---

# Ejemplo

```bash
ansible all -m reboot -b
```

---

# Tiempo de espera

```bash
ansible all -m reboot \
-a "reboot_timeout=600" -b
```

---

# Mensaje

```bash
msg="Reinicio programado por mantenimiento"
```

---

# Flujo

```text
Servidor

↓

Reinicio

↓

Espera SSH

↓

Servidor disponible

↓

Continuar
```

---

# ¿Por qué utilizar reboot?

Porque Ansible:

- Espera el reinicio.
- Comprueba SSH.
- Continúa automáticamente.

---

# Escenario Empresarial

Una empresa necesita:

- Instalar Apache.
- Crear usuarios.
- Reiniciar servicios.
- Configurar Cron.
- Reiniciar servidores.

Todo puede ejecutarse mediante Comandos Ad-Hoc.

---

# Flujo Empresarial

```text
Administrador

↓

Ansible

↓

DNF

↓

Systemd

↓

User

↓

Cron

↓

500 Servidores
```

---

# Ejemplos Reales

Instalar Git

```bash
ansible all -m dnf \
-a "name=git state=present" -b
```

---

Actualizar OpenSSH

```bash
ansible all -m dnf \
-a "name=openssh state=latest" -b
```

---

Reiniciar SSH

```bash
ansible all -m systemd \
-a "name=sshd state=restarted" -b
```

---

Crear usuario DBA

```bash
ansible database -m user \
-a "name=dba groups=wheel state=present" -b
```

---

Programar respaldo

```bash
ansible backup -m cron \
-a "name='Backup' minute=30 hour=1 job='/opt/backup.sh'" -b
```

---

Reiniciar servidores Web

```bash
ansible web -m reboot -b
```

---

# Buenas Prácticas

- Utilizar `package` cuando se administren distintas distribuciones Linux.
- Utilizar `dnf` en Fedora y RHEL modernos.
- Preferir `systemd` sobre `service` cuando se necesiten funciones avanzadas.
- Ejecutar primero en un grupo pequeño antes de desplegar a toda la infraestructura.
- Utilizar `become` para tareas administrativas.
- Programar reinicios fuera del horario de mayor actividad.
- Verificar el estado del servicio después de instalar un paquete.

---

# Errores Comunes

## Error 1

Olvidar utilizar `-b` (become) para tareas administrativas.

---

## Error 2

Utilizar `yum` en sistemas donde se recomienda `dnf`.

---

## Error 3

Actualizar todos los paquetes sin probar previamente en un entorno de Desarrollo.

---

## Error 4

Reiniciar cientos de servidores simultáneamente sin una estrategia por lotes.

---

## Error 5

Eliminar usuarios sin revisar dependencias o archivos asociados.

---

## Error 6

Crear tareas Cron duplicadas.

---

## Error 7

Reiniciar un servicio sin verificar que la configuración sea válida.

---

# Laboratorio RHCSA

## Laboratorio 1

Instalar Apache (`httpd`) en todos los servidores Web utilizando `dnf`.

---

## Laboratorio 2

Actualizar Git a la versión más reciente.

---

## Laboratorio 3

Instalar `vim`, `wget` y `curl` en todos los servidores.

---

## Laboratorio 4

Iniciar y habilitar el servicio `httpd`.

---

## Laboratorio 5

Reiniciar el servicio `sshd` utilizando el módulo `systemd`.

---

## Laboratorio 6

Crear un usuario llamado `adminlinux` con `/bin/bash` como Shell e incorporarlo al grupo `wheel`.

---

## Laboratorio 7

Crear un grupo llamado `desarrollo`.

---

## Laboratorio 8

Programar una tarea Cron que ejecute un script de limpieza diariamente a las 03:00.

---

## Laboratorio 9

Reiniciar un servidor utilizando el módulo `reboot` y comprobar que vuelve a estar disponible.

---

## Laboratorio 10

Diseñar una estrategia para desplegar una actualización de seguridad en:

- 100 servidores Web.
- 50 Bases de Datos.
- 25 Servidores de Aplicaciones.

Definir:

- Orden de actualización.
- Verificaciones previas.
- Servicios que deben reiniciarse.
- Plan de reversión.
- Validaciones posteriores al mantenimiento.

---

# Resumen

En esta tercera fase estudiamos los módulos de Ansible orientados a la administración del sistema. Aprendimos a instalar y actualizar paquetes mediante `dnf`, `yum` y `package`; administrar servicios con `service` y `systemd`; crear usuarios y grupos con `user` y `group`; programar tareas con `cron`; y realizar reinicios controlados utilizando `reboot`. Estos módulos permiten ejecutar tareas administrativas de forma masiva, consistente e idempotente sobre grandes infraestructuras Linux.

En la **Fase 4** exploraremos los **Facts** mediante el módulo `setup`, la ejecución paralela (`forks`), el uso de `--limit`, los módulos más utilizados en producción, técnicas de troubleshooting, auditoría, escenarios empresariales y un laboratorio integral tipo RHCSA.

----

# 81. Comandos Ad-Hoc en Ansible (Fase 4)

> **Módulo 11 – Automatización con Ansible**
>
> **Archivo:** `81-comandos-ad-hoc.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Comprender qué son los **Facts** de Ansible.
- Utilizar el módulo `setup`.
- Consultar información detallada de los servidores.
- Ejecutar comandos de forma paralela.
- Limitar la ejecución sobre grupos específicos.
- Aplicar buenas prácticas utilizadas en ambientes empresariales.
- Diagnosticar problemas comunes.
- Auditar ejecuciones de comandos Ad-Hoc.
- Realizar un laboratorio integral tipo RHCSA.

---

# Introducción

Hasta este punto hemos aprendido a utilizar los principales módulos de Ansible para:

- Ejecutar comandos.
- Administrar archivos.
- Instalar software.
- Administrar servicios.
- Crear usuarios.
- Programar tareas.

Sin embargo, antes de ejecutar cualquier cambio es recomendable conocer el estado del servidor.

Para ello Ansible utiliza los **Facts**.

---

# ¿Qué son los Facts?

Los **Facts** son información recopilada automáticamente acerca del sistema remoto.

Ejemplos:

- Hostname
- Dirección IP
- CPU
- Memoria
- Interfaces de red
- Sistema Operativo
- Kernel
- Arquitectura
- Disco
- DNS
- Variables del sistema

---

# Arquitectura

```text
              Servidor

                   │

                   ▼

             Módulo setup

                   │

                   ▼

          Recolección de Facts

                   │

                   ▼

              Variables JSON

                   │

                   ▼

             Administrador
```

---

# El módulo setup

El módulo encargado de obtener los Facts es:

```text
setup
```

---

# Sintaxis

```bash
ansible all -m setup
```

---

# Flujo de ejecución

```text
SSH

↓

Python

↓

Setup

↓

Facts

↓

JSON
```

---

# Ejemplo

```bash
ansible web -m setup
```

---

# Resultado

```text
SUCCESS

ansible_hostname

ansible_distribution

ansible_kernel

ansible_memory_mb

ansible_processor

ansible_interfaces
```

---

# Información recopilada

Entre los cientos de Facts disponibles encontramos:

| Fact | Descripción |
|-------|-------------|
| ansible_hostname | Nombre del servidor |
| ansible_fqdn | Nombre completo |
| ansible_distribution | Distribución Linux |
| ansible_distribution_version | Versión |
| ansible_kernel | Kernel |
| ansible_architecture | Arquitectura |
| ansible_processor | Procesador |
| ansible_memory_mb | Memoria RAM |
| ansible_interfaces | Interfaces |
| ansible_default_ipv4 | IP principal |
| ansible_mounts | Sistemas de archivos |
| ansible_devices | Discos |
| ansible_dns | DNS configurados |
| ansible_date_time | Fecha y hora |

---

# Consultar únicamente un Fact

```bash
ansible all -m setup \
-a "filter=ansible_hostname"
```

---

Resultado

```text
web01

db01

backup01
```

---

# Consultar la dirección IP

```bash
ansible all -m setup \
-a "filter=ansible_default_ipv4"
```

---

# Consultar memoria

```bash
ansible all -m setup \
-a "filter=ansible_memory_mb"
```

---

# Consultar Kernel

```bash
ansible all -m setup \
-a "filter=ansible_kernel"
```

---

# Consultar CPU

```bash
ansible all -m setup \
-a "filter=ansible_processor"
```

---

# Consultar Sistema Operativo

```bash
ansible all -m setup \
-a "filter=ansible_distribution"
```

---

# Consultar interfaces

```bash
ansible all -m setup \
-a "filter=ansible_interfaces"
```

---

# Ventajas del filtro

Sin filtro:

```text
Más de 1.000 líneas
```

Con filtro:

```text
Solo la información requerida
```

Reduce el tráfico y facilita la lectura.

---

# Ejecución Paralela

Una de las características más importantes de Ansible es su capacidad para ejecutar tareas simultáneamente.

---

# Arquitectura

```text
              Control Node

                    │

     ┌──────────────┼──────────────┐

     ▼              ▼              ▼

   Host1          Host2         Host3

     ▼              ▼              ▼

  Ejecutan simultáneamente
```

---

# Forks

La cantidad de conexiones simultáneas se controla mediante:

```text
forks
```

---

# Valor por defecto

Generalmente:

```text
5
```

Esto significa que Ansible administrará cinco servidores simultáneamente.

---

# Modificar forks

```bash
ansible all \
-f 20 \
-m ping
```

---

# Interpretación

```text
20 conexiones SSH

↓

20 servidores

↓

Ejecución simultánea
```

---

# ¿Siempre aumentar forks?

No.

Depende de:

- CPU del Control Node.
- RAM disponible.
- Red.
- Cantidad de servidores.
- Velocidad del almacenamiento.

---

# Recomendación

| Infraestructura | Forks sugeridos |
|-----------------|----------------|
| Laboratorio | 5 |
| Pequeña | 10 |
| Mediana | 20 |
| Grande | 30–50 |
| Muy grande | Según pruebas de rendimiento |

---

# Limitar la ejecución

Muchas veces no queremos ejecutar comandos sobre todos los servidores.

---

# Opción --limit

Ejemplo

```bash
ansible all \
--limit web \
-m ping
```

---

# Un único servidor

```bash
ansible all \
--limit web01 \
-m ping
```

---

# Varios Hosts

```bash
ansible all \
--limit "web01,web02"
```

---

# Grupo completo

```bash
ansible all \
--limit database
```

---

# Excluir servidores

```bash
--limit 'all:!database'
```

---

# Flujo

```text
Inventory

↓

Limit

↓

Hosts seleccionados

↓

Ejecución
```

---

# ¿Por qué utilizar --limit?

Evita errores.

Ejemplo:

```text
No reiniciar

500 servidores

↓

Solo

Web01
```

---

# Módulos más utilizados en Producción

| Módulo | Frecuencia |
|----------|------------|
| ping | Muy alta |
| setup | Muy alta |
| command | Muy alta |
| shell | Alta |
| copy | Muy alta |
| template | Muy alta |
| file | Muy alta |
| stat | Alta |
| package | Muy alta |
| dnf | Muy alta |
| service | Muy alta |
| systemd | Muy alta |
| user | Alta |
| group | Alta |
| cron | Alta |
| reboot | Media |

---

# Flujo típico empresarial

```text
Ping

↓

Facts

↓

Instalar

↓

Copiar archivos

↓

Modificar configuración

↓

Reiniciar servicio

↓

Validar
```

---

# Troubleshooting

## Error

```text
UNREACHABLE
```

Posibles causas:

- SSH detenido.
- Firewall.
- DNS.
- Usuario incorrecto.
- Clave SSH.

---

## Error

```text
Permission denied
```

Verificar:

- Usuario.
- Become.
- Permisos.

---

## Error

```text
Python not found
```

Solución

```bash
ansible all -m raw \
-a "dnf install python3 -y"
```

---

## Error

```text
No hosts matched
```

Revisar:

- Inventory.
- Grupo.
- Nombre del Host.

---

## Error

```text
MODULE FAILURE
```

Posibles causas:

- Python dañado.
- Dependencias.
- Disco lleno.
- Permisos.

---

# Flujo de Diagnóstico

```text
Problema

↓

SSH

↓

Ping

↓

Setup

↓

Command

↓

Logs

↓

Solución
```

---

# Auditoría

Antes de ejecutar un cambio importante es recomendable responder las siguientes preguntas:

- ¿Qué servidores serán afectados?
- ¿Qué usuario ejecutará la tarea?
- ¿Existe respaldo?
- ¿Se probó previamente?
- ¿Existe ventana de mantenimiento?

---

# Registro de actividades

Ejemplo de documentación:

| Fecha | Acción | Responsable |
|--------|---------|-------------|
| 02/08 | Actualización Apache | DBA |
| 03/08 | Reinicio SSH | Linux Team |
| 04/08 | Instalación Git | DevOps |

---

# Caso Empresarial 1

Actualizar Git en:

- 300 servidores Web.

Procedimiento:

```text
Ping

↓

Setup

↓

Actualizar Git

↓

Validar versión

↓

Documentar
```

---

# Caso Empresarial 2

Modificar configuración SSH.

```text
Backup

↓

Copy

↓

Lineinfile

↓

Restart SSH

↓

Validación
```

---

# Caso Empresarial 3

Crear un nuevo usuario corporativo.

```text
Group

↓

User

↓

SSH

↓

Auditoría
```

---

# Caso Empresarial 4

Reinicio programado.

```text
Aviso

↓

Backup

↓

Reboot

↓

Verificación

↓

Servicios
```

---

# Buenas Prácticas

- Ejecutar siempre `ping` antes de modificar servidores.
- Consultar Facts antes de instalar software.
- Utilizar `--limit` para cambios críticos.
- Probar primero en Desarrollo.
- Utilizar grupos bien definidos.
- Documentar todas las ejecuciones importantes.
- Utilizar `package` cuando existan varias distribuciones Linux.
- Mantener un Inventory actualizado.
- Validar el resultado de cada ejecución.
- Evitar ejecutar tareas masivas durante horas pico.

---

# Errores Comunes

## Error 1

Ejecutar comandos sobre `all` sin verificar el alcance.

---

## Error 2

No utilizar `--limit`.

---

## Error 3

No comprobar conectividad.

---

## Error 4

No revisar los Facts antes de automatizar.

---

## Error 5

Incrementar demasiado los `forks`.

---

## Error 6

No validar los resultados obtenidos.

---

## Error 7

No documentar los cambios.

---

## Error 8

Ejecutar reinicios simultáneos sobre todos los servidores.

---

## Error 9

No probar en un ambiente de Desarrollo.

---

## Error 10

Confiar únicamente en la salida del comando sin verificar el estado final del servicio o la aplicación.

---

# Laboratorio Integral RHCSA

## Escenario

Una empresa posee:

```text
120 Web Servers

45 PostgreSQL

25 SQL Server

20 DNS

30 Aplicaciones

15 Monitoreo
```

El objetivo es realizar una actualización controlada utilizando únicamente Comandos Ad-Hoc.

---

# Actividad 1

Verificar conectividad utilizando:

```bash
ansible all -m ping
```

---

# Actividad 2

Consultar:

- Sistema operativo.
- Kernel.
- Memoria.
- CPU.
- Dirección IP.

Utilizando el módulo `setup`.

---

# Actividad 3

Actualizar Git únicamente en los servidores Web.

---

# Actividad 4

Copiar un archivo de configuración utilizando `copy`.

---

# Actividad 5

Modificar `/etc/ssh/sshd_config` utilizando `lineinfile`.

---

# Actividad 6

Reiniciar el servicio SSH mediante `systemd`.

---

# Actividad 7

Crear un usuario corporativo.

---

# Actividad 8

Programar una tarea Cron para ejecutar un respaldo diario.

---

# Actividad 9

Reiniciar únicamente dos servidores de prueba utilizando:

```text
--limit
```

---

# Actividad 10

Documentar:

- Cambios realizados.
- Servidores afectados.
- Tiempo de ejecución.
- Riesgos identificados.
- Plan de reversión.
- Evidencias de validación.

---

# Preguntas de Repaso

1. ¿Qué son los Facts en Ansible?
2. ¿Qué módulo recopila los Facts?
3. ¿Qué información proporciona `ansible_default_ipv4`?
4. ¿Qué ventaja tiene utilizar `filter=` con el módulo `setup`?
5. ¿Qué es un `fork` en Ansible?
6. ¿Cuál es el valor predeterminado de `forks` en la mayoría de las instalaciones?
7. ¿Cuándo conviene aumentar el número de `forks`?
8. ¿Qué hace la opción `--limit`?
9. ¿Por qué es recomendable utilizar `--limit` durante tareas críticas?
10. ¿Cuál es el flujo recomendado antes de modificar servidores en producción?
11. ¿Qué acciones tomarías si un Host aparece como `UNREACHABLE`?
12. ¿Qué módulo utilizarías para instalar Python en un servidor que aún no lo tiene?
13. ¿Por qué es importante documentar las ejecuciones de Comandos Ad-Hoc?
14. ¿Qué diferencia existe entre recopilar Facts y ejecutar un comando con `command`?
15. ¿Qué buenas prácticas ayudan a reducir el riesgo durante automatizaciones masivas?

---

# Resumen del Capítulo

En este capítulo estudiamos los **Comandos Ad-Hoc**, una herramienta esencial para ejecutar tareas rápidas y puntuales sin necesidad de crear Playbooks. Aprendimos la sintaxis general de Ansible y el uso de módulos fundamentales como `ping`, `command`, `shell` y `raw` para la ejecución de comandos; `copy`, `fetch`, `file`, `stat`, `template`, `lineinfile`, `replace` y `blockinfile` para la administración de archivos; y `dnf`, `package`, `systemd`, `service`, `user`, `group`, `cron` y `reboot` para la administración del sistema.

Finalmente, exploramos el módulo `setup` para recopilar **Facts**, el uso de `forks` para ejecutar tareas en paralelo, la opción `--limit` para restringir el alcance de las operaciones, técnicas de troubleshooting, auditoría y un laboratorio integral orientado a escenarios reales de administración Linux. Con estos conocimientos ya es posible administrar infraestructuras completas de forma eficiente y segura utilizando Comandos Ad-Hoc.

---

# Próximo Capítulo

## **82. Introducción a los Playbooks**

En el siguiente capítulo aprenderemos a crear **Playbooks**, el mecanismo principal de automatización de Ansible. Estudiaremos la estructura de un Playbook, la sintaxis YAML, las tareas (`tasks`), los módulos, la ejecución de Playbooks, la idempotencia, el uso de variables y las mejores prácticas para construir automatizaciones reutilizables y mantenibles.





