# 57. Simulación Oficial del Examen RHCSA (EX200)

> **Módulo Final — Simulación Completa RHCSA**
>
> **Archivo:** `57-simulacion-examen-rhcsa.md`
>
> **Nivel:** RHCSA
>
> **Duración:** 3 horas (180 minutos)
>
> **Modalidad:** Práctica
>
> **Objetivo:** Reproducir una simulación extremadamente cercana al examen oficial RHCSA (EX200), integrando todos los temas estudiados durante el curso bajo presión de tiempo y sin asistencia externa.

---

# Introducción

Esta simulación reproduce la metodología utilizada en el examen oficial RHCSA.

Durante el examen NO existen preguntas de opción múltiple.

Todo consiste en administrar un servidor Linux real.

El estudiante debe completar tareas dentro de un tiempo limitado.

No existe Internet.

No existe documentación externa.

Solo se permite utilizar la documentación incluida en el sistema mediante:

```bash
man
```

y

```bash
--help
```

---

# Reglas del examen

Durante esta simulación queda prohibido:

- Buscar información en Internet.
- Utilizar IA.
- Utilizar teléfonos.
- Consultar apuntes.
- Utilizar scripts previamente preparados.
- Copiar archivos desde otros servidores.

Solo se permite utilizar:

- Documentación del sistema.
- Comandos Linux.
- Páginas man.
- Ayuda integrada.

---

# Objetivos evaluados

Durante el examen se evaluarán los siguientes dominios.

| Tema | Peso aproximado |
|-------|-----------------|
| Usuarios y grupos | 10% |
| Permisos | 8% |
| ACL | 5% |
| LVM | 12% |
| Sistemas de archivos | 8% |
| Montajes | 8% |
| Redes | 12% |
| Firewalld | 8% |
| SELinux | 12% |
| Servicios | 7% |
| Contenedores Podman | 5% |
| Bash | 5% |

---

# Escenario

La empresa **TechCloud Corporation** ha contratado a un nuevo administrador Linux.

Se entrega un servidor completamente limpio.

El administrador dispone de 180 minutos para preparar el servidor para producción.

---

# Información inicial

Hostname requerido

```text
server01.techcloud.local
```

Dominio

```text
techcloud.local
```

Usuario administrador

```text
admin01
```

---

# Tarea 1 — Hostname

Configurar el hostname solicitado.

Debe permanecer después del reinicio.

---

# Tarea 2 — Usuarios

Crear:

```text
developer

backup

auditor

operator
```

Crear grupos:

```text
developers

backups

audit

operations
```

Asignar correctamente.

---

# Tarea 3 — Contraseñas

Asignar contraseña a todos los usuarios.

Forzar cambio en el primer inicio de sesión únicamente para:

```text
developer
```

---

# Tarea 4 — Expiración

Configurar:

```text
90 días

mínimo 1

aviso 14
```

---

# Tarea 5 — sudo

Permitir que:

```text
admin01
```

pueda ejecutar:

```text
systemctl

dnf

journalctl
```

sin convertirse en root.

---

# Tarea 6 — Directorios

Crear:

```text
/empresa

/empresa/desarrollo

/empresa/backup

/empresa/auditoria

/empresa/tmp
```

---

# Tarea 7 — Permisos

Aplicar:

- SGID
- Sticky Bit
- ACL

según corresponda.

---

# Tarea 8 — LVM

Existe un disco adicional.

Crear:

Volume Group

```text
VGDATA
```

Logical Volumes

```text
LVAPP

LVBACKUP
```

Montarlos permanentemente.

---

# Tarea 9 — Sistemas de archivos

Formatear utilizando:

```text
XFS
```

---

# Tarea 10 — Auto montaje

Configurar correctamente:

```text
/etc/fstab
```

Validar.

---

# Tarea 11 — Red

Configurar:

IPv4

Gateway

DNS

Hostname

Persistencia.

---

# Tarea 12 — Firewall

Permitir únicamente:

SSH

HTTP

HTTPS

---

# Tarea 13 — SELinux

Mantener:

```text
Enforcing
```

Crear:

```text
/webdata
```

Asignar contexto adecuado.

---

# Tarea 14 — Apache

Instalar.

Configurar.

Publicar:

```text
Bienvenido al laboratorio RHCSA
```

---

# Tarea 15 — SELinux Apache

Permitir acceso al nuevo directorio.

---

# Tarea 16 — Servicios

Habilitar:

Apache

Firewalld

---

# Tarea 17 — Contenedor

Crear un contenedor Podman.

Nombre:

```text
webtest
```

Imagen:

```text
ubi9/httpd
```

Publicar puerto:

```text
8080
```

---

# Tarea 18 — Bash

Crear un script llamado:

```text
checkserver.sh
```

Debe mostrar:

- hostname
- memoria
- disco
- usuarios conectados
- uptime
- SELinux
- Firewall

---

# Tarea 19 — Logs

Encontrar:

- errores SSH
- errores Apache

---

# Tarea 20 — Backup

Crear un archivo:

```text
backup.tar.gz
```

que contenga:

```text
/empresa
```

---

# Tarea 21 — Cron

Programar el backup diariamente.

---

# Tarea 22 — Enlaces

Crear:

Hard Link

Soft Link

---

# Tarea 23 — Procesos

Encontrar el proceso que consume más CPU.

Finalizarlo.

---

# Tarea 24 — Servicios

Revisar todos los servicios iniciados automáticamente.

---

# Tarea 25 — Boot

Cambiar el target por defecto a:

```text
multi-user.target
```

---

# Tarea 26 — SELinux

Encontrar el contexto de:

```text
/etc/passwd
```

---

# Tarea 27 — Firewalld

Abrir el puerto:

```text
8080
```

---

# Tarea 28 — SSH

Deshabilitar:

```text
PermitRootLogin
```

---

# Tarea 29 — Auditoría

Encontrar los últimos accesos fallidos.

---

# Tarea 30 — Reinicio

Reiniciar.

Todo deberá continuar funcionando.

---

# Escenarios sorpresa

El instructor podrá introducir cualquiera de los siguientes problemas:

- Firewall bloqueando Apache.
- SELinux bloqueando contenido.
- UUID incorrecto.
- Error en fstab.
- Usuario bloqueado.
- ACL incorrecta.
- Servicio caído.
- Ruta incorrecta.
- DNS incorrecto.
- LVM sin montar.

El estudiante deberá diagnosticar y corregir cada problema.

---

# Evidencias

Al finalizar deberán mostrarse los siguientes comandos:

```bash
hostnamectl
```

```bash
id developer
```

```bash
groups developer
```

```bash
lsblk
```

```bash
vgs
```

```bash
lvs
```

```bash
pvs
```

```bash
mount
```

```bash
df -h
```

```bash
cat /etc/fstab
```

```bash
ip addr
```

```bash
ip route
```

```bash
firewall-cmd --list-all
```

```bash
getenforce
```

```bash
ls -Z /webdata
```

```bash
systemctl status httpd
```

```bash
systemctl status firewalld
```

```bash
podman ps
```

```bash
crontab -l
```

---

# Criterios de Evaluación

| Categoría | Puntos |
|------------|--------|
| Usuarios | 10 |
| Permisos | 8 |
| ACL | 5 |
| LVM | 12 |
| Sistemas de archivos | 8 |
| Redes | 12 |
| Firewall | 8 |
| SELinux | 12 |
| Servicios | 8 |
| Podman | 5 |
| Bash | 5 |
| Cron | 3 |
| Backup | 2 |
| Troubleshooting | 2 |
| **Total** | **100** |

---

# Penalizaciones

Se descontarán puntos por:

- Configuración no persistente.
- Uso de permisos inseguros.
- Servicios sin habilitar.
- Errores en `/etc/fstab`.
- SELinux en modo `Permissive`.
- Firewall deshabilitado.
- Scripts con errores.
- Contenedores detenidos.
- Directorios mal montados.
- Reinicio fallido.

---

# Estrategia recomendada

## Primeros 15 minutos

- Leer completamente el examen.
- Identificar tareas rápidas.
- Planificar el orden de ejecución.

## Minutos 15–120

- Completar tareas de mayor valor.
- Validar cada cambio inmediatamente.
- Evitar acumular errores.

## Minutos 120–165

- Revisar todos los servicios.
- Comprobar persistencia.
- Reiniciar el servidor si el examen lo permite.
- Validar nuevamente.

## Últimos 15 minutos

- Revisar evidencias.
- Confirmar que no existan errores.
- Verificar montajes, red, SELinux y Firewall.

---

# Checklist Final

```text
□ Hostname correcto

□ Usuarios creados

□ Grupos creados

□ Contraseñas configuradas

□ sudo funcional

□ Directorios creados

□ Permisos correctos

□ ACL configuradas

□ Sticky Bit

□ SGID

□ LVM creado

□ XFS creado

□ fstab correcto

□ Montajes permanentes

□ IPv4 correcta

□ DNS correcto

□ Firewall activo

□ SELinux Enforcing

□ Apache funcionando

□ Contextos SELinux correctos

□ Contenedor ejecutándose

□ Cron funcionando

□ Backup creado

□ Script Bash funcional

□ Reinicio exitoso
```

---

# Autoevaluación

Antes de considerar completada la simulación, responde honestamente:

1. ¿Pude completar todas las tareas dentro de las 3 horas?
2. ¿Reinicié el servidor y todo siguió funcionando?
3. ¿Validé cada configuración antes de pasar a la siguiente?
4. ¿Utilicé únicamente herramientas nativas del sistema?
5. ¿Fui capaz de diagnosticar los problemas sin ayuda externa?

Si respondes **"Sí"** a todas las preguntas, estás en una posición sólida para afrontar el examen oficial RHCSA.

---

# Resultado esperado

Al finalizar esta simulación habrás administrado un servidor Linux completo bajo condiciones muy similares al examen **Red Hat Certified System Administrator (RHCSA EX200)**, integrando usuarios, almacenamiento, redes, seguridad, servicios, SELinux, Firewalld, automatización y resolución de problemas en un único escenario práctico de principio a fin.

