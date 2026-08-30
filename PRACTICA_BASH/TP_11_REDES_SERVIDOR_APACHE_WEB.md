# Trabajo Práctico 11 — Redes: publicar una web con Apache

**Arquitectura y Sistemas Operativos**  
**Tecnicatura Universitaria en Programación**

---

## Propósito

Este TP cierra la cursada integrando:

- Linux;
- Bash;
- procesos;
- archivos;
- permisos;
- entrada/salida;
- puertos;
- TCP/IP;
- HTTP;
- modelo cliente-servidor.

Vamos a instalar y levantar un **servidor web Apache**, verificar qué puerto utiliza,
modificar una página HTML y observar cómo un cliente HTTP obtiene ese archivo.

El TP acompaña la **[Clase 11 — Nociones de redes](../MANUAL/CLASE_11_NOCIONES_DE_REDES.md)**.

---

## Objetivo final

Al terminar deberías poder explicar el recorrido:

```text
navegador / curl
        ↓
HTTP
        ↓
TCP puerto 80
        ↓
IP
        ↓
interfaz de red
        ↓
Apache
        ↓
archivo HTML
```

---

## Importante sobre el entorno

Podés realizar el TP en:

1. Linux nativo.
2. Ubuntu sobre WSL2.

Necesitarás una cuenta con permiso para utilizar `sudo` durante la instalación y
administración de Apache.

Los comandos que modifican `/var/www` y el estado del servicio deben ejecutarse
únicamente dentro del entorno de práctica.

---

## Parte 1 — Observar la red antes de Apache

Ejecutá:

```bash
ip -br link
ip -br addr
ip route
```

Respondé:

1. ¿Qué interfaces aparecen?
2. ¿Cuál es loopback?
3. ¿Qué direcciones IP tiene Linux?
4. ¿Cuál es la ruta por defecto?
5. Si utilizás WSL2, ¿por qué las interfaces pueden diferir de las que muestra Windows?

---

## Parte 2 — DNS y conectividad

Probá:

```bash
getent hosts example.com
```

Luego:

```bash
ping -c 4 example.com
```

Respondé:

1. ¿Qué resuelve DNS?
2. ¿Qué protocolo utiliza `ping`?
3. ¿Por qué un ping fallido no demuestra necesariamente que un servidor web esté caído?

---

## Parte 3 — Puertos antes de Apache

Ejecutá:

```bash
ss -tuln
```

Buscá si aparece:

```text
:80
```

Respondé:

> ¿Qué significa que un puerto TCP esté en estado de escucha?

---

## Parte 4 — Instalar Apache

Actualizá el índice de paquetes:

```bash
sudo apt update
```

Instalá Apache:

```bash
sudo apt install apache2
```

Verificá:

```bash
apache2 -v
```

Registrá en `RESPUESTAS.md` la versión instalada.

---

## Parte 5 — Iniciar Apache

En sistemas con `systemd` habilitado:

```bash
sudo systemctl start apache2
```

Comprobar:

```bash
systemctl status apache2 --no-pager
```

Si tu entorno WSL2 no utiliza `systemd` para administrar el servicio, probá:

```bash
sudo service apache2 start
```

y:

```bash
sudo service apache2 status
```

> El método disponible depende del entorno. No es necesario que todos los estudiantes
> obtengan exactamente la misma salida.

---

## Parte 6 — Verificar el puerto

Ejecutá:

```bash
ss -ltn
```

Buscá:

```text
:80
```

Si tus permisos lo permiten:

```bash
ss -ltnp
```

Respondé:

1. ¿Qué cambió respecto de la observación anterior?
2. ¿Qué proceso o servicio está asociado al puerto?
3. ¿En qué capa del modelo trabajamos cuando hablamos de puertos?

---

## Parte 7 — Apache como servidor HTTP

Probá:

```bash
curl http://localhost
```

Solo cabeceras:

```bash
curl -I http://localhost
```

Más detalle:

```bash
curl -v http://localhost
```

Respondé:

1. ¿Quién es el cliente?
2. ¿Quién es el servidor?
3. ¿Qué protocolo de aplicación se utiliza?
4. ¿Qué puerto se utiliza por defecto?
5. ¿Qué código de estado HTTP obtenés?

---

## Parte 8 — Localizar la web

La configuración habitual de Apache en Ubuntu utiliza como raíz inicial:

```text
/var/www/html
```

Observá:

```bash
ls -la /var/www/html
```

Y:

```bash
stat /var/www/html
```

Respondé:

> ¿Cómo se relacionan ahora sistemas de archivos, permisos y servidor web?

---

## Parte 9 — No editar directamente como root

Para trabajar como programadores, crearemos la web en nuestro `HOME` y luego la
publicaremos.

Prepará:

```bash
mkdir -p ~/aso/tp11/web
cd ~/aso/tp11/web
code .
```

Creá:

```text
index.html
```

Contenido mínimo:

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Arquitectura y Sistemas Operativos</title>
</head>
<body>
    <h1>Mi servidor Apache está funcionando</h1>
    <p>Trabajo Práctico 11</p>
</body>
</html>
```

Abrí el archivo localmente desde VS Code para revisarlo.

---

## Parte 10 — Modificar la web

Personalizá `index.html`.

Debe contener, como mínimo:

- un título;
- nombre de la materia;
- una explicación breve de qué es un servidor web;
- una lista con:
  - dirección IP observada;
  - puerto;
  - protocolo;
- una sección titulada `Diagnóstico del servidor`.

No incluyas contraseñas, datos privados ni información sensible del equipo.

---

## Parte 11 — Publicar la página

Primero conservá una copia de la página original si existe:

```bash
sudo cp /var/www/html/index.html /var/www/html/index.html.original
```

Publicá tu archivo:

```bash
sudo cp ~/aso/tp11/web/index.html /var/www/html/index.html
```

Verificá:

```bash
ls -l /var/www/html/index.html
```

Luego:

```bash
curl http://localhost
```

Y desde el navegador:

```text
http://localhost
```

---

## Parte 12 — Modificar y volver a desplegar

Desde VS Code cambiá el HTML:

- agregá un segundo párrafo;
- modificá el título;
- agregá una lista.

Volvé a publicar:

```bash
sudo cp ~/aso/tp11/web/index.html /var/www/html/index.html
```

Recargá el navegador.

Respondé:

> ¿Por qué no fue necesario reiniciar Apache para que cambie un archivo HTML estático?

---

## Parte 13 — Generar HTML desde Bash

Creá:

```text
generar_web.sh
```

El script generará:

```text
estado.html
```

usando información del sistema.

Podés utilizar un **here document**:

```bash
#!/bin/bash

equipo=$(hostname)
kernel=$(uname -r)
fecha=$(date)

cat > estado.html <<EOF
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Estado del servidor</title>
</head>
<body>
    <h1>Estado del servidor</h1>
    <p>Equipo: $equipo</p>
    <p>Kernel: $kernel</p>
    <p>Generado: $fecha</p>
</body>
</html>
EOF

echo "Generado: estado.html"
```

Ejecutá:

```bash
chmod +x generar_web.sh
./generar_web.sh
```

---

## Parte 14 — Publicar la página generada

Copiá:

```bash
sudo cp estado.html /var/www/html/estado.html
```

Solicitala:

```bash
curl http://localhost/estado.html
```

Después abrí:

```text
http://localhost/estado.html
```

Respondé:

> ¿Qué acabamos de automatizar mediante Bash?

---

## Parte 15 — Logs de Apache

Buscá:

```bash
ls -l /var/log/apache2
```

Archivos habituales:

```text
access.log
error.log
```

Después de realizar varias solicitudes:

```bash
curl http://localhost
curl http://localhost/estado.html
```

observá:

```bash
sudo tail -n 10 /var/log/apache2/access.log
```

Y:

```bash
sudo tail -n 10 /var/log/apache2/error.log
```

Respondé:

1. ¿Qué registra `access.log`?
2. ¿Qué registra `error.log`?
3. ¿Por qué los logs son importantes para un programador?

---

## Parte 16 — Seguir un log en vivo

En una terminal:

```bash
sudo tail -f /var/log/apache2/access.log
```

En otra:

```bash
curl http://localhost
curl http://localhost/estado.html
```

Volvé a la primera terminal.

Terminá `tail -f` con:

```text
Ctrl+C
```

Respondé:

> ¿Qué relación existe entre una solicitud HTTP y la línea que aparece en el log?

---

## Parte 17 — Analizar logs con pipes

Probá:

```bash
sudo grep 'GET' /var/log/apache2/access.log | tail
```

Contá solicitudes:

```bash
sudo grep 'GET' /var/log/apache2/access.log | wc -l
```

Buscá solicitudes a `estado.html`:

```bash
sudo grep 'estado.html' /var/log/apache2/access.log
```

Esto integra contenidos de Bash de toda la cursada:

```text
archivo
→ grep
→ pipe
→ filtro
→ conteo
```

---

## Parte 18 — Script de diagnóstico web

Creá:

```text
diagnostico_web.sh
```

Debe mostrar:

```text
=== SERVIDOR WEB ===

Hostname:
IP:
Ruta por defecto:
Apache instalado:
Puerto 80 escuchando:
Respuesta HTTP:
Documento principal:
Cantidad de GET registradas:
```

Utilizá herramientas como:

```text
hostname
ip
command -v
ss
curl
grep
wc
```

El script no debe cambiar la configuración del sistema.

Solo diagnosticar.

---

## Parte 19 — WSL2 y Windows

Si trabajás sobre WSL2, probá desde el navegador de Windows:

```text
http://localhost
```

Si no funciona en tu configuración concreta:

- verificá primero que `curl http://localhost` funcione dentro de Ubuntu;
- verificá que Apache esté escuchando;
- registrá la diferencia en `RESPUESTAS.md`.

No modifiques firewall, puertos ni configuración de red fuera de las indicaciones del
docente.

---

## Parte 20 — Relacionar con las capas

Completá:

| Acción                             | Capa / concepto principal |
| ---------------------------------- | ------------------------- |
| editar `index.html`                |                           |
| `curl http://localhost`            |                           |
| puerto 80                          |                           |
| dirección IP                       |                           |
| `ip link`                          |                           |
| archivo `/var/www/html/index.html` |                           |
| proceso Apache                     |                           |
| `access.log`                       |                           |

Después representá:

```text
Cliente
↓
HTTP
↓
TCP
↓
IP
↓
Interfaz
↓
Servidor Apache
↓
Sistema de archivos
↓
index.html
```

---

## Parte 21 — Restauración y cierre

Al finalizar el laboratorio, si el docente lo indica, podés restaurar la página original:

```bash
sudo cp /var/www/html/index.html.original /var/www/html/index.html
```

Podés detener Apache:

Con `systemd`:

```bash
sudo systemctl stop apache2
```

o, según el entorno:

```bash
sudo service apache2 stop
```

Verificá:

```bash
ss -ltn
```

---

## Desafío final

Construí una pequeña página para la materia compuesta por:

```text
index.html
estado.html
```

`index.html` debe enlazar a:

```text
/estado.html
```

`estado.html` debe ser generado por `generar_web.sh`.

El flujo esperado es:

```text
editar código en VS Code
        ↓
ejecutar script Bash
        ↓
generar HTML
        ↓
publicar en Apache
        ↓
verificar puerto
        ↓
curl
        ↓
navegador
        ↓
analizar access.log
```

En `RESPUESTAS.md` escribí una explicación final:

> **¿Qué componentes de Arquitectura y Sistemas Operativos intervienen desde que
> escribimos una URL hasta que Apache devuelve el archivo HTML?**

Intentá relacionar al menos:

- CPU;
- proceso;
- planificación;
- memoria;
- sistema de archivos;
- entrada/salida;
- interfaz de red;
- IP;
- TCP;
- puerto;
- HTTP.

---

## Entrega

```text
TP_11/
├── RESPUESTAS.md
├── web/
│   └── index.html
├── generar_web.sh
├── estado.html
├── diagnostico_web.sh
└── evidencia/
    ├── curl.txt
    ├── puertos.txt
    └── access_log.txt
```

Podés generar las evidencias, por ejemplo:

```bash
curl -I http://localhost > evidencia/curl.txt
ss -ltn > evidencia/puertos.txt
sudo tail -n 20 /var/log/apache2/access.log > evidencia/access_log.txt
```

---

## Criterios de revisión

Se evaluará:

- comprensión de interfaces, IP, rutas y puertos;
- diferenciación entre Internet y Web;
- funcionamiento del modelo cliente-servidor;
- instalación y ejecución de Apache;
- edición y publicación de HTML;
- uso de `curl` y `ss`;
- análisis de logs;
- generación de HTML mediante Bash;
- integración de redes, procesos y sistema de archivos;
- capacidad de diagnosticar antes de modificar.

> El cierre busca mostrar algo muy cercano al trabajo real:
>
> **un programador no solo escribe código; también necesita comprender el sistema donde
> ese código se ejecuta, se publica, se comunica y falla.**
