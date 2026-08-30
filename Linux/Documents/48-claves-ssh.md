# 48. Autenticación mediante Llaves SSH (SSH Keys)

> **Módulo 7: Seguridad del Sistema**  
> **Página 48 del Manual RHCSA**

---

# Objetivos de aprendizaje

Al finalizar esta lección serás capaz de:

- Comprender cómo funciona la autenticación mediante llaves SSH.
- Generar un par de llaves pública y privada.
- Copiar la llave pública al servidor.
- Conectarte sin utilizar contraseña.
- Administrar las llaves autorizadas.
- Aplicar buenas prácticas para proteger las llaves privadas.

---

# Introducción

Aunque SSH permite autenticarse mediante usuario y contraseña, la forma más segura y recomendada es utilizar **llaves criptográficas**.

La autenticación mediante llaves ofrece múltiples ventajas:

- Mayor seguridad.
- Protección frente a ataques de fuerza bruta.
- Inicio de sesión más rápido.
- Automatización de tareas.
- Acceso seguro para scripts y herramientas de administración.

En la mayoría de los entornos empresariales, el acceso mediante llaves SSH es el método preferido.

---

# ¿Cómo funciona?

SSH utiliza un par de claves criptográficas:

- **Llave privada (Private Key)**
- **Llave pública (Public Key)**

La llave pública se copia al servidor.

La llave privada permanece únicamente en el equipo del usuario.

---

# Esquema de funcionamiento

```
Cliente

┌────────────────────┐
│ Llave privada      │
└────────────────────┘

          │

Conexión SSH

          │

          ▼

Servidor

┌────────────────────┐
│ Llave pública      │
└────────────────────┘

↓

Autenticación

↓

Acceso permitido
```

---

# Llave privada

La llave privada:

- Nunca debe compartirse.
- Debe permanecer únicamente en el equipo del usuario.
- Puede protegerse con una frase de paso (*passphrase*).

Ejemplo:

```
~/.ssh/id_ed25519
```

o

```
~/.ssh/id_rsa
```

---

# Llave pública

La llave pública puede copiarse libremente al servidor.

Ejemplo:

```
~/.ssh/id_ed25519.pub
```

---

# Algoritmos disponibles

| Algoritmo | Estado |
|------------|--------|
| Ed25519 | Recomendado |
| RSA 4096 | Muy utilizado |
| ECDSA | Disponible |
| DSA | Obsoleto |

En sistemas actuales se recomienda utilizar **Ed25519**.

---

# Generar una llave SSH

Generar una llave Ed25519:

```bash
ssh-keygen -t ed25519
```

Agregar un comentario:

```bash
ssh-keygen -t ed25519 -C "ajimenez@servidor"
```

---

# Generar una llave RSA

```bash
ssh-keygen -t rsa -b 4096
```

---

# Proceso de generación

Durante la creación aparecerán preguntas similares a:

```
Enter file in which to save the key:
```

Presionar **Enter** para utilizar la ubicación predeterminada.

Después:

```
Enter passphrase:
```

Puede configurarse una frase de paso para proteger la llave privada.

---

# Resultado

En el directorio:

```text
~/.ssh/
```

Se crearán dos archivos:

```
id_ed25519

id_ed25519.pub
```

---

# Ver las llaves

```bash
ls -l ~/.ssh
```

Ejemplo:

```
id_ed25519

id_ed25519.pub

known_hosts
```

---

# Copiar la llave al servidor

La forma más sencilla es:

```bash
ssh-copy-id usuario@servidor
```

Ejemplo:

```bash
ssh-copy-id ajimenez@192.168.1.50
```

---

# ¿Qué hace ssh-copy-id?

Este comando copia automáticamente la llave pública al archivo:

```text
~/.ssh/authorized_keys
```

del usuario remoto.

---

# Copia manual

Si `ssh-copy-id` no está disponible:

Mostrar la llave pública:

```bash
cat ~/.ssh/id_ed25519.pub
```

Copiar su contenido al archivo remoto:

```text
~/.ssh/authorized_keys
```

---

# Conectarse mediante llave

```bash
ssh usuario@servidor
```

Si la llave fue configurada correctamente, no será necesario introducir la contraseña del usuario (aunque podría solicitarse la *passphrase* de la llave privada si fue configurada).

---

# Archivo authorized_keys

Ubicación:

```text
~/.ssh/authorized_keys
```

Cada línea representa una llave pública autorizada.

Ejemplo:

```
ssh-ed25519 AAAAC3Nza...

ssh-rsa AAAAB3Nza...
```

---

# Archivo known_hosts

Ubicación:

```text
~/.ssh/known_hosts
```

Contiene la huella digital (*fingerprint*) de los servidores conocidos.

Permite detectar cambios inesperados en la identidad del servidor.

---

# Ver la huella de una llave

```bash
ssh-keygen -lf ~/.ssh/id_ed25519.pub
```

---

# Cambiar el nombre de una llave

```bash
ssh-keygen \
-t ed25519 \
-f ~/.ssh/servidor-produccion
```

Resultado:

```
servidor-produccion

servidor-produccion.pub
```

---

# Utilizar una llave específica

```bash
ssh -i ~/.ssh/servidor-produccion usuario@servidor
```

---

# Utilizar un agente SSH

Iniciar el agente:

```bash
eval "$(ssh-agent -s)"
```

Agregar la llave:

```bash
ssh-add ~/.ssh/id_ed25519
```

Ver las llaves cargadas:

```bash
ssh-add -l
```

El agente evita introducir la *passphrase* en cada conexión durante la sesión.

---

# Configuración del cliente SSH

Archivo:

```text
~/.ssh/config
```

Ejemplo:

```text
Host servidor

    HostName 192.168.1.50

    User ajimenez

    IdentityFile ~/.ssh/id_ed25519
```

Ahora basta con ejecutar:

```bash
ssh servidor
```

---

# Permisos correctos

Directorio:

```bash
chmod 700 ~/.ssh
```

Llave privada:

```bash
chmod 600 ~/.ssh/id_ed25519
```

Llave pública:

```bash
chmod 644 ~/.ssh/id_ed25519.pub
```

Archivo autorizado:

```bash
chmod 600 ~/.ssh/authorized_keys
```

---

# Flujo de autenticación

```
Cliente

↓

Llave privada

↓

Servidor

↓

authorized_keys

↓

Comparación

↓

Acceso permitido
```

---

# Integración con OpenSSH

En el servidor:

```text
PubkeyAuthentication yes
```

Si únicamente se utilizarán llaves:

```text
PasswordAuthentication no
```

Antes de deshabilitar la autenticación por contraseña, verifica que el acceso mediante llaves funciona correctamente.

---

# Herramientas relacionadas

| Herramienta | Función |
|-------------|----------|
| `ssh-keygen` | Generar llaves |
| `ssh-copy-id` | Copiar llave pública |
| `ssh-add` | Agregar llaves al agente |
| `ssh-agent` | Administrar llaves en memoria |
| `ssh` | Cliente SSH |
| `scp` | Copia segura |
| `sftp` | Transferencia segura |

---

# Buenas prácticas RHCSA

✔ Utilizar **Ed25519** siempre que sea posible.

✔ Proteger la llave privada con una *passphrase*.

✔ Nunca compartir la llave privada.

✔ Utilizar permisos correctos en el directorio `.ssh`.

✔ Eliminar las llaves que ya no sean necesarias.

✔ Utilizar un archivo `~/.ssh/config` para simplificar conexiones frecuentes.

✔ Revisar periódicamente el archivo `authorized_keys`.

---

# Errores comunes

## Compartir la llave privada

Nunca debe enviarse por correo ni copiarse a otros usuarios.

---

## Permisos incorrectos

Si los permisos son demasiado abiertos, OpenSSH rechazará la autenticación.

Ejemplo:

```bash
chmod 777 ~/.ssh
```

Esto provocará errores de seguridad.

---

## Copiar la llave privada al servidor

Solo debe copiarse la **llave pública**.

---

## Deshabilitar PasswordAuthentication demasiado pronto

Comprueba primero que el acceso mediante llaves funciona correctamente.

---

## Ignorar known_hosts

Si la huella del servidor cambia inesperadamente, investiga la causa antes de aceptarla.

---

# Comandos importantes RHCSA

| Comando | Descripción |
|----------|-------------|
| `ssh-keygen -t ed25519` | Generar una llave Ed25519 |
| `ssh-keygen -t rsa -b 4096` | Generar una llave RSA |
| `ssh-copy-id usuario@host` | Copiar la llave pública |
| `ssh usuario@host` | Conectarse mediante SSH |
| `ssh -i archivo usuario@host` | Usar una llave específica |
| `ssh-add` | Agregar llaves al agente |
| `ssh-add -l` | Listar llaves cargadas |
| `ssh-keygen -lf archivo.pub` | Mostrar la huella digital |

---

# Resumen

En esta lección aprendiste a:

- Comprender el funcionamiento de la autenticación mediante llaves SSH.
- Generar llaves públicas y privadas.
- Copiar llaves al servidor utilizando `ssh-copy-id`.
- Administrar el archivo `authorized_keys`.
- Utilizar el agente SSH y el archivo `~/.ssh/config`.
- Aplicar buenas prácticas para proteger las llaves privadas.

---

# Laboratorio práctico RHCSA

## Escenario 1

Genera un nuevo par de llaves Ed25519.

```bash
ssh-keygen -t ed25519
```

Comprueba que se crearon los archivos:

```bash
ls -l ~/.ssh
```

---

## Escenario 2

Copia la llave pública a otro servidor.

```bash
ssh-copy-id usuario@servidor
```

Verifica que puedes iniciar sesión sin introducir la contraseña del usuario.

---

## Escenario 3

Protege correctamente el directorio `.ssh` y sus archivos.

```bash
chmod 700 ~/.ssh

chmod 600 ~/.ssh/id_ed25519

chmod 644 ~/.ssh/id_ed25519.pub

chmod 600 ~/.ssh/authorized_keys
```

---

## Escenario 4

Configura un alias para un servidor.

Edita:

```text
~/.ssh/config
```

Agrega:

```text
Host laboratorio

    HostName 192.168.1.50

    User ajimenez

    IdentityFile ~/.ssh/id_ed25519
```

Conéctate utilizando:

```bash
ssh laboratorio
```

---

## Escenario 5

Inicia el agente SSH, agrega tu llave y verifica que quedó cargada.

```bash
eval "$(ssh-agent -s)"

ssh-add ~/.ssh/id_ed25519

ssh-add -l
```

> **Objetivo general:** dominar la autenticación mediante **llaves SSH**, implementando conexiones seguras y eficientes entre clientes y servidores Linux. Esta técnica es el estándar en entornos empresariales y constituye una competencia esencial para el examen **RHCSA** y para la administración profesional de sistemas Red Hat Enterprise Linux.