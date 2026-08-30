# 69. Instalación y Configuración de Podman (Fase 3)

> **Módulo 10 – Contenedores con Podman**
>
> **Archivo:** `69-instalacion-configuracion-podman.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase podrás:

- Comprender el funcionamiento de User Namespaces.
- Configurar correctamente SubUID y SubGID.
- Comprender el funcionamiento de Rootless Networking.
- Comprender el almacenamiento Rootless.
- Integrar correctamente SELinux con Podman.
- Diagnosticar problemas de permisos.
- Comprender el firewall aplicado a contenedores.
- Diagnosticar errores comunes.
- Resolver problemas relacionados con permisos y almacenamiento.
- Auditar una instalación de Podman.

---

# Introducción

Una de las principales diferencias entre Podman y otros motores de contenedores es el soporte completo para **Rootless Containers**.

Esta característica permite ejecutar contenedores sin privilegios administrativos, aumentando significativamente la seguridad del sistema.

Para lograrlo, Linux utiliza varias tecnologías:

- User Namespaces
- SubUID
- SubGID
- SELinux
- OverlayFS
- Netavark
- Aardvark DNS
- Cgroups
- Capabilities

En esta fase estudiaremos cada una de ellas.

---

# 77. ¿Qué es Rootless?

Un contenedor Rootless es un contenedor ejecutado por un usuario normal.

Ejemplo:

```bash
podman run \
--rm \
docker.io/library/alpine:latest \
id
```

No fue necesario utilizar:

```bash
sudo
```

El usuario administra únicamente sus propios contenedores.

---

# 78. Rootful vs Rootless

```text
                    Rootful

Usuario
   │
 sudo
   │
Podman
   │
Contenedor


                    Rootless

Usuario
   │
Podman
   │
Contenedor
```

---

# 79. Ventajas del modo Rootless

Entre las principales ventajas:

- No requiere privilegios administrativos.
- Reduce la superficie de ataque.
- Aísla los contenedores por usuario.
- Mejor integración con escritorios Linux.
- Compatible con systemd de usuario.
- Reduce riesgos de escalamiento de privilegios.

---

# 80. User Namespaces

El User Namespace permite mapear usuarios del contenedor con usuarios distintos del sistema anfitrión.

Ejemplo conceptual:

```text
Dentro del contenedor

UID 0
(root)

        │

        ▼

Host

UID 1000
(alejandro)
```

Desde el interior parece ser root.

En realidad no posee privilegios administrativos sobre el sistema anfitrión.

---

# 81. Visualizando el mapeo

```text
Contenedor

UID 0

UID 1

UID 2

UID 3

...

        │

        ▼

Host

100000

100001

100002

100003
```

Los UID reales pertenecen al rango SubUID.

---

# 82. SubUID

Archivo:

```text
/etc/subuid
```

Consultar:

```bash
cat /etc/subuid
```

Ejemplo:

```text
alejandro:100000:65536
```

Interpretación:

| Campo | Valor |
|---------|--------|
| Usuario | alejandro |
| UID inicial | 100000 |
| Cantidad | 65536 |

---

# 83. SubGID

Archivo:

```text
/etc/subgid
```

Consultar:

```bash
cat /etc/subgid
```

Ejemplo:

```text
alejandro:100000:65536
```

---

# 84. Consultar ambos archivos

```bash
grep "^$(whoami)" \
/etc/subuid \
/etc/subgid
```

---

# 85. ¿Por qué existen?

Sin User Namespace:

```text
Root del contenedor

↓

Root del host
```

Con User Namespace:

```text
Root del contenedor

↓

UID sin privilegios
```

La diferencia es enorme desde el punto de vista de la seguridad.

---

# 86. Crear rangos manualmente

Si fuese necesario:

```text
usuario:100000:65536
```

Después puede ser necesario reiniciar la sesión.

Normalmente estos rangos son creados automáticamente.

---

# 87. Comprobar User Namespace

```bash
podman info \
--format '{{.Host.Security.Rootless}}'
```

---

# 88. Root dentro del contenedor

Entrar:

```bash
podman run \
-it \
--rm \
docker.io/library/alpine:latest \
/bin/sh
```

Dentro:

```bash
id
```

Puede mostrar:

```text
uid=0(root)
```

Eso **NO significa** que sea root del sistema anfitrión.

---

# 89. Root del host

En otra terminal:

```bash
id
```

Resultado:

```text
uid=1000(alejandro)
```

Es el usuario real.

---

# 90. Almacenamiento Rootless

Consultar:

```bash
podman info \
--format '{{.Store.GraphRoot}}'
```

Normalmente:

```text
~/.local/share/containers/storage
```

---

# 91. Ver directorios

```bash
tree \
-L 2 \
~/.local/share/containers
```

---

# 92. Directorio Overlay

Consultar:

```bash
tree \
-L 2 \
~/.local/share/containers/storage/overlay
```

Aquí residen las capas.

---

# 93. Rootful Storage

Consultar:

```bash
sudo podman info \
--format '{{.Store.GraphRoot}}'
```

Generalmente:

```text
/var/lib/containers/storage
```

---

# 94. Diferencias

| Rootless | Rootful |
|-----------|----------|
| HOME | /var/lib |
| Usuario | Root |
| Sin sudo | sudo |
| Más seguro | Mayor privilegio |

---

# 95. Rootless Networking

Consultar redes:

```bash
podman network ls
```

---

# 96. Netavark

Consultar:

```bash
rpm -q netavark
```

Netavark administra la red de Podman.

---

# 97. Aardvark DNS

Consultar:

```bash
rpm -q aardvark-dns
```

Permite resolución DNS entre contenedores.

---

# 98. Interfaces

Mostrar:

```bash
ip link
```

Observar nuevas interfaces después de iniciar un contenedor.

---

# 99. Redes disponibles

```bash
podman network ls
```

Ejemplo:

```text
podman
```

---

# 100. Inspeccionar una red

```bash
podman network inspect podman
```

---

# 101. Firewall

Consultar:

```bash
firewall-cmd \
--list-all
```

---

# 102. Abrir un puerto

Ejemplo:

```bash
sudo firewall-cmd \
--add-port=8080/tcp
```

Permanente:

```bash
sudo firewall-cmd \
--add-port=8080/tcp \
--permanent
```

Recargar:

```bash
sudo firewall-cmd --reload
```

---

# 103. Verificar puertos

```bash
ss -lnt
```

También:

```bash
podman port nombre_contenedor
```

---

# 104. SELinux y Podman

Estado:

```bash
getenforce
```

Debe indicar:

```text
Enforcing
```

---

# 105. Contextos SELinux

Consultar:

```bash
ps -eZ
```

Buscar:

```bash
grep container
```

---

# 106. Etiquetas

Consultar:

```bash
ls -Zd \
~/.local/share/containers
```

---

# 107. Problema típico

Montar:

```bash
-v /datos:/datos
```

Puede producir:

```text
Permission denied
```

---

# 108. Solución

Agregar:

```bash
:Z
```

o

```bash
:z
```

Ejemplo:

```bash
-v /datos:/datos:Z
```

---

# 109. Diferencias

| :z | :Z |
|-----|----|
| Compartido | Exclusivo |

---

# 110. Restaurar contexto

```bash
restorecon \
-Rv /datos
```

---

# 111. Buscar errores SELinux

```bash
ausearch \
-m AVC \
-ts recent
```

---

# 112. Auditoría

También:

```bash
journalctl \
-t setroubleshoot
```

---

# 113. Capabilities

Consultar:

```bash
podman inspect nombre
```

Buscar:

```text
Capabilities
```

---

# 114. Contenedor privilegiado

Ejemplo:

```bash
podman run \
--privileged
```

Debe evitarse salvo necesidad.

---

# 115. Limitar memoria

```bash
podman run \
--memory 512m
```

---

# 116. Limitar CPU

```bash
podman run \
--cpus 2
```

---

# 117. Ver consumo

```bash
podman stats
```

---

# 118. Diagnóstico inicial

Consultar:

```bash
podman info
```

---

# 119. Diagnóstico de imágenes

```bash
podman images
```

---

# 120. Diagnóstico de contenedores

```bash
podman ps -a
```

---

# 121. Diagnóstico de red

```bash
podman network ls
```

---

# 122. Diagnóstico de almacenamiento

```bash
podman system df
```

---

# 123. Diagnóstico de eventos

```bash
podman events
```

---

# 124. Diagnóstico de procesos

```bash
podman top nombre
```

---

# 125. Diagnóstico completo

```bash
podman info
```

Debe comprobarse:

- Runtime
- Driver
- Rootless
- GraphRoot
- RunRoot
- Cgroups
- Kernel
- SELinux

---

# 126. Script de auditoría

```bash
#!/bin/bash

echo "========== PODMAN =========="

podman --version

echo

echo "========== INFO =========="

podman info

echo

echo "========== IMAGES =========="

podman images

echo

echo "========== CONTAINERS =========="

podman ps -a

echo

echo "========== NETWORK =========="

podman network ls

echo

echo "========== VOLUMES =========="

podman volume ls

echo

echo "========== STORAGE =========="

podman system df
```

Guardar como:

```text
audit-podman.sh
```

Dar permisos:

```bash
chmod +x audit-podman.sh
```

---

# 127. Laboratorio RHCSA

## Ejercicio 1

Consultar:

```bash
cat /etc/subuid
```

---

## Ejercicio 2

Consultar:

```bash
cat /etc/subgid
```

---

## Ejercicio 3

Mostrar:

```bash
id
```

---

## Ejercicio 4

Consultar Rootless:

```bash
podman info \
--format '{{.Host.Security.Rootless}}'
```

---

## Ejercicio 5

Consultar almacenamiento:

```bash
podman info \
--format '{{.Store.GraphRoot}}'
```

---

## Ejercicio 6

Mostrar redes:

```bash
podman network ls
```

---

## Ejercicio 7

Inspeccionar red:

```bash
podman network inspect podman
```

---

## Ejercicio 8

Ver firewall:

```bash
firewall-cmd --list-all
```

---

## Ejercicio 9

Consultar SELinux:

```bash
getenforce
```

---

## Ejercicio 10

Consultar contextos:

```bash
ps -eZ
```

---

## Ejercicio 11

Buscar AVC:

```bash
ausearch \
-m AVC \
-ts recent
```

---

## Ejercicio 12

Consultar estadísticas:

```bash
podman stats
```

---

## Ejercicio 13

Consultar eventos:

```bash
podman events
```

---

## Ejercicio 14

Ejecutar auditoría:

```bash
./audit-podman.sh
```

---

# Buenas prácticas

- Utilizar siempre Rootless cuando sea posible.
- Mantener SELinux en Enforcing.
- No usar `--privileged` salvo necesidad.
- Utilizar User Namespaces.
- Revisar periódicamente `podman info`.
- Mantener Netavark actualizado.
- Revisar periódicamente `journalctl`.

---

# Errores comunes

## Error 1

Deshabilitar SELinux.

---

## Error 2

Eliminar GraphRoot manualmente.

---

## Error 3

Modificar `/etc/subuid` incorrectamente.

---

## Error 4

Confundir UID del contenedor con UID del host.

---

## Error 5

No revisar `podman info`.

---

# Resumen

En esta fase aprendimos:

- Funcionamiento de User Namespaces.
- Configuración de SubUID y SubGID.
- Almacenamiento Rootless.
- Redes Rootless.
- Integración con SELinux.
- Diagnóstico avanzado.
- Auditoría de Podman.
- Buenas prácticas de seguridad.

La **Fase 4** integrará todos estos conceptos mediante un laboratorio completo con más de 30 ejercicios prácticos, casos reales de troubleshooting, checklist RHCSA, preguntas de repaso y un desafío final similar al examen oficial.