# Trabajo Práctico 01 — Primer contacto con Linux y Bash

**Arquitectura y Sistemas Operativos**  
**Tecnicatura Universitaria en Programación**

---

## Propósito

En este primer trabajo práctico vamos a comenzar a usar **Linux y Bash desde el inicio de la cursada**, pero todavía sin escribir scripts.

La terminal será nuestro laboratorio para observar el sistema.

El objetivo no es memorizar comandos, sino empezar a responder preguntas como:

> **¿Quién soy en el sistema? ¿Dónde estoy? ¿Qué máquina estoy usando? ¿Qué CPU, memoria y almacenamiento ve Linux?**

Este TP acompaña los contenidos de la **[Clase 1 — Introducción a la arquitectura](../MANUAL/CLASE_01_INTRODUCCION_ARQUITECTURA.md)**.

---

## Entorno de trabajo

Podés trabajar de dos maneras:

1. **Linux nativo**.
2. **Windows + WSL2 + Ubuntu**.

Si usás Windows, desde PowerShell:

```powershell
wsl
```

o:

```powershell
wsl -d Ubuntu
```

Una vez dentro de Ubuntu, los comandos de este TP se ejecutan en **Bash**.

También utilizaremos Visual Studio Code para escribir `RESPUESTAS.md`.

Si VS Code está preparado para trabajar con WSL:

```bash
code .
```

---

## Importante: Bash sí, scripts todavía no

Desde este TP comenzamos a aprender Bash.

Durante los TP 01 a 04 trabajaremos principalmente con:

- comandos interactivos;
- navegación;
- inspección del sistema;
- redirecciones sencillas;
- pipes sencillos;
- búsqueda y filtrado.

Todavía **no vamos a programar scripts `.sh`**.

La creación de scripts Bash comenzará en el **[TP 05](TP_05_PRIMER_SCRIPT_BASH_SISTEMA_OPERATIVO.md)**, coincidiendo con el inicio formal del bloque de **Sistemas Operativos**.

---

## Parte 1 — Abrir una terminal Linux

Dentro de Linux ejecutá:

```bash
whoami
hostname
pwd
```

Respondé en `RESPUESTAS.md`:

1. ¿Qué informa `whoami`?
2. ¿Qué informa `hostname`?
3. ¿Qué informa `pwd`?
4. ¿Qué diferencia existe entre usuario, equipo y directorio actual?

---

## Parte 2 — Primeros comandos de navegación

Ejecutá:

```bash
ls
ls -l
ls -la
```

Respondé:

1. ¿Qué diferencia observás entre `ls`, `ls -l` y `ls -la`?
2. ¿Qué archivos aparecen con `-a` que antes no veías?
3. ¿Qué pensás que significa que un archivo comience con `.`?

---

## Parte 3 — Crear nuestro espacio de trabajo

Creá una carpeta para la materia:

```bash
mkdir -p ~/aso/tp01
cd ~/aso/tp01
pwd
```

Creá un archivo vacío:

```bash
touch prueba.txt
ls -l
```

Después eliminá el archivo:

```bash
rm prueba.txt
ls -l
```

---

## Parte 4 — Rutas

Desde `~/aso/tp01` ejecutá:

```bash
pwd
cd ..
pwd
cd tp01
cd ~
pwd
```

Respondé:

1. ¿Qué representa `~`?
2. ¿Qué representa `..`?
3. ¿Qué diferencia existe entre una ruta absoluta y una ruta relativa?

---

## Parte 5 — ¿Qué sistema estoy usando?

Ejecutá:

```bash
uname
uname -a
cat /etc/os-release
```

Respondé:

1. ¿Qué kernel informa el sistema?
2. ¿Qué distribución estás usando?
3. Si usás WSL2, ¿qué relación tiene Ubuntu con Windows?
4. ¿Linux y Ubuntu son exactamente lo mismo?

---

## Parte 6 — Observar la CPU

Ejecutá:

```bash
lscpu
```

Buscá:

```text
Architecture
CPU(s)
Model name
Core(s) per socket
Thread(s) per core
```

Respondé:

1. ¿Qué arquitectura aparece?
2. ¿Qué modelo de procesador se informa?
3. ¿Cuántas CPU lógicas ve Linux?
4. ¿Cuántos núcleos aparecen?
5. ¿Cuántos hilos por núcleo?

> Si usás WSL2, recordá que Linux puede estar viendo recursos presentados por la virtualización.

---

## Parte 7 — Observar memoria y almacenamiento

Ejecutá:

```bash
free -h
lsblk
df -h
```

Respondé:

1. ¿Cuánta memoria total ve Linux?
2. ¿Cuánta aparece disponible?
3. ¿Qué dispositivos o volúmenes muestra `lsblk`?
4. ¿Qué diferencia inicial observás entre `lsblk` y `df -h`?

---

## Parte 8 — Pedir ayuda

Probá:

```bash
ls --help
```

Después:

```bash
man ls
```

Si `man` no está disponible, utilizá solamente `--help`.

Respondé:

> ¿Por qué es más importante aprender a consultar la ayuda que memorizar todas las opciones de un comando?

---

## Parte 9 — Bash como herramienta para programadores

Entrá nuevamente en:

```bash
cd ~/aso/tp01
```

Abrí el directorio con VS Code:

```bash
code .
```

Desde VS Code creá `RESPUESTAS.md`.

La idea de trabajo a partir de ahora será:

```text
Bash
    → navegar
    → crear y localizar archivos
    → inspeccionar el sistema

VS Code
    → escribir documentación
    → editar código
```

---

## Parte 10 — Relacionar terminal y arquitectura

Completá:

| Comando    | ¿Qué observa? | Concepto de la materia |
| ---------- | ------------- | ---------------------- |
| `lscpu`    |               |                        |
| `free -h`  |               |                        |
| `lsblk`    |               |                        |
| `df -h`    |               |                        |
| `uname -a` |               |                        |
| `hostname` |               |                        |

---

## Desafío integrador

Sin escribir un script, ejecutá los comandos necesarios para obtener:

```text
Usuario:
Equipo:
Distribución:
Kernel:
Arquitectura:
CPU lógicas:
Memoria total:
Almacenamiento:
Directorio actual:
```

Copiá los resultados a `RESPUESTAS.md`.

Después respondé:

> ¿Por qué la terminal puede ser útil para alguien que desarrolla software, sin importar en qué lenguaje esté escrito el programa?

---

## Entrega

```text
TP_01/
└── RESPUESTAS.md
```

---

## Criterios de revisión

Se tendrá en cuenta:

- uso correcto de la terminal Linux;
- comprensión de rutas y directorio actual;
- interpretación básica de `lscpu`, `free`, `lsblk` y `df`;
- diferenciación entre Linux nativo y WSL2;
- relación entre comandos y conceptos de arquitectura;
- claridad del archivo Markdown.

> No se evalúa memorizar comandos. Se evalúa poder **utilizar la terminal para obtener información y comprender qué significa esa información**.
