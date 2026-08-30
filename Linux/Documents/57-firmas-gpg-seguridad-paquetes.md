# 57. Firmas GPG y Seguridad de Paquetes

> **Módulo 8: Administración del Software y Repositorios**  
> **Página 57 del Manual RHCSA**

---

# Objetivos de aprendizaje

Al finalizar esta lección serás capaz de:

- Comprender qué es una firma GPG.
- Diferenciar integridad, autenticidad y confianza.
- Verificar paquetes RPM antes de instalarlos.
- Consultar las claves GPG instaladas.
- Importar y eliminar claves públicas.
- Configurar la validación GPG en repositorios.
- Interpretar errores de firma.
- Comprender los riesgos de utilizar `--nogpgcheck`.
- Aplicar buenas prácticas para instalar software de forma segura.

---

# Introducción

Instalar software no consiste únicamente en descargar un paquete y ejecutarlo.

Antes de confiar en un paquete, debemos responder tres preguntas:

```text
¿El archivo fue modificado?

¿Quién creó o publicó el paquete?

¿La fuente es confiable?
```

Las firmas GPG ayudan a responder estas preguntas.

En Red Hat Enterprise Linux, RPM y DNF pueden verificar digitalmente los paquetes antes de instalarlos.

Esto reduce el riesgo de instalar:

- Paquetes modificados.
- Software malicioso.
- Archivos corruptos.
- Paquetes provenientes de fuentes no confiables.
- Versiones falsas de aplicaciones legítimas.

---

# ¿Qué es GPG?

GPG significa:

```text
GNU Privacy Guard
```

Es una implementación libre del estándar OpenPGP.

GPG utiliza criptografía de clave pública.

Esto significa que existe un par de claves:

```text
Clave privada

Clave pública
```

La clave privada se utiliza para firmar.

La clave pública se utiliza para verificar.

---

# Funcionamiento básico

```text
Publicador del paquete
        │
        ├── Crea el paquete RPM
        │
        ├── Calcula su resumen criptográfico
        │
        └── Firma con su clave privada
                │
                ▼
          Paquete firmado
                │
                ▼
          Usuario descarga
                │
                ▼
     Verifica con la clave pública
                │
        ┌───────┴────────┐
        ▼                ▼
    Firma válida     Firma inválida
```

---

# Clave privada y clave pública

| Clave privada | Clave pública |
|---------------|---------------|
| Debe mantenerse secreta | Puede distribuirse |
| Firma paquetes | Verifica firmas |
| Identifica al publicador | Permite validar al publicador |
| No debe compartirse | Puede instalarse en los clientes |

---

# ¿Qué garantiza una firma GPG?

Una firma válida ayuda a comprobar:

- Que el paquete no fue modificado después de ser firmado.
- Que el paquete fue firmado por una clave conocida.
- Que el contenido conserva su integridad.
- Que el publicador posee la clave privada asociada.

---

# ¿Qué no garantiza una firma GPG?

Una firma válida no garantiza automáticamente que:

- El software no tenga vulnerabilidades.
- El paquete esté bien configurado.
- El proveedor sea confiable.
- La clave pública haya sido obtenida de una fuente legítima.
- La aplicación sea segura para tu entorno.

Por ello, la firma es solo una parte del proceso de seguridad.

---

# Integridad, autenticidad y confianza

## Integridad

Confirma que el archivo no fue modificado.

```text
Archivo original = Archivo descargado
```

---

## Autenticidad

Confirma que la firma corresponde a una clave específica.

```text
Paquete firmado por una clave conocida
```

---

## Confianza

Es la decisión del administrador de aceptar esa clave como legítima.

```text
¿Confío realmente en el propietario de esta clave?
```

---

# Resumen conceptual

```text
Checksum
    │
    └── Comprueba integridad

Firma digital
    │
    ├── Comprueba integridad
    └── Comprueba autenticidad

Confianza
    │
    └── Depende de cómo se obtuvo y validó la clave
```

---

# Firmas en paquetes RPM

Los paquetes RPM pueden contener:

- Resúmenes criptográficos.
- Firmas digitales.
- Información del firmante.
- Algoritmos utilizados.

RPM permite verificar estos datos antes de instalar el paquete.

---

# Verificar un paquete RPM

```bash
rpm -K paquete.rpm
```

También:

```bash
rpm --checksig paquete.rpm
```

Ejemplo:

```bash
rpm -K httpd-2.4.62-1.el9.x86_64.rpm
```

---

# Ejemplo de resultado correcto

```text
httpd-2.4.62-1.el9.x86_64.rpm: digests signatures OK
```

Esto indica que:

- El resumen del paquete es correcto.
- La firma pudo verificarse.
- La clave necesaria está instalada.

---

# Verificación detallada

```bash
rpm -Kv paquete.rpm
```

La opción:

```text
-v
```

muestra información más detallada.

---

# Posibles resultados

## Firma válida

```text
digests signatures OK
```

---

## Clave no instalada

```text
NOKEY
```

Ejemplo:

```text
Header V4 RSA/SHA256 Signature, key ID abcdef12: NOKEY
```

Esto significa que:

- El paquete está firmado.
- RPM detectó la firma.
- La clave pública necesaria no está instalada.

---

## Firma incorrecta

```text
BAD
```

Esto puede indicar:

- Paquete modificado.
- Archivo corrupto.
- Firma inválida.
- Descarga incompleta.
- Manipulación del contenido.

---

## Paquete sin firma

```text
NOT OK
```

o una salida que indique que no existe firma.

No debe instalarse automáticamente en producción sin verificar cuidadosamente su origen.

---

# Verificar varios paquetes

```bash
rpm -K *.rpm
```

También:

```bash
find /ruta/paquetes \
-type f \
-name "*.rpm" \
-exec rpm -K {} \;
```

---

# Consultar firmas de paquetes instalados

```bash
rpm -q --qf \
'%{NAME} %{VERSION}-%{RELEASE} %{SIGPGP:pgpsig}\n' \
bash
```

En sistemas modernos puede utilizarse:

```bash
rpm -q --qf \
'%{NAME} %{VERSION}-%{RELEASE} %{SIGGPG:pgpsig}\n' \
bash
```

La salida exacta depende de la versión de RPM y del tipo de firma.

---

# Consultar información completa

```bash
rpm -qi bash
```

Aunque `rpm -qi` no siempre muestra todos los detalles de firma, permite revisar:

- Nombre.
- Versión.
- Release.
- Distribuidor.
- Licencia.
- Fecha de instalación.
- Resumen.

---

# Claves GPG en RPM

RPM almacena las claves públicas importadas como paquetes especiales.

Listarlas:

```bash
rpm -qa 'gpg-pubkey*'
```

Ejemplo:

```text
gpg-pubkey-fd431d51-4ae0493b
```

---

# Consultar información de una clave

```bash
rpm -qi gpg-pubkey-fd431d51-4ae0493b
```

La salida puede mostrar:

- Nombre.
- Fecha.
- Identificador.
- Resumen.
- Descripción.

---

# Ver todas las claves con detalles

```bash
for key in $(rpm -qa 'gpg-pubkey*'); do
    rpm -qi "$key"
done
```

---

# Ubicación común de claves GPG

En RHEL, muchas claves se almacenan en:

```text
/etc/pki/rpm-gpg/
```

Listar:

```bash
ls -l /etc/pki/rpm-gpg/
```

Ejemplo:

```text
RPM-GPG-KEY-redhat-release
```

---

# Examinar una clave

```bash
gpg --show-keys \
/etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
```

También:

```bash
gpg --with-fingerprint \
/etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
```

---

# ¿Qué es la huella digital?

La huella digital es una representación única de una clave pública.

Ejemplo conceptual:

```text
ABCD 1234 EFGH 5678 IJKL 9012 MNOP 3456 QRST 7890
```

Debe compararse con una fuente oficial.

---

# Importar una clave GPG

Sintaxis:

```bash
sudo rpm --import archivo_clave
```

Ejemplo:

```bash
sudo rpm --import \
/etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
```

---

# Importar desde una URL

```bash
sudo rpm --import \
https://repositorio.example.com/RPM-GPG-KEY-repo
```

Para entornos de producción es preferible:

1. Descargar la clave.
2. Verificar la huella digital.
3. Importarla localmente.

---

# Procedimiento recomendado

```bash
curl -O \
https://repositorio.example.com/RPM-GPG-KEY-repo
```

Mostrar huella:

```bash
gpg --show-keys \
--with-fingerprint \
RPM-GPG-KEY-repo
```

Comparar con la documentación oficial.

Importar:

```bash
sudo rpm --import RPM-GPG-KEY-repo
```

---

# Verificar que la clave fue importada

```bash
rpm -qa 'gpg-pubkey*'
```

Después:

```bash
rpm -qi gpg-pubkey-ID
```

---

# Eliminar una clave GPG

Primero identifica el paquete:

```bash
rpm -qa 'gpg-pubkey*'
```

Después:

```bash
sudo rpm -e gpg-pubkey-ID
```

Ejemplo:

```bash
sudo rpm -e gpg-pubkey-fd431d51-4ae0493b
```

---

# Precaución al eliminar claves

Eliminar una clave puede provocar que DNF no pueda verificar paquetes futuros.

Antes de eliminar:

- Identifica qué repositorio utiliza la clave.
- Verifica si aún está activa.
- Confirma que no sea una clave oficial del sistema.
- Documenta el cambio.
- Conserva una copia de respaldo.

---

# rpmkeys

La herramienta `rpmkeys` está especializada en firmas y claves RPM.

Verificar un paquete:

```bash
rpmkeys --checksig paquete.rpm
```

Importar una clave:

```bash
sudo rpmkeys --import clave.gpg
```

Listar claves:

```bash
rpm -qa 'gpg-pubkey*'
```

---

# Diferencia entre `rpm` y `rpmkeys`

| Herramienta | Función |
|-------------|---------|
| `rpm` | Administración general de paquetes |
| `rpmkeys` | Verificación de firmas y administración de claves |

---

# Verificación GPG con DNF

DNF verifica automáticamente los paquetes si el repositorio tiene:

```ini
gpgcheck=1
```

Ejemplo:

```ini
[repo-seguro]
name=Repositorio seguro
baseurl=https://repositorio.example.com/rhel9/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-repo
```

---

# Flujo de validación con DNF

```text
dnf install
    │
    ▼
Descarga metadatos
    │
    ▼
Descarga paquete
    │
    ▼
Obtiene clave configurada
    │
    ▼
Verifica firma
    │
    ├── Correcta → instala
    │
    └── Incorrecta → detiene
```

---

# Parámetro `gpgcheck`

Activar:

```ini
gpgcheck=1
```

Desactivar:

```ini
gpgcheck=0
```

La configuración recomendada es:

```ini
gpgcheck=1
```

---

# Parámetro `gpgkey`

Define la ubicación de la clave pública.

Ejemplo local:

```ini
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-repo
```

Ejemplo remoto:

```ini
gpgkey=https://repositorio.example.com/keys/RPM-GPG-KEY-repo
```

---

# Varias claves GPG

Puede especificarse más de una clave.

Ejemplo:

```ini
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-antigua
       file:///etc/pki/rpm-gpg/RPM-GPG-KEY-nueva
```

Esto puede ser útil durante una rotación de claves.

---

# Verificación de metadatos

Algunos repositorios también permiten verificar sus metadatos.

Parámetro:

```ini
repo_gpgcheck=1
```

Ejemplo:

```ini
[repo-seguro]
name=Repositorio seguro
baseurl=https://repo.example.com/rhel9/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-repo
```

---

# Diferencia entre `gpgcheck` y `repo_gpgcheck`

| Parámetro | Verifica |
|-----------|----------|
| `gpgcheck=1` | Firma de los paquetes RPM |
| `repo_gpgcheck=1` | Firma de los metadatos del repositorio |

---

# Configuración global

Archivo:

```text
/etc/dnf/dnf.conf
```

Ejemplo:

```ini
[main]
gpgcheck=True
```

También puede configurarse individualmente en cada archivo `.repo`.

---

# Prioridad de configuración

La configuración específica del repositorio puede modificar el comportamiento global.

Ejemplo:

```ini
[main]
gpgcheck=True
```

Repositorio:

```ini
[repo-prueba]
gpgcheck=0
```

Para ese repositorio se desactiva la verificación.

Esto no se recomienda en producción.

---

# DNF solicita importar una clave

Durante una instalación puede aparecer:

```text
Importing GPG key
```

DNF puede mostrar:

- Identificador de la clave.
- Usuario o propietario.
- Huella digital.
- Origen de la clave.

Antes de aceptar:

```text
Is this ok [y/N]
```

debes validar que la clave pertenezca al repositorio esperado.

---

# Ejemplo conceptual

```text
Importing GPG key 0xABCDEF12:
 Userid     : "Proveedor Example <security@example.com>"
 Fingerprint: 1234 5678 90AB CDEF 1234 5678 90AB CDEF 1234 5678
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-example
```

No confirmes automáticamente sin revisar.

---

# Verificar una clave antes de aceptarla

Consulta la clave:

```bash
gpg --show-keys \
--with-fingerprint \
/etc/pki/rpm-gpg/RPM-GPG-KEY-example
```

Compara la huella con:

- Documentación oficial.
- Portal del proveedor.
- Canal corporativo confiable.
- Medio seguro administrado por la empresa.

---

# Error: Public key is not installed

Ejemplo:

```text
Public key for paquete.rpm is not installed
```

Significa que:

- El paquete está firmado.
- DNF no encuentra la clave pública requerida.

---

# Solución recomendada

1. Identifica el repositorio.

```bash
dnf info paquete
```

2. Revisa el archivo `.repo`.

```bash
grep -R \
-E "^\[|^gpgcheck=|^gpgkey=" \
/etc/yum.repos.d/
```

3. Verifica la clave indicada.

```bash
ls -l /etc/pki/rpm-gpg/
```

4. Comprueba su huella.

```bash
gpg --show-keys \
--with-fingerprint \
archivo_clave
```

5. Importa la clave.

```bash
sudo rpm --import archivo_clave
```

6. Limpia la caché.

```bash
sudo dnf clean all
sudo dnf makecache --refresh
```

7. Reintenta la operación.

---

# Error: GPG check FAILED

Ejemplo:

```text
GPG check FAILED
```

Posibles causas:

- Paquete modificado.
- Clave incorrecta.
- Repositorio comprometido.
- Descarga corrupta.
- Rotación de claves.
- Caché inconsistente.
- Paquete firmado con una clave diferente.

---

# Diagnóstico

Limpiar la caché:

```bash
sudo dnf clean packages
sudo dnf clean metadata
sudo dnf makecache --refresh
```

Descargar nuevamente el paquete.

Verificar manualmente:

```bash
rpm -Kv paquete.rpm
```

Consultar claves:

```bash
rpm -qa 'gpg-pubkey*'
```

Revisar el repositorio:

```bash
dnf repoinfo nombre_repo
```

---

# Error: BAD signature

Ejemplo:

```text
BAD signature
```

Debe considerarse un evento serio.

Acciones recomendadas:

- No instalar el paquete.
- Eliminar el archivo descargado.
- Descargarlo nuevamente desde la fuente oficial.
- Verificar la URL.
- Confirmar la huella digital de la clave.
- Consultar avisos del proveedor.
- Revisar si el repositorio fue comprometido.

---

# Error: NOKEY

Ejemplo:

```text
NOKEY
```

No significa necesariamente que el paquete sea malicioso.

Significa que la clave no está disponible localmente.

Sin embargo, la clave debe obtenerse desde una fuente confiable antes de continuar.

---

# Error: llave expirada

Una clave puede tener fecha de expiración.

Síntomas posibles:

```text
key expired
```

o:

```text
signature is not valid
```

Acciones:

- Consultar la documentación oficial.
- Descargar la clave nueva.
- Verificar la huella.
- Importar la clave nueva.
- Conservar temporalmente la antigua si aún se necesitan paquetes históricos.
- Revisar la configuración del repositorio.

---

# Rotación de claves

Los proveedores pueden reemplazar una clave por motivos como:

- Expiración.
- Cambio de política.
- Cambio de algoritmo.
- Compromiso de seguridad.
- Renovación periódica.

---

# Flujo de rotación segura

```text
Proveedor publica nueva clave
        │
        ▼
Administrador descarga
        │
        ▼
Verifica huella
        │
        ▼
Importa nueva clave
        │
        ▼
Actualiza gpgkey
        │
        ▼
Verifica paquetes
        │
        ▼
Elimina clave antigua cuando corresponda
```

---

# Verificar checksums

Además de las firmas GPG, puede verificarse un checksum.

Ejemplo:

```bash
sha256sum paquete.rpm
```

Resultado:

```text
4b7f...  paquete.rpm
```

Debe compararse con el valor oficial.

---

# Diferencia entre checksum y firma

| Checksum | Firma GPG |
|----------|-----------|
| Comprueba integridad | Comprueba integridad y autenticidad |
| Cualquiera puede recalcularlo | Requiere la clave privada para firmar |
| Debe obtenerse de una fuente confiable | Se verifica con la clave pública |
| No identifica al publicador por sí solo | Vincula el paquete con una clave |

---

# Problema de obtener checksum y paquete del mismo sitio

Si un atacante modifica:

- El paquete.
- El checksum publicado.

Entonces el checksum podría coincidir.

Por ello, una firma digital ofrece una protección adicional, siempre que la clave pública haya sido validada correctamente.

---

# `--nogpgcheck`

DNF permite ignorar la validación:

```bash
sudo dnf install paquete \
--nogpgcheck
```

También puede utilizarse con un repositorio:

```bash
sudo dnf \
--enablerepo=repo-prueba \
install paquete \
--nogpgcheck
```

---

# Riesgos de `--nogpgcheck`

Esta opción omite una protección fundamental.

Puede permitir instalar:

- Paquetes modificados.
- Paquetes falsificados.
- Software malicioso.
- Archivos descargados de un repositorio comprometido.

---

# Cuándo podría utilizarse

Solo en escenarios muy controlados, como:

- Laboratorios.
- Repositorios temporales internos.
- Pruebas aisladas.
- Entornos sin acceso a producción.

Incluso en estos casos, debe verificarse el checksum y la procedencia del archivo.

---

# `gpgcheck=0`

Desactivar permanentemente:

```ini
gpgcheck=0
```

es más riesgoso que utilizar `--nogpgcheck` una sola vez, porque puede pasar desapercibido en futuras instalaciones.

---

# Buscar repositorios inseguros

```bash
grep -R \
-E "^gpgcheck=0|^repo_gpgcheck=0" \
/etc/yum.repos.d/
```

También:

```bash
grep -R \
-E "^\[|^name=|^baseurl=|^gpgcheck=|^repo_gpgcheck=|^gpgkey=" \
/etc/yum.repos.d/
```

---

# Auditar configuración GPG

```bash
for file in /etc/yum.repos.d/*.repo; do
    echo "===== $file ====="
    grep -E \
    "^\[|^name=|^enabled=|^gpgcheck=|^repo_gpgcheck=|^gpgkey=" \
    "$file"
done
```

---

# Verificar todos los paquetes instalados

RPM puede verificar la integridad de los archivos instalados:

```bash
sudo rpm -Va
```

Esto no verifica únicamente firmas.

Compara atributos como:

- Tamaño.
- Permisos.
- Checksum.
- Usuario.
- Grupo.
- Fecha.
- Enlaces simbólicos.

---

# Interpretar `rpm -Va`

Ejemplo:

```text
S.5....T.  c /etc/ssh/sshd_config
```

| Símbolo | Significado |
|---------|-------------|
| `S` | Tamaño diferente |
| `M` | Permisos o modo diferente |
| `5` | Checksum diferente |
| `D` | Dispositivo diferente |
| `L` | Enlace simbólico diferente |
| `U` | Usuario diferente |
| `G` | Grupo diferente |
| `T` | Fecha diferente |
| `P` | Capacidades diferentes |
| `c` | Archivo de configuración |

---

# Diferencia entre firma e integridad instalada

```text
rpm -K paquete.rpm
```

Verifica el archivo RPM antes de instalar.

```text
rpm -V paquete
```

Verifica los archivos instalados contra la base RPM.

```text
rpm -Va
```

Verifica todos los paquetes instalados.

---

# Cadena de confianza

La seguridad depende de varias capas.

```text
Fuente oficial
    │
    ▼
Clave pública legítima
    │
    ▼
Huella verificada
    │
    ▼
Repositorio seguro
    │
    ▼
Paquete firmado
    │
    ▼
DNF verifica
    │
    ▼
Instalación confiable
```

Si una capa falla, la confianza puede verse comprometida.

---

# Seguridad del transporte

Aunque la firma GPG valida el paquete, también se recomienda utilizar:

```text
HTTPS
```

en lugar de:

```text
HTTP
```

HTTPS protege:

- Confidencialidad.
- Integridad durante el transporte.
- Autenticación del servidor.
- Reducción del riesgo de manipulación en tránsito.

---

# GPG y HTTPS

| GPG | HTTPS |
|-----|-------|
| Verifica el paquete | Protege la conexión |
| Funciona incluso después de descargar | Actúa durante la transferencia |
| Usa claves del publicador | Usa certificados TLS |
| Detecta modificación del paquete | Reduce manipulación en tránsito |

Lo ideal es utilizar ambos.

---

# Repositorio interno seguro

Un repositorio corporativo debería incluir:

- HTTPS.
- Firmas GPG.
- Control de acceso.
- Registro de auditoría.
- Revisión de paquetes.
- Separación entre desarrollo y producción.
- Rotación de claves.
- Respaldo de metadatos.
- Monitoreo de cambios.

---

# Ejemplo de repositorio seguro

```ini
[repo-corporativo]
name=Repositorio corporativo RHEL 9
baseurl=https://repo.empresa.local/rhel9/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-empresa
sslverify=1
```

---

# Parámetro `sslverify`

Activado:

```ini
sslverify=1
```

Desactivado:

```ini
sslverify=0
```

En producción debe mantenerse:

```ini
sslverify=1
```

---

# Riesgo de `sslverify=0`

Desactiva la validación del certificado TLS.

Esto puede facilitar:

- Ataques de intermediario.
- Suplantación del repositorio.
- Redirección a un servidor falso.

---

# Buscar configuraciones TLS inseguras

```bash
grep -R \
-E "^sslverify=0" \
/etc/yum.repos.d/
```

---

# Certificados internos

En una empresa puede existir una autoridad certificadora interna.

En ese caso, en lugar de desactivar TLS, debe instalarse el certificado de confianza.

Copiar el certificado:

```bash
sudo cp empresa-ca.crt \
/etc/pki/ca-trust/source/anchors/
```

Actualizar el almacén:

```bash
sudo update-ca-trust
```

Verificar:

```bash
curl -I \
https://repo.empresa.local/
```

---

# No desactivar seguridad para solucionar certificados

Una mala práctica común es usar:

```ini
sslverify=0
```

La solución correcta es:

- Instalar la CA correcta.
- Corregir el certificado.
- Verificar el nombre del servidor.
- Revisar la fecha del sistema.
- Renovar el certificado si está vencido.

---

# Verificar fecha y hora

Las firmas y certificados pueden fallar si el reloj del sistema es incorrecto.

Consultar:

```bash
timedatectl
```

Verificar sincronización:

```bash
timedatectl show \
-p NTPSynchronized
```

Estado de Chrony:

```bash
systemctl status chronyd
```

Fuentes de tiempo:

```bash
chronyc sources -v
```

---

# Seguridad de la caché

DNF almacena paquetes y metadatos en:

```text
/var/cache/dnf/
```

Consultar espacio:

```bash
du -sh /var/cache/dnf
```

La caché debe conservar permisos adecuados y no ser modificada manualmente por usuarios no autorizados.

---

# Limpiar paquetes sospechosos

```bash
sudo dnf clean packages
```

Limpiar todo:

```bash
sudo dnf clean all
```

Regenerar:

```bash
sudo dnf makecache --refresh
```

---

# Verificar permisos de configuración

```bash
ls -ld /etc/yum.repos.d
```

```bash
ls -l /etc/yum.repos.d/
```

Los archivos deben pertenecer normalmente a:

```text
root
```

y no deben ser modificables por usuarios comunes.

---

# Verificar cambios en repositorios

```bash
rpm -V dnf
```

```bash
rpm -V redhat-release
```

También puede utilizarse auditoría:

```bash
sudo ausearch \
-f /etc/yum.repos.d/
```

Si previamente se configuraron reglas de `auditd`.

---

# Crear una regla de auditoría

Ejemplo:

```bash
sudo auditctl \
-w /etc/yum.repos.d/ \
-p wa \
-k cambios_repositorios
```

Consultar:

```bash
sudo ausearch \
-k cambios_repositorios
```

> Las reglas agregadas con `auditctl` no son persistentes después de reiniciar.

---

# Regla persistente

Crear:

```bash
sudo vi \
/etc/audit/rules.d/repositorios.rules
```

Contenido:

```text
-w /etc/yum.repos.d/ -p wa -k cambios_repositorios
-w /etc/pki/rpm-gpg/ -p wa -k cambios_claves_gpg
```

Cargar reglas:

```bash
sudo augenrules --load
```

Verificar:

```bash
sudo auditctl -l
```

---

# Registrar cambios de paquetes

DNF conserva información en:

```text
/var/log/dnf.log
```

```text
/var/log/dnf.rpm.log
```

Consultar:

```bash
sudo less /var/log/dnf.log
```

Historial:

```bash
dnf history
```

---

# Auditoría de una instalación

Para investigar un paquete:

```bash
dnf history list paquete
```

Consultar la transacción:

```bash
dnf history info ID
```

Verificar el paquete:

```bash
rpm -qi paquete
```

Consultar el repositorio:

```bash
dnf info paquete
```

---

# Respuesta ante un paquete sospechoso

```text
Detectar paquete sospechoso
        │
        ▼
No instalar o detener uso
        │
        ▼
Verificar firma y checksum
        │
        ▼
Identificar repositorio
        │
        ▼
Revisar historial DNF
        │
        ▼
Consultar proveedor
        │
        ▼
Eliminar o reinstalar desde fuente confiable
        │
        ▼
Auditar el sistema
```

---

# Procedimiento básico de respuesta

1. Identificar el paquete.

```bash
rpm -qi paquete
```

2. Consultar archivos.

```bash
rpm -ql paquete
```

3. Verificar integridad.

```bash
rpm -V paquete
```

4. Consultar historial.

```bash
dnf history list paquete
```

5. Revisar repositorio.

```bash
dnf info paquete
```

6. Reinstalar desde fuente confiable si corresponde.

```bash
sudo dnf reinstall paquete
```

7. Revisar servicios relacionados.

```bash
systemctl status servicio
```

8. Consultar logs.

```bash
journalctl -u servicio
```

---

# Paquetes descargados manualmente

Antes de instalar un RPM descargado:

```bash
file paquete.rpm
```

Consultar información:

```bash
rpm -qip paquete.rpm
```

Listar archivos:

```bash
rpm -qlp paquete.rpm
```

Consultar scripts:

```bash
rpm -qp \
--scripts \
paquete.rpm
```

Consultar dependencias:

```bash
rpm -qpR paquete.rpm
```

Verificar firma:

```bash
rpm -Kv paquete.rpm
```

---

# Importancia de revisar scripts RPM

Un paquete RPM puede ejecutar scripts durante:

- Preinstalación.
- Postinstalación.
- Preeliminación.
- Posteliminación.
- Actualización.
- Activación de servicios.

Consultar:

```bash
rpm -qp --scripts paquete.rpm
```

Esto es especialmente importante para paquetes externos.

---

# Extraer un RPM sin instalarlo

```bash
mkdir contenido-rpm
cd contenido-rpm
```

Extraer:

```bash
rpm2cpio ../paquete.rpm | cpio -idmv
```

Después puedes revisar:

```bash
find .
```

Esto permite inspeccionar el contenido sin modificar el sistema.

---

# No instalar paquetes con `--force`

Ejemplo peligroso:

```bash
sudo rpm -ivh \
--force \
paquete.rpm
```

Puede sobrescribir archivos y evitar controles importantes.

---

# No ignorar dependencias

Ejemplo peligroso:

```bash
sudo rpm -ivh \
--nodeps \
paquete.rpm
```

Puede instalar software incompleto o inconsistente.

---

# Riesgos combinados

```bash
sudo rpm -ivh \
--force \
--nodeps \
paquete.rpm
```

Este tipo de instalación puede:

- Romper dependencias.
- Sobrescribir archivos.
- Generar inconsistencias.
- Dificultar futuras actualizaciones.
- Comprometer la estabilidad del sistema.

---

# Recomendación

Para paquetes locales utiliza:

```bash
sudo dnf install ./paquete.rpm
```

DNF resolverá dependencias y aplicará validaciones configuradas.

---

# Herramientas relacionadas

| Herramienta | Función |
|-------------|---------|
| `rpm -K` | Verificar firma de un RPM |
| `rpmkeys` | Administrar claves y firmas |
| `rpm --import` | Importar clave pública |
| `rpm -qa 'gpg-pubkey*'` | Listar claves |
| `gpg --show-keys` | Examinar una clave |
| `sha256sum` | Calcular checksum |
| `dnf` | Instalar y verificar paquetes |
| `curl` | Descargar archivos |
| `rpm2cpio` | Extraer un RPM |
| `cpio` | Recuperar contenido del paquete |
| `auditctl` | Configurar auditoría |
| `ausearch` | Consultar eventos de auditoría |

---

# Buenas prácticas RHCSA

✔ Utilizar repositorios oficiales o corporativos confiables.

✔ Mantener `gpgcheck=1`.

✔ Verificar la huella digital antes de importar una clave.

✔ Utilizar HTTPS para repositorios remotos.

✔ Mantener `sslverify=1`.

✔ No utilizar `--nogpgcheck` en producción.

✔ Revisar scripts de paquetes externos.

✔ Verificar firmas antes de instalar RPM manuales.

✔ No utilizar `--force` ni `--nodeps` sin una justificación técnica excepcional.

✔ Documentar la importación y eliminación de claves.

✔ Revisar periódicamente los archivos `.repo`.

✔ Auditar cambios en repositorios y claves.

✔ Mantener sincronizada la fecha y hora del sistema.

✔ Eliminar repositorios que ya no sean necesarios.

---

# Errores comunes

## Importar una clave sin verificar la huella

Un atacante puede distribuir una clave falsa junto con un paquete malicioso.

---

## Aceptar automáticamente una clave

No debe confirmarse una clave solo porque DNF la muestra.

Debe compararse su huella con una fuente oficial.

---

## Usar `--nogpgcheck` para resolver cualquier error

Esto elimina una protección importante y oculta el problema real.

---

## Desactivar `sslverify`

La solución correcta es instalar o corregir el certificado de confianza.

---

## Confundir checksum con firma digital

Un checksum comprueba integridad, pero no identifica por sí solo al publicador.

---

## Instalar un RPM externo sin revisar scripts

El paquete puede ejecutar acciones con privilegios elevados.

---

## Pensar que una firma válida garantiza software seguro

La firma confirma el origen criptográfico, pero el software aún puede contener errores o vulnerabilidades.

---

## Eliminar claves oficiales

Puede impedir futuras actualizaciones o instalaciones.

---

# Comandos importantes RHCSA

| Comando | Descripción |
|---------|-------------|
| `rpm -K paquete.rpm` | Verificar firma |
| `rpm -Kv paquete.rpm` | Verificación detallada |
| `rpm --checksig paquete.rpm` | Comprobar firma |
| `rpmkeys --checksig paquete.rpm` | Verificar con rpmkeys |
| `rpm --import clave` | Importar una clave |
| `rpm -qa 'gpg-pubkey*'` | Listar claves importadas |
| `rpm -qi gpg-pubkey-ID` | Información de una clave |
| `rpm -e gpg-pubkey-ID` | Eliminar una clave |
| `gpg --show-keys clave` | Mostrar información |
| `gpg --with-fingerprint clave` | Mostrar huella |
| `sha256sum archivo` | Calcular checksum |
| `rpm -qp --scripts paquete.rpm` | Consultar scripts |
| `rpm -qip paquete.rpm` | Información sin instalar |
| `rpm -qlp paquete.rpm` | Archivos del paquete |
| `rpm -qpR paquete.rpm` | Dependencias |
| `rpm -V paquete` | Verificar paquete instalado |
| `rpm -Va` | Verificar todos los paquetes |
| `dnf clean all` | Limpiar caché |
| `dnf makecache --refresh` | Regenerar metadatos |
| `dnf history` | Consultar operaciones |

---

# Resumen rápido

```text
Seguridad de paquetes
    │
    ├── Integridad
    │     ├── checksum
    │     └── rpm -V
    │
    ├── Autenticidad
    │     ├── firma GPG
    │     └── clave pública
    │
    ├── Transporte
    │     ├── HTTPS
    │     └── sslverify
    │
    ├── Repositorios
    │     ├── gpgcheck
    │     ├── repo_gpgcheck
    │     └── gpgkey
    │
    └── Auditoría
          ├── dnf history
          ├── logs
          └── auditd
```

---

# Resumen

En esta lección aprendiste a:

- Comprender la función de las firmas GPG.
- Diferenciar integridad, autenticidad y confianza.
- Verificar paquetes RPM.
- Importar y eliminar claves públicas.
- Consultar claves instaladas.
- Configurar `gpgcheck`, `repo_gpgcheck` y `gpgkey`.
- Utilizar HTTPS y validar certificados.
- Diagnosticar errores de firma.
- Reconocer los riesgos de `--nogpgcheck`.
- Inspeccionar paquetes externos antes de instalarlos.
- Auditar cambios relacionados con repositorios y claves.

---

# Laboratorio práctico RHCSA

## Escenario 1: Consultar claves instaladas

Lista las claves:

```bash
rpm -qa 'gpg-pubkey*'
```

Selecciona una:

```bash
rpm -qi gpg-pubkey-ID
```

---

## Escenario 2: Examinar las claves del sistema

```bash
ls -l /etc/pki/rpm-gpg/
```

Muestra la huella de una clave:

```bash
gpg --show-keys \
--with-fingerprint \
/etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release
```

> El nombre exacto puede variar según la distribución.

---

## Escenario 3: Descargar un paquete

Instala los complementos:

```bash
sudo dnf install dnf-plugins-core
```

Crea un directorio:

```bash
mkdir -p ~/rpm-seguridad
cd ~/rpm-seguridad
```

Descarga un paquete:

```bash
dnf download tree
```

---

## Escenario 4: Verificar el RPM

```bash
rpm -Kv tree*.rpm
```

Consulta información:

```bash
rpm -qip tree*.rpm
```

Lista archivos:

```bash
rpm -qlp tree*.rpm
```

Consulta scripts:

```bash
rpm -qp \
--scripts \
tree*.rpm
```

---

## Escenario 5: Calcular checksum

```bash
sha256sum tree*.rpm
```

Guarda el resultado:

```bash
sha256sum tree*.rpm \
> tree.sha256
```

Verifica:

```bash
sha256sum -c tree.sha256
```

---

## Escenario 6: Auditar repositorios

Busca configuraciones sin verificación:

```bash
grep -R \
-E "^gpgcheck=0|^sslverify=0" \
/etc/yum.repos.d/
```

Muestra la configuración relevante:

```bash
grep -R \
-E "^\[|^name=|^baseurl=|^enabled=|^gpgcheck=|^repo_gpgcheck=|^gpgkey=|^sslverify=" \
/etc/yum.repos.d/
```

---

## Escenario 7: Crear una clave de laboratorio

Copia una clave conocida:

```bash
cp \
/etc/pki/rpm-gpg/RPM-GPG-KEY-redhat-release \
~/rpm-seguridad/clave-laboratorio
```

Muestra su huella:

```bash
gpg --show-keys \
--with-fingerprint \
~/rpm-seguridad/clave-laboratorio
```

No elimines ni modifiques las claves oficiales del sistema.

---

## Escenario 8: Verificar un paquete instalado

```bash
rpm -V tree
```

Si no aparece salida, no se detectaron diferencias en los atributos comprobados.

---

## Escenario 9: Consultar historial

```bash
dnf history list tree
```

Consulta la transacción:

```bash
dnf history info ID
```

---

## Escenario 10: Inspeccionar un RPM sin instalarlo

```bash
mkdir -p ~/rpm-seguridad/contenido
cd ~/rpm-seguridad/contenido
```

Extrae el paquete:

```bash
rpm2cpio ../tree*.rpm | cpio -idmv
```

Consulta el contenido:

```bash
find .
```

---

# Preguntas de repaso

1. ¿Qué diferencia existe entre una clave pública y una privada?
2. ¿Qué garantiza una firma GPG?
3. ¿Qué no garantiza una firma válida?
4. ¿Qué significa `NOKEY`?
5. ¿Qué significa `BAD signature`?
6. ¿Qué comando verifica un paquete RPM?
7. ¿Dónde suelen almacenarse las claves GPG en RHEL?
8. ¿Cómo se importa una clave pública?
9. ¿Qué función cumple `gpgcheck=1`?
10. ¿Cuál es la diferencia entre `gpgcheck` y `repo_gpgcheck`?
11. ¿Por qué debe evitarse `--nogpgcheck`?
12. ¿Qué diferencia existe entre un checksum y una firma?
13. ¿Por qué no debe utilizarse `sslverify=0`?
14. ¿Cómo se revisan los scripts de un RPM sin instalarlo?
15. ¿Qué comando verifica los archivos de un paquete instalado?

---

# Desafío final

Realiza las siguientes tareas:

1. Lista las claves GPG instaladas.
2. Consulta la información de una clave.
3. Muestra su huella digital.
4. Descarga un paquete RPM sin instalarlo.
5. Verifica su firma.
6. Calcula su checksum SHA-256.
7. Consulta su información.
8. Lista sus archivos.
9. Revisa sus scripts.
10. Extrae su contenido.
11. Audita los archivos `.repo`.
12. Identifica cualquier repositorio con `gpgcheck=0`.
13. Revisa el historial de instalación del paquete.
14. Documenta todas las comprobaciones realizadas.

> **Objetivo general:** comprender y aplicar los mecanismos de seguridad utilizados para verificar paquetes y repositorios en Red Hat Enterprise Linux. El dominio de firmas GPG, claves públicas, checksums, HTTPS y auditoría permite instalar software de forma confiable, detectar paquetes alterados y mantener una cadena de suministro más segura en entornos RHCSA y de producción.