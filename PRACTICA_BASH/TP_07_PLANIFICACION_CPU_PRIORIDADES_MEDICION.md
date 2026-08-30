# Trabajo Práctico 07 — Planificación: CPU, prioridades y medición

**Arquitectura y Sistemas Operativos**  
**Tecnicatura Universitaria en Programación**

---

## Propósito

Cuando varios procesos están listos, el sistema operativo debe decidir qué tarea utiliza
cada CPU lógica.

En este TP combinaremos:

- teoría de planificación;
- procesos CPU-bound;
- `nice` y `renice`;
- afinidad con `taskset`;
- medición con `ps` y `top`;
- scripting para tomar varias muestras.

El TP acompaña la **[Clase 7 — Planificación de CPU](../MANUAL/CLASE_07_PLANIFICACION_CPU.md)**.

---

## Conceptos Bash que incorporamos

```text
for
seq
sleep
comparaciones numéricas
funciones
ps --sort
nice
renice
taskset
uptime
```

---

## Parte 1 — Procesos y CPU

Ejecutá:

```bash
ps -eo pid,ppid,ni,pri,psr,stat,%cpu,comm --sort=-%cpu | head
```

Identificá:

```text
PID
NI
PRI
PSR
STAT
%CPU
```

Respondé:

1. ¿Qué representa `NI`?
2. ¿Qué representa `PRI`?
3. ¿Qué representa `PSR`?
4. ¿Por qué una fotografía de `ps` no describe por sí sola toda la planificación?

---

## Parte 2 — Carga del sistema

Ejecutá:

```bash
uptime
```

Luego:

```bash
nproc
```

Respondé:

1. ¿Cuántas CPU lógicas ve Linux?
2. ¿Qué información general proporciona `uptime`?
3. ¿Por qué una carga debe interpretarse teniendo en cuenta la cantidad de CPU?

---

## Parte 3 — Crear una tarea CPU-bound

Ejecutá:

```bash
while true; do :; done &
p1=$!
```

Observá:

```bash
ps -p "$p1" -o pid,ni,pri,psr,stat,%cpu,comm
```

Terminá:

```bash
kill "$p1"
wait "$p1" 2>/dev/null
```

No dejes procesos de prueba activos.

---

## Parte 4 — `nice`

Creá nuevamente una tarea:

```bash
nice -n 10 bash -c 'while true; do :; done' &
p2=$!
```

Observá:

```bash
ps -p "$p2" -o pid,ni,pri,psr,stat,%cpu,comm
```

Respondé:

1. ¿Qué valor `NI` aparece?
2. ¿`nice 10` significa “prioridad absoluta 10”?
3. ¿Qué intención expresa un valor nice mayor?

Terminá el proceso.

---

## Parte 5 — Competencia en una misma CPU

Verificá que exista `taskset`:

```bash
command -v taskset
```

Si está disponible:

```bash
taskset -c 0 bash -c 'while true; do :; done' &
p1=$!

taskset -c 0 nice -n 10 bash -c 'while true; do :; done' &
p2=$!
```

Observá varias veces:

```bash
ps -p "$p1,$p2" -o pid,ni,pri,psr,stat,%cpu,comm
```

Después:

```bash
kill "$p1" "$p2"
wait "$p1" "$p2" 2>/dev/null
```

Respondé:

1. ¿Por qué fijamos ambos a la misma CPU?
2. ¿Qué tendencia observaste?
3. ¿Por qué no esperamos porcentajes exactos?
4. ¿Qué cambia en WSL2 respecto de Linux nativo?

---

## Parte 6 — `renice`

Creá:

```bash
sleep 300 &
pid=$!
```

Observá:

```bash
ps -p "$pid" -o pid,ni,pri,stat,comm
```

Modificá:

```bash
renice 10 -p "$pid"
```

Volvé a observar y finalizá el proceso.

Respondé:

> ¿Qué diferencia existe entre `nice` y `renice`?

---

## Parte 7 — Afinidad

Con un proceso activo:

```bash
taskset -pc "$pid"
```

Si tenés al menos dos CPU lógicas:

```bash
taskset -pc 0,1 "$pid"
```

Respondé:

> ¿Por qué conservar una tarea cerca de determinadas CPU puede relacionarse con la
> localidad de caché?

---

## Parte 8 — Primer bucle `for`

Creá:

```text
muestras.sh
```

Contenido inicial:

```bash
#!/bin/bash

for numero in 1 2 3 4 5
do
    echo "Muestra $numero"
    date
    uptime
    echo
    sleep 1
done
```

Ejecutá:

```bash
chmod +x muestras.sh
./muestras.sh
```

Respondé:

> ¿Qué ventaja aporta un bucle frente a escribir cinco veces los mismos comandos?

---

## Parte 9 — Cantidad de muestras como parámetro

Modificá el script para usar:

```bash
cantidad="$1"
```

Validá que se haya recibido el dato.

Podés iterar:

```bash
for numero in $(seq 1 "$cantidad")
do
    ...
done
```

Uso:

```bash
./muestras.sh 10
```

El script debe rechazar la ejecución si no recibe la cantidad.

---

## Parte 10 — Monitor de un PID

Creá:

```text
monitor_cpu.sh
```

Uso:

```bash
./monitor_cpu.sh PID CANTIDAD
```

En cada muestra debe mostrar:

```text
hora
PID
NI
PRI
PSR
estado
%CPU
```

Podés usar:

```bash
ps -p "$pid" -o pid,ni,pri,psr,stat,%cpu,comm
```

Entre muestras:

```bash
sleep 1
```

Debe detenerse con un mensaje claro si el proceso deja de existir.

---

## Parte 11 — Algoritmos clásicos

Resolver en `RESPUESTAS.md`:

| Proceso | Llegada | Ráfaga |
| ------- | ------: | -----: |
| P1      |       0 |      8 |
| P2      |       1 |      4 |
| P3      |       2 |      2 |

Calcular para:

1. FCFS.
2. SJF no apropiativo.
3. SRTF.
4. Round Robin con quantum 2.

Para cada uno:

- Gantt;
- finalización;
- turnaround;
- waiting time;
- response time.

Después respondé:

> ¿Por qué el scheduler real de Linux no debe imaginarse simplemente como uno de estos
> algoritmos aplicado de forma literal?

---

## Desafío integrador — Comparar dos tareas

Creá un procedimiento documentado que:

1. lance dos procesos CPU-bound;
2. los fije a una misma CPU cuando `taskset` esté disponible;
3. dé nice distinto a cada uno;
4. tome 10 muestras con tu `monitor_cpu.sh`;
5. guarde las observaciones en archivos;
6. finalice ambos procesos correctamente.

En `RESPUESTAS.md` escribí una conclusión de no más de 10 líneas.

---

## Entrega

```text
TP_07/
├── RESPUESTAS.md
├── muestras.sh
├── monitor_cpu.sh
├── proceso_1.txt
└── proceso_2.txt
```

---

## Criterios de revisión

Se evaluará:

- interpretación de planificación;
- `nice`, `renice` y afinidad;
- diferencia entre observación y demostración;
- uso correcto de bucles;
- parámetros;
- procesos de prueba controlados;
- resolución de algoritmos clásicos;
- calidad de las conclusiones.

> Medir un sistema no consiste solamente en obtener números:
>
> **hay que saber qué representan y qué no demuestran.**
