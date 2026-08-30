# 86. Ansible Vault (Fase 1)

> **Módulo 11 – Automatización con Ansible**
>
> **Archivo:** `86-ansible-vault.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Comprender qué es Ansible Vault.
- Entender por qué es indispensable en ambientes empresariales.
- Identificar qué información debe cifrarse.
- Comprender cómo funciona el cifrado de Vault.
- Crear archivos cifrados.
- Editar archivos protegidos.
- Visualizar información cifrada.
- Integrar Vault dentro de un proyecto Ansible.

---

# Introducción

Hasta ahora nuestros Playbooks contienen información como:

- Usuarios
- Direcciones IP
- Variables
- Configuraciones
- Inventarios

Pero existe un problema muy importante.

Muchas veces necesitamos almacenar información confidencial como:

- Contraseñas
- Tokens
- API Keys
- Claves SSH
- Certificados
- Secretos de aplicaciones
- Credenciales de Bases de Datos

Nunca debemos guardar este tipo de información en texto plano.

Para resolver este problema Ansible incorpora:

# Ansible Vault

---

# ¿Qué es Ansible Vault?

Ansible Vault es el mecanismo oficial de Ansible para cifrar información sensible.

Permite almacenar secretos dentro del mismo proyecto de automatización sin exponer su contenido.

---

# Arquitectura

```text
Variables

↓

Ansible Vault

↓

Archivo Cifrado

↓

Git

↓

Repositorio Seguro
```

---

# Sin Vault

```text
group_vars/

↓

password: MiPassword123
```

Problemas.

- Visible.
- Fácil de copiar.
- Riesgo de fuga.
- No apto para Producción.

---

# Con Vault

```text
group_vars/

↓

$ANSIBLE_VAULT;1.1;AES256

xxxxxxxxxxxxxxxx

xxxxxxxxxxxxxxxx
```

La contraseña queda protegida.

---

# ¿Qué información debe protegerse?

Generalmente.

- Contraseñas.
- Tokens.
- API Keys.
- Secretos.
- Llaves privadas.
- Certificados.
- Variables sensibles.
- Cadenas de conexión.
- Passwords de Bases de Datos.
- Credenciales Cloud.

---

# Información que normalmente NO necesita Vault

Generalmente.

- Hostname.
- Dirección IP.
- Nombre del servidor.
- Zona horaria.
- Puerto HTTP.
- Nombre del dominio.
- Configuración pública.

---

# Caso Real

Servidor PostgreSQL.

Variables.

```yaml
postgres_user: postgres

postgres_password: MiPassword123
```

El usuario puede permanecer en texto plano.

La contraseña debe cifrarse.

---

# Otro Ejemplo

SQL Server.

```yaml
sa_password: P@ssw0rd!
```

Debe protegerse mediante Vault.

---

# Arquitectura Empresarial

```text
Inventario

↓

Variables

↓

Variables Sensibles

↓

Vault

↓

Playbook
```

---

# ¿Cómo funciona?

Cuando ejecutamos un Playbook.

```text
Playbook

↓

Vault

↓

Solicita Password

↓

Descifra

↓

Ejecuta Tasks
```

---

# Algoritmo de cifrado

Ansible Vault utiliza cifrado robusto.

Actualmente.

```text
AES256
```

Por esta razón el contenido no puede leerse sin la contraseña correcta.

---

# Aspecto de un Archivo Vault

Ejemplo.

```text
$ANSIBLE_VAULT;1.1;AES256

6234666335326336

3461653233333634

3766613731336264

...
```

El contenido resulta ilegible.

---

# Crear un Archivo Vault

Sintaxis.

```bash
ansible-vault create secretos.yml
```

---

Resultado.

Ansible solicitará una contraseña.

```text
New Vault password:
```

Después.

```text
Confirm New Vault password:
```

Finalmente abrirá el editor configurado.

---

# Ejemplo

```bash
ansible-vault create passwords.yml
```

---

Dentro.

```yaml
db_password: MiPasswordSegura

api_key: XXXXXXXXXXX
```

Al guardar.

El archivo quedará completamente cifrado.

---

# Flujo

```text
Create

↓

Editor

↓

Guardar

↓

Cifrado
```

---

# Ver el Archivo

Si utilizamos:

```bash
cat passwords.yml
```

Obtendremos.

```text
$ANSIBLE_VAULT;1.1;AES256

xxxxxxxxxxxxx
```

No veremos el contenido real.

---

# Visualizar el Contenido

Utilizamos.

```bash
ansible-vault view passwords.yml
```

---

Flujo.

```text
Archivo

↓

Password

↓

Descifrar

↓

Mostrar
```

---

# Editar un Archivo

No debemos utilizar.

```bash
nano passwords.yml
```

porque destruiríamos el formato cifrado.

Siempre debemos utilizar.

```bash
ansible-vault edit passwords.yml
```

---

Proceso.

```text
Archivo

↓

Password

↓

Descifrar

↓

Editar

↓

Guardar

↓

Volver a cifrar
```

---

# Cambiar la Contraseña

También es posible.

```bash
ansible-vault rekey passwords.yml
```

---

Proceso.

```text
Password Actual

↓

Nueva Password

↓

Recifrar Archivo
```

---

# Descifrar Temporalmente

Para leer.

```bash
ansible-vault view passwords.yml
```

---

Para editar.

```bash
ansible-vault edit passwords.yml
```

---

Nunca es necesario descifrar permanentemente el archivo.

---

# Integración con Git

Uno de los mayores beneficios.

Podemos almacenar.

```text
passwords.yml
```

Dentro del repositorio.

Aunque alguien clone el proyecto.

No podrá leer los secretos.

---

Arquitectura.

```text
Git

↓

Vault

↓

Repositorio

↓

Seguro
```

---

# Organización Recomendada

```text
Proyecto

├── inventories

├── group_vars

│      ├── all.yml

│      └── vault.yml

├── host_vars

├── roles

└── playbooks
```

---

Generalmente.

```text
vault.yml
```

contendrá únicamente secretos.

---

# Separación

Variables normales.

```yaml
http_port: 80

timezone: America/Santo_Domingo
```

---

Variables sensibles.

```yaml
db_password: ********

api_key: ********

secret_token: ********
```

Estas últimas deben almacenarse en Vault.

---

# Flujo Completo

```text
Playbook

↓

Variables

↓

Vault

↓

Password

↓

Descifrado

↓

Servidor
```

---

# Beneficios

- Protección de credenciales.
- Integración con Git.
- Fácil administración.
- Seguridad.
- Reutilización.
- Automatización.
- Compatible con Roles.
- Compatible con group_vars.
- Compatible con host_vars.
- Compatible con Inventarios.

---

# Caso Empresarial

Una empresa administra.

- 800 servidores Linux.
- 300 PostgreSQL.
- 150 SQL Server.

Todos utilizan:

```text
vault.yml
```

para almacenar.

- Password PostgreSQL.
- Password SQL Server.
- Password Oracle.
- API Keys.
- Tokens.
- Certificados.

Ninguna contraseña aparece en texto plano.

---

# Buenas Prácticas

- Cifrar únicamente información sensible.
- Utilizar contraseñas robustas para Vault.
- Mantener separados secretos y Variables públicas.
- No compartir la contraseña mediante correo electrónico.
- Utilizar un gestor seguro de contraseñas para custodiar la clave de Vault.
- Documentar qué archivos utilizan Vault.
- Utilizar Git para versionar archivos cifrados.
- Revisar periódicamente los secretos almacenados.
- Cambiar la contraseña de Vault cuando sea necesario.
- Aplicar el principio de mínimo privilegio para acceder a la contraseña.

---

# Errores Comunes

## Error 1

Guardar contraseñas en texto plano.

---

## Error 2

Editar un archivo Vault con:

```bash
nano
```

---

## Error 3

Olvidar la contraseña.

---

## Error 4

Compartir la contraseña por canales inseguros.

---

## Error 5

Guardar secretos mezclados con Variables públicas.

---

## Error 6

Subir secretos sin cifrar al repositorio.

---

## Error 7

No realizar copias de seguridad seguras de la contraseña de Vault.

---

## Error 8

Utilizar una contraseña demasiado sencilla.

---

## Error 9

Cifrar todo el proyecto innecesariamente.

---

## Error 10

No rotar la contraseña cuando cambian los responsables del proyecto.

---

# Laboratorio RHCSA

## Escenario

Una empresa administrará credenciales de:

- PostgreSQL.
- SQL Server.
- Oracle.
- Servidores Linux.
- API Cloud.

Toda la información deberá protegerse utilizando Ansible Vault.

---

## Laboratorio 1

Crear.

```bash
ansible-vault create vault.yml
```

---

## Laboratorio 2

Guardar.

- db_password
- api_key
- ssh_password

---

## Laboratorio 3

Visualizar el contenido utilizando.

```bash
ansible-vault view
```

---

## Laboratorio 4

Modificar una contraseña utilizando.

```bash
ansible-vault edit
```

---

## Laboratorio 5

Cambiar la contraseña mediante.

```bash
ansible-vault rekey
```

---

## Laboratorio 6

Organizar el proyecto separando Variables públicas de Variables protegidas.

---

## Laboratorio 7

Agregar el archivo cifrado al repositorio Git y verificar que su contenido permanezca protegido.

---

## Laboratorio 8

Diseñar una estructura empresarial donde cada Role utilice un archivo `vault.yml` independiente para almacenar únicamente las credenciales necesarias para ese servicio.

---

# Preguntas de Repaso

1. ¿Qué es Ansible Vault?
2. ¿Qué algoritmo de cifrado utiliza?
3. ¿Qué tipo de información debería almacenarse en Vault?
4. ¿Qué diferencia existe entre `view` y `edit`?
5. ¿Qué función cumple `rekey`?
6. ¿Por qué es recomendable separar Variables públicas de Variables sensibles?
7. ¿Qué ventajas ofrece Vault al utilizar Git?
8. ¿Por qué no debe editarse un archivo Vault con un editor convencional?
9. ¿Qué buenas prácticas deben seguirse para administrar la contraseña de Vault?
10. ¿Cómo organizarías un proyecto empresarial utilizando Ansible Vault?

---

# Resumen

En esta primera fase aprendimos que **Ansible Vault** es el mecanismo oficial para proteger información sensible dentro de proyectos Ansible. Estudiamos cómo crear, visualizar, editar y cambiar la contraseña de archivos cifrados utilizando `ansible-vault create`, `view`, `edit` y `rekey`, así como la importancia de separar Variables públicas de secretos.

También analizamos cómo Vault permite almacenar archivos cifrados dentro de repositorios Git sin exponer credenciales, convirtiéndose en una herramienta fundamental para automatizar despliegues de forma segura en entornos empresariales.

En la **Fase 2** aprenderemos a integrar Vault con Playbooks, `group_vars`, `host_vars` y Roles, ejecutar Playbooks utilizando `--ask-vault-pass` y `--vault-password-file`, trabajar con múltiples archivos Vault y automatizar el manejo seguro de secretos en infraestructuras de Producción.

-----

# 86. Ansible Vault (Fase 2)

> **Módulo 11 – Automatización con Ansible**
>
> **Archivo:** `86-ansible-vault.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Integrar Vault con Playbooks.
- Utilizar Vault dentro de Roles.
- Trabajar con `group_vars` y `host_vars`.
- Ejecutar Playbooks protegidos.
- Utilizar archivos de contraseña.
- Comprender el funcionamiento de múltiples archivos Vault.
- Automatizar la carga de secretos.
- Aplicar buenas prácticas en Producción.

---

# Introducción

En la fase anterior aprendimos a:

- Crear archivos Vault.
- Editarlos.
- Visualizarlos.
- Cambiar su contraseña.

Ahora veremos cómo esos archivos se utilizan dentro de un proyecto Ansible real.

En la práctica, Vault casi nunca se utiliza de forma aislada.

Normalmente forma parte de una estructura como esta:

```text
Inventario

↓

group_vars

↓

host_vars

↓

Roles

↓

Playbooks

↓

Vault
```

---

# Integración con Playbooks

Supongamos el siguiente proyecto.

```text
Proyecto

├── inventories

├── playbooks

├── group_vars

├── host_vars

└── roles
```

Dentro de:

```text
group_vars
```

tenemos.

```text
all.yml
```

y

```text
vault.yml
```

---

# Ejemplo

Variables normales.

```yaml
http_port: 80

timezone: America/Santo_Domingo
```

---

Variables sensibles.

```yaml
db_password: MiPasswordSegura
```

Después de aplicar Vault.

```text
$ANSIBLE_VAULT;1.1;AES256
```

---

# Flujo

```text
Playbook

↓

group_vars

↓

Vault

↓

Variables

↓

Tasks
```

---

# Uso Transparente

Una de las grandes ventajas de Vault es que los Playbooks no cambian.

Ejemplo.

```yaml
- name: Crear usuario

  user:

    name: administrador

    password: "{{ db_password }}"
```

El Playbook no necesita saber que la Variable está cifrada.

---

# Arquitectura

```text
Variable

↓

Vault

↓

Descifrado

↓

Task
```

---

# Ejecutar un Playbook

Si existen archivos Vault.

Ansible necesita la contraseña.

Forma más sencilla.

```bash
ansible-playbook site.yml \
--ask-vault-pass
```

---

Proceso.

```text
Playbook

↓

Solicitar Password

↓

Descifrar

↓

Ejecutar
```

---

# Resultado

```text
Vault password:
```

Después.

```text
PLAY RECAP
```

---

# Archivo de Contraseña

Es posible evitar escribir la contraseña manualmente.

Ejemplo.

```bash
ansible-playbook site.yml \
--vault-password-file vault.pass
```

---

Flujo.

```text
vault.pass

↓

Password

↓

Vault

↓

Playbook
```

---

# Contenido del Archivo

```text
MiPasswordMuySegura
```

---

# Permisos

Este archivo debe protegerse.

Ejemplo.

```bash
chmod 600 vault.pass
```

---

Comprobación.

```bash
ls -l vault.pass
```

Resultado.

```text
-rw-------
```

---

# Advertencia

Nunca subir.

```text
vault.pass
```

al repositorio Git.

---

Agregar.

```text
.gitignore
```

---

Ejemplo.

```text
vault.pass
```

---

# Alternativa

Variables de entorno.

Aunque existen otros mecanismos de integración con herramientas externas de gestión de secretos, para RHCSA y la mayoría de proyectos iniciales es suficiente comprender:

- `--ask-vault-pass`
- `--vault-password-file`

---

# Integración con group_vars

Proyecto.

```text
group_vars/

├── all.yml

├── web.yml

├── db.yml

└── vault.yml
```

---

Contenido.

```yaml
db_password: ********
```

El archivo.

```text
vault.yml
```

permanece cifrado.

---

# Integración con host_vars

También.

```text
host_vars/

├── web01.yml

├── web02.yml

└── db01.yml
```

Cada servidor puede tener.

- Password diferente.
- API Key distinta.
- Token diferente.

---

# Flujo

```text
Servidor

↓

host_vars

↓

Vault

↓

Variables
```

---

# Integración con Roles

Estructura.

```text
roles/

└── postgres/

    ├── defaults

    ├── vars

    ├── tasks

    └── templates
```

---

Task.

```yaml
- name: Crear usuario

  community.postgresql.postgresql_user:

    name: app

    password: "{{ postgres_password }}"
```

---

La Variable puede provenir de.

```text
Vault
```

Sin modificar el Role.

---

# Caso SQL Server

```yaml
sa_password:

********
```

Utilización.

```yaml
{{ sa_password }}
```

El Role desconoce que la Variable está cifrada.

---

# Variables Compartidas

Una empresa administra.

- Apache
- PostgreSQL
- SQL Server
- Oracle

Todos utilizan.

```text
vault.yml
```

para obtener.

```text
Passwords

↓

API Keys

↓

Tokens
```

---

# Múltiples Archivos Vault

También es posible.

```text
group_vars/

├── vault_linux.yml

├── vault_sql.yml

├── vault_cloud.yml

└── vault_network.yml
```

Cada archivo protege secretos distintos.

---

# Beneficios

- Mejor organización.
- Separación de responsabilidades.
- Menor exposición.
- Fácil mantenimiento.

---

# Estructura Recomendada

```text
Proyecto

├── inventories

├── group_vars

│      ├── all.yml

│      ├── vault_linux.yml

│      ├── vault_db.yml

│      └── vault_cloud.yml

├── host_vars

├── roles

├── playbooks

└── templates
```

---

# Variables Normales

```yaml
timezone: America/Santo_Domingo

ntp_server: pool.ntp.org
```

---

# Variables Vault

```yaml
root_password:

********

api_token:

********
```

---

# Ejemplo Completo

Playbook.

```yaml
---

- hosts: database

  become: true

  roles:

    - postgres
```

---

Role.

```yaml
password:

{{ postgres_password }}
```

---

Variables.

```text
Vault
```

---

Resultado.

```text
Servidor configurado

↓

Password protegida
```

---

# Arquitectura Empresarial

```text
Git

↓

Playbook

↓

Role

↓

Vault

↓

Variables

↓

Servidor
```

---

# Flujo Completo

```text
Playbook

↓

Role

↓

Task

↓

Variable

↓

Vault

↓

Password

↓

Servidor
```

---

# Automatización

En Producción.

```text
CI/CD

↓

Vault Password File

↓

Playbook

↓

Servidores
```

Todo ocurre automáticamente.

---

# Caso Empresarial

Empresa.

1.000 servidores.

Cada departamento administra.

```text
Vault Linux

Vault DBA

Vault Cloud

Vault Redes
```

Todos independientes.

---

# Beneficios

- Seguridad.
- Escalabilidad.
- Automatización.
- Reutilización.
- Compatibilidad con Roles.
- Fácil mantenimiento.
- Integración con Git.
- Organización.

---

# Buenas Prácticas

- Mantener secretos separados por función.
- Utilizar `group_vars` para secretos compartidos.
- Utilizar `host_vars` para secretos específicos de un servidor.
- No almacenar `vault.pass` en Git.
- Limitar permisos sobre el archivo de contraseña.
- Documentar qué archivos utilizan Vault.
- Cambiar contraseñas periódicamente.
- Utilizar Variables descriptivas.
- Evitar duplicación.
- Mantener Roles independientes del origen de las Variables.
- Revisar periódicamente los permisos de acceso a los secretos.

---

# Errores Comunes

## Error 1

Subir.

```text
vault.pass
```

a Git.

---

## Error 2

Guardar contraseñas en.

```text
all.yml
```

---

## Error 3

Olvidar utilizar.

```bash
--ask-vault-pass
```

---

## Error 4

Permisos inseguros.

```text
777
```

---

## Error 5

Duplicar secretos.

---

## Error 6

No separar Variables públicas y privadas.

---

## Error 7

Compartir el mismo archivo Vault para todos los equipos sin necesidad.

---

## Error 8

No documentar qué Variables están protegidas.

---

## Error 9

Utilizar nombres ambiguos para los archivos Vault.

---

## Error 10

Confiar únicamente en el cifrado de Vault sin proteger adecuadamente el acceso al repositorio y al archivo de contraseña.

---

# Laboratorio RHCSA

## Escenario

Una empresa administrará:

- PostgreSQL.
- SQL Server.
- Oracle.
- Apache.
- API Cloud.

Todos utilizarán Variables protegidas.

---

## Laboratorio 1

Crear.

```text
vault.yml
```

---

## Laboratorio 2

Agregar.

- postgres_password
- sql_password
- oracle_password

---

## Laboratorio 3

Ejecutar.

```bash
ansible-playbook site.yml \
--ask-vault-pass
```

---

## Laboratorio 4

Crear.

```text
vault.pass
```

---

## Laboratorio 5

Ejecutar.

```bash
--vault-password-file
```

---

## Laboratorio 6

Mover secretos compartidos a:

```text
group_vars
```

---

## Laboratorio 7

Mover secretos exclusivos a:

```text
host_vars
```

---

## Laboratorio 8

Modificar un Role PostgreSQL para consumir Variables protegidas desde Vault sin cambiar el código del Role.

---

## Laboratorio 9

Dividir los secretos de la infraestructura en varios archivos Vault (`vault_linux.yml`, `vault_db.yml` y `vault_cloud.yml`) y documentar qué información contendrá cada uno.

---

## Laboratorio 10

Diseñar una estructura empresarial donde:

- Los Playbooks no contengan credenciales.
- Los Roles utilicen Variables provenientes de Vault.
- Los secretos compartidos residan en `group_vars`.
- Los secretos específicos residan en `host_vars`.
- El archivo `vault.pass` permanezca fuera del repositorio y con permisos restrictivos.

---

# Preguntas de Repaso

1. ¿Cómo utiliza un Playbook las Variables protegidas por Vault?
2. ¿Qué función cumple `--ask-vault-pass`?
3. ¿Qué ventajas ofrece `--vault-password-file`?
4. ¿Por qué `vault.pass` nunca debe almacenarse en Git?
5. ¿Cuándo utilizarías `group_vars` para secretos?
6. ¿Cuándo utilizarías `host_vars` para secretos?
7. ¿Cómo interactúan los Roles con Variables protegidas?
8. ¿Qué ventajas ofrece dividir los secretos en varios archivos Vault?
9. ¿Qué permisos deberían asignarse al archivo `vault.pass`?
10. ¿Cómo organizarías los secretos de una infraestructura empresarial utilizando Ansible Vault?

---

# Resumen

En esta segunda fase aprendimos a integrar **Ansible Vault** con Playbooks, Roles, `group_vars` y `host_vars`, utilizando Variables protegidas de manera completamente transparente para las Tasks y los Roles. Estudiamos las opciones `--ask-vault-pass` y `--vault-password-file`, así como las consideraciones de seguridad relacionadas con el almacenamiento del archivo de contraseña.

También analizamos estrategias para organizar secretos compartidos y específicos por servidor, el uso de múltiples archivos Vault y las mejores prácticas para mantener una infraestructura segura, modular y preparada para entornos empresariales.

En la **Fase 3** estudiaremos el cifrado de Variables individuales mediante `encrypt_string`, el uso de múltiples **Vault IDs**, la administración de diferentes contraseñas para distintos equipos o entornos, la rotación de secretos, la integración con Roles complejos y las prácticas avanzadas utilizadas en organizaciones de gran escala.

----

# 86. Ansible Vault (Fase 3)

> **Módulo 11 – Automatización con Ansible**
>
> **Archivo:** `86-ansible-vault.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Cifrar Variables individuales.
- Utilizar `encrypt_string`.
- Comprender Vault IDs.
- Trabajar con múltiples contraseñas Vault.
- Administrar diferentes entornos.
- Rotar secretos.
- Integrar Vault con Roles empresariales.
- Aplicar mejores prácticas de seguridad.

---

# Introducción

Hasta ahora hemos aprendido a cifrar archivos completos utilizando:

```bash
ansible-vault create
```

Sin embargo, muchas organizaciones no desean cifrar archivos completos.

En ocasiones únicamente necesitan proteger una o dos Variables.

Para ello Ansible incorpora otra funcionalidad muy importante.

```text
encrypt_string
```

---

# ¿Qué es encrypt_string?

Permite cifrar únicamente una Variable.

No es necesario proteger todo el archivo.

Ejemplo.

Antes.

```yaml
http_port: 80

timezone: America/Santo_Domingo

db_password: MiPassword
```

Después.

```yaml
http_port: 80

timezone: America/Santo_Domingo

db_password: !vault |

   $ANSIBLE_VAULT;1.1;AES256

   xxxxxxxxxxxxxxxxx
```

---

# Ventajas

- El archivo continúa siendo legible.
- Sólo la información sensible permanece cifrada.
- Fácil mantenimiento.
- Mejor experiencia para los administradores.

---

# Sintaxis

```bash
ansible-vault encrypt_string
```

---

Ejemplo.

```bash
ansible-vault encrypt_string \
'MiPasswordSegura'
```

---

Resultado.

```text
!vault |

$ANSIBLE_VAULT;1.1;AES256

xxxxxxxxxxxxxxxx
```

---

# Asignando un Nombre

Es recomendable indicar el nombre de la Variable.

```bash
ansible-vault encrypt_string \
--name db_password \
'MiPasswordSegura'
```

---

Resultado.

```yaml
db_password: !vault |

          $ANSIBLE_VAULT;1.1;AES256

          xxxxxxxxxxxxxxxxx
```

---

# Flujo

```text
Texto Plano

↓

encrypt_string

↓

Variable Vault

↓

Playbook
```

---

# Caso Empresarial

Archivo.

```yaml
timezone: UTC

http_port: 8080

db_password:

!vault |

xxxxxxxxxxxxxxxx
```

Únicamente la contraseña queda protegida.

---

# Ventajas frente al Archivo Completo

Archivo completo.

```text
Todo ilegible
```

---

Variable individual.

```text
Configuración visible

↓

Contraseña protegida
```

---

# Uso en Playbooks

No cambia absolutamente nada.

```yaml
password:

{{ db_password }}
```

El Playbook no distingue si la Variable está cifrada.

---

# Uso en Templates

También funciona.

```jinja2
{{ db_password }}
```

---

# Uso en Roles

El Role continúa utilizando.

```yaml
{{ db_password }}
```

No requiere modificaciones.

---

# Vault IDs

Hasta ahora utilizamos.

```text
Una contraseña
```

Pero las empresas normalmente utilizan varias.

---

Ejemplo.

```text
Producción

↓

Password A
```

```text
Desarrollo

↓

Password B
```

```text
Cloud

↓

Password C
```

---

# ¿Qué es un Vault ID?

Un Vault ID identifica un conjunto de secretos.

Cada uno puede utilizar una contraseña diferente.

---

Representación.

```text
Vault Linux

↓

Password Linux
```

```text
Vault Database

↓

Password DBA
```

```text
Vault Cloud

↓

Password Cloud
```

---

# Crear utilizando Vault ID

Ejemplo.

```bash
ansible-vault create \
--vault-id produccion
```

---

También.

```bash
ansible-vault create \
--vault-id desarrollo
```

---

# Arquitectura

```text
Producción

↓

Vault ID

↓

Password

↓

Secretos
```

---

# Empresa Grande

Supongamos.

```text
Departamento Linux

↓

Vault Linux
```

```text
Departamento DBA

↓

Vault Database
```

```text
Departamento Cloud

↓

Vault Cloud
```

Cada equipo administra únicamente sus propios secretos.

---

# Beneficios

- Mayor seguridad.
- Separación.
- Auditoría.
- Administración independiente.

---

# Ejemplo

```text
vault_linux.yml

↓

Password Linux
```

---

```text
vault_sql.yml

↓

Password SQL
```

---

```text
vault_cloud.yml

↓

Password Cloud
```

---

# Ejecutar Playbooks

Puede indicarse.

```bash
--vault-id
```

Ejemplo.

```bash
ansible-playbook site.yml \
--vault-id produccion
```

---

También.

```bash
ansible-playbook site.yml \
--vault-id desarrollo
```

---

# Rotación de Secretos

Con el tiempo.

Las contraseñas cambian.

La mejor práctica.

```text
Rotarlas periódicamente
```

---

Proceso.

```text
Password Antigua

↓

Rekey

↓

Nueva Password
```

---

# Rekey

Sintaxis.

```bash
ansible-vault rekey
```

---

Beneficios.

- Reduce riesgos.
- Cumple auditorías.
- Cumple políticas corporativas.

---

# Estrategia Empresarial

Cada año.

```text
Rotación

↓

Nueva Password

↓

Vault Actualizado
```

---

# Integración con Roles

Proyecto.

```text
roles/

├── apache

├── postgres

├── sqlserver

├── nginx

└── monitoring
```

Todos utilizan Variables.

```text
Vault
```

---

# Ejemplo PostgreSQL

```yaml
postgres_password:

{{ postgres_password }}
```

---

# Ejemplo SQL Server

```yaml
sa_password:

{{ sa_password }}
```

---

# Ejemplo Oracle

```yaml
oracle_password:

{{ oracle_password }}
```

---

Todos provienen de Vault.

---

# Integración con Templates

```jinja2
Database Password:

{{ postgres_password }}
```

La Variable será descifrada automáticamente durante la ejecución del Playbook.

---

# Flujo Completo

```text
Vault

↓

Variable

↓

Role

↓

Template

↓

Servidor
```

---

# Organización Empresarial

```text
group_vars/

├── all.yml

├── vault_linux.yml

├── vault_database.yml

├── vault_cloud.yml

└── vault_network.yml
```

---

# Flujo General

```text
Playbook

↓

Vault ID

↓

Password

↓

Variables

↓

Roles

↓

Servidor
```

---

# Separación por Ambientes

Muchas empresas poseen.

```text
Desarrollo

↓

QA

↓

Producción
```

Cada ambiente utiliza.

- Vault distinto.
- Password distinta.
- Variables distintas.

---

# Ventajas

- Aislamiento.
- Seguridad.
- Auditoría.
- Menor riesgo.
- Escalabilidad.

---

# Automatización

```text
Git

↓

CI/CD

↓

Vault ID

↓

Playbook

↓

Infraestructura
```

---

# Auditoría

Debe verificarse.

```text
✓ Variables protegidas

✓ Passwords

✓ API Keys

✓ Tokens

✓ Certificados

✓ Vault IDs

✓ Rekey

✓ Git
```

---

# Buenas Prácticas

- Utilizar `encrypt_string` cuando únicamente unas pocas Variables sean sensibles.
- Utilizar archivos Vault completos cuando la mayoría de Variables sean confidenciales.
- Separar Vault IDs por departamento o ambiente.
- Rotar periódicamente las contraseñas.
- Documentar el propósito de cada Vault ID.
- Utilizar nombres descriptivos.
- Mantener Roles independientes de los secretos.
- Evitar reutilizar la misma contraseña para todos los Vault IDs.
- Limitar el acceso a los archivos de contraseña.
- Realizar pruebas antes de Producción.

---

# Errores Comunes

## Error 1

Cifrar Variables innecesarias.

---

## Error 2

Utilizar la misma contraseña para todos los Vault IDs.

---

## Error 3

No rotar las contraseñas.

---

## Error 4

No documentar qué Vault protege cada entorno.

---

## Error 5

Duplicar secretos.

---

## Error 6

Guardar archivos de contraseña en Git.

---

## Error 7

No proteger los permisos del archivo de contraseña.

---

## Error 8

Mezclar secretos de Producción y Desarrollo en el mismo Vault.

---

## Error 9

No probar los Playbooks después de realizar un `rekey`.

---

## Error 10

Utilizar nombres poco descriptivos para Variables cifradas.

---

# Laboratorio RHCSA

## Escenario

Una empresa administrará:

- Linux.
- PostgreSQL.
- SQL Server.
- Oracle.
- Kubernetes.
- AWS.

Cada departamento tendrá su propio Vault.

---

## Laboratorio 1

Cifrar únicamente una Variable utilizando:

```bash
encrypt_string
```

---

## Laboratorio 2

Agregar la Variable cifrada a:

```yaml
group_vars/all.yml
```

---

## Laboratorio 3

Utilizar la Variable desde un Playbook.

---

## Laboratorio 4

Utilizar la Variable desde un Template.

---

## Laboratorio 5

Crear tres Vault IDs.

- Producción.
- QA.
- Desarrollo.

---

## Laboratorio 6

Ejecutar un Playbook utilizando:

```bash
--vault-id
```

---

## Laboratorio 7

Realizar una rotación de contraseña utilizando:

```bash
ansible-vault rekey
```

---

## Laboratorio 8

Crear un proyecto donde Apache, PostgreSQL y SQL Server utilicen Variables protegidas mediante `encrypt_string`, manteniendo visibles únicamente las Variables públicas.

---

## Laboratorio 9

Diseñar una estructura con múltiples Vault IDs donde cada equipo (Linux, DBA y Cloud) administre sus propios secretos sin compartir contraseñas.

---

## Laboratorio 10

Construir una arquitectura empresarial con:

- Variables individuales cifradas.
- Archivos Vault.
- Múltiples Vault IDs.
- Roles.
- Templates.
- Git.
- Rotación periódica de secretos.

---

# Preguntas de Repaso

1. ¿Qué diferencia existe entre cifrar un archivo completo y utilizar `encrypt_string`?
2. ¿Cuándo conviene utilizar Variables cifradas individuales?
3. ¿Qué es un Vault ID?
4. ¿Qué ventajas ofrece utilizar varios Vault IDs?
5. ¿Cómo ayuda la separación por ambientes a mejorar la seguridad?
6. ¿Por qué es importante rotar las contraseñas de Vault?
7. ¿Cómo utilizan los Roles las Variables protegidas?
8. ¿Qué ocurre con una Variable cifrada al utilizarla dentro de un Template?
9. ¿Qué ventajas ofrece separar los secretos por departamento?
10. ¿Cómo diseñarías una estrategia de Vault para una organización con varios equipos y múltiples entornos?

---

# Resumen

En esta tercera fase aprendimos a proteger **Variables individuales** mediante `ansible-vault encrypt_string`, una técnica que permite mantener visibles las Variables públicas mientras únicamente los datos sensibles permanecen cifrados. También estudiamos el concepto de **Vault IDs**, que permite administrar múltiples conjuntos de secretos con contraseñas independientes para diferentes departamentos o entornos como Desarrollo, QA y Producción.

Finalmente, analizamos estrategias empresariales para la rotación de secretos, la separación de responsabilidades, la integración con Roles y Templates, y la organización de proyectos escalables donde distintos equipos pueden administrar sus propios secretos de forma segura sin interferir con los demás.

En la **Fase 4** estudiaremos la integración avanzada de Ansible Vault con pipelines de CI/CD, validaciones de seguridad, automatización empresarial, troubleshooting, auditorías, mejores prácticas de operación y un laboratorio integral que reunirá todos los conceptos aprendidos sobre el manejo seguro de secretos en Ansible.

-----

# 86. Ansible Vault (Fase 4)

> **Módulo 11 – Automatización con Ansible**
>
> **Archivo:** `86-ansible-vault.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Integrar Ansible Vault en ambientes empresariales.
- Comprender cómo utilizar Vault dentro de pipelines CI/CD.
- Diseñar una arquitectura segura para el manejo de secretos.
- Auditar implementaciones que utilizan Vault.
- Diagnosticar problemas comunes.
- Aplicar las mejores prácticas utilizadas por grandes organizaciones.
- Construir una solución empresarial completa basada en Vault.

---

# Introducción

En las fases anteriores aprendimos:

- Crear archivos Vault.
- Editarlos.
- Visualizarlos.
- Utilizar `encrypt_string`.
- Trabajar con múltiples Vault IDs.
- Integrar Vault con Roles y Playbooks.

En esta fase veremos cómo estas herramientas son utilizadas en organizaciones que administran cientos o miles de servidores.

---

# Arquitectura Empresarial

```text
                     Git

                      │

                      ▼

                Pull Request

                      │

                      ▼

                 Code Review

                      │

                      ▼

                 Pipeline CI

                      │

                      ▼

               Ansible Vault

                      │

                      ▼

                  Playbooks

                      │

                      ▼

                   Servidores
```

---

# Flujo Completo

```text
Administrador

↓

Repositorio Git

↓

Pipeline

↓

Vault

↓

Playbook

↓

Servidor
```

---

# Separación de Responsabilidades

Una buena arquitectura nunca entrega todas las credenciales a todos los administradores.

Ejemplo.

```text
Equipo Linux

↓

Vault Linux
```

---

```text
Equipo DBA

↓

Vault Database
```

---

```text
Equipo Cloud

↓

Vault Cloud
```

---

Cada grupo administra únicamente sus propios secretos.

---

# Diseño Empresarial

```text
Proyecto

├── inventories

├── group_vars

│      ├── all.yml

│      ├── vault_linux.yml

│      ├── vault_database.yml

│      ├── vault_cloud.yml

│      └── vault_network.yml

├── host_vars

├── playbooks

├── templates

└── roles
```

---

# Flujo de Ejecución

```text
Playbook

↓

Leer Variables

↓

Leer Vault

↓

Descifrar

↓

Ejecutar Tasks

↓

Servidor
```

---

# Integración con CI/CD

En Producción normalmente no se utiliza:

```bash
--ask-vault-pass
```

porque no existe un operador escribiendo la contraseña.

Generalmente se utiliza.

```text
Vault Password File
```

o un mecanismo seguro equivalente administrado por la plataforma de automatización.

---

Representación.

```text
Pipeline

↓

Vault Password

↓

Playbook

↓

Infraestructura
```

---

# Despliegue Automatizado

```text
Git Commit

↓

Pipeline

↓

Validaciones

↓

Vault

↓

Playbook

↓

Producción
```

---

# Integración con Git

Los archivos Vault sí pueden almacenarse en Git.

Ejemplo.

```text
vault_database.yml
```

---

Lo que nunca debe almacenarse es.

```text
vault.pass
```

---

Archivo recomendado.

```text
.gitignore
```

Contenido.

```text
vault.pass
```

---

# Versionado

Cada modificación importante debe registrarse.

```text
Git

↓

Commit

↓

Historial

↓

Auditoría
```

---

# Rotación de Secretos

Toda organización debe definir una política.

Ejemplo.

```text
90 días

↓

Rotación

↓

Rekey

↓

Nueva contraseña
```

---

Representación.

```text
Password Antigua

↓

Rekey

↓

Password Nueva

↓

Repositorio actualizado
```

---

# Auditoría

Una revisión periódica debe comprobar.

```text
✓ Contraseñas protegidas

✓ API Keys protegidas

✓ Tokens protegidos

✓ Variables públicas

✓ Variables privadas

✓ Vault IDs

✓ Permisos

✓ Git

✓ Rotación

✓ Documentación
```

---

# Checklist de Seguridad

Antes de Producción.

```text
□ No existen passwords en texto plano.

□ No existe vault.pass en Git.

□ Todos los secretos utilizan Vault.

□ Los permisos son correctos.

□ Los Roles funcionan.

□ Los Playbooks ejecutan correctamente.

□ Los Templates reciben Variables.

□ Se realizaron pruebas.

□ Existe respaldo seguro de la contraseña.

□ Existe documentación.
```

---

# Integración con Roles

Role.

```text
postgres/

↓

Tasks

↓

Templates

↓

Variables

↓

Vault
```

---

Nada cambia para el Role.

Continúa utilizando.

```yaml
{{ postgres_password }}
```

---

# Integración con Templates

Template.

```jinja2
password={{ postgres_password }}
```

Durante la ejecución.

```text
Vault

↓

Descifrado

↓

Archivo generado
```

---

# Integración con Handlers

```text
Template

↓

Archivo actualizado

↓

Notify

↓

Handler

↓

Restart Service
```

Vault no modifica este comportamiento.

---

# Integración con Inventarios

```text
Inventario

↓

group_vars

↓

Vault

↓

Variables

↓

Servidor
```

---

# Integración con host_vars

```text
Servidor

↓

host_vars

↓

Vault

↓

Password individual
```

---

# Infraestructura Empresarial

Supongamos.

```text
2.500 Servidores
```

Distribuidos en.

- Linux
- PostgreSQL
- SQL Server
- Oracle
- Kubernetes
- Nginx
- Apache

Todos administrados mediante.

```text
Roles

↓

Vault

↓

Playbooks
```

---

# Beneficios

- Automatización segura.
- Protección de secretos.
- Reutilización.
- Escalabilidad.
- Auditoría.
- Integración con Git.
- Integración con CI/CD.
- Administración centralizada.
- Separación por equipos.
- Menor riesgo operativo.

---

# Troubleshooting

## Error

```text
Decryption failed
```

Causa.

Contraseña incorrecta.

---

Solución.

Verificar.

- Password.
- Vault ID.
- Archivo utilizado.

---

## Error

```text
Attempting to decrypt but no vault secrets found
```

Generalmente significa que Ansible no recibió ninguna contraseña para descifrar el archivo.

---

Solución.

Utilizar.

```bash
--ask-vault-pass
```

o.

```bash
--vault-password-file
```

---

## Error

```text
No vault secrets found
```

Causa.

El Playbook necesita un Vault.

No se proporcionó.

---

## Error

```text
Undefined Variable
```

Generalmente.

La Variable existe.

Pero el archivo Vault no fue cargado.

---

## Error

```text
Permission denied
```

Revisar.

```text
vault.pass
```

---

## Error

```text
Wrong Vault ID
```

Cuando existen múltiples Vault IDs.

Debe utilizarse el correcto.

---

## Error

```text
Archivo cifrado corrupto
```

Generalmente ocurre porque alguien modificó el archivo Vault utilizando un editor de texto convencional en lugar de `ansible-vault edit`.

---

## Error

```text
Variables duplicadas
```

Puede suceder cuando una misma Variable está definida tanto en un archivo público como en un archivo Vault.

---

# Estrategia de Recuperación

Si ocurre una pérdida de credenciales.

Procedimiento.

```text
Recuperar Password

↓

Abrir Vault

↓

Validar Secretos

↓

Ejecutar Playbook
```

---

# Documentación

Toda empresa debería documentar.

- Qué Vault existen.
- Qué Variables contienen.
- Quién administra cada Vault.
- Política de rotación.
- Procedimiento de recuperación.
- Responsable de auditorías.

---

# Buenas Prácticas

- Mantener los secretos separados por ambiente.
- Utilizar Vault IDs cuando existan múltiples equipos.
- Utilizar `encrypt_string` para pocas Variables sensibles.
- Utilizar archivos Vault completos cuando existan muchos secretos.
- Proteger el archivo `vault.pass`.
- Utilizar permisos restrictivos.
- Realizar rotación periódica.
- Ejecutar pruebas antes de Producción.
- Versionar archivos Vault mediante Git.
- Documentar todos los secretos administrados.
- Mantener los Roles independientes del mecanismo de almacenamiento de secretos.
- Auditar periódicamente el uso de Variables sensibles.

---

# Errores Comunes

## Error 1

Subir.

```text
vault.pass
```

al repositorio.

---

## Error 2

Guardar secretos en texto plano.

---

## Error 3

No realizar rotación.

---

## Error 4

No utilizar Vault IDs.

---

## Error 5

Editar archivos cifrados utilizando:

```text
nano
```

---

## Error 6

Compartir la contraseña por correo electrónico o mensajería sin protección.

---

## Error 7

No realizar respaldos seguros de la contraseña de Vault.

---

## Error 8

No documentar la arquitectura de secretos.

---

## Error 9

Reutilizar la misma contraseña para todos los entornos.

---

## Error 10

No probar los Playbooks después de modificar los secretos.

---

# Laboratorio Integral RHCSA

## Escenario

Una empresa administrará.

- 2.000 servidores Linux.
- 500 PostgreSQL.
- 400 SQL Server.
- 300 Oracle.
- 250 Kubernetes.
- 300 Apache.

Toda la infraestructura utilizará Ansible Vault.

---

## Laboratorio 1

Crear.

```text
vault_linux.yml
```

---

## Laboratorio 2

Crear.

```text
vault_database.yml
```

---

## Laboratorio 3

Crear.

```text
vault_cloud.yml
```

---

## Laboratorio 4

Utilizar Variables cifradas mediante.

```bash
encrypt_string
```

---

## Laboratorio 5

Crear múltiples Vault IDs.

- Linux
- DBA
- Cloud

---

## Laboratorio 6

Ejecutar Playbooks utilizando.

```bash
--vault-id
```

---

## Laboratorio 7

Realizar una rotación completa de secretos utilizando:

```bash
ansible-vault rekey
```

y comprobar que los Playbooks continúan funcionando correctamente.

---

## Laboratorio 8

Diseñar una estructura empresarial donde los Roles de Apache, PostgreSQL y SQL Server consuman Variables protegidas desde archivos Vault independientes.

---

## Laboratorio 9

Simular una auditoría revisando:

- Variables públicas.
- Variables cifradas.
- Archivos Vault.
- Permisos.
- Archivos excluidos mediante `.gitignore`.
- Políticas de rotación.

---

## Laboratorio 10

Construir una solución completa que incluya:

- Inventarios.
- `group_vars`.
- `host_vars`.
- Roles.
- Templates.
- Handlers.
- Archivos Vault.
- Variables cifradas con `encrypt_string`.
- Múltiples Vault IDs.
- Validaciones previas al despliegue.
- Integración con Git.
- Flujo preparado para CI/CD.

---

# Preguntas de Repaso

1. ¿Cómo se integra Ansible Vault con un pipeline de CI/CD?
2. ¿Qué información nunca debe almacenarse en el repositorio Git?
3. ¿Por qué es recomendable separar los secretos por equipos o ambientes?
4. ¿Qué ventajas ofrece utilizar múltiples Vault IDs?
5. ¿Qué diferencias existen entre un archivo Vault completo y `encrypt_string`?
6. ¿Cómo se integra Vault con Roles y Templates?
7. ¿Qué verificaciones deberían realizarse antes de un despliegue a Producción?
8. ¿Qué errores pueden provocar un fallo al descifrar un archivo Vault?
9. ¿Qué elementos debe incluir una política de auditoría para secretos?
10. ¿Cómo diseñarías una arquitectura segura para administrar secretos en una infraestructura con miles de servidores?

---

# Resumen del Capítulo

En este capítulo estudiamos **Ansible Vault**, la solución oficial de Ansible para proteger información sensible como contraseñas, API Keys, certificados, tokens y credenciales de bases de datos. Aprendimos a crear, visualizar, editar y volver a cifrar archivos Vault, así como a proteger Variables individuales mediante `encrypt_string`.

También analizamos la integración de Vault con Playbooks, Roles, Templates, `group_vars`, `host_vars` y múltiples **Vault IDs**, permitiendo separar secretos por ambientes y equipos de trabajo. Finalmente, revisamos estrategias empresariales para el manejo seguro de secretos, rotación de contraseñas, auditorías, integración con Git y preparación para pipelines de CI/CD, siguiendo las mejores prácticas utilizadas en organizaciones de gran escala.

---

# Próximo Capítulo

## **87. Galaxy y Colecciones (Ansible Galaxy & Collections)**

En el siguiente capítulo aprenderemos a utilizar **Ansible Galaxy**, instalar Roles y Colecciones, administrar dependencias mediante `requirements.yml`, reutilizar contenido creado por la comunidad, publicar Roles propios y organizar proyectos empresariales utilizando el ecosistema oficial de Ansible.

----




