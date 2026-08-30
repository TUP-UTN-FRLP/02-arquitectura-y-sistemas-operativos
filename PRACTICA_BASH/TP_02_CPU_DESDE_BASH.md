# Trabajo Práctico 02 — CPU desde Bash

**Arquitectura y Sistemas Operativos**  
**Tecnicatura Universitaria en Programación**

---

## Propósito

En este TP vamos a utilizar Bash para observar la CPU que Linux tiene disponible.

El eje será:

```text
CPU
├── arquitectura
├── núcleos
├── procesadores lógicos
├── modelo
├── cachés
└── capacidades
```

El TP acompaña la **[Clase 2 — Arquitectura: la CPU](../MANUAL/CLASE_02_ARQUITECTURA_CPU.md)**.

Todavía no escribiremos scripts Bash.

---

## Parte 1 — Preparar el espacio de trabajo

```bash
mkdir -p ~/aso/tp02
cd ~/aso/tp02
pwd
code .
```

Creá `RESPUESTAS.md`.

---

## Parte 2 — Arquitectura de la CPU

Ejecutá:

```bash
lscpu
```

Registrá:

```text
Architecture:
CPU(s):
Thread(s) per core:
Core(s) per socket:
Socket(s):
Model name:
```

Respondé:

1. ¿Qué significa `Architecture`?
2. ¿Qué diferencia existe entre núcleo físico y procesador lógico?
3. ¿Por qué `CPU(s)` puede ser mayor que la cantidad de núcleos?
4. ¿Qué relación tiene esto con SMT?

---

## Parte 3 — `nproc`

Ejecutá:

```bash
nproc
```

Compará con `lscpu`.

Respondé:

1. ¿Coinciden `nproc` y `CPU(s)`?
2. Si no coinciden, ¿qué podría explicar la diferencia?
3. ¿`nproc` informa necesariamente la cantidad de chips físicos?

---

## Parte 4 — `/proc/cpuinfo`

Ejecutá:

```bash
cat /proc/cpuinfo
```

Como la salida puede ser larga:

```bash
less /proc/cpuinfo
```

Salir:

```text
q
```

Respondé:

1. ¿Por qué aparecen muchas secciones parecidas?
2. ¿Qué significa cada entrada `processor`?
3. ¿`/proc/cpuinfo` es un archivo común almacenado igual que un `.txt`?

---

## Parte 5 — Primer filtro con `grep`

Probá:

```bash
lscpu | grep "Architecture"
lscpu | grep "Model name"
lscpu | grep "CPU(s)"
```

El símbolo `|` es un **pipe**.

Por ahora pensalo así:

```text
salida de un comando
        ↓
      pipe
        ↓
entrada de otro comando
```

Respondé:

1. ¿Qué hace `grep`?
2. ¿Qué ventaja tiene filtrar una salida larga?
3. ¿Por qué esto puede ser útil para un programador?

---

## Parte 6 — Búsqueda sin distinguir mayúsculas

Probá:

```bash
lscpu | grep -i "cache"
```

Respondé:

1. ¿Qué cambia con `-i`?
2. ¿Qué información de caché aparece?
3. ¿Cómo se relaciona con el cuello de botella entre CPU y memoria?

---

## Parte 7 — Redirección de salida

Guardá la información en un archivo:

```bash
lscpu > cpu.txt
```

Verificá:

```bash
ls -l
cat cpu.txt
```

La redirección `>` envía la salida estándar a un archivo.

---

## Parte 8 — Sobrescribir y agregar

Probá:

```bash
echo "=== CPU ===" > informe.txt
lscpu >> informe.txt
echo "=== NPROC ===" >> informe.txt
nproc >> informe.txt
cat informe.txt
```

Respondé:

1. ¿Qué diferencia existe entre `>` y `>>`?
2. ¿Qué ocurre si volvés a ejecutar el primer `echo`?
3. ¿Por qué esto es útil para generar archivos de diagnóstico?

---

## Parte 9 — Combinar pipe y redirección

Probá:

```bash
lscpu | grep -i "model" > modelo.txt
lscpu | grep -i "cache" > caches.txt
```

Leé los archivos:

```bash
cat modelo.txt
cat caches.txt
```

Explicá:

```text
lscpu
  ↓
grep
  ↓
archivo
```

---

## Parte 10 — Reflexión sobre herramientas

Ya pudiste obtener la arquitectura y la cantidad de procesadores lógicos
usando comandos de la terminal.

Respondé en `RESPUESTAS.md`:

> Imaginá que estás desarrollando un programa y necesitás saber en qué
> arquitectura va a ejecutarse antes de correrlo. ¿Cuándo preferirías
> consultar eso desde la terminal, y cuándo tendría sentido que el propio
> programa lo detecte automáticamente?

No hay una respuesta única. El objetivo es empezar a pensar en cuándo
conviene usar una herramienta del sistema y cuándo escribir código propio.

---

## Parte 11 — Arquitectura y ejecutables

Ejecutá:

```bash
uname -m
file /bin/bash
file /bin/ls
```

Respondé:

1. ¿Qué arquitectura informa `uname -m`?
2. ¿Qué arquitectura tienen los ejecutables?
3. ¿Por qué la ISA importa para un programa compilado?
4. ¿Podría una CPU ARM ejecutar directamente instrucciones x86-64 sin traducción?

---

## Desafío integrador

Sin escribir un `.sh`, construí desde la terminal un archivo:

```text
diagnostico_cpu.txt
```

que incluya solamente:

```text
arquitectura
modelo
cantidad de CPU lógicas
información de cachés
```

Debés utilizar una combinación de:

```text
echo
lscpu
nproc
grep
>
>>
|
```

No copies manualmente los valores.

---

## Entrega

```text
TP_02/
├── RESPUESTAS.md
└── diagnostico_cpu.txt
```

---

## Criterios de revisión

Se tendrá en cuenta:

- interpretación de CPU, núcleo y procesador lógico;
- uso básico de `/proc`;
- uso de `grep`;
- comprensión inicial del pipe;
- diferenciación entre `>` y `>>`;
- capacidad de combinar comandos para resolver una tarea;
- relación entre Bash y una necesidad concreta del programador.

> El objetivo ya no es solamente ejecutar un comando. Empezamos a aprender a **combinar herramientas**.
