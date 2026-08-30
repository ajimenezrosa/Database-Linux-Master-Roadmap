# 89. Laboratorio Final de Automatización con Ansible (Fase 1)

> **Módulo 11 – Automatización con Ansible**
>
> **Archivo:** `89-laboratorio-ansible.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Integrar todos los conocimientos aprendidos durante el módulo.
- Diseñar un proyecto empresarial utilizando Ansible.
- Organizar correctamente la estructura de un proyecto.
- Automatizar la configuración de múltiples servidores.
- Aplicar buenas prácticas de desarrollo.
- Comprender cómo trabajan los equipos DevOps en ambientes reales.

---

# Introducción

Durante todo el módulo aprendimos:

- Inventarios
- Variables
- Facts
- Roles
- Templates
- Handlers
- Vault
- Galaxy
- Collections
- Troubleshooting

Ahora construiremos un proyecto completo que reúna todos estos conceptos.

Este laboratorio simula una infraestructura empresarial similar a la que administra un equipo de Linux o DevOps.

---

# Escenario Empresarial

La empresa **TechSolutions Corp.** acaba de adquirir una nueva infraestructura para alojar sus aplicaciones corporativas.

La organización cuenta con diferentes equipos de trabajo:

- Infraestructura Linux
- Bases de Datos
- Desarrollo
- DevOps
- Seguridad

Todos compartirán un mismo repositorio de automatización.

---

# Objetivos del Proyecto

Automatizar completamente:

- Instalación de servidores.
- Configuración inicial.
- Usuarios.
- Seguridad.
- Apache.
- PostgreSQL.
- Firewall.
- Actualizaciones.
- Monitoreo.

Todo utilizando Ansible.

---

# Infraestructura

```text
                    Internet
                         │
        ┌────────────────┴──────────────┐
        │                               │
        ▼                               ▼
   Load Balancer                   Bastion Host
        │                               │
        └──────────────┬────────────────┘
                       │
        ┌──────────────┼───────────────┐
        ▼              ▼               ▼
   Web Server 1   Web Server 2   Web Server 3
        │              │               │
        └──────────────┼───────────────┘
                       │
                 PostgreSQL Server
                       │
                Backup Server
```

---

# Servidores

| Servidor | Función |
|----------|---------|
| bastion | Administración |
| web01 | Apache |
| web02 | Apache |
| web03 | Apache |
| db01 | PostgreSQL |
| backup01 | Respaldos |

---

# Sistema Operativo

Todos utilizarán.

```text
Red Hat Enterprise Linux
```

o.

```text
Rocky Linux
```

---

# Requerimientos

Todos los servidores deberán tener.

- Usuario administrador.
- Actualizaciones.
- Firewall.
- SSH.
- NTP.
- SELinux.
- Logs.
- Monitoreo.

---

# Arquitectura del Proyecto

```text
empresa-ansible/

├── ansible.cfg

├── inventories/

│      ├── production

│      └── development

├── group_vars/

├── host_vars/

├── playbooks/

├── roles/

├── templates/

├── files/

├── collections/

├── requirements.yml

└── README.md
```

---

# Organización de Roles

```text
roles/

├── common

├── apache

├── postgres

├── firewall

├── monitoring

├── backup

├── users

├── security

└── ntp
```

---

# Filosofía

Cada Role debe realizar una única función.

Ejemplo.

```text
common

↓

Configuración básica
```

---

```text
apache

↓

Servidor Web
```

---

```text
postgres

↓

Base de Datos
```

---

Nunca mezclar responsabilidades.

---

# Flujo del Proyecto

```text
Git

↓

Clone

↓

requirements.yml

↓

Galaxy

↓

Playbooks

↓

Infraestructura
```

---

# Inventario

```text
production/

hosts.yml
```

---

Ejemplo.

```yaml
all:

  children:

    webservers:

      hosts:

        web01:

        web02:

        web03:

    databases:

      hosts:

        db01:

    backups:

      hosts:

        backup01:
```

---

# Variables Globales

```text
group_vars/

all.yml
```

Ejemplo.

```yaml
timezone: America/Santo_Domingo

admin_user: administrador

ntp_server: pool.ntp.org
```

---

# Variables por Grupo

```text
group_vars/

webservers.yml
```

---

Ejemplo.

```yaml
http_port: 80

https_port: 443
```

---

# Variables Base de Datos

```text
group_vars/

databases.yml
```

---

```yaml
postgres_port: 5432

postgres_version: 17
```

---

# Variables por Host

```text
host_vars/

db01.yml
```

---

```yaml
backup_enabled: true
```

---

# Playbook Principal

```text
playbooks/

site.yml
```

---

Su función será coordinar toda la infraestructura.

---

# Arquitectura

```text
site.yml

↓

Roles

↓

Servidores
```

---

# Ejemplo

```yaml
- hosts: all

  become: true

  roles:

    - common

    - security

    - users
```

---

Después.

```yaml
- hosts: webservers

  become: true

  roles:

    - apache
```

---

Finalmente.

```yaml
- hosts: databases

  become: true

  roles:

    - postgres
```

---

# Beneficios

- Modularidad.
- Reutilización.
- Fácil mantenimiento.
- Escalabilidad.

---

# Integración con Galaxy

Archivo.

```text
requirements.yml
```

---

Ejemplo.

```yaml
collections:

- name: ansible.posix

- name: community.general

- name: community.postgresql
```

---

# Integración con Vault

Las contraseñas estarán protegidas.

```text
Vault

↓

group_vars

↓

Playbooks
```

---

# Arquitectura Completa

```text
Git

↓

requirements.yml

↓

Galaxy

↓

Vault

↓

Playbooks

↓

Roles

↓

Templates

↓

Servidores
```

---

# Flujo Empresarial

```text
Administrador

↓

Git Pull

↓

Actualizar Collections

↓

Ejecutar Playbook

↓

Validar

↓

Documentar
```

---

# Convenciones de Nombres

| Elemento | Convención |
|----------|------------|
| Roles | minúsculas |
| Variables | snake_case |
| Templates | `.j2` |
| Inventarios | YAML |
| Playbooks | Verbos descriptivos |
| Collections | Namespace.Collection |

---

# Estándares

Todos los Playbooks deberán:

- Tener nombres descriptivos.
- Utilizar Variables.
- Ser idempotentes.
- Estar documentados.
- Manejar errores.
- Utilizar Handlers.

---

# Objetivos Técnicos

Durante el laboratorio construiremos una plataforma que:

- Configure servidores automáticamente.
- Instale aplicaciones.
- Proteja secretos mediante Vault.
- Utilice Roles reutilizables.
- Utilice Collections.
- Implemente recuperación automática.
- Sea completamente reproducible.

---

# Buenas Prácticas

- Separar Roles por responsabilidad.
- Evitar Variables duplicadas.
- Mantener Playbooks pequeños.
- Utilizar Handlers.
- Utilizar Templates.
- Versionar mediante Git.
- Documentar el proyecto.
- Mantener Inventarios organizados.
- Utilizar Vault para secretos.
- Probar siempre en Desarrollo antes de Producción.

---

# Errores Comunes

## Error 1

Crear un único Playbook con cientos de tareas.

---

## Error 2

No utilizar Roles.

---

## Error 3

Duplicar Variables.

---

## Error 4

Guardar contraseñas en texto plano.

---

## Error 5

No documentar.

---

## Error 6

No utilizar Inventarios separados.

---

## Error 7

No utilizar Templates.

---

## Error 8

Modificar manualmente los servidores después de automatizarlos.

---

## Error 9

No controlar versiones mediante Git.

---

## Error 10

No validar el proyecto en un entorno de pruebas antes del despliegue.

---

# Laboratorio RHCSA

## Escenario

Construiremos una plataforma empresarial desde cero.

---

## Laboratorio 1

Crear la estructura completa del proyecto.

---

## Laboratorio 2

Crear.

```text
inventories/
```

---

## Laboratorio 3

Crear.

```text
group_vars/
```

---

## Laboratorio 4

Crear.

```text
host_vars/
```

---

## Laboratorio 5

Crear.

```text
roles/
```

---

## Laboratorio 6

Crear.

```text
playbooks/
```

---

## Laboratorio 7

Crear.

```text
requirements.yml
```

---

## Laboratorio 8

Diseñar el inventario para los seis servidores del laboratorio utilizando grupos independientes para Web, Base de Datos y Respaldos.

---

## Laboratorio 9

Definir Variables globales, Variables por grupo y Variables específicas por host siguiendo las buenas prácticas de precedencia de Variables en Ansible.

---

## Laboratorio 10

Diseñar la arquitectura inicial del proyecto documentando el propósito de cada directorio, Role y archivo principal antes de comenzar la implementación.

---

# Preguntas de Repaso

1. ¿Por qué dividir un proyecto en Roles?
2. ¿Qué ventajas ofrece utilizar Inventarios separados?
3. ¿Cuál es la función de `site.yml`?
4. ¿Por qué utilizar `group_vars` y `host_vars`?
5. ¿Qué beneficios aporta Vault en un proyecto empresarial?
6. ¿Por qué integrar Galaxy desde el inicio del proyecto?
7. ¿Qué ventajas ofrece una estructura organizada?
8. ¿Cómo ayuda Git al mantenimiento del proyecto?
9. ¿Qué características debe tener un Playbook empresarial?
10. ¿Por qué la modularidad es uno de los principios más importantes en Ansible?

---

# Resumen

En esta primera fase definimos la arquitectura del proyecto final, organizando la estructura de directorios, los Inventarios, las Variables, los Roles y el Playbook principal. También establecimos los estándares de desarrollo, las convenciones de nombres y las buenas prácticas que seguiremos durante todo el laboratorio.

En la **Fase 2** comenzaremos la implementación del proyecto: desarrollaremos los Roles **common**, **users**, **security** y **firewall**, configurando automáticamente la base común que compartirán todos los servidores de la infraestructura.

-----

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

-----
# 89. Laboratorio Final de Automatización con Ansible (Fase 3)

> **Módulo 11 – Automatización con Ansible**
>
> **Archivo:** `89-laboratorio-ansible.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Implementar el Role **apache**.
- Implementar el Role **postgres**.
- Construir el Role **backup**.
- Implementar el Role **monitoring**.
- Utilizar Handlers correctamente.
- Integrar Templates con Variables.
- Proteger información mediante Ansible Vault.
- Utilizar Collections oficiales.
- Comprender cómo se construye una infraestructura completamente automatizada.

---

# Introducción

En la fase anterior construimos la base del sistema operativo mediante los Roles:

- common
- users
- security
- firewall

Ahora comenzaremos la implementación de los servicios empresariales.

Nuestro objetivo será disponer automáticamente de:

- Servidores Web
- Servidores PostgreSQL
- Sistema de respaldos
- Sistema de monitoreo

Todo administrado desde un único Playbook.

---

# Arquitectura General

```text
                   site.yml

                       │

       ┌───────────────┼────────────────┐

       ▼               ▼                ▼

    Web Servers     Database        Backup Server

       │               │                │

     Apache        PostgreSQL      Backup Role

       │               │                │

       └───────────────┼────────────────┘

                       ▼

                  Monitoring
```

---

# Flujo Empresarial

```text
Servidor Base

↓

Apache

↓

PostgreSQL

↓

Backup

↓

Monitoring

↓

Producción
```

---

# Role Apache

Este Role será responsable de instalar y configurar completamente el servidor Web.

---

# Responsabilidades

- Instalar Apache.
- Configurar Virtual Hosts.
- Configurar DocumentRoot.
- Configurar Logs.
- Configurar SSL.
- Configurar Firewall.
- Reiniciar servicios.

---

# Arquitectura

```text
apache/

├── defaults

├── files

├── handlers

├── tasks

├── templates

└── vars
```

---

# Variables

```yaml
apache_service: httpd

apache_port: 80

apache_ssl_port: 443

document_root: /var/www/html
```

---

# Instalación

```yaml
- name: Instalar Apache

  dnf:

    name: httpd

    state: present
```

---

# Configuración

Archivo.

```text
templates/httpd.conf.j2
```

---

Ejemplo

```text
Listen {{ apache_port }}

ServerAdmin admin@example.com

DocumentRoot {{ document_root }}
```

---

# Handler

Después de modificar la configuración.

Debe reiniciarse Apache.

```yaml
notify:

- Restart Apache
```

---

Handler.

```yaml
- name: Restart Apache

  service:

    name: httpd

    state: restarted
```

---

# Flujo

```text
Template

↓

notify

↓

Handler

↓

Apache Reiniciado
```

---

# Validación

```bash
systemctl status httpd
```

---

También.

```bash
curl localhost
```

---

# Role PostgreSQL

El siguiente componente será el servidor de Base de Datos.

---

# Responsabilidades

- Instalar PostgreSQL.
- Inicializar Cluster.
- Configurar postgresql.conf.
- Configurar pg_hba.conf.
- Crear Base de Datos.
- Crear Usuarios.
- Crear Roles.

---

# Variables

```yaml
postgres_version: 17

postgres_port: 5432

postgres_data:

/var/lib/pgsql/17/data
```

---

# Instalación

```yaml
- name: Instalar PostgreSQL

  dnf:

    name:

      - postgresql17-server

      - postgresql17
```

---

# Inicialización

```bash
postgresql-setup --initdb
```

En Ansible.

```yaml
command:
```

---

# Configuración

Templates.

```text
postgresql.conf.j2

pg_hba.conf.j2
```

---

# Variables

Ejemplo.

```yaml
listen_addresses: "*"

max_connections: 300
```

---

# Handler

```text
Cambios

↓

notify

↓

Restart PostgreSQL
```

---

# Creación de Usuarios

Utilizando.

```text
community.postgresql
```

Collection oficial.

---

# Ejemplo

```yaml
community.postgresql.postgresql_user
```

---

# Creación de Base de Datos

```yaml
community.postgresql.postgresql_db
```

---

# Beneficios

- Automatización.
- Consistencia.
- Seguridad.
- Rapidez.

---

# Role Backup

Toda infraestructura necesita respaldos.

---

# Objetivos

- Crear directorios.
- Crear scripts.
- Programar cron.
- Rotación.
- Compresión.
- Limpieza.

---

# Arquitectura

```text
Backup

↓

Script

↓

Cron

↓

Repositorio
```

---

# Variables

```yaml
backup_path:

/backup

backup_retention:

7
```

---

# Script

Template.

```text
backup.sh.j2
```

---

# Cron

```yaml
cron:

  minute: "0"

  hour: "2"
```

---

Resultado.

```text
02:00

↓

Backup Diario
```

---

# Rotación

Eliminar.

```text
Archivos

>

7 días
```

---

# Handler

Después de modificar el cron.

```text
Recargar Cron
```

---

# Role Monitoring

El último componente será el monitoreo.

---

# Responsabilidades

- Instalar Node Exporter.
- Instalar PostgreSQL Exporter.
- Configurar Logs.
- Configurar Métricas.
- Configurar Alertas.

---

# Arquitectura

```text
Servidor

↓

Exporters

↓

Prometheus

↓

Grafana
```

---

# Variables

```yaml
node_exporter_port:

9100

postgres_exporter_port:

9187
```

---

# Instalación

Puede realizarse mediante.

- RPM.
- Binarios.
- Collections.

---

# Validación

```bash
curl localhost:9100/metrics
```

---

También.

```bash
curl localhost:9187/metrics
```

---

# Templates

El Role Monitoring utilizará.

```text
prometheus.yml.j2

node_exporter.service.j2

postgres_exporter.service.j2
```

---

# Uso de Handlers

Cada modificación importante debe utilizar.

```yaml
notify
```

---

Nunca.

```yaml
service:

state: restarted
```

Después de cada Task.

---

# Flujo

```text
Task

↓

Template

↓

notify

↓

Handler

↓

Restart
```

---

# Integración con Vault

Las siguientes Variables nunca deben almacenarse en texto plano.

- Contraseñas PostgreSQL.
- Password SSH.
- API Keys.
- Certificados.
- Tokens.
- Password Grafana.

---

Ejemplo.

```text
group_vars/

vault.yml
```

---

Contenido.

```yaml
postgres_password:

!vault

$ANSIBLE_VAULT...
```

---

# Uso de Collections

Este proyecto utilizará.

```yaml
collections:

- ansible.posix

- community.general

- community.postgresql
```

---

# Integración Completa

```text
Collections

↓

Roles

↓

Templates

↓

Handlers

↓

Vault

↓

Playbooks

↓

Infraestructura
```

---

# Playbook Principal

```yaml
- hosts: webservers

  roles:

    - apache

- hosts: databases

  roles:

    - postgres

- hosts: backups

  roles:

    - backup

- hosts: all

  roles:

    - monitoring
```

---

# Arquitectura Completa

```text
                    site.yml

                        │

      ┌─────────────────┼────────────────┐

      ▼                 ▼                ▼

   Apache           PostgreSQL        Backup

      │                 │                │

      └─────────────────┼────────────────┘

                        ▼

                  Monitoring

                        │

                        ▼

                  Producción
```

---

# Organización Empresarial

```text
Git

↓

Commit

↓

Pull Request

↓

Review

↓

CI/CD

↓

Laboratorio

↓

Producción
```

---

# Validaciones

Después de cada despliegue.

Debe verificarse.

- Apache.
- PostgreSQL.
- Firewall.
- Exporters.
- Cron.
- Logs.
- Respaldos.

---

# Checklist

```text
□ Apache instalado

□ PostgreSQL iniciado

□ Handler ejecutado

□ Backup programado

□ Monitoring activo

□ Firewall correcto

□ Vault utilizado

□ Collections instaladas
```

---

# Buenas Prácticas

- Mantener cada servicio en un Role independiente.
- Utilizar Templates para todos los archivos de configuración.
- Reiniciar servicios únicamente mediante Handlers.
- Mantener todos los secretos en Vault.
- Utilizar módulos específicos antes que `shell` o `command`.
- Validar cada servicio inmediatamente después de instalarlo.
- Automatizar completamente los respaldos.
- Supervisar permanentemente los servicios mediante Prometheus y Grafana.
- Mantener Collections actualizadas.
- Versionar todos los cambios mediante Git.

---

# Errores Comunes

## Error 1

Modificar manualmente archivos administrados por Templates.

---

## Error 2

Guardar contraseñas PostgreSQL en texto plano.

---

## Error 3

No utilizar Handlers.

---

## Error 4

No validar que Apache realmente inició.

---

## Error 5

No comprobar la conectividad a PostgreSQL.

---

## Error 6

No verificar la ejecución de los respaldos.

---

## Error 7

Olvidar instalar las Collections requeridas.

---

## Error 8

No cifrar Variables sensibles mediante Vault.

---

## Error 9

Reiniciar servicios innecesariamente en cada ejecución del Playbook.

---

## Error 10

No comprobar que las métricas sean visibles desde Prometheus.

---

# Laboratorio RHCSA

## Escenario

La empresa debe desplegar completamente su plataforma en una nueva sucursal utilizando únicamente Ansible.

---

## Laboratorio 1

Implementar el Role `apache` instalando el servidor HTTP, configurando un Template para `httpd.conf` y utilizando un Handler para reiniciar el servicio únicamente cuando existan cambios.

---

## Laboratorio 2

Implementar el Role `postgres` instalando PostgreSQL, inicializando el clúster y configurando `postgresql.conf` y `pg_hba.conf` mediante Templates.

---

## Laboratorio 3

Crear automáticamente una Base de Datos y un usuario utilizando la Collection `community.postgresql`.

---

## Laboratorio 4

Implementar el Role `backup`, generando un script mediante un Template y programando un respaldo diario utilizando el módulo `cron`.

---

## Laboratorio 5

Agregar una política de rotación que elimine automáticamente respaldos con más de siete días de antigüedad.

---

## Laboratorio 6

Implementar el Role `monitoring` instalando Node Exporter y verificando que el puerto `9100` publique métricas correctamente.

---

## Laboratorio 7

Proteger todas las credenciales de PostgreSQL y Grafana utilizando Ansible Vault.

---

## Laboratorio 8

Integrar todos los Roles en el archivo `site.yml` y ejecutar el despliegue completo sobre la infraestructura del laboratorio.

---

## Laboratorio 9

Validar el funcionamiento de Apache, PostgreSQL, los respaldos programados y las métricas publicadas por los Exporters.

---

## Laboratorio 10

Documentar completamente el procedimiento de despliegue, incluyendo dependencias, Variables, Templates, Handlers, Collections utilizadas y pasos de validación posteriores a la ejecución.

---

# Preguntas de Repaso

1. ¿Por qué Apache y PostgreSQL deben implementarse en Roles independientes?
2. ¿Qué ventajas ofrecen los Handlers frente a reiniciar servicios en cada tarea?
3. ¿Por qué es recomendable utilizar Templates para los archivos de configuración?
4. ¿Qué información debe protegerse mediante Vault?
5. ¿Qué beneficios aporta la Collection `community.postgresql`?
6. ¿Cómo automatizarías la política de respaldos?
7. ¿Qué métricas son importantes para monitorear un servidor Linux?
8. ¿Por qué validar el estado de los servicios después del despliegue?
9. ¿Cómo contribuye Git al mantenimiento del proyecto?
10. ¿Qué controles implementarías antes de ejecutar este Playbook en producción?

---

# Resumen

En esta tercera fase desarrollamos la capa de servicios del laboratorio final. Implementamos los Roles **apache**, **postgres**, **backup** y **monitoring**, integrando **Handlers**, **Templates**, **Collections** y **Ansible Vault** para construir una infraestructura completamente automatizada, segura y preparada para entornos empresariales.

En la **Fase 4** realizaremos el despliegue completo de la infraestructura, integraremos procesos de **Troubleshooting**, validaciones posteriores al despliegue, recuperación ante fallos, CI/CD, documentación operativa y un proyecto final que reunirá todos los conocimientos adquiridos a lo largo del módulo de Ansible.

-------

# 89. Laboratorio Final de Automatización con Ansible (Fase 4)

> **Módulo 11 – Automatización con Ansible**
>
> **Archivo:** `89-laboratorio-ansible.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Ejecutar un despliegue completo utilizando Ansible.
- Validar automáticamente todos los servicios.
- Implementar procedimientos de rollback.
- Integrar Ansible con un flujo de CI/CD.
- Aplicar metodologías profesionales de Troubleshooting.
- Documentar completamente una infraestructura automatizada.
- Construir un proyecto empresarial listo para Producción.

---

# Introducción

Después de desarrollar los Roles:

- common
- users
- security
- firewall
- apache
- postgres
- backup
- monitoring

ha llegado el momento de realizar el despliegue completo de la infraestructura.

Esta fase representa la forma en que trabajan los equipos DevOps y de Infraestructura en empresas reales.

---

# Arquitectura Final

```text
                           Git Repository

                                 │

                                 ▼

                          Pull Request

                                 │

                                 ▼

                           CI/CD Pipeline

                                 │

                                 ▼

                      Validación Automática

                                 │

                                 ▼

                        Ansible Controller

                                 │

        ┌────────────────────────┼─────────────────────────┐

        ▼                        ▼                         ▼

   Web Servers             PostgreSQL                 Backup Server

        │                        │                         │

        └────────────────────────┼─────────────────────────┘

                                 ▼

                         Monitoring Stack

                                 │

                                 ▼

                            Producción
```

---

# Flujo Empresarial

```text
Administrador

↓

Git Commit

↓

Pull Request

↓

Revisión

↓

Merge

↓

Pipeline CI/CD

↓

Ansible

↓

Validación

↓

Producción
```

---

# Despliegue Completo

El despliegue debe ejecutarse mediante un único Playbook.

```bash
ansible-playbook \
-i inventories/production/hosts.yml \
playbooks/site.yml
```

---

# Orden de Ejecución

```text
Inventario

↓

Variables

↓

Collections

↓

Roles

↓

Handlers

↓

Validaciones

↓

Reporte Final
```

---

# Checklist Antes del Despliegue

Antes de ejecutar el Playbook se debe verificar:

```text
□ Inventario actualizado

□ Variables revisadas

□ Vault disponible

□ Collections instaladas

□ Roles validados

□ Templates revisados

□ Espacio suficiente en disco

□ Acceso SSH funcionando

□ Firewall configurado

□ Copia de respaldo disponible
```

---

# Validaciones Posteriores

Después del despliegue no basta con que el Playbook termine correctamente.

También deben verificarse los servicios.

---

## Apache

```bash
systemctl status httpd
```

---

```bash
curl http://localhost
```

---

## PostgreSQL

```bash
systemctl status postgresql
```

---

```bash
psql -c "SELECT version();"
```

---

## Firewall

```bash
firewall-cmd --list-services
```

---

## SELinux

```bash
getenforce
```

Resultado esperado.

```text
Enforcing
```

---

## Cron

```bash
crontab -l
```

---

## Monitoring

```bash
curl localhost:9100/metrics
```

---

# Validación Automatizada

Puede utilizarse un Playbook exclusivo para validaciones.

```text
deploy.yml

↓

validate.yml
```

---

Arquitectura.

```text
Despliegue

↓

Validación

↓

Reporte

↓

Producción
```

---

# Rollback

Todo proyecto empresarial debe disponer de un procedimiento de recuperación.

Nunca debe asumirse que un despliegue será exitoso.

---

# Estrategia

```text
Nueva Configuración

↓

Error

↓

Rollback

↓

Configuración Anterior
```

---

# ¿Qué puede restaurarse?

- Templates
- Configuración Apache
- Configuración PostgreSQL
- Variables
- Archivos de servicio
- Scripts
- Certificados

---

# Estrategia Recomendada

Antes del despliegue.

```text
Backup

↓

Cambios

↓

Validación

↓

Rollback (si es necesario)
```

---

# Automatización del Rollback

Puede implementarse mediante.

```text
block

↓

rescue

↓

always
```

---

Ejemplo conceptual.

```yaml
block:

  - Desplegar

rescue:

  - Restaurar respaldo

always:

  - Registrar incidente
```

---

# Validación de Idempotencia

Un Playbook profesional debe poder ejecutarse varias veces.

```text
Ejecución 1

↓

Cambios

↓

Ejecución 2

↓

Sin cambios

↓

Idempotencia
```

---

# Integración con Git

Toda modificación debe seguir este flujo.

```text
Branch

↓

Commit

↓

Push

↓

Pull Request

↓

Review

↓

Merge
```

---

# Integración con CI/CD

Ejemplo.

```text
GitHub

↓

GitHub Actions

↓

Syntax Check

↓

Lint

↓

Molecule

↓

Deploy

↓

Validate
```

---

# Pipeline Empresarial

```text
Developer

↓

Git Push

↓

CI/CD

↓

Ansible

↓

Producción
```

---

# Validaciones del Pipeline

Cada ejecución debería comprobar.

```text
YAML

↓

Sintaxis

↓

Roles

↓

Templates

↓

Variables

↓

Collections

↓

Playbooks
```

---

# Reporte Final

Todo despliegue debe generar un reporte.

Ejemplo.

| Elemento | Estado |
|----------|--------|
| Inventario | OK |
| Apache | OK |
| PostgreSQL | OK |
| Firewall | OK |
| Backup | OK |
| Monitoring | OK |
| Validaciones | OK |

---

# Documentación Operativa

Todo proyecto debe incluir.

```text
README.md
```

---

Debe contener.

- Objetivo.
- Arquitectura.
- Inventarios.
- Variables.
- Vault.
- Roles.
- Collections.
- Procedimiento de despliegue.
- Procedimiento de rollback.
- Procedimiento de validación.
- Solución de problemas.

---

# Base de Conocimiento

Toda incidencia debe registrarse.

| Campo | Descripción |
|--------|-------------|
| Fecha | Momento del incidente |
| Servidor | Equipo afectado |
| Error | Mensaje observado |
| Causa | Diagnóstico |
| Solución | Acción aplicada |
| Responsable | Administrador |
| Validación | Evidencia |

---

# Troubleshooting Integrado

Cuando ocurra un error.

```text
Error

↓

Verbosidad

↓

debug

↓

register

↓

Diagnóstico

↓

Corrección

↓

Nueva Validación
```

---

# Monitoreo Continuo

Después del despliegue.

```text
Prometheus

↓

Alertmanager

↓

Grafana

↓

Administrador
```

---

# Flujo Completo

```text
Git

↓

CI/CD

↓

Ansible

↓

Roles

↓

Handlers

↓

Servicios

↓

Validaciones

↓

Producción

↓

Monitoreo
```

---

# Proyecto Empresarial Final

La empresa solicita automatizar completamente una infraestructura compuesta por:

| Servidor | Función |
|----------|---------|
| bastion01 | Nodo de administración |
| web01 | Apache |
| web02 | Apache |
| web03 | Apache |
| db01 | PostgreSQL Primario |
| db02 | PostgreSQL Réplica |
| backup01 | Servidor de respaldos |
| monitor01 | Prometheus y Grafana |

---

Todos los servidores deberán quedar configurados automáticamente.

---

# Requisitos

El proyecto debe:

- Utilizar Inventarios.
- Utilizar Variables.
- Utilizar Roles.
- Utilizar Templates.
- Utilizar Handlers.
- Utilizar Vault.
- Utilizar Collections.
- Utilizar Troubleshooting.
- Ser completamente idempotente.
- Estar documentado.

---

# Arquitectura Final del Proyecto

```text
empresa-ansible/

├── ansible.cfg

├── inventories/

│   ├── production

│   └── development

├── group_vars/

├── host_vars/

├── playbooks/

│   ├── site.yml

│   ├── deploy.yml

│   ├── validate.yml

│   └── rollback.yml

├── roles/

│   ├── common

│   ├── users

│   ├── security

│   ├── firewall

│   ├── apache

│   ├── postgres

│   ├── backup

│   └── monitoring

├── templates/

├── files/

├── collections/

├── molecule/

├── tests/

├── README.md

└── requirements.yml
```

---

# Buenas Prácticas

- Automatizar absolutamente todo.
- Evitar configuraciones manuales.
- Mantener un único origen de la verdad.
- Validar cada despliegue.
- Implementar rollback.
- Documentar todas las decisiones técnicas.
- Utilizar nombres consistentes.
- Mantener Roles pequeños y reutilizables.
- Ejecutar pruebas antes de Producción.
- Versionar absolutamente todo mediante Git.

---

# Errores Comunes

## Error 1

Desplegar directamente sobre Producción sin pruebas.

---

## Error 2

No disponer de un procedimiento de rollback.

---

## Error 3

No validar los servicios después del despliegue.

---

## Error 4

Guardar secretos fuera de Vault.

---

## Error 5

Modificar manualmente servidores administrados por Ansible.

---

## Error 6

No documentar la infraestructura.

---

## Error 7

No comprobar la idempotencia.

---

## Error 8

No integrar el proyecto con Git.

---

## Error 9

No ejecutar validaciones automáticas.

---

## Error 10

No mantener una base de conocimiento de incidencias.

---

# Laboratorio RHCSA

## Escenario

La empresa debe desplegar una nueva plataforma corporativa durante una ventana de mantenimiento de dos horas.

---

## Laboratorio 1

Ejecutar el despliegue completo utilizando el Playbook `site.yml` sobre el inventario de producción.

---

## Laboratorio 2

Implementar un Playbook `validate.yml` que compruebe automáticamente el estado de Apache, PostgreSQL, Firewall y Monitoring.

---

## Laboratorio 3

Diseñar un procedimiento de rollback utilizando `block`, `rescue` y `always`.

---

## Laboratorio 4

Validar que el proyecto sea completamente idempotente ejecutando el Playbook dos veces consecutivas.

---

## Laboratorio 5

Integrar Ansible con un repositorio Git y documentar el flujo de trabajo mediante Pull Requests.

---

## Laboratorio 6

Diseñar un pipeline de CI/CD que ejecute validaciones de sintaxis y pruebas antes del despliegue.

---

## Laboratorio 7

Crear un reporte final del despliegue indicando el estado de todos los servicios administrados.

---

## Laboratorio 8

Registrar un incidente ficticio en la base de conocimiento, incluyendo causa raíz, solución y validación.

---

## Laboratorio 9

Realizar una recuperación completa utilizando el procedimiento de rollback diseñado previamente y comprobar que todos los servicios vuelvan a su estado operativo.

---

## Laboratorio 10 (Proyecto Final RHCSA)

Construir una infraestructura completamente automatizada que integre:

- Inventarios.
- Variables.
- Facts.
- Roles.
- Templates.
- Handlers.
- Vault.
- Galaxy.
- Collections.
- Troubleshooting.
- Validaciones.
- Rollback.
- Monitoreo.
- Documentación.
- CI/CD.

Presentar la solución siguiendo las mejores prácticas utilizadas por equipos profesionales de administración Linux.

---

# Preguntas de Repaso

1. ¿Por qué un despliegue exitoso no garantiza que los servicios funcionen correctamente?
2. ¿Qué ventajas aporta un procedimiento de rollback?
3. ¿Por qué es importante validar la idempotencia?
4. ¿Qué información debe contener un reporte de despliegue?
5. ¿Cómo ayuda Git al control de cambios en Ansible?
6. ¿Qué pruebas debería ejecutar un pipeline de CI/CD antes del despliegue?
7. ¿Por qué es recomendable documentar todas las incidencias?
8. ¿Qué beneficios aporta el monitoreo continuo después del despliegue?
9. ¿Cómo se relacionan Ansible, Git y CI/CD dentro de una estrategia DevOps?
10. ¿Qué características debe cumplir un proyecto empresarial para considerarse listo para Producción?

---

# Resumen del Capítulo

En esta fase completamos el laboratorio final del módulo integrando todos los conocimientos adquiridos durante el curso. Diseñamos un flujo empresarial completo que abarca el despliegue automatizado, las validaciones posteriores, los procedimientos de rollback, la documentación operativa, el monitoreo continuo y la integración con Git y CI/CD.

Con este proyecto final quedó consolidado un entorno profesional basado en Ansible, preparado para administrar infraestructuras Linux de forma segura, repetible, escalable e idempotente.

---

# Fin del Módulo 11 — Automatización con Ansible

**Competencias alcanzadas:**

- Administración de Inventarios.
- Variables y Facts.
- Playbooks avanzados.
- Roles reutilizables.
- Templates Jinja2.
- Handlers.
- Ansible Vault.
- Galaxy y Collections.
- Troubleshooting profesional.
- Automatización empresarial.
- Integración con Git y CI/CD.
- Diseño de infraestructuras automatizadas listas para Producción.

**Próximo módulo del roadmap:**

# **Módulo 12 — Bash Scripting para Administradores Linux**

En el siguiente módulo aprenderás a desarrollar scripts profesionales para automatizar tareas administrativas, manipular archivos y procesos, programar tareas con `cron`, crear funciones reutilizables, gestionar errores y construir herramientas de automatización que complementen el uso de Ansible en entornos empresariales.

----







