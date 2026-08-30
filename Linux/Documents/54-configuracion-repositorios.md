# 54. Configuración de Repositorios en Red Hat Enterprise Linux

> **Módulo 8: Administración del Software y Repositorios**  
> **Página 54 del Manual RHCSA**

---

# Objetivos de aprendizaje

Al finalizar esta lección serás capaz de:

- Comprender qué es un repositorio de software.
- Consultar los repositorios configurados en el sistema.
- Habilitar y deshabilitar repositorios.
- Crear repositorios personalizados.
- Interpretar archivos con extensión `.repo`.
- Configurar repositorios HTTP y locales.
- Verificar y solucionar problemas de repositorios.
- Aplicar buenas prácticas de seguridad.

---

# Introducción

DNF no instala software por sí solo.

Para localizar, descargar y actualizar paquetes, DNF necesita consultar uno o varios **repositorios**.

Un repositorio contiene:

- Paquetes RPM.
- Metadatos.
- Información de dependencias.
- Firmas y claves GPG.
- Versiones disponibles.
- Grupos y módulos de software.

Cuando ejecutamos:

```bash
sudo dnf install httpd
```

DNF consulta los repositorios habilitados para encontrar el paquete y sus dependencias.

---

# ¿Qué es un repositorio?

Un repositorio es una ubicación organizada que contiene paquetes RPM y sus metadatos.

Puede encontrarse en:

- Un servidor HTTP.
- Un servidor HTTPS.
- Un servidor FTP.
- Un recurso NFS.
- Un directorio local.
- Un DVD o imagen ISO.
- Una infraestructura interna de la empresa.

---

# Arquitectura de un repositorio

```text
Cliente RHEL
    │
    │ dnf install
    ▼
Archivo .repo
    │
    │ baseurl o mirrorlist
    ▼
Repositorio
    │
    ├── Paquetes RPM
    ├── Metadatos
    ├── Firmas GPG
    └── Información de dependencias
```

---

# Repositorios comunes en RHEL

En Red Hat Enterprise Linux suelen encontrarse repositorios como:

| Repositorio | Función |
|-------------|---------|
| BaseOS | Componentes fundamentales del sistema operativo |
| AppStream | Aplicaciones, lenguajes y herramientas |
| CodeReady Builder | Bibliotecas y paquetes para desarrollo |
| Supplementary | Software adicional |
| High Availability | Componentes para clústeres |
| Resilient Storage | Funciones avanzadas de almacenamiento |

La disponibilidad depende de:

- La versión de RHEL.
- La arquitectura.
- La suscripción.
- Los repositorios habilitados.

---

# Consultar los repositorios habilitados

```bash
dnf repolist
```

Ejemplo:

```text
repo id             repo name
rhel-9-baseos-rpms  Red Hat Enterprise Linux 9 BaseOS
rhel-9-appstream    Red Hat Enterprise Linux 9 AppStream
```

---

# Consultar todos los repositorios

```bash
dnf repolist all
```

Esta salida muestra repositorios:

- Habilitados.
- Deshabilitados.
- Disponibles.

---

# Obtener información detallada

```bash
dnf repoinfo
```

Para un repositorio específico:

```bash
dnf repoinfo nombre_repositorio
```

Ejemplo:

```bash
dnf repoinfo rhel-9-baseos-rpms
```

---

# Archivos de configuración

Los repositorios se configuran principalmente en:

```text
/etc/yum.repos.d/
```

Los archivos utilizan la extensión:

```text
.repo
```

Ejemplo:

```text
/etc/yum.repos.d/rhel.repo
```

---

# Listar los archivos de repositorios

```bash
ls -l /etc/yum.repos.d/
```

---

# Estructura de un archivo `.repo`

Ejemplo:

```ini
[repo-laboratorio]
name=Repositorio del laboratorio
baseurl=http://192.168.1.50/repositorio/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-laboratorio
```

---

# Interpretación de los parámetros

| Parámetro | Descripción |
|-----------|-------------|
| `[repo-laboratorio]` | Identificador único del repositorio |
| `name` | Nombre descriptivo |
| `baseurl` | Dirección donde se encuentran los paquetes |
| `enabled` | Indica si está habilitado |
| `gpgcheck` | Activa la verificación GPG |
| `gpgkey` | Ubicación de la clave pública |
| `mirrorlist` | Lista dinámica de servidores espejo |
| `metalink` | Dirección que selecciona espejos y verifica metadatos |
| `exclude` | Excluye paquetes específicos |
| `includepkgs` | Limita los paquetes permitidos |

---

# Identificador del repositorio

La primera línea define el identificador:

```ini
[repo-laboratorio]
```

Debe ser único dentro de la configuración de DNF.

Para utilizarlo:

```bash
dnf --repo=repo-laboratorio list available
```

---

# Nombre descriptivo

```ini
name=Repositorio del laboratorio
```

Este texto aparece en la salida de DNF.

---

# Dirección base

```ini
baseurl=http://192.168.1.50/repositorio/
```

También puede utilizarse HTTPS:

```ini
baseurl=https://repositorio.example.com/rhel9/
```

Repositorio local:

```ini
baseurl=file:///repositorio/
```

---

# Habilitar o deshabilitar

Repositorio habilitado:

```ini
enabled=1
```

Repositorio deshabilitado:

```ini
enabled=0
```

---

# Verificación GPG

Activada:

```ini
gpgcheck=1
```

Desactivada:

```ini
gpgcheck=0
```

La opción recomendada es:

```ini
gpgcheck=1
```

---

# Clave GPG

Ejemplo local:

```ini
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
```

También puede apuntar a una dirección HTTPS:

```ini
gpgkey=https://repositorio.example.com/keys/RPM-GPG-KEY-repo
```

---

# baseurl, mirrorlist y metalink

Un repositorio suele utilizar uno de estos métodos.

## baseurl

Dirección directa:

```ini
baseurl=https://servidor.example.com/repo/
```

## mirrorlist

Lista de servidores espejo:

```ini
mirrorlist=https://servidor.example.com/mirrorlist
```

## metalink

Proporciona espejos y validación adicional:

```ini
metalink=https://servidor.example.com/metalink
```

Normalmente no se combinan en el mismo repositorio activo.

---

# Crear un repositorio manualmente

Crear el archivo:

```bash
sudo vi /etc/yum.repos.d/laboratorio.repo
```

Contenido:

```ini
[laboratorio]
name=Repositorio del laboratorio
baseurl=http://192.168.1.50/repositorio/
enabled=1
gpgcheck=0
```

Guardar el archivo.

Limpiar la caché:

```bash
sudo dnf clean all
```

Regenerar metadatos:

```bash
sudo dnf makecache
```

Verificar:

```bash
dnf repolist
```

> `gpgcheck=0` solo debe utilizarse en laboratorios controlados. En producción se recomienda verificar siempre las firmas.

---

# Crear un repositorio con `dnf config-manager`

Primero instalar los complementos:

```bash
sudo dnf install dnf-plugins-core
```

Agregar un repositorio:

```bash
sudo dnf config-manager \
--add-repo \
https://repositorio.example.com/rhel9/repo.repo
```

En algunas versiones recientes de DNF puede utilizarse:

```bash
sudo dnf config-manager addrepo \
--from-repofile=https://repositorio.example.com/rhel9/repo.repo
```

La sintaxis disponible puede comprobarse con:

```bash
dnf config-manager --help
```

---

# Habilitar un repositorio

```bash
sudo dnf config-manager \
--set-enabled laboratorio
```

Verificar:

```bash
dnf repolist enabled
```

---

# Deshabilitar un repositorio

```bash
sudo dnf config-manager \
--set-disabled laboratorio
```

---

# Usar un repositorio temporalmente

Instalar utilizando únicamente un repositorio:

```bash
sudo dnf \
--disablerepo="*" \
--enablerepo="laboratorio" \
install paquete
```

---

# Deshabilitar un repositorio para un comando

```bash
sudo dnf \
--disablerepo=laboratorio \
update
```

Esto no modifica su configuración permanente.

---

# Habilitar temporalmente un repositorio

```bash
sudo dnf \
--enablerepo=laboratorio \
install paquete
```

---

# Consultar paquetes de un repositorio

```bash
dnf repository-packages laboratorio list
```

Solo paquetes disponibles:

```bash
dnf repository-packages laboratorio list available
```

Paquetes instalados desde ese repositorio:

```bash
dnf repository-packages laboratorio list installed
```

---

# Identificar el repositorio de un paquete

```bash
dnf info httpd
```

Buscar el campo:

```text
From repo
```

También:

```bash
dnf repoquery \
--installed \
--qf '%{name} %{repoid}' \
httpd
```

---

# Configurar un repositorio local desde una ISO

Un escenario común en RHCSA consiste en utilizar una imagen ISO de RHEL como repositorio local.

---

# Paso 1: Crear el punto de montaje

```bash
sudo mkdir -p /mnt/rhel
```

---

# Paso 2: Montar la ISO

Ejemplo:

```bash
sudo mount \
-o loop \
/ruta/rhel.iso \
/mnt/rhel
```

Si el DVD está conectado directamente:

```bash
sudo mount /dev/sr0 /mnt/rhel
```

---

# Paso 3: Verificar el contenido

```bash
ls /mnt/rhel
```

En una ISO de RHEL pueden aparecer:

```text
BaseOS
AppStream
```

---

# Paso 4: Crear el repositorio BaseOS

```bash
sudo vi /etc/yum.repos.d/rhel-local.repo
```

Agregar:

```ini
[BaseOS-local]
name=RHEL BaseOS local
baseurl=file:///mnt/rhel/BaseOS
enabled=1
gpgcheck=0
```

---

# Paso 5: Agregar AppStream

En el mismo archivo:

```ini
[AppStream-local]
name=RHEL AppStream local
baseurl=file:///mnt/rhel/AppStream
enabled=1
gpgcheck=0
```

---

# Paso 6: Actualizar la caché

```bash
sudo dnf clean all
sudo dnf makecache
```

---

# Paso 7: Verificar

```bash
dnf repolist
```

---

# Montaje persistente de la ISO

Para que el repositorio siga disponible después del reinicio, puede configurarse en:

```text
/etc/fstab
```

Ejemplo:

```fstab
/ruta/rhel.iso /mnt/rhel iso9660 loop,ro 0 0
```

Después:

```bash
sudo mount -a
```

Verificar:

```bash
findmnt /mnt/rhel
```

---

# Crear un repositorio desde paquetes RPM

Si tienes un directorio con paquetes:

```text
/repositorio/paquetes/
```

Instala la herramienta:

```bash
sudo dnf install createrepo_c
```

Crear los metadatos:

```bash
sudo createrepo_c /repositorio/paquetes/
```

Esto genera:

```text
/repositorio/paquetes/repodata/
```

Sin el directorio `repodata`, DNF no reconocerá la ubicación como repositorio válido.

---

# Actualizar los metadatos de un repositorio

Después de agregar nuevos paquetes:

```bash
sudo createrepo_c \
--update \
/repositorio/paquetes/
```

---

# Publicar un repositorio con Apache

Instalar Apache:

```bash
sudo dnf install httpd
```

Crear el directorio:

```bash
sudo mkdir -p /var/www/html/repositorio
```

Copiar los paquetes:

```bash
sudo cp *.rpm /var/www/html/repositorio/
```

Crear los metadatos:

```bash
sudo createrepo_c /var/www/html/repositorio/
```

Corregir contextos SELinux:

```bash
sudo restorecon \
-Rv \
/var/www/html/repositorio
```

Iniciar Apache:

```bash
sudo systemctl enable --now httpd
```

Abrir el Firewall:

```bash
sudo firewall-cmd \
--permanent \
--add-service=http

sudo firewall-cmd --reload
```

El repositorio puede utilizarse mediante:

```ini
baseurl=http://IP_SERVIDOR/repositorio/
```

---

# Configuración principal de DNF

Archivo:

```text
/etc/dnf/dnf.conf
```

Ejemplo:

```ini
[main]
gpgcheck=True
installonly_limit=3
clean_requirements_on_remove=True
best=True
skip_if_unavailable=False
```

---

# Parámetros comunes de `dnf.conf`

| Parámetro | Descripción |
|-----------|-------------|
| `gpgcheck` | Verifica las firmas de paquetes |
| `installonly_limit` | Cantidad de kernels que se conservan |
| `clean_requirements_on_remove` | Elimina dependencias innecesarias |
| `best` | Intenta seleccionar la mejor versión disponible |
| `skip_if_unavailable` | Decide si se ignoran repositorios inaccesibles |
| `exclude` | Excluye paquetes |
| `keepcache` | Conserva los paquetes descargados |

---

# Excluir paquetes

En un archivo `.repo`:

```ini
exclude=kernel* postgresql*
```

También en un comando:

```bash
sudo dnf update \
--exclude=kernel*
```

---

# Permitir únicamente ciertos paquetes

```ini
includepkgs=httpd* php*
```

Esto limita los paquetes visibles desde el repositorio.

---

# Red Hat Subscription Management

En sistemas RHEL con suscripción, los repositorios pueden administrarse con:

```bash
subscription-manager
```

Consultar el estado:

```bash
sudo subscription-manager status
```

Listar repositorios disponibles:

```bash
sudo subscription-manager repos --list
```

Listar repositorios habilitados:

```bash
sudo subscription-manager repos --list-enabled
```

Habilitar un repositorio:

```bash
sudo subscription-manager repos \
--enable=ID_REPOSITORIO
```

Deshabilitar:

```bash
sudo subscription-manager repos \
--disable=ID_REPOSITORIO
```

> Los identificadores exactos dependen de la versión, arquitectura y suscripción del sistema.

---

# Verificar conectividad con un repositorio

Comprobar resolución DNS:

```bash
getent hosts repositorio.example.com
```

Probar conectividad HTTPS:

```bash
curl -I https://repositorio.example.com/
```

Consultar metadatos con DNF:

```bash
dnf makecache \
--refresh
```

---

# Solución de problemas

## Error: Cannot download repomd.xml

Ejemplo:

```text
Cannot download repomd.xml
```

Posibles causas:

- Dirección `baseurl` incorrecta.
- Problemas DNS.
- Firewall bloqueando el acceso.
- Repositorio no disponible.
- Falta el directorio `repodata`.
- Certificado TLS inválido.
- Proxy mal configurado.

Comprobar:

```bash
curl -I URL_DEL_REPOSITORIO/repodata/repomd.xml
```

---

## Error: Failed to download metadata

Limpiar la caché:

```bash
sudo dnf clean all
sudo rm -rf /var/cache/dnf/*
sudo dnf makecache
```

---

## Repositorio duplicado

Puede aparecer cuando existen dos archivos `.repo` con el mismo identificador.

Buscar:

```bash
grep -R \
"^\[nombre-repo\]" \
/etc/yum.repos.d/
```

Corregir o eliminar la entrada duplicada.

---

## Clave GPG no instalada

Importar una clave:

```bash
sudo rpm \
--import \
/etc/pki/rpm-gpg/RPM-GPG-KEY-repositorio
```

Listar claves instaladas:

```bash
rpm -qa 'gpg-pubkey*'
```

---

## Repositorio local no funciona

Verificar:

```bash
findmnt /mnt/rhel
```

Comprobar la ruta:

```bash
ls /mnt/rhel/BaseOS/repodata
```

Revisar el archivo:

```bash
cat /etc/yum.repos.d/rhel-local.repo
```

---

## Repositorio deshabilitado

Consultar:

```bash
dnf repolist all
```

Habilitar temporalmente:

```bash
dnf --enablerepo=nombre-repo repolist
```

---

# Verificar la configuración completa

```bash
dnf repolist all
```

```bash
dnf repoinfo
```

```bash
dnf makecache --refresh
```

```bash
grep -R \
-E "^\[|^name=|^baseurl=|^mirrorlist=|^metalink=|^enabled=|^gpgcheck=" \
/etc/yum.repos.d/
```

---

# Flujo recomendado para agregar un repositorio

```text
Identificar la fuente
        │
        ▼
Crear o importar archivo .repo
        │
        ▼
Verificar baseurl y GPG
        │
        ▼
Limpiar caché
        │
        ▼
Generar metadatos
        │
        ▼
Consultar con dnf repolist
        │
        ▼
Instalar un paquete de prueba
```

---

# Herramientas relacionadas

| Herramienta | Función |
|-------------|---------|
| `dnf repolist` | Listar repositorios |
| `dnf repoinfo` | Mostrar información detallada |
| `dnf config-manager` | Administrar repositorios |
| `subscription-manager` | Administrar suscripciones RHEL |
| `createrepo_c` | Crear metadatos de repositorios |
| `rpm --import` | Importar claves GPG |
| `curl` | Probar acceso HTTP o HTTPS |
| `mount` | Montar ISO o medios locales |
| `restorecon` | Restaurar contextos SELinux |

---

# Buenas prácticas RHCSA

✔ Utilizar repositorios oficiales o corporativos confiables.

✔ Mantener habilitada la verificación GPG.

✔ Utilizar HTTPS cuando el repositorio esté en una red externa.

✔ Evitar mezclar repositorios destinados a versiones diferentes de RHEL.

✔ Documentar los repositorios personalizados.

✔ Deshabilitar repositorios que ya no sean necesarios.

✔ Verificar siempre el valor de `baseurl`.

✔ Limpiar y regenerar la caché después de modificar la configuración.

✔ No editar archivos administrados automáticamente sin conocer su origen.

---

# Errores comunes

## Usar una URL que apunta al directorio incorrecto

El `baseurl` debe apuntar a la ubicación que contiene:

```text
repodata/
```

---

## Desactivar la validación GPG en producción

Configurar:

```ini
gpgcheck=0
```

reduce la seguridad y permite instalar paquetes sin verificar su autenticidad.

---

## Mezclar versiones de RHEL

No debe utilizarse un repositorio de RHEL 8 en RHEL 9.

---

## Olvidar montar la ISO

Si el repositorio utiliza:

```ini
baseurl=file:///mnt/rhel/BaseOS
```

y la ISO no está montada, el repositorio fallará.

---

## Olvidar crear metadatos

Un directorio con archivos RPM no es automáticamente un repositorio DNF.

Debe ejecutarse:

```bash
createrepo_c
```

---

## Modificar un repositorio sin limpiar la caché

DNF puede continuar utilizando metadatos anteriores.

Ejecutar:

```bash
sudo dnf clean all
sudo dnf makecache
```

---

# Comandos importantes RHCSA

| Comando | Descripción |
|---------|-------------|
| `dnf repolist` | Mostrar repositorios habilitados |
| `dnf repolist all` | Mostrar todos los repositorios |
| `dnf repoinfo` | Información detallada |
| `dnf config-manager --set-enabled` | Habilitar un repositorio |
| `dnf config-manager --set-disabled` | Deshabilitar un repositorio |
| `dnf --enablerepo` | Habilitar temporalmente |
| `dnf --disablerepo` | Deshabilitar temporalmente |
| `dnf clean all` | Limpiar la caché |
| `dnf makecache` | Descargar metadatos |
| `createrepo_c` | Crear un repositorio |
| `rpm --import` | Importar una clave GPG |
| `subscription-manager repos` | Administrar repositorios de RHEL |

---

# Resumen

En esta lección aprendiste a:

- Comprender el funcionamiento de los repositorios.
- Consultar repositorios con DNF.
- Interpretar archivos `.repo`.
- Habilitar y deshabilitar repositorios.
- Configurar repositorios HTTP y locales.
- Crear repositorios mediante `createrepo_c`.
- Utilizar una ISO de RHEL como fuente de paquetes.
- Diagnosticar errores de metadatos, conectividad y firmas.
- Aplicar buenas prácticas de seguridad.

---

# Laboratorio práctico RHCSA

## Escenario 1: Consultar repositorios

Lista los repositorios habilitados:

```bash
dnf repolist
```

Muestra también los deshabilitados:

```bash
dnf repolist all
```

Consulta la información detallada:

```bash
dnf repoinfo
```

---

## Escenario 2: Crear un repositorio local de prueba

Crea el directorio:

```bash
sudo mkdir -p /repositorio/paquetes
```

Copia varios paquetes RPM al directorio.

Instala la herramienta:

```bash
sudo dnf install createrepo_c
```

Genera los metadatos:

```bash
sudo createrepo_c /repositorio/paquetes
```

Comprueba que existe:

```bash
ls /repositorio/paquetes/repodata
```

---

## Escenario 3: Registrar el repositorio

Crea:

```bash
sudo vi /etc/yum.repos.d/laboratorio.repo
```

Contenido:

```ini
[laboratorio-local]
name=Repositorio local RHCSA
baseurl=file:///repositorio/paquetes
enabled=1
gpgcheck=0
```

Actualiza la caché:

```bash
sudo dnf clean all
sudo dnf makecache
```

Verifica:

```bash
dnf repolist
```

---

## Escenario 4: Administrar el repositorio

Deshabilítalo:

```bash
sudo dnf config-manager \
--set-disabled laboratorio-local
```

Verifica:

```bash
dnf repolist all
```

Habilítalo nuevamente:

```bash
sudo dnf config-manager \
--set-enabled laboratorio-local
```

---

## Escenario 5: Utilizar únicamente el repositorio local

Lista sus paquetes:

```bash
dnf \
--disablerepo="*" \
--enablerepo="laboratorio-local" \
list available
```

Instala uno de los paquetes disponibles:

```bash
sudo dnf \
--disablerepo="*" \
--enablerepo="laboratorio-local" \
install nombre_paquete
```

---

## Escenario 6: Diagnóstico

Comprueba que el repositorio contiene metadatos:

```bash
ls /repositorio/paquetes/repodata/repomd.xml
```

Consulta la configuración:

```bash
cat /etc/yum.repos.d/laboratorio.repo
```

Regenera la caché:

```bash
sudo dnf clean all
sudo dnf makecache --refresh
```

> **Objetivo general:** dominar la configuración y administración de repositorios en Red Hat Enterprise Linux. Saber crear, habilitar, deshabilitar y diagnosticar repositorios locales o remotos es una competencia fundamental para administrar software con DNF y resolver tareas prácticas del examen **RHCSA**.