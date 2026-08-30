# 46. Configuración Básica de OpenSSH

> **Módulo 7: Seguridad del Sistema**  
> **Página 46 del Manual RHCSA**

---

# Objetivos de aprendizaje

Al finalizar esta lección serás capaz de:

- Comprender qué es OpenSSH.
- Configurar el servicio SSH en Red Hat Enterprise Linux.
- Administrar el servicio `sshd`.
- Modificar el archivo de configuración `sshd_config`.
- Comprender las principales opciones de seguridad de SSH.
- Verificar el funcionamiento del servidor SSH.
- Aplicar buenas prácticas para proteger el acceso remoto.

---

# Introducción

La administración remota es una de las tareas más importantes de un administrador Linux.

El protocolo **SSH (Secure Shell)** permite conectarse de forma segura a un servidor mediante una conexión cifrada.

SSH reemplazó a protocolos inseguros como:

- Telnet
- rlogin
- rsh

Actualmente, prácticamente todos los servidores Linux son administrados mediante **OpenSSH**.

---

# ¿Qué es OpenSSH?

**OpenSSH** es la implementación libre del protocolo SSH.

Permite:

- Administración remota.
- Copia segura de archivos.
- Túneles cifrados.
- Reenvío de puertos.
- Autenticación mediante contraseñas o llaves públicas.

---

# Arquitectura de OpenSSH

```
                Cliente
             ssh usuario@servidor
                     │
                     │
              Conexión cifrada
                     │
                     ▼
          +----------------------+
          |      Servidor SSH    |
          |        (sshd)        |
          +----------------------+
```

---

# Componentes principales

| Componente | Función |
|------------|---------|
| `ssh` | Cliente SSH |
| `sshd` | Servidor SSH |
| `scp` | Copia segura de archivos |
| `sftp` | Transferencia segura de archivos |
| `ssh-keygen` | Generar llaves SSH |
| `ssh-copy-id` | Copiar llaves públicas al servidor |

---

# Verificar que OpenSSH está instalado

Servidor:

```bash
rpm -q openssh-server
```

Cliente:

```bash
rpm -q openssh-clients
```

---

# Instalar OpenSSH

```bash
sudo dnf install openssh-server
```

---

# Verificar el estado del servicio

```bash
systemctl status sshd
```

Salida esperada:

```
Active: active (running)
```

---

# Iniciar el servicio

```bash
sudo systemctl start sshd
```

---

# Habilitar inicio automático

```bash
sudo systemctl enable sshd
```

---

# Reiniciar el servicio

```bash
sudo systemctl restart sshd
```

---

# Recargar la configuración

Después de modificar el archivo de configuración:

```bash
sudo systemctl reload sshd
```

Si el servicio no soporta recarga:

```bash
sudo systemctl restart sshd
```

---

# Verificar que SSH escucha

```bash
sudo ss -tulpn | grep ssh
```

Ejemplo:

```
LISTEN

0.0.0.0:22

sshd
```

---

# Probar una conexión local

```bash
ssh localhost
```

---

# Probar una conexión remota

```bash
ssh usuario@192.168.1.100
```

Ejemplo:

```bash
ssh ajimenez@192.168.1.50
```

---

# El archivo sshd_config

La configuración del servidor SSH se encuentra en:

```text
/etc/ssh/sshd_config
```

Editar:

```bash
sudo vi /etc/ssh/sshd_config
```

---

# Ver la configuración activa

```bash
sudo sshd -T
```

Este comando muestra la configuración efectiva que utiliza el servicio.

---

# Principales parámetros

---

## Puerto

Por defecto:

```text
Port 22
```

Ejemplo:

```text
Port 2222
```

Después de modificar el puerto:

```bash
sudo firewall-cmd \
--permanent \
--add-port=2222/tcp

sudo firewall-cmd --reload
```

Si SELinux está habilitado, también debe permitirse el nuevo puerto:

```bash
sudo semanage port -a -t ssh_port_t -p tcp 2222
```

---

## Dirección de escucha

```text
ListenAddress 0.0.0.0
```

Escuchar únicamente en una IP:

```text
ListenAddress 192.168.1.20
```

---

## Permitir acceso al usuario root

```text
PermitRootLogin yes
```

Opciones:

| Valor | Descripción |
|--------|-------------|
| `yes` | Permite acceso directo |
| `prohibit-password` | Solo mediante llave pública |
| `no` | Prohíbe el acceso directo |

**Recomendación:**

```text
PermitRootLogin no
```

---

## Autenticación por contraseña

```text
PasswordAuthentication yes
```

Puede deshabilitarse cuando se utilizan llaves SSH.

---

## Autenticación mediante llaves

```text
PubkeyAuthentication yes
```

Generalmente viene habilitada por defecto.

---

## Usuarios permitidos

```text
AllowUsers ajimenez administrador
```

Solo estos usuarios podrán conectarse.

---

## Grupos permitidos

```text
AllowGroups wheel administradores
```

---

## Usuarios denegados

```text
DenyUsers invitado prueba
```

---

## Tiempo de espera

```text
LoginGraceTime 60
```

Tiempo máximo para autenticarse.

---

## Número de intentos

```text
MaxAuthTries 3
```

Reduce ataques de fuerza bruta.

---

## Sesiones simultáneas

```text
MaxSessions 10
```

---

## Mantener conexiones activas

```text
ClientAliveInterval 300

ClientAliveCountMax 2
```

Permite detectar clientes desconectados.

---

# Verificar la sintaxis

Antes de reiniciar el servicio:

```bash
sudo sshd -t
```

Si no hay errores:

```
(no output)
```

Esto evita dejar el servidor inaccesible por un error de configuración.

---

# Reiniciar el servicio

```bash
sudo systemctl restart sshd
```

---

# Comprobar el Firewall

Verificar que el servicio SSH está permitido.

```bash
firewall-cmd --list-services
```

Debe aparecer:

```
ssh
```

Si no:

```bash
sudo firewall-cmd \
--permanent \
--add-service=ssh

sudo firewall-cmd --reload
```

---

# Comprobar SELinux

Consultar el puerto permitido para SSH:

```bash
sudo semanage port -l | grep ssh
```

Resultado habitual:

```
ssh_port_t

tcp

22
```

---

# Ver usuarios conectados por SSH

```bash
who
```

o

```bash
w
```

---

# Ver conexiones activas

```bash
ss -tn | grep :22
```

---

# Consultar los registros

```bash
sudo journalctl -u sshd
```

Últimos eventos:

```bash
sudo journalctl -u sshd -n 50
```

Seguir en tiempo real:

```bash
sudo journalctl -u sshd -f
```

---

# Flujo de conexión SSH

```
Cliente

↓

DNS/IP

↓

Firewall

↓

Puerto 22

↓

sshd

↓

Autenticación

↓

Acceso
```

---

# Buenas prácticas RHCSA

✔ Mantener el servicio `sshd` habilitado.

✔ Verificar siempre la sintaxis con `sshd -t` antes de reiniciar.

✔ Deshabilitar el acceso directo del usuario root.

✔ Limitar el acceso mediante `AllowUsers` o `AllowGroups`.

✔ Cambiar el puerto solo si existe una política de seguridad que lo justifique.

✔ Mantener el Firewall configurado correctamente.

✔ Si se cambia el puerto, actualizar también SELinux.

✔ Supervisar los registros del servicio.

---

# Errores comunes

## Reiniciar SSH sin validar la configuración

Siempre ejecutar:

```bash
sudo sshd -t
```

antes de:

```bash
systemctl restart sshd
```

---

## Cambiar el puerto y olvidar Firewalld

Debe abrirse el nuevo puerto:

```bash
firewall-cmd --permanent --add-port=2222/tcp
```

---

## Cambiar el puerto y olvidar SELinux

Debe registrarse:

```bash
sudo semanage port -a -t ssh_port_t -p tcp 2222
```

---

## Bloquear todos los usuarios

Configurar incorrectamente `AllowUsers` puede impedir cualquier acceso.

Siempre mantén una sesión SSH abierta mientras pruebas los cambios.

---

## Reiniciar el servicio desde una única sesión remota

Si existe un error de configuración, podrías perder el acceso al servidor.

Mantén una segunda sesión SSH abierta hasta verificar que la nueva configuración funciona.

---

# Comandos importantes RHCSA

| Comando | Descripción |
|----------|-------------|
| `systemctl status sshd` | Estado del servicio |
| `systemctl restart sshd` | Reiniciar SSH |
| `systemctl reload sshd` | Recargar configuración |
| `sshd -t` | Verificar sintaxis |
| `sshd -T` | Mostrar configuración efectiva |
| `ss -tulpn \| grep ssh` | Verificar puerto en escucha |
| `ssh usuario@host` | Conectarse por SSH |
| `journalctl -u sshd` | Ver registros |
| `firewall-cmd --list-services` | Verificar Firewall |
| `semanage port -l \| grep ssh` | Consultar puertos permitidos por SELinux |

---

# Resumen

En esta lección aprendiste a:

- Comprender el funcionamiento de OpenSSH.
- Administrar el servicio `sshd`.
- Configurar el archivo `sshd_config`.
- Modificar los principales parámetros del servidor SSH.
- Validar la configuración antes de reiniciar.
- Integrar OpenSSH con Firewalld y SELinux.
- Aplicar buenas prácticas para proteger el acceso remoto.

---

# Laboratorio práctico RHCSA

## Escenario 1

Verifica que OpenSSH Server está instalado y que el servicio está activo.

```bash
rpm -q openssh-server

systemctl status sshd
```

---

## Escenario 2

Consulta la configuración efectiva del servidor.

```bash
sudo sshd -T
```

Identifica:

- Puerto.
- PermitRootLogin.
- PasswordAuthentication.

---

## Escenario 3

Edita el archivo:

```bash
sudo vi /etc/ssh/sshd_config
```

Configura:

```text
PermitRootLogin no

MaxAuthTries 3
```

Verifica la sintaxis:

```bash
sudo sshd -t
```

Reinicia el servicio.

---

## Escenario 4

Consulta los registros recientes del servicio.

```bash
sudo journalctl -u sshd -n 20
```

Observa intentos de conexión o errores.

---

## Escenario 5

Verifica que el puerto SSH está escuchando y que el Firewall permite el acceso.

```bash
sudo ss -tulpn | grep ssh

firewall-cmd --list-services
```

Si cambiaste el puerto, confirma que también aparece en SELinux:

```bash
sudo semanage port -l | grep ssh
```

> **Objetivo general:** aprender a configurar y administrar **OpenSSH** de forma segura en Red Hat Enterprise Linux, integrándolo correctamente con **Firewalld** y **SELinux**. Estas habilidades son fundamentales para el examen **RHCSA** y para la administración diaria de servidores Linux en entornos empresariales.