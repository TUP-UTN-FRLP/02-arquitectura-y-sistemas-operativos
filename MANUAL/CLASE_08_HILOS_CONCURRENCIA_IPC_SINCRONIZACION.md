**Arquitectura y Sistemas Operativos**

# Clase 8 — Hilos, concurrencia, comunicación y sincronización

**Tecnicatura Universitaria en Programación**

---

## Objetivos de la clase

Al finalizar esta clase deberías poder:

- explicar qué es un hilo y cómo se relaciona con un proceso;
- distinguir qué recursos comparten los hilos y qué estado mantiene cada uno;
- diferenciar concurrencia y paralelismo;
- reconocer condiciones de carrera y regiones críticas;
- comprender el principio de exclusión mutua;
- diferenciar mutex y semáforos;
- comprender la idea de monitor y variable de condición;
- reconocer mecanismos básicos de comunicación entre procesos (IPC);
- utilizar pipes de Linux como ejemplo real de IPC;
- explicar qué es un interbloqueo (*deadlock*);
- reconocer las cuatro condiciones de Coffman;
- observar hilos e IPC desde Linux y Bash.

---

## 1. De los procesos a los hilos

En las clases anteriores estudiamos:

- procesos;
- estados;
- PID y PPID;
- cambio de contexto;
- planificación de CPU.

Hasta ahora utilizamos muchas veces al proceso como si fuera una única unidad de
ejecución.

Pero un mismo proceso puede necesitar realizar varias actividades:

```text
Navegador
├── atender interfaz
├── recibir datos
├── procesar contenido
└── realizar tareas en segundo plano
```

Una aplicación de chat puede:

```text
recibir mensajes
+
enviar mensajes
+
mantener la interfaz activa
```

Un servidor puede atender varias solicitudes concurrentemente.

Para representar varios flujos de ejecución dentro de un mismo proceso aparecen los
**hilos**.

---

## 2. ¿Qué es un hilo?

Un **hilo** (*thread*) es un flujo de ejecución dentro de un proceso.

Un proceso puede tener:

```text
1 hilo
```

o:

```text
varios hilos
```

Los hilos de un mismo proceso comparten gran parte de sus recursos, pero cada hilo
necesita mantener su propio estado de ejecución.

Modelo conceptual:

```text
PROCESO
│
├── código
├── datos
├── heap
├── archivos abiertos
│
├── HILO 1
│   ├── registros
│   ├── contador de programa
│   └── stack
│
├── HILO 2
│   ├── registros
│   ├── contador de programa
│   └── stack
│
└── HILO 3
    ├── registros
    ├── contador de programa
    └── stack
```

---

## 3. ¿Qué tiene cada hilo?

En una primera aproximación, cada hilo posee:

- identificador;
- contador de programa;
- registros de CPU;
- stack propio;
- estado de ejecución;
- información necesaria para ser planificado.

Los hilos del mismo proceso comparten normalmente:

- espacio de direcciones;
- código;
- datos globales;
- heap;
- muchos recursos del proceso;
- descriptores de archivos abiertos.

---

## 4. Una precisión sobre señales y credenciales

En un modelo introductorio podemos decir que los hilos pertenecen al mismo proceso y
comparten su entorno.

Sin embargo, en sistemas reales algunos detalles no son simplemente
“todo compartido” o “todo privado”.

Por ejemplo, en POSIX/Linux:

- la **disposición** de una señal es propiedad del proceso;
- cada hilo puede poseer su propia máscara de señales;
- determinadas señales pueden dirigirse a un hilo concreto;
- el kernel mantiene información por tarea además de información compartida por el
  grupo de hilos.

No necesitamos dominar ahora esas diferencias.

La idea central es:

> los hilos comparten el espacio de memoria del proceso, pero cada uno conserva el
> contexto necesario para ejecutar independientemente.

---

## 5. ¿Por qué los hilos suelen ser más livianos?

Crear otro proceso implica crear una nueva entidad con su propio espacio de direcciones y
estructuras asociadas.

Crear un hilo dentro de un proceso existente permite reutilizar muchos recursos ya
presentes.

Por eso, en general:

```text
crear hilo
<
crear proceso
```

en costo de creación y memoria.

También un cambio entre dos hilos del mismo proceso **puede** resultar más económico que
un cambio entre procesos con espacios de direcciones distintos.

No significa que crear o cambiar hilos sea gratis.

Cada hilo todavía necesita, entre otras cosas:

- stack;
- registros;
- estructuras del kernel;
- datos locales del hilo;
- información para el planificador.

---

## 6. Proceso vs hilo

| Característica         | Proceso                   | Hilo                                  |
| ---------------------- | ------------------------- | ------------------------------------- |
| Espacio de direcciones | propio                    | compartido con hilos del proceso      |
| Stack                  | uno o más según sus hilos | propio                                |
| Registros              | asociados a cada hilo     | propios                               |
| Archivos abiertos      | recursos del proceso      | compartidos normalmente               |
| Aislamiento            | fuerte entre procesos     | bajo dentro del mismo proceso         |
| Comunicación           | requiere IPC              | memoria compartida naturalmente       |
| Fallo grave            | puede quedar aislado      | puede comprometer al proceso completo |

---

## 7. Ventajas del multihilo

### Capacidad de respuesta

Una actividad puede continuar mientras otra espera.

Ejemplo:

```text
hilo interfaz
      +
hilo de red
```

La interfaz no necesita quedar detenida mientras llegan datos.

---

### Compartición de recursos

Los hilos pueden trabajar sobre estructuras comunes en memoria.

Esto hace eficiente el intercambio de información.

Pero también crea el principal problema de la clase:

> **varios flujos pueden acceder al mismo dato.**

---

### Economía

En general, los hilos requieren menos recursos que procesos independientes que
realizaran el mismo trabajo.

---

### Aprovechamiento de varios núcleos

Si el lenguaje, el runtime y el sistema lo permiten, distintos hilos pueden ejecutarse
simultáneamente sobre diferentes CPU lógicas.

```text
CPU 0 → hilo A
CPU 1 → hilo B
CPU 2 → hilo C
```

---

## 8. Los hilos no siempre dan paralelismo real de CPU

El comportamiento exacto de los hilos depende del lenguaje, del entorno de ejecución
(*runtime*) y del sistema operativo. No todos los entornos permiten paralelismo real de
CPU con hilos aunque el hardware lo soporte: algunos limitan la ejecución simultánea de
código a un hilo por vez dentro de un mismo programa.

En esos casos los hilos siguen siendo muy útiles para:

- operaciones de entrada/salida;
- esperar la red;
- esperar archivos;
- mantener varias actividades concurrentes.

Pero un cálculo intenso de CPU no se acelera automáticamente por crear más hilos si el
entorno no ejecuta varios en paralelo.

> Nada de esto elimina los problemas de concurrencia ni vuelve innecesarios los
> mecanismos de sincronización.

---

## 9. Riesgos del multihilo

### Condiciones de carrera

El resultado depende del orden temporal de operaciones concurrentes.

### Interbloqueos

Varios hilos pueden quedar esperando recursos que nunca serán liberados.

### Complejidad

Los errores pueden depender de una intercalación difícil de reproducir.

### Menor aislamiento

Todos los hilos trabajan dentro del mismo proceso.

Un error grave que compromete la memoria del proceso puede afectar a todos sus hilos.

---

## 10. Concurrencia y paralelismo

Son conceptos relacionados, pero distintos.

### Concurrencia

Existen varias actividades en progreso durante un mismo período.

No es necesario que estén ejecutándose físicamente al mismo tiempo.

Ejemplo con una sola CPU:

```text
tiempo ─────────────────────────────>

CPU:  AAA BBB AA BB AAA BBB
```

A y B progresan de forma intercalada.

---

### Paralelismo

Dos o más actividades se ejecutan físicamente al mismo tiempo.

Ejemplo:

```text
CPU 0: AAAAAAAAAAAAAAAA
CPU 1: BBBBBBBBBBBBBBBB
```

---

## 11. Una forma útil de pensarlo

Podemos usar esta distinción:

> **concurrencia** describe la organización y el progreso de varias actividades.

> **paralelismo** describe ejecución simultánea real.

Un programa concurrente puede ejecutarse sobre:

```text
1 CPU
```

mediante intercalación.

O sobre:

```text
varias CPU
```

y además obtener paralelismo.

---

## 12. Los problemas de concurrencia existen también con una sola CPU

Este punto es fundamental.

Supongamos:

```text
Hilo A
Hilo B
```

Aunque exista una sola CPU, el sistema puede ejecutar:

```text
A
A
B
A
B
B
```

El cambio puede ocurrir entre operaciones que nuestro código fuente aparenta presentar
como una sola acción.

Por eso:

> una condición de carrera no necesita varios núcleos para existir.

---

## 13. Condiciones de carrera

Una **condición de carrera** ocurre cuando el resultado correcto depende de una
intercalación particular entre actividades concurrentes y esas actividades no están
coordinadas adecuadamente.

Ejemplo conceptual:

```text
contador = contador + 1
```

Parece una única operación.

Pero conceptualmente puede requerir:

```text
leer contador
sumar 1
escribir contador
```

---

## 14. El incremento perdido

Estado inicial:

```text
contador = 0
```

Hilo A:

```text
lee 0
```

Hilo B:

```text
lee 0
```

Hilo A:

```text
calcula 1
escribe 1
```

Hilo B:

```text
calcula 1
escribe 1
```

Resultado:

```text
contador = 1
```

aunque esperábamos:

```text
contador = 2
```

---

## 15. Race condition y data race

En cursos introductorios suelen utilizarse ambos términos casi como sinónimos.

Conviene conocer una pequeña distinción.

### Race condition

El resultado depende incorrectamente del orden relativo de eventos concurrentes.

### Data race

En modelos de memoria que utilizan este término formalmente, ocurre cuando dos
ejecuciones acceden concurrentemente a una misma ubicación, al menos una escribe y no
existe la sincronización requerida.

Una *data race* es una forma especialmente importante de condición de carrera.

Para esta materia nos concentraremos principalmente en la idea general de
**condición de carrera**.

---

## 16. Región crítica

Una **región crítica** o **sección crítica** es una zona de código que accede a un recurso
compartido cuya consistencia requiere coordinación.

Ejemplo:

```text
leer saldo
calcular nuevo saldo
guardar saldo
```

Si dos hilos realizan esa operación simultáneamente sin coordinación:

```text
puede perderse una actualización
```

No significa que solamente pueda existir una región crítica en todo el programa.

La exclusión debe proteger a las regiones que compiten por **el mismo recurso o
invariante compartido**.

---

## 17. Requisitos clásicos de una solución

Una solución al problema de la sección crítica busca propiedades como:

### Exclusión mutua

Dos participantes que compiten por la misma sección crítica no deben ejecutarla
simultáneamente cuando eso violaría la consistencia.

### Progreso

Si la sección está libre y existen participantes que quieren entrar, el sistema debe
permitir que alguno avance.

### Espera acotada

Una tarea no debería quedar postergada indefinidamente mientras otras entran una y otra
vez.

### Independencia de velocidades relativas

La corrección no debería depender de que una CPU o un hilo sea “suficientemente rápido”.

Estas propiedades aparecen en el estudio clásico del problema de exclusión mutua.

---

## 18. Sincronización

La **sincronización** permite coordinar actividades concurrentes.

Puede utilizarse para:

- exclusión mutua;
- esperar un evento;
- limitar la cantidad de usuarios de un recurso;
- establecer orden entre operaciones;
- comunicar cambios de estado.

Entre los mecanismos más conocidos encontramos:

```text
mutex
semáforos
variables de condición
monitores
operaciones atómicas
```

---

## 19. Mutex

**Mutex** proviene de:

```text
mutual exclusion
```

Puede imaginarse como una llave asociada a un recurso.

```text
lock
  │
  ▼
región crítica
  │
  ▼
unlock
```

Pseudocódigo:

```text
mutex.lock()

contador = contador + 1

mutex.unlock()
```

Si otro hilo intenta tomar el mutex mientras está ocupado, deberá esperar según la
implementación y la política utilizada.

---

## 20. Cómo se implementa un mutex

La implementación real puede combinar:

- instrucciones atómicas del procesador;
- espera activa durante intervalos muy breves;
- intervención del kernel;
- colas de tareas bloqueadas.

Ejemplos de primitivas de hardware:

```text
compare-and-swap
exchange
test-and-set
```

En Linux existen mecanismos como **futex** que permiten que el caso sin contención pueda
resolverse principalmente en espacio de usuario y recurrir al kernel cuando es necesario
esperar.

No necesitamos implementar un mutex.

Nos interesa comprender:

> la exclusión mutua necesita finalmente alguna operación atómica sobre la cual construir
> mecanismos de mayor nivel.

---

## 21. Semáforos

Un **semáforo** mantiene un contador utilizado para coordinar acceso o eventos.

Las operaciones se conocen tradicionalmente como:

```text
wait / P / down / acquire
signal / V / up / release
```

Modelo conceptual:

```text
wait:
    si existe un recurso disponible:
        tomar uno
    si no:
        esperar
```

```text
signal:
    devolver disponibilidad
    permitir que avance algún participante
```

---

## 22. Semáforo contador

Supongamos tres recursos equivalentes:

```text
semáforo = 3
```

Los primeros tres hilos pueden adquirir uno.

```text
Hilo A → entra
Hilo B → entra
Hilo C → entra
Hilo D → espera
```

Cuando alguno libera:

```text
Hilo D → puede avanzar
```

---

## 23. Semáforo binario y mutex no son exactamente lo mismo

Un semáforo binario puede tomar conceptualmente los valores:

```text
0
1
```

y puede utilizarse para implementar exclusión mutua.

Pero no debemos decir que es **idéntico** a un mutex.

La diferencia conceptual clave es la de **propietario**:

- un mutex tiene dueño: solo el hilo que lo adquirió puede liberarlo;
- un semáforo binario no tiene dueño: cualquier hilo puede hacer la operación que lo
  incrementa, no solo el que lo decrementó.

Esa noción de propiedad tiene consecuencias prácticas. Como el mutex sabe quién lo
retiene, puede aplicar **protocolos de herencia de prioridad** (elevar temporalmente la
prioridad del hilo que lo tiene para que un hilo más prioritario no quede bloqueado
indefinidamente). Un semáforo binario, al no tener dueño, no puede hacer eso.

Las implementaciones pueden incorporar además otras características:

- detección de errores (liberar un mutex que no se posee es un error);
- políticas de espera.

Por eso:

> **semáforo binario y mutex pueden resolver problemas parecidos, pero son primitivas
> conceptualmente diferentes.**

---

## 24. Monitores

Un **monitor** es una abstracción de sincronización de más alto nivel.

Agrupa:

- estado compartido;
- operaciones sobre ese estado;
- exclusión mutua;
- mecanismos para esperar condiciones.

La idea es evitar que el programador deba manipular manualmente primitivas de bajo nivel
en cada operación.

---

## 25. Variables de condición

Una variable de condición permite expresar:

```text
"no puedo continuar hasta que ocurra X"
```

Un hilo puede:

1. comprobar una condición;
2. esperar;
3. liberar temporalmente el lock;
4. ser despertado cuando el estado cambia;
5. volver a comprobar la condición.

Ejemplos clásicos:

```text
buffer no vacío
buffer no lleno
dato disponible
trabajo terminado
```

---

## 26. Monitores integrados en el lenguaje

Algunos lenguajes y entornos ofrecen el monitor "incorporado": el programador marca una
operación como protegida y el entorno se encarga de la exclusión mutua, sin adquirir ni
liberar un lock a mano.

En pseudocódigo, la idea sería:

```text
operación protegida incrementar():
    contador = contador + 1
```

donde "protegida" significa que el entorno garantiza que solo un hilo por vez ejecuta
esa operación sobre el mismo objeto.

Aun así, un monitor es una idea más amplia que simplemente:

```text
"todas las operaciones tienen lock"
```

porque también incluye la coordinación mediante variables de condición (esperar a que
ocurra algo, como se vio en la sección 25).

---

## 27. Analizar la condición de carrera sobre el diagrama

Volvé al diagrama del **incremento perdido** de la sección 14. Recordá que dos hilos,
A y B, quieren hacer lo mismo sobre un contador compartido que empieza en `0`:

```text
leer contador
sumar 1
escribir contador
```

Y la intercalación fue:

```text
Hilo A: lee 0
Hilo B: lee 0
Hilo A: calcula 1, escribe 1
Hilo B: calcula 1, escribe 1

contador = 1   (esperábamos 2)
```

Preguntas de análisis:

1. ¿En qué paso exacto se pierde uno de los incrementos? ¿Qué dato "viejo" está usando
   el hilo B cuando calcula su resultado?
2. Si en lugar de dos hilos hubiera diez haciendo lo mismo mil veces cada uno, ¿por qué
   el resultado final puede ser cualquier número entre 1000 y 10 000?
3. ¿El error aparece siempre? ¿Por qué es difícil de reproducir?

---

## 28. ¿Qué cambia con exclusión mutua?

Supongamos ahora que la secuencia *leer-modificar-escribir* está protegida por un
mecanismo de exclusión mutua (un mutex o *lock*): antes de leer el contador, el hilo
debe **adquirir** el lock, y lo **libera** recién después de escribir.

```text
Hilo A: adquiere el lock
Hilo A: lee 0, calcula 1, escribe 1
Hilo A: libera el lock
Hilo B: adquiere el lock
Hilo B: lee 1, calcula 2, escribe 2
Hilo B: libera el lock

contador = 2   (correcto)
```

Preguntas de análisis:

1. ¿Por qué ahora el hilo B lee `1` y no `0`?
2. ¿Qué pasa con el hilo B durante el tiempo en que A tiene el lock?
3. La sección crítica quedó **serializada**. ¿Qué se pierde a cambio de la correctitud?

Estas son las ideas que desarrollan las secciones sobre **mutex**, **semáforos** y
**monitores**.

---

## 29. Comunicación entre procesos — IPC

Los hilos del mismo proceso ya comparten memoria.

Procesos diferentes tienen espacios de direcciones separados.

Cuando necesitan cooperar utilizan mecanismos de:

```text
IPC
Inter-Process Communication
```

Algunos mecanismos habituales:

- pipes;
- FIFOs;
- memoria compartida;
- sockets;
- colas de mensajes;
- señales;
- archivos.

---

## 30. Memoria compartida

El sistema puede mapear una misma región física dentro de varios espacios virtuales.

Conceptualmente:

```text
Proceso A ─┐
           ├── región compartida
Proceso B ─┘
```

Ventaja:

- comunicación eficiente una vez establecido el mapeo.

Problema:

- vuelve a aparecer la necesidad de sincronización.

---

## 31. Pasaje de mensajes

En lugar de compartir directamente memoria, los procesos intercambian mensajes.

Modelo:

```text
Proceso A
   │
   │ mensaje
   ▼
mecanismo de comunicación
   │
   ▼
Proceso B
```

Puede ser más sencillo aislar responsabilidades, aunque rendimiento y costos dependen
del mecanismo concreto.

No conviene afirmar universalmente que:

```text
message passing = lento
shared memory = rápido
```

sin conocer:

- tamaño de mensajes;
- implementación;
- copias;
- frecuencia;
- topología;
- sistema utilizado.

---

## 32. Pipes

Un pipe Unix tradicional es un canal de bytes con un extremo de escritura y otro de
lectura.

En Bash:

```bash
ps aux | grep bash
```

La shell conecta:

```text
stdout de ps
       │
       ▼
      pipe
       │
       ▼
stdin de grep
```

Cada comando se ejecuta como parte de una tubería de procesos.

---

## 33. Un pipe es IPC real

Cuando escribimos:

```bash
printf "ana\njuan\nana\n" | sort | uniq -c
```

tenemos varios programas:

```text
printf
sort
uniq
```

comunicándose mediante pipes.

El resultado puede ser:

```text
      2 ana
      1 juan
```

Lo importante no es `sort` ni `uniq`.

Lo importante es comprender:

> **la salida de un proceso está siendo utilizada como entrada de otro mediante un
> mecanismo provisto por el sistema operativo.**

---

## 34. Pipeline y paralelismo no son sinónimos

Los procesos de una tubería pueden coexistir y progresar concurrentemente.

Pero no debemos concluir:

```text
pipe = siempre ejecución paralela
```

El paralelismo efectivo dependerá de:

- CPU disponibles;
- estado de cada proceso;
- bloqueos;
- cantidad de datos;
- planificación.

El pipe crea **comunicación y concurrencia posible**.

---

## 35. FIFO — pipe con nombre

Linux también permite crear un pipe visible en el sistema de archivos.

En una terminal:

```bash
mkfifo canal
```

Comprobar:

```bash
ls -l canal
```

Podés observar una letra inicial:

```text
p
```

que indica un FIFO.

---

## 36. Laboratorio con dos terminales

### Terminal 1

```bash
cat canal
```

Quedará esperando.

### Terminal 2

```bash
echo "Hola desde el otro proceso" > canal
```

En la Terminal 1 aparecerá el mensaje.

Después:

```bash
rm canal
```

Acabamos de utilizar IPC entre dos procesos desde Bash.

---

## 37. Sockets

Un socket permite comunicación:

- local;
- o mediante una red.

Los sockets pueden utilizar protocolos diferentes.

Cuando estudiemos redes volveremos sobre:

```text
dirección
puerto
TCP
UDP
cliente
servidor
```

---

## 38. Señales

Las señales sirven para notificar eventos a procesos.

Ya utilizamos:

```bash
kill -TERM PID
kill -STOP PID
kill -CONT PID
```

Esas operaciones también son una forma simple de comunicación y control entre procesos.

Una señal transporta principalmente una **notificación**, no un flujo arbitrario de
datos como un pipe o un socket.

---

## 39. Observar los hilos de un proceso del sistema

No hace falta escribir ningún programa: en tu equipo ya hay procesos con varios hilos.
Podés listarlos junto con su cantidad de hilos con:

```bash
ps -eLf | head -20
```

Cada fila es una tarea/hilo. La columna **NLWP** indica cuántos hilos tiene el proceso y
**LWP** es el identificador de cada hilo.

Para ver solo procesos con más de un hilo, ordenados por cantidad:

```bash
ps -eo pid,nlwp,comm --sort=-nlwp | head -10
```

---

## 40. Elegir el proceso a observar

Tomá uno de los PID de la lista anterior que tenga varios hilos y guardalo en una
variable (reemplazá el número por el que hayas elegido):

```bash
pid=1234
```

Verificá:

```bash
echo "$pid"
```

> No vamos a terminar este proceso al final del laboratorio: es un proceso del sistema y
> lo dejamos como está. Solo lo observamos.

---

## 41. `ps -T`

Mostrar los hilos:

```bash
ps -T -p "$pid"
```

Más detalle:

```bash
ps -T -p "$pid" -o pid,spid,psr,stat,comm
```

Campos:

```text
PID   → identificador del grupo/proceso
SPID  → identificador de la tarea/hilo mostrado
PSR   → CPU asociada en la observación
STAT  → estado
COMM  → nombre
```

En herramientas Linux también puede aparecer el término:

```text
LWP
```

para referirse a una unidad de ejecución ligera asociada al hilo.

---

## 42. `/proc/PID/task`

Linux expone los hilos de un proceso en:

```bash
ls "/proc/$pid/task"
```

Contarlos:

```bash
ls "/proc/$pid/task" | wc -l
```

Cada entrada corresponde a una tarea/hilo visible para el kernel dentro de ese proceso.

Podemos inspeccionar una:

```bash
tid=$(ls "/proc/$pid/task" | head -n 1)

grep -E '^(Name|State|Pid|Tgid|Threads):' \
    "/proc/$pid/task/$tid/status"
```

---

## 43. PID, TID y TGID en Linux

En Linux, internamente los hilos son tareas planificables.

Podemos encontrar:

- **TID**: identificador de una tarea/hilo;
- **TGID**: identificador del grupo de hilos;
- el líder del grupo tiene un TID que coincide con el PID que normalmente vemos para el
  proceso.

Para una primera aproximación:

```text
proceso visible
     │
    TGID
     │
     ├── TID 1
     ├── TID 2
     └── TID 3
```

Esto muestra algo importante:

> el planificador necesita distinguir flujos de ejecución individuales.

---

## 44. `top` mostrando hilos

Podemos usar:

```bash
top -H -p "$pid"
```

`-H` solicita mostrar hilos.

Salir:

```text
q
```

Recordá: como es un proceso del sistema, no lo terminamos. Cerrá `top` con `q` y listo.

---

## 45. WSL2 y los hilos

Dentro de WSL2 estamos trabajando con un kernel Linux real dentro de un entorno
virtualizado.

Comandos como:

```bash
ps -T
/proc/PID/task
top -H
```

siguen mostrando las tareas que administra el kernel Linux.

La cantidad y disponibilidad de CPU pueden depender de la configuración y del entorno
virtualizado.

Esto vuelve a conectar:

```text
hilos
   ↓
scheduler Linux
   ↓
CPU visibles para WSL2
   ↓
virtualización
   ↓
hardware físico
```

---

## 46. Interbloqueo — deadlock

Un **deadlock** ocurre cuando un conjunto de tareas queda bloqueado porque cada una
espera un recurso o evento que depende de otra del mismo conjunto.

Ejemplo:

```text
Hilo 1 posee A y espera B
Hilo 2 posee B y espera A
```

```text
Hilo 1 ──espera──> B
  ▲                 │
  │                 ▼
  A <──posee──── Hilo 2
```

Ninguno puede continuar.

---

## 47. Las cuatro condiciones de Coffman

Para que pueda existir un deadlock asociado a recursos deben estar presentes cuatro
condiciones clásicas.

### Exclusión mutua

Algún recurso no puede ser utilizado simultáneamente por todos.

### Retener y esperar

Una tarea conserva recursos mientras solicita otros.

### Sin apropiación

Los recursos involucrados no pueden ser retirados arbitrariamente a quien los posee.

### Espera circular

Existe una cadena circular de espera.

```text
P1 espera recurso de P2
P2 espera recurso de P3
P3 espera recurso de P1
```

---

## 48. Una precisión importante

Es frecuente resumir:

```text
"si se cumplen las cuatro condiciones, hay deadlock"
```

pero es mejor formularlo así:

> **si existe un deadlock de este tipo, las cuatro condiciones están presentes.**

Por lo tanto:

> **si logramos romper al menos una de ellas, impedimos ese deadlock.**

Que un sistema permita exclusión mutua, retención y ausencia de apropiación no significa
que esté permanentemente en deadlock.

La espera circular concreta entre las tareas es parte central del problema.

---

## 49. Estrategias frente al deadlock

### Prevención

Diseñar reglas para impedir alguna condición.

Ejemplo:

```text
todos adquieren locks siempre en el mismo orden
```

Esto puede evitar espera circular.

---

### Evitación

Decidir si conceder un recurso manteniendo al sistema en un estado considerado seguro.

El ejemplo clásico es:

```text
algoritmo del banquero
```

---

### Detección y recuperación

Permitir que ocurra, detectar ciclos o situaciones de bloqueo y actuar.

Posibles acciones:

- terminar una tarea;
- reiniciar un componente;
- recuperar recursos cuando el sistema lo permita.

---

### No implementar una solución general para todos los casos

Los sistemas operativos de propósito general no intentan necesariamente prevenir y
detectar **todos los deadlocks posibles de las aplicaciones**.

Eso no significa que el kernel simplemente ignore cualquier bloqueo.

Existen según el subsistema:

- timeouts;
- watchdogs;
- mecanismos de diagnóstico;
- herramientas para detectar problemas de locks;
- políticas específicas.

La conocida **estrategia del avestruz** describe la decisión de no mantener un mecanismo
general y costoso para resolver automáticamente toda clase de deadlock.

---

## 50. No vamos a provocar un deadlock real como práctica

Es sencillo escribir un programa que quede bloqueado para siempre.

Pero como práctica introductoria aporta poco dejar terminales colgadas.

Vamos a trabajar con:

- diagramas;
- análisis del orden de adquisición;
- detección conceptual.

Ejemplo seguro:

```text
Hilo A:
    lock(A)
    lock(B)

Hilo B:
    lock(B)
    lock(A)
```

Pregunta:

> ¿qué cambio simple podríamos hacer para eliminar la posibilidad de espera circular?

Respuesta:

```text
obligar a ambos hilos a adquirir siempre:
A → B
```

---

## 51. Actividad práctica integrada

### Parte A — observar hilos

Elegí un proceso del sistema con varios hilos y observá sus tareas (no lo terminamos):

```bash
pid=$(ps -eo pid,nlwp --sort=-nlwp --no-headers | awk 'NR==1{print $1}')
echo "Observando el proceso $pid"

ps -T -p "$pid" -o pid,spid,psr,stat,comm

ls "/proc/$pid/task"

top -H -p "$pid"
```

---

### Parte B — condición de carrera (análisis)

Volvé a las secciones 27 y 28 y respondé sobre el diagrama del incremento perdido:

- ¿en qué paso exacto se pierde uno de los incrementos?
- ¿qué cambiaría si la sección crítica estuviera protegida por exclusión mutua?

---

### Parte C — pipe

```bash
printf "error\nok\nerror\nok\nerror\n" |
grep error |
wc -l
```

Respondé:

- ¿cuántos procesos participan?
- ¿qué dato fluye entre ellos?
- ¿qué mecanismo de IPC utilizan?

---

### Parte D — FIFO

Terminal 1:

```bash
mkfifo canal
cat canal
```

Terminal 2:

```bash
echo "mensaje" > canal
```

Al terminar:

```bash
rm canal
```

---

## 52. Preguntas de la práctica

1. ¿Cuántos hilos tenía el proceso que observaste?
2. ¿Todos compartían el mismo PID/TGID?
3. ¿Qué identificador cambiaba entre hilos?
4. ¿Todos aparecían necesariamente sobre la misma CPU?
5. ¿Qué diferencia conceptual existe entre proceso e hilo?
6. ¿Por qué un incremento sin proteger sobre un dato compartido puede producir un valor
   incorrecto?
7. ¿Qué cambia al proteger la sección crítica con exclusión mutua?
8. ¿Qué comunica el pipe de Bash?
9. ¿Qué diferencia existe entre un pipe anónimo y un FIFO?
10. ¿Por qué los hilos no necesitan IPC para compartir una variable?
11. ¿Por qué compartir memoria obliga a pensar en sincronización?
12. ¿Qué condición de Coffman rompe adquirir siempre los locks en el mismo orden?

---

## 53. Mini desafío Bash

Creá:

```text
inspeccionar_hilos.sh
```

con:

```bash
#!/bin/bash

if [ -z "$1" ]; then
    echo "Uso: $0 PID"
    exit 1
fi

pid="$1"

if [ ! -d "/proc/$pid/task" ]; then
    echo "No existe el PID $pid"
    exit 1
fi

echo "=== PROCESO $pid ==="
echo

ps -p "$pid" -o pid,ppid,stat,comm,args

echo
echo "=== HILOS ==="
ps -T -p "$pid" -o pid,spid,psr,stat,comm

echo
echo "Cantidad de hilos:"
ls "/proc/$pid/task" | wc -l
```

Permisos:

```bash
chmod +x inspeccionar_hilos.sh
```

Uso (sobre un proceso del sistema con varios hilos):

```bash
pid=$(ps -eo pid,nlwp --sort=-nlwp --no-headers | awk 'NR==1{print $1}')

./inspeccionar_hilos.sh "$pid"
```

---

## 54. Ideas para recordar

```text
PROCESO
    │
    ├── memoria compartida
    ├── archivos compartidos
    │
    ├── HILO 1 → registros + stack
    ├── HILO 2 → registros + stack
    └── HILO 3 → registros + stack
```

La ventaja:

```text
compartir
+
concurrencia
+
posible paralelismo
```

El riesgo:

```text
datos compartidos
+
orden impredecible
=
condiciones de carrera
```

La respuesta:

```text
sincronización
```

Y una regla importante:

```text
deadlock
≠
simplemente "programa lento"
```

Es una espera circular que impide el progreso.

---

## Glosario

**Hilo / thread:** flujo de ejecución dentro de un proceso.

**TID:** identificador de una tarea o hilo.

**TGID:** identificador del grupo de hilos utilizado por Linux.

**Concurrencia:** existencia y progreso de múltiples actividades durante un mismo
período.

**Paralelismo:** ejecución físicamente simultánea de actividades.

**Condición de carrera:** error cuyo resultado depende de una intercalación no coordinada
entre actividades.

**Data race:** acceso concurrente no sincronizado a una ubicación compartida según el
modelo de memoria correspondiente.

**Región crítica:** código que utiliza un recurso compartido cuya consistencia requiere
coordinación.

**Exclusión mutua:** propiedad que evita acceso simultáneo incompatible a una región
crítica.

**Mutex:** primitiva de exclusión mutua normalmente asociada a un propietario.

**Semáforo:** contador de sincronización utilizado para coordinar recursos o eventos.

**Semáforo binario:** semáforo limitado conceptualmente a dos estados.

**Monitor:** abstracción de alto nivel que combina estado, exclusión mutua y coordinación.

**Variable de condición:** mecanismo para esperar a que cambie determinado estado.

**Operación atómica:** operación que aparece indivisible frente a participantes
concurrentes según el modelo correspondiente.

**IPC:** mecanismos de comunicación entre procesos.

**Pipe:** canal de bytes con extremos de lectura y escritura.

**FIFO:** pipe con nombre visible en el sistema de archivos.

**Socket:** mecanismo de comunicación local o de red.

**Señal:** mecanismo de notificación de eventos en Unix-like.

**Deadlock:** bloqueo permanente de un conjunto de tareas que se esperan circularmente.

**Condiciones de Coffman:** exclusión mutua, retener y esperar, sin apropiación y espera
circular.

---

## Desafiate con preguntas de examen

1. ¿Qué diferencia existe entre un proceso y un hilo?
2. ¿Qué recursos comparten los hilos de un mismo proceso?
3. ¿Qué información necesita mantener individualmente cada hilo?
4. ¿Por qué un hilo suele ser más liviano que un proceso?
5. ¿Qué diferencia existe entre concurrencia y paralelismo?
6. ¿Por qué puede haber problemas de concurrencia con una sola CPU?
7. ¿Qué es una condición de carrera?
8. ¿Por qué `contador = contador + 1` no debe suponerse atómico?
9. ¿Qué diferencia conceptual existe entre race condition y data race?
10. ¿Qué es una región crítica?
11. ¿Qué propiedades busca una solución clásica a la sección crítica?
12. ¿Qué significa exclusión mutua?
13. ¿Cómo funciona conceptualmente un mutex?
14. ¿Qué papel cumplen las operaciones atómicas en la sincronización?
15. ¿Cómo funciona un semáforo contador?
16. ¿Por qué un semáforo binario y un mutex no son exactamente la misma cosa?
17. ¿Qué es un monitor?
18. ¿Para qué sirven las variables de condición?
19. ¿Qué significa IPC?
20. ¿Qué diferencias generales existen entre memoria compartida y pasaje de mensajes?
21. ¿Cómo utiliza Bash un pipe?
22. ¿Qué diferencia existe entre un pipe anónimo y un FIFO?
23. ¿Por qué los pipes son un ejemplo real de IPC?
24. ¿Qué diferencia existe entre un pipe y una señal?
25. ¿Qué información muestra `/proc/PID/task`?
26. ¿Qué relación existe entre PID, TID y TGID en Linux?
27. ¿Qué es un deadlock?
28. ¿Cuáles son las cuatro condiciones de Coffman?
29. ¿Por qué no es preciso decir que “si las cuatro condiciones existen, el sistema está
    necesariamente en deadlock”?
30. ¿Cómo puede un orden global de adquisición de locks prevenir espera circular?
31. ¿Qué diferencia existe entre prevención, evitación y detección de deadlocks?
32. ¿Qué significa la llamada estrategia del avestruz?

---

## Próxima clase

En la [próxima clase](CLASE_09_MEMORIA_ORGANIZACION_ADMINISTRACION_SEGURIDAD.md) estudiaremos **administración de memoria**.

Volveremos sobre una afirmación que utilizamos desde la [Clase 6](CLASE_06_PROCESOS_ESTADOS_PCB.md):

> cada proceso trabaja con su propio espacio de direcciones.

Ahora podremos explicar cómo se consigue mediante:

- memoria virtual;
- páginas;
- marcos;
- tablas de páginas;
- TLB;
- fallos de página;
- protección;
- reemplazo de páginas.
