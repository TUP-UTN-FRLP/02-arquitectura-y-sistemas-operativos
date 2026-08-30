# Trabajo Práctico 08 — Hilos, concurrencia e IPC

**Arquitectura y Sistemas Operativos**  
**Tecnicatura Universitaria en Programación**

---

## Propósito

En esta clase aparecen dos problemas muy cercanos al desarrollo de software:

1. varios hilos pueden acceder a datos compartidos;
2. procesos distintos necesitan comunicarse.

Vamos a utilizar herramientas del sistema y Bash para observar
hilos en procesos reales y experimentar con IPC.

El TP acompaña la **[Clase 8 — Hilos, concurrencia, comunicación y sincronización](../MANUAL/CLASE_08_HILOS_CONCURRENCIA_IPC_SINCRONIZACION.md)**.

---

## Conceptos Bash que incorporamos o reforzamos

```text
ps -T
top -H
/proc/PID/task
pipelines
mkfifo
trap
funciones
cleanup
```

---

## Parte 1 — Encontrar procesos con varios hilos

El sistema ya tiene procesos con múltiples hilos ejecutándose.
Vamos a encontrar uno para observarlo.

Ejecutá:

```bash
ps -eo pid,nlwp,comm --sort=-nlwp | head -10
```

Esto muestra los procesos ordenados por cantidad de hilos.
La columna NLWP indica cuántos hilos tiene cada proceso.

Elegí un proceso con más de un hilo y guardá su PID:

```bash
pid=PID_ELEGIDO
```

Respondé:

1. ¿Qué representa la columna NLWP?
2. ¿Qué procesos del sistema tienen más de un hilo?
3. ¿Por qué el sistema tiene procesos multihilo incluso cuando
   no estamos ejecutando nada especial?

---

## Parte 2 — Observar hilos

Usá el PID elegido en la Parte 1.

Ejecutá:

```bash
ps -T -p "$pid"
```

Más detalle:

```bash
ps -T -p "$pid" -o pid,spid,psr,stat,comm
```

Después:

```bash
ls "/proc/$pid/task"
```

Contá:

```bash
ls "/proc/$pid/task" | wc -l
```

Respondé:

1. ¿Cuántos hilos observás?
2. ¿Qué identificador se mantiene?
3. ¿Qué identificador cambia?
4. ¿Qué relación existe entre PID/TGID y TID en Linux?
5. ¿Todos tienen que estar sobre la misma CPU lógica?

> Es un proceso del sistema: no lo terminamos al final de la observación.

---

## Parte 3 — `top` por hilo

Usá el mismo PID de la Parte 1 y ejecutá:

```bash
top -H -p "$pid"
```

Salir:

```text
q
```

Respondé:

> ¿Qué unidad necesita distinguir el planificador para poder ejecutar varios hilos?

---

## Parte 4 — Condición de carrera: análisis conceptual

Considerá este escenario con dos hilos y una variable compartida
que vale 0:

```text
Hilo A                    Hilo B
lee valor: 0
                          lee valor: 0
calcula: 0 + 1
                          calcula: 0 + 1
escribe: 1
                          escribe: 1

Resultado: 1  (esperado: 2)
```

Respondé:

1. ¿En qué paso exacto ocurre la pérdida del valor?
2. ¿Por qué puede ocurrir esta situación aunque haya una sola CPU?
3. ¿Qué región del código es crítica?
4. ¿Qué necesitaría existir para que el resultado siempre sea
   correcto?

---

## Parte 5 — Exclusión mutua: análisis conceptual

Agregamos exclusión mutua al escenario anterior:

```text
Hilo A                    Hilo B
adquiere lock
lee valor: 0
calcula: 0 + 1
escribe: 1
libera lock
                          adquiere lock
                          lee valor: 1
                          calcula: 1 + 1
                          escribe: 2
                          libera lock

Resultado: 2  (correcto)
```

Respondé:

1. ¿Qué impide ahora la intercalación problemática?
2. ¿Qué ocurre si el Hilo B intenta adquirir el lock mientras
   el Hilo A lo tiene?
3. ¿Por qué la exclusión mutua tiene un costo?
4. ¿Qué relación existe entre este concepto y el mutex estudiado
   en la clase teórica?

---

## Parte 6 — Pipeline como IPC

Ejecutá:

```bash
printf "error\nok\nerror\nok\nerror\n" |
grep error |
wc -l
```

Respondé:

1. ¿Qué procesos participan?
2. ¿Qué dato fluye entre ellos?
3. ¿Qué mecanismo de IPC utiliza Bash?
4. ¿Un pipeline implica necesariamente paralelismo físico?

---

## Parte 7 — Analizar un log

Creá:

```text
app.log
```

con varias líneas:

```text
INFO inicio
ERROR archivo inexistente
INFO usuario conectado
WARNING memoria alta
ERROR timeout
INFO fin
```

Resolvé con pipelines:

1. contar `ERROR`;
2. mostrar solo `WARNING`;
3. contar líneas totales;
4. guardar todos los errores en `errores.txt`.

La idea es aplicar IPC a una tarea típica de programación: **analizar logs**.

---

## Parte 8 — FIFO

Creá:

```bash
mkfifo canal
```

Terminal 1:

```bash
cat canal
```

Terminal 2:

```bash
echo "mensaje desde otro proceso" > canal
```

Al terminar:

```bash
rm canal
```

Respondé:

1. ¿Qué diferencia existe entre un pipe anónimo y un FIFO?
2. ¿Por qué el FIFO aparece en el sistema de archivos?
3. ¿Qué ocurre si un extremo espera y el otro todavía no está disponible?

---

## Parte 9 — `trap` para limpiar recursos

Creá:

```text
fifo_demo.sh
```

Punto de partida:

```bash
#!/bin/bash

fifo="/tmp/aso_fifo_$$"

limpiar() {
    rm -f "$fifo"
}

trap limpiar EXIT

mkfifo "$fifo"

echo "FIFO creado: $fifo"
echo "Escribí desde otra terminal:"
echo "echo hola > $fifo"

cat "$fifo"
```

Ejecutá.

Respondé:

1. ¿Por qué el nombre incluye `$$`?
2. ¿Qué hace `trap ... EXIT`?
3. ¿Por qué limpiar recursos temporales es una práctica importante?

---

## Parte 10 — Mutex vs semáforo

En `RESPUESTAS.md` explicá:

1. ¿qué problema resuelve un mutex?
2. ¿qué diferencia conceptual existe entre mutex y semáforo contador?
3. ¿por qué un semáforo binario no debe considerarse idéntico a un mutex?
4. ¿qué es una variable de condición?

---

## Parte 11 — Deadlock

Analizá:

```text
Hilo A:
    lock(A)
    lock(B)

Hilo B:
    lock(B)
    lock(A)
```

Respondé:

1. ¿Cómo podría producirse un deadlock?
2. ¿Cuáles son las cuatro condiciones de Coffman?
3. ¿Cuál podemos romper si imponemos siempre el orden `A → B`?
4. ¿Por qué no vamos a provocar deliberadamente un deadlock infinito en el laboratorio?

---

## Desafío integrador — Inspector de hilos

Creá:

```text
inspeccionar_hilos.sh
```

Uso:

```bash
./inspeccionar_hilos.sh PID
```

Debe:

- validar el PID;
- mostrar información general del proceso;
- listar sus hilos con `ps -T`;
- contar `/proc/PID/task`;
- mostrar la CPU observada para cada hilo.

Probalo con el proceso multihilo que encontraste en la Parte 1.

---

## Entrega

```text
TP_08/
├── RESPUESTAS.md
├── app.log
├── errores.txt
├── fifo_demo.sh
└── inspeccionar_hilos.sh
```

---

## Criterios de revisión

Se observará:

- diferencia entre proceso e hilo;
- concurrencia vs paralelismo;
- análisis conceptual de condición de carrera;
- comprensión del mecanismo de exclusión mutua;
- comprensión de pipes como IPC;
- uso de FIFO;
- limpieza mediante `trap`;
- análisis conceptual de deadlock;
- utilidad del procesamiento de logs para un programador.

> Esta clase muestra dos caras de la misma idea:
>
> **compartir y comunicar facilita el trabajo, pero obliga a coordinar.**
