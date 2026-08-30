**Arquitectura y Sistemas Operativos**

# Clase 7 — Planificación de CPU

**Tecnicatura Universitaria en Programación**

---

## Objetivos de la clase

Al finalizar esta clase deberías poder:

- explicar por qué el sistema operativo necesita planificar el uso de la CPU;
- diferenciar planificador (*scheduler*) y despachador (*dispatcher*);
- reconocer planificación apropiativa y no apropiativa;
- distinguir ráfagas de CPU y de entrada/salida;
- utilizar criterios básicos para evaluar algoritmos de planificación;
- resolver ejercicios sencillos con FCFS, SJF, SRTF, Round Robin y prioridades;
- comprender los problemas de efecto convoy e inanición;
- explicar el papel del quantum;
- relacionar prioridades, afinidad y planificación multinúcleo;
- observar desde Linux prioridades, CPU utilizada y decisiones del planificador.

---

## 1. El problema de la planificación

En la [clase anterior](CLASE_06_PROCESOS_ESTADOS_PCB.md) estudiamos:

- programa y proceso;
- PID y PPID;
- estados;
- PCB;
- bloqueo;
- cambio de contexto.

Vimos que puede haber muchas tareas listas para ejecutar.

La pregunta ahora es:

> **si varias tareas están listas, ¿cuál recibe una CPU?**

Esa decisión forma parte de la **planificación de CPU**.

En un sistema con una sola CPU lógica solamente puede existir un flujo de ejecución
utilizando esa CPU en un instante determinado.

En una máquina con varias CPU lógicas puede haber varias tareas ejecutándose realmente
al mismo tiempo, pero normalmente existen muchas más tareas que CPU disponibles.

Por eso el sistema operativo debe decidir continuamente:

```text
tareas listas
    │
    ▼
planificador
    │
    ▼
CPU disponible
```

---

## 2. ¿Planificamos procesos o hilos?

En los algoritmos clásicos suele hablarse de:

```text
proceso P1
proceso P2
proceso P3
```

Es una simplificación didáctica muy útil.

En sistemas modernos la unidad que el planificador ejecuta suele ser una **tarea o hilo
planificable**.

Un proceso con varios hilos puede tener varios flujos que compiten por CPU.

Por ahora resolveremos los algoritmos utilizando procesos porque simplifica los
ejercicios.

Cuando estudiemos hilos retomaremos esta precisión.

---

## 3. Scheduler y dispatcher

Conviene distinguir dos conceptos.

### Scheduler — planificador

Decide **qué tarea debería ejecutarse**.

```text
¿Cuál sigue?
```

### Dispatcher — despachador

Realiza las operaciones necesarias para que la tarea elegida comience o continúe
ejecutándose.

Puede participar en:

- cambio de contexto;
- cambio de modo;
- restauración de registros;
- transferencia de control.

En forma conceptual:

```text
cola de listos
     │
     ▼
 scheduler
     │ elige
     ▼
 dispatcher
     │
     ▼
    CPU
```

---

## 4. ¿Cuándo debe tomar decisiones el planificador?

![Diagrama de estados de un proceso: Nuevo pasa a Listo cuando el proceso es creado; Listo pasa a Ejecutando al asignarse el procesador; Ejecutando vuelve a Listo si es interrumpido, pasa a Bloqueado ante un pedido a componentes de E/S y a Finalizado a la salida; Bloqueado vuelve a Listo cuando los pedidos se completan](img/clase07/estados-proceso-planificador.png)

*El planificador puede intervenir en cada cambio de estado: cuando una tarea pasa de Ejecutando a Bloqueado, de Ejecutando a Listo, de Bloqueado a Listo o cuando termina.*

Algunos momentos importantes son:

1. una tarea en ejecución se bloquea;
2. una tarea en ejecución deja de utilizar la CPU;
3. una tarea bloqueada vuelve a estar lista;
4. aparece una nueva tarea lista;
5. una tarea termina;
6. una política apropiativa decide desalojar a la tarea actual.

Ejemplos:

```text
Ejecución → Bloqueado
Ejecución → Listo
Bloqueado → Listo
Ejecución → Terminado
```

No todos esos eventos obligan necesariamente a cambiar de tarea, pero pueden provocar
una decisión del planificador.

---

## 5. Ráfagas de CPU y de E/S

Los procesos no suelen utilizar la CPU continuamente desde que empiezan hasta que
terminan.

Es habitual alternar:

```text
CPU → espera E/S → CPU → espera E/S → CPU
```

Podemos pensar en:

- **ráfaga de CPU**: intervalo durante el cual la tarea realiza cálculo;
- **ráfaga de E/S**: intervalo durante el cual espera una operación de entrada/salida.

Ejemplo:

```text
Editor de texto
CPU corta → espera teclado → CPU corta → espera teclado
```

Otro:

```text
Cálculo científico
CPU larga → CPU larga → CPU larga
```

Esto permite distinguir de forma general:

- tareas **CPU-bound**;
- tareas **I/O-bound**.

Un planificador moderno debe convivir con ambos tipos.

---

## 6. Planificación apropiativa y no apropiativa

### No apropiativa — non-preemptive

Una vez asignada la CPU, la tarea continúa hasta que:

- termina;
- se bloquea;
- cede voluntariamente.

El sistema no la desaloja simplemente porque otra tarea quiere ejecutar.

### Apropiativa — preemptive

El sistema operativo puede interrumpir una tarea que todavía podría continuar.

Ejemplos de motivos:

- vencimiento de un intervalo de ejecución;
- aparición de una tarea con requisitos diferentes;
- decisiones de prioridad;
- balance entre CPU.

Los sistemas operativos de propósito general actuales utilizan planificación
fundamentalmente apropiativa.

---

## 7. ¿Cómo recupera el control el sistema operativo?

El hardware dispone de mecanismos que permiten que el kernel recupere el control.

Entre ellos:

- temporizadores;
- interrupciones;
- excepciones;
- llamadas al sistema;
- eventos asociados al planificador.

Un temporizador puede generar eventos que permitan reconsiderar la ejecución actual.

No debemos imaginar necesariamente:

```text
"cada X milisegundos siempre ocurre exactamente el mismo tick"
```

Los kernels modernos utilizan mecanismos de temporización más sofisticados.

La idea esencial es:

> el sistema operativo dispone de mecanismos de hardware para recuperar el control y
> aplicar una política apropiativa.

---

## 8. Planificación de corto, medio y largo plazo

Los libros clásicos suelen distinguir tres niveles.

### Corto plazo

Decide qué tarea lista utiliza una CPU.

Es el tema central de esta clase.

### Medio plazo

En el modelo clásico puede suspender temporalmente procesos o reducir el conjunto de
procesos residentes cuando existe presión de memoria.

En sistemas modernos muchas decisiones se realizan a nivel de páginas, memoria virtual,
cgroups y otros mecanismos, por lo que no siempre existe un "planificador de medio
plazo" visible como componente separado.

### Largo plazo

El modelo clásico decide qué trabajos son admitidos al sistema.

Tiene especial sentido en:

- sistemas batch;
- colas de trabajos;
- clusters;
- gestores de recursos;
- plataformas de cómputo.

En una PC moderna de propósito general este concepto suele aparecer de forma menos
explícita.

---

## 9. Criterios para evaluar un planificador

No existe un único criterio.

### Utilización de CPU

Porcentaje de tiempo durante el cual la CPU está ocupada.

Una utilización alta puede ser deseable en un servidor de cálculo.

Pero:

> una CPU permanentemente al 100 % también puede indicar saturación y largas colas de
> espera.

---

### Throughput — productividad

Cantidad de trabajos completados por unidad de tiempo.

Ejemplo:

```text
200 trabajos / minuto
```

---

### Turnaround time — tiempo de retorno

Tiempo total desde la llegada o presentación de un trabajo hasta su finalización.

Incluye:

- espera;
- ejecución;
- bloqueos;
- E/S.

En un ejemplo simple:

```text
turnaround = tiempo_fin - tiempo_llegada
```

---

### Waiting time — tiempo de espera

Tiempo total que una tarea permanece **lista esperando CPU**.

No incluye:

- ejecución;
- tiempo bloqueado esperando E/S.

---

### Response time — tiempo de respuesta

Tiempo desde que una tarea o solicitud llega hasta que comienza a recibir atención
observable.

En ejercicios clásicos:

```text
response = primer_instante_de_CPU - llegada
```

En sistemas interactivos interesa que la respuesta inicial sea rápida, aunque la tarea
completa tarde más.

---

## 10. Los objetivos pueden entrar en conflicto

Podríamos querer simultáneamente:

- baja espera;
- baja latencia;
- alto throughput;
- justicia;
- poco overhead;
- cumplimiento de prioridades.

Pero mejorar una propiedad puede empeorar otra.

Por ejemplo:

```text
quantum pequeño
    ↓
mejor respuesta
    ↓
más cambios de contexto
```

Planificar consiste en elegir **compromisos**.

---

## 11. Sistemas diferentes, objetivos diferentes

### Batch

Interesan especialmente:

- throughput;
- turnaround;
- utilización.

### Interactivo

Interesan:

- respuesta;
- latencia;
- justicia.

### Tiempo real

Importa cumplir restricciones temporales.

No alcanza con obtener un buen promedio.

Un resultado que llega fuera de plazo puede ser incorrecto para el sistema.

---

## 12. Ejemplo común para los algoritmos

Trabajaremos inicialmente con:

| Proceso | Llegada | Ráfaga de CPU |
| ------- | ------: | ------------: |
| P1      |       0 |         24 ms |
| P2      |       0 |          3 ms |
| P3      |       0 |          3 ms |

Todos llegan en:

```text
t = 0
```

Esto simplifica la comparación.

---

## 13. FCFS — First Come, First Served

La regla es:

> ejecutar en orden de llegada.

Si llegan:

```text
P1, P2, P3
```

el diagrama de Gantt es:

```text
0                   24  27  30
|--------- P1 ---------|P2|P3|
```

Tiempos de espera:

```text
P1 = 0
P2 = 24
P3 = 27
```

Promedio:

```text
(0 + 24 + 27) / 3 = 17 ms
```

---

## 14. Efecto convoy

Si un proceso largo está adelante:

```text
P1 largo
   ↓
P2 corto
P3 corto
P4 corto
```

los procesos cortos esperan detrás de él.

Es el **efecto convoy**.

FCFS es:

- sencillo;
- fácil de comprender;
- sensible al orden de llegada;
- poco adecuado para interacción cuando aparecen ráfagas largas.

---

## 15. SJF — Shortest Job First

La regla:

> ejecutar primero la ráfaga de CPU más corta.

Con nuestro ejemplo:

```text
0   3   6                       30
|P2|P3|----------- P1 -----------|
```

Esperas:

```text
P2 = 0
P3 = 3
P1 = 6
```

Promedio:

```text
3 ms
```

---

## 16. ¿Por qué SJF se llama óptimo?

SJF minimiza el tiempo medio de espera bajo las condiciones del modelo clásico cuando:

- conocemos la duración de las ráfagas;
- comparamos el mismo conjunto de trabajos;
- utilizamos la variante no apropiativa correspondiente.

La frase:

```text
"SJF siempre es el mejor algoritmo"
```

sería incorrecta.

El problema principal es evidente:

> normalmente no conocemos exactamente el futuro.

---

## 17. Predicción de ráfagas

Un sistema puede estimar el comportamiento futuro basándose en el pasado.

Una forma clásica utiliza un promedio exponencial:

```text
estimación_nueva =
α × ráfaga_anterior
+
(1 - α) × estimación_anterior
```

No necesitamos memorizar la fórmula.

Nos interesa la idea:

> el sistema puede utilizar historia reciente para estimar qué tareas probablemente
> tendrán ráfagas cortas o largas.

---

## 18. Inanición — starvation

Si siempre llegan trabajos cortos:

```text
corto
corto
corto
corto
...
```

un trabajo largo podría ser postergado indefinidamente.

Eso es **inanición**.

No significa que el proceso esté bloqueado.

Está listo, pero la política nunca lo selecciona.

---

## 19. SRTF — Shortest Remaining Time First

Es la variante apropiativa de SJF.

Si aparece una tarea cuyo tiempo restante estimado es menor que el de la tarea actual:

```text
actual
  │
  └── puede ser desalojada
```

La nueva tarea pasa a ejecutar.

---

## 20. Round Robin

Round Robin está asociado históricamente a sistemas interactivos y de tiempo compartido.

Cada tarea recibe un intervalo llamado:

```text
quantum
```

Si termina antes:

```text
libera la CPU
```

Si no termina:

```text
vuelve al final de la cola
```

---

## 21. Ejemplo Round Robin

Quantum:

```text
4 ms
```

Procesos:

```text
P1 = 24
P2 = 3
P3 = 3
```

Ejecución:

```text
0    4   7   10   14   18   22   26   30
| P1 |P2|P3| P1 | P1 | P1 | P1 | P1 |
```

P2 y P3 terminan antes de consumir cuatro milisegundos.

Tiempo de espera total:

```text
P1 = 6
P2 = 4
P3 = 7
```

Promedio:

```text
17 / 3 ≈ 5,67 ms
```

---

## 22. El quantum

### Quantum demasiado pequeño

Ventaja:

- respuesta frecuente.

Costo:

- más decisiones;
- más cambios de contexto;
- más overhead.

### Quantum demasiado grande

Round Robin se aproxima progresivamente a:

```text
FCFS
```

No existe un tamaño universal de quantum correcto.

Los planificadores modernos tampoco deben imaginarse simplemente como un Round Robin
con un quantum fijo global.

---

## 23. Planificación por prioridades

Cada tarea recibe o calcula alguna forma de prioridad.

La política decide favorecer unas sobre otras.

Las prioridades pueden depender de:

- importancia;
- clase de planificación;
- comportamiento reciente;
- requisitos de latencia;
- configuración del administrador;
- política del sistema.

La convención numérica depende de cada sistema.

No debe memorizarse:

```text
número menor = siempre más prioridad
```

sin especificar qué campo estamos observando.

---

## 24. Prioridad e inanición

Si siempre existen tareas de prioridad superior:

```text
alta
alta
alta
alta
```

una de baja prioridad podría no ejecutar.

Una solución clásica es:

### Aging — envejecimiento

Aumentar progresivamente la posibilidad de ejecución de una tarea que lleva demasiado
tiempo esperando.

La idea:

> **cuanto más espera, menos probable debe ser que siga siendo ignorada.**

---

## 25. Multinúcleo y afinidad

En una máquina con varias CPU lógicas aparece otro problema:

> ¿en cuál CPU conviene ejecutar una tarea?

Mover constantemente una tarea entre CPU puede afectar:

- cachés;
- TLB;
- localidad.

Por eso existen mecanismos de **afinidad de CPU**.

La afinidad intenta limitar o favorecer determinadas CPU para una tarea.

---

## 26. Los algoritmos clásicos no son el scheduler real completo

![Tabla comparativa de los cuatro algoritmos clásicos. FCFS: no apropiativo, optimiza la simplicidad, riesgo principal efecto convoy y espera alta. SJF: no apropiativo (SRTF sí), optimiza el tiempo de espera, riesgo de inanición de procesos largos. Round Robin: apropiativo, optimiza el tiempo de respuesta, riesgo de overhead si el quantum es chico. Prioridades: apropiativo o no apropiativo, favorece los procesos importantes, riesgo de inanición de baja prioridad. En la práctica los SO reales no usan ninguno en forma pura: combinan colas con prioridades, aging y ajuste dinámico del quantum](img/clase07/comparacion-algoritmos.png)

*Cada algoritmo clásico optimiza un criterio distinto y arrastra un riesgo propio; los sistemas operativos reales combinan varias de estas ideas.*

FCFS, SJF, Round Robin y prioridades son modelos fundamentales.

Permiten comprender:

- colas;
- justicia;
- latencia;
- apropiación;
- inanición;
- overhead.

Pero Linux, Windows y otros sistemas actuales utilizan planificadores mucho más
sofisticados.

No debemos buscar dentro del kernel:

```text
if algoritmo == "Round Robin"
```

para explicar todo el comportamiento normal de una PC.

---

## 27. Linux: CFS y EEVDF

Durante muchos años la planificación normal de Linux estuvo asociada al
**Completely Fair Scheduler (CFS)**.

Una idea importante de CFS fue utilizar un concepto de tiempo virtual para aproximar un
reparto justo de CPU.

En kernels Linux modernos, la selección de tareas de la clase de planificación justa
ha evolucionado hacia **EEVDF**:

```text
Earliest Eligible Virtual Deadline First
```

EEVDF conserva la preocupación por la justicia, pero también incorpora elegibilidad,
*lag* y fechas límite virtuales para seleccionar tareas.

> **Dato de contexto.** La versión 6.6 del kernel Linux (noviembre de 2023) marcó la
> transición definitiva de CFS a EEVDF como planificador principal de la clase de
> tareas justas. Es información útil para quien quiera profundizar.

Para esta materia no hace falta estudiar su implementación.

Lo importante es comprender:

> los algoritmos clásicos explican los problemas fundamentales; un kernel real combina
> técnicas mucho más complejas para resolverlos.

---

## 28. `nice` en Linux

Para tareas normales Linux permite influir en su peso mediante el valor **nice**.

Rango habitual:

```text
-20 ... 19
```

Interpretación general:

```text
nice menor  → mayor peso / favorecida
nice mayor  → menor peso / más "amable" con las demás
```

Ejemplo:

```bash
nice -n 10 comando
```

Esto **no significa asignar directamente una prioridad absoluta de CPU**.

Modifica un parámetro que influye en la planificación de tareas normales.

Un usuario común normalmente puede aumentar el valor nice de sus tareas, pero reducirlo
hacia valores negativos requiere privilegios.

---

## 29. Ver prioridades y CPU

Podemos observar:

```bash
ps -eo pid,ppid,ni,pri,psr,stat,%cpu,comm --sort=-%cpu | head
```

Campos interesantes:

```text
PID  → proceso
NI   → nice
PRI  → prioridad mostrada por ps
PSR  → última CPU lógica asociada
STAT → estado
%CPU → uso de CPU observado
```

`PRI` y `NI` no son la misma cosa.

---

## 30. Cambiar nice

Creá un proceso:

```bash
sleep 300 &
pid=$!
```

Ver:

```bash
ps -p "$pid" -o pid,ni,pri,stat,comm
```

Cambiar a un nice mayor:

```bash
renice 10 -p "$pid"
```

Volver a observar:

```bash
ps -p "$pid" -o pid,ni,pri,stat,comm
```

Finalizar:

```bash
kill "$pid"
```

Con `sleep` podemos observar el valor, aunque no sirve para comparar reparto de CPU
porque casi todo el tiempo está esperando.

---

## 31. Laboratorio: dos tareas CPU-bound

Vamos a crear dos procesos que consumen CPU.

Primero verificá:

```bash
nproc
```

Si `taskset` está disponible:

```bash
command -v taskset
```

podemos fijar ambas tareas a una misma CPU lógica para que compitan realmente.

Proceso 1:

```bash
taskset -c 0 python3 -c 'while True: pass' &
p1=$!
```

Proceso 2 con nice mayor:

```bash
taskset -c 0 nice -n 10 python3 -c 'while True: pass' &
p2=$!
```

Observar:

```bash
ps -p "$p1,$p2" -o pid,ni,pri,psr,stat,%cpu,comm
```

También:

```bash
top -p "$p1,$p2"
```

Salir de `top`:

```text
q
```

Después de unos segundos, terminá ambos:

```bash
kill "$p1" "$p2"
wait "$p1" "$p2" 2>/dev/null
```

> No dejes estos procesos ejecutándose después de la práctica.

---

## 32. ¿Qué esperamos observar?

Ambos procesos quieren CPU permanentemente.

Al fijarlos a una misma CPU:

```text
P1 nice 0
P2 nice 10
```

el planificador puede otorgar diferente proporción de CPU a cada uno.

No esperes necesariamente números exactos.

El resultado depende de:

- kernel;
- entorno;
- WSL2 o Linux nativo;
- carga del sistema;
- tiempo de medición;
- otras tareas.

La práctica busca observar una **tendencia**, no demostrar una fórmula exacta.

---

## 33. Afinidad con `taskset`

Consultar afinidad:

```bash
taskset -pc "$p1"
```

Podemos permitir varias CPU:

```bash
taskset -pc 0,1 "$p1"
```

si el sistema posee al menos dos CPU lógicas.

Esto conecta directamente:

```text
multinúcleo
    +
planificación
    +
localidad de caché
```

---

## 34. Información de planificación en `/proc`

Linux expone detalles mediante:

```bash
cat /proc/$p1/sched
```

La información cambia entre versiones del kernel.

No hace falta memorizarla.

Buscá simplemente que el kernel mantiene muchos más datos de planificación de los que
aparecen en el modelo clásico.

---

## 35. WSL2 y planificación

En WSL2 Ubuntu se ejecuta dentro de un entorno virtualizado.

Linux planifica sus tareas sobre las CPU virtuales que tiene disponibles, mientras
Windows administra el sistema anfitrión.

Por eso los resultados del laboratorio pueden diferir de los de Linux nativo.

Esto es especialmente interesante:

```text
programa
  ↓
scheduler Linux
  ↓
CPU virtual disponible para WSL2
  ↓
Windows / virtualización
  ↓
CPU física
```

No necesitamos analizar todavía todas esas capas.

Alcanza con reconocer que existen.

---

## 36. Actividad práctica

Realizá la práctica anterior y respondé:

1. ¿Cuántas CPU lógicas ve Linux?
2. ¿Qué representa la columna `NI`?
3. ¿Qué representa `PSR`?
4. ¿Qué ocurre al aumentar el nice de una tarea?
5. ¿Por qué usamos tareas CPU-bound y no `sleep` para comparar reparto de CPU?
6. ¿Por qué fijamos ambas tareas a una sola CPU?
7. ¿Los porcentajes de CPU fueron exactamente constantes?
8. ¿Qué relación existe entre afinidad y memoria caché?
9. ¿Por qué esta práctica no demuestra que Linux utilice Round Robin?
10. Si utilizás WSL2, ¿qué capa adicional interviene entre Linux y el hardware?

---

## 37. Mini desafío Bash

Creá:

```text
planificacion.sh
```

Contenido:

```bash
#!/bin/bash

echo "=== PLANIFICACION ==="
echo

echo "CPU logicas:"
nproc

echo
echo "Procesos con mayor uso de CPU:"
ps -eo pid,ppid,ni,pri,psr,stat,%cpu,comm \
    --sort=-%cpu | head -n 12

echo
echo "Carga del sistema:"
uptime
```

Permisos:

```bash
chmod +x planificacion.sh
```

Ejecutalo:

```bash
./planificacion.sh
```

---

## 38. Ejercicio de lápiz y papel

Procesos:

| Proceso | Llegada | Ráfaga |
| ------- | ------: | -----: |
| P1      |       0 |      8 |
| P2      |       1 |      4 |
| P3      |       2 |      2 |

Resolver:

1. FCFS.
2. SJF no apropiativo.
3. SRTF.
4. Round Robin con quantum 2.

Para cada algoritmo calcular:

- instante de finalización;
- turnaround;
- waiting time;
- response time.

Después responder:

> ¿qué algoritmo elegirías para un sistema interactivo y por qué?

---

## 39. Ideas para recordar

```text
Tareas listas
     │
     ▼
Scheduler
     │
     ▼
Dispatcher
     │
     ▼
CPU
```

Y:

```text
No existe
"el mejor algoritmo"
para todos los sistemas.
```

Los criterios compiten:

```text
respuesta
throughput
espera
justicia
overhead
prioridades
```

---

## Glosario

**Scheduler:** componente o lógica que selecciona qué tarea debe ejecutar.

**Dispatcher:** mecanismo que entrega efectivamente la CPU a la tarea elegida.

**Ráfaga de CPU:** período durante el cual una tarea utiliza CPU.

**CPU-bound:** tarea dominada por cálculo.

**I/O-bound:** tarea que alterna frecuentemente con esperas de E/S.

**Preemptive / apropiativo:** el sistema puede desalojar una tarea.

**Non-preemptive / no apropiativo:** la tarea conserva CPU hasta bloquearse, terminar o
ceder.

**Throughput:** trabajos completados por unidad de tiempo.

**Turnaround:** tiempo entre llegada y finalización.

**Waiting time:** tiempo acumulado esperando CPU en estado listo.

**Response time:** tiempo hasta la primera atención.

**FCFS:** planificación en orden de llegada.

**SJF:** selecciona la ráfaga estimada más corta.

**SRTF:** variante apropiativa basada en tiempo restante.

**Round Robin:** reparto por turnos mediante quantum.

**Quantum:** intervalo de ejecución usado en el modelo Round Robin.

**Starvation:** postergación indefinida de una tarea lista.

**Aging:** mecanismo que reduce el riesgo de inanición por espera prolongada.

**Afinidad de CPU:** restricción o preferencia sobre qué CPU puede ejecutar una tarea.

**Nice:** parámetro Unix que modifica el peso relativo de tareas normales.

**CFS:** planificador justo histórico de Linux.

**EEVDF:** política moderna de selección de tareas justas utilizada por Linux.

---

## Desafiate con preguntas de examen

1. ¿Por qué el sistema operativo necesita un planificador?
2. ¿Qué diferencia existe entre scheduler y dispatcher?
3. ¿Qué diferencia existe entre proceso e hilo como unidad de planificación?
4. ¿Qué es una ráfaga de CPU?
5. ¿Qué diferencia existe entre una tarea CPU-bound y una I/O-bound?
6. ¿Qué diferencia hay entre planificación apropiativa y no apropiativa?
7. ¿Qué eventos pueden provocar una decisión de planificación?
8. ¿Qué significan utilización, throughput, turnaround, waiting y response time?
9. ¿Por qué los objetivos de planificación pueden entrar en conflicto?
10. ¿Qué es el efecto convoy?
11. ¿Bajo qué condiciones SJF minimiza el tiempo medio de espera?
12. ¿Por qué SJF resulta difícil de implementar literalmente?
13. ¿Qué es inanición?
14. ¿Qué diferencia hay entre SJF y SRTF?
15. ¿Cómo funciona Round Robin?
16. ¿Qué ocurre si el quantum es demasiado pequeño?
17. ¿Qué ocurre si es demasiado grande?
18. ¿Cómo puede aparecer inanición con prioridades?
19. ¿Qué es aging?
20. ¿Por qué la afinidad puede mejorar la localidad?
21. ¿Qué significa el valor nice en Linux?
22. ¿Por qué `nice` no debe interpretarse como una prioridad absoluta?
23. ¿Por qué los algoritmos clásicos no describen completamente un scheduler moderno?
24. ¿Qué relación histórica existe entre CFS y EEVDF en Linux?
25. ¿Por qué WSL2 puede producir resultados diferentes de Linux nativo?

---

## Próxima etapa

Después de comprender la planificación, el siguiente paso natural es estudiar qué ocurre
cuando un mismo proceso posee **varios flujos de ejecución**.

Ahí aparecen:

- hilos;
- concurrencia;
- regiones críticas;
- condiciones de carrera;
- sincronización;
- interbloqueos.

Esos conceptos completan el bloque de procesos antes de entrar en administración de
memoria.
