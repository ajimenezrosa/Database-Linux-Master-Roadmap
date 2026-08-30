# 85. Templates y Jinja2 en Ansible (Fase 1)

> **Módulo 11 – Automatización con Ansible**
>
> **Archivo:** `85-templates-jinja2.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Comprender qué son los Templates.
- Comprender qué es Jinja2.
- Diferenciar un archivo estático de un Template.
- Utilizar Variables dentro de un Template.
- Comprender el funcionamiento del módulo `template`.
- Construir archivos de configuración dinámicos.
- Aplicar buenas prácticas para la administración de configuraciones.

---

# Introducción

Hasta este momento hemos aprendido a:

- Crear Playbooks.
- Utilizar Variables.
- Trabajar con Facts.
- Crear Roles.

Sin embargo, aún existe un problema.

Supongamos que debemos configurar Apache en 300 servidores.

Cada servidor necesita un archivo:

```text
/etc/httpd/conf/httpd.conf
```

Pero:

Servidor 1

```text
Puerto 80
```

Servidor 2

```text
Puerto 8080
```

Servidor 3

```text
Puerto 9090
```

Servidor 4

```text
Puerto 8081
```

¿Deberíamos crear cuatro archivos diferentes?

No.

Para resolver este problema utilizamos:

# Templates

---

# ¿Qué es un Template?

Un Template es un archivo que contiene Variables.

Durante la ejecución del Playbook, Ansible reemplaza esas Variables por valores reales.

---

# Arquitectura

```text
Template

↓

Variables

↓

Jinja2

↓

Archivo Final

↓

Servidor
```

---

# Analogía

Imaginemos una carta.

En lugar de escribir cien cartas distintas.

Escribimos una plantilla.

```text
Estimado

{{ nombre }}

Bienvenido a la empresa.
```

---

Resultado.

Servidor A.

```text
Estimado Alejandro

Bienvenido a la empresa.
```

Servidor B.

```text
Estimado Carlos

Bienvenido a la empresa.
```

El mismo documento.

Diferentes resultados.

---

# Problema sin Templates

Servidor 1.

```text
httpd80.conf
```

Servidor 2.

```text
httpd8080.conf
```

Servidor 3.

```text
httpd9090.conf
```

Decenas de archivos diferentes.

---

# Solución

```text
httpd.conf.j2

↓

Variables

↓

Configuración dinámica
```

---

# ¿Qué es Jinja2?

Jinja2 es el motor de plantillas utilizado por Ansible.

Su función consiste en reemplazar Variables.

---

Arquitectura.

```text
Variables

↓

Jinja2

↓

Texto

↓

Archivo
```

---

# Extensión

Los Templates normalmente utilizan:

```text
.j2
```

Ejemplo.

```text
httpd.conf.j2
```

---

Otros ejemplos.

```text
nginx.conf.j2

postgresql.conf.j2

sshd_config.j2

hosts.j2

motd.j2
```

---

# Ubicación

Dentro de un Role.

```text
templates/

└── httpd.conf.j2
```

---

Proyecto.

```text
roles/

└── apache/

      └── templates/

             └── httpd.conf.j2
```

---

# Sintaxis

Las Variables utilizan:

```text
{{ variable }}
```

---

Ejemplo.

```text
Listen {{ http_port }}
```

---

Otra Variable.

```text
ServerAdmin {{ admin_email }}
```

---

Resultado.

```text
Listen 8080

ServerAdmin admin@empresa.com
```

---

# Flujo

```text
Variable

↓

Jinja2

↓

Reemplazo

↓

Archivo
```

---

# Módulo template

El encargado de copiar un Template al servidor.

---

Sintaxis.

```yaml
template:

  src:

  dest:
```

---

Ejemplo.

```yaml
- name: Copiar configuración

  template:

    src: httpd.conf.j2

    dest: /etc/httpd/conf/httpd.conf
```

---

Representación.

```text
Template

↓

Procesamiento

↓

Archivo

↓

Servidor
```

---

# Diferencia entre copy y template

## copy

```text
Archivo

↓

Se copia exactamente igual.
```

---

## template

```text
Archivo

↓

Variables

↓

Reemplazo

↓

Nuevo archivo
```

---

# Comparación

| copy | template |
|-------|----------|
| Archivo idéntico | Archivo dinámico |
| No procesa Variables | Procesa Variables |
| Copia texto | Genera configuración |
| Ideal para imágenes | Ideal para configuraciones |

---

# Ejemplo

Archivo.

```text
motd
```

No necesita Variables.

Utilizamos:

```yaml
copy:
```

---

Archivo.

```text
httpd.conf
```

Necesita Variables.

Utilizamos:

```yaml
template:
```

---

# Ejemplo Completo

Template.

```text
Listen {{ http_port }}

ServerAdmin {{ admin_email }}

DocumentRoot {{ document_root }}
```

---

Variables.

```yaml
http_port: 8080

admin_email: admin@empresa.com

document_root: /var/www/html
```

---

Resultado.

```text
Listen 8080

ServerAdmin admin@empresa.com

DocumentRoot /var/www/html
```

---

# Arquitectura Completa

```text
Playbook

↓

Variables

↓

Template

↓

Jinja2

↓

Archivo

↓

Servidor
```

---

# Variables de Inventario

También pueden utilizarse.

```yaml
{{ inventory_hostname }}
```

---

Resultado.

```text
web01
```

---

# Facts

También.

```yaml
{{ ansible_hostname }}
```

---

```yaml
{{ ansible_distribution }}
```

---

```yaml
{{ ansible_default_ipv4.address }}
```

---

Ejemplo.

```text
Hostname:

{{ ansible_hostname }}

IP:

{{ ansible_default_ipv4.address }}
```

---

Resultado.

```text
Hostname:

web01

IP:

192.168.1.15
```

---

# Variables Personalizadas

Ejemplo.

```yaml
usuario: administrador

empresa: Popular
```

---

Template.

```text
Usuario:

{{ usuario }}

Empresa:

{{ empresa }}
```

---

Resultado.

```text
Usuario:

administrador

Empresa:

Popular
```

---

# Flujo Empresarial

```text
Variables

↓

Templates

↓

Configuración

↓

Servidor

↓

Servicio
```

---

# Caso Real

Una empresa administra.

```text
400 Servidores Apache
```

Todos utilizan.

```text
httpd.conf.j2
```

Pero cada servidor recibe.

- Puerto diferente.
- IP diferente.
- Hostname diferente.
- DocumentRoot diferente.

Todo generado automáticamente.

---

# Beneficios

- No duplicar archivos.
- Configuración centralizada.
- Fácil mantenimiento.
- Automatización.
- Escalabilidad.
- Menor cantidad de errores.
- Configuración consistente.
- Reutilización.

---

# Organización Recomendada

```text
roles/

└── apache/

      ├── defaults

      ├── handlers

      ├── tasks

      ├── templates

      │      ├── httpd.conf.j2

      │      ├── virtualhost.j2

      │      └── index.html.j2

      └── vars
```

---

# Buenas Prácticas

- Utilizar Templates para archivos de configuración.
- Utilizar `copy` únicamente para archivos estáticos.
- Mantener Variables fuera del Template cuando sea posible.
- Utilizar nombres descriptivos.
- Centralizar Variables.
- Reutilizar Templates.
- Mantener un único propósito por Template.
- Validar la sintaxis antes de Producción.
- Documentar Variables utilizadas.
- Utilizar Roles para organizar Templates.

---

# Errores Comunes

## Error 1

Utilizar `copy` para un archivo que contiene Variables.

---

## Error 2

Olvidar la extensión:

```text
.j2
```

---

## Error 3

Escribir.

```text
{ variable }
```

En lugar de.

```text
{{ variable }}
```

---

## Error 4

Duplicar Templates.

---

## Error 5

Colocar lógica excesiva dentro del Template.

---

## Error 6

No utilizar Variables reutilizables.

---

## Error 7

Modificar manualmente el archivo generado.

---

## Error 8

No probar el resultado final.

---

## Error 9

Mezclar archivos estáticos con Templates.

---

## Error 10

No documentar las Variables requeridas.

---

# Laboratorio RHCSA

## Escenario

Una empresa administrará más de 300 servidores Apache utilizando un único Template.

---

## Laboratorio 1

Crear.

```text
httpd.conf.j2
```

---

## Laboratorio 2

Agregar Variables.

- Puerto.
- Administrador.
- DocumentRoot.

---

## Laboratorio 3

Crear un Playbook utilizando.

```yaml
template
```

---

## Laboratorio 4

Generar el archivo.

```text
/etc/httpd/conf/httpd.conf
```

---

## Laboratorio 5

Modificar únicamente el puerto.

Comprobar que el mismo Template genera un archivo diferente.

---

## Laboratorio 6

Mostrar Hostname e IP utilizando Facts.

---

## Laboratorio 7

Comparar el resultado utilizando:

```yaml
copy
```

y

```yaml
template
```

---

## Laboratorio 8

Crear un proyecto organizado mediante Roles donde todos los archivos de configuración utilicen Templates y todas las Variables provengan de `group_vars`.

---

# Preguntas de Repaso

1. ¿Qué es un Template?
2. ¿Qué función cumple Jinja2?
3. ¿Qué extensión utilizan normalmente los Templates?
4. ¿Cuál es la diferencia entre `copy` y `template`?
5. ¿Qué módulo procesa los Templates?
6. ¿Cómo se referencia una Variable en Jinja2?
7. ¿Qué ventajas ofrecen los Templates frente a múltiples archivos estáticos?
8. ¿Qué tipos de Variables pueden utilizarse dentro de un Template?
9. ¿Por qué es recomendable almacenar los Templates dentro de un Role?
10. ¿Cómo utilizarías un único Template para configurar cientos de servidores diferentes?

---

# Resumen

En esta primera fase aprendimos que los **Templates** permiten generar archivos de configuración dinámicos utilizando Variables, evitando la duplicación de archivos y facilitando la administración de grandes infraestructuras. Estudiamos el funcionamiento del motor **Jinja2**, la sintaxis básica para insertar Variables y el uso del módulo `template` para generar archivos personalizados en los servidores administrados.

También analizamos las diferencias entre `copy` y `template`, revisamos la organización recomendada dentro de un Role y comprendimos por qué los Templates son una herramienta esencial para administrar configuraciones de manera consistente, reutilizable y escalable.

En la **Fase 2** profundizaremos en la sintaxis de **Jinja2**, incluyendo filtros, operadores, expresiones, condicionales (`if`), bucles (`for`), comentarios, control de espacios en blanco y técnicas avanzadas para construir Templates profesionales.

------

# 85. Templates y Jinja2 en Ansible (Fase 2)

> **Módulo 11 – Automatización con Ansible**
>
> **Archivo:** `85-templates-jinja2.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Comprender la sintaxis de Jinja2.
- Utilizar expresiones.
- Aplicar operadores.
- Utilizar filtros.
- Crear condicionales.
- Crear ciclos (Loops).
- Utilizar comentarios.
- Controlar espacios en blanco.
- Construir Templates profesionales.

---

# Introducción

En la fase anterior aprendimos que un Template puede contener Variables.

Por ejemplo.

```text
Listen {{ http_port }}
```

Ahora veremos el verdadero poder de Jinja2.

Jinja2 no solamente reemplaza Variables.

También puede:

- Tomar decisiones.
- Realizar comparaciones.
- Ejecutar ciclos.
- Aplicar filtros.
- Transformar texto.
- Manipular listas.
- Construir configuraciones inteligentes.

---

# Sintaxis Básica

Jinja2 utiliza tres tipos principales de delimitadores.

| Sintaxis | Uso |
|----------|-----|
| `{{ }}` | Mostrar Variables o expresiones |
| `{% %}` | Ejecutar lógica |
| `{# #}` | Comentarios |

---

# Variables

```jinja2
{{ usuario }}
```

Resultado.

```text
Alejandro
```

---

# Expresiones

Jinja2 también permite realizar operaciones.

Ejemplo.

```jinja2
{{ 5 + 3 }}
```

Resultado.

```text
8
```

---

Otro ejemplo.

```jinja2
{{ 10 * 4 }}
```

Resultado.

```text
40
```

---

# Concatenación

```jinja2
{{ nombre }} {{ apellido }}
```

Resultado.

```text
Alejandro Jimenez
```

---

# Operadores

| Operador | Descripción |
|-----------|-------------|
| `+` | Suma |
| `-` | Resta |
| `*` | Multiplicación |
| `/` | División |
| `%` | Módulo |
| `==` | Igual |
| `!=` | Diferente |
| `>` | Mayor |
| `<` | Menor |
| `>=` | Mayor o igual |
| `<=` | Menor o igual |

---

# Comparaciones

```jinja2
{{ http_port == 80 }}
```

Resultado.

```text
True
```

---

# Lógica

Operadores.

```text
and

or

not
```

---

Ejemplo.

```jinja2
{{ puerto > 1024 and puerto < 65535 }}
```

---

# Condicionales

Una de las características más importantes.

Sintaxis.

```jinja2
{% if condición %}

...

{% endif %}
```

---

Ejemplo

```jinja2
{% if http_port == 80 %}

Puerto estándar

{% endif %}
```

---

Resultado.

```text
Puerto estándar
```

---

# Condicional if / else

```jinja2
{% if ssl %}

HTTPS

{% else %}

HTTP

{% endif %}
```

---

Resultado.

```text
HTTPS
```

---

# if / elif / else

```jinja2
{% if puerto == 80 %}

HTTP

{% elif puerto == 443 %}

HTTPS

{% else %}

Personalizado

{% endif %}
```

---

Representación.

```text
¿Puerto?

      │

      ├──80────HTTP

      │

      ├──443──HTTPS

      │

      └──────Personalizado
```

---

# Ejemplo Real

```jinja2
Listen {{ http_port }}

{% if ssl %}

SSLEngine On

{% endif %}
```

Resultado.

```text
Listen 443

SSLEngine On
```

---

# Ciclos (Loops)

Jinja2 también puede recorrer listas.

Sintaxis.

```jinja2
{% for item in lista %}

...

{% endfor %}
```

---

Ejemplo.

```jinja2
{% for usuario in usuarios %}

{{ usuario }}

{% endfor %}
```

Variables.

```yaml
usuarios:

  - admin

  - backup

  - oracle

  - postgres
```

---

Resultado.

```text
admin

backup

oracle

postgres
```

---

# Ejemplo Empresarial

```jinja2
AllowUsers

{% for usuario in usuarios %}

{{ usuario }}

{% endfor %}
```

Resultado.

```text
AllowUsers

admin

backup

oracle

postgres
```

---

# Diccionarios

Variables.

```yaml
usuario:

  nombre: Alejandro

  edad: 35

  pais: Dominicana
```

Template.

```jinja2
{{ usuario.nombre }}
```

Resultado.

```text
Alejandro
```

---

También.

```jinja2
{{ usuario.edad }}
```

---

# Acceso por Clave

También puede utilizarse.

```jinja2
{{ usuario["nombre"] }}
```

---

Muy utilizado cuando las claves contienen espacios o caracteres especiales.

---

# Filtros

Los filtros modifican la salida de una Variable.

Sintaxis.

```jinja2
{{ variable | filtro }}
```

---

Ejemplo.

```jinja2
{{ usuario | upper }}
```

Resultado.

```text
ALEJANDRO
```

---

Otro ejemplo.

```jinja2
{{ usuario | lower }}
```

Resultado.

```text
alejandro
```

---

# Filtros Más Utilizados

| Filtro | Función |
|----------|----------|
| upper | Mayúsculas |
| lower | Minúsculas |
| title | Primera letra en mayúscula |
| capitalize | Capitalizar |
| length | Longitud |
| trim | Eliminar espacios |
| default | Valor por defecto |
| replace | Reemplazar texto |
| join | Unir listas |
| sort | Ordenar |
| unique | Eliminar duplicados |
| first | Primer elemento |
| last | Último elemento |

---

# default

Muy importante.

```jinja2
{{ puerto | default(80) }}
```

Si no existe la Variable.

Resultado.

```text
80
```

---

# length

```jinja2
{{ usuarios | length }}
```

Resultado.

```text
4
```

---

# join

Variables.

```yaml
usuarios:

- admin

- backup

- oracle
```

Template.

```jinja2
{{ usuarios | join(', ') }}
```

Resultado.

```text
admin, backup, oracle
```

---

# replace

```jinja2
{{ empresa | replace(" ","_") }}
```

---

# title

```jinja2
{{ empresa | title }}
```

Resultado.

```text
Banco Popular
```

---

# Encadenamiento de Filtros

Pueden combinarse.

```jinja2
{{ usuario | lower | replace(" ","_") }}
```

---

Resultado.

```text
alejandro_jimenez
```

---

# Comentarios

No aparecen en el archivo final.

```jinja2
{#

Comentario interno

#}
```

---

Muy útiles para documentar.

---

# Espacios en Blanco

Jinja2 permite controlar espacios.

Normal.

```jinja2
{% if ssl %}
```

---

Eliminando espacios.

```jinja2
{%- if ssl -%}
```

---

Esto evita líneas vacías.

---

# Ejemplo

Sin control.

```text
Línea

(blank)

(blank)

Otra línea
```

---

Con control.

```text
Línea

Otra línea
```

---

# Variables Anidadas

```yaml
empresa:

  sede:

    ciudad: Santo Domingo
```

Template.

```jinja2
{{ empresa.sede.ciudad }}
```

Resultado.

```text
Santo Domingo
```

---

# Facts

También funcionan igual.

```jinja2
{{ ansible_hostname }}
```

---

```jinja2
{{ ansible_distribution }}
```

---

```jinja2
{{ ansible_memtotal_mb }}
```

---

```jinja2
{{ ansible_processor_vcpus }}
```

---

# Ejemplo Empresarial

```jinja2
Servidor:

{{ ansible_hostname }}

Sistema:

{{ ansible_distribution }}

RAM:

{{ ansible_memtotal_mb }}
```

---

# Template Profesional

```text
Variables

↓

Condicionales

↓

Loops

↓

Filtros

↓

Archivo

↓

Servidor
```

---

# Caso Real

Empresa.

600 servidores.

Cada servidor necesita un:

```text
sshd_config
```

Con.

- Usuarios distintos.
- Banner distinto.
- Puerto distinto.
- Configuración SSL distinta.

Todo generado automáticamente mediante un único Template.

---

# Beneficios

- Automatización.
- Reutilización.
- Configuración inteligente.
- Menor mantenimiento.
- Escalabilidad.
- Menor cantidad de errores.
- Configuración consistente.

---

# Buenas Prácticas

- Mantener la lógica sencilla.
- Utilizar filtros en lugar de lógica compleja cuando sea posible.
- Documentar los Templates.
- Utilizar `default`.
- Evitar Variables duplicadas.
- Utilizar nombres descriptivos.
- Mantener consistencia.
- Validar el resultado generado.
- Utilizar comentarios internos cuando agreguen valor.
- Mantener la lógica de negocio en Variables o Playbooks y no sobrecargar los Templates.

---

# Errores Comunes

## Error 1

Olvidar cerrar un bloque.

```jinja2
{% endif %}
```

---

## Error 2

Utilizar.

```text
{ variable }
```

En lugar de.

```text
{{ variable }}
```

---

## Error 3

No utilizar `default`.

---

## Error 4

Escribir demasiada lógica dentro del Template.

---

## Error 5

Duplicar código.

---

## Error 6

No probar los Loops.

---

## Error 7

No controlar espacios.

---

## Error 8

Olvidar documentar.

---

## Error 9

No validar Variables.

---

## Error 10

Confundir Variables de Ansible con Variables de Jinja2.

---

# Laboratorio RHCSA

## Escenario

Una empresa administrará más de 500 servidores Linux mediante Templates inteligentes.

---

## Laboratorio 1

Crear un Template utilizando Variables.

---

## Laboratorio 2

Agregar un bloque:

```jinja2
if
```

---

## Laboratorio 3

Agregar un:

```jinja2
for
```

que recorra una lista de usuarios.

---

## Laboratorio 4

Aplicar los filtros:

- upper
- lower
- title
- default

---

## Laboratorio 5

Crear un comentario interno.

---

## Laboratorio 6

Mostrar Hostname utilizando Facts.

---

## Laboratorio 7

Crear un Template que muestre información diferente según el sistema operativo del servidor (`ansible_distribution`) utilizando un bloque `if`/`elif`/`else`.

---

## Laboratorio 8

Diseñar un Template profesional para `sshd_config` que genere automáticamente diferentes configuraciones según el entorno (Desarrollo, QA o Producción), utilizando Variables, condicionales, filtros y ciclos.

---

# Preguntas de Repaso

1. ¿Qué delimitadores utiliza Jinja2?
2. ¿Cuál es la diferencia entre `{{ }}` y `{% %}`?
3. ¿Cómo se crea un condicional `if`?
4. ¿Cómo funciona un ciclo `for`?
5. ¿Qué son los filtros?
6. ¿Qué función cumple `default`?
7. ¿Cómo se accede a un diccionario?
8. ¿Para qué sirven los comentarios de Jinja2?
9. ¿Cómo se controlan los espacios en blanco?
10. ¿Qué buenas prácticas deben seguirse al escribir Templates complejos?

---

# Resumen

En esta segunda fase profundizamos en el lenguaje **Jinja2**, aprendiendo a utilizar expresiones, operadores, condicionales (`if`, `elif`, `else`), ciclos (`for`), acceso a listas y diccionarios, comentarios y control de espacios en blanco. También estudiamos los filtros más utilizados, como `upper`, `lower`, `default`, `join` y `replace`, que permiten transformar los datos antes de generar el archivo final.

Finalmente, analizamos cómo construir Templates profesionales capaces de adaptarse automáticamente a diferentes servidores, sistemas operativos y entornos utilizando Variables, Facts y lógica de presentación, manteniendo las mejores prácticas de organización y reutilización.

En la **Fase 3** estudiaremos filtros avanzados, pruebas (*tests*), expresiones más complejas, manejo de fechas y formatos, plantillas reutilizables, macros, funciones útiles de Jinja2 y técnicas empleadas en entornos empresariales para generar configuraciones altamente dinámicas y mantenibles.


-----

# 85. Templates y Jinja2 en Ansible (Fase 3)

> **Módulo 11 – Automatización con Ansible**
>
> **Archivo:** `85-templates-jinja2.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Utilizar filtros avanzados de Jinja2.
- Comprender los Tests de Jinja2.
- Manipular listas y diccionarios.
- Utilizar expresiones complejas.
- Crear Macros reutilizables.
- Utilizar Includes en Templates.
- Comprender el uso de Namespaces.
- Diseñar Templates empresariales altamente reutilizables.

---

# Introducción

En la fase anterior aprendimos:

- Variables
- Loops
- Condicionales
- Filtros básicos
- Comentarios
- Control de espacios

Ahora estudiaremos las herramientas que convierten a Jinja2 en uno de los motores de plantillas más potentes utilizados actualmente.

Estas funcionalidades permiten administrar miles de servidores utilizando un número reducido de Templates altamente reutilizables.

---

# Arquitectura Empresarial

```text
Variables

        │

        ▼

Filtros

        │

        ▼

Condicionales

        │

        ▼

Loops

        │

        ▼

Macros

        │

        ▼

Template Final

        │

        ▼

Servidor
```

---

# Filtros Avanzados

Además de:

- upper
- lower
- replace

Jinja2 incorpora muchos filtros adicionales.

---

# to_json

Convierte un objeto a JSON.

```jinja2
{{ usuario | to_json }}
```

Resultado.

```json
{"nombre":"Alejandro"}
```

---

# to_yaml

Convierte una estructura a YAML.

```jinja2
{{ usuarios | to_yaml }}
```

---

Resultado.

```yaml
- admin
- backup
- postgres
```

---

# to_nice_yaml

Genera YAML con formato legible.

```jinja2
{{ usuarios | to_nice_yaml }}
```

Muy utilizado para depuración.

---

# to_nice_json

Produce JSON con indentación.

```jinja2
{{ variables | to_nice_json }}
```

---

# int

Convierte texto a entero.

```jinja2
{{ puerto | int }}
```

---

# float

```jinja2
{{ porcentaje | float }}
```

---

# string

```jinja2
{{ numero | string }}
```

---

# abs

```jinja2
{{ -50 | abs }}
```

Resultado.

```text
50
```

---

# round

```jinja2
{{ 5.876 | round(2) }}
```

Resultado.

```text
5.88
```

---

# min

```jinja2
{{ numeros | min }}
```

---

# max

```jinja2
{{ numeros | max }}
```

---

# random

Selecciona un elemento aleatorio.

```jinja2
{{ servidores | random }}
```

---

# reverse

```jinja2
{{ usuarios | reverse }}
```

---

# unique

Elimina duplicados.

```jinja2
{{ usuarios | unique }}
```

---

# sort

Ordena elementos.

```jinja2
{{ usuarios | sort }}
```

---

# map

Permite extraer un atributo.

Variables.

```yaml
usuarios:

- nombre: Alejandro

- nombre: Carlos

- nombre: Ana
```

Template.

```jinja2
{{ usuarios | map(attribute='nombre') | list }}
```

Resultado.

```text
Alejandro
Carlos
Ana
```

---

# select

Permite seleccionar elementos.

Ejemplo conceptual.

```jinja2
{{ usuarios | select }}
```

Generalmente se utiliza junto con Tests.

---

# reject

Hace lo contrario.

```jinja2
{{ usuarios | reject }}
```

---

# selectattr

Muy utilizado.

Variables.

```yaml
usuarios:

- nombre: admin

  activo: true

- nombre: backup

  activo: false
```

Template.

```jinja2
{{ usuarios | selectattr('activo') }}
```

---

Resultado.

```text
Solo usuarios activos
```

---

# rejectattr

Permite excluir elementos.

---

# dictsort

Ordena un diccionario.

```jinja2
{{ variables | dictsort }}
```

---

# batch

Divide una lista.

```jinja2
{{ usuarios | batch(3) }}
```

---

# slice

Divide una colección.

```jinja2
{{ usuarios | slice(2) }}
```

---

# Tests

Los Tests verifican condiciones.

No modifican datos.

---

Sintaxis.

```jinja2
variable is test
```

---

# Test defined

```jinja2
{% if puerto is defined %}
```

---

# Test undefined

```jinja2
{% if puerto is undefined %}
```

---

# Test none

```jinja2
{% if variable is none %}
```

---

# Test string

```jinja2
{% if dato is string %}
```

---

# Test number

```jinja2
{% if dato is number %}
```

---

# Test iterable

```jinja2
{% if usuarios is iterable %}
```

---

# Test mapping

Comprueba si una Variable es un diccionario.

```jinja2
{% if empresa is mapping %}
```

---

# Test sequence

Verifica listas o secuencias.

```jinja2
{% if usuarios is sequence %}
```

---

# Test even

```jinja2
{% if numero is even %}
```

---

# Test odd

```jinja2
{% if numero is odd %}
```

---

# Test divisibleby

```jinja2
{% if numero is divisibleby 5 %}
```

---

# Comparación

| Test | Uso |
|-------|-----|
| defined | Existe |
| undefined | No existe |
| none | Valor nulo |
| string | Texto |
| number | Número |
| iterable | Lista |
| mapping | Diccionario |
| even | Número par |
| odd | Número impar |

---

# Ejemplo Empresarial

```jinja2
{% if puerto is defined %}

Listen {{ puerto }}

{% else %}

Listen 80

{% endif %}
```

---

# Operador in

```jinja2
{% if "admin" in usuarios %}
```

---

También.

```jinja2
{% if "Fedora" in sistemas %}
```

---

# Operador not in

```jinja2
{% if "Oracle" not in bases %}
```

---

# Expresiones Complejas

```jinja2
{% if

ansible_distribution=="Fedora"

and

ssl

and

http_port==443

%}
```

---

# Namespaces

Permiten compartir Variables entre bloques.

Ejemplo conceptual.

```jinja2
{% set ns = namespace(total=0) %}
```

---

Actualizar.

```jinja2
{% set ns.total = ns.total + 1 %}
```

---

Resultado.

```text
Total acumulado
```

---

# Variables Temporales

```jinja2
{% set servidor = ansible_hostname %}
```

---

Posteriormente.

```jinja2
{{ servidor }}
```

---

Muy utilizadas para evitar repetir expresiones largas.

---

# Includes

Un Template puede incluir otro.

Ejemplo.

```jinja2
{% include "banner.j2" %}
```

---

Arquitectura.

```text
Template Principal

       │

       ▼

banner.j2

       │

       ▼

Archivo Final
```

---

# Beneficios

- Reutilización.
- Menor duplicación.
- Fácil mantenimiento.

---

# Macros

Las Macros funcionan como funciones reutilizables.

---

Ejemplo.

```jinja2
{% macro usuario(nombre) %}

Usuario:

{{ nombre }}

{% endmacro %}
```

Uso.

```jinja2
{{ usuario("Alejandro") }}
```

---

Resultado.

```text
Usuario:

Alejandro
```

---

# Macro con Parámetros

```jinja2
{% macro puerto(numero) %}

Listen {{ numero }}

{% endmacro %}
```

---

Invocación.

```jinja2
{{ puerto(8080) }}
```

---

# Beneficios

- Evitan duplicación.
- Centralizan lógica.
- Mejor mantenimiento.

---

# Importar Macros

```jinja2
{% import "macros.j2" as macros %}
```

Uso.

```jinja2
{{ macros.usuario("Alejandro") }}
```

---

# Llamadas Repetidas

```text
Template

↓

Macro

↓

Resultado

↓

Macro

↓

Resultado
```

---

# Templates Reutilizables

```text
Templates

│

├── banner.j2

├── usuarios.j2

├── ssl.j2

├── logging.j2

└── macros.j2
```

---

# Caso Empresarial

Empresa.

1.000 servidores.

Cada archivo.

```text
sshd_config
```

Comparte.

- Banner.
- Usuarios.
- Logging.
- Hardening.

En lugar de copiar el código.

Se utilizan Includes.

---

# Organización Recomendada

```text
templates/

├── httpd.conf.j2

├── macros.j2

├── banner.j2

├── logging.j2

├── ssl.j2

└── users.j2
```

---

# Flujo Empresarial

```text
Variables

↓

Macro

↓

Include

↓

Filtros

↓

Template

↓

Servidor
```

---

# Beneficios

- Código reutilizable.
- Menos errores.
- Mayor claridad.
- Escalabilidad.
- Fácil mantenimiento.
- Modularidad.
- Configuración consistente.

---

# Buenas Prácticas

- Dividir Templates grandes en Includes.
- Utilizar Macros para bloques repetitivos.
- Validar Variables con `defined`.
- Utilizar `default` cuando sea apropiado.
- Mantener la lógica sencilla.
- Documentar Macros complejas.
- Centralizar código reutilizable.
- Utilizar nombres descriptivos.
- Evitar duplicación.
- Probar cada Template individualmente.

---

# Errores Comunes

## Error 1

Duplicar bloques de configuración.

---

## Error 2

No utilizar Macros.

---

## Error 3

No validar Variables.

---

## Error 4

Utilizar Includes con rutas incorrectas.

---

## Error 5

Sobrecargar el Template con demasiada lógica.

---

## Error 6

No separar componentes reutilizables.

---

## Error 7

No utilizar Tests.

---

## Error 8

No documentar Macros.

---

## Error 9

Ignorar el orden de evaluación.

---

## Error 10

No probar el resultado generado.

---

# Laboratorio RHCSA

## Escenario

Una empresa administrará 900 servidores Linux utilizando Templates completamente modulares.

---

## Laboratorio 1

Aplicar:

- sort
- unique
- join

sobre una lista de usuarios.

---

## Laboratorio 2

Utilizar `selectattr` para mostrar únicamente usuarios activos.

---

## Laboratorio 3

Crear un bloque utilizando:

```jinja2
is defined
```

---

## Laboratorio 4

Crear Variables temporales mediante:

```jinja2
set
```

---

## Laboratorio 5

Crear un Include.

```jinja2
banner.j2
```

---

## Laboratorio 6

Crear una Macro reutilizable.

---

## Laboratorio 7

Importar la Macro desde otro Template.

---

## Laboratorio 8

Diseñar un conjunto de Templates reutilizables para configurar `sshd_config`, separando Banner, usuarios autorizados, parámetros de seguridad y configuración de logs mediante Includes y Macros.

---

## Laboratorio 9

Crear un Template que procese una lista de servidores, seleccione únicamente aquellos marcados como activos utilizando `selectattr`, ordene el resultado alfabéticamente y genere automáticamente un archivo de configuración.

---

## Laboratorio 10

Construir un Template empresarial que utilice:

- Variables temporales (`set`).
- Tests (`defined`, `mapping`, `iterable`).
- Filtros avanzados.
- Includes.
- Macros.
- Validaciones.

para generar una configuración modular lista para Producción.

---

# Preguntas de Repaso

1. ¿Qué diferencia existe entre un filtro y un Test?
2. ¿Cuándo utilizarías `selectattr`?
3. ¿Qué ventajas ofrece `map`?
4. ¿Qué función cumple `namespace`?
5. ¿Para qué sirve `set`?
6. ¿Cuál es la diferencia entre `include` e `import` de Macros?
7. ¿Qué beneficios aportan las Macros?
8. ¿Cómo ayudan los Includes a mantener Templates organizados?
9. ¿Qué buenas prácticas deben seguirse al diseñar Templates empresariales?
10. ¿Cómo estructurarías un conjunto de Templates reutilizables para administrar cientos de servidores?

---

# Resumen

En esta tercera fase profundizamos en las capacidades avanzadas de **Jinja2**, estudiando filtros como `map`, `selectattr`, `unique`, `sort` y `batch`, así como los **Tests** (`defined`, `mapping`, `iterable`, `number`, `string`, entre otros) que permiten validar datos antes de generar el archivo final.

También aprendimos a utilizar Variables temporales mediante `set`, compartir estado con `namespace`, reutilizar fragmentos mediante `include` y crear funciones reutilizables utilizando **Macros**. Estas herramientas permiten construir Templates altamente modulares, legibles y mantenibles, preparados para administrar infraestructuras empresariales de gran escala.

En la **Fase 4** estudiaremos técnicas avanzadas de depuración de Templates, validación de configuraciones antes del despliegue, integración con Roles, mejores prácticas empresariales, optimización del rendimiento, troubleshooting y un laboratorio integral que combinará todos los conceptos aprendidos sobre Templates y Jinja2.

------

# 85. Templates y Jinja2 en Ansible (Fase 4)

> **Módulo 11 – Automatización con Ansible**
>
> **Archivo:** `85-templates-jinja2.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase serás capaz de:

- Validar Templates antes de implementarlos.
- Integrar Templates con Roles.
- Aplicar buenas prácticas empresariales.
- Optimizar Templates para grandes infraestructuras.
- Comprender el flujo completo de generación de configuraciones.
- Diagnosticar errores comunes.
- Diseñar una arquitectura empresarial basada en Templates reutilizables.

---

# Introducción

Durante este capítulo hemos aprendido:

- Variables
- Filtros
- Loops
- Condicionales
- Macros
- Includes
- Tests
- Namespaces

Ahora aprenderemos cómo utilizan realmente las empresas los Templates.

La mayoría de organizaciones grandes administran cientos o miles de servidores utilizando únicamente un conjunto reducido de Templates reutilizables.

---

# Arquitectura Empresarial

```text
                Inventario

                     │

                     ▼

               group_vars

                     │

                     ▼

                host_vars

                     │

                     ▼

                   Role

                     │

                     ▼

                Templates

                     │

                     ▼

                  Jinja2

                     │

                     ▼

            Archivo generado

                     │

                     ▼

                 Servidor
```

---

# Flujo Completo

Un Template nunca trabaja aislado.

Generalmente participa en un flujo completo.

```text
Playbook

↓

Inventario

↓

Variables

↓

Role

↓

Template

↓

Jinja2

↓

Archivo Final

↓

Servicio
```

---

# Caso Real

Supongamos una empresa con:

- 600 servidores Apache
- 300 servidores PostgreSQL
- 250 servidores Nginx

Todos necesitan archivos distintos.

Pero únicamente existen.

```text
3 Templates
```

Uno para cada servicio.

---

# ¿Cómo es posible?

Porque toda la configuración proviene de Variables.

---

Ejemplo.

```yaml
http_port: 8080

ssl: true

server_admin: admin@empresa.com
```

El Template nunca cambia.

Las Variables sí.

---

# Templates dentro de Roles

La organización recomendada.

```text
roles/

└── apache/

    ├── tasks

    ├── handlers

    ├── defaults

    ├── vars

    └── templates

           ├── httpd.conf.j2

           ├── ssl.conf.j2

           └── vhost.j2
```

---

# Flujo

```text
Role

↓

Task

↓

Template

↓

Servidor
```

---

# Validación

Antes de copiar un Template.

Es recomendable ejecutar.

```bash
ansible-playbook site.yml \
--check
```

---

Esto permite detectar problemas antes de modificar el servidor.

---

# Validación de Sintaxis

También.

```bash
ansible-playbook site.yml \
--syntax-check
```

---

Muy recomendable.

---

# Mostrar Diferencias

```bash
ansible-playbook site.yml \
--diff
```

---

Salida.

```text
Archivo Actual

↓

Cambios

↓

Archivo Nuevo
```

---

Muy útil para auditorías.

---

# Verbose

```bash
ansible-playbook site.yml \
-vvv
```

Permite observar.

- Variables
- Templates
- Tasks
- Errores

---

# Validar Archivos Antes de Aplicarlos

Uno de los aspectos más importantes del módulo `template` es el parámetro:

```yaml
validate:
```

Este parámetro permite comprobar que el archivo generado sea válido antes de reemplazar el archivo existente.

Si la validación falla, **Ansible no copiará el archivo**.

---

# Ejemplo con Apache

```yaml
- name: Instalar configuración de Apache

  template:

    src: httpd.conf.j2

    dest: /etc/httpd/conf/httpd.conf

    validate: "httpd -t -f %s"
```

---

Flujo.

```text
Template

↓

Archivo Temporal

↓

httpd -t

↓

Correcto

↓

Copiar
```

---

Si ocurre un error.

```text
Template

↓

Archivo Temporal

↓

httpd -t

↓

Error

↓

No copiar
```

---

# Ejemplo con SSH

```yaml
validate: "sshd -t -f %s"
```

---

# Ejemplo con Sudoers

```yaml
validate: "visudo -cf %s"
```

---

# Ejemplo con Nginx

```yaml
validate: "nginx -t -c %s"
```

---

# Beneficios

- Evita archivos inválidos.
- Reduce interrupciones.
- Protege Producción.
- Facilita despliegues seguros.

---

# Idempotencia

Un Template correctamente diseñado siempre debe ser idempotente.

---

¿Qué significa?

Si ejecutamos.

```bash
ansible-playbook site.yml
```

Cinco veces.

El resultado será.

```text
Servidor

↓

Sin cambios adicionales
```

---

Representación.

```text
Primera ejecución

↓

Archivo actualizado

↓

Segunda ejecución

↓

OK

↓

Sin cambios
```

---

# Templates Inteligentes

Un buen Template solamente modifica aquello que cambia.

No genera diferencias innecesarias.

---

# Notificación de Handlers

Generalmente.

```yaml
template:

↓

notify:

↓

Restart Service
```

---

Ejemplo.

```yaml
- name: Configuración Apache

  template:

    src: httpd.conf.j2

    dest: /etc/httpd/conf/httpd.conf

  notify:

    - Reiniciar Apache
```

---

Flujo.

```text
Template

↓

Archivo cambió

↓

Notify

↓

Handler

↓

Restart
```

---

Si el archivo no cambia.

```text
No Restart
```

---

Gran ventaja.

---

# Variables Centralizadas

La mejor práctica.

```text
group_vars/

↓

host_vars/

↓

defaults/

↓

Template
```

---

No colocar valores fijos dentro del Template.

---

Mala práctica.

```jinja2
Listen 8080
```

---

Buena práctica.

```jinja2
Listen {{ http_port }}
```

---

# Separación de Responsabilidades

Un Template debe encargarse únicamente de generar texto.

No debe contener lógica compleja de negocio.

---

Incorrecto.

```text
Template

↓

50 condicionales

↓

20 loops

↓

Muy difícil mantenimiento
```

---

Correcto.

```text
Variables

↓

Playbook

↓

Template sencillo
```

---

# Organización Empresarial

```text
templates/

├── apache

│      ├── httpd.conf.j2

│      ├── ssl.j2

│      └── vhost.j2

│

├── postgres

│      ├── pg_hba.conf.j2

│      └── postgresql.conf.j2

│

├── ssh

│      ├── sshd_config.j2

│      └── banner.j2

│

└── monitoring

       ├── prometheus.yml.j2

       └── grafana.ini.j2
```

---

# Integración con Facts

Ejemplo.

```jinja2
Hostname:

{{ ansible_hostname }}

Sistema:

{{ ansible_distribution }}

Memoria:

{{ ansible_memtotal_mb }}
```

Cada servidor genera automáticamente un archivo diferente.

---

# Integración con Inventarios

```text
Inventario

↓

Variables

↓

Template

↓

Configuración personalizada
```

---

# Estrategia Empresarial

```text
Producción

↓

QA

↓

Desarrollo
```

Todos utilizan.

```text
El mismo Template
```

Lo único que cambia son las Variables.

---

# Optimización

En lugar de crear.

```text
httpd_dev.conf

httpd_qa.conf

httpd_prod.conf
```

Se utiliza.

```text
httpd.conf.j2
```

Con Variables.

---

# Versionado

Todo Template debe almacenarse en Git.

```text
Git

↓

Commit

↓

Review

↓

QA

↓

Producción
```

---

# Auditoría

Antes de Producción.

Verificar.

```text
✓ Variables

✓ Includes

✓ Macros

✓ Filtros

✓ Loops

✓ Condicionales

✓ Validate

✓ Handlers

✓ Sintaxis

✓ Git
```

---

# Rendimiento

En grandes infraestructuras.

Es recomendable.

- Evitar lógica innecesaria.
- Evitar Loops excesivos.
- Utilizar Includes.
- Utilizar Macros.
- Reutilizar Variables.
- Mantener Templates pequeños.

---

# Troubleshooting

## Error

```text
Undefined Variable
```

Causa.

```text
Variable inexistente.
```

Solución.

```jinja2
default()
```

o.

```jinja2
is defined
```

---

## Error

```text
Template Syntax Error
```

Generalmente.

- Llaves sin cerrar.
- endif ausente.
- endfor ausente.

---

## Error

```text
Unexpected End of Template
```

Normalmente ocurre por.

```jinja2
{% if %}
```

Sin.

```jinja2
{% endif %}
```

---

## Error

```text
Template Not Found
```

Revisar.

```text
templates/
```

---

## Error

```text
File Not Generated
```

Revisar.

- Variables.
- Ruta.
- Permisos.
- Tasks.

---

## Error

```text
Validation Failed
```

Generalmente indica que el comando especificado en `validate` detectó un error en la configuración generada.

Revisar:

- Mensaje devuelto por el comando de validación.
- Variables utilizadas.
- Sintaxis del archivo generado.

---

## Error

```text
Handler Never Runs
```

Posibles causas.

- El Template no produjo cambios.
- No existe la directiva `notify`.
- El nombre del Handler no coincide.

---

# Caso Empresarial

Infraestructura.

```text
2.000 Servidores
```

Templates.

```text
25 Templates
```

Roles.

```text
18 Roles
```

Playbooks.

```text
4
```

Gracias a Variables y Jinja2.

Toda la infraestructura puede mantenerse utilizando únicamente esos Templates.

---

# Arquitectura Completa

```text
                   Git

                    │

                    ▼

              Playbooks

                    │

                    ▼

               Inventarios

                    │

                    ▼

               group_vars

                    │

                    ▼

                  Roles

                    │

                    ▼

               Templates

                    │

                    ▼

                  Jinja2

                    │

                    ▼

          Configuración Final

                    │

                    ▼

               2.000 Servidores
```

---

# Buenas Prácticas

- Mantener Templates pequeños.
- Utilizar Variables para todos los valores configurables.
- Validar configuraciones mediante `validate`.
- Probar siempre con `--check`.
- Revisar cambios utilizando `--diff`.
- Versionar Templates en Git.
- Reutilizar Macros e Includes.
- Documentar Variables requeridas.
- Mantener la lógica de negocio fuera del Template.
- Diseñar Templates reutilizables para distintos entornos.
- Utilizar Handlers para reiniciar servicios únicamente cuando sea necesario.
- Probar los Templates en un entorno de laboratorio antes de Producción.

---

# Errores Comunes

## Error 1

Hardcodear valores.

---

## Error 2

Duplicar Templates.

---

## Error 3

No validar sintaxis.

---

## Error 4

No utilizar Git.

---

## Error 5

No utilizar `validate`.

---

## Error 6

Modificar manualmente archivos administrados por Ansible.

---

## Error 7

Crear lógica excesiva dentro del Template.

---

## Error 8

No utilizar Handlers.

---

## Error 9

No probar en laboratorio.

---

## Error 10

No documentar Variables.

---

# Laboratorio Integral RHCSA

## Escenario

Una empresa administrará:

- 1.500 servidores Linux.
- 500 Apache.
- 300 PostgreSQL.
- 200 Nginx.
- 150 SQL Server.

Toda la configuración será generada mediante Templates.

---

## Laboratorio 1

Crear un Template para Apache.

---

## Laboratorio 2

Agregar Variables desde `group_vars`.

---

## Laboratorio 3

Crear Includes para:

- SSL.
- Logging.
- Virtual Hosts.

---

## Laboratorio 4

Crear Macros reutilizables.

---

## Laboratorio 5

Utilizar Facts.

---

## Laboratorio 6

Aplicar:

```yaml
validate
```

para verificar la sintaxis de Apache antes de reemplazar el archivo.

---

## Laboratorio 7

Ejecutar.

```bash
--check

--diff

-vvv
```

---

## Laboratorio 8

Comprobar la idempotencia ejecutando el mismo Playbook varias veces y verificando que no existan cambios cuando la configuración ya coincide con el estado deseado.

---

## Laboratorio 9

Crear un Role empresarial que utilice Templates para configurar Apache, SSH y PostgreSQL, integrando Variables provenientes de `group_vars`, Includes, Macros y Handlers.

---

## Laboratorio 10

Diseñar una arquitectura completa para Producción que incluya:

- Roles.
- Templates reutilizables.
- Variables centralizadas.
- Validación mediante `validate`.
- Handlers.
- Git.
- Pruebas con `--check`.
- Revisión con `--diff`.
- Pipeline de CI/CD.
- Estrategia de despliegue segura e idempotente.

---

# Preguntas de Repaso

1. ¿Qué ventaja ofrece el parámetro `validate` del módulo `template`?
2. ¿Por qué es importante la idempotencia en los Templates?
3. ¿Cómo interactúan los Templates con los Handlers?
4. ¿Dónde deberían almacenarse las Variables de configuración?
5. ¿Qué ventajas aporta `--diff` durante un despliegue?
6. ¿Por qué es recomendable utilizar `validate` antes de reemplazar archivos críticos?
7. ¿Qué errores comunes pueden producirse al generar un Template?
8. ¿Cómo organizarías los Templates dentro de un Role empresarial?
9. ¿Qué beneficios aporta Git al mantenimiento de Templates?
10. ¿Cómo diseñarías una solución basada en Templates para administrar miles de servidores?

---

# Resumen del Capítulo

En este capítulo estudiamos **Templates y Jinja2**, uno de los componentes más importantes de Ansible para generar archivos de configuración dinámicos y reutilizables. Aprendimos a utilizar Variables, expresiones, filtros, condicionales, ciclos, Tests, Macros, Includes y Variables temporales para construir Templates altamente flexibles.

También analizamos la integración de los Templates con Roles, Inventarios, Facts y Handlers, así como el uso del parámetro `validate` para verificar la sintaxis antes de aplicar cambios, garantizando despliegues más seguros. Finalmente, revisamos estrategias empresariales para organizar, versionar, probar y mantener Templates destinados a administrar miles de servidores de manera consistente e idempotente.

---

# Próximo Capítulo

## **86. Ansible Vault**

En el siguiente capítulo aprenderemos a proteger información sensible utilizando **Ansible Vault**, incluyendo el cifrado de archivos y Variables, administración de contraseñas, múltiples Vault IDs, integración con `group_vars` y `host_vars`, automatización segura de secretos y mejores prácticas para entornos empresariales.

-----








 









