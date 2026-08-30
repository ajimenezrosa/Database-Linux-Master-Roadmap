# 36. Fundamentos de Seguridad en Linux

> **Módulo 7: Seguridad del Sistema**  
> **Página 36 del Manual RHCSA**

---

# Objetivos de aprendizaje

Al finalizar esta lección serás capaz de:

- Comprender los principios fundamentales de la seguridad en Linux.
- Identificar las principales amenazas que afectan a un servidor.
- Conocer el modelo de seguridad de Red Hat Enterprise Linux.
- Aplicar buenas prácticas de endurecimiento (Hardening).
- Comprender el principio del menor privilegio.
- Identificar los principales componentes de seguridad del sistema.

---

# Introducción

La seguridad es uno de los pilares fundamentales de la administración de sistemas Linux.

Un servidor puede tener excelente hardware y aplicaciones perfectamente configuradas, pero si su seguridad es deficiente, toda la infraestructura estará en riesgo.

En Red Hat Enterprise Linux (RHEL), la seguridad está integrada en el sistema operativo mediante múltiples mecanismos como:

- Usuarios y grupos
- Permisos de archivos
- SELinux
- Firewalld
- OpenSSH
- PAM (Pluggable Authentication Modules)
- Auditoría del sistema
- Actualizaciones de seguridad

---

# ¿Qué es la seguridad informática?

La seguridad informática consiste en proteger:

- Los datos.
- Los servicios.
- Los usuarios.
- Los recursos del sistema.

Contra accesos no autorizados, modificaciones indebidas o interrupciones del servicio.

---

# La Triada CIA

Todo sistema de seguridad busca proteger tres principios fundamentales.

## Confidencialidad (Confidentiality)

Solo las personas autorizadas pueden acceder a la información.

Ejemplos:

- Contraseñas
- Permisos
- Cifrado

---

## Integridad (Integrity)

La información no debe modificarse sin autorización.

Ejemplos:

- Hash SHA256
- Firmas digitales
- Control de versiones

---

## Disponibilidad (Availability)

Los servicios deben permanecer disponibles cuando sean necesarios.

Ejemplos:

- Alta disponibilidad
- Backups
- Monitoreo
- UPS

---

# Capas de seguridad

La seguridad en Linux no depende de una única herramienta.

```
+--------------------------------------+
| Aplicaciones                         |
+--------------------------------------+
| Servicios (SSH, Apache, PostgreSQL)  |
+--------------------------------------+
| Firewalld                            |
+--------------------------------------+
| SELinux                              |
+--------------------------------------+
| Permisos Linux                       |
+--------------------------------------+
| Kernel                               |
+--------------------------------------+
| Hardware                             |
+--------------------------------------+
```

Cada capa añade protección adicional.

---

# Principio del menor privilegio

Uno de los principios más importantes de la seguridad.

**Cada usuario o servicio debe tener únicamente los permisos estrictamente necesarios para realizar su trabajo.**

Ejemplo correcto:

```
Usuario Web

↓

Acceso únicamente al directorio del sitio web
```

Ejemplo incorrecto:

```
Usuario Web

↓

Permisos sobre todo el sistema
```

---

# Superusuario (root)

El usuario **root** posee control absoluto sobre el sistema.

Puede:

- Crear usuarios.
- Eliminar archivos críticos.
- Modificar permisos.
- Instalar software.
- Detener servicios.

Por ello, su uso debe limitarse únicamente a tareas administrativas.

---

# Uso de sudo

En lugar de iniciar sesión directamente como **root**, se recomienda utilizar:

```bash
sudo
```

Ejemplo:

```bash
sudo dnf update
```

Ventajas:

- Registra quién ejecutó cada comando.
- Reduce el riesgo de errores.
- Mejora la auditoría.
- Permite delegar privilegios específicos.

---

# Mantener el sistema actualizado

Una de las mejores medidas de seguridad consiste en instalar las actualizaciones oficiales.

Buscar actualizaciones:

```bash
sudo dnf check-update
```

Actualizar el sistema:

```bash
sudo dnf upgrade
```

Actualizar únicamente paquetes de seguridad (cuando estén disponibles):

```bash
sudo dnf update --security
```

---

# Contraseñas seguras

Una contraseña segura debe:

- Tener al menos 12 caracteres.
- Combinar letras mayúsculas y minúsculas.
- Incluir números.
- Incluir símbolos.
- No contener información personal.

Ejemplo:

```
M1Servidor#2026!
```

---

# Bloqueo de cuentas

Linux permite bloquear temporalmente cuentas después de múltiples intentos fallidos de autenticación mediante PAM y herramientas como `faillock`.

Esto reduce los ataques de fuerza bruta.

---

# Deshabilitar servicios innecesarios

Cada servicio ejecutándose representa una posible superficie de ataque.

Ver servicios activos:

```bash
systemctl list-units --type=service
```

Deshabilitar un servicio:

```bash
sudo systemctl disable nombre-servicio
```

Detenerlo:

```bash
sudo systemctl stop nombre-servicio
```

---

# Minimizar software instalado

Instalar únicamente el software necesario.

Consultar paquetes instalados:

```bash
rpm -qa
```

Buscar un paquete:

```bash
rpm -qa | grep httpd
```

---

# Firewall

El Firewall controla qué conexiones pueden entrar o salir del servidor.

En RHEL se utiliza:

```
firewalld
```

Ejemplo:

```
Internet

↓

Firewall

↓

Servidor
```

El Firewall será estudiado en las siguientes lecciones.

---

# SELinux

SELinux añade una capa adicional de seguridad incluso cuando un proceso posee permisos tradicionales.

Permite restringir el comportamiento de los servicios.

Ejemplo:

```
Apache

↓

Solo puede acceder a determinados archivos.
```

---

# OpenSSH

SSH permite administrar servidores remotamente.

Buenas prácticas:

- Cambiar el puerto (opcional).
- Deshabilitar acceso directo de root.
- Utilizar autenticación mediante llaves.
- Limitar usuarios autorizados.

---

# Copias de seguridad

Una buena estrategia de seguridad siempre incluye respaldos.

Regla recomendada:

```
3 copias

↓

2 medios distintos

↓

1 fuera del sitio
```

Conocida como la estrategia **3-2-1**.

---

# Monitoreo

Es importante supervisar continuamente:

- CPU
- Memoria
- Disco
- Servicios
- Usuarios conectados
- Intentos fallidos de autenticación

Herramientas comunes:

```bash
top
```

```bash
htop
```

```bash
journalctl
```

---

# Registros (Logs)

Los registros permiten detectar problemas y posibles ataques.

Consultar el Journal:

```bash
journalctl
```

Ver eventos recientes:

```bash
journalctl -xe
```

---

# Usuarios conectados

Ver usuarios conectados:

```bash
who
```

Información adicional:

```bash
w
```

Últimos accesos:

```bash
last
```

---

# Procesos en ejecución

```bash
ps aux
```

Procesos con mayor consumo:

```bash
top
```

---

# Conexiones de red

```bash
ss -tuln
```

Con procesos asociados:

```bash
sudo ss -tulpn
```

---

# Principales amenazas

Entre las amenazas más comunes se encuentran:

- Fuerza bruta.
- Malware.
- Ransomware.
- Escalada de privilegios.
- Configuraciones incorrectas.
- Servicios vulnerables.
- Contraseñas débiles.
- Software desactualizado.

---

# Recomendaciones generales

✔ Mantener actualizado el sistema.

✔ Utilizar contraseñas robustas.

✔ Emplear `sudo` en lugar de iniciar sesión como root.

✔ Configurar correctamente el Firewall.

✔ Mantener SELinux en modo **Enforcing**.

✔ Instalar únicamente el software necesario.

✔ Realizar copias de seguridad periódicas.

✔ Supervisar los registros del sistema.

✔ Eliminar usuarios que ya no sean necesarios.

✔ Aplicar el principio del menor privilegio.

---

# Herramientas de seguridad más utilizadas

| Herramienta | Función |
|-------------|----------|
| `sudo` | Delegar privilegios administrativos |
| `firewalld` | Firewall del sistema |
| `SELinux` | Control obligatorio de acceso (MAC) |
| `OpenSSH` | Administración remota segura |
| `journalctl` | Consulta de registros |
| `ss` | Conexiones y puertos |
| `last` | Últimos accesos |
| `who` | Usuarios conectados |
| `w` | Actividad de usuarios |
| `top` | Procesos activos |
| `dnf` | Actualizaciones del sistema |

---

# Buenas prácticas RHCSA

✔ Mantener SELinux habilitado.

✔ Mantener Firewalld activo.

✔ No trabajar como **root** de forma permanente.

✔ Revisar periódicamente los registros.

✔ Limitar el acceso SSH.

✔ Mantener el sistema actualizado.

✔ Auditar los servicios abiertos.

✔ Documentar todos los cambios de seguridad.

---

# Errores comunes

## Deshabilitar SELinux

Muchos administradores lo hacen para "resolver" problemas.

No es una buena práctica.

Lo correcto es identificar y corregir el contexto o la política correspondiente.

---

## Desactivar el Firewall

No debe hacerse salvo en laboratorios o pruebas controladas.

---

## Utilizar siempre root

Incrementa el riesgo de errores y dificulta la auditoría.

---

## No instalar actualizaciones

Deja el sistema expuesto a vulnerabilidades conocidas.

---

## Permisos excesivos

Otorgar permisos innecesarios aumenta la superficie de ataque.

Aplicar siempre el principio del menor privilegio.

---

# Resumen

En esta lección aprendiste a:

- Comprender los principios fundamentales de la seguridad en Linux.
- Conocer la arquitectura de seguridad de Red Hat Enterprise Linux.
- Aplicar el principio del menor privilegio.
- Mantener el sistema actualizado.
- Utilizar `sudo` de forma segura.
- Identificar los principales componentes de seguridad del sistema.
- Aplicar buenas prácticas de endurecimiento para servidores Linux.

---

# Laboratorio práctico RHCSA

## Escenario 1

Consulta los usuarios conectados actualmente.

```bash
who
```

y

```bash
w
```

---

## Escenario 2

Comprueba si existen actualizaciones disponibles.

```bash
sudo dnf check-update
```

---

## Escenario 3

Lista todos los servicios activos.

```bash
systemctl list-units --type=service
```

Identifica aquellos que no sean necesarios en un servidor de laboratorio.

---

## Escenario 4

Consulta los puertos abiertos.

```bash
sudo ss -tulpn
```

Relaciona cada puerto con el servicio correspondiente.

---

## Escenario 5

Revisa los registros recientes del sistema.

```bash
journalctl -xe
```

Identifica mensajes relacionados con autenticación, servicios o errores.

> **Objetivo general:** comprender los fundamentos de la seguridad en Linux y adoptar buenas prácticas desde el inicio de la administración del sistema. Estos conceptos servirán como base para las siguientes lecciones dedicadas a **Firewalld**, **SELinux** y **OpenSSH**, componentes esenciales del examen **RHCSA**.