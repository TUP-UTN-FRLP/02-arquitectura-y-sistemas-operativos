# Trabajo Práctico 09 — Memoria virtual y diagnóstico de procesos

**Arquitectura y Sistemas Operativos**  
**Tecnicatura Universitaria en Programación**

---

## Propósito

En Arquitectura observamos la RAM. Ahora queremos comprender la memoria desde la mirada
del sistema operativo:

```text
proceso
→ direcciones virtuales
→ páginas
→ memoria física
```

El TP combina teoría de paginación con herramientas reales para inspeccionar memoria.

El TP acompaña la **[Clase 9 — Memoria: organización, administración y seguridad](../MANUAL/CLASE_09_MEMORIA_ORGANIZACION_ADMINISTRACION_SEGURIDAD.md)**.

---

## Conceptos Bash que incorporamos o reforzamos

```text
while
read
validación de PID
/proc/PID/status
/proc/PID/maps
free
vmstat
swapon
pmap
redirección a log
```

---

## Parte 1 — Estado general

Ejecutá:

```bash
free -h
swapon --show
getconf PAGESIZE
```

Respondé:

1. ¿Cuánta RAM ve Linux?
2. ¿Existe swap?
3. ¿Cuál es el tamaño de página base informado?
4. ¿Por qué memoria virtual y swap no son sinónimos?

---

## Parte 2 — `vmstat`

Ejecutá:

```bash
vmstat 1 5
```

Observá:

```text
r
si
so
us
sy
id
wa
```

Respondé:

1. ¿Qué indican `si` y `so`?
2. ¿Qué diferencia hay entre CPU de usuario y de sistema?
3. ¿Por qué una sola muestra no alcanza para diagnosticar un problema?

---

## Parte 3 — Memoria de Bash

Ejecutá:

```bash
echo $$
```

Después:

```bash
grep -E '^(Name|Pid|VmSize|VmRSS|VmSwap|Threads):' /proc/$$/status
```

Y:

```bash
head /proc/$$/maps
```

Respondé:

1. ¿Por qué `VmSize` no equivale a RAM consumida?
2. ¿Qué representa aproximadamente `VmRSS`?
3. ¿Por qué aparecen muchos mapeos?

---

## Parte 4 — Proceso controlado

Lanzamos un proceso en segundo plano para observar cómo Linux
expone su información de memoria en `/proc`.

```bash
sleep 120 &
pid=$!
```

Observá:

```bash
grep -E '^(Name|Pid|VmSize|VmRSS|VmSwap):' /proc/$pid/status
```

Y:

```bash
head /proc/$pid/maps
```

Al terminar:

```bash
kill "$pid"
wait "$pid" 2>/dev/null
```

---

## Parte 5 — Permisos de los mapeos

En `/proc/PID/maps` buscá permisos como:

```text
r--p
r-xp
rw-p
```

Respondé:

1. ¿Qué significan `r`, `w` y `x`?
2. ¿Por qué una región de código suele no ser escribible?
3. ¿Cómo contribuyen estos permisos al aislamiento?

---

## Parte 6 — `pmap`

Si está disponible:

```bash
pmap -x "$pid" | tail
```

Respondé:

> ¿Qué demuestra `pmap` respecto del modelo simplificado `text + data + heap + stack`?

---

## Parte 7 — Primer `while`

Creá:

```text
contador.sh
```

```bash
#!/bin/bash

contador=1

while [ "$contador" -le 5 ]
do
    echo "Muestra $contador"
    free -h | head -n 2
    echo
    contador=$((contador + 1))
    sleep 1
done
```

Respondé:

1. ¿Qué diferencia existe entre `while` y el `for` utilizado en el TP anterior?
2. ¿Qué significa `$((...))`?

---

## Parte 8 — Monitor de memoria de un proceso

Creá:

```text
monitor_memoria.sh
```

Uso:

```bash
./monitor_memoria.sh PID
```

Debe:

1. validar el PID;
2. mientras exista `/proc/PID`, mostrar cada dos segundos:

```text
hora
VmSize
VmRSS
VmSwap
Threads
```

3. terminar automáticamente cuando desaparezca el proceso.

Para evitar un bucle infinito durante la corrección, agregá además un máximo de 30
muestras.

---

## Parte 9 — Guardar muestras

Permití:

```bash
./monitor_memoria.sh PID > memoria_pid.log
```

Después analizá:

```bash
head memoria_pid.log
tail memoria_pid.log
wc -l memoria_pid.log
```

Respondé:

> ¿Por qué los logs son una forma natural de combinar scripting Bash con análisis
> posterior?

---

## Parte 10 — Paginación en papel

Supongamos:

```text
tamaño de página = 1024 bytes
dirección virtual = 2500
```

Calculá:

```text
página
offset
```

Si:

```text
página 2 → marco 7
```

calculá la dirección física conceptual.

Explicá por qué el offset no cambia.

---

## Parte 11 — Reemplazo de páginas

Con tres marcos y:

```text
1 2 3 1 4 1 2 5
```

simulá:

1. FIFO;
2. LRU.

Para cada acceso registrá:

- contenido de los marcos;
- fallo sí/no.

Respondé:

> ¿Por qué una política distinta puede cambiar el rendimiento aunque la secuencia de
> referencias sea idéntica?

---

## Parte 12 — Page fault

Explicá la diferencia entre:

```text
page fault por página válida no residente
page fault por copy-on-write
page fault por archivo mapeado
acceso inválido
```

Y respondé:

> ¿Por qué `page fault` no significa siempre “error” ni “leer desde swap”?

---

## Desafío integrador — `memoria_proceso.sh`

Creá:

```text
memoria_proceso.sh
```

Uso:

```bash
./memoria_proceso.sh PID
```

Debe mostrar:

```text
Nombre
PID
PPID
Threads
VmSize
VmRSS
VmSwap
primeros 10 mapeos
```

Debe validar:

- parámetro presente;
- PID existente;
- permiso de lectura.

---

## Entrega

```text
TP_09/
├── RESPUESTAS.md
├── contador.sh
├── monitor_memoria.sh
├── memoria_proceso.sh
└── memoria_pid.log
```

---

## Criterios de revisión

Se observará:

- diferencia entre memoria física y virtual;
- interpretación de `VmSize`, `VmRSS` y `VmSwap`;
- uso de `/proc/PID`;
- comprensión de páginas y marcos;
- page faults;
- algoritmos de reemplazo;
- uso correcto de `while`;
- control de bucles y experimentos acotados.

> No vamos a llenar deliberadamente la RAM:
>
> **un buen laboratorio observa un fenómeno sin poner en riesgo el entorno de trabajo.**
