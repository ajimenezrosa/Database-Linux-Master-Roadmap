# 53. Laboratorio de Administración de Usuarios y Grupos en Linux — Fase 1

> **Módulo 5 — Administración de Usuarios, Grupos y Permisos**
>
> **Archivo:** `53-laboratorio-usuarios.md`
>
> **Nivel:** RHCSA
>
> **Objetivo general:** Aplicar de manera práctica la creación, modificación, bloqueo, eliminación y auditoría de usuarios y grupos en sistemas Red Hat Enterprise Linux, Rocky Linux, AlmaLinux o Fedora.

---

# 1. Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Crear usuarios locales desde la línea de comandos.
- Crear y administrar grupos primarios y secundarios.
- Configurar UID y GID personalizados.
- Asignar shells de inicio de sesión.
- Crear directorios personales.
- Establecer y modificar contraseñas.
- Consultar información de cuentas.
- Modificar propiedades de usuarios existentes.
- Bloquear y desbloquear cuentas.
- Configurar fechas de expiración.
- Administrar pertenencia a grupos.
- Interpretar los archivos `/etc/passwd`, `/etc/shadow`, `/etc/group` y `/etc/gshadow`.
- Utilizar `getent`, `id`, `groups`, `chage`, `passwd`, `useradd`, `usermod` y `groupmod`.
- Resolver errores comunes relacionados con usuarios y grupos.
- Aplicar buenas prácticas de administración.
- Completar ejercicios similares a los evaluados en RHCSA.

---

# 2. Introducción al laboratorio

La administración de usuarios es una de las responsabilidades más importantes de un administrador Linux.

Cada usuario que accede a un servidor debe poseer:

- Una identidad única.
- Un UID.
- Un grupo primario.
- Cero o más grupos secundarios.
- Un directorio personal.
- Un shell.
- Una contraseña o método de autenticación.
- Permisos adecuados.
- Una política de expiración.

En un entorno empresarial, administrar usuarios incorrectamente puede provocar:

- Accesos no autorizados.
- Pérdida de información.
- Modificación accidental de archivos.
- Elevación indebida de privilegios.
- Incumplimiento de políticas de seguridad.
- Problemas de auditoría.
- Dificultad para identificar responsables.

Este laboratorio simula la administración de cuentas en una empresa ficticia.

---

# 3. Escenario empresarial

La empresa **TechData Solutions** ha contratado nuevos empleados para sus departamentos de:

- Administración Linux.
- Bases de Datos.
- Desarrollo.
- Seguridad.
- Soporte.
- Auditoría.

El administrador debe crear usuarios y grupos siguiendo una política corporativa.

---

# 4. Usuarios del laboratorio

| Usuario | Nombre completo | Departamento | Shell | Grupo primario |
|---|---|---|---|---|
| `ajimenez` | Alejandro Jiménez | Linux | `/bin/bash` | `linuxadmins` |
| `mgarcia` | María García | Base de Datos | `/bin/bash` | `dba` |
| `jperez` | Juan Pérez | Desarrollo | `/bin/bash` | `developers` |
| `lrodriguez` | Laura Rodríguez | Seguridad | `/bin/bash` | `security` |
| `csoporte` | Cuenta de Soporte | Soporte | `/bin/bash` | `support` |
| `auditor` | Cuenta de Auditoría | Auditoría | `/sbin/nologin` | `audit` |

---

# 5. Grupos del laboratorio

| Grupo | Propósito | GID solicitado |
|---|---|---:|
| `linuxadmins` | Administración de Linux | 3000 |
| `dba` | Administración de bases de datos | 3001 |
| `developers` | Desarrollo de aplicaciones | 3002 |
| `security` | Administración de seguridad | 3003 |
| `support` | Personal de soporte | 3004 |
| `audit` | Auditores del sistema | 3005 |
| `project_alpha` | Proyecto compartido | 3100 |

---

# 6. Arquitectura de identidades

```text
                           Sistema Linux

                                 │
                                 ▼

                       Usuarios y Grupos

              ┌──────────────────┴──────────────────┐
              │                                     │
              ▼                                     ▼

          Usuarios                               Grupos

              │                                     │
      ┌───────┼────────┐                 ┌──────────┼──────────┐
      │       │        │                 │          │          │
      ▼       ▼        ▼                 ▼          ▼          ▼

     UID    Shell    Home             Primario  Secundario   Compartido
```

---

# 7. Preparación del laboratorio

## 7.1 Requisitos

Necesitarás:

- RHEL.
- Rocky Linux.
- AlmaLinux.
- Fedora.
- Una máquina virtual o servidor de laboratorio.
- Acceso mediante `root` o `sudo`.
- Consola o sesión SSH.

---

## 7.2 Confirmar la distribución

```bash
cat /etc/os-release
```

También:

```bash
hostnamectl
```

---

## 7.3 Confirmar privilegios

```bash
whoami
```

Resultado esperado si utilizas `root`:

```text
root
```

Si utilizas una cuenta administrativa:

```bash
sudo -v
```

---

## 7.4 Crear un directorio de evidencias

```bash
sudo mkdir -p /root/laboratorio-usuarios/evidencias
```

Verificar:

```bash
sudo ls -ld /root/laboratorio-usuarios/evidencias
```

---

# 8. Archivos principales de usuarios y grupos

Linux almacena la información de las cuentas locales en varios archivos.

| Archivo | Contenido |
|---|---|
| `/etc/passwd` | Información general de usuarios |
| `/etc/shadow` | Contraseñas cifradas y expiración |
| `/etc/group` | Información general de grupos |
| `/etc/gshadow` | Información protegida de grupos |

---

# 9. Archivo `/etc/passwd`

Consultar:

```bash
cat /etc/passwd
```

Una entrada típica:

```text
ajimenez:x:1001:1001:Alejandro Jimenez:/home/ajimenez:/bin/bash
```

---

## 9.1 Campos de `/etc/passwd`

```text
usuario:x:UID:GID:comentario:home:shell
```

| Posición | Campo | Ejemplo |
|---:|---|---|
| 1 | Nombre de usuario | `ajimenez` |
| 2 | Contraseña representada | `x` |
| 3 | UID | `1001` |
| 4 | GID primario | `1001` |
| 5 | Comentario o GECOS | `Alejandro Jimenez` |
| 6 | Directorio personal | `/home/ajimenez` |
| 7 | Shell | `/bin/bash` |

---

## 9.2 No editar `/etc/passwd` directamente

Aunque técnicamente puede editarse, no es recomendable hacerlo con un editor de texto común.

Utiliza:

```bash
vipw
```

Para grupos:

```bash
vigr
```

Estas herramientas:

- Bloquean temporalmente el archivo.
- Reducen conflictos.
- Ayudan a evitar corrupción.
- Aplican controles básicos de seguridad.

La práctica recomendada sigue siendo utilizar:

- `useradd`
- `usermod`
- `userdel`
- `groupadd`
- `groupmod`
- `groupdel`

---

# 10. Archivo `/etc/shadow`

Consultar como `root`:

```bash
sudo cat /etc/shadow
```

Ejemplo:

```text
ajimenez:$6$HASH:20650:0:99999:7:::
```

---

## 10.1 Campos principales

```text
usuario:hash:último_cambio:mínimo:máximo:aviso:inactividad:expiración:reservado
```

| Campo | Descripción |
|---|---|
| Usuario | Nombre de la cuenta |
| Hash | Contraseña cifrada |
| Último cambio | Días desde el 1 de enero de 1970 |
| Mínimo | Días antes de poder cambiar contraseña |
| Máximo | Días de validez |
| Aviso | Días de advertencia |
| Inactividad | Días posteriores a la expiración |
| Expiración | Fecha de expiración de cuenta |
| Reservado | Uso futuro |

---

# 11. Archivo `/etc/group`

Consultar:

```bash
cat /etc/group
```

Entrada típica:

```text
project_alpha:x:3100:ajimenez,jperez
```

Formato:

```text
grupo:x:GID:miembros
```

---

# 12. Consultar bases de identidad con getent

`getent` es preferible a leer directamente los archivos cuando el sistema utiliza:

- LDAP.
- Active Directory.
- SSSD.
- NIS.
- Bases de datos centralizadas.

Consultar usuario:

```bash
getent passwd root
```

Consultar grupo:

```bash
getent group wheel
```

Mostrar todos los usuarios visibles:

```bash
getent passwd
```

Mostrar todos los grupos visibles:

```bash
getent group
```

---

# 13. Diferencia entre usuario local y usuario centralizado

```text
                    Solicitud de identidad

                             │
                             ▼

                            NSS

              ┌──────────────┼──────────────┐
              │              │              │
              ▼              ▼              ▼

        /etc/passwd         SSSD           LDAP
```

Un usuario puede existir aunque no aparezca directamente en `/etc/passwd`.

Por eso se recomienda:

```bash
getent passwd nombre_usuario
```

---

# 14. Crear los grupos del laboratorio

## 14.1 Crear `linuxadmins`

```bash
sudo groupadd -g 3000 linuxadmins
```

---

## 14.2 Crear `dba`

```bash
sudo groupadd -g 3001 dba
```

---

## 14.3 Crear `developers`

```bash
sudo groupadd -g 3002 developers
```

---

## 14.4 Crear `security`

```bash
sudo groupadd -g 3003 security
```

---

## 14.5 Crear `support`

```bash
sudo groupadd -g 3004 support
```

---

## 14.6 Crear `audit`

```bash
sudo groupadd -g 3005 audit
```

---

## 14.7 Crear grupo compartido del proyecto

```bash
sudo groupadd -g 3100 project_alpha
```

---

# 15. Verificar los grupos

```bash
getent group linuxadmins
getent group dba
getent group developers
getent group security
getent group support
getent group audit
getent group project_alpha
```

También:

```bash
grep -E '^(linuxadmins|dba|developers|security|support|audit|project_alpha):' /etc/group
```

---

# 16. Comprobar que los GID estén disponibles

Antes de crear un grupo con GID específico:

```bash
getent group 3000
```

Si no produce salida, normalmente el GID está disponible.

Puede utilizarse:

```bash
getent group 3000 || echo "GID 3000 disponible"
```

---

# 17. Crear el usuario `ajimenez`

Requerimientos:

- Usuario: `ajimenez`.
- Nombre: Alejandro Jiménez.
- UID: `4000`.
- Grupo primario: `linuxadmins`.
- Home: `/home/ajimenez`.
- Shell: `/bin/bash`.

Comando:

```bash
sudo useradd \
  -u 4000 \
  -g linuxadmins \
  -c "Alejandro Jimenez" \
  -m \
  -s /bin/bash \
  ajimenez
```

---

# 18. Interpretar el comando useradd

| Opción | Significado |
|---|---|
| `-u 4000` | Asigna UID 4000 |
| `-g linuxadmins` | Define grupo primario |
| `-c` | Agrega comentario |
| `-m` | Crea directorio personal |
| `-s /bin/bash` | Asigna shell |
| `ajimenez` | Nombre de la cuenta |

---

# 19. Crear el usuario `mgarcia`

```bash
sudo useradd \
  -u 4001 \
  -g dba \
  -c "Maria Garcia" \
  -m \
  -s /bin/bash \
  mgarcia
```

---

# 20. Crear el usuario `jperez`

```bash
sudo useradd \
  -u 4002 \
  -g developers \
  -c "Juan Perez" \
  -m \
  -s /bin/bash \
  jperez
```

---

# 21. Crear el usuario `lrodriguez`

```bash
sudo useradd \
  -u 4003 \
  -g security \
  -c "Laura Rodriguez" \
  -m \
  -s /bin/bash \
  lrodriguez
```

---

# 22. Crear la cuenta `csoporte`

```bash
sudo useradd \
  -u 4004 \
  -g support \
  -c "Cuenta de Soporte" \
  -m \
  -s /bin/bash \
  csoporte
```

---

# 23. Crear la cuenta de auditoría

La cuenta `auditor` no debe iniciar sesión interactiva.

```bash
sudo useradd \
  -u 4005 \
  -g audit \
  -c "Cuenta de Auditoria" \
  -m \
  -s /sbin/nologin \
  auditor
```

---

# 24. Verificar los usuarios

```bash
getent passwd ajimenez
getent passwd mgarcia
getent passwd jperez
getent passwd lrodriguez
getent passwd csoporte
getent passwd auditor
```

---

# 25. Consultar identidad con id

```bash
id ajimenez
```

Resultado conceptual:

```text
uid=4000(ajimenez) gid=3000(linuxadmins) groups=3000(linuxadmins)
```

Consultar los demás:

```bash
id mgarcia
id jperez
id lrodriguez
id csoporte
id auditor
```

---

# 26. Consultar grupos con groups

```bash
groups ajimenez
```

Resultado inicial:

```text
ajimenez : linuxadmins
```

---

# 27. Verificar directorios personales

```bash
ls -ld /home/ajimenez
ls -ld /home/mgarcia
ls -ld /home/jperez
ls -ld /home/lrodriguez
ls -ld /home/csoporte
ls -ld /home/auditor
```

---

# 28. Establecer contraseñas

Asignar contraseña interactivamente:

```bash
sudo passwd ajimenez
```

Continuar:

```bash
sudo passwd mgarcia
sudo passwd jperez
sudo passwd lrodriguez
sudo passwd csoporte
```

La cuenta `auditor` puede mantenerse bloqueada si no requiere autenticación.

---

# 29. No escribir contraseñas en scripts

Evita:

```bash
echo "Clave123" | passwd --stdin usuario
```

Especialmente porque:

- Puede aparecer en el historial.
- Puede verse en procesos.
- Puede guardarse en scripts.
- Puede filtrarse por logs.
- Puede quedar expuesta en repositorios.

Para automatización empresarial utiliza:

- Ansible Vault.
- Hashes protegidos.
- Sistemas de secretos.
- Administración centralizada de identidades.

---

# 30. Consultar estado de una contraseña

```bash
sudo passwd -S ajimenez
```

Salida conceptual:

```text
ajimenez PS 2026-07-28 0 99999 7 -1
```

Estados frecuentes:

| Código | Significado |
|---|---|
| `PS` | Tiene contraseña |
| `LK` | Contraseña bloqueada |
| `NP` | No tiene contraseña |

---

# 31. Forzar cambio de contraseña en el próximo inicio

```bash
sudo chage -d 0 ajimenez
```

Verificar:

```bash
sudo chage -l ajimenez
```

La próxima vez que el usuario inicie sesión, deberá cambiar su contraseña.

---

# 32. Configurar política de expiración

Para `ajimenez`:

- Duración mínima: 1 día.
- Duración máxima: 90 días.
- Advertencia: 14 días.
- Inactividad: 7 días.

```bash
sudo chage \
  -m 1 \
  -M 90 \
  -W 14 \
  -I 7 \
  ajimenez
```

---

# 33. Interpretar chage

| Opción | Función |
|---|---|
| `-m` | Días mínimos entre cambios |
| `-M` | Días máximos de validez |
| `-W` | Días de advertencia |
| `-I` | Días de inactividad |
| `-E` | Fecha de expiración |
| `-d` | Fecha del último cambio |
| `-l` | Lista configuración |

---

# 34. Consultar política completa

```bash
sudo chage -l ajimenez
```

Salida conceptual:

```text
Last password change                                    : password must be changed
Password expires                                        : password must be changed
Password inactive                                       : password must be changed
Account expires                                         : never
Minimum number of days between password change          : 1
Maximum number of days between password change          : 90
Number of days of warning before password expires       : 14
```

---

# 35. Configurar fecha de expiración de cuenta

Supongamos que `csoporte` es una cuenta temporal que debe vencer el 31 de diciembre de 2026.

```bash
sudo chage -E 2026-12-31 csoporte
```

Verificar:

```bash
sudo chage -l csoporte
```

---

# 36. Diferencia entre expiración de contraseña y cuenta

| Concepto | Resultado |
|---|---|
| Expira la contraseña | El usuario debe cambiarla |
| Expira la cuenta | El usuario ya no puede iniciar sesión |
| Se bloquea la contraseña | No puede autenticarse con esa contraseña |
| Shell `nologin` | No puede obtener sesión interactiva |

---

# 37. Agregar usuarios a grupos secundarios

Agregar `ajimenez` al proyecto:

```bash
sudo usermod -aG project_alpha ajimenez
```

Agregar `jperez`:

```bash
sudo usermod -aG project_alpha jperez
```

Agregar `mgarcia`:

```bash
sudo usermod -aG project_alpha mgarcia
```

---

# 38. Importancia de `-aG`

La opción correcta para agregar grupos sin eliminar los existentes es:

```bash
usermod -aG grupo usuario
```

| Opción | Significado |
|---|---|
| `-G` | Establece grupos secundarios |
| `-a` | Agrega sin sustituir |

---

# 39. Error peligroso con usermod

Supongamos que `ajimenez` pertenece a:

```text
wheel
linuxadmins
project_alpha
```

Este comando:

```bash
sudo usermod -G project_alpha ajimenez
```

puede reemplazar los grupos secundarios anteriores.

La forma segura es:

```bash
sudo usermod -aG project_alpha ajimenez
```

---

# 40. Verificar pertenencia a grupos

```bash
id ajimenez
```

También:

```bash
groups ajimenez
```

Consultar directamente el grupo:

```bash
getent group project_alpha
```

Resultado conceptual:

```text
project_alpha:x:3100:ajimenez,jperez,mgarcia
```

---

# 41. Agregar administradores a wheel

En sistemas Red Hat, el grupo `wheel` suele controlar acceso mediante `sudo`.

Agregar `ajimenez`:

```bash
sudo usermod -aG wheel ajimenez
```

Agregar `lrodriguez`:

```bash
sudo usermod -aG wheel lrodriguez
```

Verificar:

```bash
id ajimenez
id lrodriguez
```

---

# 42. Comprobar privilegios sudo

```bash
sudo -l -U ajimenez
```

También puede probarse iniciando sesión:

```bash
su - ajimenez
```

Luego:

```bash
sudo whoami
```

Resultado esperado:

```text
root
```

---

# 43. Sesiones y actualización de grupos

Cuando se agrega un usuario a un nuevo grupo, una sesión ya abierta puede no reconocer inmediatamente la nueva membresía.

Soluciones:

- Cerrar sesión.
- Volver a iniciar sesión.
- Utilizar `newgrp`.
- Crear una nueva sesión SSH.

Ejemplo:

```bash
newgrp project_alpha
```

---

# 44. Cambiar el comentario de un usuario

```bash
sudo usermod \
  -c "Alejandro Jimenez - Linux Administrator" \
  ajimenez
```

Verificar:

```bash
getent passwd ajimenez
```

---

# 45. Cambiar el shell

Cambiar el shell de `csoporte` a `nologin`:

```bash
sudo usermod -s /sbin/nologin csoporte
```

Restaurar:

```bash
sudo usermod -s /bin/bash csoporte
```

Verificar:

```bash
getent passwd csoporte
```

---

# 46. Ver shells permitidos

```bash
cat /etc/shells
```

Ejemplo:

```text
/bin/sh
/bin/bash
/usr/bin/sh
/usr/bin/bash
```

`/sbin/nologin` puede estar disponible aunque no aparezca en todos los sistemas dentro de `/etc/shells`.

---

# 47. Cambiar directorio personal

Crear nuevo directorio:

```bash
sudo usermod \
  -d /srv/home/csoporte \
  -m \
  csoporte
```

| Opción | Función |
|---|---|
| `-d` | Define nuevo home |
| `-m` | Mueve contenido existente |

Verificar:

```bash
getent passwd csoporte
sudo ls -ld /srv/home/csoporte
```

---

# 48. No mover un home sin planificación

Antes de mover un directorio personal, verifica:

- Espacio disponible.
- Procesos abiertos.
- Sesiones activas.
- Montajes.
- Permisos.
- Contextos SELinux.
- Aplicaciones que usan rutas absolutas.
- Copias de respaldo.

---

# 49. Cambiar nombre de usuario

Cambiar `csoporte` por `soporte01`:

```bash
sudo usermod -l soporte01 csoporte
```

Esto modifica el nombre de inicio de sesión, pero no necesariamente el home.

Cambiar también el home:

```bash
sudo usermod \
  -d /home/soporte01 \
  -m \
  soporte01
```

Verificar:

```bash
getent passwd soporte01
```

---

# 50. Riesgos al renombrar usuarios

Renombrar una cuenta puede afectar:

- Cron.
- Servicios.
- Scripts.
- Permisos ACL.
- Configuraciones de aplicaciones.
- Claves SSH.
- Archivos externos.
- Sistemas centralizados.
- Procesos activos.

Debe realizarse durante una ventana controlada.

---

# 51. Bloquear una cuenta

Bloquear contraseña:

```bash
sudo passwd -l jperez
```

También:

```bash
sudo usermod -L jperez
```

Verificar:

```bash
sudo passwd -S jperez
```

---

# 52. Desbloquear una cuenta

```bash
sudo passwd -u jperez
```

También:

```bash
sudo usermod -U jperez
```

Verificar:

```bash
sudo passwd -S jperez
```

---

# 53. Bloquear contraseña no siempre bloquea todo acceso

Un usuario con contraseña bloqueada todavía podría acceder mediante:

- Clave SSH.
- Certificado.
- Kerberos.
- SSO.
- Otro mecanismo PAM.
- Sesión ya abierta.

Para deshabilitar completamente una cuenta puede combinarse:

```bash
sudo usermod -L jperez
sudo usermod -s /sbin/nologin jperez
sudo chage -E 0 jperez
```

Estas acciones deben aplicarse cuidadosamente.

---

# 54. Consultar sesiones activas antes de bloquear

```bash
who
```

También:

```bash
w
```

Consultar procesos del usuario:

```bash
ps -u jperez
```

Consultar sesiones con `loginctl`:

```bash
loginctl list-sessions
```

---

# 55. Finalizar procesos de una cuenta

Para finalizar procesos de un usuario:

```bash
sudo pkill -u jperez
```

Esta acción puede:

- Interrumpir trabajos.
- Perder datos no guardados.
- Detener procesos importantes.
- Romper tareas automatizadas.

Debe utilizarse sólo después de validar el impacto.

---

# 56. Crear una cuenta de servicio

Una cuenta de servicio normalmente:

- Tiene UID de sistema.
- No necesita contraseña interactiva.
- No utiliza `/bin/bash`.
- Ejecuta un servicio específico.
- Tiene permisos mínimos.
- Puede no tener home.

Crear:

```bash
sudo useradd \
  --system \
  --shell /sbin/nologin \
  --no-create-home \
  --comment "Agente de monitoreo" \
  monitoragent
```

Verificar:

```bash
getent passwd monitoragent
```

---

# 57. Diferencia entre usuario normal y de sistema

| Característica | Usuario normal | Usuario de sistema |
|---|---|---|
| Uso | Persona | Servicio |
| UID | Rango normal | Rango reservado |
| Shell | Frecuentemente Bash | `nologin` |
| Home | Normalmente sí | Opcional |
| Contraseña | Puede tener | Generalmente bloqueada |
| Inicio interactivo | Permitido | No recomendado |

---

# 58. Configuración predeterminada de useradd

Consultar:

```bash
useradd -D
```

Salida conceptual:

```text
GROUP=100
HOME=/home
INACTIVE=-1
EXPIRE=
SHELL=/bin/bash
SKEL=/etc/skel
CREATE_MAIL_SPOOL=yes
```

---

# 59. Directorio `/etc/skel`

Los archivos de `/etc/skel` se copian al home de nuevos usuarios.

Consultar:

```bash
ls -la /etc/skel
```

Contenido frecuente:

```text
.bash_logout
.bash_profile
.bashrc
```

---

# 60. Personalizar `/etc/skel`

Crear un archivo corporativo:

```bash
echo "Bienvenido a TechData Solutions" | \
sudo tee /etc/skel/README_EMPRESA.txt
```

Crear usuario de prueba:

```bash
sudo useradd -m usuario_prueba
```

Verificar:

```bash
sudo ls -la /home/usuario_prueba
```

El archivo debe aparecer en el nuevo home.

---

# 61. Los cambios en `/etc/skel` no son retroactivos

Modificar `/etc/skel` afecta únicamente a usuarios creados posteriormente.

No modifica automáticamente:

```text
/home/usuarios_existentes
```

---

# 62. Cambiar el grupo primario

Cambiar grupo primario de `jperez`:

```bash
sudo usermod -g project_alpha jperez
```

Verificar:

```bash
id jperez
```

Restaurar:

```bash
sudo usermod -g developers jperez
```

---

# 63. Cambiar el GID de un grupo

Supongamos que `project_alpha` debe cambiar de GID 3100 a 3200.

```bash
sudo groupmod -g 3200 project_alpha
```

Verificar:

```bash
getent group project_alpha
```

---

# 64. Archivos existentes después de cambiar GID

Cambiar el GID del grupo no siempre actualiza automáticamente todos los archivos que poseían el GID antiguo.

Buscar archivos con el GID anterior:

```bash
sudo find / -xdev -group 3100 -ls 2>/dev/null
```

Corregir:

```bash
sudo find / -xdev -group 3100 -exec chgrp project_alpha {} +
```

Debe revisarse cuidadosamente antes de ejecutarlo en Producción.

---

# 65. Cambiar el UID de un usuario

Cambiar UID de una cuenta:

```bash
sudo usermod -u 4100 ajimenez
```

Verificar:

```bash
id ajimenez
```

---

# 66. Archivos huérfanos después de cambiar UID

Buscar archivos con UID anterior:

```bash
sudo find / -xdev -uid 4000 -ls 2>/dev/null
```

Reasignar:

```bash
sudo find / -xdev -uid 4000 -exec chown ajimenez {} +
```

Debe realizarse con extremo cuidado.

---

# 67. Eliminar un usuario sin borrar su home

```bash
sudo userdel usuario_prueba
```

El directorio puede permanecer:

```bash
ls -ld /home/usuario_prueba
```

---

# 68. Eliminar usuario y directorio personal

```bash
sudo userdel -r usuario_prueba
```

La opción `-r` intenta eliminar:

- Home.
- Mail spool.
- Archivos asociados dentro de rutas conocidas.

No elimina automáticamente todos los archivos que el usuario posea en otras partes del sistema.

---

# 69. Buscar archivos antes de eliminar una cuenta

```bash
sudo find / -xdev -user usuario_prueba -ls 2>/dev/null
```

En sistemas con múltiples sistemas de archivos puede ser necesario revisar:

```text
/home
/var
/opt
/srv
/data
```

---

# 70. Eliminar un grupo

Antes de eliminar:

```bash
getent group project_alpha
```

Verificar usuarios:

```bash
getent group project_alpha
```

Eliminar:

```bash
sudo groupdel project_alpha
```

No puede eliminarse un grupo que sea grupo primario de una cuenta activa sin modificar primero esa relación.

---

# 71. Flujo seguro para eliminar una cuenta

```text
Solicitud aprobada

↓

Identificar usuario

↓

Revisar sesiones

↓

Revisar procesos

↓

Revisar archivos

↓

Respaldar información

↓

Bloquear cuenta

↓

Esperar período de retención

↓

Eliminar cuenta

↓

Transferir o eliminar archivos

↓

Documentar
```

---

# 72. Auditoría de usuarios humanos

Los usuarios humanos normalmente tienen:

- UID dentro del rango regular.
- Home en `/home`.
- Shell interactivo.
- Contraseña o llave SSH.
- Nombre o comentario identificable.

Consulta:

```bash
getent passwd | awk -F: '$3 >= 1000 {print $1, $3, $6, $7}'
```

---

# 73. Auditoría de cuentas con shell interactivo

```bash
getent passwd | \
awk -F: '$7 ~ /(bash|sh|zsh|ksh)$/ {print $1, $3, $7}'
```

---

# 74. Detectar cuentas sin contraseña

Como `root`:

```bash
sudo awk -F: '($2 == "") {print $1}' /etc/shadow
```

Una cuenta sin contraseña debe investigarse inmediatamente.

---

# 75. Detectar cuentas bloqueadas

```bash
sudo awk -F: '($2 ~ /^!|^\*/) {print $1}' /etc/shadow
```

El resultado puede incluir numerosas cuentas de sistema correctamente bloqueadas.

No todas representan un problema.

---

# 76. Detectar UID duplicados

```bash
getent passwd | \
awk -F: '{print $3, $1}' | \
sort -n | \
uniq -D -w 5
```

Alternativa más clara:

```bash
getent passwd | \
awk -F: '{count[$3]++; users[$3]=users[$3]" "$1} END {
  for (uid in count)
    if (count[uid] > 1)
      print "UID duplicado:", uid, users[uid]
}'
```

---

# 77. Detectar GID duplicados

```bash
getent group | \
awk -F: '{count[$3]++; groups[$3]=groups[$3]" "$1} END {
  for (gid in count)
    if (count[gid] > 1)
      print "GID duplicado:", gid, groups[gid]
}'
```

---

# 78. Detectar directorios personales inexistentes

```bash
getent passwd | \
awk -F: '$3 >= 1000 {print $1 ":" $6}' | \
while IFS=: read -r user home
do
  if [ ! -d "$home" ]; then
    echo "Home inexistente: $user -> $home"
  fi
done
```

---

# 79. Detectar propietarios incorrectos de home

```bash
getent passwd | \
awk -F: '$3 >= 1000 {print $1 ":" $6}' | \
while IFS=: read -r user home
do
  if [ -d "$home" ]; then
    owner=$(stat -c '%U' "$home")
    if [ "$owner" != "$user" ]; then
      echo "Propietario incorrecto: $home pertenece a $owner, esperado $user"
    fi
  fi
done
```

---

# 80. Guardar evidencia del laboratorio

Crear reporte:

```bash
{
  echo "=== FECHA ==="
  date

  echo
  echo "=== USUARIOS ==="
  getent passwd ajimenez
  getent passwd mgarcia
  getent passwd jperez
  getent passwd lrodriguez
  getent passwd csoporte
  getent passwd auditor

  echo
  echo "=== GRUPOS ==="
  getent group linuxadmins
  getent group dba
  getent group developers
  getent group security
  getent group support
  getent group audit
  getent group project_alpha

  echo
  echo "=== IDENTIDADES ==="
  id ajimenez
  id mgarcia
  id jperez
  id lrodriguez
  id csoporte
  id auditor
} | sudo tee /root/laboratorio-usuarios/evidencias/reporte-fase1.txt
```

---

# 81. Validaciones finales

## 81.1 Validar usuarios

```bash
for user in ajimenez mgarcia jperez lrodriguez csoporte auditor
do
  getent passwd "$user" >/dev/null && \
  echo "OK: usuario $user existe" || \
  echo "ERROR: usuario $user no existe"
done
```

---

## 81.2 Validar grupos

```bash
for group in linuxadmins dba developers security support audit project_alpha
do
  getent group "$group" >/dev/null && \
  echo "OK: grupo $group existe" || \
  echo "ERROR: grupo $group no existe"
done
```

---

## 81.3 Validar homes

```bash
for user in ajimenez mgarcia jperez lrodriguez csoporte auditor
do
  home=$(getent passwd "$user" | cut -d: -f6)

  if [ -d "$home" ]; then
    echo "OK: $user tiene home $home"
  else
    echo "ERROR: $user no tiene home válido"
  fi
done
```

---

## 81.4 Validar shell de auditor

```bash
test "$(getent passwd auditor | cut -d: -f7)" = "/sbin/nologin" && \
echo "OK: auditor no tiene shell interactivo" || \
echo "ERROR: revisar shell de auditor"
```

---

## 81.5 Validar grupo compartido

```bash
getent group project_alpha
```

Debe incluir:

```text
ajimenez
mgarcia
jperez
```

---

# 82. Errores comunes

## Error 1: usuario ya existe

Mensaje:

```text
useradd: user 'ajimenez' already exists
```

Diagnóstico:

```bash
getent passwd ajimenez
```

No crees una cuenta duplicada sin investigar la existente.

---

## Error 2: UID ya utilizado

Mensaje:

```text
useradd: UID 4000 is not unique
```

Verificar:

```bash
getent passwd 4000
```

Seleccionar otro UID o revisar la identidad existente.

---

## Error 3: grupo no existe

Mensaje:

```text
useradd: group 'linuxadmins' does not exist
```

Solución:

```bash
sudo groupadd linuxadmins
```

Luego crear el usuario.

---

## Error 4: GID ya utilizado

```bash
getent group 3000
```

No fuerces el mismo GID sin comprender las consecuencias.

---

## Error 5: home no creado

Posible causa:

- Se omitió `-m`.
- Configuración de `useradd`.
- Ruta personalizada inválida.
- Permisos insuficientes.

Crear manualmente:

```bash
sudo mkdir -p /home/usuario
sudo chown usuario:grupo /home/usuario
sudo chmod 700 /home/usuario
```

---

## Error 6: usuario no reconoce nuevo grupo

Cerrar sesión y volver a entrar.

También:

```bash
newgrp nombre_grupo
```

---

## Error 7: se eliminaron grupos secundarios

Causa probable:

```bash
usermod -G grupo usuario
```

sin `-a`.

Corregir agregando nuevamente todos los grupos necesarios.

---

## Error 8: no se puede eliminar un grupo

Posible causa:

```text
El grupo es primario de un usuario
```

Buscar usuarios que utilizan el GID:

```bash
gid=$(getent group nombre_grupo | cut -d: -f3)

getent passwd | awk -F: -v gid="$gid" '$4 == gid {print $1}'
```

---

## Error 9: cuenta bloqueada todavía accede por SSH

La contraseña puede estar bloqueada, pero la clave pública sigue funcionando.

Revisar:

```text
~/.ssh/authorized_keys
```

También:

- Shell.
- Expiración.
- Configuración SSH.
- Certificados.
- SSSD.
- Sesiones activas.

---

## Error 10: archivos con UID numérico

Después de eliminar o cambiar un usuario, `ls -l` puede mostrar:

```text
4000
```

en lugar del nombre.

Buscar:

```bash
sudo find / -xdev -uid 4000 -ls 2>/dev/null
```

---

# 83. Buenas prácticas

- Utilizar nombres de cuenta estandarizados.
- Asignar un usuario individual a cada persona.
- Evitar cuentas compartidas.
- Utilizar grupos para asignar acceso.
- Mantener privilegios mínimos.
- No trabajar diariamente como `root`.
- Utilizar `sudo`.
- Revisar sesiones antes de bloquear cuentas.
- Respaldar datos antes de eliminar usuarios.
- Configurar expiración para cuentas temporales.
- Utilizar `/sbin/nologin` para cuentas de servicio.
- Bloquear contraseñas de servicios.
- No almacenar contraseñas en scripts.
- Verificar UID y GID antes de asignarlos.
- Documentar cambios.
- Auditar cuentas periódicamente.
- Eliminar accesos de empleados desvinculados.
- Revisar claves SSH.
- Utilizar sistemas centralizados en infraestructuras grandes.
- Validar siempre los cambios mediante `getent` e `id`.

---

# 84. Checklist del administrador

```text
□ Grupos creados con GID correcto

□ Usuarios creados con UID correcto

□ Grupo primario validado

□ Grupos secundarios validados

□ Directorio personal creado

□ Propietario del home correcto

□ Shell configurado

□ Contraseña asignada o bloqueada

□ Expiración configurada

□ Cuenta temporal documentada

□ Cuenta de servicio sin acceso interactivo

□ Privilegios sudo revisados

□ Archivos del usuario auditados

□ Evidencia almacenada

□ Cambios documentados
```

---

# 85. Laboratorios RHCSA

## Laboratorio 1: crear grupo y usuario

Crea:

```text
Grupo: operaciones
GID: 3300

Usuario: operador01
UID: 4300
Grupo primario: operaciones
Shell: /bin/bash
Home: /home/operador01
```

Valida con:

```bash
getent passwd operador01
getent group operaciones
id operador01
```

---

## Laboratorio 2: grupo secundario

Crea:

```text
Grupo: mantenimiento
GID: 3301
```

Agrega `operador01` como miembro secundario sin eliminar sus grupos existentes.

Valida:

```bash
id operador01
```

---

## Laboratorio 3: política de contraseña

Configura para `operador01`:

```text
Mínimo: 2 días
Máximo: 60 días
Aviso: 10 días
Inactividad: 5 días
```

Valida:

```bash
sudo chage -l operador01
```

---

## Laboratorio 4: cuenta temporal

Crea:

```text
Usuario: temporal01
Expiración: 2026-12-31
Shell: /bin/bash
```

Verifica la fecha de expiración.

---

## Laboratorio 5: cuenta de servicio

Crea:

```text
Usuario: appservice
Tipo: sistema
Shell: /sbin/nologin
Sin home
Sin contraseña interactiva
```

Valida:

```bash
getent passwd appservice
sudo passwd -S appservice
```

---

## Laboratorio 6: bloquear y desbloquear

Bloquea `operador01`.

Verifica su estado.

Luego desbloquéalo y vuelve a validar.

---

## Laboratorio 7: modificar home

Cambia el home de `operador01` a:

```text
/srv/users/operador01
```

Mueve el contenido existente.

Verifica:

- Entrada en `/etc/passwd`.
- Existencia del nuevo directorio.
- Propietario.
- Grupo.
- Permisos.

---

## Laboratorio 8: auditoría

Genera un reporte que muestre:

- Nombre de usuario.
- UID.
- GID.
- Home.
- Shell.
- Grupos.
- Estado de contraseña.
- Política de expiración.

---

## Laboratorio 9: eliminación segura

Crea un usuario de prueba.

Genera archivos bajo:

```text
/home
/tmp
/var/tmp
```

Después:

- Busca todos sus archivos.
- Bloquea la cuenta.
- Respalda el home.
- Elimina la cuenta.
- Verifica si quedaron archivos huérfanos.

---

## Laboratorio 10: reconstrucción

Elimina una cuenta de laboratorio sin borrar su home.

Luego:

- Crea nuevamente la cuenta con el mismo UID.
- Asigna el grupo correcto.
- Reasigna el propietario del home.
- Verifica el acceso.

---

# 86. Desafío final de la Fase 1

La empresa incorporará el siguiente personal:

| Usuario | UID | Grupo primario | Grupos secundarios | Shell |
|---|---:|---|---|---|
| `admin02` | 4400 | `linuxadmins` | `wheel`, `project_alpha` | `/bin/bash` |
| `dba02` | 4401 | `dba` | `project_alpha` | `/bin/bash` |
| `dev02` | 4402 | `developers` | `project_alpha` | `/bin/bash` |
| `audit02` | 4403 | `audit` | Ninguno | `/sbin/nologin` |
| `backupsvc` | Automático | Grupo de sistema | Ninguno | `/sbin/nologin` |

Requisitos:

```text
□ Crear todos los usuarios

□ Verificar UID y GID

□ Crear directorios personales cuando corresponda

□ Configurar comentarios

□ Configurar shells

□ Agregar grupos secundarios sin sobrescribir otros grupos

□ Asignar contraseña a usuarios humanos

□ Bloquear la contraseña de cuentas de servicio

□ Configurar expiración de 90 días para contraseñas humanas

□ Configurar advertencia de 14 días

□ Forzar cambio de contraseña en el próximo inicio

□ Generar un reporte de validación

□ Guardar evidencia del laboratorio
```

---

# 87. Criterios de evaluación

| Criterio | Puntos |
|---|---:|
| Creación correcta de grupos | 10 |
| Creación correcta de usuarios | 15 |
| UID y GID correctos | 10 |
| Home y shell correctos | 10 |
| Contraseñas configuradas | 10 |
| Expiración configurada | 10 |
| Grupos secundarios correctos | 10 |
| Cuenta de servicio segura | 10 |
| Validación final | 10 |
| Documentación y evidencias | 5 |
| **Total** | **100** |

---

# 88. Preguntas de repaso

1. ¿Qué información almacena `/etc/passwd`?
2. ¿Por qué las contraseñas no se almacenan directamente en `/etc/passwd`?
3. ¿Qué información contiene `/etc/shadow`?
4. ¿Qué diferencia existe entre UID y GID?
5. ¿Qué es un grupo primario?
6. ¿Qué es un grupo secundario?
7. ¿Para qué sirve `useradd -m`?
8. ¿Qué función cumple `useradd -s`?
9. ¿Cómo se asigna un UID específico?
10. ¿Cómo se consulta una cuenta mediante `getent`?
11. ¿Qué diferencia existe entre `id` y `groups`?
12. ¿Por qué debe utilizarse `usermod -aG`?
13. ¿Qué puede ocurrir al omitir `-a`?
14. ¿Cómo se bloquea una contraseña?
15. ¿Cómo se desbloquea?
16. ¿Bloquear una contraseña impide necesariamente el acceso por clave SSH?
17. ¿Para qué sirve `chage -M`?
18. ¿Para qué sirve `chage -E`?
19. ¿Qué diferencia existe entre expiración de contraseña y expiración de cuenta?
20. ¿Qué shell debería utilizar una cuenta sin acceso interactivo?
21. ¿Qué función cumple `/etc/skel`?
22. ¿Cómo se crea una cuenta de sistema?
23. ¿Por qué deben buscarse archivos antes de eliminar una cuenta?
24. ¿Qué sucede con los archivos después de cambiar un UID?
25. ¿Qué comando permite modificar un GID?
26. ¿Por qué no deben existir UID duplicados?
27. ¿Qué ventajas ofrece una administración centralizada de identidades?
28. ¿Por qué las cuentas compartidas dificultan la auditoría?
29. ¿Qué debe revisarse antes de finalizar procesos de un usuario?
30. ¿Cuál sería un procedimiento seguro para dar de baja una cuenta?

---

# 89. Resumen de la Fase 1

En esta fase desarrollamos un laboratorio completo de administración local de usuarios y grupos.

Aprendimos a:

- Crear grupos mediante `groupadd`.
- Asignar GID específicos.
- Crear usuarios mediante `useradd`.
- Asignar UID.
- Definir grupos primarios.
- Configurar grupos secundarios.
- Crear homes.
- Asignar shells.
- Establecer contraseñas.
- Forzar cambios de contraseña.
- Configurar vencimiento mediante `chage`.
- Bloquear y desbloquear cuentas.
- Crear cuentas de servicio.
- Utilizar `/sbin/nologin`.
- Modificar usuarios mediante `usermod`.
- Modificar grupos mediante `groupmod`.
- Consultar identidades con `getent`.
- Verificar membresías mediante `id` y `groups`.
- Auditar archivos de usuarios.
- Identificar UID y GID duplicados.
- Eliminar cuentas de forma segura.
- Generar evidencias de administración.

Estas competencias son esenciales tanto para el examen RHCSA como para la administración diaria de servidores Linux empresariales.

---

# Próxima fase

## Fase 2 — Administración avanzada, permisos compartidos y automatización

En la siguiente fase se desarrollarán:

- Directorios colaborativos.
- Grupos privados de usuario.
- Permisos especiales.
- SGID en directorios.
- Sticky Bit.
- Umask.
- ACL para usuarios y grupos.
- Permisos predeterminados.
- Administración de sudo.
- Restricción de comandos.
- Claves SSH.
- Bloqueo integral de cuentas.
- Auditoría de accesos.
- Scripts para altas y bajas.
- Validaciones automatizadas.
- Laboratorio final RHCSA de usuarios, grupos y permisos.