# 84. Roles en Ansible (Fase 1)

> **Módulo 11 – Automatización con Ansible**
>
> **Archivo:** `84-roles-ansible.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Comprender qué es un Role en Ansible.
- Entender por qué los Roles son fundamentales en ambientes empresariales.
- Diferenciar un Playbook tradicional de uno basado en Roles.
- Conocer la estructura estándar de un Role.
- Crear Roles manualmente.
- Utilizar `ansible-galaxy init`.
- Comprender la filosofía modular de Ansible.
- Aplicar buenas prácticas de organización.

---

# Introducción

Hasta ahora hemos creado Playbooks que contienen:

- Variables
- Tasks
- Handlers
- Templates
- Files

Todo dentro de uno o pocos archivos.

Para pequeños laboratorios esto funciona correctamente.

Sin embargo, imaginemos una empresa que administra:

- 200 servidores Web
- 150 servidores PostgreSQL
- 90 servidores SQL Server
- 60 servidores Oracle
- 300 servidores Linux

¿Sería conveniente tener un Playbook de 5,000 líneas?

La respuesta es **no**.

Para resolver este problema Ansible incorpora uno de sus componentes más importantes:

# Roles

---

# ¿Qué es un Role?

Un Role es una estructura estandarizada que organiza todos los componentes de una automatización.

En lugar de colocar todo dentro del Playbook, cada elemento se almacena en su directorio correspondiente.

---

# Filosofía

Sin Roles.

```text
Playbook

│

├── Variables

├── Tasks

├── Handlers

├── Templates

├── Files

├── Usuarios

├── Firewall

├── Apache

├── PostgreSQL

└── Backup
```

Resultado:

Un archivo enorme.

---

Con Roles.

```text
Playbook

↓

Role Apache

↓

Role PostgreSQL

↓

Role Firewall

↓

Role Usuarios

↓

Role Backup
```

Mucho más organizado.

---

# Analogía

Imaginemos la construcción de un edificio.

Sin organización.

```text
Ladrillos

Tuberías

Ventanas

Cables

Pintura

Todo mezclado.
```

---

Con organización.

```text
Electricidad

↓

Plomería

↓

Estructura

↓

Acabados

↓

Inspección
```

Cada equipo trabaja de forma independiente.

---

Los Roles siguen exactamente la misma filosofía.

---

# Arquitectura General

```text
                    Playbook

                        │

      ┌─────────────────┼─────────────────┐

      ▼                 ▼                 ▼

   Role Web        Role Database     Role Firewall

      │                 │                 │

      ▼                 ▼                 ▼

 Variables         Tasks            Templates

 Handlers          Files            Defaults

 Meta              Tests            Vars
```

---

# ¿Por qué utilizar Roles?

Porque permiten:

- Dividir proyectos grandes.
- Reutilizar código.
- Facilitar mantenimiento.
- Compartir automatizaciones.
- Mejorar la legibilidad.
- Escalar proyectos.
- Trabajar en equipos.
- Versionar componentes.
- Simplificar auditorías.

---

# Playbook Tradicional

Ejemplo.

```text
instalar.yml

↓

1,200 líneas

↓

Todo mezclado
```

---

# Playbook utilizando Roles

```text
site.yml

↓

Role Apache

↓

Role Firewall

↓

Role Usuarios

↓

Role PostgreSQL
```

---

# Diferencia Visual

Sin Roles.

```text
Playbook

↓

500 Tasks

↓

Difícil mantenimiento
```

---

Con Roles.

```text
Playbook

↓

5 Roles

↓

Cada uno con una responsabilidad
```

---

# Principio de Responsabilidad Única

Una de las mejores prácticas del desarrollo de software.

Cada Role debe realizar una única función.

Por ejemplo.

```text
Role Apache

↓

Instalar Apache
```

No debería:

- Crear usuarios.
- Configurar PostgreSQL.
- Administrar Firewall.

---

Otro ejemplo.

```text
Role PostgreSQL

↓

Instalar PostgreSQL
```

No debe instalar Nginx.

---

# Beneficios

```text
Role

↓

Una responsabilidad

↓

Menos errores

↓

Mayor reutilización
```

---

# Estructura de un Proyecto

```text
ansible/

├── inventories/

├── playbooks/

├── roles/

├── templates/

├── files/

├── group_vars/

└── host_vars/
```

---

Dentro de:

```text
roles/
```

encontraremos los distintos Roles.

---

# Ejemplo

```text
roles/

├── apache/

├── postgres/

├── sqlserver/

├── firewall/

├── usuarios/

└── backup/
```

---

Cada uno es independiente.

---

# Crear un Role

Existen dos opciones.

## Manualmente

```text
mkdir roles
```

Luego.

```text
mkdir roles/apache
```

Después crear todos los directorios.

---

## Utilizando Ansible Galaxy

Es la forma recomendada.

```bash
ansible-galaxy init apache
```

---

Resultado.

```text
apache/

├── defaults

├── files

├── handlers

├── meta

├── tasks

├── templates

├── tests

├── vars
```

---

# Arquitectura

```text
Role

│

├── defaults

├── files

├── handlers

├── meta

├── tasks

├── templates

├── tests

└── vars
```

---

# Directorio defaults

Contiene Variables por defecto.

```text
defaults/

└── main.yml
```

---

Ejemplo.

```yaml
---

http_port: 80

document_root: /var/www/html
```

---

Características.

- Baja prioridad.
- Fácilmente sobrescribible.
- Ideal para configuraciones generales.

---

# Directorio vars

Contiene Variables del Role.

```text
vars/

└── main.yml
```

---

Ejemplo.

```yaml
---

paquete: httpd

servicio: httpd
```

---

Generalmente contienen Variables que no cambian frecuentemente.

---

# Diferencia

```text
defaults

↓

Valores modificables
```

---

```text
vars

↓

Valores internos del Role
```

---

# Directorio tasks

Es el corazón del Role.

```text
tasks/

└── main.yml
```

Aquí viven todas las Tasks.

---

Ejemplo.

```yaml
---

- name: Instalar Apache

  dnf:

    name: httpd

    state: present

- name: Iniciar Apache

  service:

    name: httpd

    state: started
```

---

# Flujo

```text
Role

↓

tasks/main.yml

↓

Tasks

↓

Servidor
```

---

# Directorio handlers

Contiene los Handlers del Role.

```text
handlers/

└── main.yml
```

---

Ejemplo.

```yaml
---

- name: Reiniciar Apache

  service:

    name: httpd

    state: restarted
```

---

# Directorio files

Aquí se almacenan archivos que serán copiados al servidor.

Ejemplo.

```text
files/

├── index.html

├── motd

├── banner.txt

└── logo.png
```

---

Luego pueden utilizarse mediante:

```yaml
copy:
```

---

# Directorio templates

Contiene plantillas Jinja2.

```text
templates/

└── httpd.conf.j2
```

---

Las Variables serán reemplazadas automáticamente.

---

# Directorio meta

Contiene información del Role.

```text
meta/

└── main.yml
```

---

Ejemplo.

```yaml
galaxy_info:

  author: Alejandro

  description: Apache Role

  company: Empresa

  license: GPL
```

---

También puede definir dependencias entre Roles.

---

# Directorio tests

Permite probar el Role.

```text
tests/

├── inventory

└── test.yml
```

---

Muy útil antes de Producción.

---

# Flujo Completo

```text
Playbook

↓

Role

↓

Tasks

↓

Handlers

↓

Templates

↓

Servidor
```

---

# Comparación

## Proyecto pequeño

```text
1 Playbook

↓

Sin Roles
```

---

## Proyecto empresarial

```text
Roles

↓

Organización

↓

Escalabilidad
```

---

# Ejemplo Empresarial

Empresa.

300 servidores.

Necesita:

- Apache
- PostgreSQL
- Firewall
- Usuarios
- Backup

Arquitectura.

```text
Playbook

↓

Role Apache

↓

Role PostgreSQL

↓

Role Firewall

↓

Role Backup

↓

Producción
```

---

# Buenas Prácticas

- Crear un Role para cada servicio.
- Mantener una única responsabilidad por Role.
- Utilizar `ansible-galaxy init`.
- No mezclar configuraciones distintas.
- Documentar el propósito del Role.
- Utilizar nombres descriptivos.
- Mantener Roles pequeños.
- Reutilizar Roles siempre que sea posible.
- Versionar Roles mediante Git.
- Probar Roles antes de Producción.

---

# Errores Comunes

## Error 1

Crear un único Role para toda la infraestructura.

---

## Error 2

Guardar archivos de Apache dentro del Role PostgreSQL.

---

## Error 3

No utilizar la estructura estándar.

---

## Error 4

Duplicar código entre Roles.

---

## Error 5

No documentar el Role.

---

## Error 6

Crear Variables duplicadas.

---

## Error 7

No separar Templates y Files.

---

## Error 8

Modificar directamente un Role descargado sin control de versiones.

---

## Error 9

No probar el Role.

---

## Error 10

Utilizar nombres ambiguos.

---

# Laboratorio RHCSA

## Escenario

Una empresa administrará:

- 120 servidores Web.
- 60 servidores PostgreSQL.
- 30 servidores SQL Server.

El objetivo es convertir todos los Playbooks actuales en Roles reutilizables.

---

## Laboratorio 1

Crear el directorio:

```text
roles/
```

---

## Laboratorio 2

Crear un Role manualmente.

```text
apache/
```

---

## Laboratorio 3

Crear el mismo Role utilizando:

```bash
ansible-galaxy init apache
```

Comparar ambas estructuras.

---

## Laboratorio 4

Identificar la función de cada directorio generado.

---

## Laboratorio 5

Crear Variables por defecto dentro de:

```text
defaults/main.yml
```

---

## Laboratorio 6

Crear Variables internas dentro de:

```text
vars/main.yml
```

---

## Laboratorio 7

Agregar dos Tasks dentro de:

```text
tasks/main.yml
```

---

## Laboratorio 8

Crear un Handler dentro de:

```text
handlers/main.yml
```

---

## Preguntas de Repaso

1. ¿Qué es un Role?
2. ¿Qué ventajas ofrecen los Roles frente a un único Playbook?
3. ¿Cuál es la responsabilidad del directorio `tasks`?
4. ¿Qué diferencias existen entre `defaults` y `vars`?
5. ¿Qué almacena el directorio `templates`?
6. ¿Qué función cumple `handlers`?
7. ¿Qué utilidad tiene `meta`?
8. ¿Por qué es recomendable utilizar `ansible-galaxy init`?
9. ¿Qué significa el principio de responsabilidad única aplicado a un Role?
10. ¿Cómo organizarías una infraestructura con cientos de servidores utilizando Roles?

---

# Resumen

En esta primera fase aprendimos que los **Roles** constituyen la base para organizar proyectos Ansible de tamaño mediano y grande. Estudiamos su propósito, ventajas y la estructura estándar generada por `ansible-galaxy init`, comprendiendo la función de directorios como `tasks`, `handlers`, `defaults`, `vars`, `templates`, `files`, `meta` y `tests`.

También analizamos cómo los Roles permiten dividir una infraestructura en componentes reutilizables, independientes y fáciles de mantener, siguiendo el principio de responsabilidad única. Esta organización facilita el trabajo en equipo, la reutilización del código y la escalabilidad de los proyectos.

En la **Fase 2** aprenderemos a utilizar Roles dentro de los Playbooks mediante la directiva `roles`, la ejecución de múltiples Roles, el paso de Variables entre Roles, el orden de ejecución y las dependencias entre ellos.

----

# 84. Roles en Ansible (Fase 2)

> **Módulo 11 – Automatización con Ansible**
>
> **Archivo:** `84-roles-ansible.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Utilizar Roles dentro de un Playbook.
- Ejecutar múltiples Roles.
- Comprender el orden de ejecución.
- Pasar Variables a los Roles.
- Controlar la ejecución mediante Tags.
- Comprender las dependencias entre Roles.
- Organizar proyectos empresariales utilizando Roles.
- Aplicar buenas prácticas para infraestructuras de Producción.

---

# Introducción

En la fase anterior aprendimos:

- Qué es un Role.
- La estructura estándar.
- Los directorios.
- Cómo crear un Role utilizando:

```bash
ansible-galaxy init
```

Ahora veremos cómo utilizar esos Roles dentro de un proyecto real.

---

# Arquitectura General

```text
                 site.yml

                     │

      ┌──────────────┼──────────────┐

      ▼              ▼              ▼

   Apache         PostgreSQL     Firewall

      │              │              │

      ▼              ▼              ▼

   Tasks         Handlers      Templates

      │

      ▼

 Servidores Linux
```

---

# El Playbook Principal

En proyectos empresariales normalmente existe un único Playbook principal.

Por ejemplo.

```text
site.yml
```

Este archivo no contiene cientos de Tasks.

Su función principal consiste en invocar Roles.

---

# Ejemplo

```yaml
---
- hosts: web

  roles:

    - apache
```

---

Representación.

```text
site.yml

↓

Role Apache

↓

tasks/main.yml

↓

Servidor
```

---

# Flujo de Ejecución

```text
Playbook

↓

Leer Roles

↓

Entrar al Role

↓

Ejecutar Tasks

↓

Handlers

↓

Finalizar
```

---

# Ejecutar varios Roles

Un mismo Playbook puede utilizar múltiples Roles.

```yaml
---
- hosts: web

  roles:

    - firewall

    - usuarios

    - apache

    - backup
```

---

Flujo.

```text
Firewall

↓

Usuarios

↓

Apache

↓

Backup
```

---

Los Roles se ejecutan en el orden en que aparecen.

---

# Orden de Ejecución

Supongamos:

```yaml
roles:

  - firewall

  - apache

  - backup
```

Ansible ejecutará:

```text
Firewall

↓

Apache

↓

Backup
```

Nunca cambia el orden automáticamente.

---

# ¿Por qué es importante?

Imaginemos.

Apache necesita que el Firewall ya permita el puerto 80.

Orden correcto.

```text
Firewall

↓

Apache
```

Orden incorrecto.

```text
Apache

↓

Firewall
```

Durante unos segundos el servicio podría quedar inaccesible.

---

# Arquitectura

```text
Playbook

↓

Role 1

↓

Role 2

↓

Role 3

↓

PLAY RECAP
```

---

# Roles Condicionales

Los Roles también pueden ejecutarse utilizando condiciones.

Ejemplo.

```yaml
---
- hosts: all

  roles:

    - role: apache

      when:

        ansible_distribution=="Fedora"
```

---

Resultado.

```text
Fedora

↓

Apache
```

---

```text
Ubuntu

↓

No ejecutar
```

---

# Variables dentro de Roles

Un Role puede recibir Variables.

Ejemplo.

```yaml
---
- hosts: web

  roles:

    - role: apache

      vars:

        http_port: 8080
```

---

Representación.

```text
Playbook

↓

Variable

↓

Role

↓

Task
```

---

# Ventajas

El mismo Role puede utilizar diferentes configuraciones.

Servidor 1.

```text
Puerto 80
```

Servidor 2.

```text
Puerto 8080
```

Servidor 3.

```text
Puerto 9090
```

Todo utilizando el mismo Role.

---

# Variables Externas

También pueden utilizarse:

```yaml
vars_files:
```

Ejemplo.

```yaml
vars_files:

  - variables.yml
```

El Role utilizará automáticamente esas Variables.

---

# Variables del Inventario

También pueden recibirse desde:

```text
group_vars

host_vars

Inventario
```

El Role no necesita conocer su origen.

---

# Flujo

```text
Variables

↓

Role

↓

Tasks

↓

Servidor
```

---

# Roles y Tags

Los Roles también soportan Tags.

---

Ejemplo.

```yaml
roles:

  - role: apache

    tags:

      - web
```

---

Ejecución.

```bash
ansible-playbook site.yml \
--tags web
```

---

Resultado.

```text
Solo Apache
```

---

Otro ejemplo.

```yaml
roles:

  - role: backup

    tags:

      - backup
```

---

Podemos ejecutar únicamente:

```bash
ansible-playbook site.yml \
--tags backup
```

---

# Flujo

```text
Role

↓

Tag

↓

Seleccionado

↓

Ejecutar
```

---

# Roles con Múltiples Tags

```yaml
roles:

  - role: apache

    tags:

      - web

      - produccion
```

---

Ejecutar.

```bash
ansible-playbook site.yml \
--tags produccion
```

---

# Dependencias entre Roles

Algunos Roles necesitan que otros se ejecuten primero.

Ejemplo.

Apache depende de:

- Firewall
- Usuarios

---

Representación.

```text
Firewall

↓

Usuarios

↓

Apache
```

---

# Dependencias mediante meta

Archivo.

```text
meta/main.yml
```

---

Ejemplo.

```yaml
dependencies:

  - role: firewall

  - role: usuarios
```

---

Cuando ejecutemos Apache.

```text
Apache

↓

Firewall

↓

Usuarios

↓

Apache
```

Las dependencias serán ejecutadas automáticamente.

---

# Caso Empresarial

Role PostgreSQL.

Necesita:

- Usuario postgres.
- Directorios.
- Firewall.

Arquitectura.

```text
Usuarios

↓

Firewall

↓

PostgreSQL
```

---

# Roles Reutilizables

El mismo Role puede utilizarse en:

Producción.

```text
Role Apache
```

---

Desarrollo.

```text
Role Apache
```

---

Laboratorio.

```text
Role Apache
```

---

No es necesario duplicar código.

---

# Proyecto Empresarial

```text
Proyecto

│

├── inventories

├── playbooks

├── group_vars

├── host_vars

├── roles

│     ├── apache

│     ├── postgres

│     ├── sqlserver

│     ├── firewall

│     └── backup

└── templates
```

---

# Ejemplo Completo

```yaml
---
- hosts: web

  become: true

  vars:

    http_port: 8080

  roles:

    - firewall

    - usuarios

    - role: apache

      tags:

        - web
```

---

Flujo.

```text
Variables

↓

Firewall

↓

Usuarios

↓

Apache

↓

Servidor
```

---

# Roles en Grandes Empresas

Infraestructura.

```text
600 Servidores
```

Roles.

```text
Apache

↓

Usuarios

↓

Seguridad

↓

Logs

↓

Backup

↓

Monitoreo

↓

PostgreSQL

↓

SQL Server
```

Cada equipo mantiene únicamente sus propios Roles.

---

# Comunicación entre Equipos

```text
Equipo Linux

↓

Role Usuarios
```

---

```text
Equipo DBA

↓

Role PostgreSQL
```

---

```text
Equipo Seguridad

↓

Role Firewall
```

Todos trabajan sobre el mismo proyecto.

---

# Beneficios

- Separación de responsabilidades.
- Reutilización.
- Menor duplicación.
- Mejor mantenimiento.
- Facilita auditorías.
- Fácil escalabilidad.
- Menor cantidad de errores.

---

# Buenas Prácticas

- Mantener Roles pequeños.
- Un único objetivo por Role.
- Utilizar nombres descriptivos.
- Aprovechar `group_vars`.
- Utilizar Variables por defecto.
- Documentar dependencias.
- Utilizar Tags para despliegues parciales.
- Definir claramente el orden de ejecución.
- Versionar Roles con Git.
- Validar cada Role individualmente.

---

# Errores Comunes

## Error 1

Crear un Role que configure Apache, PostgreSQL y Firewall al mismo tiempo.

---

## Error 2

Ignorar el orden de ejecución.

---

## Error 3

Duplicar Variables entre distintos Roles.

---

## Error 4

No documentar dependencias.

---

## Error 5

Crear dependencias circulares.

Ejemplo.

```text
Apache

↓

Firewall

↓

Apache
```

---

## Error 6

No utilizar Tags.

---

## Error 7

Modificar Roles reutilizables directamente en Producción.

---

## Error 8

No validar Variables antes de ejecutar un Role.

---

## Error 9

Utilizar demasiadas Variables internas.

---

## Error 10

No probar los Roles individualmente antes de integrarlos.

---

# Laboratorio RHCSA

## Escenario

Una empresa administrará:

- 200 servidores Web.
- 80 PostgreSQL.
- 40 SQL Server.
- 60 servidores de Aplicaciones.

Todos los servicios deberán instalarse mediante Roles.

---

## Laboratorio 1

Crear un Playbook principal llamado:

```text
site.yml
```

---

## Laboratorio 2

Agregar los Roles.

- firewall
- usuarios
- apache

---

## Laboratorio 3

Modificar el orden de ejecución y analizar el resultado.

---

## Laboratorio 4

Pasar Variables directamente al Role Apache.

---

## Laboratorio 5

Agregar Tags a cada Role.

---

## Laboratorio 6

Ejecutar únicamente:

```text
backup
```

mediante:

```bash
--tags
```

---

## Laboratorio 7

Crear una dependencia en:

```text
meta/main.yml
```

para que Apache dependa del Role Firewall.

---

## Laboratorio 8

Crear un proyecto con:

- 5 Roles.
- Variables compartidas.
- Inventario.
- group_vars.
- host_vars.

---

## Laboratorio 9

Diseñar un flujo de despliegue para Producción donde cada equipo (Linux, Seguridad y DBA) mantenga sus propios Roles, asegurando que puedan ejecutarse conjuntamente mediante un único `site.yml`.

---

## Laboratorio 10

Implementar un Playbook principal que:

- Ejecute únicamente Roles necesarios según el grupo de servidores.
- Utilice Variables provenientes de `group_vars`.
- Aplique Tags para despliegues parciales.
- Respete las dependencias entre Roles.
- Genere un despliegue repetible e idempotente.

---

# Preguntas de Repaso

1. ¿Qué función cumple el archivo `site.yml`?
2. ¿Cómo se ejecuta un Role desde un Playbook?
3. ¿En qué orden se ejecutan varios Roles?
4. ¿Cómo se pasan Variables a un Role?
5. ¿Qué ventajas ofrecen los Tags aplicados a Roles?
6. ¿Qué son las dependencias entre Roles?
7. ¿Dónde se definen las dependencias de un Role?
8. ¿Por qué es importante respetar el orden de ejecución?
9. ¿Cómo ayudan los Roles a que varios equipos trabajen sobre el mismo proyecto?
10. ¿Qué buenas prácticas deben seguirse al diseñar Roles reutilizables?

---

# Resumen

En esta segunda fase aprendimos a integrar **Roles** dentro de un Playbook mediante la directiva `roles`. Estudiamos cómo ejecutar uno o varios Roles, cómo controlar el orden de ejecución y cómo pasar Variables provenientes del Playbook, del Inventario o de `group_vars` y `host_vars`.

También analizamos el uso de **Tags** para ejecutar únicamente determinados Roles y el mecanismo de **dependencias** definido en `meta/main.yml`, que permite automatizar el orden correcto de ejecución entre componentes relacionados. Finalmente, revisamos una arquitectura empresarial donde distintos equipos colaboran manteniendo Roles independientes y reutilizables.

En la **Fase 3** profundizaremos en el contenido interno de un Role: organización avanzada de `tasks`, `handlers`, `files`, `templates`, `defaults`, `vars`, reutilización mediante `include_tasks` e `import_tasks`, así como estrategias para construir Roles modulares preparados para proyectos empresariales de gran escala.

-------

# 84. Roles en Ansible (Fase 3)

> **Módulo 11 – Automatización con Ansible**
>
> **Archivo:** `84-roles-ansible.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Comprender el funcionamiento interno de un Role.
- Organizar Tasks mediante `include_tasks` e `import_tasks`.
- Comprender cuándo utilizar `include_role` e `import_role`.
- Reutilizar componentes entre Roles.
- Utilizar Templates, Files y Handlers de forma profesional.
- Construir Roles modulares.
- Aplicar buenas prácticas utilizadas en grandes organizaciones.

---

# Introducción

En las fases anteriores aprendimos:

- Qué es un Role.
- Cómo crear Roles.
- Cómo utilizarlos dentro de un Playbook.
- Cómo pasar Variables.
- Cómo administrar dependencias.

Ahora veremos cómo desarrollar Roles verdaderamente profesionales.

En una empresa un Role puede llegar fácilmente a tener:

- cientos de Tasks
- múltiples Templates
- numerosos archivos
- varios Handlers
- lógica condicional
- diferentes configuraciones para Producción y Desarrollo

Si toda esa lógica permanece en un único archivo `tasks/main.yml`, el mantenimiento se vuelve muy complicado.

La solución consiste en dividir el Role en componentes más pequeños.

---

# Arquitectura de un Role Empresarial

```text
Role Apache

│

├── defaults

├── vars

├── tasks

├── handlers

├── files

├── templates

├── meta

└── tests
```

Cada directorio tiene una responsabilidad específica.

---

# Organización de Tasks

El error más común es colocar todas las Tasks aquí:

```text
tasks/

└── main.yml
```

y terminar con un archivo de miles de líneas.

---

# Mejor Organización

```text
tasks/

├── main.yml

├── install.yml

├── configure.yml

├── firewall.yml

├── service.yml

├── security.yml

└── verify.yml
```

Cada archivo representa una etapa del despliegue.

---

# Flujo

```text
main.yml

│

├── install.yml

├── configure.yml

├── firewall.yml

├── service.yml

└── verify.yml
```

---

# include_tasks

Permite cargar Tasks dinámicamente.

Ejemplo.

```yaml
- name: Instalar Apache

  include_tasks: install.yml
```

---

Posteriormente.

```yaml
- name: Configuración

  include_tasks: configure.yml
```

---

Representación.

```text
main.yml

↓

include_tasks

↓

install.yml
```

---

# Ventajas

- Divide el código.
- Facilita mantenimiento.
- Mejora legibilidad.
- Reduce duplicación.

---

# import_tasks

También permite dividir Tasks.

Ejemplo.

```yaml
- import_tasks: install.yml
```

---

# Diferencia

Aunque ambos parecen similares, su funcionamiento es diferente.

---

## include_tasks

Carga las Tasks durante la ejecución.

```text
Playbook

↓

Ejecución

↓

Leer archivo
```

---

## import_tasks

Carga las Tasks antes de iniciar la ejecución.

```text
Playbook

↓

Analizar

↓

Importar

↓

Ejecutar
```

---

# Comparación

| include_tasks | import_tasks |
|----------------|--------------|
| Dinámico | Estático |
| Se carga durante la ejecución | Se carga antes de ejecutar |
| Admite mayor flexibilidad | Mejor para estructuras fijas |
| Ideal para condiciones | Ideal para código permanente |

---

# ¿Cuál utilizar?

En la mayoría de proyectos modernos:

```text
include_tasks
```

porque ofrece mayor flexibilidad.

---

# Ejemplo

Archivo principal.

```yaml
---

- include_tasks: install.yml

- include_tasks: configure.yml

- include_tasks: service.yml
```

---

Resultado.

```text
Main

↓

Install

↓

Configure

↓

Service
```

---

# include_role

También podemos ejecutar un Role desde otro Playbook o desde una Task.

Ejemplo.

```yaml
- include_role:

    name: apache
```

---

Flujo.

```text
Task

↓

include_role

↓

Role Apache
```

---

# import_role

Versión estática.

```yaml
- import_role:

    name: apache
```

---

# Comparación

| include_role | import_role |
|---------------|-------------|
| Dinámico | Estático |
| Flexible | Predecible |
| Muy utilizado con condiciones | Muy utilizado en Roles fijos |

---

# Organización Profesional

Un Role empresarial suele dividir sus Tasks por responsabilidades.

Ejemplo.

```text
tasks/

├── install.yml

↓

Instalación

├── configure.yml

↓

Configuración

├── firewall.yml

↓

Firewall

├── users.yml

↓

Usuarios

├── service.yml

↓

Servicios

├── security.yml

↓

Hardening

└── verify.yml

↓

Validaciones
```

---

# Flujo Empresarial

```text
Instalar

↓

Configurar

↓

Usuarios

↓

Firewall

↓

Servicio

↓

Validación
```

---

# Templates

Ya estudiamos Jinja2.

Los Templates viven en:

```text
templates/
```

---

Ejemplo.

```text
templates/

└── httpd.conf.j2
```

---

Uso.

```yaml
template:

  src: httpd.conf.j2

  dest: /etc/httpd/conf/httpd.conf
```

---

Flujo.

```text
Template

↓

Variables

↓

Archivo

↓

Servidor
```

---

# Files

Archivos que no necesitan Variables.

Ejemplo.

```text
files/

├── motd

├── banner

├── index.html

└── logo.png
```

---

Uso.

```yaml
copy:

  src: index.html

  dest: /var/www/html/
```

---

# Diferencia

Templates.

```text
Variables

↓

Sí
```

---

Files.

```text
Variables

↓

No
```

---

# Handlers

Dentro del Role.

```text
handlers/

└── main.yml
```

---

Ejemplo.

```yaml
- name: Reiniciar Apache

  service:

    name: httpd

    state: restarted
```

---

Invocación.

```yaml
notify:

  - Reiniciar Apache
```

---

Flujo.

```text
Task

↓

Notify

↓

Handler

↓

Restart
```

---

# Defaults

Configuraciones modificables.

```yaml
http_port: 80
```

---

Muy utilizadas para permitir personalización.

---

# Vars

Configuraciones internas.

```yaml
paquete: httpd
```

---

Normalmente no deberían modificarse.

---

# Meta

Contiene:

- Dependencias.
- Autor.
- Licencia.
- Información del Role.

---

Ejemplo.

```yaml
dependencies:

  - role: firewall
```

---

# Tests

Permiten validar el Role.

```text
tests/

├── inventory

└── test.yml
```

---

Muy recomendables antes de Producción.

---

# Modularización

En grandes organizaciones.

```text
Role Apache

↓

20 archivos

↓

Cada archivo

↓

Una responsabilidad
```

---

# Caso Empresarial

Infraestructura.

```text
600 Servidores
```

Role Apache.

```text
Instalación

↓

Configuración

↓

SSL

↓

Firewall

↓

Logs

↓

SELinux

↓

Servicio

↓

Validación
```

Cada componente en un archivo diferente.

---

# Beneficios

- Fácil mantenimiento.
- Escalable.
- Fácil depuración.
- Código reutilizable.
- Trabajo colaborativo.
- Menor riesgo de errores.

---

# Organización Recomendada

```text
roles/

└── apache/

    ├── defaults

    │     └── main.yml

    │

    ├── files

    │

    ├── handlers

    │     └── main.yml

    │

    ├── meta

    │     └── main.yml

    │

    ├── tasks

    │     ├── main.yml

    │     ├── install.yml

    │     ├── configure.yml

    │     ├── firewall.yml

    │     ├── service.yml

    │     └── verify.yml

    │

    ├── templates

    │

    ├── tests

    │

    └── vars

          └── main.yml
```

---

# Buenas Prácticas

- Dividir Tasks por funcionalidad.
- Mantener archivos pequeños.
- Utilizar `include_tasks` para Roles complejos.
- Separar Templates y Files.
- Mantener Handlers independientes.
- Evitar Variables duplicadas.
- Documentar cada archivo.
- Probar cada módulo individualmente.
- Utilizar nombres descriptivos.
- Diseñar Roles reutilizables.

---

# Errores Comunes

## Error 1

Crear un único `tasks/main.yml` de miles de líneas.

---

## Error 2

Guardar archivos de configuración dentro de `files` cuando deberían ser Templates.

---

## Error 3

Duplicar Tasks.

---

## Error 4

No reutilizar `include_tasks`.

---

## Error 5

Colocar Variables internas en `defaults`.

---

## Error 6

No documentar dependencias.

---

## Error 7

No utilizar Handlers.

---

## Error 8

Copiar archivos manualmente en lugar de usar `template`.

---

## Error 9

No validar el Role antes de Producción.

---

## Error 10

Mezclar responsabilidades dentro del mismo archivo.

---

# Laboratorio RHCSA

## Escenario

Una empresa administrará más de 500 servidores Linux mediante un único Role Apache.

El objetivo es mantener el Role completamente modular.

---

## Laboratorio 1

Crear la estructura completa del Role Apache.

---

## Laboratorio 2

Dividir las Tasks en:

- install.yml
- configure.yml
- firewall.yml
- service.yml
- verify.yml

---

## Laboratorio 3

Crear un `main.yml` que utilice:

```yaml
include_tasks
```

---

## Laboratorio 4

Crear un Template para:

```text
httpd.conf
```

---

## Laboratorio 5

Copiar un archivo estático utilizando:

```yaml
copy
```

---

## Laboratorio 6

Crear un Handler para reiniciar Apache.

---

## Laboratorio 7

Modificar una configuración utilizando Variables definidas en `defaults`.

---

## Laboratorio 8

Crear una dependencia en `meta/main.yml` para que el Role Apache requiera previamente el Role Firewall.

---

## Laboratorio 9

Reemplazar `include_tasks` por `import_tasks` y comparar el comportamiento del Role, identificando en qué escenarios resulta más conveniente cada enfoque.

---

## Laboratorio 10

Diseñar un Role empresarial capaz de instalar y configurar Apache siguiendo una estructura completamente modular, con:

- Tasks separadas por responsabilidad.
- Templates parametrizados.
- Archivos estáticos en `files`.
- Handlers para reinicios controlados.
- Variables en `defaults` y `vars`.
- Dependencias declaradas en `meta`.
- Validaciones finales antes de concluir la ejecución.

---

# Preguntas de Repaso

1. ¿Por qué es recomendable dividir `tasks/main.yml` en varios archivos?
2. ¿Qué diferencia existe entre `include_tasks` e `import_tasks`?
3. ¿Cuándo utilizarías `include_role`?
4. ¿Qué ventajas ofrece `import_role`?
5. ¿Qué diferencias existen entre `templates` y `files`?
6. ¿Cuál es la función de `handlers/main.yml`?
7. ¿Qué información suele almacenarse en `defaults`?
8. ¿Qué propósito tiene el directorio `tests`?
9. ¿Cómo mejora la modularización el mantenimiento de un Role?
10. ¿Qué estructura utilizarías para un Role destinado a cientos de servidores?

---

# Resumen

En esta tercera fase profundizamos en la organización interna de los **Roles**. Aprendimos a dividir la lógica utilizando `include_tasks` e `import_tasks`, analizamos las diferencias entre `include_role` e `import_role` y estudiamos cómo estructurar un Role para que sea modular, reutilizable y fácil de mantener.

También revisamos el uso profesional de los directorios `templates`, `files`, `handlers`, `defaults`, `vars`, `meta` y `tests`, comprendiendo cómo cada uno cumple una función específica dentro de un proyecto Ansible. Finalmente, analizamos una arquitectura empresarial orientada a Roles de gran tamaño, preparados para administrar cientos o miles de servidores.

En la **Fase 4** estudiaremos **Ansible Galaxy**, la reutilización de Roles de terceros, la creación y publicación de Roles propios, control de versiones, pruebas, buenas prácticas empresariales, troubleshooting y un laboratorio integral orientado al examen RHCSA y a entornos de Producción.

----

# 84. Roles en Ansible (Fase 4)

> **Módulo 11 – Automatización con Ansible**
>
> **Archivo:** `84-roles-ansible.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Comprender qué es Ansible Galaxy.
- Instalar Roles desde Galaxy.
- Crear Roles reutilizables para otras organizaciones.
- Publicar Roles propios.
- Administrar versiones de Roles.
- Implementar pruebas antes de Producción.
- Aplicar estrategias de mantenimiento empresarial.
- Diagnosticar problemas relacionados con Roles.
- Implementar una arquitectura empresarial basada completamente en Roles.

---

# Introducción

Durante este capítulo hemos aprendido:

- Crear Roles.
- Organizar Tasks.
- Dividir proyectos.
- Utilizar Templates.
- Utilizar Handlers.
- Utilizar Variables.
- Modularizar automatizaciones.

Ahora aprenderemos el último paso.

La reutilización profesional.

Una empresa rara vez desarrolla todos sus Roles desde cero.

Normalmente reutiliza Roles ya existentes.

Ese es precisamente el objetivo de:

# Ansible Galaxy

---

# ¿Qué es Ansible Galaxy?

Es el repositorio oficial de Roles y Colecciones de Ansible.

Puede compararse con:

| Tecnología | Repositorio |
|------------|-------------|
| Python | PyPI |
| NodeJS | npm |
| Docker | Docker Hub |
| Kubernetes | Helm Hub |
| Ansible | Galaxy |

---

# Arquitectura

```text
Administrador

      │

      ▼

ansible-galaxy

      │

      ▼

Ansible Galaxy

      │

      ▼

Roles

      │

      ▼

Proyecto
```

---

# Beneficios

Galaxy permite:

- Compartir Roles.
- Descargar Roles.
- Reutilizar automatizaciones.
- Reducir tiempo de desarrollo.
- Mantener versiones.
- Estandarizar proyectos.
- Compartir conocimiento.

---

# Ejemplo

Supongamos que necesitamos instalar Apache.

Tenemos dos opciones.

Opción 1.

```text
Crear todo desde cero.
```

---

Opción 2.

```text
Descargar un Role

↓

Modificar únicamente

↓

Lo necesario
```

---

# El comando ansible-galaxy

Ver ayuda.

```bash
ansible-galaxy --help
```

---

Crear un Role.

```bash
ansible-galaxy init apache
```

---

Listar Roles.

```bash
ansible-galaxy role list
```

---

Buscar Roles.

```bash
ansible-galaxy role search apache
```

---

Instalar un Role.

```bash
ansible-galaxy role install geerlingguy.apache
```

---

Eliminar un Role.

```bash
ansible-galaxy role remove geerlingguy.apache
```

---

# Arquitectura

```text
Galaxy

↓

Descarga

↓

roles/

↓

Playbook
```

---

# Ejemplo

```bash
ansible-galaxy role install geerlingguy.apache
```

Resultado.

```text
roles/

└── geerlingguy.apache
```

---

# Utilizar el Role

```yaml
roles:

  - geerlingguy.apache
```

---

Muy sencillo.

---

# Archivo requirements.yml

En proyectos profesionales no se instalan Roles manualmente.

Se utiliza:

```text
requirements.yml
```

---

Ejemplo.

```yaml
---

roles:

  - name: geerlingguy.apache

  - name: geerlingguy.postgresql

  - name: geerlingguy.firewall
```

---

Instalación.

```bash
ansible-galaxy install \
-r requirements.yml
```

---

Arquitectura.

```text
requirements.yml

↓

Galaxy

↓

Roles

↓

Proyecto
```

---

# Ventajas

- Repetibilidad.
- Automatización.
- Fácil reconstrucción.
- Versionado.
- Auditoría.

---

# Versiones

No siempre queremos la última versión.

Ejemplo.

```yaml
roles:

  - name: geerlingguy.apache

    version: 4.0.0
```

---

Beneficios.

```text
Evita cambios inesperados.
```

---

Muy importante en Producción.

---

# Actualizar Roles

```bash
ansible-galaxy install \
-r requirements.yml \
--force
```

---

# Organización Profesional

```text
Proyecto

├── inventories

├── playbooks

├── roles

├── collections

├── requirements.yml

├── group_vars

├── host_vars

└── README.md
```

---

# Roles Propios

No todos los Roles provienen de Galaxy.

Las empresas desarrollan Roles internos.

Ejemplo.

```text
Role SQL Server

Role PostgreSQL

Role Oracle

Role Backup

Role Monitoring

Role Hardening
```

---

# Compartir Roles

Una empresa puede mantener un repositorio Git.

```text
Git

↓

Role Apache

↓

Role PostgreSQL

↓

Role Backup
```

Todos los administradores utilizan los mismos Roles.

---

# Control de Versiones

Cada modificación importante debe quedar registrada.

Ejemplo.

```text
Versión 1.0

↓

Versión 1.1

↓

Versión 2.0
```

---

# Beneficios

- Reversión.
- Auditoría.
- Historial.
- Colaboración.

---

# Documentación

Todo Role profesional debe incluir.

```text
README.md
```

---

Debe explicar.

- Objetivo.
- Variables.
- Dependencias.
- Ejemplos.
- Requisitos.
- Compatibilidad.
- Autor.

---

# Ejemplo

```text
README.md

↓

Instalación

↓

Variables

↓

Uso

↓

Ejemplos
```

---

# Convenciones de Nombres

Buenos nombres.

```text
apache

postgres

backup

firewall

usuarios
```

---

Malos nombres.

```text
nuevo

role1

prueba

config

linux2
```

---

# Roles Pequeños

Es preferible.

```text
Role Apache

↓

250 líneas
```

Que.

```text
Role Infraestructura

↓

12.000 líneas
```

---

# Testing

Antes de Producción.

Cada Role debe probarse.

---

Proceso recomendado.

```text
Laboratorio

↓

QA

↓

Preproducción

↓

Producción
```

---

# Validación

```bash
ansible-playbook site.yml \
--syntax-check
```

---

Modo simulación.

```bash
ansible-playbook site.yml \
--check
```

---

Mostrar diferencias.

```bash
ansible-playbook site.yml \
--diff
```

---

Verbose.

```bash
ansible-playbook site.yml \
-vvv
```

---

# Integración Continua

Muchas empresas utilizan.

```text
GitHub

↓

Push

↓

CI/CD

↓

Pruebas

↓

Producción
```

Cada cambio en un Role dispara pruebas automáticas.

---

# Estrategia Empresarial

Infraestructura.

```text
800 Servidores
```

Roles.

```text
Apache

↓

PostgreSQL

↓

SQL Server

↓

Backup

↓

Firewall

↓

SELinux

↓

Usuarios

↓

Logs
```

Todos independientes.

---

# Arquitectura Completa

```text
                    Git

                     │

                     ▼

             requirements.yml

                     │

                     ▼

              Ansible Galaxy

                     │

                     ▼

                 Roles

                     │

      ┌──────────────┼──────────────┐

      ▼              ▼              ▼

   Apache        PostgreSQL     Firewall

      │              │              │

      └──────────────┼──────────────┘

                     ▼

                 site.yml

                     ▼

               Inventario

                     ▼

              Servidores Linux
```

---

# Mantenimiento

Una vez en Producción.

Los Roles deben mantenerse.

Proceso.

```text
Nueva Versión

↓

Pruebas

↓

QA

↓

Aprobación

↓

Producción
```

---

# Auditoría

Antes de liberar un Role.

Verificar.

```text
✓ Variables

✓ Templates

✓ Files

✓ Handlers

✓ Dependencias

✓ README

✓ Sintaxis

✓ Versionado

✓ Git

✓ Pruebas
```

---

# Troubleshooting

## Error

```text
Role not found
```

Posibles causas.

- No instalado.
- Ruta incorrecta.
- Nombre incorrecto.

---

## Error

```text
Role already exists
```

Ya existe una copia.

---

## Error

```text
Dependency not found
```

Verificar.

```text
meta/main.yml
```

---

## Error

```text
Undefined Variable
```

Revisar.

- defaults
- vars
- group_vars
- host_vars

---

## Error

```text
Template Error
```

Normalmente ocurre por:

- Variable inexistente.
- Error de sintaxis Jinja2.

---

## Error

```text
Handler not found
```

Verificar.

```yaml
notify
```

y

```text
handlers/main.yml
```

---

# Buenas Prácticas

- Mantener Roles pequeños.
- Utilizar una única responsabilidad.
- Versionar con Git.
- Documentar todos los Roles.
- Utilizar `requirements.yml`.
- Fijar versiones en Producción.
- Validar con `--check`.
- Utilizar `README.md`.
- Reutilizar Roles existentes cuando sea apropiado.
- Automatizar pruebas mediante CI/CD.
- Mantener compatibilidad entre versiones.
- Revisar cambios mediante Pull Requests.
- Evitar modificaciones directas sobre Roles descargados; crear un fork o contribuir al proyecto cuando sea necesario.
- Eliminar código obsoleto y mantener el Role actualizado.

---

# Errores Comunes

## Error 1

No documentar el Role.

---

## Error 2

No utilizar Git.

---

## Error 3

Modificar directamente un Role descargado.

---

## Error 4

No controlar versiones.

---

## Error 5

No realizar pruebas.

---

## Error 6

No utilizar `requirements.yml`.

---

## Error 7

Mezclar responsabilidades.

---

## Error 8

Duplicar Roles.

---

## Error 9

No validar dependencias.

---

## Error 10

Actualizar Producción sin probar previamente.

---

# Laboratorio Integral RHCSA

## Escenario

Una empresa internacional administrará:

- 1.200 servidores Linux.
- 300 servidores PostgreSQL.
- 180 SQL Server.
- 250 servidores Web.
- 80 Balanceadores.
- 120 servidores de Aplicaciones.

Toda la infraestructura será administrada mediante Roles reutilizables.

---

## Laboratorio 1

Crear un proyecto basado completamente en Roles.

---

## Laboratorio 2

Crear:

```text
requirements.yml
```

---

## Laboratorio 3

Agregar Roles públicos.

- Apache.
- PostgreSQL.
- Firewall.

---

## Laboratorio 4

Instalar todos los Roles.

---

## Laboratorio 5

Crear un Role propio.

```text
sqlserver
```

---

## Laboratorio 6

Crear documentación completa.

```text
README.md
```

---

## Laboratorio 7

Versionar el proyecto utilizando Git.

---

## Laboratorio 8

Ejecutar:

```bash
--syntax-check

--check

--diff
```

---

## Laboratorio 9

Diseñar un flujo de integración continua donde cada cambio en un Role sea validado automáticamente antes de llegar a Producción.

---

## Laboratorio 10

Diseñar una arquitectura empresarial completa que incluya:

- Roles propios.
- Roles descargados desde Galaxy.
- `requirements.yml`.
- Inventarios separados para Desarrollo, QA y Producción.
- `group_vars` y `host_vars`.
- Templates y Handlers.
- Validación automática.
- Versionado con Git.
- Documentación técnica.
- Estrategia de despliegue segura y repetible.

---

# Preguntas de Repaso

1. ¿Qué es Ansible Galaxy?
2. ¿Qué ventajas ofrece frente a desarrollar todos los Roles desde cero?
3. ¿Qué función cumple `requirements.yml`?
4. ¿Por qué es recomendable fijar versiones de los Roles?
5. ¿Qué información debe contener un `README.md`?
6. ¿Por qué los Roles deben mantenerse pequeños y especializados?
7. ¿Qué validaciones deben realizarse antes de Producción?
8. ¿Qué problemas comunes pueden aparecer al utilizar Roles descargados?
9. ¿Cómo ayuda Git a la administración de Roles?
10. ¿Cómo organizarías una infraestructura empresarial completamente basada en Roles?

---

# Resumen del Capítulo

En este capítulo aprendimos a utilizar **Roles**, el mecanismo recomendado por Ansible para organizar automatizaciones de forma modular, reutilizable y escalable. Estudiamos la estructura estándar de un Role, la función de cada uno de sus directorios, el uso de Variables, Templates, Files, Handlers y la división de Tasks mediante `include_tasks` e `import_tasks`.

También aprendimos a integrar Roles dentro de un Playbook, administrar dependencias, reutilizar componentes y aprovechar **Ansible Galaxy** para descargar, compartir y versionar Roles. Finalmente, analizamos estrategias de documentación, pruebas, auditoría, control de versiones y despliegue orientadas a entornos empresariales.

Con estos conocimientos ya es posible desarrollar proyectos Ansible organizados, mantenibles y preparados para administrar infraestructuras de gran escala siguiendo las mejores prácticas utilizadas en organizaciones profesionales.

---

# Próximo Capítulo

## **85. Ansible Vault**

En el siguiente capítulo estudiaremos **Ansible Vault**, la herramienta oficial para proteger información sensible en Ansible. Aprenderemos a cifrar archivos, Variables y cadenas de texto, administrar contraseñas de Vault, integrar Vault con `group_vars` y `host_vars`, trabajar con múltiples Vault IDs, automatizar el uso de secretos en entornos empresariales y aplicar buenas prácticas de seguridad para proteger credenciales, claves y datos confidenciales.






