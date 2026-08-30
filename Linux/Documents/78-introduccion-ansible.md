# 78. Introducción a Ansible (Fase 1)

> **Módulo 11 – Automatización con Ansible**
>
> **Manual RHCSA**
>
> **Archivo:** `78-introduccion-ansible.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Comprender qué es Ansible.
- Entender por qué es una de las herramientas de automatización más utilizadas en Linux.
- Conocer la arquitectura de Ansible.
- Identificar sus componentes principales.
- Comprender cómo funciona la comunicación mediante SSH.
- Diferenciar Ansible de otras herramientas de automatización.
- Prepararte para comenzar a automatizar servidores Linux.

---

# ¿Qué es Ansible?

Ansible es una herramienta de automatización de infraestructura desarrollada originalmente por Michael DeHaan y actualmente mantenida por **Red Hat**.

Su objetivo principal es automatizar tareas repetitivas relacionadas con:

- Administración de servidores.
- Configuración de sistemas.
- Instalación de software.
- Gestión de usuarios.
- Despliegues de aplicaciones.
- Configuración de redes.
- Automatización de nubes públicas y privadas.
- Orquestación de múltiples servidores.

---

# Automatización

Antes de Ansible, un administrador debía conectarse servidor por servidor.

Ejemplo:

```text
Servidor1

↓

SSH

↓

Actualizar

↓

Salir

↓

Servidor2

↓

SSH

↓

Actualizar

↓

Servidor3

↓

SSH
```

Cuando existen cientos de servidores, este procedimiento se vuelve lento y propenso a errores.

---

# Con Ansible

```text
               Administrador

                     │

                     ▼

                 Ansible

     ┌────────────┼────────────┐

     ▼            ▼            ▼

Servidor1     Servidor2    Servidor3

     ▼            ▼            ▼

 Ejecuta      Ejecuta      Ejecuta

 exactamente la misma tarea
```

Un solo comando puede administrar cientos o miles de servidores simultáneamente.

---

# ¿Qué problemas resuelve?

Ansible ayuda a eliminar tareas repetitivas como:

- Crear usuarios.
- Instalar paquetes.
- Reiniciar servicios.
- Configurar Apache.
- Configurar Nginx.
- Configurar PostgreSQL.
- Configurar SQL Server.
- Actualizar servidores.
- Copiar archivos.
- Cambiar permisos.
- Modificar archivos de configuración.

---

# Administración Tradicional

```text
Servidor 1

↓

SSH

↓

Cambios

↓

Servidor 2

↓

SSH

↓

Cambios

↓

Servidor 3

↓

SSH
```

Tiempo:

Muchos minutos u horas.

---

# Administración con Ansible

```text
Administrador

↓

Playbook

↓

Ansible

↓

100 Servidores

↓

Todos quedan exactamente iguales
```

Tiempo:

Segundos o pocos minutos.

---

# ¿Qué significa Idempotencia?

Uno de los conceptos más importantes de Ansible es la **idempotencia**.

Significa que una tarea puede ejecutarse múltiples veces sin producir efectos secundarios innecesarios.

Ejemplo:

Instalar Apache.

Primera ejecución

```text
Apache

NO existe

↓

Se instala
```

Segunda ejecución

```text
Apache

YA existe

↓

No hace nada
```

El resultado siempre será el mismo.

---

# Características principales

- Open Source.
- Fácil de aprender.
- Basado en YAML.
- Utiliza SSH.
- No requiere agentes.
- Escalable.
- Multiplataforma.
- Idempotente.
- Modular.
- Extensible.

---

# ¿Qué significa "Agentless"?

Muchas herramientas requieren instalar un programa en cada servidor.

Ejemplo:

```text
Servidor

↓

Instalar Agente

↓

Configurar Agente

↓

Mantener Agente
```

Con Ansible esto no es necesario.

---

# Arquitectura Agentless

```text
Control Node

↓

SSH

↓

Managed Node

↓

Ejecuta

↓

Termina
```

No queda ningún servicio ejecutándose.

---

# Componentes de Ansible

```text
Control Node

↓

Inventory

↓

Playbooks

↓

Modules

↓

Managed Nodes
```

---

# Control Node

Es el servidor donde está instalado Ansible.

Desde aquí se ejecutan todas las automatizaciones.

Puede ser:

- Fedora
- RHEL
- Rocky Linux
- AlmaLinux
- Ubuntu

---

# Managed Nodes

Son los servidores administrados.

Ejemplos

```text
Servidor Web

Servidor SQL

Servidor PostgreSQL

Servidor DNS

Servidor Backup

Servidor Docker

Servidor Podman
```

---

# Inventory

Es la lista de servidores administrados.

Ejemplo

```text
web01

web02

db01

db02

backup01
```

---

# Playbooks

Son archivos escritos en YAML.

Describen exactamente lo que Ansible debe hacer.

Ejemplo

```text
Instalar Apache

↓

Copiar configuración

↓

Iniciar servicio

↓

Verificar estado
```

---

# Modules

Los módulos son pequeñas herramientas incorporadas en Ansible.

Ejemplos

```text
copy

service

dnf

yum

user

group

package

template

file

ping
```

---

# Plugins

Los plugins amplían las capacidades de Ansible.

Ejemplos:

- Callback Plugins
- Cache Plugins
- Connection Plugins
- Lookup Plugins
- Filter Plugins

---

# Collections

Las Collections agrupan:

- Roles
- Plugins
- Modules
- Documentación

Ejemplos

```text
community.general

ansible.posix

containers.podman

kubernetes.core
```

---

# Flujo de Trabajo

```text
Administrador

↓

Playbook

↓

Inventory

↓

SSH

↓

Servidor

↓

Módulo

↓

Resultado

↓

Reporte
```

---

# Comunicación

Ansible utiliza principalmente:

```text
SSH

Puerto 22
```

No necesita:

- Agentes.
- Daemons.
- Servicios adicionales.

---

# Arquitectura Completa

```text
                 Administrador

                      │

                      ▼

               Control Node

                      │

          ┌───────────┼───────────┐

          ▼           ▼           ▼

      Inventory   Playbook     Modules

                      │

                      ▼

                     SSH

                      │

        ┌─────────────┼─────────────┐

        ▼             ▼             ▼

     Web01         DB01         Backup01
```

---

# Casos de Uso

Ansible puede utilizarse para:

- Administración Linux.
- Automatización DevOps.
- Cloud Computing.
- Kubernetes.
- Docker.
- Podman.
- VMware.
- AWS.
- Azure.
- Google Cloud.
- Redes Cisco.
- Balanceadores.
- Bases de datos.

---

# Comparación

| Herramienta | Requiere Agente | Lenguaje |
|-------------|-----------------|----------|
| Ansible | No | YAML |
| Puppet | Sí | DSL |
| Chef | Sí | Ruby |
| Salt | Generalmente Sí | YAML/Python |

---

# Ventajas

- Muy sencillo.
- Poco consumo.
- Fácil mantenimiento.
- Excelente documentación.
- Gran comunidad.
- Integración con Red Hat.
- Muy utilizado en empresas.

---

# Limitaciones

- SSH debe funcionar correctamente.
- No es la mejor opción para automatización en tiempo real.
- En infraestructuras extremadamente grandes puede requerir optimización del inventario y la ejecución paralela.

---

# Caso Empresarial

Supongamos una empresa con:

```text
25 Web Servers

15 Database Servers

10 DNS Servers

20 Application Servers

30 Backup Servers
```

Total

```text
100 servidores
```

Actualizar todos manualmente podría tomar varias horas.

Con Ansible:

```text
Administrador

↓

Playbook

↓

100 servidores

↓

Actualizados
```

---

# ¿Por qué Red Hat utiliza Ansible?

Red Hat adquirió Ansible en 2015 debido a su simplicidad y adopción masiva.

Actualmente forma parte del ecosistema empresarial junto con:

- Red Hat Enterprise Linux.
- Red Hat Satellite.
- Red Hat OpenShift.
- Red Hat Ansible Automation Platform.

---

# Laboratorio RHCSA

## Laboratorio 1

Investigar la diferencia entre automatización y orquestación.

---

## Laboratorio 2

Identificar cinco tareas repetitivas que realizas normalmente como administrador Linux.

---

## Laboratorio 3

Diseñar un diagrama con:

- Control Node.
- Tres Managed Nodes.
- Comunicación SSH.

---

## Laboratorio 4

Investigar qué significa infraestructura como código (Infrastructure as Code).

---

## Laboratorio 5

Comparar Ansible con Puppet.

---

## Laboratorio 6

Comparar Ansible con Chef.

---

## Laboratorio 7

Investigar qué es YAML.

---

## Laboratorio 8

Buscar cinco módulos populares de Ansible.

---

## Laboratorio 9

Explicar qué es un Inventory.

---

## Laboratorio 10

Explicar qué es un Playbook.

---

# Buenas prácticas

- Automatizar tareas repetitivas.
- Utilizar SSH con autenticación por claves.
- Mantener los Playbooks simples y legibles.
- Documentar cada automatización.
- Probar siempre en un entorno de laboratorio antes de producción.
- Organizar los inventarios por grupos de servidores.

---

# Errores comunes

## Error 1

Pensar que Ansible instala un agente en cada servidor.

---

## Error 2

Ejecutar automatizaciones directamente en producción sin pruebas previas.

---

## Error 3

No documentar los Playbooks.

---

## Error 4

Utilizar un único inventario para todos los ambientes (Desarrollo, QA y Producción).

---

## Error 5

Automatizar procesos sin comprender primero cómo se realizan manualmente.

---

# Resumen

En esta primera fase aprendimos:

- Qué es Ansible.
- Cómo funciona su arquitectura.
- Qué es un Control Node.
- Qué son los Managed Nodes.
- Qué es un Inventory.
- Qué es un Playbook.
- Qué son los módulos.
- Cómo utiliza SSH para administrar servidores.
- Qué significa la idempotencia.
- Por qué Ansible es una de las herramientas de automatización más importantes en entornos Linux y Red Hat.

En la **Fase 2** instalaremos Ansible en Fedora, configuraremos la autenticación mediante SSH, crearemos el primer inventario y ejecutaremos nuestros primeros comandos contra uno o varios servidores administrados.
-----

# 78. Introducción a Ansible (Fase 2)

> **Módulo 11 – Automatización con Ansible**
>
> **Archivo:** `78-introduccion-ansible.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Instalar Ansible en Fedora, RHEL y derivados.
- Verificar la instalación.
- Comprender la estructura de directorios de Ansible.
- Configurar la autenticación mediante SSH.
- Crear el primer Inventory.
- Organizar servidores en grupos.
- Ejecutar comandos Ad-Hoc.
- Comprender el flujo completo entre Control Node y Managed Nodes.

---

# Requisitos Previos

Antes de instalar Ansible debemos contar con:

- Fedora / RHEL / Rocky Linux / AlmaLinux
- Acceso Root o sudo
- Conectividad SSH
- DNS o IP de los servidores
- Python instalado en los servidores administrados

---

# Arquitectura de Laboratorio

Durante este módulo utilizaremos el siguiente laboratorio.

```text
                   Fedora

               Control Node
               192.168.100.10

                     │
          SSH         │        SSH

        ┌─────────────┼─────────────┐

        ▼             ▼             ▼

   web01          db01          backup01

192.168.100.20 192.168.100.30 192.168.100.40
```

---

# Instalación en Fedora

Actualizar el sistema

```bash
sudo dnf update -y
```

Instalar Ansible

```bash
sudo dnf install ansible -y
```

---

# Verificar instalación

```bash
ansible --version
```

Salida esperada

```text
ansible [core 2.x]

python version ...

config file = ...

module search path ...

collection location ...
```

---

# Información detallada

```bash
ansible --version
```

Ejemplo

```text
ansible-core 2.18

python 3.12

jinja 3.x

libyaml=True
```

---

# Ubicación del ejecutable

```bash
which ansible
```

Resultado

```text
/usr/bin/ansible
```

---

# Verificar paquetes

```bash
rpm -qa | grep ansible
```

---

# Ayuda

```bash
ansible --help
```

---

# Componentes Instalados

Después de instalar tendremos herramientas como:

```text
ansible

ansible-playbook

ansible-doc

ansible-config

ansible-console

ansible-galaxy

ansible-vault

ansible-inventory
```

---

# ansible

Permite ejecutar comandos Ad-Hoc.

Ejemplo

```bash
ansible all -m ping
```

---

# ansible-playbook

Ejecuta Playbooks.

```bash
ansible-playbook sitio.yml
```

---

# ansible-doc

Muestra documentación de módulos.

```bash
ansible-doc copy
```

---

# ansible-galaxy

Gestiona Roles y Collections.

```bash
ansible-galaxy collection list
```

---

# ansible-vault

Protege información sensible.

```bash
ansible-vault create secretos.yml
```

---

# ansible-config

Permite consultar la configuración.

```bash
ansible-config list
```

---

# ansible-inventory

Permite visualizar el inventario.

```bash
ansible-inventory --list
```

---

# ¿Cómo trabaja Ansible?

```text
Administrador

↓

Playbook

↓

Inventory

↓

SSH

↓

Servidor Linux

↓

Python

↓

Módulo

↓

Resultado
```

---

# ¿Por qué necesita Python?

Ansible copia temporalmente pequeños módulos escritos en Python.

```text
Control Node

↓

SSH

↓

Servidor

↓

Ejecuta Python

↓

Devuelve resultado

↓

Elimina archivos temporales
```

No deja agentes instalados.

---

# Comunicación SSH

La comunicación utiliza normalmente

```text
TCP 22
```

No requiere abrir puertos adicionales.

---

# Autenticación

Existen dos métodos.

## Usuario y contraseña

```text
Administrador

↓

SSH

↓

Password
```

---

## Claves SSH (Recomendado)

```text
Administrador

↓

Llave Privada

↓

SSH

↓

Llave Pública

↓

Servidor
```

Es el método utilizado en producción.

---

# Crear claves SSH

Generar una llave

```bash
ssh-keygen
```

Resultado

```text
~/.ssh/id_ed25519

~/.ssh/id_ed25519.pub
```

También puede utilizarse RSA

```bash
ssh-keygen -t rsa -b 4096
```

---

# Copiar la llave pública

```bash
ssh-copy-id usuario@192.168.100.20
```

---

# Verificar acceso

```bash
ssh usuario@192.168.100.20
```

No debería solicitar contraseña.

---

# Directorio .ssh

```text
~/.ssh

├── config
├── known_hosts
├── id_ed25519
├── id_ed25519.pub
└── authorized_keys
```

---

# Inventory

El Inventory es la lista de servidores administrados.

Ejemplo

```text
web01

db01

backup01
```

---

# Inventory Estático

Archivo

```text
inventory.ini
```

Contenido

```ini
web01 ansible_host=192.168.100.20

db01 ansible_host=192.168.100.30

backup01 ansible_host=192.168.100.40
```

---

# Organizar por grupos

```ini
[web]

web01

web02

web03

[database]

db01

db02

[backup]

backup01
```

---

# Ejemplo Empresarial

```ini
[apache]

web01

web02

web03

web04

[mysql]

db01

db02

db03

[monitoring]

zabbix01

grafana01
```

---

# Variables del Inventory

Ejemplo

```ini
web01 ansible_host=192.168.100.20 ansible_user=ajimenez
```

---

# Definir usuario

```ini
ansible_user=root
```

---

# Definir puerto SSH

```ini
ansible_port=2222
```

---

# Definir llave privada

```ini
ansible_ssh_private_key_file=~/.ssh/id_ed25519
```

---

# Ejemplo completo

```ini
web01

ansible_host=192.168.100.20

ansible_user=admin

ansible_port=22

ansible_ssh_private_key_file=~/.ssh/id_ed25519
```

---

# Validar Inventory

```bash
ansible-inventory -i inventory.ini --graph
```

Ejemplo

```text
@all

|--@ungrouped

|--@web

| |--web01

| |--web02

|--@database

| |--db01
```

---

# Visualizar el Inventory

```bash
ansible-inventory -i inventory.ini --list
```

---

# Primer comando Ad-Hoc

Comprobar conectividad.

```bash
ansible all -i inventory.ini -m ping
```

Salida

```text
web01

SUCCESS

db01

SUCCESS

backup01

SUCCESS
```

---

# ¿Qué hace el módulo ping?

No envía paquetes ICMP.

Lo que realmente hace es:

- Conectarse mediante SSH.
- Ejecutar Python.
- Ejecutar el módulo.
- Devolver "pong".

---

# Ejecutar un comando

```bash
ansible all -m command -a "hostname"
```

Resultado

```text
web01

web01

db01

db01

backup01

backup01
```

---

# Obtener la fecha

```bash
ansible all -m command -a "date"
```

---

# Ver Kernel

```bash
ansible all -m command -a "uname -r"
```

---

# Espacio en disco

```bash
ansible all -m command -a "df -h"
```

---

# Uso de Memoria

```bash
ansible all -m command -a "free -m"
```

---

# Tiempo de actividad

```bash
ansible all -m command -a "uptime"
```

---

# Ejecutar sólo un grupo

```bash
ansible web -m ping
```

---

# Ejecutar un solo servidor

```bash
ansible web01 -m ping
```

---

# Flujo completo

```text
Administrador

↓

ansible

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

# Buenas prácticas

- Utilizar autenticación mediante llaves SSH.
- Mantener un Inventory organizado.
- Agrupar servidores por función.
- Asignar nombres descriptivos.
- Validar siempre la conectividad antes de ejecutar automatizaciones.
- Evitar trabajar como root cuando no sea necesario.
- Utilizar usuarios administrativos con sudo.

---

# Errores comunes

## Error 1

No copiar correctamente la llave pública.

---

## Error 2

Firewall bloqueando el puerto 22.

---

## Error 3

Servidor sin Python instalado.

---

## Error 4

Dirección IP incorrecta en el Inventory.

---

## Error 5

Permisos incorrectos sobre la llave privada.

```bash
chmod 600 ~/.ssh/id_ed25519
```

---

## Error 6

No verificar el acceso SSH antes de utilizar Ansible.

---

# Laboratorio RHCSA

## Laboratorio 1

Instalar Ansible en Fedora.

---

## Laboratorio 2

Verificar la versión instalada.

---

## Laboratorio 3

Crear un par de llaves SSH.

---

## Laboratorio 4

Copiar la llave pública a un servidor remoto.

---

## Laboratorio 5

Comprobar acceso sin contraseña.

---

## Laboratorio 6

Crear un Inventory con tres servidores.

---

## Laboratorio 7

Agrupar servidores por función.

---

## Laboratorio 8

Ejecutar el módulo `ping`.

---

## Laboratorio 9

Ejecutar el comando `hostname`.

---

## Laboratorio 10

Ejecutar `df -h` y comparar el espacio disponible de todos los servidores desde un único comando.

---

# Resumen

En esta segunda fase aprendimos:

- Cómo instalar Ansible.
- Los componentes instalados con ansible-core.
- Cómo funciona la comunicación mediante SSH.
- La importancia de Python en los Managed Nodes.
- Cómo crear y organizar un Inventory.
- Cómo utilizar claves SSH.
- Cómo ejecutar comandos Ad-Hoc.
- Cómo validar la conectividad con el módulo `ping`.

En la **Fase 3** comenzaremos a trabajar con **Playbooks**, comprenderemos la sintaxis YAML, aprenderemos el concepto de **idempotencia** en la práctica y construiremos nuestras primeras automatizaciones completas utilizando Ansible.

----

# 78. Introducción a Ansible (Fase 3)

> **Módulo 11 – Automatización con Ansible**
>
> **Archivo:** `78-introduccion-ansible.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Comprender la sintaxis YAML.
- Crear tu primer Playbook.
- Entender cómo funciona la idempotencia.
- Ejecutar Playbooks.
- Comprender las Tasks.
- Utilizar Variables.
- Crear Handlers.
- Ejecutar tareas selectivas mediante Tags.
- Organizar correctamente un proyecto de Ansible.

---

# ¿Qué es un Playbook?

Un **Playbook** es un archivo escrito en **YAML** que describe paso a paso las tareas que Ansible debe ejecutar.

Puede contener:

- Instalación de software.
- Creación de usuarios.
- Reinicio de servicios.
- Configuración de archivos.
- Actualizaciones.
- Automatizaciones completas.

En otras palabras:

```text
Playbook

↓

Lista ordenada de tareas

↓

Ansible las ejecuta

↓

Los servidores quedan configurados
```

---

# ¿Por qué YAML?

YAML significa

> YAML Ain't Markup Language

Fue diseñado para ser:

- Fácil de leer.
- Fácil de escribir.
- Muy limpio.
- Legible para humanos.

Por ello Red Hat utiliza YAML en:

- Ansible
- OpenShift
- Kubernetes
- Automation Platform

---

# Sintaxis YAML

Ejemplo sencillo

```yaml
nombre: Alejandro

edad: 30

pais: Dominicana
```

Observa que utiliza:

- Clave
- Dos puntos
- Valor

---

# Listas

```yaml
usuarios:

  - juan

  - maria

  - pedro
```

---

# Diccionarios

```yaml
usuario:

  nombre: alejandro

  shell: /bin/bash

  uid: 1001
```

---

# Comentarios

```yaml
# Crear usuario administrador

usuario:

  nombre: admin
```

---

# Indentación

YAML utiliza espacios.

Nunca tabulaciones.

Correcto

```yaml
usuarios:

  - juan

  - maria
```

Incorrecto

```yaml
usuarios:
<TAB>- juan
```

---

# Estructura de un Playbook

```text
Playbook

↓

Play

↓

Tasks

↓

Modules
```

---

# Diagrama

```text
Playbook

│

├── Hosts

├── Variables

├── Tasks

├── Handlers

└── Roles
```

---

# Primer Playbook

Archivo

```text
primer_playbook.yml
```

Contenido

```yaml
---
- name: Primer Playbook

  hosts: all

  become: true

  tasks:

    - name: Mostrar hostname

      command: hostname
```

---

# Explicación

```yaml
hosts:
```

Define sobre qué servidores trabajará.

---

```yaml
hosts: all
```

Todos los servidores.

---

```yaml
hosts: web
```

Sólo servidores web.

---

```yaml
hosts: database
```

Sólo servidores de bases de datos.

---

# become

```yaml
become: true
```

Equivale a utilizar

```bash
sudo
```

Permite ejecutar tareas con privilegios administrativos.

---

# Tasks

Las tareas son el corazón de un Playbook.

Ejemplo

```yaml
tasks:

- Instalar Apache

- Copiar configuración

- Reiniciar servicio

- Validar estado
```

---

# Modules

Cada Task normalmente ejecuta un módulo.

```yaml
tasks:

- name: Instalar Apache

  dnf:

    name: httpd

    state: present
```

---

# Primer Playbook útil

```yaml
---

- name: Instalar Apache

  hosts: web

  become: true

  tasks:

    - name: Instalar paquete

      dnf:

        name: httpd

        state: present

    - name: Habilitar servicio

      service:

        name: httpd

        state: started

        enabled: yes
```

---

# Ejecución

```bash
ansible-playbook -i inventory.ini apache.yml
```

---

# Flujo de ejecución

```text
Playbook

↓

Inventory

↓

Grupo Web

↓

SSH

↓

Servidor

↓

Instala Apache

↓

Inicia servicio

↓

Finaliza
```

---

# Resultado

```text
PLAY

TASK

changed

TASK

ok

PLAY RECAP

web01

ok=2

changed=1

failed=0
```

---

# ¿Qué significa "changed"?

Significa que Ansible realizó una modificación.

Ejemplo

```text
Apache no existía

↓

Fue instalado

↓

changed=1
```

---

# ¿Qué significa "ok"?

La tarea fue ejecutada pero no fue necesario realizar cambios.

Ejemplo

```text
Apache ya estaba instalado

↓

ok
```

---

# Idempotencia

Supongamos este Playbook

```yaml
dnf:

  name: httpd

  state: present
```

Primera ejecución

```text
Apache

↓

No existe

↓

Se instala

↓

changed
```

---

Segunda ejecución

```text
Apache

↓

Ya existe

↓

No hace nada

↓

ok
```

Eso es idempotencia.

---

# Variables

Las variables permiten reutilizar valores.

Ejemplo

```yaml
vars:

  paquete: httpd
```

Luego

```yaml
dnf:

  name: "{{ paquete }}"

  state: present
```

---

# Variables múltiples

```yaml
vars:

  usuario: administrador

  shell: /bin/bash

  uid: 1500
```

---

# Uso

```yaml
user:

  name: "{{ usuario }}"
```

---

# Ventajas

- Reutilización.
- Fácil mantenimiento.
- Menos código.
- Más claridad.

---

# Handlers

Los Handlers sólo se ejecutan cuando una tarea los notifica.

Ejemplo

```text
Modificar configuración

↓

Notificar Handler

↓

Reiniciar Apache
```

---

# Ejemplo

```yaml
tasks:

- name: Copiar configuración

  copy:

    src: httpd.conf

    dest: /etc/httpd/conf/httpd.conf

  notify:

    - Reiniciar Apache
```

---

Handler

```yaml
handlers:

- name: Reiniciar Apache

  service:

    name: httpd

    state: restarted
```

---

# ¿Por qué Handlers?

Sin Handler

```text
Copiar archivo

↓

Reiniciar

↓

Copiar archivo

↓

Reiniciar

↓

Copiar archivo

↓

Reiniciar
```

Tres reinicios.

---

Con Handler

```text
Copiar

↓

Copiar

↓

Copiar

↓

Un solo reinicio
```

Mucho más eficiente.

---

# Tags

Permiten ejecutar únicamente ciertas tareas.

Ejemplo

```yaml
tasks:

- name: Instalar Apache

  tags:

    - apache
```

---

Ejecución

```bash
ansible-playbook apache.yml --tags apache
```

---

Múltiples Tags

```yaml
tags:

- apache

- web

- produccion
```

---

# Saltar Tags

```bash
ansible-playbook sitio.yml --skip-tags backup
```

---

# Estructura recomendada

```text
Proyecto

│

├── inventory.ini

├── playbook.yml

├── templates/

├── files/

├── vars/

├── group_vars/

├── host_vars/

└── roles/
```

---

# Organización empresarial

```text
Ansible

│

├── Inventories

├── Playbooks

├── Roles

├── Templates

├── Variables

├── Collections

└── Documentation
```

---

# Validar sintaxis

Antes de ejecutar

```bash
ansible-playbook playbook.yml --syntax-check
```

---

# Simulación

Modo Dry Run

```bash
ansible-playbook playbook.yml --check
```

Muy útil en producción.

---

# Mostrar diferencias

```bash
ansible-playbook playbook.yml --diff
```

---

# Ejecución detallada

```bash
ansible-playbook playbook.yml -vvv
```

---

# Buenas prácticas

- Un Playbook debe tener un objetivo específico.
- Utilizar nombres descriptivos para cada Task.
- Aprovechar la idempotencia.
- Utilizar Variables en lugar de valores fijos.
- Utilizar Handlers para reinicios.
- Validar la sintaxis antes de ejecutar.
- Probar primero en laboratorio.
- Organizar correctamente el proyecto.

---

# Errores comunes

## Error 1

Utilizar tabulaciones.

---

## Error 2

Indentación incorrecta.

---

## Error 3

No utilizar Handlers.

---

## Error 4

Repetir código innecesariamente.

---

## Error 5

No validar la sintaxis.

---

## Error 6

Modificar producción sin utilizar

```bash
--check
```

---

## Error 7

Crear Playbooks demasiado grandes.

Es preferible dividirlos por función.

---

# Laboratorio RHCSA

## Laboratorio 1

Crear el primer Playbook.

---

## Laboratorio 2

Ejecutarlo sobre un servidor.

---

## Laboratorio 3

Crear un Playbook que instale Apache.

---

## Laboratorio 4

Agregar Variables.

---

## Laboratorio 5

Crear un Handler para reiniciar Apache.

---

## Laboratorio 6

Utilizar Tags.

---

## Laboratorio 7

Ejecutar únicamente un Tag.

---

## Laboratorio 8

Ejecutar el Playbook en modo

```bash
--check
```

---

## Laboratorio 9

Validar la sintaxis.

---

## Laboratorio 10

Crear un proyecto Ansible con la estructura recomendada y documentar cada uno de sus directorios.

---

# Resumen

En esta tercera fase aprendimos:

- Qué es un Playbook.
- La sintaxis básica de YAML.
- Cómo crear Tasks.
- Cómo utilizar Variables.
- Qué son los Handlers.
- Cómo funcionan los Tags.
- La importancia de la idempotencia.
- Cómo validar y ejecutar Playbooks de forma segura.
- Cómo organizar un proyecto profesional de Ansible.

En la **Fase 4** construiremos un escenario empresarial completo, automatizaremos la configuración de múltiples servidores, estudiaremos técnicas de troubleshooting, aplicaremos buenas prácticas de producción y realizaremos un laboratorio integral con ejercicios similares a los del examen RHCSA.

----

# 78. Introducción a Ansible (Fase 4)

> **Módulo 11 – Automatización con Ansible**
>
> **Archivo:** `78-introduccion-ansible.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Diseñar una infraestructura automatizada utilizando Ansible.
- Comprender el flujo de trabajo empresarial.
- Identificar errores comunes durante la automatización.
- Aplicar técnicas de troubleshooting.
- Construir proyectos escalables.
- Implementar buenas prácticas utilizadas en entornos corporativos.
- Resolver un laboratorio completo similar al examen RHCSA.

---

# Proyecto Empresarial

Supongamos una empresa con la siguiente infraestructura:

```text
                    Empresa

                        │

        ┌───────────────┼────────────────┐

        ▼               ▼                ▼

     Web Farm       Databases        Infrastructure

        │               │                │

    Apache01        PostgreSQL01      DNS01

    Apache02        PostgreSQL02      Backup01

    Apache03        SQLServer01       Monitor01

    Apache04                          Proxy01
```

Todos estos servidores deben mantenerse exactamente iguales.

---

# Sin Automatización

Cada cambio implica:

```text
Conectarse

↓

Modificar

↓

Guardar

↓

Validar

↓

Repetir
```

Para 100 servidores:

```text
100 conexiones SSH

100 modificaciones

100 validaciones

100 oportunidades de cometer errores
```

---

# Con Automatización

```text
Administrador

↓

Playbook

↓

Inventory

↓

Ansible

↓

100 Servidores

↓

Configuración idéntica
```

Una única ejecución.

---

# Flujo Empresarial

```text
Administrador

↓

Repositorio Git

↓

Playbook

↓

Servidor Ansible

↓

SSH

↓

Todos los servidores

↓

Reporte
```

---

# Flujo DevOps

```text
Git

↓

Pull Request

↓

Revisión

↓

Merge

↓

Playbook

↓

Producción
```

---

# Organización Profesional

```text
ansible-project/

│

├── inventories/

│   ├── dev/

│   ├── qa/

│   └── production/

│

├── group_vars/

├── host_vars/

├── roles/

├── collections/

├── playbooks/

├── templates/

├── files/

├── documentation/

└── README.md
```

---

# Separación de Ambientes

Nunca utilizar un único Inventory.

Correcto

```text
Development

Testing

QA

Production
```

Incorrecto

```text
Todos los servidores

↓

Mismo Inventory
```

---

# Ejemplo

```text
inventories/

├── development

├── testing

├── production
```

---

# Control de Versiones

Todos los Playbooks deberían almacenarse en Git.

```text
Administrador

↓

Git

↓

Versiones

↓

Rollback

↓

Historial
```

---

# Flujo recomendado

```text
Crear Playbook

↓

Validar Sintaxis

↓

Ejecutar --check

↓

Probar en DEV

↓

QA

↓

Producción
```

---

# Dry Run

Antes de producción

```bash
ansible-playbook sitio.yml --check
```

Permite conocer qué cambios realizará.

---

# Mostrar diferencias

```bash
ansible-playbook sitio.yml --diff
```

---

# Validar sintaxis

```bash
ansible-playbook sitio.yml --syntax-check
```

---

# Ejecución limitada

Actualizar únicamente un servidor.

```bash
ansible-playbook sitio.yml --limit web01
```

---

Actualizar únicamente un grupo.

```bash
ansible-playbook sitio.yml --limit database
```

---

# Ejecución Paralela

Ansible trabaja en paralelo.

```text
Control Node

↓

Forks

↓

Servidor1

Servidor2

Servidor3

Servidor4

Servidor5
```

---

# Parámetro Forks

Por defecto

```text
5 servidores simultáneamente
```

Puede modificarse.

```bash
ansible-playbook sitio.yml --forks 20
```

---

# Escalabilidad

```text
5 Servidores

↓

50 Servidores

↓

500 Servidores

↓

5000 Servidores
```

El mismo Playbook.

---

# Seguridad

Buenas prácticas

- SSH por claves.
- No utilizar root.
- Utilizar sudo.
- Proteger secretos.
- Utilizar Ansible Vault.
- Limitar privilegios.

---

# Logging

Registrar todas las ejecuciones.

```text
Fecha

↓

Playbook

↓

Servidor

↓

Resultado

↓

Administrador
```

---

# Auditoría

Toda automatización debería responder:

- ¿Quién ejecutó?
- ¿Cuándo?
- ¿Qué cambió?
- ¿En cuáles servidores?
- ¿Fue exitoso?

---

# Troubleshooting

## Error 1

UNREACHABLE

```text
SSH

↓

No conecta
```

Posibles causas

- Firewall
- SSH detenido
- Puerto incorrecto
- IP incorrecta

---

## Error 2

Permission Denied

```text
SSH

↓

Authentication Failed
```

Revisar

- Llaves SSH
- Usuario
- Permisos

---

## Error 3

Python no encontrado

```text
FAILED

Python interpreter not found
```

Solución

Instalar Python.

---

## Error 4

YAML inválido

```text
Syntax Error
```

Generalmente ocurre por:

- Espacios
- Indentación
- Dos puntos
- Tabulaciones

---

## Error 5

Host no encontrado

```text
UNREACHABLE

Name Resolution Failed
```

Revisar

- DNS
- /etc/hosts
- Inventory

---

## Error 6

Módulo inexistente

```text
Module not found
```

Revisar

- Nombre correcto
- Collection instalada

---

# Troubleshooting Paso a Paso

```text
¿Funciona SSH?

↓

Sí

↓

¿Existe Python?

↓

Sí

↓

¿Inventory correcto?

↓

Sí

↓

¿Playbook válido?

↓

Sí

↓

Ejecutar
```

---

# Checklist antes de Producción

□ Inventory correcto

□ Playbook probado

□ Backup realizado

□ Sintaxis validada

□ Dry Run ejecutado

□ Variables revisadas

□ SSH funcionando

□ Servidores disponibles

□ Cambio aprobado

□ Documentación actualizada

---

# Caso Práctico

La empresa necesita:

- Instalar Apache.
- Crear usuarios.
- Actualizar paquetes.
- Reiniciar servicios.
- Configurar Firewall.
- Copiar certificados.
- Validar servicios.

Todo ello debe ejecutarse automáticamente.

---

# Resultado

```text
Administrador

↓

Playbook

↓

25 Web Servers

↓

Configurados

↓

Reporte Final
```

---

# Buenas Prácticas Empresariales

- Un Playbook para cada objetivo.
- Utilizar Roles.
- Utilizar Variables.
- Reutilizar código.
- Versionar en Git.
- Ejecutar pruebas antes de producción.
- Utilizar nombres descriptivos.
- Documentar todas las automatizaciones.
- Automatizar únicamente procesos comprendidos.

---

# Errores Frecuentes

## Automatizar sin comprender el proceso

Nunca automatizar algo que aún no funciona manualmente.

---

## Ejecutar directamente en Producción

Siempre utilizar:

```text
Development

↓

QA

↓

Producción
```

---

## No utilizar Inventories separados

Es una de las causas más comunes de errores.

---

## No utilizar Variables

Provoca duplicación de código.

---

## Ignorar Handlers

Produce reinicios innecesarios.

---

## No documentar

Después de algunos meses nadie recordará el propósito del Playbook.

---

# Laboratorio Integral RHCSA

## Escenario

Dispones de:

```text
Control Node

↓

web01

↓

web02

↓

db01

↓

backup01
```

Debes:

- Verificar conectividad.
- Crear el Inventory.
- Ejecutar comandos Ad-Hoc.
- Instalar Apache.
- Crear un usuario administrador.
- Reiniciar el servicio únicamente cuando sea necesario.
- Validar la configuración.
- Ejecutar el Playbook en modo `--check`.
- Ejecutarlo finalmente en producción.

---

# Ejercicio 2

Crear un proyecto profesional con:

```text
inventories/

roles/

playbooks/

templates/

group_vars/

host_vars/

documentation/
```

---

# Ejercicio 3

Automatizar la instalación de:

- Apache
- Nginx
- PostgreSQL

Utilizando Variables.

---

# Ejercicio 4

Agregar Handlers para todos los servicios.

---

# Ejercicio 5

Utilizar Tags para ejecutar únicamente:

- Apache
- PostgreSQL
- Usuarios

---

# Preguntas de Repaso

1. ¿Qué es Ansible?

2. ¿Qué es un Control Node?

3. ¿Qué es un Managed Node?

4. ¿Qué es un Inventory?

5. ¿Qué es un Playbook?

6. ¿Qué significa idempotencia?

7. ¿Qué función cumplen los Handlers?

8. ¿Qué ventajas ofrece YAML?

9. ¿Qué hace el módulo `ping`?

10. ¿Por qué Ansible no necesita agentes?

11. ¿Cuál es la diferencia entre un comando Ad-Hoc y un Playbook?

12. ¿Qué ventajas ofrece el uso de Variables?

13. ¿Por qué es importante utilizar Git?

14. ¿Qué utilidad tiene el modo `--check`?

15. ¿Qué utilidad tiene `--syntax-check`?

16. ¿Por qué separar Development, QA y Production?

17. ¿Qué ventajas tiene utilizar SSH por llaves?

18. ¿Qué ocurre cuando un módulo devuelve `changed`?

19. ¿Qué ocurre cuando devuelve `ok`?

20. ¿Por qué Ansible es una herramienta fundamental para administradores Linux?

---

# Conclusiones

En este capítulo aprendimos los fundamentos necesarios para comenzar a trabajar con Ansible:

- Su arquitectura y componentes.
- La comunicación mediante SSH.
- El uso de Inventories.
- Los comandos Ad-Hoc.
- La creación de Playbooks.
- La sintaxis YAML.
- Variables, Handlers y Tags.
- Buenas prácticas empresariales.
- Técnicas de troubleshooting.
- Organización profesional de proyectos.

Estos conceptos constituyen la base de toda automatización con Ansible y serán utilizados en los capítulos siguientes.

---

# Próximo Capítulo

## **79. Instalación y Configuración de Ansible**

En el siguiente capítulo profundizaremos en la instalación de **ansible-core**, la configuración mediante `ansible.cfg`, el uso de inventarios avanzados, la autenticación por claves SSH, la configuración de privilegios (`become`), la administración de múltiples entornos y las mejores prácticas para preparar un entorno empresarial listo para automatización a gran escala.















