# 41. Modos de Operación de SELinux

> **Módulo 7: Seguridad del Sistema**  
> **Página 41 del Manual RHCSA**

---

# Objetivos de aprendizaje

Al finalizar esta lección serás capaz de:

- Comprender los tres modos de funcionamiento de SELinux.
- Identificar el modo actual del sistema.
- Cambiar temporalmente entre modos.
- Configurar el modo permanente de SELinux.
- Comprender cuándo utilizar cada modo.
- Aplicar buenas prácticas durante la administración de servidores Red Hat Enterprise Linux.

---

# Introducción

SELinux puede operar en **tres modos diferentes**.

Cada modo determina cómo se aplican las políticas de seguridad sobre los procesos y recursos del sistema.

Elegir el modo adecuado es fundamental para mantener un equilibrio entre:

- Seguridad
- Diagnóstico
- Administración
- Disponibilidad del servicio

En producción, la recomendación de Red Hat es mantener SELinux en **Enforcing**.

---

# Los tres modos de SELinux

```
             SELinux
                 │
     ┌───────────┼───────────┐
     │           │           │
 Enforcing   Permissive   Disabled
```

---

# 1. Enforcing

Es el modo predeterminado en Red Hat Enterprise Linux.

Características:

- Aplica todas las políticas.
- Bloquea accesos no autorizados.
- Registra los eventos en los logs.
- Es el modo recomendado para producción.

```
Proceso

↓

Política SELinux

↓

Permitido

↓

Acceso

o

↓

Denegado
```

---

## Ejemplo

Apache intenta acceder a un directorio no autorizado.

Resultado:

```
Acceso Denegado
```

El evento queda registrado en el sistema.

---

# 2. Permissive

En este modo:

- No bloquea accesos.
- Registra todas las violaciones.
- Permite analizar problemas.
- Es ideal para pruebas y diagnóstico.

```
Proceso

↓

Violación de política

↓

Se registra

↓

Pero el acceso continúa
```

---

## ¿Cuándo utilizar Permissive?

- Durante migraciones.
- Para investigar errores.
- Antes de crear políticas.
- En laboratorios.
- Durante pruebas de aplicaciones.

No debe permanecer activo permanentemente en un servidor de producción.

---

# 3. Disabled

SELinux queda completamente deshabilitado.

Consecuencias:

- No se aplican políticas.
- No existen contextos activos.
- No se generan eventos de SELinux.
- Se pierde una importante capa de seguridad.

```
Aplicación

↓

Sistema de permisos Linux

↓

Acceso
```

---

## ¿Cuándo utilizar Disabled?

Prácticamente nunca.

Solo podría justificarse en situaciones muy específicas, como:

- Compatibilidad con software antiguo.
- Laboratorios temporales.
- Entornos de prueba controlados.

Red Hat recomienda mantener SELinux habilitado.

---

# Comparación de los modos

| Característica | Enforcing | Permissive | Disabled |
|----------------|-----------|------------|-----------|
| Aplica políticas | ✔ | ✘ | ✘ |
| Bloquea accesos | ✔ | ✘ | ✘ |
| Registra eventos | ✔ | ✔ | ✘ |
| Recomendado para producción | ✔ | ✘ | ✘ |

---

# Consultar el modo actual

```bash
getenforce
```

Ejemplo:

```
Enforcing
```

---

# Consultar información completa

```bash
sestatus
```

Ejemplo:

```
SELinux status: enabled

Current mode: enforcing

Mode from config file: enforcing

Policy: targeted
```

---

# Cambiar temporalmente a Permissive

```bash
sudo setenforce 0
```

Comprobar:

```bash
getenforce
```

Resultado:

```
Permissive
```

---

# Volver a Enforcing

```bash
sudo setenforce 1
```

Verificar:

```bash
getenforce
```

Resultado:

```
Enforcing
```

---

# ¿Qué hace setenforce?

| Comando | Resultado |
|----------|-----------|
| `setenforce 1` | Activa Enforcing |
| `setenforce 0` | Activa Permissive |

Este cambio es **temporal**.

Después de reiniciar el servidor se utilizará nuevamente la configuración del archivo:

```
/etc/selinux/config
```

---

# Cambiar el modo permanente

Editar:

```bash
sudo vi /etc/selinux/config
```

Ejemplo:

```text
SELINUX=enforcing
```

Opciones válidas:

```text
SELINUX=enforcing

SELINUX=permissive

SELINUX=disabled
```

Guardar el archivo.

---

# Aplicar el cambio permanente

Los cambios realizados en:

```
/etc/selinux/config
```

requieren reiniciar el sistema.

```bash
sudo reboot
```

---

# Verificar la configuración permanente

```bash
cat /etc/selinux/config
```

---

# Diferencia entre cambio temporal y permanente

## Temporal

```bash
sudo setenforce 0
```

- No requiere reinicio.
- Se pierde después del reinicio.

---

## Permanente

Modificar:

```text
/etc/selinux/config
```

- Requiere reinicio.
- Permanece después del reinicio.

---

# Flujo recomendado para diagnosticar un problema

```
Aplicación falla

↓

Verificar logs

↓

Cambiar temporalmente

a Permissive

↓

¿Funciona?

↓

Sí

↓

El problema está relacionado con SELinux

↓

Corregir contexto o política

↓

Volver a Enforcing
```

---

# Ejemplo práctico

Servidor Apache.

La página web devuelve error.

Paso 1

```bash
getenforce
```

Resultado:

```
Enforcing
```

Paso 2

Cambiar temporalmente.

```bash
sudo setenforce 0
```

Si el problema desaparece, probablemente se deba a un contexto SELinux incorrecto.

Una vez corregido:

```bash
sudo setenforce 1
```

---

# Verificar eventos registrados

```bash
sudo journalctl | grep AVC
```

También:

```bash
sudo ausearch -m AVC
```

Estos comandos se estudiarán en detalle en la lección de diagnóstico.

---

# Buenas prácticas RHCSA

✔ Mantener SELinux en **Enforcing** en servidores de producción.

✔ Utilizar **Permissive** únicamente para diagnóstico temporal.

✔ Evitar el modo **Disabled**.

✔ Después de las pruebas, volver siempre a **Enforcing**.

✔ Revisar los registros antes de modificar políticas.

✔ Documentar cualquier cambio en el modo de operación.

---

# Errores comunes

## Desactivar SELinux definitivamente

Muchos administradores utilizan:

```text
SELINUX=disabled
```

como solución rápida.

No es una buena práctica.

---

## Olvidar volver a Enforcing

Después de realizar pruebas:

```bash
sudo setenforce 1
```

---

## Modificar el archivo de configuración y esperar un cambio inmediato

El archivo:

```
/etc/selinux/config
```

solo se aplica tras reiniciar el sistema.

---

## Confundir setenforce con la configuración permanente

`setenforce` modifica únicamente el modo actual de ejecución.

---

# Comandos importantes RHCSA

| Comando | Descripción |
|----------|-------------|
| `getenforce` | Mostrar el modo actual |
| `sestatus` | Estado completo de SELinux |
| `setenforce 0` | Cambiar temporalmente a Permissive |
| `setenforce 1` | Cambiar temporalmente a Enforcing |
| `cat /etc/selinux/config` | Ver configuración permanente |
| `vi /etc/selinux/config` | Modificar configuración permanente |
| `journalctl \| grep AVC` | Consultar eventos SELinux |
| `ausearch -m AVC` | Buscar eventos de denegación |

---

# Resumen

En esta lección aprendiste a:

- Comprender los tres modos de SELinux.
- Diferenciar Enforcing, Permissive y Disabled.
- Cambiar temporalmente entre modos.
- Configurar el modo permanente.
- Aplicar buenas prácticas durante el diagnóstico de problemas.
- Mantener un entorno seguro utilizando SELinux correctamente.

---

# Laboratorio práctico RHCSA

## Escenario 1

Consulta el modo actual de SELinux.

```bash
getenforce
```

Luego obtén información completa.

```bash
sestatus
```

---

## Escenario 2

Cambia temporalmente a modo **Permissive**.

```bash
sudo setenforce 0
```

Verifica:

```bash
getenforce
```

Finalmente, vuelve a **Enforcing**.

```bash
sudo setenforce 1
```

---

## Escenario 3

Consulta la configuración permanente.

```bash
cat /etc/selinux/config
```

Identifica el valor de:

```text
SELINUX=
```

---

## Escenario 4

Simula una revisión de configuración editando el archivo:

```bash
sudo vi /etc/selinux/config
```

**No guardes cambios** si estás trabajando en un sistema de producción.

---

## Escenario 5

Consulta si existen eventos recientes relacionados con SELinux.

```bash
sudo journalctl | grep AVC
```

o

```bash
sudo ausearch -m AVC
```

> **Objetivo general:** dominar los modos de funcionamiento de **SELinux** y aprender a utilizarlos correctamente durante la administración y el diagnóstico de sistemas Red Hat Enterprise Linux. En la siguiente lección estudiarás los **Contextos de Seguridad (Security Contexts)**, uno de los conceptos más importantes de SELinux y del examen **RHCSA**.