# 47. Hardening de SSH (Asegurando el Acceso Remoto)

> **Módulo 7: Seguridad del Sistema**  
> **Página 47 del Manual RHCSA**

---

# Objetivos de aprendizaje

Al finalizar esta lección serás capaz de:

- Comprender qué es el Hardening de SSH.
- Aplicar buenas prácticas para proteger un servidor SSH.
- Reducir la superficie de ataque del servicio.
- Configurar restricciones de acceso.
- Mitigar ataques de fuerza bruta.
- Implementar una configuración recomendada para servidores de producción.

---

# Introducción

El servicio **SSH** suele ser el principal punto de acceso remoto a un servidor Linux.

Por esta razón, también es uno de los servicios más atacados en Internet.

Los ataques más comunes incluyen:

- Fuerza bruta.
- Robo de credenciales.
- Escaneo automático de puertos.
- Intentos masivos de autenticación.
- Uso de contraseñas débiles.

El **Hardening** consiste en aplicar una serie de medidas para minimizar estos riesgos sin afectar la administración del sistema.

---

# ¿Qué es Hardening?

Hardening significa:

```
Reducir la superficie de ataque
```

Consiste en deshabilitar funciones innecesarias y permitir únicamente aquello que realmente se necesita.

---

# Principios del Hardening

```
Menos servicios

↓

Menos usuarios

↓

Menos permisos

↓

Mayor seguridad
```

---

# Verificar la configuración actual

Antes de realizar cambios:

```bash
sudo sshd -T
```

Esto muestra la configuración efectiva del servidor.

---

# 1. Deshabilitar el acceso directo de root

No es recomendable permitir que el usuario **root** inicie sesión directamente.

En:

```text
/etc/ssh/sshd_config
```

Configurar:

```text
PermitRootLogin no
```

Después:

```bash
sudo sshd -t
sudo systemctl restart sshd
```

---

# ¿Por qué deshabilitar root?

En lugar de conectarse como root:

```
Internet

↓

root

↓

Servidor
```

Es preferible:

```
Internet

↓

Usuario normal

↓

sudo

↓

root
```

Esto mejora la trazabilidad y reduce el riesgo de comprometer la cuenta más privilegiada.

---

# 2. Limitar los usuarios permitidos

Permitir únicamente los usuarios autorizados.

```text
AllowUsers ajimenez administrador
```

---

También puede limitarse por grupos.

```text
AllowGroups wheel administradores
```

---

# 3. Bloquear usuarios específicos

```text
DenyUsers invitado prueba
```

---

# 4. Limitar intentos de autenticación

Reducir el número de intentos.

```text
MaxAuthTries 3
```

Con esto disminuye la eficacia de los ataques de fuerza bruta.

---

# 5. Reducir el tiempo de autenticación

```text
LoginGraceTime 30
```

El cliente dispone de 30 segundos para autenticarse.

---

# 6. Deshabilitar contraseñas (cuando se usan llaves)

Si todos los administradores utilizan autenticación mediante llaves públicas:

```text
PasswordAuthentication no
```

Mantener:

```text
PubkeyAuthentication yes
```

Esto elimina la posibilidad de ataques por contraseña.

> **Importante:** Verifica que las llaves públicas funcionan correctamente antes de deshabilitar la autenticación por contraseña.

---

# 7. Deshabilitar autenticación vacía

```text
PermitEmptyPasswords no
```

Nunca deben permitirse cuentas sin contraseña.

---

# 8. Deshabilitar autenticación por Challenge-Response

```text
ChallengeResponseAuthentication no
```

En versiones recientes puede aparecer como:

```text
KbdInteractiveAuthentication no
```

---

# 9. Deshabilitar reenvío innecesario

Si el servidor no utiliza túneles SSH:

```text
AllowTcpForwarding no
```

---

Deshabilitar X11 Forwarding cuando no sea necesario.

```text
X11Forwarding no
```

---

# 10. Limitar sesiones simultáneas

```text
MaxSessions 5
```

---

También:

```text
MaxStartups 5:30:10
```

Controla conexiones simultáneas antes de autenticarse.

---

# 11. Mantener conexiones activas

```text
ClientAliveInterval 300

ClientAliveCountMax 2
```

Las sesiones inactivas serán cerradas automáticamente.

---

# 12. Cambiar el puerto SSH

Ejemplo:

```text
Port 2222
```

Después:

```bash
sudo firewall-cmd \
--permanent \
--add-port=2222/tcp

sudo firewall-cmd --reload
```

Y actualizar SELinux:

```bash
sudo semanage port \
-a \
-t ssh_port_t \
-p tcp 2222
```

> **Nota:** Cambiar el puerto **no reemplaza otras medidas de seguridad**. Reduce parte del ruido generado por escáneres automáticos, pero no detiene ataques dirigidos.

---

# 13. Utilizar SSH versión 2

```text
Protocol 2
```

En las versiones actuales de OpenSSH únicamente se admite SSHv2, por lo que normalmente no es necesario configurarlo.

---

# 14. Mantener OpenSSH actualizado

Verificar actualizaciones:

```bash
sudo dnf check-update openssh-server
```

Actualizar:

```bash
sudo dnf update openssh-server
```

---

# 15. Supervisar los registros

Consultar los registros:

```bash
sudo journalctl -u sshd
```

Tiempo real:

```bash
sudo journalctl -fu sshd
```

---

# 16. Utilizar Fail2Ban (opcional)

Aunque no forma parte del examen RHCSA, **Fail2Ban** es una herramienta ampliamente utilizada para bloquear temporalmente direcciones IP que realizan múltiples intentos fallidos de autenticación.

---

# Ejemplo de configuración recomendada

```text
PermitRootLogin no

PasswordAuthentication no

PubkeyAuthentication yes

AllowUsers ajimenez

LoginGraceTime 30

MaxAuthTries 3

PermitEmptyPasswords no

AllowTcpForwarding no

X11Forwarding no

ClientAliveInterval 300

ClientAliveCountMax 2
```

---

# Validar antes de reiniciar

Siempre comprobar la sintaxis:

```bash
sudo sshd -t
```

Si no aparecen errores:

```bash
sudo systemctl restart sshd
```

---

# Flujo recomendado

```
Editar sshd_config

↓

Validar con sshd -t

↓

Reiniciar sshd

↓

Probar conexión

↓

Cerrar sesión anterior
```

> **Nunca cierres tu sesión SSH actual hasta confirmar que puedes iniciar una nueva con la configuración actualizada.**

---

# Integración con Firewalld

Verificar:

```bash
firewall-cmd --list-services
```

Agregar SSH:

```bash
sudo firewall-cmd \
--permanent \
--add-service=ssh

sudo firewall-cmd --reload
```

---

# Integración con SELinux

Consultar puertos permitidos:

```bash
sudo semanage port -l | grep ssh
```

---

# Herramientas relacionadas

| Herramienta | Función |
|-------------|----------|
| `sshd -t` | Verificar sintaxis |
| `sshd -T` | Mostrar configuración efectiva |
| `systemctl` | Administrar el servicio |
| `journalctl` | Consultar registros |
| `firewall-cmd` | Configurar Firewalld |
| `semanage` | Configurar SELinux |
| `ssh` | Cliente SSH |
| `ss` | Verificar puertos en escucha |

---

# Buenas prácticas RHCSA

✔ Deshabilitar el acceso directo de root.

✔ Utilizar autenticación mediante llaves públicas siempre que sea posible.

✔ Limitar los usuarios autorizados.

✔ Verificar la sintaxis antes de reiniciar el servicio.

✔ Mantener OpenSSH actualizado.

✔ Supervisar periódicamente los registros de autenticación.

✔ Utilizar contraseñas robustas cuando la autenticación por contraseña esté habilitada.

✔ Mantener al menos una sesión SSH abierta mientras se prueban cambios.

---

# Errores comunes

## Deshabilitar PasswordAuthentication antes de configurar las llaves

Puede impedir el acceso remoto al servidor.

---

## Reiniciar sin validar la configuración

Siempre ejecutar:

```bash
sudo sshd -t
```

---

## Cambiar el puerto y olvidar Firewalld

Debe abrirse el nuevo puerto.

---

## Cambiar el puerto y olvidar SELinux

Debe añadirse el nuevo puerto con:

```bash
sudo semanage port -a -t ssh_port_t -p tcp <puerto>
```

---

## Bloquear accidentalmente todos los usuarios

Verifica cuidadosamente las directivas:

```text
AllowUsers

AllowGroups
```

---

# Comandos importantes RHCSA

| Comando | Descripción |
|----------|-------------|
| `sshd -T` | Mostrar configuración efectiva |
| `sshd -t` | Validar configuración |
| `systemctl restart sshd` | Reiniciar SSH |
| `journalctl -u sshd` | Consultar registros |
| `firewall-cmd --list-services` | Ver servicios permitidos |
| `semanage port -l \| grep ssh` | Consultar puertos SSH en SELinux |
| `ss -tulpn \| grep ssh` | Verificar puerto en escucha |

---

# Resumen

En esta lección aprendiste a:

- Aplicar medidas de Hardening sobre OpenSSH.
- Reducir la superficie de ataque del servicio SSH.
- Configurar restricciones de acceso para usuarios.
- Proteger el servidor frente a intentos de fuerza bruta.
- Integrar OpenSSH con Firewalld y SELinux.
- Validar la configuración antes de reiniciar el servicio.

---

# Laboratorio práctico RHCSA

## Escenario 1

Consulta la configuración efectiva del servidor.

```bash
sudo sshd -T
```

Identifica:

- Puerto.
- `PermitRootLogin`.
- `PasswordAuthentication`.
- `MaxAuthTries`.

---

## Escenario 2

Edita el archivo:

```bash
sudo vi /etc/ssh/sshd_config
```

Configura:

```text
PermitRootLogin no

MaxAuthTries 3

LoginGraceTime 30
```

Valida la sintaxis:

```bash
sudo sshd -t
```

Reinicia el servicio.

---

## Escenario 3

Restringe el acceso a un usuario específico.

```text
AllowUsers tu_usuario
```

Comprueba que puedes iniciar sesión con ese usuario.

---

## Escenario 4

Consulta los registros del servicio.

```bash
sudo journalctl -u sshd -n 30
```

Identifica conexiones exitosas y fallidas.

---

## Escenario 5

Comprueba que SSH está escuchando correctamente y que el Firewall y SELinux permiten el acceso.

```bash
sudo ss -tulpn | grep ssh

firewall-cmd --list-services

sudo semanage port -l | grep ssh
```

> **Objetivo general:** aplicar técnicas de **Hardening de SSH** para proteger el acceso remoto a servidores Red Hat Enterprise Linux. Estas prácticas incrementan significativamente la seguridad del sistema y representan conocimientos fundamentales para el examen **RHCSA** y para la administración de infraestructuras Linux en entornos empresariales.