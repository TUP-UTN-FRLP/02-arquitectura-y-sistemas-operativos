# Trabajo Práctico 10 — Sistemas de archivos, permisos y E/S para programadores

**Arquitectura y Sistemas Operativos**  
**Tecnicatura Universitaria en Programación**

---

## Propósito

Los programadores trabajan continuamente con:

- archivos;
- directorios;
- permisos;
- rutas;
- logs;
- entrada y salida estándar.

En este TP vamos a conectar esas tareas cotidianas con las estructuras del sistema
operativo.

El TP acompaña la **[Clase 10 — Sistemas de archivos y dispositivos de E/S](../MANUAL/CLASE_10_SISTEMAS_DE_ARCHIVOS_Y_DISPOSITIVOS_IO.md)**.

---

## Conceptos Bash que incorporamos o reforzamos

```text
stat
ls -i
ln
ln -s
readlink
chmod
find
du
df
findmnt
/proc/PID/fd
stdin stdout stderr
2>
2>&1
/dev/null
here document
```

---

## Parte 1 — Proyecto de práctica

Creá:

```bash
mkdir -p ~/aso/tp10/proyecto/{src,docs,logs,backup}
cd ~/aso/tp10/proyecto
```

Creá algunos archivos:

```bash
touch src/main.txt
touch src/utilidades.txt
touch docs/README.md
echo "INFO inicio" > logs/app.log
```

Visualizá:

```bash
find . -type f
```

---

## Parte 2 — Metadatos e inodos

Ejecutá:

```bash
ls -li src/main.txt
stat src/main.txt
```

Identificá:

```text
inode
tamaño
bloques
propietario
permisos
fechas
```

Respondé:

> ¿Qué diferencia existe entre el nombre del archivo y su inodo?

---

## Parte 3 — Hard link

Creá:

```bash
echo "documentación" > docs/original.txt
ln docs/original.txt docs/otro_nombre.txt
```

Compará:

```bash
ls -li docs/original.txt docs/otro_nombre.txt
```

Eliminá:

```bash
rm docs/original.txt
```

Leé:

```bash
cat docs/otro_nombre.txt
```

Respondé:

1. ¿Por qué el contenido sigue existiendo?
2. ¿Qué comparten los dos nombres?

---

## Parte 4 — Enlace simbólico

Creá:

```bash
ln -s otro_nombre.txt docs/acceso.txt
```

Observá:

```bash
ls -li docs
readlink docs/acceso.txt
```

Respondé:

1. ¿El enlace simbólico posee el mismo inodo?
2. ¿Qué contiene conceptualmente?
3. ¿Qué diferencia existe con un hard link?

---

## Parte 5 — Permisos

Sobre:

```text
src/main.txt
```

ejecutá:

```bash
ls -l src/main.txt
chmod 600 src/main.txt
ls -l src/main.txt
chmod 644 src/main.txt
ls -l src/main.txt
```

Respondé:

1. ¿Qué representa `600`?
2. ¿Qué representa `644`?
3. ¿Qué diferencia hay entre permisos de usuario, grupo y otros?

---

## Parte 6 — Un script ejecutable

Creá:

```text
src/estado.sh
```

```bash
#!/bin/bash

echo "Proyecto: $(pwd)"
echo "Usuario: $(whoami)"
date
```

Probá sin permiso de ejecución.

Después:

```bash
chmod +x src/estado.sh
./src/estado.sh
```

Relacioná esta práctica con el shebang visto en [TP 05](TP_05_PRIMER_SCRIPT_BASH_SISTEMA_OPERATIVO.md).

---

## Parte 7 — Sistemas de archivos y montajes

Ejecutá:

```bash
df -T .
findmnt -T .
lsblk -f
du -sh .
```

Respondé:

1. ¿Qué sistema de archivos contiene tu proyecto?
2. ¿Dónde está montado?
3. ¿Qué diferencia existe entre `df` y `du`?
4. ¿Qué diferencias podés encontrar entre WSL2 y Linux nativo?

---

## Parte 8 — Descriptores de archivo

Ejecutá:

```bash
echo $$
ls -l /proc/$$/fd
```

Identificá:

```text
0
1
2
```

Respondé:

> ¿Qué representan stdin, stdout y stderr?

---

## Parte 9 — Descriptores de un proceso con archivo abierto

Vamos a observar qué descriptores tiene un proceso que mantiene
un archivo abierto.

Ejecutá en una terminal:

```bash
tail -f logs/app.log &
pid=$!
```

`tail -f` mantiene el archivo abierto mientras espera nuevas líneas.

Observá sus descriptores:

```bash
ls -l /proc/$pid/fd
```

Identificá cuál corresponde al archivo abierto.

Terminá el proceso:

```bash
kill "$pid"
wait "$pid" 2>/dev/null
```

Respondé:

1. ¿Qué descriptores aparecen además del archivo?
2. ¿Cuál es stdin, stdout y stderr en ese proceso?
3. ¿Por qué el sistema operativo representa un archivo abierto
   con un número en lugar del nombre del archivo?
4. ¿Qué ventaja tiene ese mecanismo para el sistema operativo?

---

## Parte 10 — stderr

Probá:

```bash
ls archivo_inexistente
```

Redirigí el error:

```bash
ls archivo_inexistente 2> logs/errores.log
```

Verificá:

```bash
cat logs/errores.log
```

Ahora:

```bash
ls . archivo_inexistente > logs/salida.log 2> logs/errores.log
```

Respondé:

1. ¿Qué representa `1`?
2. ¿Qué representa `2`?
3. ¿Por qué separar salida normal y errores es útil en automatizaciones?

---

## Parte 11 — `/dev/null`

Ejecutá:

```bash
echo "esto se descarta" > /dev/null
```

Y:

```bash
ls archivo_inexistente > /dev/null 2>&1
```

Respondé:

> ¿Cuándo puede ser útil descartar una salida y cuándo podría ocultar información
> importante?

---

## Parte 12 — Script de auditoría de proyecto

Creá:

```text
auditar_proyecto.sh
```

Uso:

```bash
./auditar_proyecto.sh RUTA
```

Debe mostrar:

```text
ruta absoluta
cantidad de archivos
cantidad de directorios
cantidad de .txt
cantidad de .md
tamaño total
archivos ejecutables
archivos modificados más recientemente
```

Podés utilizar:

```bash
realpath
find
wc
du
ls
head
```

Debe validar que la ruta exista y sea un directorio.

---

## Parte 13 — Copia simple del proyecto

Sin utilizar `sudo`, creá una copia:

```bash
cp -r ~/aso/tp10/proyecto ~/aso/tp10/proyecto_copia
```

Compará cantidad de archivos:

```bash
find proyecto -type f | wc -l
find proyecto_copia -type f | wc -l
```

No buscamos implementar un sistema de backup completo.

La finalidad es pensar en:

```text
archivos
metadatos
espacio
automatización
```

---

## Parte 14 — E/S y dispositivos

Ejecutá:

```bash
ls -l /dev/null
ls -l /dev/zero
ls -l /dev/tty
```

Respondé:

1. ¿Por qué `/dev` resulta conceptualmente interesante en UNIX?
2. ¿Qué diferencia existe entre un driver y una controladora de hardware?
3. ¿Qué función cumplen interrupciones y DMA?

---

## Parte 15 — Planificación de HDD

Resolver conceptualmente una secuencia de solicitudes para:

```text
FCFS
SSTF
SCAN
C-SCAN
```

El docente indicará:

- posición inicial;
- solicitudes pendientes;
- dirección inicial para SCAN.

Respondé:

> ¿Por qué estos algoritmos pierden gran parte de su motivación mecánica en un SSD, pero
> la planificación de E/S sigue existiendo?

---

## Desafío integrador

Mejorá `auditar_proyecto.sh` para permitir:

```bash
./auditar_proyecto.sh ~/mi_proyecto > reporte.txt
```

El reporte debe ser suficientemente claro como para enviárselo a otro desarrollador.

No debe modificar el proyecto analizado.

---

## Entrega

```text
TP_10/
├── RESPUESTAS.md
├── proyecto/
├── proyecto_copia/
├── auditar_proyecto.sh
└── reporte.txt
```

---

## Criterios de revisión

Se observará:

- comprensión de archivos, nombres e inodos;
- enlaces duros y simbólicos;
- permisos;
- descriptores;
- redirecciones de stdout/stderr;
- interpretación de montajes;
- uso de herramientas de búsqueda y conteo;
- scripting aplicado a un proyecto real;
- comprensión conceptual de E/S.

> Este TP busca que el sistema de archivos deje de ser solamente “carpetas”:
>
> **es una abstracción programable que utilizamos todos los días al desarrollar software.**
