# Trabajo Práctico 06 — Procesos: observar, controlar y automatizar

**Arquitectura y Sistemas Operativos**  
**Tecnicatura Universitaria en Programación**

---

## Propósito

Un programa almacenado es una entidad pasiva. Cuando se ejecuta aparece un **proceso**,
con PID, estado, memoria y recursos propios.

En este TP vamos a utilizar Bash para:

- crear procesos;
- observar PID y PPID;
- ejecutar trabajos en segundo plano;
- suspender y continuar procesos;
- enviar señales;
- esperar su terminación;
- inspeccionar `/proc/PID`;
- construir un script de diagnóstico de procesos.

El TP acompaña la **[Clase 6 — Procesos: concepto, estados y PCB](../MANUAL/CLASE_06_PROCESOS_ESTADOS_PCB.md)**.

---

## Conceptos Bash que reforzamos

```text
&
$!
$$
jobs
fg
bg
wait
kill
parámetros
if
exit
/proc/PID
```

---

## Parte 1 — Programa vs proceso

Ejecutá tres veces:

```bash
sleep 300 &
```

Observá:

```bash
ps -C sleep -o pid,ppid,stat,comm,args
```

Respondé:

1. ¿Cuántos programas diferentes utilizamos?
2. ¿Cuántos procesos existen?
3. ¿Por qué tienen PID diferentes?
4. ¿Qué representa el PPID?

Terminá los procesos antes de continuar.

---

## Parte 2 — `$!`

Ejecutá:

```bash
sleep 300 &
pid=$!

echo "$pid"
```

Observá:

```bash
ps -p "$pid" -o pid,ppid,stat,comm,args
```

Respondé:

> ¿Qué diferencia existe entre `$$` y `$!`?

---

## Parte 3 — Jobs de Bash

Ejecutá:

```bash
sleep 300 &
sleep 300 &
jobs
jobs -l
```

Respondé:

1. ¿Qué diferencia existe entre un número de job y un PID?
2. ¿Un job tiene que corresponder siempre exactamente a un único proceso?

---

## Parte 4 — Primer plano y segundo plano

Ejecutá:

```bash
sleep 300
```

Suspendelo desde la terminal con:

```text
Ctrl+Z
```

Observá:

```bash
jobs
```

Reanudalo en segundo plano:

```bash
bg
```

Traelo nuevamente:

```bash
fg
```

Terminá con:

```text
Ctrl+C
```

Relacioná esas acciones con los estados estudiados.

---

## Parte 5 — Señales

Creá:

```bash
sleep 300 &
pid=$!
```

Estado:

```bash
ps -p "$pid" -o pid,stat,comm
```

Detenelo:

```bash
kill -STOP "$pid"
```

Volvé a observar.

Reanudalo:

```bash
kill -CONT "$pid"
```

Finalizalo de manera normal:

```bash
kill -TERM "$pid"
```

Respondé:

1. ¿Por qué `kill` no significa necesariamente “matar”?
2. ¿Qué diferencia existe entre `STOP`, `CONT` y `TERM`?
3. ¿Por qué no conviene utilizar `SIGKILL` como primera opción?

---

## Parte 6 — Árbol de procesos

Ejecutá:

```bash
pstree -p $$
```

También:

```bash
ps -o pid,ppid,user,stat,comm,args --forest
```

Buscá:

- tu shell;
- un comando lanzado desde ella;
- procesos padre e hijo.

Representá una pequeña parte del árbol en `RESPUESTAS.md`.

---

## Parte 7 — `/proc/PID`

Creá:

```bash
sleep 300 &
pid=$!
```

Ejecutá:

```bash
cat /proc/$pid/status
```

Filtrá:

```bash
grep -E '^(Name|State|Pid|PPid|VmSize|VmRSS|Threads):' \
    /proc/$pid/status
```

Y:

```bash
head /proc/$pid/maps
```

Respondé:

1. ¿Qué estado informa Linux?
2. ¿Cuál es el PID?
3. ¿Cuál es el PPID?
4. ¿Qué significa `VmSize`?
5. ¿Qué significa `VmRSS`?
6. ¿Por qué el proceso posee varios mapeos de memoria?

---

## Parte 8 — Cambios de contexto

Para el mismo proceso:

```bash
grep ctxt /proc/$pid/status
```

Podés encontrar:

```text
voluntary_ctxt_switches
nonvoluntary_ctxt_switches
```

Respondé:

> ¿Qué relación existe entre esos contadores y el concepto de cambio de contexto?

---

## Parte 9 — `wait` y código de salida

Ejecutá:

```bash
sleep 2 &
pid=$!

wait "$pid"

echo $?
```

Ahora probá:

```bash
bash -c 'exit 7' &
pid=$!

wait "$pid"

echo $?
```

Respondé:

1. ¿Qué hace `wait`?
2. ¿Qué código de salida devuelve el segundo proceso?
3. ¿Por qué un script padre puede necesitar conocer ese valor?

---

## Parte 10 — Script con parámetros

Creá:

```text
inspeccionar_proceso.sh
```

Debe ejecutarse así:

```bash
./inspeccionar_proceso.sh PID
```

Requisitos:

1. comprobar que se recibió un PID;
2. comprobar que existe `/proc/PID`;
3. mostrar:

```text
PID
PPID
estado
usuario
comando
cantidad de hilos
VmSize
VmRSS
cambios de contexto
```

Podés utilizar:

```bash
ps
grep
/proc/$pid/status
```

Si el PID no existe:

```text
No existe un proceso con PID ...
```

y el script debe finalizar con un código de error.

---

## Parte 11 — Función Bash

Agregá a `inspeccionar_proceso.sh`:

```bash
mostrar_titulo() {
    echo
    echo "=== $1 ==="
}
```

Utilizala para separar las secciones.

Respondé:

> ¿Qué ventaja aporta una función cuando una operación se repite?

---

## Desafío integrador — Controlar un proceso de prueba

Creá:

```text
probar_proceso.sh
```

El script debe:

1. lanzar `sleep 60` en segundo plano;
2. guardar el PID;
3. mostrar su estado;
4. enviar `SIGSTOP`;
5. volver a mostrarlo;
6. esperar dos segundos;
7. enviar `SIGCONT`;
8. mostrar nuevamente el estado;
9. enviar `SIGTERM`;
10. realizar `wait`;
11. informar el código de salida.

No utilices PIDs escritos manualmente.

---

## Entrega

```text
TP_06/
├── RESPUESTAS.md
├── inspeccionar_proceso.sh
└── probar_proceso.sh
```

---

## Criterios de revisión

Se observará:

- diferencia entre programa, proceso y job;
- uso correcto de PID y PPID;
- uso de `$!`, `jobs`, `kill` y `wait`;
- interpretación básica de estados Linux;
- uso de `/proc/PID`;
- validación de parámetros;
- funciones Bash sencillas;
- control responsable de procesos.

> El objetivo no es “matar procesos”.
>
> Es comprender que **Bash puede controlar objetos reales administrados por el sistema
> operativo**.
