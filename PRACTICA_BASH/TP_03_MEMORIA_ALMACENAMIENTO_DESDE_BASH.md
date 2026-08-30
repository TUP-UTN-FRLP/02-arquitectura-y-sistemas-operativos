# Trabajo Práctico 03 — Memoria y almacenamiento desde Bash

**Arquitectura y Sistemas Operativos**  
**Tecnicatura Universitaria en Programación**

---

## Propósito

En este TP utilizaremos Linux y Bash para observar:

- memoria principal;
- swap;
- cachés;
- dispositivos de bloques;
- sistemas de archivos montados;
- capacidad utilizada;
- diferencias entre WSL2 y Linux nativo.

El TP acompaña la **[Clase 3 — Memoria, almacenamiento, buses y firmware](../MANUAL/CLASE_03_MEMORIA_BUSES_FIRMWARE.md)**.

Seguimos trabajando con Bash de forma interactiva.

Todavía no programamos scripts `.sh`.

---

## Parte 1 — Preparar el espacio de trabajo

```bash
mkdir -p ~/aso/tp03
cd ~/aso/tp03
code .
```

Creá `RESPUESTAS.md`.

---

## Parte 2 — Memoria con `free`

Ejecutá:

```bash
free -h
```

Identificá:

```text
total
used
free
available
swap
```

Respondé:

1. ¿Qué diferencia observás entre `free` y `available`?
2. ¿Por qué un sistema puede usar RAM como caché?
3. ¿Memoria utilizada significa necesariamente “memoria desperdiciada”?
4. ¿Existe swap en tu entorno?

---

## Parte 3 — `/proc/meminfo`

Ejecutá:

```bash
cat /proc/meminfo
```

Ahora filtrá:

```bash
grep -E 'MemTotal|MemAvailable|SwapTotal|SwapFree' /proc/meminfo
```

Respondé:

1. ¿Qué ventaja tiene `grep -E` en este caso?
2. ¿Qué información resultó más fácil de leer?
3. ¿Por qué `/proc` resulta útil para programadores y administradores?

---

## Parte 4 — Guardar un diagnóstico de memoria

Ejecutá:

```bash
echo "=== MEMORIA ===" > memoria.txt
free -h >> memoria.txt
echo "=== DETALLE ===" >> memoria.txt
grep -E 'MemTotal|MemAvailable|SwapTotal|SwapFree' /proc/meminfo >> memoria.txt
cat memoria.txt
```

Explicá qué función cumplen `>` y `>>`.

---

## Parte 5 — Dispositivos de bloques

Ejecutá:

```bash
lsblk
```

Después:

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS
```

Respondé:

1. ¿Qué representa `NAME`?
2. ¿Qué representa `SIZE`?
3. ¿Qué significa `TYPE`?
4. ¿Qué significa `FSTYPE`?
5. ¿Qué significa `MOUNTPOINTS`?

---

## Parte 6 — ¿Disco o sistema de archivos?

Ejecutá:

```bash
df -h
```

Compará con:

```bash
lsblk
```

Respondé:

> ¿Por qué `lsblk` y `df` no muestran exactamente el mismo tipo de información?

Intentá expresarlo así:

```text
lsblk → ...
df    → ...
```

---

## Parte 7 — Uso de una carpeta con `du`

Ejecutá:

```bash
du -sh ~
du -sh ~/aso/tp03
```

Respondé:

1. ¿Qué mide `du`?
2. ¿Qué diferencia existe entre `du` y `df`?
3. ¿Cuál usarías para saber cuánto ocupa un proyecto?
4. ¿Cuál usarías para saber cuánto espacio libre queda en un sistema de archivos?

---

## Parte 8 — Montajes

Ejecutá:

```bash
findmnt
```

Si estás en WSL2:

```bash
ls /mnt
```

y, si existe:

```bash
findmnt /mnt/c
```

Respondé:

1. ¿Qué significa montar un sistema de archivos?
2. ¿Qué representa `/mnt/c` en WSL2?
3. ¿Por qué un Linux nativo puede mostrar resultados diferentes?

---

## Parte 9 — Identificar dispositivos rotacionales

Probá:

```bash
lsblk -d -o NAME,ROTA,TRAN,SIZE,MODEL
```

Si tu entorno no informa todos los campos, registralo como observación.

Interpretación habitual:

```text
ROTA = 1 → rotacional
ROTA = 0 → no rotacional
```

Respondé:

1. ¿Qué tipo de dispositivo esperás que sea rotacional?
2. ¿Por qué un SSD presenta otra característica?
3. ¿WSL2 necesariamente expone el dispositivo físico real?

---

## Parte 10 — Pipes encadenados

Probá:

```bash
lsblk -o NAME,SIZE,TYPE | grep disk
```

Y:

```bash
df -h | grep -v tmpfs
```

La opción `-v` de `grep` invierte el filtro.

Respondé:

1. ¿Qué hace el primer comando?
2. ¿Qué hace el segundo?
3. ¿Por qué encadenar herramientas pequeñas puede resultar útil?

---

## Parte 11 — `head` y `tail`

Probá:

```bash
head /proc/meminfo
tail /proc/meminfo
head -n 5 memoria.txt
```

Respondé:

1. ¿Qué hace `head`?
2. ¿Qué hace `tail`?
3. ¿En qué tipo de archivos de programación podrían ser útiles?

Pensá especialmente en:

```text
logs
resultados
archivos de datos
```

---

## Parte 12 — `wc`

Contá líneas, palabras y bytes:

```bash
wc -l memoria.txt
wc -w memoria.txt
wc -c memoria.txt
```

Respondé:

> ¿Qué información podría obtener un programador con `wc` sobre un archivo de log, un CSV o un archivo de código?

---

## Parte 13 — Filtrar y contar

Probá:

```bash
grep -i "mem" /proc/meminfo | wc -l
```

Interpretación:

```text
grep
    → encuentra líneas

wc -l
    → las cuenta
```

Este es un patrón muy común en Bash:

```text
generar
→ filtrar
→ transformar
→ contar
```

---

## Parte 14 — Reflexión sobre herramientas

Con un solo comando pudiste contar las líneas de un archivo.

Respondé en `RESPUESTAS.md`:

> `wc -l memoria.txt` resuelve este conteo en una palabra.
> ¿Para qué tipo de tareas crees que eso es suficiente?
> ¿Qué tipo de problemas necesitarían algo más que combinar
> comandos de la terminal?

No hay una respuesta incorrecta. El objetivo es empezar a distinguir
cuándo alcanza con herramientas existentes y cuándo hace falta
construir algo propio.

---

## Desafío integrador

Construí, sin utilizar scripts, un archivo:

```text
diagnostico_memoria_disco.txt
```

que contenga:

```text
=== MEMORIA ===
resumen de memoria

=== SWAP ===
información de swap

=== DISCOS ===
dispositivos de bloques

=== SISTEMAS DE ARCHIVOS ===
uso de espacio
```

Debés utilizar desde la terminal:

```text
echo
free
grep
lsblk
df
>
>>
|
```

---

## Entrega

```text
TP_03/
├── RESPUESTAS.md
├── memoria.txt
└── diagnostico_memoria_disco.txt
```

---

## Criterios de revisión

Se observará:

- interpretación de memoria y almacenamiento;
- diferenciación entre `lsblk`, `df` y `du`;
- uso de montajes;
- uso de `grep`, `head`, `tail` y `wc`;
- capacidad para encadenar herramientas;
- reflexión sobre elección de herramientas para distintos tipos de tareas;
- interpretación de diferencias entre Linux nativo y WSL2.

> Bash empieza a mostrarnos una idea importante: **muchos problemas pequeños pueden resolverse combinando programas existentes en lugar de escribir un programa nuevo desde cero**.
