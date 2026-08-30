# 33. Rutas Estáticas (Static Routes)

> **Módulo 6: Redes en Red Hat Enterprise Linux**  
> **Página 33 del Manual RHCSA**

---

# Objetivos de aprendizaje

Al finalizar esta lección serás capaz de:

- Comprender qué es una ruta de red.
- Entender el funcionamiento de la tabla de enrutamiento.
- Visualizar las rutas configuradas en Linux.
- Agregar rutas estáticas temporales y permanentes.
- Configurar rutas mediante `ip` y `nmcli`.
- Verificar la conectividad utilizando rutas estáticas.
- Diagnosticar problemas comunes relacionados con el enrutamiento.

---

# ¿Qué es una ruta?

Una **ruta** indica el camino que debe seguir un paquete para llegar a una red de destino.

Cuando un servidor necesita comunicarse con otra red, consulta su **tabla de rutas** para decidir por dónde enviar el tráfico.

---

# Ejemplo de una red

```
                Router
          192.168.1.1
               │
 ┌─────────────┴─────────────┐
 │                           │
Servidor Linux           PC Cliente
192.168.1.100         192.168.1.50
```

Para comunicarse con redes externas, el servidor utiliza el **Gateway** o **Puerta de Enlace**.

---

# ¿Qué es una ruta estática?

Una **ruta estática** es una ruta configurada manualmente por el administrador del sistema.

A diferencia de los protocolos de enrutamiento dinámico (OSPF, BGP, RIP, etc.), las rutas estáticas:

- No cambian automáticamente.
- Son simples de administrar.
- Consumen pocos recursos.
- Son ideales para redes pequeñas o servidores.

---

# Ver la tabla de rutas

```bash
ip route
```

Ejemplo:

```
default via 192.168.1.1 dev ens160

192.168.1.0/24 dev ens160 proto kernel scope link src 192.168.1.100
```

---

# Interpretando la salida

```
default via 192.168.1.1
```

Significa:

```
Todo el tráfico desconocido
↓

Se envía al Gateway

192.168.1.1
```

---

# Ver únicamente la ruta por defecto

```bash
ip route | grep default
```

Resultado:

```
default via 192.168.1.1 dev ens160
```

---

# Ver rutas IPv6

```bash
ip -6 route
```

---

# Agregar una ruta temporal

Ejemplo:

```
Red destino:

10.10.20.0/24

Gateway:

192.168.1.254
```

Comando:

```bash
sudo ip route add \
10.10.20.0/24 \
via 192.168.1.254
```

---

# Verificar la ruta

```bash
ip route
```

Debe aparecer:

```
10.10.20.0/24 via 192.168.1.254
```

---

# Eliminar una ruta temporal

```bash
sudo ip route del \
10.10.20.0/24
```

---

# Agregar una ruta utilizando una interfaz

```bash
sudo ip route add \
172.16.0.0/16 \
dev ens160
```

---

# Agregar varias rutas

```bash
sudo ip route add 10.0.0.0/8 via 192.168.1.254

sudo ip route add 172.16.0.0/16 via 192.168.1.254

sudo ip route add 192.168.50.0/24 via 192.168.1.254
```

---

# Configurar una ruta permanente con nmcli

Las rutas agregadas con `ip route add` desaparecen después de reiniciar el sistema.

Para hacerlas permanentes:

```bash
sudo nmcli connection modify LAN \
+ipv4.routes "10.10.20.0/24 192.168.1.254"
```

---

# Aplicar la configuración

```bash
sudo nmcli connection down LAN

sudo nmcli connection up LAN
```

---

# Ver las rutas configuradas

```bash
nmcli connection show LAN
```

Buscar:

```
ipv4.routes
```

---

# Configurar varias rutas permanentes

```bash
sudo nmcli connection modify LAN \
+ipv4.routes "10.10.20.0/24 192.168.1.254"

sudo nmcli connection modify LAN \
+ipv4.routes "172.16.0.0/16 192.168.1.254"
```

---

# Eliminar una ruta permanente

```bash
sudo nmcli connection modify LAN \
-ipv4.routes "10.10.20.0/24 192.168.1.254"
```

Aplicar nuevamente la conexión.

---

# Ver información completa

```bash
nmcli device show
```

---

# Ver la puerta de enlace

```bash
ip route | grep default
```

---

# Comprobar conectividad

```bash
ping 10.10.20.10
```

---

# Ver el camino que siguen los paquetes

Si está instalado:

```bash
traceroute 10.10.20.10
```

---

# Consultar la configuración de la conexión

```bash
nmcli connection show LAN
```

---

# Diferencia entre rutas temporales y permanentes

| Temporal | Permanente |
|----------|------------|
| Se pierde al reiniciar | Sobrevive al reinicio |
| `ip route add` | `nmcli connection modify` |
| Ideal para pruebas | Ideal para producción |

---

# Escenario práctico

Servidor:

```
192.168.1.100
```

Gateway:

```
192.168.1.1
```

Existe otra red:

```
10.20.30.0/24
```

Accesible mediante:

```
192.168.1.254
```

Configuración:

```bash
sudo nmcli connection modify LAN \
+ipv4.routes "10.20.30.0/24 192.168.1.254"
```

---

# Verificar las rutas

```bash
ip route
```

Resultado:

```
default via 192.168.1.1

10.20.30.0/24 via 192.168.1.254

192.168.1.0/24 dev ens160
```

---

# Buenas prácticas RHCSA

✔ Configurar rutas permanentes mediante `nmcli`.

✔ Documentar todas las rutas agregadas.

✔ Comprobar siempre la conectividad después de modificar la tabla de rutas.

✔ Utilizar rutas temporales únicamente para pruebas.

✔ Mantener una única ruta por defecto, salvo que exista un diseño de red específico.

---

# Errores comunes

## Gateway incorrecto

La ruta no funcionará si el Gateway no es accesible.

Verificar:

```bash
ping 192.168.1.254
```

---

## Red equivocada

Ejemplo:

```
10.10.20.0/24
```

cuando realmente era:

```
10.10.2.0/24
```

Revisar cuidadosamente la red de destino.

---

## Ruta desaparece después del reinicio

Probablemente fue creada con:

```bash
ip route add
```

Debe configurarse mediante `nmcli`.

---

## No existe ruta por defecto

Verificar:

```bash
ip route
```

Debe existir una línea similar a:

```
default via 192.168.1.1
```

---

# Comandos importantes RHCSA

| Comando | Descripción |
|----------|-------------|
| `ip route` | Mostrar rutas IPv4 |
| `ip -6 route` | Mostrar rutas IPv6 |
| `ip route add` | Agregar ruta temporal |
| `ip route del` | Eliminar ruta temporal |
| `nmcli connection modify +ipv4.routes` | Agregar ruta permanente |
| `nmcli connection show` | Ver configuración de rutas |
| `traceroute` | Ver el recorrido de un paquete |
| `ping` | Verificar conectividad |

---

# Resumen

En esta lección aprendiste a:

- Comprender el funcionamiento de las rutas estáticas.
- Interpretar la tabla de enrutamiento de Linux.
- Agregar y eliminar rutas temporales.
- Configurar rutas permanentes mediante `nmcli`.
- Diagnosticar problemas de conectividad relacionados con el enrutamiento.
- Aplicar buenas prácticas para la administración de redes en Red Hat Enterprise Linux.

---

# Ejercicio práctico RHCSA

1. Visualiza la tabla de rutas actual utilizando `ip route`.
2. Identifica la ruta por defecto y el Gateway configurado.
3. Agrega una ruta temporal hacia la red `10.10.20.0/24` utilizando el Gateway `192.168.1.254`.
4. Verifica que la nueva ruta aparezca en la tabla de rutas.
5. Elimina la ruta temporal.
6. Configura la misma ruta de forma permanente utilizando `nmcli`.
7. Reactiva la conexión de red para aplicar los cambios.
8. Comprueba nuevamente la tabla de rutas.
9. Verifica la conectividad hacia un host de la red remota mediante `ping` o `traceroute`.
10. Elimina la ruta permanente y confirma que ya no aparece en la configuración.

> **Objetivo:** aprender a administrar rutas estáticas en Red Hat Enterprise Linux, comprendiendo la diferencia entre configuraciones temporales y permanentes, una competencia importante para el examen **RHCSA** y para la administración de servidores Linux en entornos empresariales.