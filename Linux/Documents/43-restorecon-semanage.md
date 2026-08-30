# 43. Gestión de Contextos con restorecon, chcon y semanage

> **Módulo 7: Seguridad del Sistema**  
> **Página 43 del Manual RHCSA**

---

# Objetivos de aprendizaje

Al finalizar esta lección serás capaz de:

- Comprender la diferencia entre `chcon`, `restorecon` y `semanage`.
- Modificar temporalmente el contexto de un archivo.
- Restaurar el contexto predeterminado de SELinux.
- Crear reglas permanentes para contextos personalizados.
- Verificar los cambios realizados.
- Aplicar correctamente estas herramientas durante la administración de servidores Red Hat Enterprise Linux.

---

# Introducción

Uno de los errores más frecuentes al administrar SELinux consiste en modificar únicamente los permisos de Linux cuando el problema realmente está relacionado con el **contexto de seguridad**.

Para administrar correctamente los contextos existen tres herramientas fundamentales:

| Herramienta | Uso |
|-------------|-----|
| `chcon` | Cambiar temporalmente un contexto |
| `restorecon` | Restaurar el contexto predeterminado |
| `semanage` | Crear cambios permanentes en la política SELinux |

Comprender cuándo utilizar cada una es esencial para el examen **RHCSA** y para la administración de servidores en producción.

---

# ¿Cuál herramienta debo utilizar?

```
¿Necesito cambiar un contexto?

        │
        ▼

¿Será un cambio temporal?

        │
     Sí ▼ No

     chcon

          ▼

¿Quiero volver al contexto original?

          │

          ▼

     restorecon

          │

¿Necesito que el cambio permanezca?

          │

          ▼

      semanage
```

---

# ¿Qué hace chcon?

`chcon` (**Change Context**) modifica el contexto SELinux de un archivo o directorio.

Ejemplo:

```bash
sudo chcon -t httpd_sys_content_t index.html
```

Consultar el resultado:

```bash
ls -Z index.html
```

---

# Problema de chcon

El cambio realizado por `chcon` **no es permanente**.

Si posteriormente se ejecuta:

```bash
sudo restorecon index.html
```

El contexto volverá al definido por la política SELinux.

Por esta razón, **chcon no debe utilizarse como solución permanente**.

---

# ¿Qué hace restorecon?

`restorecon` restaura el contexto correcto definido por la política de SELinux.

Ejemplo:

```bash
sudo restorecon index.html
```

---

# Restaurar un directorio completo

```bash
sudo restorecon -Rv /var/www/html
```

Opciones:

| Opción | Descripción |
|---------|-------------|
| `-R` | Recursivo |
| `-v` | Mostrar cambios realizados |

---

# Ejemplo práctico

Archivo copiado desde:

```
/home/admin/index.html
```

Contexto:

```
user_home_t
```

Apache espera:

```
httpd_sys_content_t
```

Ejecutar:

```bash
sudo restorecon -Rv /var/www/html
```

Resultado:

```
httpd_sys_content_t
```

---

# ¿Qué hace semanage?

`semanage` permite modificar la política SELinux sin editarla directamente.

Se utiliza para:

- Contextos permanentes.
- Puertos SELinux.
- Usuarios SELinux.
- Mapeos.
- Booleanos.

Es la herramienta recomendada cuando un cambio debe sobrevivir a un `restorecon`.

---

# Instalar semanage

En RHEL suele formar parte del paquete:

```bash
policycoreutils-python-utils
```

Instalación:

```bash
sudo dnf install policycoreutils-python-utils
```

Verificar:

```bash
semanage --help
```

---

# Crear un contexto permanente

Ejemplo:

```bash
sudo semanage fcontext \
-a \
-t httpd_sys_content_t \
"/web(/.*)?"
```

Este comando indica que todo el contenido de:

```
/web
```

debe utilizar el contexto:

```
httpd_sys_content_t
```

---

# Aplicar la nueva regla

Después de crear la regla:

```bash
sudo restorecon -Rv /web
```

Ahora el contexto queda correctamente asignado.

---

# Consultar reglas creadas

```bash
sudo semanage fcontext -l
```

Filtrar:

```bash
sudo semanage fcontext -l | grep /web
```

---

# Modificar una regla existente

```bash
sudo semanage fcontext \
-m \
-t httpd_sys_rw_content_t \
"/web(/.*)?"
```

---

# Eliminar una regla

```bash
sudo semanage fcontext \
-d \
"/web(/.*)?"
```

Después:

```bash
sudo restorecon -Rv /web
```

---

# Diferencia entre chcon y semanage

## chcon

```
Archivo

↓

Cambio inmediato

↓

Temporal
```

---

## semanage

```
Política SELinux

↓

restorecon

↓

Cambio permanente
```

---

# Ejemplo completo

Crear un nuevo sitio web:

```bash
sudo mkdir /web
```

Crear un archivo:

```bash
echo "RHCSA" | sudo tee /web/index.html
```

Consultar el contexto:

```bash
ls -lZ /web
```

Crear la regla permanente:

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

Resultado esperado:

```
httpd_sys_content_t
```

---

# ¿Qué ocurre si uso solamente chcon?

```
chcon

↓

Funciona

↓

restorecon

↓

El cambio desaparece
```

---

# Flujo recomendado

```
Nuevo directorio

↓

Crear regla con semanage

↓

Ejecutar restorecon

↓

Verificar contexto

↓

Aplicación funciona
```

---

# Consultar diferencias

Antes:

```bash
ls -Z archivo
```

Después:

```bash
sudo restorecon archivo
```

Volver a consultar:

```bash
ls -Z archivo
```

---

# Restaurar todo un árbol de directorios

```bash
sudo restorecon -RFv /var/www
```

Opciones:

| Opción | Descripción |
|---------|-------------|
| `-R` | Recursivo |
| `-F` | Forzar restauración del contexto |
| `-v` | Mostrar detalles |

---

# Otros usos de semanage

Administrar puertos:

```bash
sudo semanage port -l
```

Administrar usuarios SELinux:

```bash
sudo semanage login -l
```

Administrar interfaces:

```bash
sudo semanage interface -l
```

Estos temas pertenecen a un nivel más avanzado, pero es útil conocer que `semanage` puede administrar mucho más que contextos de archivos.

---

# Herramientas relacionadas

| Herramienta | Función |
|-------------|---------|
| `ls -Z` | Consultar contextos |
| `chcon` | Cambiar contexto temporalmente |
| `restorecon` | Restaurar contexto predeterminado |
| `semanage fcontext` | Administrar contextos permanentes |
| `sestatus` | Estado de SELinux |

---

# Buenas prácticas RHCSA

✔ Utilizar `restorecon` antes de pensar que existe un problema con SELinux.

✔ Utilizar `semanage` para cualquier cambio permanente.

✔ Evitar depender de `chcon` en servidores de producción.

✔ Verificar siempre los contextos con `ls -Z`.

✔ Documentar todas las reglas creadas con `semanage`.

✔ Aplicar `restorecon` después de crear una nueva regla.

---

# Errores comunes

## Usar únicamente chcon

El cambio desaparecerá después de ejecutar:

```bash
restorecon
```

---

## Olvidar ejecutar restorecon

`semanage` registra la regla, pero no modifica automáticamente los archivos existentes.

Siempre ejecutar:

```bash
restorecon
```

---

## Cambiar permisos cuando el problema es el contexto

Modificar:

```bash
chmod 777
```

no solucionará un contexto SELinux incorrecto.

---

## Crear reglas demasiado amplias

Evita aplicar contextos a directorios innecesarios.

Define reglas únicamente para las rutas requeridas.

---

# Comandos importantes RHCSA

| Comando | Descripción |
|----------|-------------|
| `ls -Z` | Ver contexto |
| `chcon -t` | Cambiar contexto temporal |
| `restorecon` | Restaurar contexto |
| `restorecon -Rv` | Restaurar directorio recursivamente |
| `semanage fcontext -a` | Crear regla permanente |
| `semanage fcontext -m` | Modificar regla |
| `semanage fcontext -d` | Eliminar regla |
| `semanage fcontext -l` | Listar reglas |

---

# Resumen

En esta lección aprendiste a:

- Diferenciar entre `chcon`, `restorecon` y `semanage`.
- Modificar contextos temporalmente.
- Restaurar contextos predeterminados.
- Crear reglas permanentes para nuevos directorios.
- Aplicar correctamente los cambios mediante `restorecon`.
- Utilizar las herramientas recomendadas por Red Hat para administrar SELinux.

---

# Laboratorio práctico RHCSA

## Escenario 1

Crea un nuevo directorio para un sitio web.

```bash
sudo mkdir /web
```

Consulta su contexto:

```bash
ls -ldZ /web
```

---

## Escenario 2

Modifica temporalmente el contexto.

```bash
sudo chcon -t httpd_sys_content_t /web
```

Verifica:

```bash
ls -ldZ /web
```

---

## Escenario 3

Restaura el contexto original.

```bash
sudo restorecon -v /web
```

Comprueba el cambio.

---

## Escenario 4

Crea una regla permanente.

```bash
sudo semanage fcontext \
-a \
-t httpd_sys_content_t \
"/web(/.*)?"
```

Aplica la política:

```bash
sudo restorecon -Rv /web
```

---

## Escenario 5

Verifica que la regla quedó registrada.

```bash
sudo semanage fcontext -l | grep /web
```

Finalmente, comprueba nuevamente el contexto del directorio:

```bash
ls -ldZ /web
```

> **Objetivo general:** dominar el uso de **`restorecon`**, **`chcon`** y **`semanage`** para administrar correctamente los contextos de seguridad en SELinux. Estas herramientas son esenciales para solucionar problemas de acceso y configurar aplicaciones como Apache, Nginx, PostgreSQL y otros servicios en Red Hat Enterprise Linux, además de ser un tema recurrente en el examen **RHCSA**.