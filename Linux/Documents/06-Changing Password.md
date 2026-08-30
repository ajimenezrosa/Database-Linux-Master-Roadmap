# 6. Administración y Cambio de Contraseñas con `passwd`

![Administrador Linux cambiando contraseñas desde la terminal](images/06-passwd-portada.jpg)

> **Objetivo del capítulo**
>
> Aprender a cambiar contraseñas de usuarios en Linux utilizando el comando `passwd`, comprender la diferencia entre cambiar la contraseña propia y la de otros usuarios, conocer las políticas de seguridad y aplicar buenas prácticas para la administración de credenciales.

---

# Introducción

Las contraseñas son el primer mecanismo de autenticación en la mayoría de los sistemas Linux.

Todo usuario debe proteger su cuenta mediante una contraseña segura, mientras que los administradores del sistema deben conocer cómo:

- Cambiar contraseñas.
- Restablecer contraseñas olvidadas.
- Obligar a un usuario a cambiar su contraseña.
- Bloquear y desbloquear cuentas.
- Verificar el estado de una contraseña.

En Linux, la herramienta principal para realizar estas tareas es el comando **`passwd`**.

---

# ¿Qué es `passwd`?

El comando **`passwd`** permite administrar las contraseñas de los usuarios.

Su sintaxis básica es:

```bash
passwd [usuario]
```

Es importante recordar que el comando es:

```text
passwd
```

y **no**:

```text
password
```

Muchos principiantes cometen este error.

---

# Cambiar la contraseña del usuario actual

Si deseas cambiar tu propia contraseña, simplemente ejecuta:

```bash
passwd
```

El sistema solicitará:

1. Contraseña actual.
2. Nueva contraseña.
3. Confirmación de la nueva contraseña.

Ejemplo:

```text
Changing password for user ajimenez.

Current password:
New password:
Retype new password:

passwd: all authentication tokens updated successfully
```

---

# ¿Por qué solicita dos veces la nueva contraseña?

Linux solicita la contraseña dos veces para verificar que no existan errores de escritura.

Si ambas coinciden, la contraseña se actualiza correctamente.

---

# Cambiar la contraseña de otro usuario

Solo el usuario **root** o un usuario con privilegios mediante `sudo` puede modificar la contraseña de otra cuenta.

Ejemplo:

```bash
sudo passwd maria
```

o como root:

```bash
passwd maria
```

El sistema solicitará únicamente la nueva contraseña.

No pedirá la contraseña anterior del usuario.

---

# Ejemplo práctico

Supongamos que el administrador necesita cambiar la contraseña del usuario `juan`.

```bash
sudo passwd juan
```

Salida:

```text
Changing password for user juan

New password:
Retype new password:

passwd: all authentication tokens updated successfully
```

---

# ¿Qué ocurre si un usuario intenta cambiar la contraseña de otra cuenta?

Ejemplo:

```bash
passwd maria
```

Si el usuario actual no tiene privilegios administrativos aparecerá un mensaje similar a:

```text
passwd: Only root can specify a user name.
```

Esto protege las cuentas del sistema contra modificaciones no autorizadas.

---

# Políticas de contraseñas

![Política de seguridad de contraseñas](images/06-password-policy.jpg)

Linux puede aplicar reglas para garantizar contraseñas seguras.

Dependiendo de la configuración del sistema, una contraseña puede ser rechazada si:

- Es demasiado corta.
- Contiene palabras del diccionario.
- Es demasiado simple.
- Es similar a la contraseña anterior.
- No contiene suficientes caracteres diferentes.

Por ejemplo:

```text
BAD PASSWORD: is too short
```

o

```text
BAD PASSWORD: is based on a dictionary word
```

---

# ¿Quién controla estas reglas?

En la mayoría de las distribuciones modernas estas políticas son gestionadas mediante **PAM (Pluggable Authentication Modules)**.

PAM permite configurar requisitos como:

- Longitud mínima.
- Complejidad.
- Tiempo de expiración.
- Historial de contraseñas.
- Número máximo de intentos.

---

# Recomendaciones para una contraseña segura

Una buena contraseña debería contener:

- Al menos 12 caracteres.
- Letras mayúsculas.
- Letras minúsculas.
- Números.
- Símbolos especiales.

Ejemplo:

```text
Rhcsa@2026_Admin!
```

Evita utilizar:

- 12345678
- password
- admin
- qwerty
- fechas de nacimiento
- nombres propios

---

# Forzar el cambio de contraseña

Un administrador puede obligar a un usuario a cambiar su contraseña en el próximo inicio de sesión.

```bash
sudo passwd --expire juan
```

La próxima vez que el usuario inicie sesión aparecerá un mensaje indicando que debe establecer una nueva contraseña.

---

# Bloquear una contraseña

Para impedir temporalmente el acceso mediante contraseña:

```bash
sudo passwd -l juan
```

Salida:

```text
Locking password for user juan.
```

---

# Desbloquear una contraseña

```bash
sudo passwd -u juan
```

---

# Ver el estado de una contraseña

```bash
passwd -S juan
```

Ejemplo:

```text
juan PS 2026-07-24 0 90 7 14
```

La salida muestra información como:

- Estado de la contraseña.
- Fecha del último cambio.
- Días mínimos entre cambios.
- Tiempo máximo de validez.
- Días de advertencia antes del vencimiento.

---

# Caducidad de contraseñas

Linux permite establecer políticas de expiración utilizando el comando `chage`.

Ejemplo:

```bash
sudo chage -l juan
```

Salida:

```text
Last password change
Password expires
Password inactive
Account expires
Minimum number of days
Maximum number of days
Warning period
```

Este tema se estudiará con mayor profundidad en capítulos posteriores.

---

# Archivos relacionados

Linux almacena la información de autenticación principalmente en:

| Archivo | Contenido |
|----------|-----------|
| `/etc/passwd` | Información básica de usuarios |
| `/etc/shadow` | Contraseñas cifradas y políticas |
| `/etc/login.defs` | Configuración general de contraseñas |
| `/etc/pam.d/` | Configuración de PAM |

> **Importante:** Las contraseñas no se almacenan en texto plano, sino como hashes criptográficos.

---

# Buenas prácticas

- Cambia las contraseñas iniciales inmediatamente.
- No compartas credenciales.
- Utiliza contraseñas largas y complejas.
- Cambia las contraseñas periódicamente cuando la política de la organización lo requiera.
- Utiliza `sudo` en lugar de trabajar permanentemente como root.
- No escribas contraseñas en scripts o archivos de texto.

---

# Errores comunes

❌ Escribir `password` en lugar de `passwd`.

❌ Elegir contraseñas demasiado simples.

❌ Trabajar siempre como root.

❌ Compartir la contraseña del administrador.

❌ Utilizar la misma contraseña en varios sistemas.

---

# Laboratorio práctico

## Ejercicio 1

Consulta el usuario actual:

```bash
whoami
```

---

## Ejercicio 2

Cambia tu contraseña:

```bash
passwd
```

---

## Ejercicio 3

Como administrador, cambia la contraseña de otro usuario:

```bash
sudo passwd usuario
```

---

## Ejercicio 4

Consulta el estado de la contraseña:

```bash
passwd -S usuario
```

---

## Ejercicio 5

Fuerza el cambio de contraseña:

```bash
sudo passwd --expire usuario
```

---

## Ejercicio 6

Bloquea y desbloquea la contraseña de un usuario:

```bash
sudo passwd -l usuario
sudo passwd -u usuario
```

---

# Preguntas de repaso

1. ¿Para qué sirve el comando `passwd`?
2. ¿Cuál es la diferencia entre `passwd` y `passwd usuario`?
3. ¿Quién puede cambiar la contraseña de otro usuario?
4. ¿Por qué Linux solicita dos veces la nueva contraseña?
5. ¿Qué es PAM?
6. ¿Dónde se almacenan las contraseñas cifradas?
7. ¿Cómo bloquear una contraseña?
8. ¿Cómo desbloquear una contraseña?
9. ¿Cómo obligar a un usuario a cambiar su contraseña en el próximo inicio de sesión?
10. ¿Qué comando muestra el estado de una contraseña?

---

# Resumen

El comando **`passwd`** es la herramienta principal para administrar contraseñas en Linux.

Permite cambiar contraseñas, bloquear cuentas, desbloquearlas, consultar su estado y forzar cambios de contraseña.

Comprender su funcionamiento y las políticas de seguridad asociadas es una competencia esencial para cualquier administrador Linux y un conocimiento requerido para la certificación RHCSA.

---

# Próxima lección

➡ **7. Cambio de contraseña del usuario Root y uso de `sudo`**