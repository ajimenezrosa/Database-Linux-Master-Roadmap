# 37. Firewalld: Conceptos y Zonas

> **Módulo 7: Seguridad del Sistema**  
> **Página 37 del Manual RHCSA**

---

# Objetivos de aprendizaje

Al finalizar esta lección serás capaz de:

- Comprender qué es Firewalld.
- Entender cómo funciona un Firewall.
- Conocer las zonas de Firewalld.
- Identificar la zona activa de una interfaz.
- Cambiar zonas de red.
- Comprender la diferencia entre servicios y puertos.
- Aplicar buenas prácticas de seguridad utilizando zonas.

---

# ¿Qué es un Firewall?

Un **Firewall** es un mecanismo de seguridad que controla el tráfico de red que entra y sale del servidor.

Su función consiste en decidir qué conexiones están permitidas y cuáles deben bloquearse.

Ejemplo:

```
                Internet
                    │
                    ▼
          +------------------+
          |    Firewalld     |
          +------------------+
             │           │
     Permitido      Bloqueado
             │
             ▼
        Servidor Linux
```

---

# ¿Qué es Firewalld?

**Firewalld** es el firewall dinámico utilizado por Red Hat Enterprise Linux.

Permite administrar reglas sin reiniciar el servicio y utiliza **zonas** para aplicar diferentes niveles de confianza a las interfaces de red.

Características principales:

- Configuración dinámica.
- Soporte para IPv4 e IPv6.
- Administración mediante `firewall-cmd`.
- Integración con NetworkManager.
- Uso de zonas de seguridad.

---

# Verificar que Firewalld está instalado

```bash
rpm -q firewalld
```

Ejemplo:

```
firewalld-1.x.x
```

---

# Verificar el estado del servicio

```bash
systemctl status firewalld
```

Salida esperada:

```
Active: active (running)
```

---

# Iniciar Firewalld

```bash
sudo systemctl start firewalld
```

---

# Habilitar Firewalld al iniciar

```bash
sudo systemctl enable firewalld
```

---

# Reiniciar Firewalld

```bash
sudo systemctl restart firewalld
```

---

# Recargar la configuración

Cuando se modifican reglas normalmente basta con:

```bash
sudo firewall-cmd --reload
```

No es necesario reiniciar el servicio.

---

# ¿Qué son las zonas?

Las **zonas** representan distintos niveles de confianza para las conexiones de red.

Cada interfaz de red pertenece a una zona.

Cada zona tiene reglas diferentes.

---

# Zonas disponibles

Consultar:

```bash
firewall-cmd --get-zones
```

Ejemplo:

```
block

dmz

drop

external

home

internal

public

trusted

work
```

---

# Zona por defecto

Consultar:

```bash
firewall-cmd --get-default-zone
```

Normalmente:

```
public
```

---

# Zona activa

Consultar:

```bash
firewall-cmd --get-active-zones
```

Ejemplo:

```
public

interfaces:

ens160
```

---

# Significado de las zonas

## drop

La más restrictiva.

- Descarta todos los paquetes.
- No responde siquiera con mensajes ICMP.

Ideal para bloquear completamente un origen.

---

## block

Bloquea el tráfico entrante.

Responde indicando que el acceso está prohibido.

---

## public

Zona predeterminada.

Permite únicamente los servicios autorizados.

Es la más utilizada en servidores.

---

## external

Pensada para equipos conectados directamente a Internet.

Puede utilizar NAT y enrutamiento.

---

## dmz

Utilizada para servidores ubicados en una zona desmilitarizada (DMZ).

Ejemplo:

- Servidor Web
- Servidor DNS público

---

## work

Adecuada para redes corporativas.

Permite un nivel de confianza mayor que la zona pública.

---

## home

Pensada para redes domésticas.

Permite más servicios que la zona pública.

---

## internal

Utilizada en redes internas completamente confiables.

---

## trusted

La zona más permisiva.

Permite prácticamente todo el tráfico.

Debe utilizarse únicamente en redes totalmente seguras.

---

# Ver la configuración de una zona

Ejemplo:

```bash
firewall-cmd --zone=public --list-all
```

Salida:

```
public

services:

ssh dhcpv6-client

ports:

interfaces:

ens160
```

---

# Ver todas las zonas

```bash
firewall-cmd --list-all-zones
```

---

# Cambiar la zona de una interfaz

Ejemplo:

```bash
sudo firewall-cmd \
--zone=work \
--change-interface=ens160
```

---

# Hacer el cambio permanente

```bash
sudo firewall-cmd \
--permanent \
--zone=work \
--change-interface=ens160
```

Después:

```bash
sudo firewall-cmd --reload
```

---

# Asociar una interfaz a una zona

```bash
sudo firewall-cmd \
--zone=public \
--change-interface=ens160
```

---

# Consultar la zona de una interfaz

```bash
firewall-cmd --get-active-zones
```

---

# Ver las interfaces del sistema

```bash
ip link
```

---

# Diferencia entre servicio y puerto

Servicio:

```
SSH
```

↓

Puerto:

```
22/TCP
```

Servicio:

```
HTTP
```

↓

Puerto:

```
80/TCP
```

En Firewalld normalmente se administran **servicios**, no directamente los puertos.

---

# Ver servicios permitidos

```bash
firewall-cmd --list-services
```

Ejemplo:

```
ssh

dhcpv6-client
```

---

# Ver puertos abiertos

```bash
firewall-cmd --list-ports
```

---

# Ver la configuración completa

```bash
firewall-cmd --list-all
```

---

# Configuración temporal vs permanente

## Temporal

Se pierde al reiniciar.

Ejemplo:

```bash
firewall-cmd --add-service=http
```

---

## Permanente

Sobrevive al reinicio.

Ejemplo:

```bash
firewall-cmd \
--permanent \
--add-service=http
```

Después:

```bash
firewall-cmd --reload
```

---

# Relación con NetworkManager

Cuando una interfaz cambia de perfil en NetworkManager, puede cambiar también la zona asociada.

Consultar:

```bash
nmcli connection show
```

---

# Ejemplo práctico

Servidor Web

```
Interfaz:

ens160

Zona:

public

Servicios permitidos:

SSH

HTTP

HTTPS
```

Todo lo demás permanece bloqueado.

---

# Buenas prácticas RHCSA

✔ Utilizar la zona **public** para servidores normales.

✔ Abrir únicamente los servicios necesarios.

✔ Preferir servicios en lugar de puertos.

✔ Realizar cambios permanentes únicamente cuando se hayan probado correctamente.

✔ Verificar siempre la configuración después de realizar modificaciones.

✔ Mantener una única zona por interfaz, salvo casos específicos.

---

# Errores comunes

## El Firewall está detenido

Verificar:

```bash
systemctl status firewalld
```

---

## Cambios desaparecen tras reiniciar

Probablemente se realizaron sin:

```bash
--permanent
```

---

## La interfaz está en la zona incorrecta

Consultar:

```bash
firewall-cmd --get-active-zones
```

---

## Servicio permitido pero sigue sin responder

Comprobar:

- Que el servicio esté iniciado.
- Que el puerto esté escuchando.
- Que SELinux no esté bloqueando el acceso.

---

# Comandos importantes RHCSA

| Comando | Descripción |
|----------|-------------|
| `systemctl status firewalld` | Estado del Firewall |
| `firewall-cmd --state` | Verificar si Firewalld está activo |
| `firewall-cmd --get-default-zone` | Zona por defecto |
| `firewall-cmd --get-active-zones` | Zonas activas |
| `firewall-cmd --get-zones` | Listar zonas disponibles |
| `firewall-cmd --list-all` | Configuración de la zona activa |
| `firewall-cmd --list-all-zones` | Mostrar todas las zonas |
| `firewall-cmd --reload` | Recargar configuración |

---

# Resumen

En esta lección aprendiste a:

- Comprender el funcionamiento de Firewalld.
- Entender el concepto de zonas de seguridad.
- Identificar la zona activa de una interfaz.
- Cambiar la zona de una interfaz de red.
- Diferenciar entre servicios y puertos.
- Comprender la diferencia entre configuraciones temporales y permanentes.
- Aplicar buenas prácticas para administrar el Firewall en Red Hat Enterprise Linux.

---

# Laboratorio práctico RHCSA

## Escenario 1

Comprueba que Firewalld está activo.

```bash
systemctl status firewalld
```

---

## Escenario 2

Identifica la zona por defecto.

```bash
firewall-cmd --get-default-zone
```

---

## Escenario 3

Consulta la zona activa y la interfaz asociada.

```bash
firewall-cmd --get-active-zones
```

---

## Escenario 4

Visualiza la configuración completa de la zona **public**.

```bash
firewall-cmd --zone=public --list-all
```

---

## Escenario 5

Cambia temporalmente la interfaz `ens160` a la zona **work**, verifica el cambio y luego vuelve a dejarla en la zona **public**.

```bash
sudo firewall-cmd --zone=work --change-interface=ens160
```

Finalmente:

```bash
sudo firewall-cmd --zone=public --change-interface=ens160
```

> **Objetivo general:** comprender el funcionamiento de **Firewalld** y de las **zonas de seguridad**, base para administrar el acceso a los servicios del sistema. En la siguiente lección aprenderás a abrir y cerrar puertos y servicios mediante `firewall-cmd`, una de las tareas más frecuentes en el examen **RHCSA**.