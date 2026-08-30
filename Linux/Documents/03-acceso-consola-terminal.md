# 3. Acceso a la Consola y Terminal

![Administrador Linux trabajando desde una terminal](images/03-consola-terminal-portada.jpg)

> **Objetivo del capítulo**
>
> Aprender las diferentes formas de acceder a un sistema Linux, comprender la diferencia entre consola y terminal, conocer cuándo utilizar una interfaz gráfica o la línea de comandos y familiarizarse con el entorno donde se desarrollarán la mayoría de las tareas de administración en RHCSA.

---

# Introducción

Uno de los mayores cambios para quienes vienen de Windows es descubrir que en Linux **la terminal no es una herramienta opcional**, sino el principal medio de administración del sistema.

Aunque muchas distribuciones incluyen un entorno gráfico moderno, en servidores empresariales normalmente no existe interfaz gráfica. Todas las tareas administrativas se realizan desde una consola de texto o una terminal remota.

Por esta razón, dominar la terminal es uno de los requisitos más importantes para aprobar la certificación **Red Hat Certified System Administrator (RHCSA)**.

---

# ¿Qué es la consola?

![Consola virtual de Linux](images/03-consola-linux.jpg)

La **consola** es una interfaz de texto que permite interactuar directamente con el sistema operativo.

Históricamente, la consola era un monitor y un teclado conectados físicamente al servidor. Hoy en día, el término también se utiliza para referirse a las **consolas virtuales** disponibles en Linux.

Desde la consola es posible:

- Iniciar sesión.
- Ejecutar comandos.
- Administrar usuarios.
- Configurar servicios.
- Revisar registros.
- Solucionar problemas.
- Reiniciar el sistema.

En muchos servidores Linux la consola es el único medio disponible para administrar el equipo.

---

# ¿Qué es una terminal?

![Terminal GNOME abierta](images/03-terminal-gnome.jpg)

Una **terminal** es una aplicación que permite acceder a una consola desde un entorno gráfico.

Es decir, la terminal actúa como un emulador de consola.

Algunas terminales populares son:

- GNOME Terminal
- Konsole
- XFCE Terminal
- Tilix
- Terminator

Aunque cada aplicación tiene un aspecto diferente, todas permiten ejecutar exactamente los mismos comandos del sistema.

---

# Consola vs Terminal

Muchas personas utilizan ambos términos como sinónimos, pero técnicamente existe una diferencia.

| Consola | Terminal |
|----------|----------|
| Interfaz directa del sistema | Aplicación que emula una consola |
| Puede existir sin entorno gráfico | Requiere un entorno gráfico |
| Disponible incluso si GNOME no inicia | Se ejecuta dentro del escritorio |
| Muy utilizada en servidores | Muy utilizada en estaciones de trabajo |

En la práctica diaria escucharás ambos términos con frecuencia.

---

# ¿Qué es un Shell?

![Bash ejecutándose en Fedora](images/03-shell-bash.jpg)

La consola por sí sola no interpreta comandos.

Para ello utiliza un programa llamado **Shell**.

El Shell recibe las instrucciones del usuario y las envía al sistema operativo.

El shell más utilizado en Linux es:

```text
Bash (Bourne Again Shell)
```

Otros shells conocidos son:

- Zsh
- Fish
- Ksh
- Tcsh

Durante el examen RHCSA trabajarás principalmente con **Bash**.

---

# Flujo de funcionamiento

```text
Usuario
     │
     ▼
 Terminal
     │
     ▼
 Shell (Bash)
     │
     ▼
 Kernel Linux
     │
     ▼
 Hardware
```

Cada comando sigue este flujo hasta ejecutarse.

---

# Abrir una terminal en Fedora

Existen varias formas de abrir una terminal.

## Método 1

Presiona

```
Ctrl + Alt + T
```

*(Disponible en algunas configuraciones de escritorio.)*

---

## Método 2

Desde el menú:

```
Activities
      ↓
Terminal
```

---

## Método 3

Buscar

```
Terminal
```

en el buscador de aplicaciones.

---

# Consolas virtuales (TTY)

Linux dispone de varias consolas virtuales.

Puedes cambiar entre ellas utilizando:

```
Ctrl + Alt + F1
Ctrl + Alt + F2
Ctrl + Alt + F3
...
Ctrl + Alt + F6
```

En Fedora moderno:

- **TTY1** suele utilizarse para la interfaz gráfica.
- **TTY2–TTY6** son consolas de texto disponibles para iniciar sesión.

Para regresar al escritorio gráfico normalmente se utiliza:

```
Ctrl + Alt + F1
```

o

```
Ctrl + Alt + F2
```

(según la versión de Fedora).

---

# Acceso remoto mediante SSH

![Administrador conectado por SSH](images/03-ssh-terminal.jpg)

En ambientes empresariales rara vez se trabaja directamente frente al servidor.

Lo habitual es utilizar **SSH (Secure Shell)**.

Ejemplo:

```bash
ssh administrador@192.168.1.50
```

Con SSH puedes administrar un servidor desde cualquier lugar de forma segura.

---

# El Prompt

Al iniciar sesión aparecerá algo parecido a:

```bash
ajimenez@fedora:~$
```

Cada parte tiene un significado.

| Elemento | Significado |
|----------|-------------|
| ajimenez | Usuario conectado |
| fedora | Nombre del equipo |
| ~ | Directorio personal |
| $ | Usuario normal |

Cuando aparece:

```bash
#
```

significa que estás trabajando como **root**.

---

# Primeros comandos

Verificar usuario:

```bash
whoami
```

Ver el directorio actual:

```bash
pwd
```

Listar archivos:

```bash
ls
```

Limpiar pantalla:

```bash
clear
```

Ver fecha:

```bash
date
```

Ver calendario:

```bash
cal
```

Salir de la sesión:

```bash
exit
```

---

# Buenas prácticas

- Trabaja normalmente con un usuario estándar.
- Utiliza `sudo` únicamente cuando sea necesario.
- Evita iniciar sesión permanentemente como root.
- Lee cuidadosamente cada comando antes de ejecutarlo.
- No copies comandos desconocidos desde Internet sin comprender su función.

---

# Errores comunes

❌ Confundir consola con terminal.

❌ Pensar que Linux necesita interfaz gráfica.

❌ Trabajar siempre como root.

❌ Cerrar accidentalmente la sesión SSH sin utilizar herramientas como `screen` o `tmux` cuando ejecutas procesos largos.

---

# Actividad práctica

Abre una terminal y ejecuta:

```bash
whoami
hostname
pwd
date
uptime
uname -r
```

Responde:

1. ¿Cuál es tu usuario?
2. ¿Cuál es el nombre del equipo?
3. ¿En qué directorio estás?
4. ¿Qué versión del kernel utiliza tu sistema?
5. ¿Cuánto tiempo lleva encendido?

---

# Resumen

En este capítulo aprendiste que la terminal será la herramienta principal durante todo el curso RHCSA.

Comprendiste la diferencia entre consola, terminal y shell, conociste las consolas virtuales (TTY), el acceso remoto mediante SSH y los primeros comandos que utilizarás continuamente como administrador Linux.

Dominar la terminal es el primer paso para administrar servidores Linux de manera profesional.

---

# Próxima lección

➡ **4. Primeros comandos de Linux**