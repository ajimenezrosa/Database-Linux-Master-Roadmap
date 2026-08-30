# 56. Laboratorio de Seguridad en Linux

> **Módulo 7 — Seguridad en Red Hat Enterprise Linux (RHCSA)**
>
> **Archivo:** `56-laboratorio-seguridad.md`
>
> **Nivel:** RHCSA
>
> **Laboratorio:** Integración completa de Seguridad
>
> **Duración estimada:** 8 a 10 horas
>
> **Objetivo:** Implementar un servidor Linux completamente asegurado aplicando autenticación, permisos, ACL, SELinux, sudo, políticas de contraseñas, auditoría, Firewall, SSH, servicios seguros y procedimientos de endurecimiento (Hardening) similares a un entorno empresarial.

---

# 1. Objetivos del laboratorio

Al finalizar este laboratorio el estudiante será capaz de:

- Aplicar el principio de mínimo privilegio.
- Administrar usuarios y grupos de forma segura.
- Configurar permisos especiales.
- Administrar ACL.
- Implementar políticas de contraseña.
- Configurar sudo de forma segura.
- Endurecer SSH.
- Implementar SELinux.
- Administrar contextos.
- Crear políticas básicas.
- Configurar Firewalld.
- Administrar servicios seguros.
- Configurar auditoría.
- Analizar eventos mediante auditd.
- Proteger archivos críticos.
- Detectar intentos de acceso.
- Automatizar verificaciones de seguridad.
- Documentar toda la configuración.

---

# 2. Escenario empresarial

La empresa **TechCloud Security** ha sido contratada para proteger la infraestructura Linux de una institución financiera.

El servidor almacenará información crítica y deberá cumplir con estrictas políticas de seguridad.

El administrador será responsable de implementar todas las medidas necesarias para minimizar riesgos.

---

# 3. Objetivos empresariales

El servidor deberá cumplir con los siguientes requisitos:

- Acceso únicamente mediante SSH seguro.
- Firewall configurado.
- SELinux en modo Enforcing.
- Usuarios con privilegios mínimos.
- Auditoría de cambios.
- Políticas de contraseñas.
- Protección contra accesos no autorizados.
- Restricción del uso de sudo.
- Registro de todas las actividades importantes.

---

# 4. Información del servidor

| Parámetro | Valor |
|-----------|-------|
| Hostname | secure01.techcloud.local |
| Dominio | techcloud.local |
| Sistema | Rocky Linux / RHEL / AlmaLinux |
| SELinux | Enforcing |
| Firewall | Firewalld |
| SSH | Puerto 22 |

---

# 5. Requerimientos generales

El laboratorio deberá incluir:

- Usuarios.
- Grupos.
- ACL.
- Permisos especiales.
- sudo.
- PAM.
- Políticas de contraseña.
- SELinux.
- Firewalld.
- SSH.
- auditd.
- Logs.
- Scripts de validación.
- Reportes.

---

# 6. Usuarios

Crear los usuarios:

| Usuario | Grupo |
|----------|--------|
| admin01 | administradores |
| admin02 | administradores |
| auditor | auditoria |
| developer | desarrollo |
| backup | backups |
| operador | operadores |

---

# 7. Grupos

Crear:

```text
administradores

auditoria

desarrollo

backups

operadores
```

---

# 8. Contraseñas

Todas deberán cumplir:

- mínimo 14 caracteres
- mayúsculas
- minúsculas
- números
- caracteres especiales

---

# 9. Política de expiración

Configurar:

| Parámetro | Valor |
|-----------|-------|
| Expiración | 90 días |
| Aviso | 14 días |
| Mínimo | 1 día |

---

# 10. Directorios protegidos

Crear:

```text
/empresa

/empresa/desarrollo

/empresa/auditoria

/empresa/backups

/empresa/privado
```

---

# 11. Permisos

Aplicar permisos adecuados.

Ningún usuario fuera del grupo correspondiente deberá acceder.

---

# 12. ACL

Configurar permisos especiales para:

developer

auditor

admin01

---

# 13. Sticky Bit

Crear:

```text
/empresa/tmp
```

Aplicar:

```text
1777
```

---

# 14. SGID

Aplicar SGID al directorio:

```text
/empresa/desarrollo
```

---

# 15. SUID

Verificar todos los binarios SUID.

Generar reporte.

---

# 16. sudo

Permitir:

admin01

admin02

Ejecutar únicamente:

- systemctl
- journalctl
- dnf
- firewall-cmd

---

# 17. SSH

Configurar:

- LoginRoot no
- PasswordAuthentication yes
- MaxAuthTries 3
- PermitEmptyPasswords no
- ClientAliveInterval 300
- ClientAliveCountMax 2

---

# 18. Firewall

Permitir:

SSH

HTTPS

DNS

ICMP

Bloquear:

FTP

TELNET

SMTP

---

# 19. SELinux

Mantener:

```text
Enforcing
```

---

# 20. Contextos

Crear:

```text
/empresa/web
```

Asignar contexto adecuado para contenido web.

---

# 21. Booleanos

Revisar:

```text
httpd_can_network_connect

httpd_enable_homedirs

ssh_sysadm_login
```

---

# 22. Auditoría

Registrar:

- creación de usuarios
- eliminación
- cambios en sudoers
- cambios en passwd
- cambios en shadow
- cambios SELinux

---

# 23. Logs

Analizar:

```text
/var/log/secure

journalctl

ausearch

aureport
```

---

# 24. Hardening

Aplicar:

- permisos mínimos
- servicios innecesarios deshabilitados
- puertos mínimos
- cuentas bloqueadas
- root protegido

---

# 25. Servicios

Revisar todos los servicios.

Deshabilitar aquellos innecesarios.

---

# 26. Cron

Verificar permisos.

---

# 27. Systemd

Verificar servicios automáticos.

---

# 28. Integridad

Verificar:

- passwd
- group
- shadow
- sudoers

---

# 29. Reporte

Crear un reporte completo de seguridad.

---

# 30. Script de validación

Crear un script que verifique automáticamente:

- SELinux
- Firewall
- SSH
- sudo
- ACL
- Usuarios
- Contraseñas
- Auditoría
- Servicios
- Logs

---

# 31. Escenarios de fallo

Resolver:

- SELinux bloqueando Apache.
- SSH rechazando usuarios.
- Firewall bloqueando PostgreSQL.
- ACL incorrecta.
- sudo sin permisos.
- Contexto SELinux incorrecto.
- Booleano mal configurado.
- Usuario bloqueado.
- Contraseña expirada.
- Servicio no inicia.

---

# 32. Evidencias

Guardar:

```text
getenforce

sestatus

firewall-cmd --list-all

ss -tulpn

id

groups

getfacl

ls -l

ls -Z

ps -ef

systemctl

journalctl

ausearch

aureport
```

---

# 33. Desafío RHCSA

El instructor eliminará:

- usuarios
- grupos
- ACL
- sudo
- reglas firewall
- contextos SELinux

El estudiante dispondrá de **120 minutos** para reconstruir completamente la seguridad del servidor.

---

# 34. Checklist

```text
□ Usuarios

□ Grupos

□ Contraseñas

□ Expiración

□ ACL

□ Sticky Bit

□ SGID

□ SUID

□ sudo

□ SSH

□ Firewall

□ SELinux

□ Contextos

□ Booleanos

□ Auditd

□ Logs

□ Hardening

□ Servicios

□ Cron

□ Reporte

□ Script de validación

□ Reinicio exitoso
```

---

# 35. Evaluación

| Tema | Puntos |
|--------|---------|
| Usuarios | 5 |
| ACL | 10 |
| Permisos | 10 |
| sudo | 10 |
| SSH | 10 |
| Firewall | 10 |
| SELinux | 20 |
| Auditoría | 10 |
| Hardening | 10 |
| Automatización | 5 |

**Total: 100 puntos**

---

# 36. Resultado esperado

Al finalizar este laboratorio el estudiante habrá implementado un servidor Linux endurecido siguiendo buenas prácticas de seguridad utilizadas en entornos empresariales, integrando usuarios, permisos, ACL, sudo, SELinux, Firewalld, SSH, auditoría y automatización. Este laboratorio sirve como preparación práctica para los objetivos de seguridad del examen **RHCSA** y como base para futuras certificaciones como **RHCE**, **Security+** o especializaciones en administración segura de sistemas Linux.

----
