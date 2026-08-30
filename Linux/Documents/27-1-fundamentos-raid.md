# 27.1 Fundamentos de RAID y conceptos de almacenamiento

RAID significa **Redundant Array of Independent Disks**. Es una técnica que combina varios dispositivos de almacenamiento para mejorar uno o más de estos objetivos:

- Disponibilidad.
- Tolerancia a fallos.
- Rendimiento.
- Capacidad.
- Continuidad operativa.

RAID no sustituye los respaldos. Un arreglo puede proteger frente al fallo de uno o más discos, pero no frente a borrado accidental, corrupción lógica, ransomware, errores administrativos o pérdida completa del servidor.

---

# Objetivos

Al finalizar esta sección podrás:

- Comprender qué es RAID.
- Diferenciar disponibilidad, redundancia y respaldo.
- Comparar RAID por hardware y por software.
- Identificar los niveles RAID más comunes.
- Calcular capacidad utilizable.
- Elegir un nivel RAID adecuado.
- Comprender striping, mirroring y parity.
- Reconocer limitaciones y riesgos.

---

# Conceptos fundamentales

## Disponibilidad

La disponibilidad describe la capacidad de un sistema para continuar ofreciendo servicio.

Un RAID puede mejorar la disponibilidad cuando un disco falla, pero no garantiza disponibilidad total.

La disponibilidad también depende de:

- Controladora.
- Fuente de poder.
- Sistema operativo.
- Aplicación.
- Red.
- Personal operativo.
- Procedimientos de recuperación.

## Redundancia

La redundancia consiste en mantener datos adicionales o copias que permiten tolerar fallos.

Ejemplo:

```text
Disco 1: datos
Disco 2: copia de los datos
```

Esto corresponde al principio de RAID 1.

## Rendimiento

Algunos niveles RAID distribuyen operaciones entre discos.

Ejemplo:

```text
Bloque A → Disco 1
Bloque B → Disco 2
Bloque C → Disco 3
```

Esto corresponde al principio de striping.

## Tolerancia a fallos

Es la capacidad de continuar operando después de perder uno o más discos.

| Nivel | Fallos tolerados |
|---|---:|
| RAID 0 | 0 |
| RAID 1 | 1 por espejo |
| RAID 5 | 1 |
| RAID 6 | 2 |
| RAID 10 | Puede tolerar varios, según qué discos fallen |

---

# RAID no es backup

RAID puede proteger frente a:

- Fallo físico de un disco.
- Ciertos errores de lectura.
- Degradación de un miembro.
- Interrupciones por reemplazo de discos.

RAID no protege frente a:

- `rm -rf`.
- DROP DATABASE.
- Corrupción lógica.
- Ransomware.
- Error de aplicación.
- Sobrescritura.
- Incendio o robo.
- Fallo simultáneo superior a la tolerancia.
- Corrupción replicada a todos los miembros.

Debe mantenerse una estrategia de respaldo independiente.

---

# Hardware RAID frente a Software RAID

## Hardware RAID

Se implementa mediante una controladora especializada.

Ventajas:

- Descarga trabajo de la CPU.
- Caché protegida por batería o flash.
- Administración centralizada.
- Puede ofrecer mejor rendimiento.
- Abstracción transparente al sistema operativo.

Desventajas:

- Costo.
- Dependencia de la controladora.
- Riesgo de incompatibilidad.
- Recuperación más compleja si falla la controladora.
- Firmware propietario.

## Software RAID

En Linux se administra normalmente con:

```text
mdadm
```

Ventajas:

- Bajo costo.
- Independencia de controladora RAID.
- Transparencia.
- Flexibilidad.
- Buena integración con Linux.
- Fácil migración entre equipos compatibles.

Desventajas:

- Consume recursos de CPU.
- Depende del sistema operativo.
- Requiere administración cuidadosa.
- El arranque puede requerir initramfs actualizado.

---

# Componentes de mdadm

| Elemento | Descripción |
|---|---|
| `/dev/md0` | Dispositivo RAID |
| Miembros | Discos o particiones |
| Superblock | Metadatos del RAID |
| Nivel | RAID 0, 1, 5, 6, 10 |
| Chunk | Tamaño de fragmento |
| Bitmap | Seguimiento de cambios |
| Spare | Disco de reserva |
| `/proc/mdstat` | Estado del RAID |
| `/etc/mdadm.conf` | Configuración persistente |

---

# Striping

Striping distribuye bloques entre varios discos.

```text
Disco 1: A1 A3 A5
Disco 2: A2 A4 A6
```

Ventaja:

- Mayor rendimiento.

Desventaja:

- Sin redundancia.

RAID 0 utiliza striping.

---

# Mirroring

Mirroring mantiene copias idénticas.

```text
Disco 1: A B C D
Disco 2: A B C D
```

Ventaja:

- Tolerancia a fallos.

Desventaja:

- Solo se utiliza una parte de la capacidad total.

RAID 1 utiliza mirroring.

---

# Paridad

La paridad permite reconstruir datos faltantes.

Ejemplo simplificado:

```text
Datos A + Datos B = Paridad P
```

Si se pierde A, puede reconstruirse usando B y P.

RAID 5 y RAID 6 utilizan paridad distribuida.

---

# RAID 0

## Características

- Requiere mínimo 2 discos.
- No ofrece redundancia.
- Utiliza toda la capacidad.
- Alto rendimiento secuencial.
- Si falla un disco, se pierde el arreglo.

## Capacidad

```text
Número de discos × tamaño del disco más pequeño
```

Ejemplo:

```text
2 discos de 1 TB = 2 TB útiles
```

## Uso recomendado

- Datos temporales.
- Caché.
- Scratch space.
- Cargas donde la pérdida sea aceptable.

---

# RAID 1

## Características

- Requiere mínimo 2 discos.
- Datos duplicados.
- Buena tolerancia.
- Lecturas potencialmente rápidas.
- Escrituras duplicadas.

## Capacidad

```text
Tamaño del disco más pequeño
```

Ejemplo:

```text
2 discos de 1 TB = 1 TB útil
```

## Uso recomendado

- Sistema operativo.
- Datos críticos pequeños.
- Servidores donde prima simplicidad.
- Volúmenes de arranque.

---

# RAID 5

## Características

- Requiere mínimo 3 discos.
- Tolera un fallo.
- Paridad distribuida.
- Buena eficiencia de capacidad.
- Escrituras penalizadas por cálculo de paridad.

## Capacidad

```text
(N - 1) × tamaño del disco más pequeño
```

Ejemplo:

```text
4 discos de 2 TB = 6 TB útiles
```

## Riesgos

- Reconstrucciones largas.
- Riesgo elevado con discos grandes.
- Fallo de segundo disco durante rebuild provoca pérdida.

---

# RAID 6

## Características

- Requiere mínimo 4 discos.
- Tolera dos fallos.
- Doble paridad.
- Mayor seguridad que RAID 5.
- Mayor penalización de escritura.

## Capacidad

```text
(N - 2) × tamaño del disco más pequeño
```

Ejemplo:

```text
6 discos de 2 TB = 8 TB útiles
```

---

# RAID 10

RAID 10 combina mirroring y striping.

## Características

- Requiere mínimo 4 discos.
- Excelente rendimiento.
- Buena tolerancia.
- Reconstrucción rápida.
- Capacidad aproximada del 50%.

## Capacidad

```text
(N / 2) × tamaño del disco más pequeño
```

Ejemplo:

```text
4 discos de 2 TB = 4 TB útiles
```

## Uso recomendado

- Bases de datos.
- Virtualización.
- Cargas intensivas.
- Sistemas transaccionales.

---

# RAID 4

RAID 4 utiliza striping con un disco dedicado de paridad.

No es habitual porque el disco de paridad se convierte en cuello de botella.

---

# Linear Mode

Combina discos de forma secuencial.

```text
Disco 1 se llena
        ↓
Disco 2
        ↓
Disco 3
```

No proporciona redundancia.

Es similar a concatenar almacenamiento.

---

# Tabla comparativa

| Nivel | Mínimo discos | Fallos tolerados | Capacidad | Rendimiento | Uso típico |
|---|---:|---:|---|---|---|
| RAID 0 | 2 | 0 | 100% | Muy alto | Temporal |
| RAID 1 | 2 | 1 | 50% | Alto en lectura | Sistema y datos críticos |
| RAID 5 | 3 | 1 | N-1 | Medio | Archivos |
| RAID 6 | 4 | 2 | N-2 | Medio-bajo en escritura | Archivo crítico |
| RAID 10 | 4 | Variable | 50% | Muy alto | Bases de datos |
| Linear | 2 | 0 | 100% | Normal | Concatenación |

---

# Tamaño de chunk

El chunk define cuánto dato se escribe en un disco antes de pasar al siguiente.

Ejemplo:

```text
Chunk = 512 KiB
```

Debe seleccionarse según:

- Patrón de I/O.
- Tamaño de bloque.
- Sistema de archivos.
- Aplicación.
- Número de discos.

No existe un valor universal.

---

# Write hole

El write hole ocurre cuando se interrumpe una escritura y datos y paridad quedan inconsistentes.

Puede mitigarse mediante:

- Bitmap.
- Journal.
- Caché protegida.
- UPS.
- Sistemas de archivos con journaling.
- Controladoras avanzadas.

---

# Bitmaps

Un bitmap registra qué regiones fueron modificadas.

Puede acelerar reconstrucciones después de una interrupción.

Tipos:

- Interno.
- Externo.

Ejemplo:

```bash
sudo mdadm --grow /dev/md0 --bitmap=internal
```

---

# Discos spare

Un spare es un disco de reserva.

```text
RAID activo + disco spare
```

Cuando falla un miembro, el spare puede entrar automáticamente en reconstrucción.

---

# Riesgos de mezclar tamaños

Si se mezclan discos:

```text
500 GB
1 TB
2 TB
```

el RAID utiliza como referencia el disco más pequeño.

En RAID 5 con tres discos:

```text
(3 - 1) × 500 GB = 1 TB útil
```

El espacio sobrante de los discos grandes no se aprovecha dentro del mismo arreglo.

---

# Riesgos de mezclar velocidades

Mezclar discos de diferentes velocidades puede provocar que el arreglo quede limitado por el más lento.

También puede complicar:

- Latencia.
- Rebuild.
- Monitoreo.
- Predicción de fallos.

---

# RAID y discos SSD/NVMe

RAID funciona con SSD y NVMe, pero deben considerarse:

- Desgaste.
- TRIM.
- Write amplification.
- Latencia.
- Temperatura.
- Firmware.
- Resistencia de escritura.
- Compatibilidad del kernel.

---

# RAID y bases de datos

Para bases de datos:

- RAID 10 suele ser preferido por rendimiento y reconstrucción.
- RAID 5 puede penalizar escrituras.
- RAID 6 ofrece más tolerancia, pero más costo de escritura.
- Debe mantenerse respaldo externo.
- Debe monitorearse latencia, no solo capacidad.
- Deben alinearse bloques, chunk y stripe unit.

---

# Buenas prácticas

- Utiliza discos de tamaño y rendimiento similares.
- Mantén respaldos independientes.
- Documenta seriales y posiciones físicas.
- Configura alertas.
- Prueba reemplazos.
- Revisa `/proc/mdstat`.
- Configura `mdadm.conf`.
- Actualiza initramfs cuando corresponda.
- Realiza scrubbing periódico.
- Evita RAID 5 para cargas críticas de alta escritura sin análisis.
- No uses RAID 0 para datos importantes.
- Mantén repuestos compatibles.

---

# Resumen

En esta sección aprendiste:

- Qué es RAID.
- Diferencias entre redundancia y respaldo.
- Hardware RAID y Software RAID.
- Striping, mirroring y parity.
- Niveles RAID principales.
- Capacidad utilizable.
- Tolerancia a fallos.
- Riesgos y criterios de selección.
