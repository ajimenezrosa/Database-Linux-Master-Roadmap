# 74. Podman Rootless vs Rootful (Fase 1)

> **Módulo 10 – Contenedores con Podman**
>
> **Manual RHCSA**
>
> **Archivo:** `74-podman-rootless-rootful.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase podrás:

- Comprender qué significa ejecutar Podman en modo Rootless.
- Comprender qué significa ejecutar Podman en modo Rootful.
- Identificar las diferencias entre ambos modos.
- Entender cómo funcionan los espacios de nombres de usuarios (User Namespaces).
- Comprender el mapeo de UID y GID.
- Conocer las ventajas de seguridad del modo Rootless.
- Determinar cuándo utilizar cada modalidad en ambientes empresariales.
- Prepararte para preguntas del examen RHCSA relacionadas con seguridad y contenedores.

---

# Introducción

Una de las características más importantes de Podman es que **no necesita ejecutarse como root**.

A diferencia de muchos motores de contenedores tradicionales, Podman fue diseñado para funcionar de forma segura utilizando cuentas normales de Linux.

Esto representa una enorme ventaja desde el punto de vista de la seguridad.

En la actualidad, la mayoría de las distribuciones modernas de Red Hat recomiendan utilizar **Rootless Containers** siempre que sea posible.

---

# ¿Qué significa Rootless?

Un contenedor Rootless es un contenedor ejecutado por un usuario normal del sistema.

Por ejemplo:

```text
Usuario Linux

alejandro
```

Ejecuta:

```bash
podman run nginx
```

Sin utilizar:

```bash
sudo
```

---

# ¿Qué significa Rootful?

Un contenedor Rootful es ejecutado por el usuario:

```text
root
```

Ejemplo

```bash
sudo podman run nginx
```

o

```bash
su -

podman run nginx
```

---

# Comparación rápida

| Característica | Rootless | Rootful |
|---------------|----------|----------|
| Requiere root | No | Sí |
| Mayor seguridad | Sí | No |
| Servicios privilegiados | Limitado | Completo |
| Ideal para desarrollo | Sí | Sí |
| Ideal para producción | Sí | Depende del caso |
| Riesgo ante compromiso | Bajo | Alto |

---

# Arquitectura General

## Rootless

```text
Usuario

alejandro

      │

      ▼

   Podman

      │

      ▼

Container

      │

      ▼

User Namespace
```

---

## Rootful

```text
Root

      │

      ▼

   Podman

      │

      ▼

Container

      │

      ▼

Kernel
```

---

# ¿Por qué Rootless es más seguro?

En Linux, el usuario root posee control absoluto sobre el sistema.

Si un atacante consigue escapar del contenedor y el proceso se ejecuta como root, el impacto puede ser muy alto.

Con Rootless:

```text
Container

↓

Usuario normal

↓

Permisos limitados
```

Incluso si existiera una vulnerabilidad grave, el atacante quedaría limitado por los permisos del usuario propietario del contenedor.

---

# Principio de Mínimos Privilegios

Podman Rootless sigue uno de los principios fundamentales de seguridad:

```text
Least Privilege
```

o

```text
Principio de Mínimos Privilegios
```

Consiste en otorgar únicamente los permisos estrictamente necesarios.

---

# Arquitectura de Seguridad

```text
Usuario

      │

      ▼

Permisos mínimos

      │

      ▼

Contenedor

      │

      ▼

Kernel
```

---

# ¿Cómo funciona Rootless?

Internamente Podman utiliza varias tecnologías del Kernel Linux.

Las principales son:

- User Namespaces
- Mount Namespaces
- PID Namespaces
- Network Namespaces
- Cgroups
- Seccomp
- SELinux

---

# User Namespace

Es probablemente la característica más importante del modo Rootless.

Permite que un usuario sea:

```text
root

↓

Dentro del contenedor
```

Pero continúe siendo:

```text
usuario normal

↓

En el Host
```

---

# Ejemplo

Dentro del contenedor:

```bash
id
```

Resultado

```text
uid=0(root)
```

Pero en el Host:

```bash
id
```

Resultado

```text
uid=1000(alejandro)
```

---

# Diagrama

```text
Host

UID 1000

      │

      ▼

User Namespace

      │

      ▼

Container

UID 0
```

---

# Traducción de UID

El Kernel realiza una traducción automática.

```text
Host

UID 1000

↓

Container

UID 0
```

Esto NO significa que el usuario tenga privilegios reales sobre el sistema operativo.

---

# SubUID

Linux necesita reservar rangos de identificadores.

Consultar:

```bash
cat /etc/subuid
```

Ejemplo

```text
alejandro:100000:65536
```

---

# Significado

```text
Usuario

alejandro

↓

UID inicial

100000

↓

Cantidad

65536
```

---

# SubGID

Consultar

```bash
cat /etc/subgid
```

Ejemplo

```text
alejandro:100000:65536
```

---

# ¿Para qué sirven?

Permiten mapear miles de UID y GID dentro del contenedor sin afectar las cuentas reales del Host.

---

# Arquitectura

```text
Host

UID 1000

↓

SubUID

100000-165535

↓

Container

UID 0-65535
```

---

# Consultar el mapa

Dentro del contenedor

```bash
cat /proc/self/uid_map
```

Ejemplo

```text
         Inside     Outside

0          100000      65536
```

---

# Consultar GID

```bash
cat /proc/self/gid_map
```

---

# Verificar desde el Host

Consultar el proceso.

```bash
ps -ef | grep podman
```

Luego:

```bash
cat /proc/PID/uid_map
```

---

# Directorios utilizados

## Rootless

Normalmente utiliza:

```text
$HOME/.local/share/containers
```

---

# Consultar

```bash
podman info \
--format '{{.Store.GraphRoot}}'
```

---

# Rootful

Utiliza:

```text
/var/lib/containers
```

---

# Consultar

```bash
sudo podman info \
--format '{{.Store.GraphRoot}}'
```

---

# Configuración

## Rootless

```text
$HOME/.config/containers
```

---

# Rootful

```text
/etc/containers
```

---

# Comparación

| Elemento | Rootless | Rootful |
|----------|----------|----------|
| Storage | HOME | /var/lib |
| Configuración | HOME | /etc |
| Permisos | Usuario | Root |
| Riesgo | Bajo | Alto |

---

# Verificar el usuario

```bash
whoami
```

---

# Verificar UID

```bash
id
```

---

# Ejecutar un contenedor Rootless

```bash
podman run \
-it \
ubi9
```

---

# Verificar

```bash
id
```

Dentro del contenedor:

```text
uid=0(root)
```

En el Host:

```bash
id
```

```text
uid=1000
```

---

# Ejecutar Rootful

```bash
sudo podman run \
-it \
ubi9
```

---

# Comparación visual

```text
Rootless

Host UID 1000

↓

Container UID 0

↓

Mapeado


Rootful

Host UID 0

↓

Container UID 0

↓

Sin traducción
```

---

# Ventajas de Rootless

- Mayor seguridad.
- Menor riesgo de escalación.
- No requiere privilegios administrativos.
- Ideal para estaciones de trabajo.
- Adecuado para múltiples usuarios.
- Reduce la superficie de ataque.
- Compatible con SELinux.

---

# Ventajas de Rootful

- Acceso completo al sistema.
- Redes privilegiadas.
- Menor cantidad de restricciones.
- Algunos servicios funcionan únicamente en este modo.
- Mayor control sobre el Host.

---

# Desventajas de Rootless

- Algunas funciones requieren configuraciones adicionales.
- Existen restricciones para ciertos puertos y dispositivos.
- Algunas cargas de trabajo avanzadas pueden necesitar Rootful.

---

# Desventajas de Rootful

- Mayor impacto de seguridad.
- Riesgo superior si un contenedor es comprometido.
- Requiere privilegios administrativos.
- Mayor responsabilidad en la administración del sistema.

---

# Casos de uso

| Escenario | Recomendación |
|------------|---------------|
| Desarrollo personal | Rootless |
| Laboratorios RHCSA | Rootless |
| Aplicaciones web | Rootless |
| Bases de datos pequeñas | Rootless |
| Servicios privilegiados | Rootful |
| Administración del sistema | Rootful |
| Contenedores de usuarios finales | Rootless |
| Multiusuario | Rootless |

---

# Laboratorio RHCSA

## Laboratorio 1

Consultar el usuario actual.

```bash
whoami
```

---

## Laboratorio 2

Consultar UID.

```bash
id
```

---

## Laboratorio 3

Consultar SubUID.

```bash
cat /etc/subuid
```

---

## Laboratorio 4

Consultar SubGID.

```bash
cat /etc/subgid
```

---

## Laboratorio 5

Consultar almacenamiento Rootless.

```bash
podman info \
--format '{{.Store.GraphRoot}}'
```

---

## Laboratorio 6

Ejecutar un contenedor Rootless.

```bash
podman run \
-it \
ubi9
```

---

## Laboratorio 7

Consultar el UID dentro del contenedor.

```bash
id
```

---

## Laboratorio 8

Salir del contenedor.

```bash
exit
```

---

## Laboratorio 9

Ejecutar el mismo contenedor como Root.

```bash
sudo podman run \
-it \
ubi9
```

---

## Laboratorio 10

Comparar ambos resultados.

---

# Buenas prácticas

- Utilizar Rootless siempre que sea posible.
- Limitar el uso de Rootful únicamente a servicios que realmente lo requieran.
- Mantener configurados correctamente los rangos de SubUID y SubGID.
- Supervisar los permisos otorgados a los usuarios que ejecutan contenedores.
- Aprovechar SELinux y los Namespaces como capas adicionales de seguridad.

---

# Errores comunes

## Error 1

Pensar que `uid=0` dentro del contenedor equivale a privilegios absolutos sobre el Host.

---

## Error 2

Eliminar o modificar incorrectamente los archivos `/etc/subuid` y `/etc/subgid`.

---

## Error 3

Ejecutar todos los contenedores como Root por costumbre, sin evaluar si realmente es necesario.

---

## Error 4

Confundir el almacenamiento y la configuración de Rootless con los de Rootful.

---

## Error 5

No comprender el funcionamiento del User Namespace y asumir que no existe aislamiento entre el Host y el contenedor.

---

# Resumen

En esta primera fase aprendimos:

- La diferencia entre Rootless y Rootful.
- Cómo funcionan los User Namespaces.
- El mapeo de UID y GID mediante SubUID y SubGID.
- La organización del almacenamiento y la configuración en ambos modos.
- Las ventajas y desventajas de cada enfoque.
- Cuándo utilizar Rootless y cuándo es apropiado utilizar Rootful.

En la **Fase 2** aprenderemos a administrar contenedores Rootless y Rootful en profundidad, configurar usuarios, comprender las limitaciones de puertos privilegiados, dispositivos, systemd, redes y almacenamiento, además de realizar laboratorios avanzados orientados al examen RHCSA.

----

# 74. Podman Rootless vs Rootful (Fase 2)

> **Módulo 10 – Contenedores con Podman**
>
> **Manual RHCSA**
>
> **Archivo:** `74-podman-rootless-rootful.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase podrás:

- Configurar y administrar contenedores Rootless.
- Comprender las limitaciones del modo Rootless.
- Administrar contenedores Rootful.
- Comprender el acceso a puertos privilegiados.
- Entender el acceso a dispositivos físicos.
- Configurar almacenamiento Rootless.
- Comprender la integración con systemd.
- Identificar cuándo utilizar cada modalidad en ambientes empresariales.

---

# Introducción

En la fase anterior aprendimos las diferencias conceptuales entre Rootless y Rootful.

Ahora veremos las diferencias operativas.

Muchas incidencias en producción ocurren porque un administrador intenta ejecutar un contenedor Rootless esperando exactamente el mismo comportamiento que uno Rootful.

Aunque ambos utilizan Podman, existen diferencias importantes.

---

# Verificar el modo de ejecución

Consultar el usuario:

```bash
whoami
```

Consultar UID:

```bash
id
```

Consultar almacenamiento:

```bash
podman info \
--format '{{.Store.GraphRoot}}'
```

---

# Verificar Rootless

Ejecutar:

```bash
podman info
```

Buscar:

```text
rootless: true
```

También:

```bash
podman info \
--format '{{.Host.Security.Rootless}}'
```

Resultado esperado:

```text
true
```

---

# Verificar Rootful

Como root:

```bash
sudo podman info \
--format '{{.Host.Security.Rootless}}'
```

Resultado:

```text
false
```

---

# Arquitectura

```text
                Host

                  │

      Usuario Normal

                  │

                  ▼

        Rootless Podman

                  │

                  ▼

         User Namespace

                  │

                  ▼

            Contenedor
```

---

# Arquitectura Rootful

```text
              Root

                │

                ▼

             Podman

                │

                ▼

           Contenedor

                │

                ▼

             Kernel
```

---

# Configuración Rootless

Directorio principal:

```text
$HOME/.config/containers
```

Consultar:

```bash
ls ~/.config/containers
```

Puede contener:

```text
containers.conf

registries.conf

policy.json
```

---

# Configuración Global

Consultar:

```text
/etc/containers
```

---

# Prioridad

```text
Configuración Usuario

↓

Configuración Global

↓

Valores por defecto
```

---

# Archivo containers.conf

Consultar:

```bash
cat ~/.config/containers/containers.conf
```

---

# Ejemplo

```toml
[containers]

netns="private"

ipcns="private"

utsns="private"
```

---

# Crear configuración personal

```bash
mkdir -p ~/.config/containers
```

---

# Copiar configuración global

```bash
cp \
/usr/share/containers/containers.conf \
~/.config/containers/
```

---

# Almacenamiento Rootless

Consultar:

```bash
podman info \
--format '{{.Store.GraphRoot}}'
```

Ejemplo

```text
/home/alejandro/.local/share/containers/storage
```

---

# Almacenamiento Rootful

Consultar:

```bash
sudo podman info \
--format '{{.Store.GraphRoot}}'
```

Resultado

```text
/var/lib/containers/storage
```

---

# Independencia

Un usuario Rootless NO comparte automáticamente:

- imágenes
- contenedores
- volúmenes

con el usuario root.

Ejemplo

Usuario:

```bash
podman images
```

Puede mostrar:

```text
nginx
ubi9
```

Mientras que:

```bash
sudo podman images
```

Puede mostrar:

```text
No images found
```

---

# Arquitectura

```text
Usuario

HOME

│

├── Images

├── Containers

└── Volumes


Root

│

├── Images

├── Containers

└── Volumes
```

---

# Acceso a puertos

Linux reserva determinados puertos.

```text
1-1023
```

Se conocen como:

```text
Privileged Ports
```

---

# Ejemplo

Intentar:

```bash
podman run \
-p 80:80 \
nginx
```

Puede generar un error en modo Rootless dependiendo de la configuración del sistema.

---

# Solución 1

Utilizar un puerto superior.

```bash
-p 8080:80
```

---

# Solución 2

Modificar el límite permitido por el kernel.

Consultar:

```bash
sysctl \
net.ipv4.ip_unprivileged_port_start
```

Ejemplo

```text
1024
```

---

# Cambiar temporalmente

```bash
sudo sysctl \
net.ipv4.ip_unprivileged_port_start=80
```

---

# Hacer permanente

Crear un archivo:

```text
/etc/sysctl.d/99-rootless.conf
```

Contenido:

```text
net.ipv4.ip_unprivileged_port_start=80
```

Aplicar:

```bash
sudo sysctl --system
```

> **Advertencia:** Reducir este valor permite que usuarios sin privilegios puedan enlazar puertos bajos. Evalúa el impacto de seguridad antes de aplicarlo en producción.

---

# Acceso a dispositivos

Modo Rootless:

Normalmente NO puede acceder libremente a:

```text
/dev/sda

/dev/kvm

/dev/fuse

/dev/ttyUSB0
```

---

# Rootful

Sí puede acceder cuando se asignan correctamente.

Ejemplo:

```bash
podman run \
--device /dev/sda \
imagen
```

---

# Casos de uso

| Dispositivo | Rootless | Rootful |
|--------------|----------|----------|
| Disco | Limitado | Sí |
| GPU | Limitado | Sí |
| USB | Limitado | Sí |
| Cámaras | Limitado | Sí |
| KVM | Limitado | Sí |

---

# Acceso a GPU

Ejemplo

```bash
podman run \
--device /dev/dri \
imagen
```

Normalmente requiere:

- permisos adecuados
- grupos correctos
- en muchos casos ejecución Rootful

---

# Acceso a directorios

Rootless únicamente puede escribir donde el usuario tenga permisos.

Ejemplo

Correcto:

```text
/home/alejandro
```

---

Incorrecto

```text
/root
```

---

# Ejemplo

```bash
podman run \
-v /root/data:/datos \
nginx
```

Fallará para un usuario sin privilegios.

---

# Solución

Utilizar:

```text
/home/usuario/datos
```

---

# Directorios recomendados

```text
~/containers

~/volumes

~/backups

~/projects
```

---

# Arquitectura

```text
HOME

│

├── containers

├── volumes

├── backup

└── logs
```

---

# Redes Rootless

Generalmente utilizan:

```text
slirp4netns
```

o

```text
pasta
```

Consultar:

```bash
podman info
```

Buscar:

```text
networkBackend
```

---

# Limitaciones

Comparado con Rootful:

- menor rendimiento
- algunas funciones avanzadas no están disponibles
- ciertas configuraciones requieren herramientas adicionales

---

# Servicios systemd

Una de las ventajas de Podman es su integración con systemd.

Generar una unidad:

```bash
podman generate systemd \
--name web
```

---

# Crear archivo

```bash
podman generate systemd \
--files \
--name web
```

---

# Rootless

Instalar la unidad en:

```text
~/.config/systemd/user/
```

---

# Recargar

```bash
systemctl --user daemon-reload
```

---

# Habilitar

```bash
systemctl --user enable container-web.service
```

---

# Iniciar

```bash
systemctl --user start container-web.service
```

---

# Consultar

```bash
systemctl --user status \
container-web.service
```

---

# Rootful

Ubicación:

```text
/etc/systemd/system/
```

---

# Recargar

```bash
sudo systemctl daemon-reload
```

---

# Iniciar

```bash
sudo systemctl start \
container-web.service
```

---

# Habilitar

```bash
sudo systemctl enable \
container-web.service
```

---

# Habilitar "Linger"

Si el usuario cierra sesión, los servicios Rootless pueden detenerse.

Consultar:

```bash
loginctl show-user $USER
```

Buscar:

```text
Linger=yes
```

---

# Habilitar

```bash
sudo loginctl enable-linger $USER
```

---

# Arquitectura

```text
Usuario

      │

systemd --user

      │

Container

      │

Podman
```

---

# Variables de entorno

Consultar:

```bash
env
```

Variables útiles:

```text
HOME

USER

XDG_RUNTIME_DIR
```

---

# Verificar Runtime

```bash
echo $XDG_RUNTIME_DIR
```

Ejemplo

```text
/run/user/1000
```

---

# Comparación completa

| Característica | Rootless | Rootful |
|----------------|----------|----------|
| Requiere root | No | Sí |
| Storage | HOME | /var/lib |
| Configuración | HOME | /etc |
| Puertos privilegiados | Limitados | Sí |
| Dispositivos | Limitados | Sí |
| Seguridad | Muy alta | Alta |
| Riesgo | Bajo | Mayor |
| Multiusuario | Excelente | Limitado |
| Integración systemd | `systemctl --user` | `systemctl` |

---

# Escenarios empresariales

## Desarrollo

```text
Developer

↓

Rootless

↓

Mayor Seguridad
```

---

## Producción

```text
Cluster

↓

Servicios

↓

Rootful

↓

Solo cuando sea necesario
```

---

## Laboratorios

```text
Usuario

↓

Rootless

↓

Sin afectar el sistema
```

---

# Laboratorio RHCSA

## Laboratorio 1

Verificar Rootless.

```bash
podman info \
--format '{{.Host.Security.Rootless}}'
```

---

## Laboratorio 2

Consultar Storage.

```bash
podman info \
--format '{{.Store.GraphRoot}}'
```

---

## Laboratorio 3

Consultar imágenes.

```bash
podman images
```

---

## Laboratorio 4

Consultar imágenes Rootful.

```bash
sudo podman images
```

---

## Laboratorio 5

Generar un servicio.

```bash
podman generate systemd \
--name web
```

---

## Laboratorio 6

Crear archivos.

```bash
podman generate systemd \
--files \
--name web
```

---

## Laboratorio 7

Recargar systemd.

```bash
systemctl --user daemon-reload
```

---

## Laboratorio 8

Consultar Runtime.

```bash
echo $XDG_RUNTIME_DIR
```

---

## Laboratorio 9

Consultar variables.

```bash
env
```

---

## Laboratorio 10

Consultar puertos.

```bash
sysctl \
net.ipv4.ip_unprivileged_port_start
```

---

## Laboratorio 11

Consultar directorio de configuración.

```bash
ls ~/.config/containers
```

---

## Laboratorio 12

Consultar configuración global.

```bash
ls /etc/containers
```

---

## Laboratorio 13

Consultar servicios del usuario.

```bash
systemctl --user list-units
```

---

## Laboratorio 14

Consultar linger.

```bash
loginctl show-user $USER
```

---

## Laboratorio 15

Habilitar linger.

```bash
sudo loginctl enable-linger $USER
```

---

# Buenas prácticas

- Utilizar Rootless como primera opción.
- Ejecutar Rootful únicamente cuando sea indispensable.
- Mantener configuraciones separadas para usuarios y sistema.
- Utilizar `systemctl --user` para servicios Rootless.
- Habilitar `linger` únicamente para usuarios que realmente necesiten ejecutar contenedores después de cerrar sesión.
- Evitar almacenar información sensible en directorios con permisos excesivos.
- Documentar claramente cuándo un servicio requiere privilegios elevados.

---

# Errores comunes

## Error 1

Esperar que las imágenes Rootless sean visibles para Rootful.

---

## Error 2

Intentar montar directorios sobre los cuales el usuario no tiene permisos.

---

## Error 3

Olvidar habilitar `linger` cuando se desea que un servicio Rootless permanezca activo tras cerrar la sesión.

---

## Error 4

Utilizar puertos privilegiados sin comprender las restricciones del modo Rootless.

---

## Error 5

Modificar parámetros globales del kernel para evitar una limitación puntual sin evaluar el impacto de seguridad.

---

# Resumen

En esta segunda fase aprendimos a:

- Verificar si Podman se ejecuta en modo Rootless o Rootful.
- Comprender la separación del almacenamiento y la configuración entre ambos modos.
- Administrar puertos privilegiados y conocer sus limitaciones.
- Trabajar con dispositivos físicos desde contenedores.
- Integrar contenedores con systemd tanto a nivel de usuario como del sistema.
- Configurar `linger` para mantener servicios Rootless activos.
- Aplicar buenas prácticas para administrar contenedores de forma segura y eficiente.

En la **Fase 3** profundizaremos en la administración avanzada de Rootless y Rootful, incluyendo capacidades Linux, permisos especiales, contenedores privilegiados, namespaces avanzados, cgroups, límites de recursos, troubleshooting y escenarios empresariales utilizados en el examen RHCSA.

----

# 74. Podman Rootless vs Rootful (Fase 3)

> **Módulo 10 – Contenedores con Podman**
>
> **Manual RHCSA**
>
> **Archivo:** `74-podman-rootless-rootful.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase podrás:

- Comprender las Linux Capabilities.
- Ejecutar contenedores privilegiados de forma segura.
- Administrar capacidades individuales.
- Comprender los Namespaces avanzados.
- Configurar límites mediante Cgroups.
- Controlar CPU, memoria y procesos.
- Comprender el aislamiento del Kernel.
- Diagnosticar problemas relacionados con Rootless y Rootful.
- Aplicar configuraciones utilizadas en ambientes empresariales.

---

# Introducción

Hasta este punto hemos aprendido que:

- Rootless ofrece mayor seguridad.
- Rootful proporciona mayor nivel de control.
- Ambos utilizan Namespaces y Cgroups.

Ahora profundizaremos en las tecnologías que hacen posible este aislamiento.

Estos conceptos son fundamentales para comprender cómo Linux protege el sistema operativo mientras ejecuta cientos de contenedores simultáneamente.

---

# ¿Qué son las Linux Capabilities?

Tradicionalmente, Linux sólo distinguía dos niveles de privilegios:

```text
Usuario Normal

o

Root
```

Actualmente el Kernel divide los privilegios de root en pequeños permisos independientes llamados:

```text
Linux Capabilities
```

---

# Arquitectura

```text
              Root

                │

                ▼

      Linux Capabilities

     ┌──────┼────────┐

     ▼      ▼        ▼

NET_ADMIN SYS_TIME SYS_ADMIN
```

---

# Beneficio

En lugar de entregar todos los privilegios de root, podemos otorgar únicamente los necesarios.

Este enfoque sigue el principio de:

```text
Least Privilege
```

---

# Consultar Capabilities

Dentro del contenedor:

```bash
cat /proc/self/status
```

Buscar:

```text
CapEff

CapPrm

CapBnd
```

---

# Utilizar capsh

Instalar:

```bash
dnf install libcap
```

Consultar:

```bash
capsh --print
```

---

# Algunas Capabilities importantes

| Capability | Función |
|------------|----------|
| CAP_NET_ADMIN | Administración de red |
| CAP_NET_RAW | Paquetes RAW |
| CAP_SYS_TIME | Cambiar hora |
| CAP_SYS_ADMIN | Administración avanzada |
| CAP_CHOWN | Cambiar propietario |
| CAP_SETUID | Cambiar UID |
| CAP_SETGID | Cambiar GID |
| CAP_MKNOD | Crear dispositivos |
| CAP_DAC_OVERRIDE | Ignorar permisos DAC |

---

# Contenedor Privilegiado

Ejemplo:

```bash
podman run \
--privileged \
-it \
ubi9
```

---

# ¿Qué significa?

El contenedor recibe prácticamente todos los privilegios disponibles.

```text
Container

↓

Acceso completo

↓

Kernel
```

---

# Riesgos

Un contenedor privilegiado:

- incrementa la superficie de ataque
- reduce el aislamiento
- puede acceder a más recursos del Host
- debe utilizarse únicamente cuando sea imprescindible

---

# Buenas prácticas

Evitar:

```bash
--privileged
```

Siempre que sea posible.

---

# Agregar una Capability

Ejemplo:

```bash
podman run \
--cap-add NET_ADMIN \
ubi9
```

---

# Eliminar una Capability

```bash
podman run \
--cap-drop NET_RAW \
ubi9
```

---

# Agregar múltiples

```bash
podman run \
--cap-add NET_ADMIN \
--cap-add SYS_TIME \
ubi9
```

---

# Eliminar múltiples

```bash
podman run \
--cap-drop NET_RAW \
--cap-drop MKNOD \
ubi9
```

---

# Arquitectura

```text
Container

      │

Capabilities

      │

      ▼

Kernel
```

---

# ¿Cuándo utilizar Capabilities?

En lugar de:

```text
Privileged
```

es preferible:

```text
Agregar únicamente
la Capability necesaria.
```

---

# Namespaces

Linux utiliza distintos Namespaces para aislar recursos.

Los principales son:

- PID
- Network
- Mount
- IPC
- UTS
- User
- Cgroup
- Time (Kernel recientes)

---

# PID Namespace

Aísla los procesos.

Dentro del contenedor:

```bash
ps aux
```

Únicamente observará los procesos del propio contenedor.

---

# Arquitectura

```text
Host

1000 procesos

        │

        ▼

PID Namespace

        │

        ▼

Container

12 procesos
```

---

# Network Namespace

Cada contenedor posee su propia pila de red.

Consultar:

```bash
ip addr
```

---

# Mount Namespace

Cada contenedor visualiza únicamente sus propios sistemas de archivos montados.

Consultar:

```bash
mount
```

---

# UTS Namespace

Permite utilizar un hostname independiente.

Consultar:

```bash
hostname
```

Cambiar:

```bash
hostname servidor-web
```

---

# IPC Namespace

Aísla:

- memoria compartida
- semáforos
- colas de mensajes

Consultar:

```bash
ipcs
```

---

# Cgroup Namespace

Permite controlar los recursos disponibles.

---

# Cgroups

Los Control Groups limitan recursos del sistema.

Pueden controlar:

- CPU
- Memoria
- Procesos
- Disco
- I/O

---

# Arquitectura

```text
Kernel

      │

Cgroups

      │

Container
```

---

# Limitar Memoria

Ejemplo:

```bash
podman run \
--memory 512m \
ubi9
```

---

# Limitar CPU

```bash
podman run \
--cpus 2 \
ubi9
```

---

# Limitar Procesos

```bash
podman run \
--pids-limit 200 \
ubi9
```

---

# Limitar Swap

```bash
podman run \
--memory 1g \
--memory-swap 2g \
ubi9
```

---

# Verificar límites

Dentro del contenedor

Consultar memoria:

```bash
cat /sys/fs/cgroup/memory.max
```

Consultar procesos:

```bash
cat /sys/fs/cgroup/pids.max
```

---

# Recursos disponibles

Consultar:

```bash
podman stats
```

Ejemplo

```text
CPU %

MEM USAGE

NET I/O

BLOCK I/O
```

---

# Monitoreo continuo

```bash
podman stats
```

o

```bash
podman stats --no-stream
```

---

# Arquitectura Empresarial

```text
              Servidor

                   │

       ┌───────────┼───────────┐

       ▼           ▼           ▼

 PostgreSQL     Nginx      Redis

       │           │           │

   Cgroups     Cgroups     Cgroups

       │           │           │

       └───────────┼───────────┘

                   ▼

                 Kernel
```

---

# Compartir Namespace

Ejemplo:

```bash
podman run \
--pid=host \
ubi9
```

---

# Advertencia

El contenedor visualizará procesos del Host.

Debe utilizarse únicamente cuando sea necesario.

---

# Compartir Red

```bash
--network host
```

---

# Compartir IPC

```bash
--ipc host
```

---

# Compartir UTS

```bash
--uts host
```

---

# Compartir Cgroup

```bash
--cgroupns host
```

---

# ¿Cuándo compartir?

Solo cuando exista una necesidad técnica claramente justificada.

En producción se recomienda mantener el mayor aislamiento posible.

---

# Rootless y Cgroups

Consultar:

```bash
podman info
```

Buscar:

```text
cgroupManager
```

Ejemplo:

```text
systemd
```

---

# Rootful

También puede utilizar:

```text
systemd
```

o

```text
cgroupfs
```

---

# Recursos recomendados

| Servicio | CPU | Memoria |
|----------|------|----------|
| Nginx | 1 | 256 MB |
| Redis | 1 | 512 MB |
| PostgreSQL | 2 | 2 GB |
| MariaDB | 2 | 2 GB |
| MongoDB | 2 | 2 GB |

---

# Escenarios Empresariales

## Servidor Web

```text
Rootless

↓

Nginx

↓

512 MB

↓

1 CPU
```

---

## PostgreSQL

```text
Rootful

↓

Volumen Persistente

↓

4 GB RAM

↓

2 CPU
```

---

## Redis

```text
Rootless

↓

256 MB

↓

1 CPU
```

---

# Diagnóstico

Consultar:

```bash
podman inspect web
```

Buscar:

```text
HostConfig
```

---

# Consultar estadísticas

```bash
podman stats
```

---

# Consultar procesos

```bash
podman top web
```

---

# Consultar logs

```bash
podman logs web
```

---

# Laboratorio RHCSA

## Laboratorio 1

Consultar estadísticas.

```bash
podman stats
```

---

## Laboratorio 2

Ejecutar con límite de memoria.

```bash
podman run \
--memory 512m \
ubi9
```

---

## Laboratorio 3

Ejecutar con límite de CPU.

```bash
podman run \
--cpus 2 \
ubi9
```

---

## Laboratorio 4

Consultar Cgroups.

```bash
podman info
```

---

## Laboratorio 5

Consultar Capabilities.

```bash
cat /proc/self/status
```

---

## Laboratorio 6

Agregar NET_ADMIN.

```bash
podman run \
--cap-add NET_ADMIN \
ubi9
```

---

## Laboratorio 7

Eliminar NET_RAW.

```bash
podman run \
--cap-drop NET_RAW \
ubi9
```

---

## Laboratorio 8

Consultar procesos.

```bash
podman top
```

---

## Laboratorio 9

Consultar logs.

```bash
podman logs
```

---

## Laboratorio 10

Consultar estadísticas sin monitoreo continuo.

```bash
podman stats --no-stream
```

---

## Laboratorio 11

Consultar hostname.

```bash
hostname
```

---

## Laboratorio 12

Consultar interfaces.

```bash
ip addr
```

---

## Laboratorio 13

Consultar sistemas montados.

```bash
mount
```

---

## Laboratorio 14

Consultar procesos.

```bash
ps aux
```

---

## Laboratorio 15

Ejecutar un contenedor privilegiado únicamente en un laboratorio y comparar la salida de:

```bash
capsh --print
```

con la de un contenedor sin privilegios.

---

# Buenas prácticas

- Preferir agregar capacidades específicas (`--cap-add`) antes que utilizar `--privileged`.
- Definir límites de memoria y CPU para todas las cargas de trabajo en producción.
- Mantener aislados los Namespaces siempre que sea posible.
- Supervisar periódicamente el consumo de recursos mediante `podman stats`.
- Utilizar Rootless para aplicaciones que no requieran privilegios especiales.
- Documentar las Capabilities adicionales concedidas a cada contenedor.

---

# Errores comunes

## Error 1

Ejecutar todos los contenedores con `--privileged`.

---

## Error 2

No establecer límites de memoria, permitiendo que un contenedor consuma todos los recursos del servidor.

---

## Error 3

Compartir el Namespace del Host sin una necesidad técnica real.

---

## Error 4

Conceder `CAP_SYS_ADMIN` cuando bastaría una Capability mucho más específica.

---

## Error 5

Asumir que Rootless elimina la necesidad de supervisar el consumo de CPU y memoria.

---

# Resumen

En esta tercera fase aprendimos a:

- Comprender el funcionamiento de las Linux Capabilities.
- Agregar y eliminar capacidades específicas para mejorar la seguridad.
- Identificar los riesgos asociados a los contenedores privilegiados.
- Comprender los distintos Namespaces utilizados por el Kernel Linux.
- Configurar límites de CPU, memoria y procesos mediante Cgroups.
- Supervisar el consumo de recursos con `podman stats`.
- Aplicar buenas prácticas de aislamiento y administración de recursos utilizadas en entornos empresariales.

En la **Fase 4** integraremos todos estos conceptos mediante escenarios reales de producción, resolución de problemas, auditorías de seguridad, scripts de diagnóstico, checklist RHCSA, preguntas de repaso y un desafío final similar al examen oficial.

----

# 74. Podman Rootless vs Rootful (Fase 4)

> **Módulo 10 – Contenedores con Podman**
>
> **Manual RHCSA**
>
> **Archivo:** `74-podman-rootless-rootful.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase podrás:

- Diagnosticar problemas relacionados con Rootless y Rootful.
- Resolver incidencias de permisos.
- Diagnosticar problemas de almacenamiento y redes.
- Auditar la seguridad de los contenedores.
- Automatizar revisiones mediante scripts.
- Aplicar buenas prácticas empresariales.
- Resolver escenarios similares al examen RHCSA.

---

# Metodología de Diagnóstico

Cuando un contenedor presenta problemas, nunca debes comenzar modificando la configuración al azar.

Siempre sigue un procedimiento estructurado.

```text
             Contenedor falla

                    │

                    ▼

         ¿Rootless o Rootful?

                    │

                    ▼

         ¿Usuario correcto?

                    │

                    ▼

      ¿Permisos suficientes?

                    │

                    ▼

     ¿SELinux permite acceso?

                    │

                    ▼

       ¿Volúmenes correctos?

                    │

                    ▼

      ¿Red correctamente creada?

                    │

                    ▼

     ¿Capabilities suficientes?

                    │

                    ▼

        Resolver incidencia
```

---

# Checklist Inicial

Siempre consultar:

```bash
whoami
```

↓

```bash
id
```

↓

```bash
podman info
```

↓

```bash
podman ps
```

↓

```bash
podman inspect
```

↓

```bash
podman logs
```

↓

```bash
podman stats
```

↓

```bash
journalctl
```

---

# Escenario 1

## El contenedor no inicia

Consultar:

```bash
podman ps -a
```

Revisar estado.

Consultar:

```bash
podman logs web
```

---

# Posibles causas

- Imagen incorrecta
- Parámetros inválidos
- Variables de entorno
- Volumen inexistente
- Permisos

---

# Escenario 2

## Error Permission Denied

Consultar:

```bash
ls -l
```

Consultar:

```bash
ls -lZ
```

Consultar:

```bash
id
```

Verificar:

- propietario
- permisos
- contexto SELinux

---

# Escenario 3

## Rootless no puede acceder al directorio

Ejemplo

```text
/root/data
```

Rootless no posee permisos.

Debe utilizar:

```text
/home/usuario
```

---

# Escenario 4

## Rootless no publica el puerto 80

Consultar:

```bash
sysctl \
net.ipv4.ip_unprivileged_port_start
```

Resultado:

```text
1024
```

Solución:

Utilizar

```bash
8080
```

o modificar el parámetro del kernel de forma consciente y documentada.

---

# Escenario 5

## Imágenes desaparecieron

Consultar:

```bash
podman images
```

Luego:

```bash
sudo podman images
```

Recordar:

Rootless y Rootful poseen almacenamientos independientes.

---

# Escenario 6

## Volumen vacío

Consultar:

```bash
podman inspect web
```

↓

Buscar:

```text
Mounts
```

↓

Consultar:

```bash
podman volume inspect
```

---

# Escenario 7

## SELinux bloquea

Consultar:

```bash
ls -lZ
```

Si el problema es un Bind Mount:

Utilizar:

```text
:Z
```

o

```text
:z
```

---

# Escenario 8

## Rootless no puede utilizar un dispositivo

Ejemplo:

```text
/dev/kvm
```

Consultar:

```bash
ls -l /dev/kvm
```

Verificar:

- permisos
- grupos
- necesidad real de Rootful

---

# Escenario 9

## Alto consumo de memoria

Consultar:

```bash
podman stats
```

↓

Configurar:

```bash
--memory
```

---

# Escenario 10

## Alto consumo de CPU

Consultar:

```bash
podman stats
```

↓

Limitar:

```bash
--cpus
```

---

# Escenario 11

## Demasiados procesos

Consultar:

```bash
podman top web
```

Configurar:

```bash
--pids-limit
```

---

# Escenario 12

## Servicio Rootless deja de funcionar después de cerrar sesión

Consultar:

```bash
loginctl show-user $USER
```

Buscar:

```text
Linger=yes
```

Si aparece:

```text
no
```

Configurar:

```bash
sudo loginctl enable-linger $USER
```

---

# Escenario 13

## Problemas con systemd

Consultar:

```bash
systemctl --user status
```

o

```bash
sudo systemctl status
```

Según el modo utilizado.

---

# Escenario 14

## Problemas de almacenamiento

Consultar:

```bash
podman system df
```

↓

```bash
df -h
```

↓

```bash
df -i
```

---

# Escenario 15

## Problemas de red

Consultar:

```bash
podman network ls
```

↓

```bash
podman network inspect
```

↓

```bash
podman inspect
```

---

# Flujo recomendado de diagnóstico

```text
Contenedor

      │

      ▼

Logs

      │

      ▼

Inspect

      │

      ▼

Storage

      │

      ▼

Red

      │

      ▼

SELinux

      │

      ▼

Capabilities

      │

      ▼

Solución
```

---

# Herramientas de Diagnóstico

## Podman

```bash
podman ps
```

```bash
podman ps -a
```

```bash
podman logs
```

```bash
podman inspect
```

```bash
podman stats
```

```bash
podman top
```

```bash
podman info
```

```bash
podman images
```

```bash
podman network ls
```

```bash
podman volume ls
```

---

## Linux

```bash
id
```

```bash
whoami
```

```bash
journalctl
```

```bash
ss
```

```bash
ip addr
```

```bash
mount
```

```bash
lsblk
```

```bash
findmnt
```

```bash
df -h
```

```bash
df -i
```

```bash
getenforce
```

```bash
sestatus
```

---

# Script de Auditoría Rootless / Rootful

Guardar como:

```text
podman_security_audit.sh
```

```bash
#!/bin/bash

echo "======================================"
echo " PODMAN SECURITY AUDIT"
echo "======================================"

echo
echo "Usuario:"
whoami

echo
echo "UID:"
id

echo
echo "Modo Rootless:"
podman info --format '{{.Host.Security.Rootless}}'

echo
echo "Storage:"
podman info --format '{{.Store.GraphRoot}}'

echo
echo "Contenedores:"
podman ps

echo
echo "Imágenes:"
podman images

echo
echo "Volúmenes:"
podman volume ls

echo
echo "Redes:"
podman network ls

echo
echo "SELinux:"
getenforce

echo
echo "Espacio:"
df -h

echo
echo "Inodos:"
df -i
```

Permisos:

```bash
chmod +x podman_security_audit.sh
```

---

# Script para revisar límites

Guardar como:

```text
podman_resources.sh
```

```bash
#!/bin/bash

echo "=============================="
echo " Recursos Podman"
echo "=============================="

podman stats --no-stream

echo

podman system df

echo

free -h

echo

nproc
```

---

# Script para auditar contenedores

```bash
#!/bin/bash

for c in $(podman ps -q)
do

echo
echo "=========================="
echo "$c"
echo "=========================="

podman inspect "$c"

done
```

---

# Arquitectura Empresarial

```text
                Usuarios

                     │

         ┌───────────┴────────────┐

         ▼                        ▼

   Rootless Users           Administradores

         │                        │

         ▼                        ▼

    Rootless Pods          Rootful Pods

         │                        │

         └──────────┬─────────────┘

                    ▼

               Fedora Kernel

                    │

                    ▼

                 SELinux
```

---

# Modelo recomendado

```text
                  Internet

                      │

                Reverse Proxy

                      │

                 Rootless Web

                      │

             API Rootless

                      │

          PostgreSQL Rootful

                      │

          Storage Persistente
```

---

# Recomendaciones Empresariales

## Desarrollo

```text
Rootless
```

---

## Laboratorios

```text
Rootless
```

---

## CI/CD

```text
Rootless
```

---

## Bases de Datos

```text
Rootless

↓

Siempre que sea posible.

Rootful

↓

Solo cuando exista
un requerimiento técnico.
```

---

## Hardware especializado

```text
GPU

↓

Rootful
```

---

# Laboratorio RHCSA

## Laboratorio 1

Consultar Rootless.

```bash
podman info \
--format '{{.Host.Security.Rootless}}'
```

---

## Laboratorio 2

Consultar imágenes.

```bash
podman images
```

---

## Laboratorio 3

Consultar almacenamiento.

```bash
podman info \
--format '{{.Store.GraphRoot}}'
```

---

## Laboratorio 4

Consultar estadísticas.

```bash
podman stats
```

---

## Laboratorio 5

Consultar procesos.

```bash
podman top web
```

---

## Laboratorio 6

Consultar logs.

```bash
podman logs web
```

---

## Laboratorio 7

Consultar volúmenes.

```bash
podman volume ls
```

---

## Laboratorio 8

Consultar redes.

```bash
podman network ls
```

---

## Laboratorio 9

Consultar SELinux.

```bash
getenforce
```

---

## Laboratorio 10

Consultar espacio.

```bash
df -h
```

---

## Laboratorio 11

Consultar inodos.

```bash
df -i
```

---

## Laboratorio 12

Ejecutar auditoría.

```bash
./podman_security_audit.sh
```

---

## Laboratorio 13

Ejecutar auditoría de recursos.

```bash
./podman_resources.sh
```

---

## Laboratorio 14

Consultar servicios Rootless.

```bash
systemctl --user list-units
```

---

## Laboratorio 15

Comparar:

```bash
podman info
```

contra

```bash
sudo podman info
```

Analizar:

- Storage
- Rootless
- GraphRoot
- Configuración
- Imágenes
- Volúmenes

---

# Checklist RHCSA

```text
□ Comprendo la diferencia entre Rootless y Rootful.

□ Sé consultar SubUID y SubGID.

□ Puedo identificar el GraphRoot.

□ Conozco dónde se almacenan imágenes y volúmenes.

□ Comprendo las restricciones de puertos privilegiados.

□ Sé configurar servicios Rootless con systemd.

□ Comprendo el funcionamiento de Linux Capabilities.

□ Sé agregar y eliminar Capabilities.

□ Comprendo los Namespaces del Kernel.

□ Sé limitar CPU y memoria mediante Cgroups.

□ Puedo interpretar podman inspect.

□ Sé interpretar podman stats.

□ Comprendo el funcionamiento de SELinux.

□ Sé diagnosticar problemas de permisos.

□ Puedo ejecutar auditorías de seguridad.
```

---

# Preguntas de Repaso

1. ¿Cuál es la principal ventaja de Rootless?
2. ¿Qué son los User Namespaces?
3. ¿Qué función cumplen los archivos `/etc/subuid` y `/etc/subgid`?
4. ¿Por qué Rootless utiliza un almacenamiento diferente al de Rootful?
5. ¿Qué comando indica si Podman se está ejecutando en modo Rootless?
6. ¿Qué diferencia existe entre `--cap-add` y `--privileged`?
7. ¿Qué recurso controla un Cgroup?
8. ¿Qué Namespace aísla los procesos?
9. ¿Qué comando muestra el consumo de recursos de los contenedores?
10. ¿Qué utilidad tiene `loginctl enable-linger`?
11. ¿Qué ocurre si un usuario Rootless intenta montar un directorio para el cual no tiene permisos?
12. ¿Qué problema suele resolver la opción `:Z` en un Bind Mount?
13. ¿Qué comando muestra el almacenamiento utilizado por imágenes, contenedores y volúmenes?
14. ¿Cuándo es recomendable utilizar Rootful?
15. ¿Qué herramientas utilizarías para investigar un contenedor que no inicia?
16. ¿Por qué no es recomendable ejecutar todos los contenedores con `--privileged`?
17. ¿Qué ventaja aporta limitar CPU y memoria en producción?
18. ¿Cómo verificarías si SELinux está en modo Enforcing?
19. ¿Qué diferencia existe entre `systemctl` y `systemctl --user`?
20. ¿Qué pasos seguirías para diagnosticar un problema de permisos en un contenedor Rootless?

---

# Respuestas

1. Reduce el riesgo de seguridad al ejecutar contenedores sin privilegios de root en el Host.
2. Permiten mapear usuarios del Host a identidades diferentes dentro del contenedor.
3. Definen los rangos de UID y GID utilizados por los User Namespaces.
4. Para mantener el aislamiento entre los recursos del usuario y los del sistema.
5. `podman info --format '{{.Host.Security.Rootless}}'`
6. `--cap-add` concede únicamente capacidades específicas; `--privileged` concede prácticamente todos los privilegios disponibles.
7. CPU, memoria, procesos, I/O y otros recursos del sistema.
8. El PID Namespace.
9. `podman stats`
10. Permite que los servicios Rootless continúen ejecutándose incluso después de cerrar la sesión del usuario.
11. El montaje fallará por falta de permisos.
12. Resolver problemas de contexto SELinux al compartir directorios con contenedores.
13. `podman system df`
14. Cuando una carga de trabajo requiere acceso privilegiado al sistema o a dispositivos específicos.
15. `podman ps -a`, `podman logs`, `podman inspect` y `journalctl`.
16. Porque reduce el aislamiento y aumenta significativamente la superficie de ataque.
17. Evita que un contenedor consuma todos los recursos del servidor y afecte a otros servicios.
18. Con `getenforce` o `sestatus`.
19. `systemctl` administra servicios del sistema; `systemctl --user` administra servicios del usuario.
20. Verificar usuario, permisos Linux, contexto SELinux, volumen montado, Capabilities y registros del contenedor.

---

# Desafío Final RHCSA

Dispones de un servidor Fedora con Podman instalado.

Realiza las siguientes tareas:

1. Verificar si Podman se ejecuta en modo Rootless.
2. Consultar el GraphRoot del usuario actual.
3. Crear un contenedor Rootless utilizando una imagen UBI.
4. Verificar el mapeo de UID dentro del contenedor.
5. Configurar un volumen persistente para el contenedor.
6. Generar una unidad `systemd` para administrarlo.
7. Habilitar el servicio utilizando `systemctl --user`.
8. Activar `linger` para que el servicio continúe ejecutándose después de cerrar sesión.
9. Limitar el contenedor a **1 CPU**, **512 MB** de memoria y **200 procesos**.
10. Crear un segundo contenedor agregando únicamente la Capability `NET_ADMIN`.
11. Comparar la salida de `capsh --print` entre ambos contenedores.
12. Ejecutar el script `podman_security_audit.sh`.
13. Analizar los resultados y documentar las diferencias entre Rootless y Rootful.
14. Elaborar una recomendación justificando cuándo utilizar cada modalidad en un entorno empresarial.

---

# Buenas prácticas

- Utilizar Rootless como opción predeterminada.
- Reservar Rootful únicamente para cargas de trabajo que realmente lo requieran.
- Evitar el uso de `--privileged`; conceder únicamente las Capabilities necesarias.
- Definir límites de CPU, memoria y procesos para todos los contenedores de producción.
- Revisar periódicamente los permisos de usuarios, volúmenes y dispositivos.
- Mantener SELinux en modo **Enforcing** siempre que sea posible.
- Automatizar auditorías de seguridad y consumo de recursos.
- Documentar cualquier excepción que requiera privilegios elevados.

---

# Resumen del Capítulo 74

En este capítulo aprendimos a:

- Comprender las diferencias entre Podman Rootless y Rootful.
- Administrar User Namespaces, SubUID y SubGID.
- Trabajar con almacenamiento y configuración específicos para cada modo.
- Configurar servicios mediante `systemd` y `systemctl --user`.
- Comprender las Linux Capabilities y el impacto de los contenedores privilegiados.
- Utilizar Cgroups para controlar CPU, memoria y procesos.
- Diagnosticar problemas relacionados con permisos, SELinux, almacenamiento y redes.
- Automatizar auditorías mediante scripts Bash.
- Aplicar prácticas recomendadas para implementar contenedores seguros y eficientes en entornos empresariales y prepararse para el examen **RHCSA**.

---

# Fin del capítulo
