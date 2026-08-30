# 58. Laboratorio Práctico: Administración del Software

> **Módulo 8: Administración del Software y Repositorios**  
> **Página 58 del Manual RHCSA**

---

# Objetivos del laboratorio

Al finalizar este laboratorio serás capaz de:

- Consultar paquetes instalados y disponibles.
- Buscar software mediante DNF.
- Instalar, actualizar, reinstalar y eliminar paquetes.
- Consultar información con RPM.
- Configurar un repositorio local.
- Crear metadatos con `createrepo_c`.
- Trabajar con AppStream y módulos.
- Consultar y revertir transacciones de DNF.
- Verificar firmas GPG.
- Diagnosticar errores comunes de paquetes y repositorios.
- Aplicar buenas prácticas similares a las evaluadas en RHCSA.

---

# Descripción del escenario

Eres el administrador de un servidor Red Hat Enterprise Linux utilizado por el equipo de desarrollo.

El servidor necesita:

- Herramientas de diagnóstico.
- Un servidor web Apache.
- Un repositorio local.
- Un entorno de aplicaciones administrado mediante AppStream.
- Verificación de firmas de paquetes.
- Registro y revisión de transacciones DNF.

Tu responsabilidad será configurar y validar el entorno sin comprometer la estabilidad del sistema.

---

# Requisitos del laboratorio

Se recomienda disponer de:

- RHEL 8, RHEL 9 o una distribución compatible.
- Acceso a una cuenta con privilegios `sudo`.
- Conectividad con repositorios.
- Al menos 2 GB de espacio disponible.
- Una terminal.
- Una imagen ISO de RHEL, opcional.
- Varios paquetes RPM para crear un repositorio local.

---

# Topología del laboratorio

```text
Administrador
    │
    ▼
Servidor RHEL
    │
    ├── RPM
    ├── DNF
    ├── BaseOS
    ├── AppStream
    ├── Repositorio local
    ├── Claves GPG
    └── Historial de transacciones
```

---

# Normas del laboratorio

Antes de comenzar:

- Lee cada tarea completamente.
- No utilices `--force`.
- No utilices `--nodeps`.
- No desactives GPG permanentemente.
- Revisa cada transacción antes de confirmarla.
- Registra los comandos utilizados.
- Evita modificar paquetes críticos.
- Utiliza paquetes pequeños para las pruebas.

---

# Preparación inicial

Verifica la versión del sistema:

```bash
cat /etc/redhat-release
```

También:

```bash
cat /etc/os-release
```

Verifica el usuario actual:

```bash
whoami
```

Confirma que puedes utilizar `sudo`:

```bash
sudo -v
```

---

# Crear un directorio de trabajo

```bash
mkdir -p ~/laboratorio-software
```

Entra al directorio:

```bash
cd ~/laboratorio-software
```

Crea un archivo para registrar resultados:

```bash
touch resultados.txt
```

---

# Registrar información inicial

```bash
{
    echo "===== LABORATORIO DE SOFTWARE ====="
    echo "Fecha: $(date)"
    echo "Host: $(hostname)"
    echo "Sistema:"
    cat /etc/redhat-release
    echo
    echo "Kernel:"
    uname -r
} | tee resultados.txt
```

---

# Parte 1: Consultar el estado de DNF

## Tarea 1.1: Verificar la versión

```bash
dnf --version
```

Registra la salida:

```bash
dnf --version | tee -a resultados.txt
```

---

## Tarea 1.2: Consultar repositorios habilitados

```bash
dnf repolist
```

Registra:

```bash
dnf repolist | tee -a resultados.txt
```

---

## Tarea 1.3: Mostrar todos los repositorios

```bash
dnf repolist all
```

Responde:

1. ¿Cuántos repositorios están habilitados?
2. ¿Cuántos están deshabilitados?
3. ¿Existe un repositorio AppStream?
4. ¿Existe un repositorio BaseOS?

---

## Tarea 1.4: Consultar la configuración principal

```bash
cat /etc/dnf/dnf.conf
```

Identifica los valores de:

- `gpgcheck`
- `installonly_limit`
- `clean_requirements_on_remove`
- `best`
- `skip_if_unavailable`

---

# Parte 2: Búsqueda e información de paquetes

## Tarea 2.1: Buscar el paquete `tree`

```bash
dnf search tree
```

---

## Tarea 2.2: Consultar información

```bash
dnf info tree
```

Identifica:

- Nombre.
- Arquitectura.
- Versión.
- Release.
- Tamaño.
- Repositorio.
- Licencia.
- Descripción.

---

## Tarea 2.3: Consultar si está instalado

```bash
rpm -q tree
```

También:

```bash
dnf list installed tree
```

---

## Tarea 2.4: Buscar el paquete que proporciona un comando

```bash
dnf provides /usr/bin/tree
```

Otro ejemplo:

```bash
dnf provides /usr/bin/curl
```

---

## Tarea 2.5: Consultar archivos de un paquete disponible

```bash
dnf repoquery --list tree
```

Si `repoquery` no está disponible:

```bash
sudo dnf install dnf-plugins-core
```

---

# Parte 3: Instalación de paquetes

## Tarea 3.1: Simular la instalación

```bash
sudo dnf install tree --assumeno
```

Analiza:

- Paquete principal.
- Dependencias.
- Tamaño de descarga.
- Repositorio de origen.

---

## Tarea 3.2: Instalar el paquete

```bash
sudo dnf install tree
```

---

## Tarea 3.3: Verificar la instalación

```bash
rpm -q tree
```

```bash
dnf list installed tree
```

```bash
command -v tree
```

---

## Tarea 3.4: Ejecutar la aplicación

```bash
tree ~/laboratorio-software
```

---

## Tarea 3.5: Consultar información con RPM

```bash
rpm -qi tree
```

---

## Tarea 3.6: Listar archivos instalados

```bash
rpm -ql tree
```

---

## Tarea 3.7: Consultar documentación

```bash
rpm -qd tree
```

---

## Tarea 3.8: Consultar dependencias

```bash
rpm -qR tree
```

---

# Parte 4: Verificación de archivos instalados

## Tarea 4.1: Verificar integridad

```bash
rpm -V tree
```

Si no aparece salida, no se detectaron diferencias.

---

## Tarea 4.2: Identificar el propietario de un archivo

```bash
rpm -qf /usr/bin/tree
```

---

## Tarea 4.3: Reinstalar el paquete

Simula:

```bash
sudo dnf reinstall tree --assumeno
```

Después:

```bash
sudo dnf reinstall tree
```

---

## Tarea 4.4: Verificar nuevamente

```bash
rpm -V tree
```

---

# Parte 5: Historial de DNF

## Tarea 5.1: Consultar el historial

```bash
dnf history
```

Identifica la transacción correspondiente a la instalación de `tree`.

---

## Tarea 5.2: Consultar los detalles

```bash
dnf history info ID
```

Sustituye `ID` por el número correspondiente.

Identifica:

- Usuario.
- Fecha y hora.
- Comando.
- Paquetes modificados.
- Resultado.
- Código de retorno.

---

## Tarea 5.3: Consultar historial del paquete

```bash
dnf history list tree
```

---

## Tarea 5.4: Simular una reversión

```bash
sudo dnf history undo ID --assumeno
```

Revisa qué acciones propone DNF.

---

## Tarea 5.5: Revertir la transacción

```bash
sudo dnf history undo ID
```

Verifica:

```bash
rpm -q tree
```

Resultado esperado:

```text
package tree is not installed
```

---

## Tarea 5.6: Repetir la transacción

```bash
sudo dnf history redo ID
```

Verifica:

```bash
rpm -q tree
```

---

# Parte 6: Actualización y seguridad

## Tarea 6.1: Consultar actualizaciones

```bash
dnf check-update
```

> El comando puede devolver un código distinto de cero cuando existen actualizaciones. Esto no significa necesariamente que haya fallado.

---

## Tarea 6.2: Consultar avisos

```bash
dnf updateinfo summary
```

---

## Tarea 6.3: Consultar avisos de seguridad

```bash
dnf updateinfo list security
```

---

## Tarea 6.4: Simular una actualización de seguridad

```bash
sudo dnf upgrade --security --assumeno
```

No confirmes la instalación durante esta tarea.

---

## Tarea 6.5: Consultar paquetes con varias versiones

```bash
dnf list --showduplicates bash
```

También:

```bash
dnf list --showduplicates kernel
```

---

# Parte 7: Descarga de paquetes

## Tarea 7.1: Instalar complementos

```bash
sudo dnf install dnf-plugins-core
```

---

## Tarea 7.2: Crear directorio

```bash
mkdir -p ~/laboratorio-software/rpm
```

---

## Tarea 7.3: Descargar un paquete

```bash
dnf download \
--destdir=~/laboratorio-software/rpm \
tree
```

---

## Tarea 7.4: Verificar el archivo

```bash
ls -lh ~/laboratorio-software/rpm
```

---

## Tarea 7.5: Consultar información sin instalar

```bash
rpm -qip ~/laboratorio-software/rpm/tree*.rpm
```

---

## Tarea 7.6: Listar archivos del RPM

```bash
rpm -qlp ~/laboratorio-software/rpm/tree*.rpm
```

---

## Tarea 7.7: Consultar scripts

```bash
rpm -qp \
--scripts \
~/laboratorio-software/rpm/tree*.rpm
```

---

## Tarea 7.8: Consultar dependencias

```bash
rpm -qpR ~/laboratorio-software/rpm/tree*.rpm
```

---

# Parte 8: Firmas GPG

## Tarea 8.1: Verificar la firma del paquete

```bash
rpm -Kv ~/laboratorio-software/rpm/tree*.rpm
```

Resultado esperado:

```text
digests signatures OK
```

---

## Tarea 8.2: Consultar claves instaladas

```bash
rpm -qa 'gpg-pubkey*'
```

---

## Tarea 8.3: Consultar una clave

```bash
rpm -qi gpg-pubkey-ID
```

Sustituye el identificador por una clave real.

---

## Tarea 8.4: Consultar claves del sistema

```bash
ls -l /etc/pki/rpm-gpg/
```

---

## Tarea 8.5: Mostrar una huella digital

```bash
gpg --show-keys \
--with-fingerprint \
/etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
```

El nombre puede variar en tu distribución.

---

## Tarea 8.6: Calcular checksum

```bash
sha256sum ~/laboratorio-software/rpm/tree*.rpm
```

Guardar:

```bash
sha256sum ~/laboratorio-software/rpm/tree*.rpm \
> ~/laboratorio-software/tree.sha256
```

Verificar:

```bash
sha256sum -c ~/laboratorio-software/tree.sha256
```

---

# Parte 9: Creación de un repositorio local

## Tarea 9.1: Instalar `createrepo_c`

```bash
sudo dnf install createrepo_c
```

---

## Tarea 9.2: Crear el directorio

```bash
sudo mkdir -p /repositorio/rhcsa
```

---

## Tarea 9.3: Copiar el paquete

```bash
sudo cp \
~/laboratorio-software/rpm/tree*.rpm \
/repositorio/rhcsa/
```

---

## Tarea 9.4: Crear metadatos

```bash
sudo createrepo_c /repositorio/rhcsa
```

---

## Tarea 9.5: Verificar metadatos

```bash
ls -l /repositorio/rhcsa
```

```bash
ls -l /repositorio/rhcsa/repodata
```

Debe existir:

```text
repomd.xml
```

---

## Tarea 9.6: Crear el archivo `.repo`

```bash
sudo vi /etc/yum.repos.d/rhcsa-local.repo
```

Contenido:

```ini
[rhcsa-local]
name=Repositorio local RHCSA
baseurl=file:///repositorio/rhcsa
enabled=1
gpgcheck=1
```

---

## Tarea 9.7: Limpiar caché

```bash
sudo dnf clean all
```

---

## Tarea 9.8: Regenerar metadatos

```bash
sudo dnf makecache
```

---

## Tarea 9.9: Verificar el repositorio

```bash
dnf repolist
```

Debe aparecer:

```text
rhcsa-local
```

---

## Tarea 9.10: Consultar paquetes

```bash
dnf repository-packages rhcsa-local list available
```

También:

```bash
dnf \
--disablerepo="*" \
--enablerepo="rhcsa-local" \
list available
```

---

# Parte 10: Instalación desde el repositorio local

## Tarea 10.1: Eliminar el paquete

```bash
sudo dnf remove tree
```

Verifica:

```bash
rpm -q tree
```

---

## Tarea 10.2: Instalar únicamente desde el repositorio local

```bash
sudo dnf \
--disablerepo="*" \
--enablerepo="rhcsa-local" \
install tree
```

---

## Tarea 10.3: Verificar el origen

```bash
dnf info tree
```

También:

```bash
dnf repoquery \
--installed \
--qf '%{name} %{repoid}' \
tree
```

---

# Parte 11: Administración del repositorio

## Tarea 11.1: Deshabilitar temporalmente

```bash
dnf \
--disablerepo=rhcsa-local \
repolist
```

---

## Tarea 11.2: Deshabilitar permanentemente

```bash
sudo dnf config-manager \
--set-disabled rhcsa-local
```

Verifica:

```bash
dnf repolist all
```

---

## Tarea 11.3: Habilitar nuevamente

```bash
sudo dnf config-manager \
--set-enabled rhcsa-local
```

---

## Tarea 11.4: Actualizar el repositorio

Copia otro paquete RPM:

```bash
dnf download \
--destdir=~/laboratorio-software/rpm \
lsof
```

Después:

```bash
sudo cp \
~/laboratorio-software/rpm/lsof*.rpm \
/repositorio/rhcsa/
```

Actualiza metadatos:

```bash
sudo createrepo_c \
--update \
/repositorio/rhcsa
```

Limpia y actualiza:

```bash
sudo dnf clean metadata
sudo dnf makecache --refresh
```

---

## Tarea 11.5: Verificar el paquete nuevo

```bash
dnf \
--disablerepo="*" \
--enablerepo="rhcsa-local" \
list available
```

---

# Parte 12: Configuración desde una ISO

> Esta sección es opcional y requiere una imagen ISO o un DVD de RHEL.

## Tarea 12.1: Crear punto de montaje

```bash
sudo mkdir -p /mnt/rhel
```

---

## Tarea 12.2: Montar la ISO

```bash
sudo mount \
-o loop \
/ruta/rhel.iso \
/mnt/rhel
```

Si utilizas DVD:

```bash
sudo mount /dev/sr0 /mnt/rhel
```

---

## Tarea 12.3: Verificar contenido

```bash
ls /mnt/rhel
```

Busca:

```text
BaseOS
AppStream
```

---

## Tarea 12.4: Crear repositorios

```bash
sudo vi /etc/yum.repos.d/rhel-iso.repo
```

Contenido:

```ini
[BaseOS-ISO]
name=RHEL BaseOS desde ISO
baseurl=file:///mnt/rhel/BaseOS
enabled=1
gpgcheck=0

[AppStream-ISO]
name=RHEL AppStream desde ISO
baseurl=file:///mnt/rhel/AppStream
enabled=1
gpgcheck=0
```

> `gpgcheck=0` se utiliza aquí únicamente como simplificación de laboratorio. En un entorno real deben configurarse las claves correspondientes.

---

## Tarea 12.5: Actualizar caché

```bash
sudo dnf clean all
sudo dnf makecache
```

---

## Tarea 12.6: Verificar

```bash
dnf repolist
```

---

# Parte 13: AppStream y módulos

## Tarea 13.1: Listar módulos

```bash
dnf module list
```

---

## Tarea 13.2: Buscar un módulo

Ejemplo:

```bash
dnf module list postgresql
```

También puedes probar:

```bash
dnf module list nodejs
```

o:

```bash
dnf module list php
```

Utiliza uno que esté disponible en tu sistema.

---

## Tarea 13.3: Consultar información

```bash
dnf module info postgresql
```

Para un stream:

```bash
dnf module info postgresql:STREAM
```

---

## Tarea 13.4: Identificar indicadores

Busca:

- `[d]`: predeterminado.
- `[e]`: habilitado.
- `[i]`: instalado.
- `[x]`: deshabilitado.

---

## Tarea 13.5: Habilitar un stream

```bash
sudo dnf module enable postgresql:STREAM
```

Sustituye `STREAM` por una versión disponible.

---

## Tarea 13.6: Verificar

```bash
dnf module list postgresql
```

---

## Tarea 13.7: Simular una instalación

```bash
sudo dnf module install \
postgresql:STREAM/server \
--assumeno
```

---

## Tarea 13.8: Restablecer el módulo

```bash
sudo dnf module reset postgresql
```

---

## Tarea 13.9: Verificar el estado

```bash
dnf module list postgresql
```

---

# Parte 14: Dependencias y mantenimiento

## Tarea 14.1: Consultar paquetes innecesarios

```bash
dnf repoquery --unneeded
```

---

## Tarea 14.2: Simular `autoremove`

```bash
sudo dnf autoremove --assumeno
```

No confirmes sin revisar.

---

## Tarea 14.3: Consultar paquetes extras

```bash
dnf repoquery --extras
```

---

## Tarea 14.4: Consultar duplicados

```bash
dnf repoquery --duplicates
```

---

## Tarea 14.5: Consultar dependencias incumplidas

```bash
dnf repoquery --unsatisfied
```

---

## Tarea 14.6: Comprobar la base de paquetes

```bash
sudo dnf check
```

---

# Parte 15: Caché y metadatos

## Tarea 15.1: Consultar espacio

```bash
du -sh /var/cache/dnf
```

---

## Tarea 15.2: Limpiar paquetes

```bash
sudo dnf clean packages
```

---

## Tarea 15.3: Limpiar metadatos

```bash
sudo dnf clean metadata
```

---

## Tarea 15.4: Limpiar todo

```bash
sudo dnf clean all
```

---

## Tarea 15.5: Regenerar caché

```bash
sudo dnf makecache --refresh
```

---

# Parte 16: Diagnóstico de repositorios

## Tarea 16.1: Verificar archivos `.repo`

```bash
ls -l /etc/yum.repos.d/
```

---

## Tarea 16.2: Auditar parámetros principales

```bash
grep -R \
-E "^\[|^name=|^baseurl=|^mirrorlist=|^metalink=|^enabled=|^gpgcheck=|^gpgkey=|^sslverify=" \
/etc/yum.repos.d/
```

---

## Tarea 16.3: Buscar repositorios inseguros

```bash
grep -R \
-E "^gpgcheck=0|^sslverify=0" \
/etc/yum.repos.d/
```

---

## Tarea 16.4: Verificar la URL de un repositorio HTTP

```bash
curl -I URL_DEL_REPOSITORIO
```

---

## Tarea 16.5: Verificar `repomd.xml`

```bash
curl -I \
URL_DEL_REPOSITORIO/repodata/repomd.xml
```

Para el repositorio local:

```bash
ls -l \
/repositorio/rhcsa/repodata/repomd.xml
```

---

# Parte 17: Diagnóstico de errores simulados

## Escenario A: BaseURL incorrecto

Edita temporalmente:

```bash
sudo vi /etc/yum.repos.d/rhcsa-local.repo
```

Cambia:

```ini
baseurl=file:///repositorio/rhcsa
```

por:

```ini
baseurl=file:///repositorio/no-existe
```

Después:

```bash
sudo dnf clean all
sudo dnf makecache
```

Observa el error.

Corrige la ruta y repite:

```bash
sudo dnf clean all
sudo dnf makecache
```

---

## Escenario B: Repositorio deshabilitado

Cambia:

```ini
enabled=1
```

por:

```ini
enabled=0
```

Consulta:

```bash
dnf repolist all
```

Después habilítalo:

```bash
sudo dnf config-manager \
--set-enabled rhcsa-local
```

---

## Escenario C: Metadatos eliminados

Renombra temporalmente:

```bash
sudo mv \
/repositorio/rhcsa/repodata \
/repositorio/rhcsa/repodata.bak
```

Ejecuta:

```bash
sudo dnf clean all
sudo dnf makecache
```

Observa el error.

Restaura:

```bash
sudo mv \
/repositorio/rhcsa/repodata.bak \
/repositorio/rhcsa/repodata
```

---

## Escenario D: Regenerar metadatos

```bash
sudo rm -rf /repositorio/rhcsa/repodata
```

Regenera:

```bash
sudo createrepo_c /repositorio/rhcsa
```

Verifica:

```bash
ls /repositorio/rhcsa/repodata
```

---

# Parte 18: Administración avanzada

## Tarea 18.1: Consultar paquetes instalados por el usuario

```bash
dnf history userinstalled
```

---

## Tarea 18.2: Marcar un paquete

```bash
sudo dnf mark install tree
```

---

## Tarea 18.3: Sincronizar un paquete

Simula:

```bash
sudo dnf distro-sync tree --assumeno
```

---

## Tarea 18.4: Consultar versiones disponibles

```bash
dnf list --showduplicates tree
```

---

## Tarea 18.5: Consultar el origen

```bash
dnf repoquery \
--installed \
--qf '%{name}-%{version}-%{release} %{repoid}' \
tree
```

---

# Parte 19: Logs y auditoría

## Tarea 19.1: Revisar el log de DNF

```bash
sudo tail -n 100 /var/log/dnf.log
```

---

## Tarea 19.2: Buscar instalaciones

```bash
sudo grep -i install \
/var/log/dnf.log | tail -n 20
```

---

## Tarea 19.3: Revisar operaciones RPM

```bash
sudo tail -n 100 /var/log/dnf.rpm.log
```

---

## Tarea 19.4: Consultar historial

```bash
dnf history
```

---

## Tarea 19.5: Registrar el historial

```bash
dnf history | tee -a ~/laboratorio-software/resultados.txt
```

---

# Parte 20: Limpieza del laboratorio

## Tarea 20.1: Eliminar paquetes de prueba

```bash
sudo dnf remove tree lsof
```

Revisa la transacción antes de confirmar.

---

## Tarea 20.2: Eliminar el repositorio local

```bash
sudo rm -f \
/etc/yum.repos.d/rhcsa-local.repo
```

---

## Tarea 20.3: Eliminar los paquetes del repositorio

```bash
sudo rm -rf /repositorio/rhcsa
```

---

## Tarea 20.4: Limpiar caché

```bash
sudo dnf clean all
```

---

## Tarea 20.5: Regenerar metadatos

```bash
sudo dnf makecache
```

---

## Tarea 20.6: Verificar repositorios

```bash
dnf repolist
```

---

# Evaluación práctica RHCSA

Completa las siguientes tareas sin consultar las soluciones.

## Reto 1

Instala el paquete `zip`.

Verifica:

- Versión.
- Archivos.
- Dependencias.
- Documentación.
- Propietario de `/usr/bin/zip`.

---

## Reto 2

Descarga el paquete `unzip` sin instalarlo.

Después:

- Consulta su información.
- Lista sus archivos.
- Consulta sus scripts.
- Verifica su firma.
- Calcula su checksum.

---

## Reto 3

Crea un repositorio local en:

```text
/opt/repositorio-rhcsa
```

Debe contener:

- `zip`
- `unzip`
- `tree`

El identificador debe ser:

```text
repo-practica
```

El nombre debe ser:

```text
Repositorio de práctica RHCSA
```

---

## Reto 4

Configura el repositorio con:

```ini
enabled=1
gpgcheck=1
```

Después instala `tree` utilizando únicamente ese repositorio.

---

## Reto 5

Deshabilita el repositorio permanentemente.

Verifica su estado.

Después habilítalo únicamente para una operación.

---

## Reto 6

Consulta la última transacción DNF.

Identifica:

- ID.
- Usuario.
- Comando.
- Fecha.
- Paquetes afectados.
- Resultado.

---

## Reto 7

Revierta una instalación de prueba mediante:

```bash
dnf history undo
```

Después repite la transacción con:

```bash
dnf history redo
```

---

## Reto 8

Consulta los módulos disponibles.

Selecciona uno y documenta:

- Nombre.
- Streams.
- Stream predeterminado.
- Perfiles.
- Estado.

---

## Reto 9

Audita todos los archivos `.repo`.

Identifica cualquier entrada con:

```ini
gpgcheck=0
```

o:

```ini
sslverify=0
```

---

## Reto 10

Genera un reporte final en:

```text
~/laboratorio-software/reporte-final.txt
```

Debe contener:

- Información del sistema.
- Repositorios habilitados.
- Paquetes instalados en el laboratorio.
- Transacciones ejecutadas.
- Claves GPG instaladas.
- Resultado de `dnf check`.
- Resultado de `dnf repoquery --extras`.
- Resultado de `dnf repoquery --duplicates`.

---

# Script opcional de recolección

```bash
#!/bin/bash

REPORTE="$HOME/laboratorio-software/reporte-final.txt"

mkdir -p "$(dirname "$REPORTE")"

{
    echo "=================================================="
    echo "REPORTE DE ADMINISTRACIÓN DEL SOFTWARE"
    echo "=================================================="
    echo

    echo "Fecha:"
    date
    echo

    echo "Hostname:"
    hostname
    echo

    echo "Sistema operativo:"
    cat /etc/redhat-release
    echo

    echo "Kernel:"
    uname -r
    echo

    echo "Repositorios habilitados:"
    dnf repolist
    echo

    echo "Últimas transacciones:"
    dnf history | head -n 15
    echo

    echo "Claves GPG:"
    rpm -qa 'gpg-pubkey*'
    echo

    echo "Comprobación DNF:"
    sudo dnf check
    echo

    echo "Paquetes extras:"
    dnf repoquery --extras
    echo

    echo "Paquetes duplicados:"
    dnf repoquery --duplicates
    echo

    echo "Módulos habilitados:"
    dnf module list --enabled
    echo

    echo "=================================================="
    echo "FIN DEL REPORTE"
    echo "=================================================="

} > "$REPORTE" 2>&1

echo "Reporte generado en: $REPORTE"
```

Guardar como:

```text
~/laboratorio-software/generar-reporte.sh
```

Asignar permisos:

```bash
chmod +x \
~/laboratorio-software/generar-reporte.sh
```

Ejecutar:

```bash
~/laboratorio-software/generar-reporte.sh
```

---

# Lista de comprobación

Marca cada tarea completada:

```text
[ ] Consulté la versión de DNF
[ ] Revisé los repositorios
[ ] Busqué paquetes
[ ] Instalé paquetes
[ ] Consulté información RPM
[ ] Verifiqué archivos instalados
[ ] Consulté el historial DNF
[ ] Revertí una transacción
[ ] Repetí una transacción
[ ] Descargué un paquete
[ ] Verifiqué una firma GPG
[ ] Calculé un checksum
[ ] Creé un repositorio local
[ ] Generé metadatos
[ ] Instalé desde un repositorio específico
[ ] Habilité y deshabilité repositorios
[ ] Consulté AppStream
[ ] Revisé módulos y streams
[ ] Consulté dependencias innecesarias
[ ] Revisé los logs
[ ] Generé un reporte final
[ ] Limpié el entorno
```

---

# Criterios de evaluación

| Criterio | Puntuación |
|----------|-----------:|
| Consultas RPM y DNF | 10 |
| Instalación y eliminación | 10 |
| Historial y transacciones | 15 |
| Configuración de repositorios | 20 |
| Creación de metadatos | 10 |
| AppStream y módulos | 10 |
| Verificación GPG | 10 |
| Diagnóstico | 10 |
| Documentación final | 5 |
| **Total** | **100** |

---

# Resultados esperados

Al terminar correctamente debes poder demostrar que:

- Sabes utilizar RPM y DNF.
- Puedes identificar paquetes y archivos.
- Sabes instalar desde diferentes fuentes.
- Puedes crear y registrar un repositorio.
- Comprendes el funcionamiento de AppStream.
- Puedes revisar y revertir transacciones.
- Sabes verificar firmas y checksums.
- Puedes diagnosticar fallos de repositorios.
- Puedes documentar los cambios realizados.

---

# Preguntas de repaso

1. ¿Qué diferencia existe entre RPM y DNF?
2. ¿Qué comando muestra los repositorios habilitados?
3. ¿Qué archivo contiene la configuración global de DNF?
4. ¿Dónde se almacenan los archivos `.repo`?
5. ¿Qué directorio debe existir dentro de un repositorio?
6. ¿Qué función cumple `createrepo_c`?
7. ¿Cómo se habilita un repositorio para una sola operación?
8. ¿Qué diferencia existe entre AppStream y un módulo?
9. ¿Qué representa un stream?
10. ¿Qué función cumple un perfil?
11. ¿Cómo se consulta el historial de DNF?
12. ¿Qué diferencia existe entre `undo` y `redo`?
13. ¿Qué comprueba `rpm -K`?
14. ¿Qué significa `NOKEY`?
15. ¿Por qué debe evitarse `--nogpgcheck`?
16. ¿Qué comando detecta paquetes innecesarios?
17. ¿Qué función cumple `dnf distro-sync`?
18. ¿Qué comando muestra dependencias incumplidas?
19. ¿Qué riesgo tiene `dnf autoremove`?
20. ¿Por qué debe revisarse siempre la transacción antes de confirmarla?

---

# Resumen final del módulo

Durante el Módulo 8 aprendiste a:

- Comprender la arquitectura de paquetes RPM.
- Administrar software con DNF.
- Crear y configurar repositorios.
- Utilizar AppStream, módulos, streams y perfiles.
- Consultar el historial de transacciones.
- Revertir y repetir operaciones.
- Verificar firmas GPG.
- Auditar paquetes y repositorios.
- Diagnosticar errores.
- Aplicar prácticas seguras de administración.

---

# Mapa conceptual del módulo

```text
Administración del Software
        │
        ├── RPM
        │     ├── Consultar
        │     ├── Instalar
        │     ├── Verificar
        │     └── Eliminar
        │
        ├── DNF
        │     ├── Buscar
        │     ├── Instalar
        │     ├── Actualizar
        │     ├── Historial
        │     └── Dependencias
        │
        ├── Repositorios
        │     ├── Locales
        │     ├── HTTP/HTTPS
        │     ├── ISO
        │     └── Metadatos
        │
        ├── AppStream
        │     ├── Módulos
        │     ├── Streams
        │     └── Perfiles
        │
        └── Seguridad
              ├── GPG
              ├── Checksums
              ├── HTTPS
              └── Auditoría
```

---

# Conclusión

La administración del software es una de las responsabilidades más importantes de un administrador de sistemas Linux.

Un administrador debe ser capaz de:

- Instalar únicamente software confiable.
- Mantener control sobre las versiones.
- Verificar la procedencia de los paquetes.
- Configurar fuentes de software.
- Investigar cambios.
- Resolver dependencias.
- Recuperarse de transacciones incorrectas.
- Evitar repositorios inseguros.
- Documentar cada modificación.

> **Objetivo general:** integrar todos los conocimientos relacionados con RPM, DNF, repositorios, AppStream, transacciones y firmas GPG en un laboratorio completo. Al dominar estas tareas estarás preparado para resolver escenarios reales de administración de software y ejercicios prácticos similares a los evaluados en el examen **RHCSA**.