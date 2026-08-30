# 5. Comprendiendo el concepto de Root en Linux

![Administrador Linux trabajando como usuario root](images/05-root-portada.jpg)

> **Objetivo del capítulo**
>
> Comprender los diferentes significados del término **root** en Linux y aprender a distinguir entre el usuario **root**, el directorio raíz (`/`) y el directorio personal del usuario root (`/root`).

---

# Introducción

Uno de los errores más comunes entre quienes comienzan a trabajar con Linux es pensar que **todo lo que contiene la palabra "root" significa lo mismo**.

Durante la administración de servidores escucharás expresiones como:

- "Inicia sesión como root."
- "Ve al directorio raíz."
- "Accede al directorio `/root`."

Aunque todas contienen la palabra **root**, **no significan lo mismo**.

Comprender esta diferencia es fundamental para administrar correctamente un sistema Linux y evitar errores durante el examen RHCSA.

---

# ¿Qué significa Root?

En Linux, la palabra **root** puede referirse a **tres conceptos completamente diferentes**.

1. El usuario **root**.
2. El directorio raíz **/**.
3. El directorio personal del usuario root **/root**.

Veamos cada uno de ellos.

---

# 1. El usuario Root

![Administrador utilizando privilegios root](images/05-root-user.jpg)

El usuario **root** es la cuenta administrativa más poderosa del sistema Linux.

Tiene permisos para:

- Crear usuarios.
- Eliminar usuarios.
- Instalar software.
- Configurar servicios.
- Cambiar permisos.
- Modificar archivos del sistema.
- Reiniciar el servidor.
- Administrar discos.
- Configurar redes.
- Acceder a cualquier archivo.

En otras palabras, **root puede realizar prácticamente cualquier operación sobre el sistema**.

---

## ¿Cómo identificar al usuario root?

Cuando iniciamos sesión como root, el prompt suele terminar con el símbolo:

```bash
#
```

Ejemplo:

```bash
root@server:~#
```

Mientras que un usuario normal utiliza:

```bash
$
```

Ejemplo:

```bash
ajimenez@fedora:~$
```

---

## ¿Cómo convertirse en root?

En distribuciones como Fedora o Red Hat es común utilizar:

```bash
sudo -i
```

o

```bash
sudo su -
```

Si la cuenta root está habilitada también puede utilizarse:

```bash
su -
```

El sistema solicitará la contraseña correspondiente antes de otorgar privilegios administrativos.

---

# ¿Por qué no debemos trabajar siempre como root?

Aunque root tiene control total del sistema, **no es recomendable utilizar esta cuenta para el trabajo diario**.

Las razones son:

- Un error puede afectar todo el sistema.
- Es posible eliminar archivos críticos accidentalmente.
- Se incrementa el riesgo de seguridad.
- Resulta más difícil auditar quién realizó una acción.

La práctica recomendada consiste en trabajar con un usuario normal y utilizar `sudo` únicamente cuando sea necesario.

---

# 2. El directorio raíz (/)

![Estructura del sistema de archivos Linux](images/05-root-directory.jpg)

El segundo significado de **root** hace referencia al **directorio raíz**, representado por una barra inclinada:

```text
/
```

Este es el punto más alto del sistema de archivos Linux.

Todos los directorios y archivos parten de esta ubicación.

Ejemplo:

```text
/
├── bin
├── boot
├── dev
├── etc
├── home
├── lib
├── media
├── opt
├── proc
├── root
├── run
├── srv
├── sys
├── tmp
├── usr
└── var
```

Cuando alguien dice:

> "Ve al directorio raíz."

Debe interpretarse como:

```bash
cd /
```

No significa cambiar al usuario root ni acceder a `/root`.

---

# ¿Qué contiene el directorio raíz?

Desde el directorio `/` se organizan todos los componentes del sistema operativo.

Algunos directorios importantes son:

| Directorio | Contenido |
|------------|-----------|
| `/bin` | Comandos esenciales |
| `/boot` | Archivos de arranque |
| `/dev` | Dispositivos del sistema |
| `/etc` | Configuración |
| `/home` | Directorios personales de usuarios |
| `/opt` | Software adicional |
| `/proc` | Información del kernel y procesos |
| `/root` | Directorio personal del usuario root |
| `/tmp` | Archivos temporales |
| `/usr` | Programas y bibliotecas |
| `/var` | Datos variables y registros |

---

# 3. El directorio /root

![Directorio personal del usuario root](images/05-root-home.jpg)

El tercer significado corresponde al directorio:

```text
/root
```

Este es el **directorio personal (Home)** del usuario root.

Funciona igual que el directorio personal de cualquier otro usuario.

Ejemplo:

| Usuario | Directorio personal |
|----------|---------------------|
| juan | `/home/juan` |
| maria | `/home/maria` |
| admin | `/home/admin` |
| root | `/root` |

Por ejemplo:

```bash
cd /root
```

nos lleva al directorio personal del administrador.

---

# Diferencia entre / y /root

Esta es una de las confusiones más frecuentes.

| Ruta | Significado |
|------|-------------|
| `/` | Directorio raíz del sistema |
| `/root` | Directorio personal del usuario root |

No representan el mismo lugar.

---

# Ejemplo práctico

Supongamos que ejecutamos:

```bash
cd /
pwd
```

Resultado:

```text
/
```

Ahora ejecutamos:

```bash
cd /root
pwd
```

Resultado:

```text
/root
```

Aunque ambos contienen la palabra **root**, son directorios diferentes.

---

# Resumen visual

```text
Sistema de archivos

            /
            │
 ├──────────┼─────────────┐
 │          │             │
home       etc         root
 │                        │
 ├── juan                 │
 ├── maria                │
 └── pedro                │
                          │
                    Directorio personal
                    del usuario root
```

---

# Comparación de los tres significados

| Concepto | Significado |
|----------|-------------|
| root | Usuario administrador |
| / | Directorio raíz del sistema |
| /root | Directorio personal del usuario root |

---

# Comandos útiles

Ver el usuario actual:

```bash
whoami
```

Mostrar el directorio actual:

```bash
pwd
```

Ir al directorio raíz:

```bash
cd /
```

Ir al directorio personal de root:

```bash
cd /root
```

Cambiar al usuario root:

```bash
sudo -i
```

---

# Buenas prácticas

✅ Trabajar normalmente con un usuario estándar.

✅ Utilizar `sudo` cuando sea necesario.

✅ Evitar iniciar sesión permanentemente como root.

✅ Verificar siempre en qué directorio te encuentras utilizando:

```bash
pwd
```

---

# Errores comunes

❌ Pensar que `/` y `/root` son el mismo directorio.

❌ Creer que root siempre significa usuario.

❌ Trabajar permanentemente con privilegios de administrador.

❌ Ejecutar comandos peligrosos como root sin revisar cuidadosamente su sintaxis.

---

# Laboratorio práctico

Realiza las siguientes actividades:

Consultar el usuario actual:

```bash
whoami
```

Mostrar el directorio actual:

```bash
pwd
```

Ir al directorio raíz:

```bash
cd /
pwd
```

Ir al directorio personal del usuario root (si tienes permisos):

```bash
sudo -i
cd /root
pwd
```

Comparar ambas rutas y responder:

1. ¿Cuál es la diferencia entre `/` y `/root`?
2. ¿Qué usuario está utilizando la sesión?
3. ¿Qué símbolo aparece en el prompt cuando eres root?
4. ¿Qué comando utilizaste para obtener privilegios administrativos?

---

# Preguntas de repaso

1. ¿Cuáles son los tres significados de la palabra **root** en Linux?
2. ¿Qué permisos tiene el usuario root?
3. ¿Qué representa el directorio `/`?
4. ¿Qué representa el directorio `/root`?
5. ¿Por qué no es recomendable trabajar siempre como root?
6. ¿Qué comando muestra el usuario actual?
7. ¿Qué comando muestra el directorio actual?
8. ¿Qué comando permite acceder al directorio raíz?
9. ¿Qué comando permite cambiar al usuario root utilizando `sudo`?

---

# Resumen

En Linux, la palabra **root** puede referirse al usuario administrador, al directorio raíz del sistema (`/`) o al directorio personal del usuario root (`/root`).

Comprender esta diferencia evita errores frecuentes y constituye uno de los conceptos fundamentales para administrar correctamente un sistema Linux.

Durante todo el curso RHCSA trabajarás con estos tres conceptos, por lo que es importante identificarlos con claridad desde el inicio.

---

# Próxima lección

➡ **6. Navegación por el sistema de archivos Linux**