# 80. Inventarios en Ansible (Fase 1)

> **Módulo 11 – Automatización con Ansible**
>
> **Archivo:** `80-inventarios-ansible.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Comprender qué es un Inventory.
- Identificar los diferentes tipos de Inventarios.
- Crear Inventarios estáticos.
- Utilizar el formato INI.
- Utilizar el formato YAML.
- Organizar servidores mediante grupos.
- Crear grupos hijos.
- Definir variables por host.
- Validar un Inventory.
- Comprender cómo Ansible identifica cada servidor.

---

# Introducción

Uno de los componentes más importantes de Ansible es el **Inventory**.

Podemos imaginar el Inventory como la agenda telefónica de Ansible.

Sin un Inventory, Ansible no sabe:

- Qué servidores existen.
- Cómo conectarse.
- Qué usuario utilizar.
- Qué puerto SSH utilizar.
- Qué variables pertenecen a cada servidor.

---

# ¿Qué es un Inventory?

Un **Inventory** es un archivo que contiene la lista de todos los servidores administrados por Ansible.

En él se define:

- Nombre del servidor.
- Dirección IP.
- Nombre DNS.
- Usuario.
- Puerto SSH.
- Variables.
- Grupos.

---

# Arquitectura

```text
                 Administrador

                      │

                      ▼

                 ansible-playbook

                      │

                      ▼

                  Inventory

                      │

      ┌───────────────┼───────────────┐

      ▼               ▼               ▼

    Web01          DB01          Backup01
```

---

# ¿Por qué es importante?

Imaginemos una empresa con:

```text
25 Servidores Web

15 Bases de Datos

10 DNS

5 Balanceadores

8 Monitoreo

20 Aplicaciones
```

Total

```text
83 Servidores
```

Sería imposible recordar todas las direcciones IP.

El Inventory organiza toda esta información.

---

# Inventario Estático

El tipo más utilizado durante el aprendizaje.

Los servidores son definidos manualmente.

Ejemplo

```text
inventory.ini
```

---

# Inventario Dinámico

Los servidores son descubiertos automáticamente.

Ejemplos:

- AWS
- Azure
- VMware
- Kubernetes
- OpenStack

Este tipo será estudiado más adelante.

---

# Formatos soportados

Ansible soporta varios formatos.

Los más utilizados son:

- INI
- YAML

---

# Formato INI

Es el más sencillo.

Ejemplo

```ini
web01 ansible_host=192.168.100.20

db01 ansible_host=192.168.100.30

backup01 ansible_host=192.168.100.40
```

---

# Formato YAML

También es ampliamente utilizado.

```yaml
all:

  hosts:

    web01:

      ansible_host: 192.168.100.20

    db01:

      ansible_host: 192.168.100.30
```

---

# Comparación

| Característica | INI | YAML |
|----------------|-----|------|
| Fácil de escribir | Sí | Sí |
| Fácil de leer | Sí | Muy Sí |
| Variables complejas | Limitado | Excelente |
| Grandes proyectos | Bueno | Excelente |

---

# Estructura Básica

```text
Inventory

│

├── Hosts

├── Variables

└── Grupos
```

---

# Hosts

Cada servidor representa un Host.

Ejemplo

```ini
web01

web02

web03

db01
```

Cada Host representa una máquina.

---

# Alias

Podemos asignar un nombre amigable.

```ini
web_principal ansible_host=192.168.100.20
```

Ahora Ansible utilizará

```text
web_principal
```

en lugar de la dirección IP.

---

# Ventajas de utilizar Alias

En lugar de

```text
192.168.100.20
```

podemos utilizar

```text
web01
```

o

```text
servidor_web_principal
```

El código será mucho más legible.

---

# Variables por Host

Ejemplo

```ini
web01

ansible_host=192.168.100.20

ansible_user=ansible
```

---

También podemos indicar

```ini
ansible_port=2222
```

---

Y además

```ini
ansible_ssh_private_key_file=~/.ssh/id_ed25519
```

---

# Ejemplo completo

```ini
web01

ansible_host=192.168.100.20

ansible_user=ansible

ansible_port=22

ansible_ssh_private_key_file=~/.ssh/id_ed25519
```

---

# Conexión

```text
Inventory

↓

Host

↓

SSH

↓

Servidor Linux
```

---

# Grupos

Los grupos permiten organizar servidores similares.

Ejemplo

```ini
[web]

web01

web02

web03
```

---

Grupo Database

```ini
[database]

db01

db02
```

---

Grupo Backup

```ini
[backup]

backup01

backup02
```

---

# Arquitectura

```text
Inventory

│

├── Web

├── Database

├── Backup

└── Monitoring
```

---

# Beneficios

En lugar de ejecutar

```bash
ansible web01

ansible web02

ansible web03
```

simplemente ejecutamos

```bash
ansible web
```

Todos los servidores serán administrados.

---

# Grupos Hijos

Podemos crear grupos que contienen otros grupos.

Ejemplo

```ini
[linux:children]

web

database

backup
```

---

Diagrama

```text
Linux

│

├── Web

├── Database

└── Backup
```

---

Otro ejemplo

```ini
[production:children]

web

database

monitoring
```

---

# Grupo all

Todos los Inventories contienen automáticamente

```text
all
```

Representa todos los servidores.

---

# Grupo ungrouped

Contiene servidores que no pertenecen a ningún grupo.

```text
all

│

├── web

├── database

└── ungrouped
```

---

# Inventario por defecto

Generalmente

```text
/etc/ansible/hosts
```

Aunque puede modificarse mediante

```ini
inventory=
```

en

```text
ansible.cfg
```

---

# Validar el Inventory

```bash
ansible-inventory --graph
```

Resultado

```text
@all

|--@ungrouped

|--@web

|  |--web01

|  |--web02

|--@database

|  |--db01
```

---

# Mostrar formato JSON

```bash
ansible-inventory --list
```

---

# Mostrar únicamente un Host

```bash
ansible-inventory --host web01
```

---

# Mostrar variables

```bash
ansible-inventory --host db01
```

---

# Verificar sintaxis

```bash
ansible-inventory -i inventory.ini --graph
```

---

# Comprobar conectividad

```bash
ansible all -m ping
```

---

# Flujo completo

```text
Administrador

↓

Playbook

↓

Inventory

↓

Grupo

↓

Host

↓

SSH

↓

Servidor Linux
```

---

# Caso Empresarial

Supongamos que administramos:

```text
40 Web Servers

15 PostgreSQL

10 SQL Server

5 DNS

5 Proxy

8 Monitor

12 Backup
```

Un Inventory correctamente organizado permitirá administrar toda la infraestructura mediante grupos.

---

# Buenas prácticas

- Utilizar nombres descriptivos.
- Agrupar servidores por función.
- Evitar utilizar únicamente direcciones IP.
- Mantener un Inventory por ambiente.
- Utilizar alias consistentes.
- Documentar el propósito de cada grupo.
- Validar el Inventory antes de ejecutar Playbooks.

---

# Errores comunes

## Error 1

Utilizar nombres poco descriptivos.

```text
server1

server2

server3
```

---

## Error 2

No utilizar grupos.

---

## Error 3

Duplicar Hosts.

---

## Error 4

Direcciones IP incorrectas.

---

## Error 5

No validar el Inventory.

---

## Error 6

Mezclar servidores de Producción y Desarrollo.

---

## Error 7

Modificar el Inventory sin documentar los cambios.

---

# Laboratorio RHCSA

## Laboratorio 1

Crear un Inventory en formato INI.

---

## Laboratorio 2

Agregar cinco servidores.

---

## Laboratorio 3

Crear los grupos:

- Web
- Database
- Backup

---

## Laboratorio 4

Crear un grupo padre llamado **production**.

---

## Laboratorio 5

Asignar variables por Host.

---

## Laboratorio 6

Agregar un Alias para cada servidor.

---

## Laboratorio 7

Validar el Inventory utilizando:

```bash
ansible-inventory --graph
```

---

## Laboratorio 8

Mostrar el Inventory completo en formato JSON.

---

## Laboratorio 9

Verificar la conectividad utilizando:

```bash
ansible all -m ping
```

---

## Laboratorio 10

Diseñar un Inventory para una empresa con:

- 30 Servidores Web.
- 20 Bases de Datos.
- 10 DNS.
- 5 Balanceadores.
- 5 Servidores de Monitoreo.

Organizar todos los Hosts mediante grupos y grupos hijos.

---

# Resumen

En esta primera fase aprendimos qué es un Inventory, cómo se estructura, los formatos INI y YAML, el uso de Hosts, Alias, Variables por Host, Grupos y Grupos Hijos. También conocimos los comandos básicos para validar y visualizar un Inventory, sentando las bases para administrar infraestructuras de cualquier tamaño.

En la **Fase 2** profundizaremos en **group_vars**, **host_vars**, variables globales, variables de conexión, precedencia de variables y la organización de Inventarios para múltiples ambientes empresariales.

----

# 80. Inventarios en Ansible (Fase 2)

> **Módulo 11 – Automatización con Ansible**
>
> **Archivo:** `80-inventarios-ansible.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Comprender el funcionamiento de `group_vars`.
- Comprender el funcionamiento de `host_vars`.
- Utilizar variables globales.
- Configurar variables de conexión.
- Entender la precedencia de variables.
- Organizar Inventarios para múltiples ambientes.
- Diseñar Inventarios empresariales escalables.
- Aplicar buenas prácticas utilizadas en grandes infraestructuras.

---

# Introducción

En proyectos pequeños es posible colocar todas las variables directamente dentro del Inventory.

Sin embargo, cuando la infraestructura comienza a crecer, esta práctica deja de ser recomendable.

Por ejemplo:

```text
10 servidores

↓

Fácil de administrar
```

Pero...

```text
800 servidores

↓

Miles de variables

↓

Difícil mantenimiento
```

Por esta razón Ansible incorpora los directorios:

- group_vars
- host_vars

---

# Organización recomendada

```text
ansible-project/

├── ansible.cfg

├── inventories/

├── group_vars/

├── host_vars/

├── playbooks/

└── roles/
```

---

# ¿Qué es group_vars?

Permite definir variables que serán compartidas por todos los servidores de un grupo.

Ejemplo:

```text
Grupo Web

↓

Puerto HTTP

↓

DocumentRoot

↓

Usuario Apache
```

Todos los servidores utilizarán la misma configuración.

---

# Directorio group_vars

```text
group_vars/

├── all.yml

├── web.yml

├── database.yml

├── monitoring.yml

├── backup.yml

└── dns.yml
```

Cada archivo corresponde a un grupo.

---

# Ejemplo

Archivo

```text
group_vars/web.yml
```

Contenido

```yaml
---

http_port: 80

https_port: 443

apache_service: httpd

document_root: /var/www/html
```

Todos los servidores pertenecientes al grupo **web** heredarán estas variables.

---

# Arquitectura

```text
               group_vars

                    │

      ┌─────────────┴─────────────┐

      ▼                           ▼

group_vars/web.yml       group_vars/database.yml

      │                           │

      ▼                           ▼

 Grupo Web               Grupo Database
```

---

# Variables Globales

Existe un archivo muy especial.

```text
group_vars/all.yml
```

Todas las máquinas reciben estas variables.

Ejemplo

```yaml
---

empresa: "Empresa Demo"

timezone: "America/Santo_Domingo"

ntp_server: "pool.ntp.org"

dns_server: "192.168.100.10"
```

---

# ¿Cuándo utilizar all.yml?

Cuando una configuración aplica para toda la infraestructura.

Ejemplos

- Zona horaria.
- Servidor NTP.
- Dominio corporativo.
- DNS.
- Proxy.
- Repositorios internos.

---

# ¿Qué es host_vars?

Permite definir variables para un único servidor.

---

# Directorio

```text
host_vars/

├── web01.yml

├── web02.yml

├── db01.yml

├── backup01.yml

└── dns01.yml
```

---

# Ejemplo

Archivo

```text
host_vars/web01.yml
```

Contenido

```yaml
---

server_id: WEB001

virtual_ip: 192.168.100.200

priority: 200
```

Solamente **web01** utilizará estas variables.

---

# ¿Cuándo utilizar host_vars?

Cuando únicamente un servidor necesita una configuración diferente.

Ejemplos

- Dirección IP Virtual.
- Prioridad HA.
- Identificador.
- Licencia.
- Ruta específica.
- Nombre personalizado.

---

# Comparación

| group_vars | host_vars |
|------------|-----------|
| Afecta un grupo | Afecta un Host |
| Fácil mantenimiento | Muy específico |
| Reutilizable | Configuración individual |
| Ideal para cientos de servidores | Ideal para excepciones |

---

# Variables dentro del Inventory

También es posible definir variables directamente.

Ejemplo

```ini
web01

ansible_host=192.168.100.20

http_port=80
```

Aunque funciona, no es recomendable para proyectos grandes.

---

# Variables de Conexión

Ansible reconoce variables especiales.

Las más utilizadas son:

| Variable | Función |
|----------|----------|
| ansible_host | Dirección IP |
| ansible_user | Usuario SSH |
| ansible_port | Puerto SSH |
| ansible_password | Contraseña |
| ansible_connection | Tipo de conexión |
| ansible_python_interpreter | Python remoto |
| ansible_ssh_private_key_file | Llave privada |

---

# ansible_host

```yaml
ansible_host: 192.168.100.20
```

Indica la dirección real del servidor.

---

# ansible_user

```yaml
ansible_user: ansible
```

Usuario utilizado para conectarse.

---

# ansible_port

```yaml
ansible_port: 2222
```

Cuando SSH utiliza un puerto diferente al 22.

---

# ansible_connection

Ejemplo

```yaml
ansible_connection: ssh
```

También existen otros métodos.

- local
- docker
- podman
- paramiko

---

# ansible_python_interpreter

```yaml
ansible_python_interpreter: /usr/bin/python3
```

Muy útil cuando existen múltiples versiones de Python.

---

# ansible_ssh_private_key_file

```yaml
ansible_ssh_private_key_file: ~/.ssh/id_ed25519
```

Permite especificar una llave diferente para determinados servidores.

---

# Variables por Grupo

También pueden declararse dentro del Inventory.

Ejemplo

```ini
[web]

web01

web02

[web:vars]

http_port=80

apache_service=httpd
```

Sin embargo, para proyectos empresariales se recomienda utilizar `group_vars`.

---

# Variables por Host

```ini
web01

http_port=8080
```

Este servidor utilizará un puerto diferente.

---

# Organización por Ambientes

Una empresa normalmente posee varios ambientes.

```text
Development

↓

Testing

↓

QA

↓

Production

↓

Disaster Recovery
```

Cada ambiente debería tener su propio Inventory.

---

# Ejemplo

```text
inventories/

├── development/

├── testing/

├── qa/

├── production/

└── dr/
```

---

# Estructura Completa

```text
inventories/

├── development/

│   ├── hosts.yml

│   ├── group_vars/

│   └── host_vars/

├── qa/

│   ├── hosts.yml

│   ├── group_vars/

│   └── host_vars/

└── production/

    ├── hosts.yml

    ├── group_vars/

    └── host_vars/
```

Cada ambiente mantiene sus propias variables.

---

# Precedencia de Variables

Cuando una misma variable aparece en varios lugares, Ansible debe decidir cuál utilizar.

Simplificando el proceso:

```text
Host Variables

↓

Group Variables

↓

Variables del Inventory

↓

Variables por defecto
```

La variable más específica tiene prioridad.

---

# Ejemplo

Supongamos:

```text
group_vars/web.yml

http_port=80
```

Y además

```text
host_vars/web01.yml

http_port=8080
```

Resultado

```text
web01

↓

8080
```

Los demás servidores continuarán utilizando:

```text
80
```

---

# Flujo de Resolución

```text
Playbook

↓

Inventory

↓

group_vars

↓

host_vars

↓

Variable Final
```

---

# Infraestructuras Grandes

Supongamos una empresa con:

```text
500 Web Servers

200 PostgreSQL

150 SQL Server

80 DNS

60 Proxy

40 Backup

30 Monitor
```

Total

```text
1060 Servidores
```

Sin `group_vars` y `host_vars`, administrar esta infraestructura sería extremadamente complejo.

---

# Organización Recomendada

```text
Producción

│

├── Web

├── PostgreSQL

├── SQL Server

├── DNS

├── Proxy

├── Backup

└── Monitoring
```

Cada grupo tendrá sus propias variables.

---

# Convenciones de Nombres

Utilizar nombres consistentes.

Correcto

```text
web01

web02

web03
```

Incorrecto

```text
server

server2

hostnuevo
```

---

# Separación por Función

Organizar servidores por su propósito.

```text
web

database

proxy

backup

monitoring

dns
```

No mezclar funciones diferentes en un mismo grupo.

---

# Integración con Git

La estructura completa debe almacenarse en un repositorio.

```text
Git

↓

Inventory

↓

group_vars

↓

host_vars

↓

Playbooks
```

---

# Arquitectura Empresarial

```text
                     Git

                      │

                      ▼

              Proyecto Ansible

                      │

      ┌───────────────┼───────────────┐

      ▼               ▼               ▼

 Inventories     group_vars      host_vars

                      │

                      ▼

                 Playbooks

                      │

                      ▼

                 Infraestructura
```

---

# Buenas prácticas

- Utilizar `group_vars` siempre que sea posible.
- Utilizar `host_vars` únicamente para excepciones.
- Evitar duplicar variables.
- Separar cada ambiente.
- Mantener nombres consistentes.
- Documentar cada grupo.
- Utilizar Git para controlar cambios.
- Evitar almacenar contraseñas en texto plano.
- Validar el Inventory antes de ejecutar Playbooks.

---

# Errores comunes

## Error 1

Duplicar la misma variable en varios archivos.

---

## Error 2

Colocar todas las variables dentro del Inventory.

---

## Error 3

Crear cientos de `host_vars` cuando deberían utilizarse `group_vars`.

---

## Error 4

No separar Producción y Desarrollo.

---

## Error 5

Modificar variables directamente en Producción sin pruebas previas.

---

## Error 6

Utilizar nombres inconsistentes para grupos y Hosts.

---

## Error 7

Guardar información sensible sin utilizar Ansible Vault.

---

# Laboratorio RHCSA

## Laboratorio 1

Crear el directorio:

```text
group_vars/
```

---

## Laboratorio 2

Crear:

```text
group_vars/web.yml
```

Agregar:

- Puerto HTTP.
- Puerto HTTPS.
- Servicio Apache.
- DocumentRoot.

---

## Laboratorio 3

Crear:

```text
group_vars/database.yml
```

Agregar variables comunes para PostgreSQL o SQL Server.

---

## Laboratorio 4

Crear:

```text
group_vars/all.yml
```

Configurar:

- Zona horaria.
- DNS.
- Servidor NTP.
- Dominio.

---

## Laboratorio 5

Crear:

```text
host_vars/web01.yml
```

Asignar una dirección IP virtual y una prioridad específica.

---

## Laboratorio 6

Configurar `ansible_user`, `ansible_host` y `ansible_port` para varios servidores.

---

## Laboratorio 7

Crear Inventarios separados para:

- Development.
- QA.
- Production.

---

## Laboratorio 8

Modificar una variable en `host_vars` y comprobar cómo sobrescribe el valor definido en `group_vars`.

---

## Laboratorio 9

Validar la estructura utilizando:

```bash
ansible-inventory --graph
```

y

```bash
ansible-inventory --list
```

---

## Laboratorio 10

Diseñar la estructura completa de Inventarios para una empresa con más de 1,000 servidores, organizando adecuadamente los directorios `inventories`, `group_vars`, `host_vars` y documentando la estrategia utilizada.

---

# Resumen

En esta segunda fase aprendimos a organizar las variables de un Inventory de forma profesional mediante `group_vars` y `host_vars`. Estudiamos las variables de conexión, las variables globales, la precedencia de variables y la separación de Inventarios por ambientes, aplicando un enfoque escalable utilizado en infraestructuras empresariales de gran tamaño.

En la **Fase 3** profundizaremos en los **Inventarios dinámicos**, los **Plugins de Inventory**, la integración con proveedores de nube como **AWS**, **Azure**, **VMware**, **OpenStack** y **Kubernetes**, así como las mejores prácticas para administrar infraestructuras que cambian constantemente.

-----

# 80. Inventarios en Ansible (Fase 3)

> **Módulo 11 – Automatización con Ansible**
>
> **Archivo:** `80-inventarios-ansible.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Comprender qué es un Inventario Dinámico.
- Conocer la arquitectura de los Plugins de Inventory.
- Utilizar Inventarios Dinámicos en AWS.
- Utilizar Inventarios Dinámicos en Azure.
- Utilizar Inventarios Dinámicos en VMware.
- Utilizar Inventarios Dinámicos en Kubernetes.
- Utilizar Inventarios Construidos (Constructed Inventory).
- Comprender cómo Ansible descubre servidores automáticamente.
- Aplicar buenas prácticas para infraestructuras híbridas y Cloud.

---

# Introducción

Hasta ahora hemos trabajado con Inventarios estáticos.

Ejemplo:

```ini
[web]

web01

web02

web03
```

Este enfoque funciona muy bien cuando la infraestructura cambia poco.

Sin embargo, en ambientes Cloud los servidores pueden:

- Crearse automáticamente.
- Destruirse automáticamente.
- Cambiar de dirección IP.
- Escalar horizontalmente.
- Migrar entre zonas.

En estos escenarios un Inventory estático deja de ser práctico.

---

# ¿Qué es un Inventario Dinámico?

Un Inventario Dinámico permite que Ansible obtenga la lista de servidores automáticamente desde una fuente externa.

En lugar de escribir manualmente:

```text
web01

web02

web03
```

Ansible consulta directamente al proveedor.

---

# Arquitectura General

```text
                Administrador

                      │

                      ▼

               ansible-playbook

                      │

                      ▼

            Dynamic Inventory Plugin

                      │

         ┌────────────┼────────────┐

         ▼            ▼            ▼

        AWS         VMware      Azure

                      │

                      ▼

             Lista de servidores

                      │

                      ▼

              Ejecución del Playbook
```

---

# Flujo de Trabajo

```text
Playbook

↓

Inventory Plugin

↓

Proveedor Cloud

↓

Consulta API

↓

Lista de Hosts

↓

Playbook
```

---

# Inventory Plugins

Actualmente Ansible utiliza Plugins de Inventory.

Algunos de los más conocidos son:

| Plugin | Plataforma |
|---------|------------|
| amazon.aws.aws_ec2 | Amazon AWS |
| azure.azcollection.azure_rm | Microsoft Azure |
| community.vmware.vmware_vm_inventory | VMware |
| kubernetes.core.k8s | Kubernetes |
| openstack.cloud.openstack | OpenStack |
| constructed | Inventario construido |

---

# ¿Por qué utilizar Plugins?

Porque permiten descubrir automáticamente:

- Nuevos servidores.
- Servidores eliminados.
- Etiquetas.
- Regiones.
- Direcciones IP.
- Sistemas Operativos.
- Metadata.

Sin modificar el Inventory manualmente.

---

# Ejemplo Empresarial

Supongamos que una empresa posee:

```text
AWS

↓

120 Máquinas Virtuales
```

Cada día:

- Se crean 30 nuevas.
- Se eliminan 25.
- Cambian 15 direcciones IP.

Actualizar un archivo manualmente sería prácticamente imposible.

---

# Inventario Dinámico en AWS

Ansible consulta directamente la API de AWS.

```text
Control Node

↓

AWS Inventory Plugin

↓

AWS API

↓

EC2 Instances

↓

Inventory generado
```

---

# Ejemplo de Configuración

```yaml
plugin: amazon.aws.aws_ec2

regions:

  - us-east-1

filters:

  instance-state-name: running
```

---

# Agrupación por Región

Ansible puede crear grupos automáticamente.

```text
us-east-1

us-west-1

eu-central-1
```

---

# Agrupación por Etiquetas

Supongamos que una instancia posee:

```text
Environment = Production
```

Ansible puede crear automáticamente:

```text
production
```

como grupo.

---

# Ejemplo

```text
EC2

↓

Tag

↓

Environment=Production

↓

Grupo Production
```

---

# Inventario Dinámico en Azure

La arquitectura es muy similar.

```text
Control Node

↓

Azure Plugin

↓

Azure API

↓

Virtual Machines

↓

Inventory
```

---

# Ejemplo

```yaml
plugin: azure.azcollection.azure_rm
```

---

# Inventario VMware

En muchos Data Centers la infraestructura continúa utilizando VMware.

Ansible puede descubrir automáticamente:

- Máquinas Virtuales.
- Datacenters.
- Clusters.
- Hosts ESXi.

---

# Arquitectura VMware

```text
Control Node

↓

VMware Plugin

↓

vCenter

↓

Máquinas Virtuales
```

---

# Plugin VMware

```yaml
plugin: community.vmware.vmware_vm_inventory
```

---

# Descubrimiento

Ansible obtiene:

- Nombre.
- IP.
- Estado.
- Datacenter.
- Cluster.
- Sistema Operativo.

---

# Inventario OpenStack

También existe soporte oficial.

```yaml
plugin: openstack.cloud.openstack
```

---

# Arquitectura

```text
Ansible

↓

OpenStack API

↓

Instances

↓

Inventory
```

---

# Inventario Kubernetes

En Kubernetes los Pods cambian constantemente.

El Inventory debe actualizarse automáticamente.

---

# Arquitectura

```text
Control Node

↓

Kubernetes API

↓

Namespaces

↓

Pods

↓

Inventory
```

---

# Plugin

```yaml
plugin: kubernetes.core.k8s
```

---

# Descubrimiento

Ansible puede descubrir:

- Pods.
- Nodes.
- Deployments.
- Services.

---

# Inventarios Construidos (Constructed)

No siempre necesitamos un proveedor Cloud.

A veces queremos reorganizar la información obtenida.

Para ello existe el plugin:

```text
constructed
```

---

# ¿Qué hace?

Permite construir nuevos grupos utilizando variables existentes.

---

# Ejemplo

Supongamos que obtenemos:

```text
Servidor

↓

Sistema Operativo

↓

Ubuntu
```

El plugin puede crear automáticamente

```text
ubuntu
```

como grupo.

---

# Otro Ejemplo

Servidor

```text
web01
```

Variable

```text
environment=production
```

Resultado

```text
Grupo Production
```

---

# Arquitectura

```text
Inventory

↓

Variables

↓

Constructed Plugin

↓

Nuevos Grupos
```

---

# Beneficios

No es necesario mantener grupos manualmente.

---

# Caché del Inventory

Consultar continuamente una API puede consumir tiempo.

Ansible permite almacenar resultados temporalmente.

```text
Cloud

↓

Consulta

↓

Cache

↓

Playbook
```

---

# Ventajas

- Mayor velocidad.
- Menor tráfico.
- Menor consumo de API.
- Menor carga sobre el proveedor.

---

# Actualización

Cuando sea necesario:

```text
Eliminar Cache

↓

Nueva Consulta

↓

Nuevo Inventory
```

---

# Arquitectura Híbrida

Muchas empresas poseen infraestructura mixta.

```text
On-Premises

+

AWS

+

Azure

+

VMware
```

Todos estos recursos pueden administrarse desde un mismo Control Node.

---

# Ejemplo

```text
Control Node

│

├── Inventory Local

├── AWS

├── Azure

├── VMware

└── Kubernetes
```

---

# Infraestructura Multi-Cloud

```text
                   Control Node

                        │

      ┌─────────────────┼──────────────────┐

      ▼                 ▼                  ▼

     AWS             Azure            VMware

      │                 │                  │

      └─────────────────┼──────────────────┘

                        ▼

                 Inventory Unificado
```

---

# Descubrimiento Automático

Cada vez que ejecutamos un Playbook:

```text
Playbook

↓

Plugin

↓

Consulta API

↓

Lista Actualizada

↓

Playbook
```

No es necesario editar archivos.

---

# Consideraciones de Seguridad

Los Plugins requieren credenciales para acceder a las APIs.

Nunca almacenar:

- Tokens.
- Passwords.
- Access Keys.
- Secret Keys.

En texto plano.

---

# Recomendación

Utilizar siempre:

- Ansible Vault.
- Variables de entorno.
- Servicios de gestión de secretos.

---

# Casos de Uso

Los Inventarios Dinámicos son ideales para:

- Kubernetes.
- OpenShift.
- AWS Auto Scaling.
- Azure Virtual Machine Scale Sets.
- VMware vSphere.
- OpenStack.
- Grandes Data Centers.

---

# Buenas Prácticas

- Utilizar Inventarios Dinámicos en Cloud.
- Mantener separados los ambientes DEV, QA y Producción.
- Utilizar etiquetas (Tags) consistentes.
- Aprovechar el Plugin Constructed.
- Habilitar caché cuando existan miles de servidores.
- Validar el Inventory antes de ejecutar Playbooks.
- Almacenar credenciales de forma segura.
- Documentar los Plugins utilizados.

---

# Errores Comunes

## Error 1

Actualizar manualmente un Inventory dinámico.

---

## Error 2

Guardar credenciales en texto plano.

---

## Error 3

No utilizar etiquetas consistentes en la nube.

---

## Error 4

No habilitar caché en infraestructuras grandes.

---

## Error 5

No validar el resultado del Plugin antes de ejecutar Playbooks.

---

## Error 6

Mezclar recursos de Producción y Desarrollo en el mismo grupo.

---

## Error 7

No controlar los permisos de acceso a las APIs del proveedor.

---

# Laboratorio RHCSA

> **Nota:** Los Inventarios Dinámicos no forman parte del examen RHCSA, pero conocerlos es muy útil para entornos empresariales y constituye una excelente base para RHCE y automatización avanzada.

## Laboratorio 1

Investigar los Plugins de Inventory disponibles mediante:

```bash
ansible-doc -t inventory -l
```

---

## Laboratorio 2

Identificar qué Plugin utilizarías para:

- AWS
- Azure
- VMware
- Kubernetes
- OpenStack

---

## Laboratorio 3

Diseñar una arquitectura híbrida que combine servidores locales y recursos en AWS.

---

## Laboratorio 4

Crear un diagrama donde un único Control Node administre recursos en:

- VMware
- AWS
- Azure

---

## Laboratorio 5

Explicar las ventajas del Plugin `constructed`.

---

## Laboratorio 6

Investigar cómo se organiza un Inventory utilizando etiquetas (Tags) en AWS.

---

## Laboratorio 7

Explicar cuándo es recomendable habilitar la caché del Inventory.

---

## Laboratorio 8

Diseñar una estrategia para administrar una infraestructura con más de 5,000 máquinas distribuidas entre varios proveedores.

---

## Laboratorio 9

Comparar un Inventory Estático y uno Dinámico indicando ventajas, desventajas y escenarios de uso.

---

## Laboratorio 10

Diseñar la arquitectura de una empresa con:

- 500 servidores On-Premises.
- 300 máquinas virtuales VMware.
- 250 instancias EC2 en AWS.
- 120 máquinas virtuales Azure.
- 10 clústeres Kubernetes.

Definir qué tipo de Inventory utilizarías para cada plataforma y justificar tu decisión.

---

# Resumen

En esta tercera fase estudiamos los **Inventarios Dinámicos**, los principales **Plugins de Inventory**, la integración con **AWS**, **Azure**, **VMware**, **OpenStack** y **Kubernetes**, así como el uso del plugin **Constructed** para generar grupos dinámicamente. También analizamos las ventajas de la caché del Inventory y las buenas prácticas para administrar infraestructuras híbridas y Multi-Cloud de forma automática y escalable.

En la **Fase 4** integraremos todos los conceptos del capítulo mediante escenarios empresariales, técnicas de troubleshooting, auditoría, validación de Inventarios, un laboratorio completo tipo RHCSA y un conjunto de preguntas de repaso para consolidar los conocimientos adquiridos.

----

# 80. Inventarios en Ansible (Fase 4)

> **Módulo 11 – Automatización con Ansible**
>
> **Archivo:** `80-inventarios-ansible.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Diagnosticar problemas relacionados con Inventarios.
- Auditar un Inventory antes de ejecutar Playbooks.
- Aplicar buenas prácticas empresariales.
- Diseñar Inventarios para grandes organizaciones.
- Comprender errores frecuentes.
- Realizar un laboratorio integral tipo RHCSA.
- Consolidar todos los conocimientos adquiridos sobre Inventarios.

---

# El Inventory como Fuente de Verdad

En una infraestructura automatizada, el Inventory representa la **fuente de verdad (Source of Truth)**.

Si el Inventory contiene información incorrecta:

- Los Playbooks fallarán.
- Se modificarán servidores equivocados.
- Se instalarán paquetes donde no corresponde.
- Se reiniciarán servicios incorrectos.
- Puede producirse una interrupción del servicio.

Por ello, antes de ejecutar cualquier Playbook, el Inventory debe ser validado cuidadosamente.

---

# Flujo Seguro de Automatización

```text
Modificar Inventory

        │

        ▼

Validar Sintaxis

        │

        ▼

Validar Variables

        │

        ▼

Probar SSH

        │

        ▼

Ejecutar --check

        │

        ▼

Ejecutar en Desarrollo

        │

        ▼

Ejecutar en QA

        │

        ▼

Producción
```

---

# Checklist de Validación

Antes de ejecutar cualquier automatización verifica:

```text
☑ Inventory correcto

☑ Hosts accesibles

☑ Variables completas

☑ Usuarios SSH válidos

☑ Claves SSH correctas

☑ Grupos correctamente definidos

☑ group_vars revisado

☑ host_vars revisado

☑ Playbook validado

☑ Backup disponible
```

---

# Validar el Inventory

Visualizar la estructura completa.

```bash
ansible-inventory --graph
```

Ejemplo

```text
@all

|--@production

|  |--@web

|  |   |--web01

|  |   |--web02

|  |

|  |--@database

|      |--db01

|      |--db02
```

---

# Visualizar Variables

```bash
ansible-inventory --list
```

Salida simplificada

```json
{

  "web01": {

      "ansible_host": "192.168.100.20",

      "http_port": 80

  }

}
```

---

# Consultar un Host Específico

```bash
ansible-inventory --host web01
```

Muy útil para comprobar qué variables recibirá exactamente ese servidor.

---

# Verificar Conectividad

```bash
ansible all -m ping
```

Resultado esperado

```text
web01 | SUCCESS

web02 | SUCCESS

db01 | SUCCESS
```

---

# Verificar un Grupo

```bash
ansible web -m ping
```

Solo comprobará los servidores pertenecientes al grupo **web**.

---

# Error: Host no encontrado

```text
ERROR!

No hosts matched
```

### Posibles causas

- Nombre incorrecto.
- Grupo inexistente.
- Error de escritura.
- Inventory equivocado.

---

# Diagnóstico

Comprobar el Inventory.

```bash
ansible-inventory --graph
```

---

# Error: Inventory vacío

```text
No inventory was parsed
```

---

### Posibles causas

- Archivo inexistente.
- Ruta incorrecta.
- Sintaxis inválida.
- ansible.cfg mal configurado.

---

# Verificar la Configuración

```bash
ansible --version
```

Ejemplo

```text
config file =

/etc/ansible/ansible.cfg
```

Verificar que el Inventory indicado exista realmente.

---

# Error: Host duplicado

Ejemplo incorrecto

```ini
web01

web01
```

Puede producir resultados inesperados.

---

# Error: Variables duplicadas

Ejemplo

```text
group_vars/web.yml

↓

http_port=80
```

y además

```text
host_vars/web01.yml

↓

http_port=8080
```

Esto no genera un error, pero es importante conocer la precedencia para evitar confusión.

---

# Error: Dirección IP incorrecta

```ini
web01

ansible_host=192.168.10.50
```

Cuando la dirección correcta es

```text
192.168.100.50
```

La conexión fallará.

---

# Error: Usuario SSH incorrecto

```yaml
ansible_user: administrador
```

Cuando el servidor solo permite

```yaml
ansible_user: ansible
```

Resultado

```text
Permission denied
```

---

# Error: Puerto SSH incorrecto

```yaml
ansible_port: 2222
```

Cuando realmente el servidor utiliza

```text
22
```

---

# Diagnóstico Manual

Comprobar primero mediante SSH.

```bash
ssh ansible@web01
```

Si SSH falla, Ansible también fallará.

---

# Error: DNS

```text
Could not resolve hostname
```

Verificar

```bash
host web01
```

o

```bash
nslookup web01
```

---

# Error: Firewall

Comprobar puerto SSH.

```bash
nc -zv web01 22
```

---

# Error: Variables Inexistentes

```text
Undefined variable
```

Comprobar:

```text
group_vars

host_vars

vars

defaults
```

---

# Flujo de Diagnóstico

```text
Error

↓

Inventory

↓

Host

↓

SSH

↓

Variables

↓

Playbook

↓

Resultado
```

---

# Auditoría del Inventory

Un buen Inventory debe responder preguntas como:

- ¿Qué servidores existen?
- ¿Quién los administra?
- ¿A qué ambiente pertenecen?
- ¿Qué sistema operativo utilizan?
- ¿Qué función cumplen?

---

# Ejemplo de Documentación

| Host | Función | Ambiente | Responsable |
|------|----------|-----------|-------------|
| web01 | Apache | Producción | Infraestructura |
| web02 | Apache | Producción | Infraestructura |
| db01 | PostgreSQL | Producción | DBA |
| monitor01 | Prometheus | Producción | Observabilidad |

---

# Inventario Empresarial

```text
Producción

│

├── Web

├── Database

├── DNS

├── Proxy

├── Monitoring

├── Backup

└── Security
```

Cada grupo posee sus propias variables.

---

# Separación por Ambientes

```text
inventories/

├── development/

├── testing/

├── qa/

├── production/

└── disaster_recovery/
```

Nunca utilizar el mismo Inventory para todos los ambientes.

---

# Convenciones de Nombres

### Correcto

```text
web01

web02

db01

dns01

backup01
```

---

### Incorrecto

```text
equipo1

hostA

servidorNuevo

prueba2
```

Los nombres deben describir claramente la función del servidor.

---

# Integración con Git

Todo cambio en el Inventory debe registrarse.

```text
Editar Inventory

↓

Git Commit

↓

Pull Request

↓

Revisión

↓

Merge

↓

Producción
```

---

# Control de Cambios

Nunca modificar directamente el Inventory de Producción.

Proceso recomendado:

```text
Development

↓

QA

↓

Producción
```

---

# Seguridad

Evitar almacenar:

```text
Passwords

Tokens

API Keys

Private Keys
```

Dentro del Inventory.

Utilizar:

- Ansible Vault.
- Variables cifradas.
- Gestores de secretos.

---

# Inventarios Escalables

Ejemplo

```text
Empresa

↓

2,500 Servidores

↓

20 Grupos

↓

150 Variables

↓

Un único Proyecto Ansible
```

Gracias a una estructura organizada, el mantenimiento continúa siendo sencillo.

---

# Arquitectura Empresarial Completa

```text
                      Git

                       │

                       ▼

               Proyecto Ansible

                       │

         ┌─────────────┼─────────────┐

         ▼             ▼             ▼

   Inventories    group_vars    host_vars

         │             │             │

         └─────────────┼─────────────┘

                       ▼

                  Playbooks

                       ▼

                SSH / WinRM

                       ▼

                Infraestructura

        ┌────────┬────────┬─────────┐

        ▼        ▼        ▼         ▼

      Web      DB      Proxy     Backup
```

---

# Buenas Prácticas

- Mantener un Inventory por ambiente.
- Validar el Inventory antes de cada despliegue.
- Utilizar nombres descriptivos.
- Documentar todos los grupos.
- Mantener variables organizadas.
- Utilizar `group_vars` para configuraciones comunes.
- Utilizar `host_vars` únicamente para excepciones.
- Mantener todo el proyecto bajo Git.
- Automatizar la validación del Inventory.
- Probar siempre con `--check` antes de ejecutar cambios.

---

# Errores Comunes

## Error 1

Utilizar direcciones IP en lugar de nombres descriptivos.

---

## Error 2

Duplicar Hosts.

---

## Error 3

Duplicar variables.

---

## Error 4

No documentar el Inventory.

---

## Error 5

Modificar Producción directamente.

---

## Error 6

Guardar contraseñas en texto plano.

---

## Error 7

No revisar la salida de:

```bash
ansible-inventory --graph
```

---

## Error 8

No probar conectividad antes de ejecutar un Playbook.

---

## Error 9

Mezclar diferentes funciones en un mismo grupo.

---

## Error 10

No separar Development, QA y Producción.

---

# Laboratorio Integral RHCSA

## Escenario

Una empresa posee la siguiente infraestructura:

```text
15 Web Servers

8 PostgreSQL

6 SQL Server

4 DNS

4 Balanceadores

3 Proxy

5 Backup

6 Monitoreo

2 Bastion Hosts
```

---

## Requerimientos

Construir un proyecto Ansible que incluya:

- Inventarios por ambiente.
- Variables globales.
- Variables por grupo.
- Variables específicas por Host.
- Integración con Git.
- Validación del Inventory.
- Conectividad SSH.
- Documentación.

---

# Ejercicio 1

Crear la siguiente estructura:

```text
ansible-project/

├── ansible.cfg

├── inventories/

│   ├── development/

│   ├── qa/

│   ├── production/

│   └── disaster_recovery/

├── group_vars/

├── host_vars/

├── playbooks/

├── roles/

├── templates/

├── files/

└── documentation/
```

---

# Ejercicio 2

Diseñar el Inventory para administrar:

- 50 servidores Web.
- 20 PostgreSQL.
- 10 SQL Server.
- 8 DNS.
- 5 Balanceadores.
- 10 Servidores de Monitoreo.

---

# Ejercicio 3

Crear grupos hijos:

```text
production

↓

web

↓

database

↓

monitoring

↓

backup
```

---

# Ejercicio 4

Crear:

```text
group_vars/all.yml
```

Definir:

- Zona horaria.
- DNS.
- Servidor NTP.
- Dominio.
- Proxy corporativo.

---

# Ejercicio 5

Crear:

```text
host_vars/web01.yml
```

Asignar:

- Dirección IP virtual.
- Prioridad.
- Identificador del servidor.

---

# Ejercicio 6

Validar:

```bash
ansible-inventory --graph
```

---

# Ejercicio 7

Mostrar:

```bash
ansible-inventory --list
```

---

# Ejercicio 8

Comprobar conectividad:

```bash
ansible all -m ping
```

---

# Ejercicio 9

Documentar el propósito de cada grupo y las variables que contiene.

---

# Ejercicio 10

Simular un cambio en Producción utilizando:

```bash
ansible-playbook sitio.yml --check --diff
```

Registrar los resultados obtenidos.

---

# Preguntas de Repaso

1. ¿Qué es un Inventory en Ansible?

2. ¿Cuál es la diferencia entre un Inventory estático y uno dinámico?

3. ¿Qué ventajas ofrece el formato YAML frente al formato INI?

4. ¿Qué función cumple `group_vars`?

5. ¿Cuándo utilizar `host_vars`?

6. ¿Qué representa el grupo `all`?

7. ¿Qué representa el grupo `ungrouped`?

8. ¿Qué hace `ansible-inventory --graph`?

9. ¿Qué hace `ansible-inventory --list`?

10. ¿Qué utilidad tiene `ansible-inventory --host`?

11. ¿Qué variables de conexión son las más utilizadas?

12. ¿Qué es un Inventory Plugin?

13. ¿Qué ventajas ofrecen los Inventarios Dinámicos?

14. ¿Por qué es recomendable separar Development, QA y Producción?

15. ¿Por qué un Inventory bien diseñado reduce el riesgo de errores en la automatización?

---

# Resumen del Capítulo

En este capítulo estudiamos uno de los pilares fundamentales de Ansible: los **Inventarios**. Aprendimos a crear Inventarios estáticos en formato **INI** y **YAML**, organizar servidores mediante grupos y grupos hijos, utilizar `group_vars` y `host_vars`, administrar variables de conexión, comprender la precedencia de variables e implementar Inventarios para múltiples ambientes.

También exploramos los **Inventarios Dinámicos**, los principales Plugins de Inventory para plataformas como **AWS**, **Azure**, **VMware**, **OpenStack** y **Kubernetes**, así como las mejores prácticas para infraestructuras híbridas y de gran escala.

Finalmente, revisamos técnicas de validación, troubleshooting, auditoría y diseño empresarial, dejando preparado el entorno para comenzar a ejecutar comandos y Playbooks sobre Inventarios bien estructurados.

---

# Próximo Capítulo

## **81. Comandos Ad-Hoc en Ansible**

En el siguiente capítulo aprenderemos a utilizar los **Comandos Ad-Hoc**, una de las herramientas más potentes de Ansible para ejecutar tareas rápidas sin necesidad de crear Playbooks. Exploraremos módulos como `ping`, `command`, `shell`, `copy`, `file`, `service`, `dnf`, `user`, `cron`, `reboot` y muchos otros, aplicándolos en escenarios reales de administración de sistemas Linux.











