# 50. Laboratorio Integrador de Seguridad

> **Módulo 7: Seguridad del Sistema**  
> **Página 50 del Manual RHCSA**

---

# Objetivos del laboratorio

Al finalizar este laboratorio serás capaz de:

- Configurar Firewalld.
- Administrar zonas del Firewall.
- Abrir y cerrar servicios.
- Trabajar con SELinux.
- Corregir contextos.
- Administrar booleanos.
- Diagnosticar problemas de SELinux.
- Configurar OpenSSH.
- Implementar Hardening básico.
- Configurar autenticación mediante llaves SSH.
- Consultar registros de seguridad.
- Resolver incidentes utilizando herramientas de auditoría.

Este laboratorio integra todos los conocimientos adquiridos durante el **Módulo 7**.

---

# Escenario

Acabas de recibir un servidor Red Hat Enterprise Linux recién instalado.

El servidor será utilizado para:

- Administración remota mediante SSH.
- Servidor Web (Apache).
- Auditoría de seguridad.
- Acceso únicamente desde administradores autorizados.

Tu tarea consiste en preparar el servidor siguiendo las mejores prácticas de seguridad.

---

# Información del laboratorio

| Elemento | Valor |
|----------|--------|
| Hostname | `server01.lab.local` |
| Dirección IP | `192.168.1.100` |
| Usuario administrador | `adminuser` |
| Servicio Web | Apache |
| Firewall | Firewalld |
| SELinux | Enforcing |
| Servicio SSH | OpenSSH |

---

# Actividad 1 - Verificar SELinux

Consultar el estado:

```bash
getenforce
```

Consultar información completa:

```bash
sestatus
```

### Resultado esperado

```
Current mode:

Enforcing
```

---

# Actividad 2 - Verificar Firewalld

Consultar el estado:

```bash
systemctl status firewalld
```

Verificar la zona activa:

```bash
firewall-cmd --get-active-zones
```

Mostrar la configuración:

```bash
firewall-cmd --list-all
```

### Objetivo

Confirmar que el Firewall está funcionando correctamente.

---

# Actividad 3 - Configurar Apache

Instalar:

```bash
sudo dnf install httpd -y
```

Iniciar:

```bash
sudo systemctl enable --now httpd
```

Comprobar:

```bash
systemctl status httpd
```

---

# Actividad 4 - Configurar Firewalld

Permitir HTTP.

```bash
sudo firewall-cmd \
--permanent \
--add-service=http
```

Aplicar:

```bash
sudo firewall-cmd --reload
```

Verificar:

```bash
firewall-cmd --list-services
```

Debe aparecer:

```
http
```

---

# Actividad 5 - Verificar SELinux

Consultar el contexto:

```bash
ls -ldZ /var/www/html
```

Consultar el contexto del proceso:

```bash
ps -eZ | grep httpd
```

---

# Actividad 6 - Crear un nuevo sitio web

Crear un nuevo directorio.

```bash
sudo mkdir /web
```

Crear una página.

```bash
echo "<h1>RHCSA</h1>" \
| sudo tee /web/index.html
```

Consultar el contexto.

```bash
ls -lZ /web
```

Observa que el contexto no es el esperado para un sitio web.

---

# Actividad 7 - Configurar un contexto permanente

Crear la regla:

```bash
sudo semanage fcontext \
-a \
-t httpd_sys_content_t \
"/web(/.*)?"
```

Aplicar:

```bash
sudo restorecon -Rv /web
```

Verificar:

```bash
ls -lZ /web
```

### Resultado esperado

```
httpd_sys_content_t
```

---

# Actividad 8 - Configurar un booleano

Consultar:

```bash
getsebool httpd_can_network_connect
```

Activarlo permanentemente:

```bash
sudo setsebool \
-P \
httpd_can_network_connect on
```

Verificar:

```bash
getsebool httpd_can_network_connect
```

---

# Actividad 9 - Diagnóstico

Buscar eventos AVC.

```bash
sudo ausearch -m AVC
```

Consultar:

```bash
journalctl | grep AVC
```

Interpretar:

```bash
sudo ausearch -m AVC \
| audit2why
```

---

# Actividad 10 - Configurar OpenSSH

Consultar:

```bash
systemctl status sshd
```

Editar:

```bash
sudo vi /etc/ssh/sshd_config
```

Configurar:

```text
PermitRootLogin no

MaxAuthTries 3
```

Validar:

```bash
sudo sshd -t
```

Reiniciar:

```bash
sudo systemctl restart sshd
```

---

# Actividad 11 - Hardening

Agregar:

```text
LoginGraceTime 30

PermitEmptyPasswords no

ClientAliveInterval 300

ClientAliveCountMax 2
```

Verificar:

```bash
sudo sshd -T
```

---

# Actividad 12 - Crear llaves SSH

Generar:

```bash
ssh-keygen -t ed25519
```

Copiar:

```bash
ssh-copy-id adminuser@192.168.1.100
```

Conectarse:

```bash
ssh adminuser@192.168.1.100
```

---

# Actividad 13 - Consultar registros SSH

```bash
journalctl -u sshd
```

Seguir eventos:

```bash
journalctl -fu sshd
```

Realizar una nueva conexión SSH desde otro equipo y observar los registros en tiempo real.

---

# Actividad 14 - Consultar auditoría

Consultar autenticaciones.

```bash
ausearch -m USER_LOGIN
```

Generar un informe.

```bash
aureport -au
```

Consultar conexiones.

```bash
last
```

Intentos fallidos.

```bash
sudo lastb
```

---

# Actividad 15 - Verificar puertos

Mostrar puertos abiertos.

```bash
ss -tulpn
```

Verificar el Firewall.

```bash
firewall-cmd --list-all
```

---

# Actividad 16 - Comprobación final

Verificar:

## SELinux

```bash
getenforce
```

Debe indicar:

```
Enforcing
```

---

## Firewalld

```bash
systemctl status firewalld
```

Debe estar:

```
active (running)
```

---

## SSH

```bash
systemctl status sshd
```

Debe estar:

```
active (running)
```

---

## Apache

```bash
systemctl status httpd
```

Debe estar:

```
active (running)
```

---

## Contexto Web

```bash
ls -ldZ /web
```

Debe mostrar:

```
httpd_sys_content_t
```

---

## Booleano

```bash
getsebool httpd_can_network_connect
```

Debe indicar:

```
on
```

---

# Escenarios de resolución de problemas

## Escenario 1

No puedes acceder al sitio web.

### Comprueba:

```bash
systemctl status httpd

firewall-cmd --list-services

ls -lZ /web

getenforce
```

---

## Escenario 2

No puedes conectarte por SSH.

### Comprueba:

```bash
systemctl status sshd

ss -tulpn | grep ssh

journalctl -u sshd

sudo sshd -t
```

---

## Escenario 3

SELinux bloquea Apache.

### Comprueba:

```bash
ausearch -m AVC

audit2why

restorecon -Rv /web
```

---

## Escenario 4

Firewall bloquea HTTP.

Verifica:

```bash
firewall-cmd --list-services
```

Si HTTP no aparece:

```bash
sudo firewall-cmd \
--permanent \
--add-service=http

sudo firewall-cmd --reload
```

---

# Checklist final RHCSA

Marca cada tarea cuando la completes.

| Tarea | Estado |
|--------|:------:|
| SELinux en modo Enforcing | ☐ |
| Firewalld iniciado | ☐ |
| Zona activa verificada | ☐ |
| Servicio HTTP habilitado | ☐ |
| Apache funcionando | ☐ |
| Contextos SELinux verificados | ☐ |
| `restorecon` ejecutado | ☐ |
| `semanage` configurado | ☐ |
| Booleano configurado | ☐ |
| AVC revisados | ☐ |
| OpenSSH configurado | ☐ |
| `PermitRootLogin no` | ☐ |
| Hardening aplicado | ☐ |
| Llaves SSH creadas | ☐ |
| Acceso por llave probado | ☐ |
| Logs revisados | ☐ |
| `ausearch` utilizado | ☐ |
| `aureport` ejecutado | ☐ |
| Firewall validado | ☐ |
| Laboratorio completado | ☐ |

---

# Desafío Final RHCSA

Realiza todas las tareas anteriores **sin consultar apuntes**.

Al finalizar, verifica que eres capaz de responder las siguientes preguntas:

1. ¿Cuál es la diferencia entre **Firewalld** y **SELinux**?
2. ¿Cuándo utilizarías `restorecon` y cuándo `semanage`?
3. ¿Qué diferencia existe entre un **contexto** y un **booleano**?
4. ¿Cómo interpretarías un mensaje **AVC**?
5. ¿Cómo habilitarías un nuevo puerto para SSH considerando **Firewalld** y **SELinux**?
6. ¿Cómo restringirías el acceso SSH únicamente a un grupo específico de administradores?
7. ¿Qué herramientas utilizarías para investigar un intento fallido de inicio de sesión?
8. ¿Qué harías si Apache devuelve un error **403 Forbidden** mientras SELinux está en modo **Enforcing**?

---

# Resumen del Módulo 7

En este módulo aprendiste a:

- Comprender los principios de seguridad en Red Hat Enterprise Linux.
- Configurar y administrar **Firewalld**.
- Trabajar con **SELinux** (modos, contextos, `restorecon`, `semanage` y booleanos).
- Diagnosticar problemas utilizando **AVC**, `journalctl`, `ausearch` y `audit2why`.
- Configurar y proteger **OpenSSH**.
- Implementar **Hardening** en SSH.
- Utilizar autenticación mediante **llaves SSH**.
- Consultar registros y realizar auditorías con **journalctl** y **auditd**.

Estas habilidades forman parte de las competencias fundamentales evaluadas en el examen **RHCSA** y representan buenas prácticas para la administración segura de servidores Linux en entornos empresariales.

---

# Próximo módulo

Con este laboratorio finaliza el **Módulo 7: Seguridad**.

El siguiente paso en el roadmap es el:

> **Módulo 8: Almacenamiento (Storage)**

En él aprenderás a administrar discos, particiones, sistemas de archivos, LVM, cuotas, montaje automático, Stratis, VDO y otras tecnologías de almacenamiento utilizadas en Red Hat Enterprise Linux.