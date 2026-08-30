# 83. Variables y Facts en Ansible (Fase 1)

> **Módulo 11 – Automatización con Ansible**
>
> **Archivo:** `83-variables-facts.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Comprender qué son las Variables en Ansible.
- Entender por qué las Variables son fundamentales para la automatización.
- Conocer los diferentes tipos de Variables.
- Aprender el ciclo de vida de una Variable.
- Comprender qué son los Facts.
- Diferenciar Variables definidas por el usuario de Variables automáticas.
- Utilizar correctamente Variables y Facts dentro de un Playbook.
- Aplicar buenas prácticas empresariales para el manejo de Variables.

---

# Introducción

Hasta este momento hemos utilizado Variables de forma sencilla.

Por ejemplo:

```yaml
vars:

  paquete: httpd
```

Posteriormente utilizábamos esa Variable.

```yaml
dnf:

  name: "{{ paquete }}"
```

Aunque este ejemplo es simple, las Variables constituyen uno de los componentes más importantes de Ansible.

Prácticamente todas las automatizaciones profesionales dependen de ellas.

Sin Variables sería necesario escribir un Playbook diferente para cada servidor.

Con Variables podemos utilizar un único Playbook para administrar cientos o miles de equipos.

---

# ¿Qué es una Variable?

Una Variable es un contenedor que almacena información para ser utilizada posteriormente durante la ejecución de un Playbook.

Una Variable puede contener:

- Texto
- Números
- Valores booleanos
- Listas
- Diccionarios
- Objetos complejos
- Información recopilada automáticamente por Ansible

---

# Analogía

Imaginemos una empresa que administra 300 servidores.

Sin Variables:

```text
Servidor 1

Apache

Servidor 2

Nginx

Servidor 3

Tomcat

Servidor 4

Apache

...

Servidor 300
```

Sería necesario modificar el Playbook constantemente.

---

Con Variables:

```text
Playbook

↓

Variable

↓

Valor diferente

↓

Servidor correspondiente
```

El mismo Playbook puede adaptarse automáticamente.

---

# Automatización Tradicional

```text
Script

↓

Valores escritos manualmente

↓

Difícil mantenimiento
```

---

# Automatización con Variables

```text
Playbook

↓

Variables

↓

Información dinámica

↓

Infraestructura
```

---

# Beneficios

Las Variables permiten:

- Reutilizar código.
- Reducir errores.
- Centralizar configuraciones.
- Evitar duplicación.
- Facilitar mantenimiento.
- Adaptar Playbooks.
- Simplificar cambios.
- Mejorar la legibilidad.
- Escalar automatizaciones.
- Facilitar auditorías.

---

# Arquitectura General

```text
                  Variables

                       │

        ┌──────────────┼──────────────┐

        ▼              ▼              ▼

   Playbook       Inventario      Facts

        │              │              │

        └──────────────┼──────────────┘

                       ▼

                    Jinja2

                       ▼

                    Módulos

                       ▼

                  Servidor Linux
```

---

# Tipos de Variables

Ansible soporta múltiples fuentes de Variables.

Las principales son:

- Variables del Playbook
- Variables del Inventario
- Variables de Host
- Variables de Grupo
- Variables Registradas
- Variables Extra
- Facts
- Variables Mágicas

En esta primera fase estudiaremos las más importantes.

---

# Variables Definidas por el Usuario

Son creadas manualmente.

Ejemplo:

```yaml
vars:

  paquete: httpd
```

---

Otro ejemplo:

```yaml
vars:

  usuario: administrador

  shell: /bin/bash
```

---

# Variables Numéricas

```yaml
vars:

  puerto: 8080

  memoria: 4096
```

---

# Variables Booleanas

```yaml
vars:

  habilitado: true

  respaldo: false
```

---

# Variables tipo Cadena

```yaml
vars:

  servidor: web01

  dominio: empresa.local
```

---

# Variables tipo Lista

```yaml
vars:

  paquetes:

    - git

    - wget

    - curl

    - vim
```

---

# Representación

```text
Lista

↓

git

wget

curl

vim
```

---

# Variables tipo Diccionario

```yaml
vars:

  usuario:

    nombre: administrador

    shell: /bin/bash

    uid: 2000
```

---

Representación

```text
usuario

│

├── nombre

├── shell

└── uid
```

---

# Acceder a un Diccionario

```yaml
{{ usuario.nombre }}
```

---

```yaml
{{ usuario.shell }}
```

---

```yaml
{{ usuario.uid }}
```

---

# Variables Complejas

También pueden combinarse listas y diccionarios.

Ejemplo:

```yaml
vars:

  servidores:

    - nombre: web01

      ip: 192.168.1.10

    - nombre: web02

      ip: 192.168.1.11
```

---

Representación

```text
Servidores

│

├── web01

│      │

│      └── 192.168.1.10

│

└── web02

       │

       └── 192.168.1.11
```

---

# ¿Dónde se utilizan?

Las Variables pueden utilizarse prácticamente en cualquier módulo.

Ejemplo:

```yaml
dnf:

  name: "{{ paquete }}"
```

---

```yaml
service:

  name: "{{ servicio }}"
```

---

```yaml
user:

  name: "{{ usuario }}"
```

---

```yaml
file:

  path: "{{ directorio }}"
```

---

# Motor Jinja2

Las Variables utilizan el motor de plantillas llamado **Jinja2**.

Siempre deben escribirse utilizando:

```text
{{ variable }}
```

---

Ejemplo

```yaml
{{ paquete }}
```

---

No utilizar:

```yaml
$paquete
```

---

Ni:

```yaml
%paquete
```

---

# Flujo de Resolución

```text
Variable

↓

Jinja2

↓

Valor

↓

Módulo

↓

Host
```

---

# Variables en un Playbook

```yaml
---
- name: Instalar Apache

  hosts: web

  vars:

    paquete: httpd

  tasks:

    - name: Instalar

      dnf:

        name: "{{ paquete }}"

        state: present
```

---

# Resultado

```text
Variable

↓

httpd

↓

dnf

↓

Servidor
```

---

# Variables en varias Tasks

```yaml
vars:

  servicio: httpd
```

Posteriormente.

```yaml
service:

  name: "{{ servicio }}"
```

---

Y nuevamente.

```yaml
debug:

  msg: "{{ servicio }}"
```

---

La misma Variable puede utilizarse cientos de veces.

---

# ¿Qué son los Facts?

Los Facts son Variables creadas automáticamente por Ansible.

No necesitamos definirlas.

Ansible las obtiene del sistema remoto.

---

# Flujo

```text
Servidor

↓

Módulo setup

↓

Facts

↓

Variables automáticas
```

---

# Ejemplos

```text
Hostname

Sistema Operativo

Kernel

CPU

RAM

Interfaces

IP

DNS

Arquitectura

Discos
```

---

# Obtención de Facts

Durante la ejecución de un Playbook, Ansible recopila automáticamente los Facts (salvo que se desactive esta funcionalidad).

También pueden consultarse manualmente mediante:

```bash
ansible all -m setup
```

---

# Arquitectura

```text
Host

↓

SSH

↓

Python

↓

Setup

↓

Facts

↓

Playbook
```

---

# Ejemplos de Facts

| Fact | Descripción |
|-------|-------------|
| ansible_hostname | Nombre del Host |
| ansible_distribution | Distribución Linux |
| ansible_kernel | Kernel |
| ansible_architecture | Arquitectura |
| ansible_default_ipv4 | Dirección IP principal |
| ansible_processor | Procesador |
| ansible_memory_mb | Memoria RAM |
| ansible_mounts | Sistemas de archivos |
| ansible_interfaces | Interfaces de red |
| ansible_dns | Configuración DNS |

---

# Mostrar un Fact

```yaml
- name: Mostrar Hostname

  debug:

    msg: "{{ ansible_hostname }}"
```

---

Mostrar Kernel

```yaml
debug:

  msg: "{{ ansible_kernel }}"
```

---

Mostrar Distribución

```yaml
debug:

  msg: "{{ ansible_distribution }}"
```

---

Mostrar Arquitectura

```yaml
debug:

  msg: "{{ ansible_architecture }}"
```

---

Mostrar Dirección IP

```yaml
debug:

  msg: "{{ ansible_default_ipv4.address }}"
```

---

# Variables del Usuario vs Facts

| Variables | Facts |
|------------|-------|
| Las crea el administrador | Las crea Ansible |
| Pueden modificarse | Son información del sistema |
| Se escriben manualmente | Se recopilan automáticamente |
| Definen configuraciones | Describen el servidor |
| Son reutilizables | Cambian según cada host |

---

# ¿Por qué son importantes los Facts?

Permiten crear automatizaciones inteligentes.

Ejemplo.

```text
Si el sistema es Fedora

↓

Instalar usando dnf
```

---

```text
Si tiene menos de 4 GB RAM

↓

No instalar determinada aplicación
```

---

```text
Si el Kernel es antiguo

↓

Mostrar advertencia
```

---

# Comparación

Sin Facts

```text
Administrador

↓

Decisión manual
```

---

Con Facts

```text
Servidor

↓

Facts

↓

Playbook

↓

Decisión automática
```

---

# Buenas Prácticas

- Utilizar Variables descriptivas.
- Evitar nombres ambiguos.
- Centralizar configuraciones comunes.
- Reutilizar Variables siempre que sea posible.
- Evitar valores escritos directamente dentro de las Tasks.
- Aprovechar los Facts antes de ejecutar cambios.
- Utilizar `debug` únicamente durante el desarrollo.
- Mantener nombres consistentes en todo el proyecto.
- Documentar Variables importantes.
- Evitar duplicar información.

---

# Errores Comunes

## Error 1

Escribir Variables sin `{{ }}`.

---

## Error 2

Utilizar nombres poco descriptivos.

Ejemplo.

```text
dato1

valor2

config3
```

---

## Error 3

Duplicar Variables.

---

## Error 4

Confundir Facts con Variables definidas por el usuario.

---

## Error 5

Modificar manualmente información que ya existe como Fact.

---

## Error 6

No utilizar Variables reutilizables.

---

## Error 7

Crear Playbooks con valores fijos.

---

## Error 8

No aprovechar Jinja2.

---

## Error 9

Utilizar nombres inconsistentes.

---

## Error 10

No documentar Variables críticas.

---

# Laboratorio RHCSA

## Escenario

Una empresa administra:

- 120 servidores Web
- 40 servidores PostgreSQL
- 20 servidores SQL Server

Todos deben administrarse utilizando un único Playbook.

---

## Laboratorio 1

Crear Variables para:

- Nombre del paquete.
- Servicio.
- Usuario.
- Directorio.

---

## Laboratorio 2

Utilizar las Variables dentro de los módulos:

- dnf
- service
- file
- user

---

## Laboratorio 3

Crear una lista de paquetes e instalarlos posteriormente mediante un `loop`.

---

## Laboratorio 4

Crear un diccionario con la información de un usuario y acceder a cada uno de sus atributos.

---

## Laboratorio 5

Mostrar mediante `debug`:

- Hostname.
- Kernel.
- Distribución.
- Arquitectura.

---

## Laboratorio 6

Mostrar la dirección IP principal utilizando `ansible_default_ipv4.address`.

---

## Laboratorio 7

Comparar Variables definidas por el usuario con Facts recopilados automáticamente e identificar las diferencias.

---

## Laboratorio 8

Modificar únicamente el valor de una Variable y comprobar cómo el mismo Playbook cambia su comportamiento sin necesidad de editar las Tasks.

---

## Preguntas de Repaso

1. ¿Qué es una Variable en Ansible?
2. ¿Qué ventajas ofrecen las Variables frente a valores fijos?
3. ¿Qué tipos de datos puede almacenar una Variable?
4. ¿Qué función cumple Jinja2?
5. ¿Cómo se referencia una Variable?
6. ¿Qué son los Facts?
7. ¿Qué módulo recopila los Facts?
8. ¿Cuál es la diferencia entre una Variable y un Fact?
9. ¿Por qué los Facts permiten automatizaciones más inteligentes?
10. ¿Qué buenas prácticas deben seguirse al nombrar Variables?

---

# Resumen

En esta primera fase aprendimos que las **Variables** representan el mecanismo principal para crear Playbooks reutilizables y escalables. Estudiamos los distintos tipos de Variables, el uso de **Jinja2**, la forma correcta de referenciarlas y las ventajas que ofrecen frente al uso de valores fijos.

También conocimos los **Facts**, Variables generadas automáticamente por Ansible que describen el estado del sistema remoto. Gracias a los Facts es posible crear automatizaciones capaces de adaptarse a cada servidor sin modificar el Playbook.

En la **Fase 2** profundizaremos en las distintas fuentes de Variables, incluyendo **`vars`**, **`vars_files`**, **`host_vars`**, **`group_vars`**, Variables del Inventario, Variables Extra (`--extra-vars`) y la precedencia de Variables, uno de los conceptos más importantes para desarrollar automatizaciones profesionales con Ansible.
----


# 83. Variables y Facts en Ansible (Fase 2)

> **Módulo 11 – Automatización con Ansible**
>
> **Archivo:** `83-variables-facts.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Comprender las diferentes fuentes de Variables.
- Utilizar correctamente `vars`, `vars_files`, `host_vars` y `group_vars`.
- Definir Variables dentro del Inventario.
- Utilizar Variables externas (`--extra-vars`).
- Comprender la precedencia de Variables.
- Organizar proyectos empresariales utilizando Variables.
- Aplicar buenas prácticas para administrar cientos de servidores.

---

# Introducción

En la fase anterior aprendimos a crear Variables directamente dentro del Playbook.

Ejemplo:

```yaml
vars:

  paquete: httpd
```

Este método funciona correctamente para pequeños laboratorios.

Sin embargo, una empresa puede administrar:

- 50 servidores Web
- 80 servidores PostgreSQL
- 40 servidores SQL Server
- 20 servidores DNS
- 30 servidores Proxy

Cada grupo tendrá configuraciones diferentes.

Mantener todas esas Variables dentro de un único Playbook sería muy difícil.

Por esta razón Ansible permite almacenar Variables en diferentes ubicaciones.

---

# Arquitectura General

```text
                 Variables

                      │

      ┌───────────────┼────────────────┐

      ▼               ▼                ▼

   Playbook       Inventario      Archivos

      │               │                │

      ▼               ▼                ▼

 host_vars      group_vars      vars_files

      │

      ▼

   Extra Vars

      │

      ▼

    Playbook
```

---

# Variables Locales (vars)

Son las Variables declaradas directamente dentro del Playbook.

```yaml
---
- name: Variables

  hosts: web

  vars:

    paquete: httpd

    puerto: 80
```

---

## Ventajas

- Muy simples.
- Fáciles de leer.
- Ideales para laboratorios.
- Útiles para pequeños Playbooks.

---

## Desventajas

- Difíciles de reutilizar.
- Poco escalables.
- Mezclan datos con lógica.

---

# Variables Externas (vars_files)

En proyectos grandes es recomendable separar la información.

Ejemplo.

```yaml
vars_files:

  - variables.yml
```

---

Archivo:

```text
variables.yml
```

Contenido.

```yaml
---

paquete: httpd

servicio: httpd

puerto: 80
```

---

Playbook.

```yaml
---
- name: Apache

  hosts: web

  vars_files:

    - variables.yml

  tasks:

    - name: Instalar

      dnf:

        name: "{{ paquete }}"

        state: present
```

---

# Arquitectura

```text
Playbook

↓

variables.yml

↓

Variables

↓

Tasks
```

---

# Beneficios

- Código más limpio.
- Variables centralizadas.
- Fácil mantenimiento.
- Reutilización.

---

# Múltiples Archivos

```yaml
vars_files:

  - apache.yml

  - usuarios.yml

  - backup.yml
```

---

# Organización Recomendada

```text
ansible/

├── playbooks/

├── vars/

│      ├── apache.yml

│      ├── usuarios.yml

│      ├── backup.yml

│      └── firewall.yml
```

---

# Variables del Inventario

También pueden definirse directamente en el Inventario.

Ejemplo.

```ini
[web]

web01 puerto=80

web02 puerto=8080
```

---

Otro ejemplo.

```ini
[database]

db01 puerto=5432

db02 puerto=5432
```

---

Playbook.

```yaml
debug:

  msg: "{{ puerto }}"
```

---

Resultado.

```text
web01

80
```

---

```text
web02

8080
```

---

# Arquitectura

```text
Inventario

↓

Variables

↓

Host

↓

Playbook
```

---

# Variables de Grupo (group_vars)

Una de las mejores características de Ansible.

Permite asignar Variables a todo un grupo.

---

Ejemplo.

```text
group_vars/

└── web.yml
```

---

Contenido.

```yaml
---

paquete: httpd

servicio: httpd

puerto: 80
```

---

Playbook.

```yaml
dnf:

  name: "{{ paquete }}"
```

---

Todos los Hosts del grupo **web** utilizarán automáticamente esas Variables.

---

# Arquitectura

```text
Grupo Web

│

├── web01

├── web02

├── web03

└── web04

       │

       ▼

 group_vars/web.yml
```

---

# Beneficios

- Configuración centralizada.
- Excelente mantenimiento.
- Muy utilizada en Producción.
- Fácil reutilización.

---

# Variables por Host (host_vars)

En ocasiones un servidor necesita una configuración distinta.

Ejemplo.

```text
host_vars/

└── web01.yml
```

---

Contenido.

```yaml
---

puerto: 8080

servidor: principal
```

---

Mientras que otro Host.

```text
host_vars/

└── web02.yml
```

---

```yaml
---

puerto: 9090

servidor: respaldo
```

---

Arquitectura.

```text
Host

↓

host_vars

↓

Variables

↓

Playbook
```

---

# Diferencia

```text
group_vars

↓

Todos los servidores
```

---

```text
host_vars

↓

Solo un servidor
```

---

# Organización Empresarial

```text
inventories/

├── production/

│      ├── hosts

│      ├── group_vars/

│      │      ├── web.yml

│      │      ├── database.yml

│      │      └── backup.yml

│      │

│      └── host_vars/

│             ├── web01.yml

│             ├── web02.yml

│             └── db01.yml
```

---

# Variables Extra

Las Variables también pueden pasarse desde la línea de comandos.

---

Sintaxis.

```bash
ansible-playbook sitio.yml \
-e "paquete=httpd"
```

---

Otro ejemplo.

```bash
ansible-playbook sitio.yml \
-e "usuario=dba"
```

---

Múltiples Variables.

```bash
ansible-playbook sitio.yml \
-e "usuario=dba shell=/bin/bash puerto=8080"
```

---

Archivo JSON.

```bash
ansible-playbook sitio.yml \
-e @variables.json
```

---

Archivo YAML.

```bash
ansible-playbook sitio.yml \
-e @variables.yml
```

---

# ¿Cuándo utilizar Extra Vars?

Muy útiles para:

- CI/CD.
- Jenkins.
- GitLab CI.
- Azure DevOps.
- GitHub Actions.
- Automatizaciones temporales.

---

Ejemplo.

Producción.

```bash
ansible-playbook deploy.yml \
-e "version=3.2"
```

---

Desarrollo.

```bash
ansible-playbook deploy.yml \
-e "version=4.0-beta"
```

---

El mismo Playbook.

Diferente comportamiento.

---

# Variables Registradas

Ya estudiamos `register`.

Ejemplo.

```yaml
- name: Obtener Kernel

  command: uname -r

  register: kernel
```

Posteriormente.

```yaml
debug:

  msg: "{{ kernel.stdout }}"
```

Estas Variables existen únicamente durante la ejecución del Playbook.

---

# Variables Temporales

```text
Task

↓

Register

↓

Variable

↓

Fin del Playbook

↓

Desaparece
```

---

# Variables Persistentes

```text
group_vars

↓

host_vars

↓

vars_files

↓

Inventario
```

Permanecen en el proyecto.

---

# Precedencia de Variables

Una misma Variable puede definirse en varios lugares.

¿Cuál utilizará Ansible?

La respuesta depende de la **precedencia**.

---

# Ejemplo

Tenemos:

```text
vars

↓

host_vars

↓

group_vars

↓

extra-vars
```

Todas contienen:

```text
puerto
```

¿Cuál gana?

---

# Orden Simplificado de Precedencia

De menor prioridad a mayor prioridad.

| Prioridad | Fuente |
|------------|---------|
| Baja | Variables del Inventario |
| | group_vars |
| | host_vars |
| | vars_files |
| | vars |
| Alta | Extra Vars (`-e`) |

---

# Ejemplo

group_vars.

```yaml
puerto: 80
```

---

host_vars.

```yaml
puerto: 8080
```

---

Resultado.

```text
8080
```

Porque **host_vars** tiene mayor prioridad.

---

Ahora ejecutamos.

```bash
ansible-playbook sitio.yml \
-e "puerto=9090"
```

Resultado.

```text
9090
```

Porque **Extra Vars** tienen la prioridad más alta.

---

# Flujo de Resolución

```text
Variable

↓

Buscar

↓

¿Existe Extra Vars?

↓

Sí

↓

Utilizar

↓

No

↓

host_vars

↓

group_vars

↓

Inventario
```

---

# Organización Recomendada

```text
Proyecto

├── inventories

├── playbooks

├── group_vars

├── host_vars

├── vars

├── templates

├── files

└── roles
```

---

# Buenas Prácticas

- Evitar valores fijos dentro de las Tasks.
- Utilizar `group_vars` para configuraciones comunes.
- Utilizar `host_vars` únicamente para excepciones.
- Separar Variables en archivos independientes.
- Nombrar Variables claramente.
- Evitar duplicación.
- Mantener Variables bajo control de versiones.
- Utilizar `extra-vars` para despliegues temporales.
- Comprender la precedencia antes de redefinir Variables.
- Documentar el propósito de cada archivo de Variables.

---

# Errores Comunes

## Error 1

Definir la misma Variable en cinco lugares distintos.

---

## Error 2

No comprender la precedencia.

---

## Error 3

Guardar información sensible junto con Variables normales.

(Posteriormente estudiaremos **Ansible Vault** para este propósito).

---

## Error 4

Utilizar `host_vars` para configuraciones comunes.

---

## Error 5

Crear archivos `vars_files` demasiado grandes.

---

## Error 6

No separar Desarrollo y Producción.

---

## Error 7

Modificar Variables directamente en Producción sin control de versiones.

---

## Error 8

No utilizar `group_vars` para grupos grandes.

---

## Error 9

Abusar de `extra-vars` para configuraciones permanentes.

---

## Error 10

No documentar el origen de las Variables.

---

# Laboratorio RHCSA

## Escenario

Una empresa administra:

- 100 servidores Web.
- 50 servidores PostgreSQL.
- 20 servidores SQL Server.

Cada grupo utiliza configuraciones distintas.

---

## Laboratorio 1

Crear un archivo:

```text
group_vars/web.yml
```

con:

- paquete
- servicio
- puerto

---

## Laboratorio 2

Crear:

```text
host_vars/web01.yml
```

cambiando únicamente el puerto.

---

## Laboratorio 3

Crear un archivo:

```text
vars/apache.yml
```

y cargarlo mediante `vars_files`.

---

## Laboratorio 4

Agregar Variables al Inventario para un servidor de pruebas.

---

## Laboratorio 5

Ejecutar el mismo Playbook utilizando:

```bash
-e "puerto=9090"
```

y comprobar cómo cambia el resultado.

---

## Laboratorio 6

Crear un Playbook que muestre el valor de:

```text
puerto
```

e identificar de dónde proviene la Variable en cada escenario.

---

## Laboratorio 7

Organizar un proyecto siguiendo la estructura recomendada para Producción.

---

## Laboratorio 8

Definir la misma Variable en `group_vars`, `host_vars` y mediante `-e`. Ejecutar el Playbook y verificar cuál valor prevalece. Explicar el resultado utilizando las reglas de precedencia.

---

# Preguntas de Repaso

1. ¿Qué ventajas ofrecen `vars_files` frente a declarar Variables dentro del Playbook?
2. ¿Cuándo utilizar `group_vars`?
3. ¿Cuándo utilizar `host_vars`?
4. ¿Qué diferencias existen entre Variables temporales y persistentes?
5. ¿Qué son las Variables Extra?
6. ¿Qué ventajas ofrecen en un pipeline de CI/CD?
7. ¿Qué ocurre si una Variable existe tanto en `group_vars` como en `host_vars`?
8. ¿Cuál es la fuente de Variables con mayor prioridad en el orden simplificado de precedencia?
9. ¿Por qué es recomendable separar los datos de la lógica del Playbook?
10. ¿Cómo organizarías un proyecto Ansible para administrar cientos de servidores?

---

# Resumen

En esta segunda fase estudiamos las principales **fuentes de Variables** en Ansible. Aprendimos a utilizar `vars`, `vars_files`, Variables del Inventario, `group_vars`, `host_vars` y **Extra Vars**, comprendiendo cuándo utilizar cada una según el tamaño y la complejidad del proyecto.

También analizamos el concepto de **precedencia de Variables**, fundamental para entender cómo Ansible decide qué valor utilizar cuando una misma Variable está definida en varios lugares. Finalmente, revisamos una estructura de proyecto orientada a entornos empresariales y las mejores prácticas para mantener Playbooks organizados, reutilizables y fáciles de administrar.

En la **Fase 3** profundizaremos en los **Facts**, incluyendo `gather_facts`, `setup`, filtros de Facts, Variables mágicas, Variables especiales, Facts personalizados y el uso avanzado de Jinja2 para construir automatizaciones inteligentes.

----

# 83. Variables y Facts en Ansible (Fase 3)

> **Módulo 11 – Automatización con Ansible**
>
> **Archivo:** `83-variables-facts.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Comprender cómo funciona `gather_facts`.
- Utilizar el módulo `setup`.
- Consultar Facts específicos.
- Comprender las Variables Mágicas (Magic Variables).
- Utilizar Variables Especiales.
- Crear automatizaciones inteligentes utilizando Facts.
- Utilizar filtros de Jinja2 para manipular Variables.
- Aplicar buenas prácticas en ambientes empresariales.

---

# Introducción

Hasta ahora hemos aprendido:

- Variables definidas por el usuario.
- group_vars
- host_vars
- vars_files
- Variables del Inventario.
- Extra Vars.

Ahora estudiaremos el segundo gran componente de la automatización inteligente:

# Los Facts

Los Facts permiten que un Playbook adapte automáticamente su comportamiento según el servidor donde se ejecuta.

En otras palabras:

```text
El servidor describe su estado

↓

Ansible recopila esa información

↓

El Playbook toma decisiones
```

---

# Arquitectura

```text
Servidor Linux

      │

      ▼

 Módulo setup

      │

      ▼

  Recolección

      │

      ▼

 Facts (JSON)

      │

      ▼

 Variables

      │

      ▼

 Playbook
```

---

# ¿Qué es gather_facts?

Cuando un Playbook inicia, Ansible ejecuta automáticamente el módulo:

```text
setup
```

El resultado se almacena en cientos de Variables conocidas como Facts.

Esta operación se controla mediante:

```yaml
gather_facts:
```

---

# Valor por defecto

```yaml
gather_facts: true
```

---

Esto significa:

```text
Inicio Playbook

↓

SSH

↓

Setup

↓

Facts

↓

Tasks
```

---

# Desactivar Facts

Cuando un Playbook no necesita información del sistema.

```yaml
---
- hosts: all

  gather_facts: false
```

---

# ¿Por qué desactivarlos?

Porque recopilar Facts consume tiempo.

En grandes infraestructuras:

```text
500 Hosts

↓

Setup

↓

Miles de Variables

↓

Mayor tiempo
```

---

Si únicamente ejecutamos:

```yaml
command: uptime
```

No necesitamos recopilar información del sistema.

---

# Flujo

Con Facts.

```text
Playbook

↓

Setup

↓

Facts

↓

Tasks
```

---

Sin Facts.

```text
Playbook

↓

Tasks
```

---

# El módulo setup

El módulo encargado de recopilar Facts.

Ejecución manual.

```bash
ansible all -m setup
```

---

Resultado.

```text
SUCCESS

ansible_hostname

ansible_kernel

ansible_distribution

ansible_interfaces

ansible_processor

...
```

Generalmente devuelve cientos de Variables.

---

# Consultar un único Fact

En lugar de mostrar todos.

```bash
ansible all -m setup \
-a "filter=ansible_hostname"
```

---

Mostrar Kernel.

```bash
ansible all -m setup \
-a "filter=ansible_kernel"
```

---

Mostrar Memoria.

```bash
ansible all -m setup \
-a "filter=ansible_memory_mb"
```

---

Mostrar Interfaces.

```bash
ansible all -m setup \
-a "filter=ansible_interfaces"
```

---

Mostrar Dirección IP.

```bash
ansible all -m setup \
-a "filter=ansible_default_ipv4"
```

---

# Ventajas del filtro

Sin filtro.

```text
Más de 1.000 líneas
```

Con filtro.

```text
Solo la información necesaria
```

Reduce:

- Tiempo de lectura.
- Tráfico.
- Complejidad.

---

# Facts más utilizados

| Fact | Descripción |
|--------|-------------|
| ansible_hostname | Nombre del servidor |
| ansible_fqdn | Nombre completo |
| ansible_distribution | Distribución |
| ansible_distribution_version | Versión |
| ansible_kernel | Kernel |
| ansible_architecture | Arquitectura |
| ansible_processor | CPU |
| ansible_memory_mb | RAM |
| ansible_interfaces | Interfaces |
| ansible_default_ipv4.address | Dirección IP |
| ansible_mounts | Sistemas de archivos |
| ansible_devices | Discos |
| ansible_dns | DNS |
| ansible_env | Variables de entorno |
| ansible_date_time | Fecha y hora |

---

# Mostrar Facts

```yaml
- name: Mostrar Hostname

  debug:

    msg: "{{ ansible_hostname }}"
```

---

Kernel.

```yaml
debug:

  msg: "{{ ansible_kernel }}"
```

---

Sistema Operativo.

```yaml
debug:

  msg: "{{ ansible_distribution }}"
```

---

Arquitectura.

```yaml
debug:

  msg: "{{ ansible_architecture }}"
```

---

Memoria.

```yaml
debug:

  msg: "{{ ansible_memory_mb.real.total }}"
```

---

Dirección IP.

```yaml
debug:

  msg: "{{ ansible_default_ipv4.address }}"
```

---

# Variables Mágicas (Magic Variables)

Ansible incorpora Variables especiales creadas automáticamente.

No provienen del módulo setup.

Son generadas por el propio motor de Ansible.

---

# Arquitectura

```text
Ansible Engine

       │

       ▼

Magic Variables

       │

       ▼

Playbook
```

---

# Variables Mágicas más utilizadas

| Variable | Descripción |
|-----------|-------------|
| inventory_hostname | Nombre del Host en el Inventario |
| inventory_hostname_short | Nombre corto |
| groups | Lista de grupos |
| group_names | Grupos del Host |
| hostvars | Variables de todos los Hosts |
| play_hosts | Hosts del Play |
| ansible_play_hosts | Hosts activos |
| inventory_dir | Directorio del Inventario |
| inventory_file | Archivo Inventario |
| playbook_dir | Directorio del Playbook |

---

# inventory_hostname

```yaml
debug:

  msg: "{{ inventory_hostname }}"
```

Resultado.

```text
web01
```

---

# playbook_dir

```yaml
debug:

  msg: "{{ playbook_dir }}"
```

---

# group_names

```yaml
debug:

  var: group_names
```

Resultado.

```text
web

production
```

---

# groups

```yaml
debug:

  var: groups
```

Resultado.

```text
web

database

backup
```

---

# hostvars

Una de las Variables más poderosas.

Permite acceder a Variables de otros Hosts.

Ejemplo.

```yaml
{{ hostvars['db01'].ansible_hostname }}
```

---

Arquitectura.

```text
Host Actual

↓

hostvars

↓

Otro Host

↓

Variables
```

---

# Caso Empresarial

Servidor Web.

Necesita conocer la IP del servidor PostgreSQL.

```yaml
{{ hostvars['db01'].ansible_default_ipv4.address }}
```

No es necesario escribir la IP manualmente.

---

# Variables Especiales

Existen Variables utilizadas por Ansible durante la ejecución.

Ejemplo.

```text
ansible_check_mode
```

Indica si estamos utilizando:

```bash
--check
```

---

Otra.

```text
ansible_diff_mode
```

Indica si estamos utilizando.

```bash
--diff
```

---

Otra.

```text
ansible_version
```

Muestra la versión de Ansible.

---

Mostrar.

```yaml
debug:

  var: ansible_version
```

---

# Arquitectura

```text
Playbook

↓

Modo Check

↓

Variable Especial

↓

Decisión
```

---

# Uso de Facts en Condiciones

Ejemplo.

```yaml
when:

  ansible_distribution=="Fedora"
```

---

Otro.

```yaml
when:

  ansible_memory_mb.real.total > 4096
```

---

Otro.

```yaml
when:

  ansible_processor_vcpus >= 4
```

---

# Automatización Inteligente

```text
RAM

↓

Más de 8 GB

↓

Instalar PostgreSQL
```

---

```text
RAM

↓

Menos de 2 GB

↓

No instalar
```

---

# Jinja2 Filters

Jinja2 incorpora filtros para transformar Variables.

---

# upper

```yaml
{{ usuario | upper }}
```

Resultado.

```text
ADMINISTRADOR
```

---

# lower

```yaml
{{ usuario | lower }}
```

---

# capitalize

```yaml
{{ usuario | capitalize }}
```

---

# length

```yaml
{{ paquetes | length }}
```

Resultado.

```text
4
```

---

# default

Permite asignar un valor si la Variable no existe.

```yaml
{{ puerto | default(80) }}
```

---

# join

```yaml
{{ paquetes | join(', ') }}
```

Resultado.

```text
git, wget, vim
```

---

# split

```yaml
{{ texto | split(',') }}
```

---

# replace

```yaml
{{ hostname | replace('-', '_') }}
```

---

# trim

```yaml
{{ usuario | trim }}
```

---

# first

```yaml
{{ paquetes | first }}
```

---

# last

```yaml
{{ paquetes | last }}
```

---

# sort

```yaml
{{ paquetes | sort }}
```

---

# unique

```yaml
{{ paquetes | unique }}
```

---

# Flujo

```text
Variable

↓

Filtro

↓

Resultado

↓

Task
```

---

# Ejemplo Completo

```yaml
---
- name: Información del Sistema

  hosts: all

  tasks:

    - name: Mostrar Hostname

      debug:

        msg: "{{ ansible_hostname }}"

    - name: Mostrar Kernel

      debug:

        msg: "{{ ansible_kernel }}"

    - name: Mostrar Distribución

      debug:

        msg: "{{ ansible_distribution }}"

    - name: Mostrar IP

      debug:

        msg: "{{ ansible_default_ipv4.address }}"
```

---

# Buenas Prácticas

- Utilizar Facts únicamente cuando sean necesarios.
- Desactivar `gather_facts` en tareas simples.
- Consultar únicamente los Facts requeridos.
- Aprovechar `hostvars` para evitar información duplicada.
- Utilizar filtros de Jinja2 para simplificar expresiones.
- Evitar cálculos complejos dentro de las Tasks.
- Utilizar Variables descriptivas.
- Probar los filtros antes de Producción.
- Documentar Variables especiales.
- Evitar depender de Facts que puedan no existir en todos los sistemas.

---

# Errores Comunes

## Error 1

Olvidar que `gather_facts` puede estar deshabilitado.

---

## Error 2

Intentar acceder a un Fact inexistente.

---

## Error 3

No utilizar filtros cuando son necesarios.

---

## Error 4

No comprender la diferencia entre Facts y Magic Variables.

---

## Error 5

Copiar manualmente información disponible mediante `hostvars`.

---

## Error 6

Ejecutar `setup` innecesariamente sobre cientos de servidores.

---

## Error 7

Utilizar nombres incorrectos de Variables.

---

## Error 8

No validar el resultado del módulo `setup`.

---

## Error 9

Olvidar que algunos Facts dependen del sistema operativo.

---

## Error 10

No aprovechar las Variables especiales durante pruebas.

---

# Laboratorio RHCSA

## Escenario

Una empresa administra:

- 150 servidores Fedora.
- 80 servidores Red Hat Enterprise Linux.
- 40 servidores PostgreSQL.

El objetivo es crear un Playbook inteligente que adapte su comportamiento según las características de cada servidor.

---

## Laboratorio 1

Ejecutar el módulo:

```bash
ansible all -m setup
```

Analizar la información obtenida.

---

## Laboratorio 2

Consultar únicamente:

- Hostname.
- Kernel.
- Memoria.
- Dirección IP.

Utilizando `filter=`.

---

## Laboratorio 3

Crear un Playbook que muestre:

- Distribución.
- Kernel.
- Arquitectura.
- Memoria.
- CPU.

---

## Laboratorio 4

Utilizar `inventory_hostname`.

---

## Laboratorio 5

Mostrar:

```yaml
group_names
```

---

## Laboratorio 6

Acceder mediante `hostvars` a la dirección IP de otro servidor del Inventario.

---

## Laboratorio 7

Crear una condición que instale un paquete únicamente cuando el servidor tenga más de 4 GB de memoria RAM.

---

## Laboratorio 8

Utilizar los filtros:

- upper
- lower
- length
- join
- default

para transformar Variables y analizar los resultados.

---

## Laboratorio 9

Ejecutar el Playbook con:

```bash
ansible-playbook sistema.yml --check --diff
```

Mostrar mediante Variables especiales si el Playbook está ejecutándose en modo de comprobación.

---

## Laboratorio 10

Desactivar `gather_facts`, ejecutar el Playbook y comparar el tiempo de ejecución con una versión que tenga `gather_facts: true`. Explicar cuándo conviene habilitar o deshabilitar la recopilación automática de Facts.

---

# Preguntas de Repaso

1. ¿Qué función cumple `gather_facts`?
2. ¿Qué módulo recopila los Facts?
3. ¿Cuándo conviene desactivar `gather_facts`?
4. ¿Qué diferencia existe entre un Fact y una Magic Variable?
5. ¿Qué utilidad tiene `inventory_hostname`?
6. ¿Qué información proporciona `hostvars`?
7. ¿Qué ventajas ofrecen los filtros de Jinja2?
8. ¿Qué hace el filtro `default`?
9. ¿Qué representa la Variable `ansible_check_mode`?
10. ¿Cómo utilizarías los Facts para crear un Playbook adaptable a distintos sistemas operativos?

---

# Resumen

En esta tercera fase profundizamos en el uso de **Facts**, el mecanismo mediante el cual Ansible recopila información del sistema remoto utilizando el módulo `setup`. Estudiamos el funcionamiento de `gather_facts`, aprendimos a consultar únicamente los Facts necesarios y analizamos algunos de los más utilizados en entornos empresariales.

También exploramos las **Magic Variables**, las Variables especiales generadas por el motor de Ansible y los filtros de **Jinja2**, herramientas fundamentales para construir Playbooks dinámicos, reutilizables e inteligentes. Con estos conocimientos es posible crear automatizaciones capaces de adaptarse automáticamente a cada servidor sin modificar el código.

En la **Fase 4** estudiaremos los **Facts personalizados (Custom Facts)**, el almacenamiento persistente de información, el uso avanzado de `set_fact`, `delegate_facts`, la administración de secretos mediante **Ansible Vault**, las mejores prácticas empresariales, troubleshooting, auditoría y un laboratorio integral orientado al examen RHCSA y a entornos de Producción.

----


# 83. Variables y Facts en Ansible (Fase 4)

> **Módulo 11 – Automatización con Ansible**
>
> **Archivo:** `83-variables-facts.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Comprender qué son los Custom Facts.
- Crear Facts personalizados.
- Utilizar `set_fact`.
- Comprender `delegate_facts`.
- Administrar información persistente.
- Introducir el uso de Ansible Vault para proteger Variables sensibles.
- Aplicar buenas prácticas empresariales.
- Diagnosticar problemas relacionados con Variables y Facts.
- Implementar un laboratorio integral tipo RHCSA.

---

# Introducción

Hasta ahora hemos aprendido:

- Variables
- Facts
- group_vars
- host_vars
- vars_files
- Extra Vars
- Magic Variables
- Jinja2

Sin embargo, todavía existe una necesidad muy frecuente en ambientes empresariales.

Muchas veces necesitamos almacenar información que:

- no existe en el sistema operativo;
- no proviene del Inventario;
- no queremos escribir directamente en el Playbook.

Por ejemplo:

- Código del centro de datos
- Número del rack
- Responsable del servidor
- Ambiente (Producción, QA o Desarrollo)
- Aplicación instalada
- Fecha del último mantenimiento

Para ello utilizamos los **Custom Facts**.

---

# Arquitectura General

```text
                 Variables

                      │

       ┌──────────────┼──────────────┐

       ▼              ▼              ▼

 User Vars         Facts        Custom Facts

       │              │              │

       └──────────────┼──────────────┘

                      ▼

                  Playbook

                      ▼

                 Automatización
```

---

# ¿Qué son los Custom Facts?

Son Facts creados por el administrador.

A diferencia de los Facts normales:

```text
Facts

↓

Sistema Operativo
```

Los Custom Facts provienen de información definida por la organización.

---

# Ejemplos

```text
Empresa

↓

Servidor Web

↓

Aplicación

↓

Responsable

↓

Ubicación
```

---

# Directorio

En Red Hat y Fedora normalmente se almacenan en:

```text
/etc/ansible/facts.d/
```

---

# Ejemplo

```text
/etc/ansible/facts.d/

└── empresa.fact
```

---

# Archivo

Puede escribirse en formato INI.

```ini
[general]

empresa=Popular

ambiente=Produccion

responsable=DBA
```

---

También puede utilizar JSON.

```json
{
   "empresa":"Popular",
   "ambiente":"Produccion",
   "responsable":"DBA"
}
```

---

# Flujo

```text
Archivo

↓

facts.d

↓

setup

↓

Custom Facts

↓

Playbook
```

---

# Recopilar nuevamente

Después de crear un Custom Fact.

```bash
ansible all -m setup
```

---

# Acceder

```yaml
{{ ansible_local }}
```

---

Ejemplo.

```yaml
{{ ansible_local.empresa.general.empresa }}
```

---

Resultado.

```text
Popular
```

---

# Arquitectura

```text
setup

↓

ansible_local

↓

empresa

↓

general

↓

empresa
```

---

# Beneficios

- Información centralizada.
- No depende del Inventario.
- Fácil mantenimiento.
- Persistente.
- Muy utilizada en Producción.

---

# set_fact

Una de las instrucciones más utilizadas.

---

## ¿Qué hace?

Crea Variables durante la ejecución del Playbook.

---

# Sintaxis

```yaml
- name: Crear Variable

  set_fact:

    servidor_web: web01
```

---

Posteriormente.

```yaml
debug:

  msg: "{{ servidor_web }}"
```

---

Resultado.

```text
web01
```

---

# Flujo

```text
Task

↓

set_fact

↓

Nueva Variable

↓

Tasks siguientes
```

---

# Ejemplo

```yaml
- name: Puerto

  set_fact:

    puerto: 8080
```

---

# Variables Calculadas

```yaml
- name: Calcular

  set_fact:

    total_memoria: "{{ ansible_memory_mb.real.total }}"
```

---

# Concatenar

```yaml
set_fact:

  servidor: "{{ ansible_hostname }}.empresa.local"
```

---

Resultado.

```text
web01.empresa.local
```

---

# Uso con Condiciones

```yaml
- name: Ambiente

  set_fact:

    ambiente: Produccion

  when:

    ansible_hostname=="web01"
```

---

# Uso con Loops

```yaml
set_fact:

  paquete_actual: "{{ item }}"
```

---

# Arquitectura

```text
Facts

↓

set_fact

↓

Nueva Variable

↓

Playbook
```

---

# delegate_facts

Una característica avanzada.

---

## ¿Qué hace?

Permite almacenar Variables en otro Host.

---

Ejemplo.

```yaml
delegate_to: db01

delegate_facts: true
```

---

Arquitectura.

```text
Host Web

↓

Delegar

↓

Host DB

↓

Guardar Facts
```

---

Muy útil para:

- Clústeres.
- Balanceadores.
- Inventarios dinámicos.
- Automatización distribuida.

---

# Variables Persistentes

```text
Custom Facts

↓

Persisten

↓

Reinicio

↓

Continúan disponibles
```

---

# Variables Temporales

```text
set_fact

↓

Durante ejecución

↓

Fin Playbook

↓

Desaparecen
```

---

# Cuándo utilizar cada una

| Tipo | Uso recomendado |
|--------|----------------|
| vars | Datos simples |
| vars_files | Configuración compartida |
| group_vars | Variables comunes para un grupo |
| host_vars | Excepciones de un servidor |
| Facts | Información del sistema |
| set_fact | Información temporal |
| Custom Facts | Información permanente del servidor |

---

# Introducción a Ansible Vault

No todas las Variables pueden almacenarse en texto plano.

Ejemplos.

```text
Contraseñas

Tokens

API Keys

Certificados

Llaves privadas
```

---

# Problema

```yaml
password: MiPassword123
```

Esto representa un riesgo de seguridad.

---

# Solución

Ansible Vault.

---

# ¿Qué es Ansible Vault?

Permite cifrar archivos que contienen información sensible.

---

# Crear un archivo cifrado

```bash
ansible-vault create secretos.yml
```

---

# Editar

```bash
ansible-vault edit secretos.yml
```

---

# Ver

```bash
ansible-vault view secretos.yml
```

---

# Cambiar contraseña

```bash
ansible-vault rekey secretos.yml
```

---

# Cifrar un archivo existente

```bash
ansible-vault encrypt variables.yml
```

---

# Descifrar

```bash
ansible-vault decrypt variables.yml
```

---

# Ejecutar Playbook

```bash
ansible-playbook sitio.yml \
--ask-vault-pass
```

---

O utilizando un archivo de contraseña.

```bash
ansible-playbook sitio.yml \
--vault-password-file vault.key
```

---

# Flujo

```text
Variables

↓

Vault

↓

Archivo Cifrado

↓

Playbook

↓

Variables disponibles
```

---

# Ventajas

- Protección de credenciales.
- Integración con Git.
- Cumplimiento de políticas.
- Mayor seguridad.
- Administración centralizada.

---

# Caso Empresarial

Empresa.

500 servidores.

Cada servidor necesita:

- Usuario Oracle.
- Password PostgreSQL.
- Token API.
- Clave SSH.

Nunca deben almacenarse en texto plano.

La solución:

```text
Ansible Vault
```

---

# Troubleshooting

## Variable no encontrada

```text
VARIABLE IS UNDEFINED
```

Verificar.

- Nombre.
- Ortografía.
- Precedencia.
- Archivo cargado.

---

## Fact inexistente

```text
undefined variable
```

Posibles causas.

- gather_facts=false
- setup no ejecutado
- Nombre incorrecto

---

## Archivo Vault

```text
Decryption failed
```

Verificar.

- Contraseña.
- Archivo.
- Integridad.

---

## host_vars

No carga.

Verificar.

```text
Nombre exacto

↓

web01.yml

↓

Debe coincidir

↓

inventory
```

---

# Auditoría

Antes de Producción.

Verificar.

```text
Variables

↓

Facts

↓

Vault

↓

Inventario

↓

group_vars

↓

host_vars

↓

Sintaxis
```

---

# Flujo recomendado

```text
Diseño

↓

Variables

↓

Vault

↓

Testing

↓

QA

↓

Producción
```

---

# Organización Profesional

```text
ansible/

├── inventories/

├── group_vars/

├── host_vars/

├── vars/

├── vault/

│      ├── database.yml

│      ├── oracle.yml

│      └── passwords.yml

├── templates/

├── files/

└── playbooks/
```

---

# Buenas Prácticas

- Utilizar nombres descriptivos para Variables.
- Evitar duplicar información.
- Aprovechar Facts antes de crear nuevas Variables.
- Utilizar `set_fact` únicamente cuando sea necesario.
- Utilizar Custom Facts para información permanente.
- Proteger credenciales con Vault.
- Mantener archivos Vault fuera de repositorios públicos.
- Documentar el origen de cada Variable.
- Comprender la precedencia de Variables.
- Separar claramente Variables de Desarrollo, QA y Producción.

---

# Errores Comunes

## Error 1

Guardar contraseñas en texto plano.

---

## Error 2

Crear demasiadas Variables temporales.

---

## Error 3

Duplicar Variables entre `group_vars` y `host_vars`.

---

## Error 4

No utilizar Vault.

---

## Error 5

Confundir `set_fact` con Variables permanentes.

---

## Error 6

Olvidar ejecutar `setup` después de crear un Custom Fact.

---

## Error 7

Modificar Custom Facts manualmente sin control de versiones.

---

## Error 8

Utilizar nombres ambiguos.

---

## Error 9

No documentar Variables críticas.

---

## Error 10

No validar la precedencia de Variables antes de Producción.

---

# Laboratorio Integral RHCSA

## Escenario

Una empresa administra:

- 250 servidores Linux.
- 80 PostgreSQL.
- 50 SQL Server.
- 40 Oracle.
- 100 Web Servers.

Todos deben compartir Variables comunes y credenciales seguras.

---

## Laboratorio 1

Crear un Custom Fact con:

- Empresa.
- Ambiente.
- Responsable.

---

## Laboratorio 2

Mostrar los valores utilizando:

```yaml
ansible_local
```

---

## Laboratorio 3

Crear Variables mediante `set_fact`.

---

## Laboratorio 4

Calcular una nueva Variable utilizando información proveniente de un Fact.

---

## Laboratorio 5

Crear un archivo cifrado utilizando:

```bash
ansible-vault create secretos.yml
```

---

## Laboratorio 6

Editar el archivo.

---

## Laboratorio 7

Ejecutar un Playbook utilizando:

```bash
--ask-vault-pass
```

---

## Laboratorio 8

Mover todas las contraseñas del proyecto hacia archivos protegidos con Vault.

---

## Laboratorio 9

Crear una estructura empresarial con:

- group_vars
- host_vars
- vars
- vault
- templates
- playbooks

---

## Laboratorio 10

Diseñar una estrategia completa para una infraestructura empresarial que contemple:

- Variables reutilizables.
- Facts automáticos.
- Custom Facts.
- Variables temporales mediante `set_fact`.
- Variables protegidas mediante Ansible Vault.
- Validación de sintaxis.
- Pruebas en laboratorio.
- Despliegue seguro en Producción.
- Documentación de Variables y credenciales.

---

# Preguntas de Repaso

1. ¿Qué es un Custom Fact?
2. ¿Dónde se almacenan normalmente los Custom Facts?
3. ¿Qué diferencia existe entre un Fact normal y un Custom Fact?
4. ¿Qué hace `set_fact`?
5. ¿Cuándo conviene utilizar `set_fact`?
6. ¿Qué función cumple `delegate_facts`?
7. ¿Qué diferencia existe entre Variables temporales y persistentes?
8. ¿Qué es Ansible Vault?
9. ¿Qué tipo de información debe protegerse con Vault?
10. ¿Por qué no es recomendable almacenar credenciales en texto plano?
11. ¿Qué comando permite crear un archivo protegido con Vault?
12. ¿Qué ventajas ofrecen los Custom Facts frente a escribir datos directamente en el Inventario?
13. ¿Qué problemas comunes pueden aparecer al trabajar con Variables y Facts?
14. ¿Qué buenas prácticas ayudan a mantener un proyecto Ansible seguro y organizado?
15. ¿Cómo estructurarías un proyecto empresarial que utilice Variables, Facts, Custom Facts y Ansible Vault?

---

# Resumen del Capítulo

En este capítulo profundizamos en la administración de **Variables y Facts**, componentes fundamentales para desarrollar automatizaciones flexibles y escalables con Ansible. Estudiamos las distintas fuentes de Variables, la precedencia entre ellas, el funcionamiento de `gather_facts`, los Facts automáticos, las Variables mágicas y los filtros de Jinja2.

Además, incorporamos conceptos avanzados como los **Custom Facts**, `set_fact` y `delegate_facts`, que permiten crear y administrar información dinámica y persistente durante la ejecución de los Playbooks. Finalmente, introdujimos **Ansible Vault**, la herramienta recomendada para proteger contraseñas, claves y demás información sensible, junto con prácticas de organización, auditoría y troubleshooting orientadas a entornos empresariales.

Con este conocimiento ya es posible diseñar Playbooks altamente reutilizables, seguros y preparados para infraestructuras de gran escala.

---

# Próximo Capítulo

## **84. Roles en Ansible**

En el siguiente capítulo aprenderemos a utilizar **Roles**, el mecanismo recomendado para organizar proyectos Ansible profesionales. Estudiaremos la estructura estándar de un Role, directorios como `tasks`, `handlers`, `templates`, `files`, `defaults`, `vars` y `meta`, la reutilización de componentes, el uso de **Ansible Galaxy** y las mejores prácticas para construir automatizaciones modulares y mantenibles.

----





