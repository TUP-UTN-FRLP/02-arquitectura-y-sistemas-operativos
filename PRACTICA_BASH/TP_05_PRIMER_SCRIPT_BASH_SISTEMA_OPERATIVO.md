# Trabajo Práctico 05 — Primer script Bash: diagnóstico del sistema operativo

**Arquitectura y Sistemas Operativos**  
**Tecnicatura Universitaria en Programación**

---

## Propósito

Hasta el [TP 04](TP_04_BASH_DIAGNOSTICO_ENTORNO_PROGRAMACION.md) utilizamos Bash de forma interactiva:

```text
comando
→ filtro
→ redirección
→ resultado
```

A partir de este TP comenzamos a **programar scripts Bash**.

El cambio coincide con el inicio formal del bloque de Sistemas Operativos. La idea es
automatizar tareas que un programador realiza con frecuencia:

- identificar el entorno;
- verificar herramientas;
- obtener información del sistema;
- generar diagnósticos;
- detectar errores antes de ejecutar un proyecto.

El TP acompaña la **[Clase 5 — Sistema operativo: funciones y evolución](../MANUAL/CLASE_05_SISTEMA_OPERATIVO_FUNCIONES_EVOLUCION.md)**.

---

## Conceptos Bash que incorporamos

En este TP aparecen por primera vez:

```text
archivo .sh
#!/bin/bash
chmod +x
variables
"$variable"
$(comando)
parámetros
$0 $1
if
test / [ ]
exit
código de salida $?
```

No buscamos aprender Bash como un lenguaje aislado. Cada recurso se incorpora porque
resuelve una necesidad concreta.

---

## Parte 1 — Crear el primer script

Prepará:

```bash
mkdir -p ~/aso/tp05
cd ~/aso/tp05
code .
```

Creá:

```text
hola.sh
```

Contenido:

```bash
#!/bin/bash

echo "Hola desde Bash"
echo "Usuario:"
whoami

echo "Kernel:"
uname -r
```

Desde la terminal:

```bash
chmod +x hola.sh
./hola.sh
```

Respondé:

1. ¿Qué función cumple `#!/bin/bash`?
2. ¿Por qué fue necesario `chmod +x`?
3. ¿Qué diferencia existe entre escribir tres comandos manualmente y guardarlos en un
   script?

---

## Parte 2 — Variables

Creá `variables.sh`:

```bash
#!/bin/bash

materia="Arquitectura y Sistemas Operativos"
usuario="$USER"

echo "Materia: $materia"
echo "Usuario: $usuario"
```

Ejecutá:

```bash
chmod +x variables.sh
./variables.sh
```

Respondé:

1. ¿Por qué no hay espacios alrededor de `=`?
2. ¿Qué diferencia existe entre `usuario` y `$usuario`?
3. ¿De dónde proviene `$USER`?
4. ¿Por qué es una buena práctica utilizar `"$variable"` cuando el contenido puede
   incluir espacios?

---

## Parte 3 — Sustitución de comandos

Creá:

```text
entorno.sh
```

```bash
#!/bin/bash

usuario=$(whoami)
equipo=$(hostname)
kernel=$(uname -r)
arquitectura=$(uname -m)

echo "Usuario: $usuario"
echo "Equipo: $equipo"
echo "Kernel: $kernel"
echo "Arquitectura: $arquitectura"
```

La forma:

```bash
variable=$(comando)
```

permite guardar la salida de un comando.

Respondé:

> ¿Qué ventaja aporta esto frente a ejecutar directamente `uname -r` dentro de `echo`?

---

## Parte 4 — Kernel, distribución y shell

Ampliá `entorno.sh` para mostrar:

```text
Usuario
Equipo
Distribución
Kernel
Arquitectura
Shell
```

Para la distribución podés consultar:

```bash
grep '^PRETTY_NAME=' /etc/os-release
```

Para la shell:

```bash
echo "$SHELL"
```

No copies los valores manualmente.

---

## Parte 5 — Bash también es un proceso

Ejecutá:

```bash
echo $$
```

Luego:

```bash
ps -p $$ -o pid,ppid,user,stat,comm,args
```

Agregá a tu script:

```bash
echo "PID de este script: $$"
```

Respondé:

1. ¿Qué representa `$$`?
2. ¿Por qué Bash aparece como un proceso?
3. ¿Qué diferencia conceptual existe entre shell, kernel y sistema operativo?

---

## Parte 6 — Parámetros

Creá:

```text
saludar.sh
```

```bash
#!/bin/bash

echo "Script: $0"
echo "Primer parámetro: $1"
```

Ejecutá:

```bash
chmod +x saludar.sh
./saludar.sh Sergio
```

Después:

```bash
./saludar.sh Ana
```

Respondé:

1. ¿Qué representa `$0`?
2. ¿Qué representa `$1`?
3. ¿Qué ocurre si ejecutás el script sin parámetro?

---

## Parte 7 — Primera validación con `if`

Modificá `saludar.sh`:

```bash
#!/bin/bash

if [ -z "$1" ]; then
    echo "Uso: $0 NOMBRE"
    exit 1
fi

echo "Hola, $1"
```

Probá:

```bash
./saludar.sh
echo $?
```

Después:

```bash
./saludar.sh Ana
echo $?
```

Respondé:

1. ¿Qué prueba `-z`?
2. ¿Qué hace `exit 1`?
3. ¿Qué contiene `$?`?
4. ¿Por qué los códigos de salida son útiles para encadenar herramientas y automatizar?

---

## Parte 8 — Verificar herramientas de desarrollo

Creá:

```text
verificar_herramienta.sh
```

Debe recibir el nombre de un comando.

Ejemplo:

```bash
./verificar_herramienta.sh curl
./verificar_herramienta.sh git
./verificar_herramienta.sh comando_inexistente
```

Podés usar:

```bash
command -v "$1"
```

El comportamiento esperado:

```text
curl está disponible en /usr/bin/curl
```

o:

```text
comando_inexistente NO está disponible
```

El script debe:

- validar que exista `$1`;
- devolver `0` si encuentra la herramienta;
- devolver un código distinto de cero si no la encuentra.

---

## Parte 9 — Sistema operativo como máquina extendida

En `RESPUESTAS.md` explicá la relación entre:

```text
programa
↓
biblioteca/runtime
↓
llamada al sistema
↓
kernel
↓
hardware
```

Después relacioná:

| Recurso real    | Abstracción del SO |
| --------------- | ------------------ |
| CPU             |                    |
| memoria física  |                    |
| almacenamiento  |                    |
| interfaz de red |                    |
| dispositivos    |                    |

---

## Parte 10 — `/proc`

Ejecutá:

```bash
cat /proc/version
cat /proc/$$/status
```

Filtrá:

```bash
grep -E '^(Name|State|Pid|PPid|Threads):' /proc/$$/status
```

Respondé:

1. ¿Qué información expone `/proc`?
2. ¿Por qué `/proc` es útil para un programador?
3. ¿Esos archivos deben interpretarse como documentos normales guardados en el disco?

---

## Parte 11 — `strace` opcional

Si `strace` está instalado:

```bash
strace --version
```

Creá:

```bash
echo "hola" > datos.txt
```

Observá:

```bash
strace -e trace=openat,read,write,close cat datos.txt
```

Si no está instalado, esta parte puede omitirse o instalarse con autorización del
docente.

Respondé:

> ¿Qué relación observás entre una orden sencilla como `cat datos.txt` y las llamadas al
> sistema?

---

## Desafío integrador — `sistema.sh`

Creá un script:

```text
sistema.sh
```

que genere en pantalla un informe como:

```text
=== INFORME DEL SISTEMA ===

Usuario:
Equipo:
Distribución:
Kernel:
Arquitectura:
Shell:
PID del script:
curl:
Git:
```

Requisitos:

- usar variables;
- usar al menos dos sustituciones `$(...)`;
- verificar curl y Git con `command -v`;
- no escribir manualmente datos que Bash pueda obtener;
- devolver código `0` si termina correctamente.

Luego ejecutá:

```bash
./sistema.sh > informe_sistema.txt
```

---

## Entrega

```text
TP_05/
├── RESPUESTAS.md
├── hola.sh
├── variables.sh
├── entorno.sh
├── saludar.sh
├── verificar_herramienta.sh
├── sistema.sh
└── informe_sistema.txt
```

---

## Criterios de revisión

Se tendrá en cuenta:

- comprensión de qué es un script;
- uso de shebang y permisos;
- variables y sustitución de comandos;
- parámetros;
- validaciones sencillas;
- códigos de salida;
- relación entre Bash, shell, kernel y sistema operativo;
- utilidad concreta del script para un entorno de programación.

> A partir de este TP ya no usamos Bash solamente para ejecutar órdenes:
>
> **empezamos a programar automatizaciones.**
