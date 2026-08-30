# 79. Instalación y Configuración de Ansible (Fase 1)

> **Módulo 11 – Automatización con Ansible**
>
> **Archivo:** `79-instalacion-configuracion-ansible.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Comprender las diferentes ediciones de Ansible.
- Instalar Ansible Core en Fedora, RHEL, Rocky y AlmaLinux.
- Verificar la instalación.
- Conocer todos los componentes instalados.
- Comprender la estructura de directorios de Ansible.
- Identificar los archivos de configuración.
- Preparar un Control Node para producción.

---

# Introducción

Después de comprender los conceptos básicos de Ansible, el siguiente paso consiste en preparar correctamente el **Control Node**, que será el servidor encargado de administrar toda la infraestructura.

Una instalación correcta evitará la mayoría de los problemas que suelen aparecer durante la automatización.

---

# ¿Qué se instala realmente?

Cuando instalamos Ansible no solamente obtenemos un único programa.

Se instala un conjunto completo de herramientas.

```text
                 Ansible

                     │

     ┌───────────────┼────────────────┐

     ▼               ▼                ▼

 ansible      ansible-playbook   ansible-doc

     ▼               ▼                ▼

ansible-vault ansible-config ansible-galaxy
```

---

# Ansible Core vs Ansible Community

Actualmente existen varias distribuciones de Ansible.

| Edición | Descripción |
|----------|-------------|
| ansible-core | Núcleo oficial mantenido por Red Hat |
| ansible | Paquete comunitario con múltiples colecciones |
| Red Hat Ansible Automation Platform | Producto empresarial con soporte comercial |

Para RHCSA trabajaremos con **ansible-core**.

---

# Sistemas Operativos Compatibles

Ansible puede instalarse sobre:

- Fedora
- Red Hat Enterprise Linux
- Rocky Linux
- AlmaLinux
- Ubuntu
- Debian
- SUSE
- macOS

El Control Node normalmente ejecuta Linux.

---

# Requisitos del Control Node

Mínimos recomendados

| Recurso | Recomendado |
|----------|-------------|
| CPU | 2 Núcleos |
| RAM | 4 GB |
| Disco | 20 GB |
| Python | 3.x |
| SSH | Instalado |

En ambientes empresariales estos recursos suelen ser mayores.

---

# Actualizar el sistema

Antes de instalar cualquier software es recomendable actualizar los paquetes.

Fedora

```bash
sudo dnf update -y
```

RHEL

```bash
sudo dnf update -y
```

---

# Buscar el paquete

```bash
dnf search ansible
```

Resultado esperado

```text
ansible-core

ansible

ansible-collection-*
```

---

# Instalar Ansible Core

```bash
sudo dnf install ansible-core -y
```

---

# Instalar la versión Community

```bash
sudo dnf install ansible -y
```

---

# Verificar la instalación

```bash
ansible --version
```

Ejemplo

```text
ansible [core 2.18.x]

python version = 3.12

jinja version = 3.x

libyaml = True
```

---

# Información obtenida

El comando anterior muestra:

- Versión de Ansible.
- Versión de Python.
- Archivo de configuración utilizado.
- Ruta de módulos.
- Ruta de Collections.
- Motor YAML.

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

# Confirmar instalación mediante RPM

```bash
rpm -qi ansible-core
```

---

# Consultar todos los archivos instalados

```bash
rpm -ql ansible-core
```

---

# Componentes principales

Después de la instalación estarán disponibles:

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

Ejecuta comandos Ad-Hoc.

```bash
ansible all -m ping
```

---

# ansible-playbook

Ejecuta Playbooks completos.

```bash
ansible-playbook sitio.yml
```

---

# ansible-doc

Permite consultar la documentación oficial instalada.

```bash
ansible-doc service
```

---

# ansible-config

Permite visualizar la configuración.

```bash
ansible-config dump
```

---

# ansible-vault

Protege información sensible.

```bash
ansible-vault create secretos.yml
```

---

# ansible-galaxy

Administra Roles y Collections.

```bash
ansible-galaxy collection list
```

---

# ansible-inventory

Muestra el contenido del Inventory.

```bash
ansible-inventory --list
```

---

# Verificar Python

```bash
python3 --version
```

---

# Verificar OpenSSH

```bash
ssh -V
```

---

# Verificar conectividad

```bash
ping 192.168.100.20
```

---

# Verificar resolución DNS

```bash
host web01
```

o

```bash
nslookup web01
```

---

# Directorios importantes

Los más utilizados son:

```text
/etc/ansible/

/usr/bin/

/usr/share/ansible/

/usr/lib/python3/

/usr/share/doc/
```

---

# Directorio /etc/ansible

Generalmente contiene:

```text
/etc/ansible

├── ansible.cfg

├── hosts

└── roles/
```

---

# Archivo hosts

Es el Inventory predeterminado.

```text
/etc/ansible/hosts
```

---

# Archivo ansible.cfg

Contiene la configuración global.

```text
/etc/ansible/ansible.cfg
```

Más adelante estudiaremos todas sus opciones.

---

# Jerarquía de configuración

Cuando Ansible inicia busca su configuración en el siguiente orden:

```text
ANSIBLE_CONFIG

↓

ansible.cfg

↓

~/.ansible.cfg

↓

/etc/ansible/ansible.cfg
```

La primera que encuentre será utilizada.

---

# Comprobar el archivo utilizado

```bash
ansible --version
```

Ejemplo

```text
config file = /etc/ansible/ansible.cfg
```

---

# Arquitectura del Control Node

```text
               Administrador

                     │

                     ▼

              Control Node

                     │

    ┌────────────────┼─────────────────┐

    ▼                ▼                 ▼

Inventory      ansible.cfg       Playbooks

                     │

                     ▼

                   SSH

                     │

        ┌────────────┼────────────┐

        ▼            ▼            ▼

      Web01        DB01       Backup01
```

---

# Buenas prácticas

- Mantener el sistema actualizado.
- Instalar únicamente ansible-core cuando sea suficiente.
- Utilizar versiones soportadas de Python.
- Mantener sincronizada la hora mediante NTP.
- Documentar la versión instalada.
- Verificar la configuración después de cada actualización.

---

# Errores comunes

## Error 1

Instalar una versión incompatible de Python.

---

## Error 2

No actualizar los repositorios antes de instalar.

---

## Error 3

Modificar archivos del sistema sin respaldo.

---

## Error 4

Confundir ansible con ansible-core.

---

## Error 5

No verificar la versión instalada.

---

# Laboratorio RHCSA

## Laboratorio 1

Actualizar Fedora.

---

## Laboratorio 2

Instalar ansible-core.

---

## Laboratorio 3

Verificar la versión instalada.

---

## Laboratorio 4

Consultar todos los binarios disponibles.

---

## Laboratorio 5

Mostrar la información del paquete RPM.

---

## Laboratorio 6

Identificar el archivo de configuración utilizado.

---

## Laboratorio 7

Explorar el directorio `/etc/ansible`.

---

## Laboratorio 8

Consultar la ayuda de `ansible-doc`.

---

## Laboratorio 9

Listar las Collections instaladas.

---

## Laboratorio 10

Documentar la arquitectura del Control Node utilizada en tu laboratorio.

---

# Resumen

En esta primera fase instalamos correctamente **ansible-core**, conocimos todos los componentes principales, revisamos la estructura de directorios, identificamos el archivo de configuración principal y preparamos el Control Node para continuar con la configuración avanzada.

En la **Fase 2** estudiaremos en profundidad el archivo **ansible.cfg**, todas sus opciones de configuración, la autenticación mediante SSH, el uso de usuarios administrativos (`become`) y la preparación completa de un entorno profesional de automatización.
----

# 79. Instalación y Configuración de Ansible (Fase 2)

> **Módulo 11 – Automatización con Ansible**
>
> **Archivo:** `79-instalacion-configuracion-ansible.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Comprender el funcionamiento del archivo `ansible.cfg`.
- Configurar un entorno profesional de Ansible.
- Personalizar el comportamiento del Control Node.
- Configurar autenticación mediante SSH.
- Configurar el uso de `become`.
- Entender la precedencia de configuración.
- Validar el entorno antes de comenzar la automatización.

---

# ¿Qué es ansible.cfg?

El archivo **ansible.cfg** contiene la configuración global de Ansible.

Desde este archivo es posible controlar prácticamente todo el comportamiento del sistema.

Ejemplos:

- Inventario por defecto.
- Usuario remoto.
- Tiempo de espera.
- Cantidad de conexiones paralelas.
- Uso de sudo.
- Ruta de Roles.
- Ruta de Collections.
- Logging.
- Deshabilitar comprobación de claves SSH.

---

# ¿Dónde busca Ansible este archivo?

Cuando ejecutamos cualquier comando, Ansible busca la configuración en el siguiente orden.

```text
                ANSIBLE_CONFIG
                      │
                      ▼
             ./ansible.cfg
                      │
                      ▼
           ~/.ansible.cfg
                      │
                      ▼
      /etc/ansible/ansible.cfg
```

La primera configuración encontrada será utilizada.

---

# Verificar el archivo utilizado

```bash
ansible --version
```

Ejemplo

```text
ansible [core 2.18]

config file = /etc/ansible/ansible.cfg
```

---

# Ver configuración completa

```bash
ansible-config dump
```

---

# Mostrar únicamente los cambios

```bash
ansible-config dump --only-changed
```

Muy útil para auditorías.

---

# Ver ayuda

```bash
ansible-config list
```

---

# Generar un archivo base

```bash
ansible-config init --disabled > ansible.cfg
```

Genera un archivo comentado con todas las opciones disponibles.

---

# Estructura general

```ini
[defaults]

inventory = inventory.ini

remote_user = ansible

forks = 20

timeout = 30
```

---

# Sección [defaults]

Es la sección utilizada con mayor frecuencia.

Aquí se configuran los parámetros generales.

---

# inventory

Define el Inventory predeterminado.

```ini
[defaults]

inventory = inventories/production.ini
```

Ahora ya no será necesario indicar:

```bash
-i inventory.ini
```

---

# remote_user

Usuario utilizado para conectarse mediante SSH.

```ini
remote_user = ansible
```

---

# Ejemplo

Sin configurar

```bash
ansible all -u administrador
```

Con `remote_user`

```bash
ansible all
```

---

# forks

Número de servidores administrados simultáneamente.

```ini
forks = 20
```

---

# Ejemplo

```text
forks = 5

Servidor1

Servidor2

Servidor3

Servidor4

Servidor5
```

Los demás esperan su turno.

---

# timeout

Tiempo máximo de espera.

```ini
timeout = 30
```

---

# host_key_checking

Por defecto

```ini
host_key_checking = True
```

En laboratorios suele utilizarse

```ini
host_key_checking = False
```

---

# ¿Qué hace?

Cuando es **True**

```text
¿Confía en este servidor?

yes/no
```

Cuando es **False**

No pregunta.

---

# Advertencia

En producción es recomendable mantener

```ini
host_key_checking = True
```

Para evitar ataques de tipo **Man in the Middle (MITM)**.

---

# retry_files_enabled

Por defecto Ansible crea archivos `.retry`.

Pueden deshabilitarse.

```ini
retry_files_enabled = False
```

---

# log_path

Guardar todas las ejecuciones.

```ini
log_path = /var/log/ansible.log
```

---

# stdout_callback

Permite modificar el formato de salida.

Ejemplo

```ini
stdout_callback = yaml
```

---

# interpreter_python

Especifica el intérprete de Python.

```ini
interpreter_python = auto
```

---

# Configuración recomendada

```ini
[defaults]

inventory = inventories/production.ini

remote_user = ansible

forks = 20

timeout = 30

host_key_checking = True

retry_files_enabled = False

interpreter_python = auto

log_path = /var/log/ansible.log
```

---

# become

Muchas tareas requieren privilegios administrativos.

Ansible utiliza:

```text
become
```

Equivalente a:

```bash
sudo
```

---

# become en Playbooks

```yaml
become: true
```

---

# become_method

Por defecto

```ini
become_method = sudo
```

Otros métodos

- su
- doas
- pfexec

---

# become_user

```ini
become_user = root
```

---

# Ejemplo

```yaml
---

hosts: web

become: true

tasks:

- name: Instalar Apache

  dnf:

    name: httpd

    state: present
```

---

# Configuración SSH

Ansible utiliza OpenSSH.

Puede configurarse.

```ini
[ssh_connection]
```

---

# pipelining

Reduce el número de conexiones SSH.

```ini
pipelining = True
```

---

# Beneficios

```text
Menos conexiones

↓

Mayor velocidad

↓

Menor consumo
```

---

# ControlPersist

OpenSSH reutiliza conexiones.

```text
Primera conexión

↓

Autenticación

↓

Conexión reutilizada

↓

Mayor rendimiento
```

---

# SSH por claves

Siempre recomendado.

```text
Control Node

↓

Llave Privada

↓

SSH

↓

Servidor Linux

↓

Llave Pública
```

---

# Generar claves

```bash
ssh-keygen -t ed25519
```

---

# Copiar la llave

```bash
ssh-copy-id usuario@web01
```

---

# Verificar

```bash
ssh usuario@web01
```

No debe solicitar contraseña.

---

# Archivo ~/.ssh/config

Permite simplificar conexiones.

Ejemplo

```text
Host web01

HostName 192.168.100.20

User ansible

IdentityFile ~/.ssh/id_ed25519
```

Ahora bastará con ejecutar

```bash
ssh web01
```

---

# Variables de entorno

También es posible modificar la configuración temporalmente.

Ejemplo

```bash
export ANSIBLE_CONFIG=~/ansible.cfg
```

---

# Verificar la variable

```bash
echo $ANSIBLE_CONFIG
```

---

# Prioridad de configuración

```text
Variables de Entorno

↓

Opciones de Línea de Comandos

↓

Playbooks

↓

ansible.cfg

↓

Valores por defecto
```

---

# Proyecto recomendado

```text
ansible-project/

├── ansible.cfg

├── inventories/

├── playbooks/

├── roles/

├── group_vars/

├── host_vars/

├── templates/

├── files/

└── collections/
```

---

# Flujo profesional

```text
Administrador

↓

Git

↓

ansible.cfg

↓

Inventory

↓

SSH

↓

Playbook

↓

Servidor
```

---

# Validar configuración

```bash
ansible-config dump --only-changed
```

Debe mostrar únicamente las opciones personalizadas.

---

# Verificar inventario

```bash
ansible-inventory --graph
```

---

# Probar conectividad

```bash
ansible all -m ping
```

---

# Buenas prácticas

- Mantener un único `ansible.cfg` por proyecto.
- Versionar el archivo en Git.
- Utilizar autenticación mediante claves SSH.
- Habilitar logging en producción.
- Mantener `host_key_checking=True` en ambientes productivos.
- Ajustar `forks` según la capacidad del Control Node.
- Utilizar `become` en lugar de iniciar sesión como root.

---

# Errores comunes

## Error 1

Modificar el archivo equivocado.

---

## Error 2

Desactivar permanentemente `host_key_checking` en producción.

---

## Error 3

No revisar qué archivo `ansible.cfg` está utilizando Ansible.

---

## Error 4

No habilitar registros de auditoría.

---

## Error 5

Utilizar demasiados `forks` en un servidor con pocos recursos.

---

## Error 6

Conectarse como root en todos los servidores.

---

# Laboratorio RHCSA

## Laboratorio 1

Crear un archivo `ansible.cfg` dentro del proyecto.

---

## Laboratorio 2

Configurar el Inventory por defecto.

---

## Laboratorio 3

Configurar `remote_user`.

---

## Laboratorio 4

Configurar `forks=20`.

---

## Laboratorio 5

Habilitar el registro de auditoría mediante `log_path`.

---

## Laboratorio 6

Configurar autenticación por claves SSH.

---

## Laboratorio 7

Verificar el archivo de configuración activo.

---

## Laboratorio 8

Mostrar únicamente las opciones modificadas.

---

## Laboratorio 9

Ejecutar `ansible all -m ping` sin especificar el Inventory.

---

## Laboratorio 10

Documentar cada parámetro utilizado en tu `ansible.cfg` indicando su propósito y el impacto que tendría una configuración incorrecta.

---

# Resumen

En esta segunda fase aprendimos a configurar el archivo **ansible.cfg**, comprendimos la jerarquía de configuración de Ansible, configuramos el Inventory por defecto, el usuario remoto, el número de conexiones paralelas, la autenticación mediante SSH, el uso de `become` y las mejores prácticas para preparar un entorno profesional.

En la **Fase 3** profundizaremos en la organización del proyecto Ansible, el manejo de inventarios avanzados, `group_vars`, `host_vars`, configuraciones específicas por entorno y técnicas utilizadas en infraestructuras empresariales de gran escala.

----

# 79. Instalación y Configuración de Ansible (Fase 3)

> **Módulo 11 – Automatización con Ansible**
>
> **Archivo:** `79-instalacion-configuracion-ansible.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Organizar un proyecto profesional de Ansible.
- Comprender la estructura recomendada para ambientes empresariales.
- Utilizar múltiples Inventories.
- Configurar `group_vars` y `host_vars`.
- Administrar múltiples ambientes (Development, QA y Production).
- Comprender cómo Ansible localiza archivos.
- Preparar un proyecto listo para crecer mediante Roles.

---

# Organización Profesional

A medida que una infraestructura crece, resulta indispensable organizar correctamente el proyecto.

No es recomendable almacenar todos los archivos en un mismo directorio.

Una estructura organizada facilita:

- El mantenimiento.
- La reutilización.
- El trabajo en equipo.
- La auditoría.
- El control de versiones.

---

# Proyecto Empresarial

```text
ansible-project/

├── ansible.cfg
├── inventories/
├── playbooks/
├── group_vars/
├── host_vars/
├── roles/
├── templates/
├── files/
├── collections/
├── plugins/
├── scripts/
├── documentation/
└── README.md
```

---

# Arquitectura del Proyecto

```text
                  Proyecto

                     │

    ┌────────────────┼────────────────┐

    ▼                ▼                ▼

Inventories      Playbooks         Roles

    ▼                ▼                ▼

Variables     Templates       Archivos

                     │

                     ▼

                 Managed Nodes
```

---

# Directorio inventories

Aquí se almacenan todos los Inventories.

```text
inventories/

├── development
├── testing
├── qa
└── production
```

---

# ¿Por qué separar Inventories?

Cada ambiente posee características diferentes.

```text
Development

↓

Pruebas

↓

QA

↓

Producción
```

Nunca deberían compartir exactamente la misma configuración.

---

# Ejemplo

```text
inventories/

├── dev.ini

├── qa.ini

└── prod.ini
```

---

# Ejemplo de Inventory

```ini
[web]

web01

web02

web03

[database]

db01

db02
```

---

# Directorio Playbooks

Aquí se almacenan todos los Playbooks.

```text
playbooks/

├── apache.yml

├── users.yml

├── updates.yml

├── firewall.yml

└── backup.yml
```

Cada Playbook debe cumplir un único propósito.

---

# Directorio files

Contiene archivos que serán copiados mediante Ansible.

Ejemplo

```text
files/

├── motd

├── banner.txt

├── logo.png

└── certificados/
```

---

# Directorio templates

Contiene archivos Jinja2.

```text
templates/

├── httpd.conf.j2

├── nginx.conf.j2

├── sshd_config.j2

└── index.html.j2
```

Estos archivos permiten generar configuraciones dinámicas.

---

# Directorio roles

Aquí se almacenan los Roles.

```text
roles/

├── apache/

├── nginx/

├── postgres/

├── mysql/

└── firewall/
```

Los Roles serán estudiados en profundidad más adelante.

---

# Directorio documentation

Es recomendable documentar:

- Inventarios.
- Variables.
- Arquitectura.
- Procedimientos.
- Cambios.

Ejemplo

```text
documentation/

README.md

Arquitectura.md

Inventario.md

Cambios.md
```

---

# Directorio scripts

Puede contener utilidades auxiliares.

Ejemplo

```text
scripts/

backup.sh

cleanup.sh

healthcheck.sh
```

---

# Directorio collections

Aquí se instalan Collections adicionales.

```text
collections/

ansible_collections/

community/

containers/
```

---

# group_vars

Permite definir variables para grupos completos.

```text
group_vars/

├── all.yml

├── web.yml

├── database.yml

└── backup.yml
```

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

service_name: httpd

document_root: /var/www/html
```

Todos los servidores del grupo **web** heredarán estas variables.

---

# host_vars

Permite definir variables específicas para un único servidor.

```text
host_vars/

├── web01.yml

├── web02.yml

└── db01.yml
```

---

# Ejemplo

```yaml
---

ip_virtual: 192.168.100.200

priority: 100
```

Sólo afectará a ese servidor.

---

# Jerarquía de Variables

Cuando una misma variable aparece en varios lugares, Ansible aplica una prioridad.

Ejemplo simplificado:

```text
host_vars

↓

group_vars

↓

Playbook

↓

Variables por defecto
```

La variable más específica tiene prioridad.

---

# Organización por Ambiente

```text
inventories/

├── development

│   ├── hosts

│   └── group_vars/

├── qa

│   ├── hosts

│   └── group_vars/

└── production

    ├── hosts

    └── group_vars/
```

Cada ambiente mantiene su propia configuración.

---

# Ejemplo Empresarial

```text
Empresa

│

├── Desarrollo

├── QA

├── Producción

└── Disaster Recovery
```

Cada uno utiliza un Inventory independiente.

---

# Uso de Git

Toda la configuración debería almacenarse en Git.

```text
Repositorio

│

├── Historial

├── Versiones

├── Branches

└── Pull Requests
```

---

# Flujo recomendado

```text
Crear Playbook

↓

Git Commit

↓

Code Review

↓

Merge

↓

QA

↓

Producción
```

---

# Convenciones de nombres

Utilizar nombres descriptivos.

Correcto

```text
install_apache.yml

configure_firewall.yml

create_users.yml
```

Incorrecto

```text
test.yml

nuevo.yml

archivo.yml
```

---

# Organización de Variables

Agrupar variables relacionadas.

Correcto

```yaml
apache:

  port: 80

  service: httpd

  root: /var/www/html
```

Evita crear cientos de variables dispersas.

---

# Estructura Escalable

```text
Proyecto

↓

Inventories

↓

Playbooks

↓

Roles

↓

Templates

↓

Variables

↓

Producción
```

---

# Validación del Proyecto

Verificar la sintaxis.

```bash
ansible-playbook playbooks/apache.yml --syntax-check
```

---

# Verificar Inventory

```bash
ansible-inventory --graph
```

---

# Mostrar Variables

```bash
ansible-inventory --list
```

---

# Comprobar conectividad

```bash
ansible all -m ping
```

---

# Documentación

Todo proyecto profesional debería incluir un README.

Ejemplo

```text
Proyecto

Objetivo

Requisitos

Inventarios

Playbooks

Variables

Dependencias

Procedimiento de ejecución
```

---

# Arquitectura Empresarial

```text
                    Git

                     │

                     ▼

             Proyecto Ansible

                     │

      ┌──────────────┼──────────────┐

      ▼              ▼              ▼

 Inventories     Playbooks      Variables

                     │

                     ▼

                 SSH

                     │

      ┌──────────────┼──────────────┐

      ▼              ▼              ▼

   Web01          DB01          Backup01
```

---

# Buenas prácticas

- Mantener un proyecto por infraestructura.
- Utilizar Git para todos los cambios.
- Separar Development, QA y Production.
- Utilizar `group_vars`.
- Utilizar `host_vars` únicamente cuando sea necesario.
- Mantener los Playbooks pequeños.
- Documentar todos los cambios.
- Mantener nombres consistentes.
- Evitar duplicación de variables.

---

# Errores comunes

## Error 1

Guardar todos los Playbooks en un único directorio.

---

## Error 2

No utilizar `group_vars`.

---

## Error 3

Duplicar variables en múltiples archivos.

---

## Error 4

Utilizar un único Inventory para todos los ambientes.

---

## Error 5

No documentar el proyecto.

---

## Error 6

No utilizar Git.

---

## Error 7

Crear estructuras diferentes para cada proyecto.

Siempre es recomendable seguir un estándar.

---

# Laboratorio RHCSA

## Laboratorio 1

Crear la estructura completa del proyecto.

---

## Laboratorio 2

Crear tres Inventories:

- Development
- QA
- Production

---

## Laboratorio 3

Crear un directorio `group_vars`.

---

## Laboratorio 4

Crear un directorio `host_vars`.

---

## Laboratorio 5

Agregar variables para el grupo **web**.

---

## Laboratorio 6

Crear variables específicas para **web01**.

---

## Laboratorio 7

Mover todos los Playbooks al directorio correspondiente.

---

## Laboratorio 8

Inicializar un repositorio Git para el proyecto.

---

## Laboratorio 9

Crear un archivo `README.md` documentando la estructura.

---

## Laboratorio 10

Diseñar una arquitectura empresarial para administrar:

- 20 servidores Web.
- 10 Bases de Datos.
- 5 Servidores DNS.
- 5 Servidores de Monitoreo.
- 3 Servidores de Respaldo.

Organizar todos los directorios siguiendo las buenas prácticas estudiadas.

---

# Resumen

En esta tercera fase aprendimos cómo organizar un proyecto profesional de Ansible. Estudiamos la estructura recomendada de directorios, el uso de múltiples Inventories, la configuración mediante `group_vars` y `host_vars`, la integración con Git y las mejores prácticas para construir proyectos escalables y fáciles de mantener.

En la **Fase 4** integraremos todos estos conceptos en un escenario empresarial completo. Aprenderemos técnicas avanzadas de troubleshooting, auditoría, validación, despliegues seguros y resolveremos un laboratorio integral similar al que podría encontrarse en un entorno de producción o en una certificación RHCSA.

-----

# 79. Instalación y Configuración de Ansible (Fase 4)

> **Módulo 11 – Automatización con Ansible**
>
> **Archivo:** `79-instalacion-configuracion-ansible.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Implementar un entorno empresarial de Ansible.
- Validar la configuración antes de automatizar.
- Diagnosticar problemas frecuentes.
- Aplicar buenas prácticas de seguridad.
- Preparar el Control Node para producción.
- Realizar un laboratorio completo tipo RHCSA.

---

# Escenario Empresarial

Supongamos que una empresa posee la siguiente infraestructura.

```text
                        Empresa

                            │

        ┌───────────────────┼───────────────────┐

        ▼                   ▼                   ▼

   Desarrollo             QA              Producción

        │                   │                   │

   10 Servidores      20 Servidores      120 Servidores
```

Todos estos servidores serán administrados desde un único Control Node.

---

# Arquitectura Completa

```text
                    Administrador

                          │

                          ▼

                   Control Node

                          │

               ansible-playbook

                          │

                 Inventory + SSH

                          │

     ┌────────────┬────────────┬────────────┐

     ▼            ▼            ▼            ▼

   Web01        Web02        DB01       Backup01
```

---

# Flujo recomendado

```text
Escribir Playbook

        │

        ▼

Validar Sintaxis

        │

        ▼

Ejecutar --check

        │

        ▼

Ambiente DEV

        │

        ▼

Ambiente QA

        │

        ▼

Producción
```

Nunca ejecutar un Playbook nuevo directamente en producción.

---

# Validación previa

Antes de ejecutar cualquier automatización debemos verificar:

- Inventario.
- SSH.
- Python.
- Variables.
- Roles.
- Plantillas.
- Conectividad.
- Espacio en disco.
- Permisos.

---

# Lista de verificación

```text
□ Inventario correcto

□ SSH funcionando

□ Python instalado

□ Variables verificadas

□ Playbook validado

□ Backup disponible

□ Git actualizado

□ Cambios aprobados
```

---

# Validar sintaxis

Siempre ejecutar:

```bash
ansible-playbook playbooks/apache.yml --syntax-check
```

Salida esperada

```text
playbook: playbooks/apache.yml
```

Si existe algún error de sintaxis deberá corregirse antes de continuar.

---

# Simular cambios

```bash
ansible-playbook playbooks/apache.yml --check
```

Este modo no realiza cambios reales.

Permite conocer:

- Qué tareas se ejecutarán.
- Qué servidores serán modificados.
- Qué recursos cambiarán.

---

# Mostrar diferencias

```bash
ansible-playbook playbooks/apache.yml --diff
```

Ejemplo

```text
ANTES

Listen 80

DESPUÉS

Listen 8080
```

Muy útil para auditorías.

---

# Ejecutar únicamente un servidor

```bash
ansible-playbook playbooks/apache.yml --limit web01
```

---

# Ejecutar únicamente un grupo

```bash
ansible-playbook playbooks/apache.yml --limit web
```

---

# Aumentar el detalle

```bash
ansible-playbook playbooks/apache.yml -vvv
```

Niveles disponibles

```text
-v

-vv

-vvv

-vvvv
```

Cuanto mayor sea el nivel, mayor información mostrará Ansible.

---

# Flujo de ejecución interno

```text
Playbook

↓

Inventory

↓

SSH

↓

Python

↓

Module

↓

Resultado

↓

Reporte
```

---

# Reporte Final

Al finalizar una ejecución aparece un resumen.

```text
PLAY RECAP

web01

ok=8

changed=2

failed=0

unreachable=0
```

---

# Interpretación

| Resultado | Significado |
|------------|-------------|
| ok | La tarea fue correcta |
| changed | Hubo modificaciones |
| failed | Ocurrió un error |
| unreachable | No fue posible conectar |
| skipped | La tarea fue omitida |

---

# Error: UNREACHABLE

Ejemplo

```text
UNREACHABLE

Failed to connect
```

Posibles causas

- SSH detenido.
- Firewall.
- IP incorrecta.
- DNS.
- Inventario incorrecto.

---

# Diagnóstico

Verificar SSH.

```bash
ssh usuario@web01
```

---

Verificar red.

```bash
ping web01
```

---

Verificar puerto.

```bash
nc -zv web01 22
```

---

# Error: Permission Denied

```text
Permission denied (publickey)
```

Revisar

- Llave privada.
- Llave pública.
- Usuario.
- Permisos.

---

# Verificar permisos

```bash
ls -la ~/.ssh
```

---

Permisos recomendados

```bash
chmod 700 ~/.ssh

chmod 600 ~/.ssh/id_ed25519

chmod 644 ~/.ssh/id_ed25519.pub
```

---

# Error: Python no encontrado

```text
FAILED

Python interpreter not found
```

Comprobar

```bash
python3 --version
```

Instalar Python si es necesario.

---

# Error: YAML

```text
mapping values are not allowed
```

Generalmente se debe a:

- Mala indentación.
- Tabulaciones.
- Dos puntos faltantes.
- Espacios incorrectos.

---

# Validar YAML

```bash
ansible-playbook sitio.yml --syntax-check
```

---

# Error: Inventory

```text
No hosts matched
```

Comprobar

```bash
ansible-inventory --graph
```

---

# Error: Variables

```text
Undefined variable
```

Revisar

- group_vars
- host_vars
- vars
- nombres de variables

---

# Error: Collection inexistente

```text
Collection not found
```

Consultar

```bash
ansible-galaxy collection list
```

---

# Instalar una Collection

```bash
ansible-galaxy collection install community.general
```

---

# Verificar módulos

```bash
ansible-doc copy
```

---

# Verificar configuración

```bash
ansible-config dump --only-changed
```

---

# Auditoría

Toda automatización debería registrar:

- Fecha.
- Usuario.
- Playbook.
- Inventario.
- Resultado.
- Cambios realizados.

---

# Logging

Ejemplo

```ini
log_path=/var/log/ansible.log
```

---

# Ejemplo de auditoría

```text
2026-08-05

Administrador

↓

playbooks/apache.yml

↓

120 servidores

↓

2 cambios

↓

Sin errores
```

---

# Integración con Git

Todo proyecto debe mantenerse bajo control de versiones.

```text
Repositorio

↓

Commit

↓

Push

↓

Pull Request

↓

Producción
```

---

# Estrategia recomendada

```text
Crear Branch

↓

Modificar Playbook

↓

Pruebas

↓

Merge

↓

Producción
```

---

# Recuperación

Si un cambio produce problemas:

```text
Git

↓

Rollback

↓

Versión anterior

↓

Nueva ejecución
```

---

# Seguridad

Nunca almacenar:

- Contraseñas.
- Tokens.
- API Keys.
- Secretos.

En texto plano.

Utilizar siempre:

```text
Ansible Vault
```

Este tema será estudiado más adelante.

---

# Buenas prácticas

- Un proyecto por infraestructura.
- Inventarios separados.
- Variables organizadas.
- Uso de Git.
- SSH mediante claves.
- Logging habilitado.
- Validar sintaxis.
- Ejecutar `--check`.
- Documentar cambios.
- Realizar respaldos antes de cambios importantes.

---

# Errores comunes

## Error 1

Ejecutar Playbooks directamente sobre Producción.

---

## Error 2

No utilizar Git.

---

## Error 3

No probar la conectividad SSH.

---

## Error 4

Deshabilitar permanentemente `host_key_checking`.

---

## Error 5

No revisar el resultado del PLAY RECAP.

---

## Error 6

Utilizar usuarios root para todas las tareas.

---

## Error 7

No mantener respaldos del proyecto.

---

# Laboratorio Integral RHCSA

## Escenario

La empresa dispone de:

```text
Control Node

↓

web01

↓

web02

↓

web03

↓

db01

↓

backup01
```

Se requiere:

- Instalar Ansible.
- Configurar `ansible.cfg`.
- Configurar SSH mediante llaves.
- Crear Inventarios.
- Configurar `group_vars`.
- Configurar `host_vars`.
- Validar conectividad.
- Ejecutar Playbooks.
- Simular cambios.
- Ejecutar cambios reales.
- Documentar el procedimiento.

---

# Ejercicio 2

Crear un proyecto completo con la siguiente estructura.

```text
ansible-project/

├── ansible.cfg

├── inventories/

├── playbooks/

├── roles/

├── templates/

├── files/

├── group_vars/

├── host_vars/

├── collections/

├── scripts/

└── documentation/
```

---

# Ejercicio 3

Diseñar una infraestructura para administrar:

- 50 servidores Web.
- 20 Bases de Datos.
- 10 Balanceadores.
- 10 Servidores DNS.
- 5 Servidores de Monitoreo.
- 5 Servidores de Respaldo.

Crear:

- Inventarios.
- Variables.
- Estructura de directorios.

---

# Ejercicio 4

Ejecutar un Playbook utilizando:

- `--check`
- `--diff`
- `--limit`
- `--syntax-check`

Documentar el resultado obtenido en cada caso.

---

# Preguntas de Repaso

1. ¿Qué función cumple `ansible.cfg`?

2. ¿Cuál es la diferencia entre `ansible` y `ansible-playbook`?

3. ¿Qué utilidad tiene `ansible-config`?

4. ¿Qué hace `ansible-inventory`?

5. ¿Qué es `become`?

6. ¿Por qué se recomienda utilizar SSH mediante llaves?

7. ¿Qué significa `UNREACHABLE`?

8. ¿Qué significa `changed`?

9. ¿Qué utilidad tiene `--check`?

10. ¿Qué utilidad tiene `--diff`?

11. ¿Qué hace `--limit`?

12. ¿Cuál es la diferencia entre `group_vars` y `host_vars`?

13. ¿Por qué utilizar Git?

14. ¿Qué ventajas ofrece separar los ambientes DEV, QA y Producción?

15. ¿Qué beneficios ofrece un proyecto bien organizado?

---

# Resumen del capítulo

En este capítulo aprendimos a instalar y preparar un entorno profesional de Ansible. Estudiamos la instalación de **ansible-core**, el archivo **ansible.cfg**, la configuración del Control Node, la autenticación mediante SSH, el uso de `become`, la organización de proyectos, la administración de variables, la integración con Git y las principales técnicas de validación y troubleshooting.

Con estos conocimientos ya contamos con una plataforma completamente preparada para comenzar a automatizar infraestructura de forma segura y escalable.

---

# Próximo Capítulo

## **80. Inventarios en Ansible**

En el siguiente capítulo profundizaremos en uno de los componentes más importantes de Ansible: los **Inventarios**. Aprenderemos a crear inventarios estáticos y dinámicos, organizar servidores mediante grupos y grupos anidados, definir variables por host y por grupo, administrar múltiples ambientes y aplicar buenas prácticas utilizadas en infraestructuras empresariales con cientos o miles de servidores.

----











