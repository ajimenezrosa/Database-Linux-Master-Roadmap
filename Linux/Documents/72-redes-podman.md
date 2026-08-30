# 72. Redes en Podman (Fase 1)

> **Módulo 10 – Contenedores con Podman**
>
> **Manual RHCSA**
>
> **Archivo:** `72-redes-podman.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase podrás:

- Comprender cómo funciona la red en Podman.
- Conocer los diferentes tipos de redes.
- Comprender la diferencia entre Rootful y Rootless Networking.
- Administrar redes existentes.
- Crear redes personalizadas.
- Comprender el funcionamiento de Netavark.
- Comprender el rol de Aardvark DNS.
- Conectar contenedores entre sí.
- Prepararte para las prácticas del examen RHCSA.

---

# Introducción

Los contenedores rara vez trabajan solos.

Normalmente encontramos escenarios como:

- Servidor Web
- Base de Datos
- Redis
- Balanceadores
- APIs
- Sistemas de monitoreo

Todos necesitan comunicarse entre sí.

Para ello Podman proporciona un completo sistema de redes.

---

# ¿Qué es una red en Podman?

Una red es un mecanismo que permite que uno o varios contenedores puedan comunicarse utilizando direcciones IP virtuales.

La red puede permitir comunicación:

- Contenedor → Contenedor
- Contenedor → Host
- Contenedor → Internet
- Host → Contenedor

---

# Arquitectura general

```text
                    Internet
                        │
                        │
                 ───────┼────────
                        │
                 Fedora Host
                        │
                 Netavark Engine
                        │
        ┌───────────────┼───────────────┐
        │               │               │
        ▼               ▼               ▼
   Container A     Container B     Container C
```

---

# Componentes principales

Desde Podman 4.x la arquitectura utiliza:

- Netavark
- Aardvark DNS
- iptables/nftables
- Interfaces virtuales
- Bridges Linux

---

# ¿Qué es Netavark?

Netavark es el backend moderno de redes utilizado por Podman.

Sus responsabilidades son:

- Crear Bridges
- Crear Interfaces Virtuales
- Configurar NAT
- Configurar Firewall
- Configurar rutas
- Administrar IPv4
- Administrar IPv6

---

# Ventajas de Netavark

Comparado con CNI:

- Más rápido
- Menos dependencias
- Mejor soporte IPv6
- Configuración sencilla
- Integración con Podman 4+

---

# ¿Qué es Aardvark DNS?

Es el servidor DNS utilizado por Podman.

Permite resolver nombres de contenedores.

Ejemplo:

```text
web

↓

192.168.100.5
```

Sin necesidad de conocer la IP.

---

# Ejemplo

Contenedor:

```text
postgres
```

Otro contenedor puede conectarse utilizando:

```text
postgres
```

en lugar de:

```text
10.89.0.12
```

---

# Arquitectura

```text
Container A

hostname=db

        │

DNS Query

        │

        ▼

Aardvark DNS

        │

        ▼

10.89.0.15
```

---

# Tipos de red

Los más utilizados son:

| Tipo | Descripción |
|-------|-------------|
| Bridge | Red privada |
| Host | Comparte la red del Host |
| None | Sin conectividad |
| Slirp4netns | Rootless |
| Pasta | Rootless moderno |

---

# Bridge

Es la red por defecto.

```text
Host

│

Bridge

│

├── Web

├── DB

└── Redis
```

Todos pueden comunicarse.

---

# Host Network

El contenedor utiliza directamente la red del Host.

```bash
podman run --network host nginx
```

No existe NAT.

---

# Ventajas

- Mayor rendimiento
- Menor latencia

---

# Desventajas

- Menor aislamiento
- Riesgo de conflictos de puertos

---

# None

```bash
podman run \
--network none \
alpine
```

Resultado:

El contenedor no tiene acceso a ninguna red.

---

# ¿Cuándo utilizar None?

- Laboratorios
- Seguridad
- Procesamiento local
- Contenedores aislados

---

# Rootful Networking

Ejecutado por:

```text
root
```

Características:

- Bridges reales
- Interfaces virtuales
- NAT completo
- Mayor flexibilidad

---

# Rootless Networking

Ejecutado por:

Usuarios normales.

No requiere privilegios.

Utiliza:

- slirp4netns
- pasta

---

# Comparación

| Rootful | Rootless |
|----------|-----------|
| Requiere root | No |
| Mayor control | Mayor seguridad |
| Bridges reales | Emulación |
| Alto rendimiento | Muy buen rendimiento |

---

# Slirp4netns

Durante años fue la solución Rootless.

Ventajas:

- Seguro
- Fácil de usar
- Sin privilegios

Limitaciones:

- Más lento
- NAT por software

---

# Pasta

Actualmente recomendado.

Ventajas:

- Más rápido
- Mejor integración
- Menor latencia
- Mejor IPv6

---

# Consultar redes

```bash
podman network ls
```

Ejemplo

```text
NETWORK ID

NAME

DRIVER
```

---

# Inspeccionar una red

```bash
podman network inspect podman
```

---

Información mostrada

- Subred
- Gateway
- Driver
- IPv6
- DNS
- Interfaces

---

# Crear una red

```bash
podman network create red-web
```

---

Verificar

```bash
podman network ls
```

---

# Crear una subred

```bash
podman network create \
--subnet 10.20.30.0/24 \
red-laboratorio
```

---

# Gateway

```bash
podman network create \
--gateway 10.20.30.1 \
--subnet 10.20.30.0/24 \
red1
```

---

# Crear con IPv6

```bash
podman network create \
--ipv6 \
red-ipv6
```

---

# Eliminar una red

```bash
podman network rm red-web
```

---

# Restricción

No puede eliminarse una red utilizada por contenedores.

---

# Conectar un contenedor

```bash
podman run \
-d \
--network red-web \
--name nginx \
nginx
```

---

# Verificar

```bash
podman inspect nginx
```

Consultar:

```text
NetworkSettings
```

---

# Dirección IP

```bash
podman inspect \
--format '{{.NetworkSettings.IPAddress}}' \
nginx
```

---

# Resolver nombres

Supongamos:

```text
web

db

redis
```

Todos dentro de:

```text
red-web
```

Entonces:

```text
web

↓

db
```

utilizando simplemente:

```text
db
```

---

# Comunicación

```text
           red-web

      ┌──────────────┐

      │              │

      ▼              ▼

    nginx         postgres
```

---

# Publicación de puertos

La comunicación entre contenedores NO requiere:

```text
-p
```

Solo es necesaria para:

```text
Host

↓

Contenedor
```

---

# Diferencia

```text
Container

↓

Container

No necesita -p
```

---

```text
Host

↓

Container

Sí necesita -p
```

---

# Flujo completo

```text
Internet

      │

      ▼

Host Fedora

      │

      ▼

Bridge Podman

      │

 ┌────┴─────┐

 ▼          ▼

Web      PostgreSQL
```

---

# Laboratorio RHCSA

## Laboratorio 1

Consultar redes.

```bash
podman network ls
```

---

## Laboratorio 2

Inspeccionar la red.

```bash
podman network inspect podman
```

---

## Laboratorio 3

Crear una red.

```bash
podman network create red-web
```

---

## Laboratorio 4

Crear una red con subred.

```bash
podman network create \
--subnet 10.20.30.0/24 \
red-app
```

---

## Laboratorio 5

Crear una red IPv6.

```bash
podman network create \
--ipv6 \
red6
```

---

## Laboratorio 6

Ejecutar Nginx.

```bash
podman run \
-d \
--network red-web \
--name web \
nginx
```

---

## Laboratorio 7

Ejecutar PostgreSQL.

```bash
podman run \
-d \
--network red-web \
--name db \
postgres:17
```

---

## Laboratorio 8

Consultar IP.

```bash
podman inspect \
--format '{{.NetworkSettings.IPAddress}}' \
web
```

---

## Laboratorio 9

Consultar redes nuevamente.

```bash
podman network ls
```

---

## Laboratorio 10

Eliminar la red creada.

```bash
podman network rm red-app
```

---

# Buenas prácticas

- Crear redes independientes para cada aplicación.
- No utilizar la red `host` salvo que sea estrictamente necesario.
- Preferir redes Bridge para la mayoría de los servicios.
- Utilizar nombres descriptivos para las redes.
- Evitar publicar puertos innecesarios.
- Aprovechar la resolución DNS proporcionada por Aardvark en lugar de direcciones IP fijas.

---

# Errores comunes

## Error 1

Pensar que todos los contenedores pueden comunicarse automáticamente.

---

## Error 2

Publicar todos los puertos hacia el Host.

---

## Error 3

Eliminar una red que todavía tiene contenedores asociados.

---

## Error 4

Usar direcciones IP fijas cuando es posible utilizar nombres DNS.

---

## Error 5

Confundir la comunicación **Contenedor → Contenedor** con **Host → Contenedor**.

---

# Resumen

En esta primera fase aprendimos:

- Cómo funciona la arquitectura de red de Podman.
- El papel de Netavark y Aardvark DNS.
- Los distintos tipos de redes disponibles.
- Las diferencias entre Rootful y Rootless Networking.
- Cómo crear, inspeccionar y eliminar redes.
- Cómo conectar contenedores a redes personalizadas.
- Cuándo es necesario publicar puertos y cuándo no.

En la **Fase 2** aprenderemos a conectar y desconectar contenedores de múltiples redes, configurar direcciones IP estáticas, aliases DNS, opciones avanzadas de `podman network`, resolución de nombres, pruebas de conectividad y escenarios reales de producción utilizados en el examen **RHCSA**.

-----

# 72. Redes en Podman (Fase 2)

> **Módulo 10 – Contenedores con Podman**
>
> **Manual RHCSA**
>
> **Archivo:** `72-redes-podman.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase podrás:

- Conectar un contenedor a múltiples redes.
- Desconectar un contenedor de una red.
- Asignar direcciones IP estáticas.
- Configurar direcciones MAC.
- Utilizar Alias DNS.
- Comprender la resolución de nombres.
- Probar la conectividad entre contenedores.
- Administrar configuraciones avanzadas de redes.
- Diagnosticar problemas de conectividad.

---

# Introducción

En ambientes reales es muy común que un mismo contenedor pertenezca a más de una red.

Por ejemplo:

```text
                 Internet
                     │
                     │
             Frontend Network
                     │
                 Web Server
                     │
             Backend Network
                     │
                 PostgreSQL
```

El servidor Web necesita comunicarse con Internet y con la base de datos.

---

# Un contenedor puede pertenecer a varias redes

Ejemplo:

```text
             web

         ┌────┴────┐

         ▼         ▼

   frontend     backend
```

Esto permite separar el tráfico de usuarios del tráfico interno.

---

# Crear dos redes

```bash
podman network create frontend
```

```bash
podman network create backend
```

Verificar:

```bash
podman network ls
```

---

# Crear un contenedor en una red

```bash
podman run -d \
--name web \
--network frontend \
nginx
```

---

# Conectar una segunda red

Una vez creado el contenedor:

```bash
podman network connect \
backend \
web
```

---

# Arquitectura

```text
             frontend

                 │

                 ▼

              nginx

                 ▲

                 │

             backend
```

---

# Verificar

```bash
podman inspect web
```

Buscar:

```text
NetworkSettings
```

Aparecerán ambas redes.

---

# Desconectar una red

```bash
podman network disconnect \
frontend \
web
```

---

# Verificar nuevamente

```bash
podman inspect web
```

Ahora únicamente aparecerá:

```text
backend
```

---

# ¿Cuándo utilizar múltiples redes?

Ejemplos:

- Servidor Web
- API Gateway
- Balanceadores
- Proxy Inverso
- Firewalls

---

# Direcciones IP Automáticas

Por defecto Podman asigna una dirección IP automáticamente.

Ejemplo:

```text
10.88.0.15
```

---

# IP Estática

Podemos asignar una IP fija.

Ejemplo:

```bash
podman run \
-d \
--network frontend \
--ip 10.20.30.15 \
nginx
```

---

# Restricciones

La dirección IP:

- debe pertenecer a la subred
- no puede repetirse

---

# Arquitectura

```text
frontend

10.20.30.0/24

        │

        ├── 10.20.30.10

        ├── 10.20.30.15

        └── 10.20.30.20
```

---

# Verificar IP

```bash
podman inspect \
--format '{{.NetworkSettings.IPAddress}}' \
web
```

---

# Dirección MAC

También es posible definirla.

```bash
podman run \
--mac-address \
02:42:ac:11:00:02 \
nginx
```

---

# ¿Cuándo utilizar MAC fija?

Muy poco frecuente.

Generalmente:

- Laboratorios
- Sistemas heredados
- Integraciones especiales

---

# Alias DNS

Un mismo contenedor puede responder por varios nombres.

Ejemplo:

```bash
podman run \
-d \
--network frontend \
--network-alias sitio \
nginx
```

---

Resultado

El contenedor responderá por:

```text
nginx
```

y también por:

```text
sitio
```

---

# Varios Alias

```bash
podman run \
-d \
--network frontend \
--network-alias web \
--network-alias portal \
--network-alias empresa \
nginx
```

---

# Resolución DNS

Supongamos:

```text
frontend

↓

web

↓

10.20.30.15
```

Otro contenedor puede ejecutar:

```bash
ping web
```

Sin conocer la IP.

---

# Resolución utilizando Alias

```text
portal

↓

10.20.30.15
```

También funcionará.

---

# Comunicación entre contenedores

Supongamos:

```text
Web

↓

PostgreSQL
```

Configuración:

```text
frontend

↓

web
```

```text
backend

↓

postgres
```

Si ambos no pertenecen a la misma red:

No podrán comunicarse.

---

# Solución

Conectar ambos a:

```text
backend
```

---

# Arquitectura

```text
backend

│

├── web

└── postgres
```

---

# Probar conectividad

Ingresar:

```bash
podman exec -it web sh
```

Luego:

```bash
ping postgres
```

---

# Probar DNS

```bash
getent hosts postgres
```

Resultado:

```text
10.20.30.20
```

---

# Ver Interfaces

```bash
ip addr
```

Dentro del contenedor.

---

# Ver Rutas

```bash
ip route
```

---

# Consultar DNS

```bash
cat /etc/resolv.conf
```

---

# Consultar Hostname

```bash
hostname
```

---

# Ver Gateway

```bash
ip route
```

Ejemplo:

```text
default via 10.20.30.1
```

---

# Crear una red interna

```bash
podman network create \
--internal \
red-interna
```

---

# ¿Qué significa?

Los contenedores:

- pueden comunicarse entre sí

pero

- no tienen salida hacia Internet

---

# Arquitectura

```text
Internet

    X

    │

──────────────

red-interna

│

├── app

└── db
```

---

# Crear una red sin DNS

```bash
podman network create \
--disable-dns \
red-pruebas
```

---

# ¿Qué ocurre?

Los contenedores deberán comunicarse utilizando direcciones IP.

---

# Consultar configuración

```bash
podman network inspect frontend
```

Información:

- Gateway
- DNS
- IPv6
- Driver
- Interfaces
- Subred

---

# Eliminar un contenedor de la red

```bash
podman network disconnect \
backend \
web
```

---

# Reconectar

```bash
podman network connect \
backend \
web
```

---

# Cambiar de red

Procedimiento:

```text
Disconnect

↓

Connect
```

---

# Caso práctico

Servidor:

```text
Nginx
```

Necesita:

- Internet

y además:

```text
PostgreSQL
```

Configuración:

```text
Frontend

↓

Internet
```

```text
Backend

↓

DB
```

---

# Caso práctico empresarial

```text
                 Usuarios

                     │

                     ▼

                 Frontend

                     │

             Reverse Proxy

                     │

             Backend Network

         ┌──────────────┐

         ▼              ▼

      API Server    PostgreSQL
```

La base de datos nunca se expone directamente al exterior.

---

# Laboratorio RHCSA

## Laboratorio 1

Crear dos redes.

```bash
podman network create frontend
```

```bash
podman network create backend
```

---

## Laboratorio 2

Listarlas.

```bash
podman network ls
```

---

## Laboratorio 3

Crear un servidor Web.

```bash
podman run \
-d \
--name web \
--network frontend \
nginx
```

---

## Laboratorio 4

Conectar Backend.

```bash
podman network connect \
backend \
web
```

---

## Laboratorio 5

Verificar.

```bash
podman inspect web
```

---

## Laboratorio 6

Desconectar Frontend.

```bash
podman network disconnect \
frontend \
web
```

---

## Laboratorio 7

Reconectarlo.

```bash
podman network connect \
frontend \
web
```

---

## Laboratorio 8

Crear un PostgreSQL.

```bash
podman run \
-d \
--name db \
--network backend \
postgres:17
```

---

## Laboratorio 9

Entrar al Web.

```bash
podman exec -it web sh
```

---

## Laboratorio 10

Probar DNS.

```bash
ping db
```

---

## Laboratorio 11

Consultar IP.

```bash
getent hosts db
```

---

## Laboratorio 12

Consultar interfaces.

```bash
ip addr
```

---

## Laboratorio 13

Consultar rutas.

```bash
ip route
```

---

## Laboratorio 14

Consultar DNS.

```bash
cat /etc/resolv.conf
```

---

## Laboratorio 15

Inspeccionar la red.

```bash
podman network inspect backend
```

---

# Buenas prácticas

- Separar aplicaciones mediante redes independientes.
- Utilizar Alias DNS en lugar de direcciones IP fijas cuando sea posible.
- Asignar IP estática únicamente cuando exista una necesidad específica.
- No conectar un contenedor a más redes de las necesarias.
- Mantener la base de datos en una red privada.
- Utilizar redes internas (`--internal`) para aplicaciones que no requieren acceso a Internet.

---

# Errores comunes

## Error 1

Asignar una dirección IP fuera del rango de la subred.

---

## Error 2

Conectar todos los contenedores a una única red.

---

## Error 3

Pensar que dos contenedores pueden comunicarse si pertenecen a redes distintas.

---

## Error 4

Publicar la base de datos hacia Internet innecesariamente.

---

## Error 5

Eliminar una red sin desconectar previamente los contenedores asociados.

---

# Resumen

En esta segunda fase aprendimos a:

- Conectar y desconectar contenedores de múltiples redes.
- Configurar direcciones IP y MAC estáticas.
- Utilizar Alias DNS para facilitar la comunicación entre contenedores.
- Comprender el funcionamiento de la resolución de nombres mediante Aardvark DNS.
- Probar la conectividad utilizando `ping`, `getent`, `ip addr`, `ip route` y `resolv.conf`.
- Implementar redes internas y escenarios con múltiples segmentos de red similares a los utilizados en entornos empresariales.

En la **Fase 3** aprenderemos sobre el reenvío de puertos (Port Forwarding), NAT, exposición segura de servicios, IPv6, comunicación entre el Host y los contenedores, integración con firewalld y nftables, así como técnicas avanzadas de diagnóstico de redes en Podman.


---

# 72. Redes en Podman (Fase 3)

> **Módulo 10 – Contenedores con Podman**
>
> **Manual RHCSA**
>
> **Archivo:** `72-redes-podman.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase podrás:

- Comprender cómo funciona el Port Forwarding.
- Entender el funcionamiento del NAT.
- Publicar servicios de forma segura.
- Configurar múltiples puertos.
- Comprender el soporte IPv4 e IPv6.
- Entender la comunicación entre el Host y los contenedores.
- Integrar Podman con firewalld.
- Comprender el papel de nftables e iptables.
- Diagnosticar problemas de conectividad.

---

# Introducción

Hasta ahora hemos aprendido que los contenedores pueden comunicarse entre sí utilizando redes Bridge.

Ahora estudiaremos cómo un usuario externo puede acceder a esos contenedores.

El mecanismo utilizado es el **Port Forwarding**, también conocido como **Publicación de Puertos**.

---

# Comunicación entre redes

Existen tres escenarios principales.

```text
Host
 │
 ▼
Contenedor
```

---

```text
Contenedor
 │
 ▼
Contenedor
```

---

```text
Internet
 │
 ▼
Host
 │
 ▼
Contenedor
```

Cada escenario utiliza mecanismos diferentes.

---

# ¿Qué es Port Forwarding?

Es el proceso mediante el cual un puerto del Host es redirigido hacia un puerto del contenedor.

Ejemplo:

```text
Cliente

↓

192.168.1.20:8080

↓

Fedora Host

↓

NAT

↓

Container

80
```

---

# Sintaxis

```bash
podman run \
-p HOST:CONTENEDOR
```

---

# Ejemplo

```bash
podman run \
-d \
-p 8080:80 \
nginx
```

---

# Interpretación

```text
Host

Puerto 8080

↓

Container

Puerto 80
```

---

# Acceso

Desde el navegador:

```text
http://IP_DEL_HOST:8080
```

---

# Publicar PostgreSQL

```bash
podman run \
-d \
-p 5432:5432 \
postgres:17
```

---

# Redis

```bash
podman run \
-d \
-p 6379:6379 \
redis
```

---

# Publicar múltiples puertos

```bash
podman run \
-d \
-p 80:80 \
-p 443:443 \
httpd
```

---

# Arquitectura

```text
               Internet

                    │

                    ▼

             Fedora Host

          80      443

            │      │

            ▼      ▼

          Apache HTTPD
```

---

# Publicar UDP

Por defecto:

```text
TCP
```

Para UDP:

```bash
podman run \
-p 53:53/udp \
dns-server
```

---

# Publicar TCP y UDP

```bash
-p 53:53/tcp

-p 53:53/udp
```

---

# Publicación automática

Podman puede elegir un puerto libre.

```bash
podman run \
-P \
nginx
```

---

# Diferencia

```text
-p

Puerto específico
```

---

```text
-P

Puerto aleatorio
```

---

# Consultar puertos

```bash
podman port web
```

Ejemplo:

```text
80/tcp

↓

8080
```

---

# Verificar desde el Host

```bash
curl localhost:8080
```

---

También:

```bash
curl 127.0.0.1:8080
```

---

# Verificar escucha

```bash
ss -lnt
```

---

También:

```bash
ss -lntp
```

---

# NAT

Cuando utilizamos:

```bash
-p
```

Podman configura automáticamente reglas NAT.

---

# Flujo NAT

```text
Cliente

↓

Puerto 8080

↓

NAT

↓

80

↓

Nginx
```

---

# ¿Qué es NAT?

NAT significa:

```text
Network Address Translation
```

Permite traducir direcciones y puertos.

---

# Beneficios

- Oculta la IP del contenedor.
- Permite múltiples servicios.
- Reduce la exposición directa.
- Facilita el aislamiento.

---

# Comunicación Host → Contenedor

```text
Host

↓

8080

↓

Container

80
```

---

# Comunicación Contenedor → Internet

```text
Container

↓

Bridge

↓

NAT

↓

Internet
```

---

# Comunicación Contenedor → Host

En la mayoría de escenarios basta con utilizar la IP del Host.

Ejemplo:

```text
192.168.1.20
```

---

# Hostname especial

Podman también soporta:

```text
host.containers.internal
```

Ejemplo:

```bash
curl http://host.containers.internal:8080
```

Muy útil cuando un contenedor necesita consumir un servicio que se ejecuta en el Host.

---

# IPv6

Crear una red:

```bash
podman network create \
--ipv6 \
red6
```

---

# Consultar IPv6

```bash
podman inspect web
```

Buscar:

```text
GlobalIPv6Address
```

---

# Publicar IPv6

Ejemplo conceptual:

```text
[2001:db8::10]:8080
```

↓

```text
Container

80
```

---

# Consultar interfaces

```bash
ip addr
```

---

# Consultar rutas

```bash
ip route
```

---

# Consultar sockets

```bash
ss -tulnp
```

---

# Firewalld

Fedora utiliza:

```text
firewalld
```

para administrar el firewall.

---

# Verificar estado

```bash
sudo systemctl status firewalld
```

---

# Consultar zonas

```bash
sudo firewall-cmd \
--get-active-zones
```

---

# Abrir un puerto

Ejemplo:

```bash
sudo firewall-cmd \
--permanent \
--add-port=8080/tcp
```

---

Aplicar cambios

```bash
sudo firewall-cmd --reload
```

---

# Consultar puertos

```bash
sudo firewall-cmd \
--list-ports
```

---

# nftables

Las distribuciones modernas utilizan:

```text
nftables
```

como backend principal.

---

Consultar reglas

```bash
sudo nft list ruleset
```

---

# iptables

En algunos sistemas todavía encontraremos:

```text
iptables
```

Consultar:

```bash
sudo iptables -L
```

---

# Verificar conectividad

Desde otro servidor:

```bash
ping HOST
```

---

Luego:

```bash
curl http://HOST:8080
```

---

# Diagnóstico

Si no responde:

Verificar:

```bash
podman ps
```

↓

```bash
podman port
```

↓

```bash
ss -lnt
```

↓

```bash
firewall-cmd
```

↓

```bash
curl
```

---

# Caso práctico

Servidor Fedora

```text
192.168.100.20
```

Contenedor:

```text
Nginx
```

Configuración:

```bash
podman run \
-d \
-p 8080:80 \
nginx
```

Los clientes accederán mediante:

```text
http://192.168.100.20:8080
```

---

# Caso práctico empresarial

```text
                 Internet

                     │

                     ▼

             Firewall Corporativo

                     │

                     ▼

               Fedora Server

              443 → Container

                     │

                     ▼

               Reverse Proxy

                     │

             Backend Network

          ┌──────────────┐

          ▼              ▼

      API Server      PostgreSQL
```

La base de datos nunca publica puertos hacia Internet.

---

# Laboratorio RHCSA

## Laboratorio 1

Ejecutar Nginx.

```bash
podman run \
-d \
--name web \
-p 8080:80 \
nginx
```

---

## Laboratorio 2

Consultar puertos.

```bash
podman port web
```

---

## Laboratorio 3

Consultar sockets.

```bash
ss -lnt
```

---

## Laboratorio 4

Probar acceso.

```bash
curl localhost:8080
```

---

## Laboratorio 5

Publicar PostgreSQL.

```bash
podman run \
-d \
-p 5432:5432 \
postgres:17
```

---

## Laboratorio 6

Publicar Redis.

```bash
podman run \
-d \
-p 6379:6379 \
redis
```

---

## Laboratorio 7

Publicar múltiples puertos.

```bash
podman run \
-d \
-p 80:80 \
-p 443:443 \
httpd
```

---

## Laboratorio 8

Consultar firewalld.

```bash
sudo firewall-cmd \
--list-ports
```

---

## Laboratorio 9

Abrir puerto 8080.

```bash
sudo firewall-cmd \
--permanent \
--add-port=8080/tcp
```

---

## Laboratorio 10

Recargar firewall.

```bash
sudo firewall-cmd --reload
```

---

## Laboratorio 11

Consultar nftables.

```bash
sudo nft list ruleset
```

---

## Laboratorio 12

Consultar reglas iptables.

```bash
sudo iptables -L
```

---

## Laboratorio 13

Verificar IPv6.

```bash
podman inspect web
```

---

## Laboratorio 14

Consultar interfaces.

```bash
ip addr
```

---

## Laboratorio 15

Consultar rutas.

```bash
ip route
```

---

# Buenas prácticas

- Publicar únicamente los puertos necesarios.
- Mantener las bases de datos en redes privadas.
- Utilizar `host.containers.internal` cuando un contenedor necesite acceder a un servicio del Host.
- Preferir HTTPS (443) para servicios públicos.
- Validar la configuración del firewall después de publicar un servicio.
- Supervisar periódicamente los puertos abiertos mediante `ss` y `firewall-cmd`.

---

# Errores comunes

## Error 1

Pensar que publicar un puerto es suficiente sin revisar el firewall.

---

## Error 2

Publicar servicios sensibles como PostgreSQL o Redis directamente hacia Internet.

---

## Error 3

Confundir el puerto del Host con el puerto interno del contenedor.

---

## Error 4

No comprobar que el servicio dentro del contenedor realmente esté escuchando en el puerto esperado.

---

## Error 5

Olvidar verificar las reglas de `firewalld`, `nftables` o `iptables` cuando un servicio no es accesible.

---

# Resumen

En esta tercera fase aprendimos a:

- Comprender el funcionamiento del Port Forwarding y del NAT.
- Publicar uno o varios puertos utilizando `-p` y `-P`.
- Exponer servicios TCP y UDP.
- Verificar la publicación de puertos con `podman port`, `ss` y `curl`.
- Comprender la comunicación entre el Host y los contenedores.
- Trabajar con IPv6.
- Integrar Podman con `firewalld`, `nftables` e `iptables`.
- Diagnosticar problemas de conectividad mediante un procedimiento estructurado.

En la **Fase 4** realizaremos un laboratorio completo de redes en Podman, incluyendo escenarios reales de producción, scripts de auditoría, resolución de problemas, checklist RHCSA, preguntas de repaso y un desafío final similar al examen oficial.

---

# 72. Redes en Podman (Fase 4)

> **Módulo 10 – Contenedores con Podman**
>
> **Manual RHCSA**
>
> **Archivo:** `72-redes-podman.md`

---

# Objetivos de aprendizaje

Al finalizar esta fase podrás:

- Diagnosticar problemas de red en Podman.
- Resolver incidencias relacionadas con DNS, NAT y Port Forwarding.
- Auditar redes y contenedores.
- Automatizar verificaciones mediante scripts.
- Implementar buenas prácticas utilizadas en ambientes empresariales.
- Resolver escenarios similares al examen RHCSA.

---

# Metodología de Diagnóstico

Cuando una aplicación no responde, el problema puede encontrarse en distintos niveles.

Nunca asumas inmediatamente que el error pertenece a Podman.

Sigue siempre un procedimiento estructurado.

```text
               Aplicación inaccesible
                        │
                        ▼
          ¿Contenedor ejecutándose?
                        │
                        ▼
          ¿Puerto publicado correctamente?
                        │
                        ▼
             ¿Firewall abierto?
                        │
                        ▼
          ¿Servicio escuchando?
                        │
                        ▼
         ¿Red correcta del contenedor?
                        │
                        ▼
          ¿DNS funciona correctamente?
                        │
                        ▼
             Resolver el problema
```

---

# Lista de verificación

Antes de modificar cualquier configuración verifica:

```bash
podman ps
```

↓

```bash
podman port
```

↓

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

↓

```bash
ss -lntp
```

↓

```bash
firewall-cmd
```

↓

```bash
curl
```

---

# Escenario 1

## El sitio web no responde

Configuración:

```bash
podman run \
-d \
-p 8080:80 \
nginx
```

El navegador devuelve:

```text
Connection Refused
```

Diagnóstico

```bash
podman ps
```

↓

```bash
podman port web
```

↓

```bash
ss -lnt
```

↓

```bash
curl localhost:8080
```

↓

```bash
firewall-cmd --list-ports
```

---

# Posibles causas

- Contenedor detenido
- Puerto incorrecto
- Firewall bloqueando
- Servicio interno detenido

---

# Escenario 2

## Dos contenedores no se comunican

Consultar:

```bash
podman inspect web
```

Consultar:

```bash
podman inspect db
```

Verificar que ambos pertenezcan a:

```text
backend
```

---

# Solución

```bash
podman network connect \
backend \
web
```

---

# Escenario 3

## DNS no funciona

Entrar:

```bash
podman exec -it web sh
```

Consultar:

```bash
getent hosts db
```

Si no devuelve resultados:

Verificar:

```bash
podman network inspect backend
```

---

# Escenario 4

## El puerto está ocupado

Intentamos:

```bash
podman run \
-p 80:80 \
nginx
```

Resultado:

```text
address already in use
```

Consultar:

```bash
ss -lntp
```

o

```bash
lsof -i :80
```

---

# Solución

Utilizar otro puerto.

```bash
-p 8080:80
```

---

# Escenario 5

## El firewall bloquea el acceso

Consultar:

```bash
sudo firewall-cmd \
--list-ports
```

Abrir puerto:

```bash
sudo firewall-cmd \
--permanent \
--add-port=8080/tcp
```

Aplicar:

```bash
sudo firewall-cmd --reload
```

---

# Escenario 6

## NAT incorrecto

Consultar:

```bash
podman port web
```

Debe aparecer:

```text
80/tcp

↓

8080
```

---

# Escenario 7

## El servicio escucha únicamente en localhost

Dentro del contenedor:

```bash
ss -lnt
```

Si aparece:

```text
127.0.0.1
```

Debe modificarse la aplicación para escuchar en:

```text
0.0.0.0
```

---

# Escenario 8

## La IP cambió

Consultar:

```bash
podman inspect web
```

Recordar:

Las IP pueden cambiar al recrear un contenedor.

Se recomienda utilizar:

```text
DNS
```

en lugar de IPs fijas.

---

# Escenario 9

## Error de Gateway

Consultar:

```bash
ip route
```

Debe existir:

```text
default via
```

---

# Escenario 10

## IPv6

Consultar:

```bash
ip -6 addr
```

Verificar:

```bash
podman network inspect
```

---

# Herramientas de Diagnóstico

## Podman

```bash
podman ps
```

```bash
podman port
```

```bash
podman network ls
```

```bash
podman network inspect
```

```bash
podman inspect
```

---

## Linux

```bash
ip addr
```

```bash
ip route
```

```bash
ss -lntp
```

```bash
ping
```

```bash
curl
```

```bash
getent
```

```bash
hostname
```

---

# Script de Auditoría de Redes

Guardar como:

```text
network_audit.sh
```

```bash
#!/bin/bash

echo "======================================="
echo " PODMAN NETWORK AUDIT "
echo "======================================="

echo
echo "REDES"
podman network ls

echo
echo "CONTENEDORES"
podman ps

echo
echo "PUERTOS"
for c in $(podman ps -q)
do
    echo
    echo "=========="
    echo "$c"
    echo "=========="
    podman port "$c"
done

echo
echo "FIREWALL"
firewall-cmd --list-ports

echo
echo "SOCKETS"
ss -lnt

echo
echo "FIN"
```

Permisos:

```bash
chmod +x network_audit.sh
```

---

# Script para auditar redes

```bash
#!/bin/bash

for n in $(podman network ls --format "{{.Name}}")
do

echo
echo "==============================="
echo "$n"
echo "==============================="

podman network inspect "$n"

done
```

---

# Script para probar conectividad

```bash
#!/bin/bash

HOST=$1

echo

ping -c 4 "$HOST"

echo

curl -I http://"$HOST"
```

Uso:

```bash
./network_test.sh localhost:8080
```

---

# Arquitectura Empresarial

```text
                    Internet

                        │

                Firewall Externo

                        │

                 Reverse Proxy

                        │

              Frontend Network

                        │

          ┌─────────────┴─────────────┐

          ▼                           ▼

      Web Server                 API Server

                  Backend Network

          ┌─────────────┴─────────────┐

          ▼                           ▼

      PostgreSQL                  Redis
```

---

# Arquitectura de Alta Seguridad

```text
                 Internet

                     │

               Firewall

                     │

             Reverse Proxy

                     │

              Zona DMZ

                     │

          ┌──────────┴──────────┐

          ▼                     ▼

      Web App              API Gateway

                     │

             Red Privada

                     │

          ┌──────────┴──────────┐

          ▼                     ▼

      PostgreSQL             Redis
```

---

# Laboratorio RHCSA

## Laboratorio 1

Crear una red.

```bash
podman network create backend
```

---

## Laboratorio 2

Crear un servidor Web.

```bash
podman run \
-d \
--network backend \
-p 8080:80 \
--name web \
nginx
```

---

## Laboratorio 3

Consultar redes.

```bash
podman network ls
```

---

## Laboratorio 4

Consultar la configuración.

```bash
podman network inspect backend
```

---

## Laboratorio 5

Consultar los puertos.

```bash
podman port web
```

---

## Laboratorio 6

Probar acceso.

```bash
curl localhost:8080
```

---

## Laboratorio 7

Consultar sockets.

```bash
ss -lnt
```

---

## Laboratorio 8

Consultar interfaces.

```bash
ip addr
```

---

## Laboratorio 9

Consultar rutas.

```bash
ip route
```

---

## Laboratorio 10

Consultar firewall.

```bash
firewall-cmd --list-ports
```

---

## Laboratorio 11

Abrir un puerto.

```bash
sudo firewall-cmd \
--permanent \
--add-port=8080/tcp
```

---

## Laboratorio 12

Aplicar cambios.

```bash
sudo firewall-cmd --reload
```

---

## Laboratorio 13

Ejecutar la auditoría.

```bash
./network_audit.sh
```

---

## Laboratorio 14

Ejecutar el script de redes.

```bash
./network_networks.sh
```

---

## Laboratorio 15

Eliminar el laboratorio.

```bash
podman stop web

podman rm web

podman network rm backend
```

---

# Checklist RHCSA

```text
□ Las redes fueron creadas correctamente.

□ Los contenedores utilizan la red adecuada.

□ El DNS funciona correctamente.

□ Los Alias funcionan.

□ La IP pertenece a la subred.

□ El Gateway es correcto.

□ El puerto fue publicado.

□ El Firewall permite el acceso.

□ El servicio escucha correctamente.

□ NAT funciona.

□ El Host accede al contenedor.

□ Los contenedores se comunican entre sí.

□ IPv6 funciona cuando es requerido.

□ Las auditorías no muestran errores.
```

---

# Preguntas de Repaso

1. ¿Qué hace `podman network create`?
2. ¿Cómo listar las redes disponibles?
3. ¿Qué diferencia existe entre `bridge` y `host`?
4. ¿Qué es Netavark?
5. ¿Cuál es la función de Aardvark DNS?
6. ¿Cómo conectar un contenedor a una segunda red?
7. ¿Cómo desconectar un contenedor de una red?
8. ¿Qué hace `--network-alias`?
9. ¿Qué comando muestra la configuración completa de una red?
10. ¿Qué hace `podman port`?
11. ¿Cuál es la diferencia entre `-p` y `-P`?
12. ¿Qué es NAT?
13. ¿Qué utilidad tiene `host.containers.internal`?
14. ¿Qué herramienta administra el firewall en Fedora?
15. ¿Qué comando muestra las reglas activas del firewall?
16. ¿Cómo inspeccionar las rutas de red dentro de un contenedor?
17. ¿Qué comando muestra las interfaces de red?
18. ¿Por qué es recomendable utilizar nombres DNS en lugar de direcciones IP?
19. ¿Qué ocurre si dos contenedores están en redes diferentes sin un punto de conexión común?
20. ¿Qué pasos seguirías para diagnosticar un problema de conectividad?

---

# Respuestas

1. Crea una nueva red administrada por Podman.
2. `podman network ls`
3. `bridge` proporciona aislamiento mediante una red virtual; `host` comparte directamente la pila de red del Host.
4. Es el backend moderno encargado de la configuración de redes en Podman.
5. Resolver nombres DNS de los contenedores dentro de una misma red.
6. `podman network connect`
7. `podman network disconnect`
8. Permite asignar uno o varios nombres DNS adicionales a un contenedor.
9. `podman network inspect`
10. Muestra la relación entre los puertos del Host y del contenedor.
11. `-p` publica un puerto específico; `-P` publica automáticamente todos los puertos expuestos por la imagen en puertos aleatorios del Host.
12. Es el mecanismo que traduce direcciones y puertos entre diferentes redes.
13. Permite que un contenedor acceda a servicios ejecutándose directamente en el Host.
14. `firewalld`
15. `firewall-cmd --list-ports`
16. `ip route`
17. `ip addr`
18. Porque las direcciones IP pueden cambiar al recrear un contenedor, mientras que los nombres DNS permanecen estables dentro de la red.
19. No podrán comunicarse directamente hasta compartir una red o establecer un mecanismo de interconexión.
20. Verificar el estado del contenedor, revisar la publicación de puertos, inspeccionar la red, comprobar el firewall, validar el servicio interno y realizar pruebas con `curl`, `ping` y `getent`.

---

# Desafío Final RHCSA

Dispones de un servidor Fedora con Podman instalado.

Realiza las siguientes tareas:

1. Crear una red llamada `frontend`.
2. Crear una red llamada `backend`.
3. Ejecutar un contenedor Nginx conectado inicialmente a `frontend`.
4. Conectar el mismo contenedor a `backend`.
5. Ejecutar un contenedor PostgreSQL únicamente en `backend`.
6. Verificar que Nginx puede resolver el nombre `postgres`.
7. Publicar el puerto **8080** del Host hacia el puerto **80** del contenedor Nginx.
8. Comprobar el acceso mediante `curl`.
9. Abrir el puerto correspondiente en `firewalld`.
10. Verificar las reglas del firewall.
11. Ejecutar el script `network_audit.sh`.
12. Desconectar Nginx de `frontend` y confirmar que continúa comunicándose con PostgreSQL a través de `backend`.
13. Eliminar todos los contenedores y redes creadas sin dejar recursos residuales.

---

# Buenas prácticas

- Diseñar una red específica para cada aplicación o servicio.
- Mantener las bases de datos en redes privadas sin exposición directa al exterior.
- Publicar únicamente los puertos estrictamente necesarios.
- Utilizar nombres DNS y aliases en lugar de direcciones IP cuando sea posible.
- Revisar siempre la configuración de `firewalld` después de exponer un nuevo servicio.
- Automatizar auditorías de red para detectar configuraciones incorrectas antes de poner un entorno en producción.
- Documentar la topología de red y los puertos utilizados por cada contenedor.

---

# Resumen del Capítulo 72

En este capítulo aprendimos a:

- Comprender la arquitectura de red de Podman basada en Netavark y Aardvark DNS.
- Crear, inspeccionar y administrar redes personalizadas.
- Conectar contenedores a una o varias redes.
- Utilizar direcciones IP estáticas, aliases DNS y redes internas.
- Publicar servicios mediante Port Forwarding y comprender el funcionamiento del NAT.
- Integrar Podman con `firewalld`, `nftables` e `iptables`.
- Diagnosticar problemas de conectividad utilizando herramientas de Linux y Podman.
- Automatizar auditorías mediante scripts Bash.
- Resolver escenarios prácticos similares a los encontrados en el examen **RHCSA** y en entornos empresariales.

---

# Fin del capítulo

```text
72-redes-podman.md
```

**Próximo capítulo:**

```text
73-volumenes-persistencia-podman.md
```





