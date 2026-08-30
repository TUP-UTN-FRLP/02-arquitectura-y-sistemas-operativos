# Trabajo Práctico 04 — Bash para diagnosticar el entorno de programación

**Arquitectura y Sistemas Operativos**  
**Tecnicatura Universitaria en Programación**

---

## Propósito

Este TP cierra el bloque inicial de Arquitectura.

Vamos a integrar Bash con los conceptos estudiados para construir un diagnóstico del
entorno donde desarrollamos software.

El objetivo es que puedas responder desde la terminal:

```text
¿qué sistema estoy usando?
¿qué arquitectura?
¿32 o 64 bits?
¿qué CPU ve Linux?
¿qué ejecutables estoy usando?
¿estoy en WSL2?
¿dónde están mis herramientas?
```

Todavía no vamos a escribir scripts.

Eso comienza en el **[TP 05](TP_05_PRIMER_SCRIPT_BASH_SISTEMA_OPERATIVO.md)**.

---

## Parte 1 — Preparar el proyecto

```bash
mkdir -p ~/aso/tp04
cd ~/aso/tp04
code .
```

Creá `RESPUESTAS.md`.

---

## Parte 2 — Arquitectura visible

Ejecutá:

```bash
uname -m
getconf LONG_BIT
lscpu | grep -E 'Architecture|CPU\(s\)|Core|Thread|Socket'
```

Respondé:

1. ¿Qué arquitectura informa Linux?
2. ¿El entorno es de 32 o 64 bits?
3. ¿Cuántos procesadores lógicos aparecen?
4. ¿Cuántos núcleos físicos aparecen?
5. ¿Qué relación existe entre esos datos?

---

## Parte 3 — Ejecutables y arquitectura

Ejecutá:

```bash
file /bin/bash
file /bin/ls
```

Ejecutá también:

```bash
file /usr/bin/ls
file /usr/bin/grep
```

Respondé:

1. ¿Todos los ejecutables tienen la misma arquitectura?
2. ¿Qué pasaría si un ejecutable fuera de una arquitectura
   diferente a la del sistema?

---

## Parte 4 — `command -v` y `type`

Probá:

```bash
command -v bash
command -v code
type cd
type echo
type ls
```

Respondé:

1. ¿`cd` es un ejecutable externo?
2. ¿Qué diferencia observás entre `cd` y `ls`?
3. ¿Por qué una shell necesita algunos comandos internos?
4. ¿Por qué `command -v` es útil al programar?

---

## Parte 5 — Variables de entorno

Mostrá:

```bash
echo "$HOME"
echo "$USER"
echo "$SHELL"
echo "$PATH"
```

Respondé:

1. ¿Qué representa `HOME`?
2. ¿Qué representa `USER`?
3. ¿Qué representa `SHELL`?
4. ¿Qué función cumple `PATH`?

---

## Parte 6 — Explorar `PATH`

Convertí los `:` de `PATH` en saltos de línea:

```bash
echo "$PATH" | tr ':' '\n'
```

Respondé:

1. ¿Qué hizo `tr`?
2. ¿Por qué Bash busca programas en esos directorios?
3. ¿Qué ocurriría si `bash` no estuviera en ningún directorio de `PATH`?

---

## Parte 7 — Procesar texto con `cut`

Probá:

```bash
echo "usuario:sergio:programador" | cut -d ':' -f 2
```

Después:

```bash
echo "$PATH" | tr ':' '\n' | head
```

La idea es comenzar a reconocer un patrón:

```text
salida
→ transformación
→ filtro
```

No hace falta memorizar todas las opciones.

---

## Parte 8 — Detectar WSL2

Ejecutá:

```bash
cat /proc/version
grep -i "microsoft" /proc/version
uname -r
```

Respondé:

1. ¿Aparece alguna referencia a Microsoft?
2. ¿Qué conclusión razonable podés sacar?
3. ¿Por qué esto no sería necesario en Linux nativo?

---

## Parte 9 — PowerShell y Linux ven capas diferentes

Si trabajás con Windows + WSL2, desde PowerShell ejecutá:

```powershell
wsl -l -v
```

Después, dentro de Ubuntu:

```bash
uname -a
ip addr
lsblk
```

Respondé:

> ¿Por qué Windows y Ubuntu pueden presentar visiones diferentes del mismo equipo?

Relacioná la respuesta con:

```text
virtualización
host
guest
hardware virtual
```

---

## Parte 10 — Comparar archivos con `diff`

Creá:

```bash
echo "version 1" > version_a.txt
echo "version 2" > version_b.txt
```

Compará:

```bash
diff version_a.txt version_b.txt
```

Respondé:

1. ¿Qué informa `diff`?
2. ¿Por qué puede ser útil para un programador?
3. ¿Qué tipo de archivos compararías con esta herramienta?

---

## Parte 11 — Buscar archivos con `find`

Desde `~/aso` ejecutá:

```bash
find . -name "*.md"
find . -type f
```

Respondé:

1. ¿Qué diferencia existe entre buscar por nombre y buscar por tipo?
2. ¿Cómo utilizarías `find` en un proyecto con cientos de archivos?

---

## Parte 12 — Buscar dentro de archivos de texto

Desde `~/aso` ejecutá:

```bash
grep -R "RESPUESTAS" ~/aso --include="*.md"
grep -R "====" ~/aso --include="*.md"
```

Respondé:

1. ¿Qué significa `-R`?
2. ¿Qué hace `--include="*.md"`?
3. ¿Por qué esto puede ser útil para encontrar información
   dentro de muchos archivos?

---

## Parte 13 — Contar resultados

Probá:

```bash
find ~/aso -name "*.md" | wc -l
find ~/aso -name "*.txt" | wc -l
grep -R "===" ~/aso --include="*.md" | wc -l
```

Respondé:

> ¿Qué patrón común tienen estos tres comandos?

---

## Parte 14 — Generar un diagnóstico sin script

Creá desde la terminal:

```text
diagnostico_entorno.txt
```

Debe contener automáticamente:

```text
=== USUARIO ===

=== SISTEMA ===

=== ARQUITECTURA ===

=== CPU ===

=== MEMORIA ===

=== GIT ===

=== BASH ===

=== WSL ===
```

Podés utilizar:

```text
echo
whoami
uname
lscpu
free
command -v
file
grep
>
>>
|
```

No copies manualmente la información.

Para la sección `=== GIT ===`, usá `command -v git`: si el comando devuelve una
ruta, git está instalado; si no devuelve nada, no lo está.

---

## Parte 15 — Reflexión sobre automatización

A lo largo de este TP construiste un diagnóstico completo del entorno
usando únicamente herramientas de la terminal.

Respondé en `RESPUESTAS.md`:

> Pensá en alguna tarea repetitiva que hagas habitualmente con tu
> computadora: verificar algo, organizar archivos, buscar información.
> ¿Creés que esa tarea podría automatizarse con comandos de la terminal?
> ¿Qué pasos implicaría?

No hace falta que la respuesta sea técnicamente perfecta. El objetivo
es empezar a pensar en automatización como concepto.

---

## Parte 16 — Lo que ya sabemos hacer con Bash

Al finalizar este TP ya utilizaste:

```text
pwd
cd
ls
mkdir
touch
rm
cat
less
echo
uname
lscpu
nproc
free
lsblk
df
du
findmnt
grep
head
tail
wc
tr
cut
file
type
command -v
find
diff
>
>>
|
```

No necesitás memorizarlos todos.

Lo importante es reconocer familias de problemas:

```text
navegar
inspeccionar
buscar
filtrar
transformar
comparar
contar
redirigir
encadenar
```

---

## Parte 17 — Preparación del TP 05

Hasta acá usamos Bash **interactivamente**.

Cada vez que queríamos un diagnóstico ejecutábamos nuevamente:

```text
comando 1
comando 2
comando 3
...
```

La próxima pregunta es:

> ¿Qué pasa si quiero repetir siempre la misma secuencia?

Ahí aparece el script.

En el **[TP 05](TP_05_PRIMER_SCRIPT_BASH_SISTEMA_OPERATIVO.md)**, junto con el inicio de Sistemas Operativos, comenzaremos con:

```text
archivo .sh
shebang
permisos de ejecución
variables
sustitución de comandos
parámetros
condicionales sencillos
```

Y el primer objetivo será automatizar muchas de las tareas que ya aprendimos manualmente.

---

## Desafío integrador

Sin escribir todavía un `.sh`, construí una única línea de comandos que responda:

> ¿Cuántos archivos `.md` de tus prácticas contienen la palabra `=====`?

Podés combinar:

```text
grep
find
wc
|
```

No hay una única solución obligatoria.

Después explicá cada etapa de tu comando.

---

## Entrega

```text
TP_04/
├── RESPUESTAS.md
└── diagnostico_entorno.txt
```

---

## Criterios de revisión

Se observará:

- integración de conceptos de arquitectura;
- uso de Bash para problemas concretos de programación;
- manejo de variables de entorno;
- búsqueda de archivos y contenido;
- filtros y transformación de texto;
- uso razonado de pipes y redirecciones;
- reflexión sobre automatización y elección de herramientas;
- comprensión del papel de WSL2 y la virtualización.

---

## Cierre del primer bloque

Los cuatro TP quedan organizados así:

```text
TP 01
terminal + navegación + observar hardware

TP 02
CPU + grep + pipes + redirecciones

TP 03
memoria/almacenamiento + filtros + conteos

TP 04
entorno del programador + find/diff/PATH + integración

TP 05
inicio de Sistemas Operativos
+
inicio de scripting Bash
```

La idea es que cuando llegue el TP 05 no tengamos que aprender simultáneamente:

```text
qué es un comando
+
qué es un pipe
+
qué es una redirección
+
cómo se hace un script
```

Los primeros conceptos ya estarán incorporados.

Entonces podremos concentrarnos en **automatizar**.
