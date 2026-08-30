**Arquitectura y Sistemas Operativos**

# Clase 11 — Nociones de redes

**Tecnicatura Universitaria en Programación**

---

## Objetivos de la clase

Al finalizar esta clase deberías poder:

- diferenciar Internet de la World Wide Web;
- comprender la idea de conmutación de paquetes;
- reconocer el papel histórico de ARPANET, TCP/IP y la Web;
- diferenciar navegador y motor de búsqueda;
- explicar por qué la comunicación se organiza en capas;
- reconocer las siete capas del modelo OSI;
- relacionar OSI con el modelo TCP/IP;
- comprender encapsulación y desencapsulación;
- distinguir NIC, dirección MAC, dirección IP y nombre de dominio;
- comprender el papel de DNS;
- reconocer puertos TCP y UDP;
- observar interfaces, rutas, direcciones, puertos y HTTP desde Linux.

---

## 1. Internet no es la Web

Estas palabras suelen utilizarse como si fueran sinónimos:

```text
Internet
Web
Chrome
Google
```

pero representan cosas diferentes.

---

## 2. Internet

Internet es una **red de redes**.

Está formada por sistemas interconectados capaces de intercambiar información utilizando
una familia común de protocolos.

Incluye:

- redes físicas;
- routers;
- enlaces;
- direcciones;
- protocolos;
- proveedores;
- servidores;
- dispositivos finales.

Podemos imaginar:

```text
Internet = infraestructura de comunicación
```

---

## 3. World Wide Web

La **Web** es uno de los servicios construidos sobre Internet.

Utiliza tecnologías como:

```text
HTTP
HTML
URLs/URIs
navegadores
servidores web
```

Internet también transporta:

- correo electrónico;
- mensajería;
- videojuegos;
- videollamadas;
- SSH;
- transferencia de archivos;
- APIs;
- muchos otros servicios.

Por eso:

```text
Internet ≠ Web
```

---

## 4. Una analogía

Podemos imaginar:

```text
Internet → red de rutas

Web → uno de los servicios que utiliza esas rutas
```

La analogía tiene límites, pero ayuda a separar infraestructura y servicio.

---

## 5. Una breve historia: ARPANET

ARPANET comenzó a operar en 1969 y conectó inicialmente cuatro nodos académicos de
Estados Unidos.

Fue un proyecto de ARPA que buscó explorar nuevas formas de compartir recursos de
computación y construir redes de comunicación mediante conmutación de paquetes.

Una precisión histórica importante:

> Es frecuente afirmar que ARPANET fue creada específicamente para sobrevivir a una
> guerra nuclear. Esa versión es una simplificación muy extendida. La resiliencia de
> redes distribuidas formaba parte del contexto de investigación de la época, pero
> ARPANET surgió principalmente como una red experimental para interconectar centros de
> investigación y compartir recursos.

---

## 6. El primer mensaje

En 1969 se intentó enviar:

```text
LOGIN
```

desde UCLA hacia SRI.

El sistema falló después de transmitir:

```text
LO
```

Ese episodio se convirtió en una anécdota clásica de la historia de Internet.

---

## 7. Conmutación de paquetes

En una red de paquetes, la información se divide en unidades que pueden transportarse
por la red.

Modelo:

```text
mensaje
   ↓
+--------+--------+--------+
| paq. 1 | paq. 2 | paq. 3 |
+--------+--------+--------+
   ↓        ↓        ↓
        red
   ↓        ↓        ↓
       destino
```

No es necesario reservar un circuito físico exclusivo extremo a extremo durante toda la
comunicación.

Esto permite compartir eficientemente la infraestructura entre múltiples comunicaciones.

---

## 8. TCP/IP

Durante los años 70 Robert Kahn y Vinton Cerf desarrollaron ideas fundamentales que
llevaron a TCP/IP.

El 1 de enero de 1983 ARPANET realizó su transición oficial a TCP/IP.

Esta fecha es un hito importante en la evolución de Internet.

---

## 9. IP

IP — **Internet Protocol** — proporciona un mecanismo para:

- direccionar;
- transportar datagramas entre redes;
- permitir que routers los encaminen hacia un destino.

IP no garantiza por sí solo:

- entrega;
- orden;
- ausencia de duplicados.

Es un servicio de datagramas.

---

## 10. TCP

TCP — **Transmission Control Protocol** — construye sobre IP una comunicación orientada a
conexión que ofrece al software un **flujo confiable y ordenado de bytes** mientras la
conexión pueda mantenerse.

Utiliza mecanismos como:

- números de secuencia;
- confirmaciones;
- retransmisiones;
- control de flujo;
- control de congestión.

Esto no significa que una comunicación TCP sea incapaz de fallar.

Ante problemas graves la conexión puede terminar con un error.

---

## 11. UDP

UDP — **User Datagram Protocol** — ofrece datagramas sin establecer la misma clase de
conexión y garantías de TCP.

Tiene:

- menos mecanismos;
- menor overhead protocolario;
- límites claros de mensajes.

No es correcto reducirlo a:

```text
UDP = más rápido
TCP = más lento
```

El rendimiento depende de la aplicación, red y condiciones concretas.

UDP resulta adecuado cuando la aplicación necesita controlar por sí misma aspectos como:

- temporización;
- recuperación;
- orden;
- tolerancia a pérdidas.

---

## 12. La World Wide Web

Tim Berners-Lee desarrolló la Web en CERN a partir de 1989.

Su propuesta integró elementos fundamentales como:

- hipertexto;
- HTML;
- HTTP;
- identificadores de recursos;
- navegador/editor;
- servidor.

En 1991 la Web comenzó a hacerse accesible fuera de CERN.

En 1993 CERN puso el software de la Web a disposición pública sin regalías, una decisión
fundamental para su expansión.

---

## 13. HTML, HTTP y URL

### HTML

Describe la estructura y contenido de documentos web.

### HTTP

Protocolo de aplicación para intercambiar solicitudes y respuestas.

El protocolo también evolucionó:

- **HTTP/1.1**: una solicitud por vez sobre cada conexión TCP;
- **HTTP/2**: multiplexa varias solicitudes sobre una misma conexión TCP;
- **HTTP/3**: no usa TCP, sino **QUIC** (sobre UDP), lo que reduce latencia y evita el
  bloqueo por cabeza de línea. Ya se usa ampliamente en producción: navegadores como
  Chrome y Firefox y los principales CDN lo soportan.

### URL

Una URL identifica la ubicación o forma de acceso a un recurso.

Ejemplo:

```text
https://www.ejemplo.com/documentos/informe
```

---

## 14. Evolución de la Web

Los términos:

```text
Web 1.0
Web 2.0
Web 3.0 / Web3
```

no representan versiones oficiales instalables.

Son etiquetas utilizadas para describir tendencias históricas.

---

## 15. Web 1.0

Suele utilizarse para describir la primera Web pública:

- sitios relativamente estáticos;
- publicación principalmente unidireccional;
- menor participación del usuario;
- HTML sencillo.

Una imagen conceptual:

```text
editor/publicador
       ↓
      Web
       ↓
    lector
```

---

## 16. Web 2.0

Etiqueta popularizada para describir una Web:

- participativa;
- social;
- dinámica;
- con contenido generado por usuarios;
- con aplicaciones web complejas.

Ejemplos:

- redes sociales;
- wikis;
- blogs;
- plataformas colaborativas;
- video;
- aplicaciones web.

---

## 17. Web semántica y Web3

El término “Web 3.0” se utiliza de maneras diferentes.

### Web semántica

Visión asociada históricamente a Tim Berners-Lee y al W3C para describir datos de manera
que puedan relacionarse y procesarse mediante significado formal.

### Web3

Término más reciente asociado a:

- blockchain;
- activos digitales;
- identidad;
- servicios descentralizados.

No son exactamente el mismo concepto.

Tampoco existe una frontera histórica universal en la que “la Web 3.0 comenzó”.

---

## 18. Navegador

Un **navegador** es una aplicación que permite utilizar la Web.

Ejemplos:

```text
Firefox
Chrome
Edge
Safari
Brave
```

Un navegador puede:

- realizar solicitudes HTTP;
- interpretar HTML;
- aplicar CSS;
- ejecutar JavaScript;
- descargar imágenes, fuentes y otros recursos;
- mantener almacenamiento local;
- administrar certificados;
- ofrecer herramientas para desarrolladores.

---

## 19. Buscador

Un motor de búsqueda es un **servicio** que permite localizar información.

Ejemplos:

```text
Google
Bing
DuckDuckGo
```

Podemos usar:

```text
Firefox → Google
Chrome  → DuckDuckGo
Edge    → Bing
```

Entonces:

```text
navegador ≠ buscador
```

Y:

```text
Chrome ≠ Google Search
```

---

## 20. ¿Por qué necesitamos capas?

Enviar información a través de una red involucra muchos problemas:

- aplicación;
- representación de datos;
- confiabilidad;
- direccionamiento;
- elección de ruta;
- acceso al medio;
- señal física.

Resolver todo dentro de una sola enorme interfaz sería difícil.

La estrategia es dividir responsabilidades.

```text
capa superior
     ↓
capa inferior
     ↓
capa inferior
```

Este patrón ya apareció en:

- jerarquía de memoria;
- sistema de E/S;
- sistema de archivos.

---

## 21. Modelo OSI

OSI — **Open Systems Interconnection** — es un modelo de referencia desarrollado en el
ámbito de ISO.

Propone siete capas:

| Capa | Nombre          |
| ---: | --------------- |
|    7 | Aplicación      |
|    6 | Presentación    |
|    5 | Sesión          |
|    4 | Transporte      |
|    3 | Red             |
|    2 | Enlace de datos |
|    1 | Física          |

Es un **modelo conceptual**, no la descripción exacta de la implementación de Internet.

---

## 22. Capa 1 — Física

Se ocupa de la transmisión física de símbolos o bits mediante un medio.

Ejemplos:

- señales eléctricas;
- fibra óptica;
- radio;
- conectores;
- modulación y características físicas.

---

## 23. Capa 2 — Enlace

Se ocupa de comunicación sobre un enlace local.

Conceptos frecuentes:

- tramas;
- Ethernet;
- Wi-Fi;
- direcciones MAC;
- detección de errores de trama;
- acceso al medio.

---

## 24. Capa 3 — Red

Se ocupa de direccionamiento entre redes y encaminamiento.

Ejemplo central:

```text
IP
```

Los routers toman decisiones relacionadas con esta capa.

---

## 25. Capa 4 — Transporte

Proporciona comunicación extremo a extremo entre aplicaciones.

Ejemplos:

```text
TCP
UDP
```

Conceptos:

- puertos;
- multiplexación de aplicaciones;
- confiabilidad, según protocolo;
- control de flujo, según protocolo.

---

## 26. Capa 5 — Sesión

En el modelo OSI representa funciones relacionadas con el establecimiento y
administración de diálogos o sesiones.

En la Internet real muchas de estas funciones aparecen integradas dentro de protocolos y
bibliotecas de aplicación.

---

## 27. Capa 6 — Presentación

En el modelo OSI se ocupa de aspectos de representación.

Ejemplos conceptuales:

- codificación;
- serialización;
- compresión;
- cifrado.

En las arquitecturas de Internet estas funciones no forman necesariamente una capa
independiente visible.

---

## 28. Capa 7 — Aplicación

Es donde aparecen protocolos utilizados directamente por aplicaciones.

Ejemplos:

```text
HTTP
DNS
SMTP
SSH
```

No significa:

```text
"la aplicación completa está dentro del protocolo"
```

sino que estos protocolos ofrecen servicios a nivel de aplicación.

---

## 29. Encapsulación

Cuando una aplicación envía información, cada nivel añade información necesaria para
cumplir su función.

Modelo simplificado:

```text
datos de aplicación
       ↓
segmento / datagrama de transporte
       ↓
paquete IP
       ↓
trama
       ↓
señales
```

En el receptor ocurre el proceso inverso.

---

## 30. Las capas pares no se comunican mágicamente de forma directa

Suele decirse que:

```text
"la capa N habla con la capa N del otro equipo"
```

Esto describe una **relación lógica entre pares**.

Los datos reales:

```text
bajan por la pila local
viajan por el medio
suben por la pila remota
```

La capa 7 del equipo A no posee un cable directo a la capa 7 del equipo B.

---

## 31. Modelo TCP/IP

Para Internet suele utilizarse una representación de cuatro capas:

```text
Aplicación
Transporte
Internet
Acceso a red
```

Algunos libros utilizan cinco:

```text
Aplicación
Transporte
Red
Enlace
Física
```

No hay problema en encontrar ambas representaciones.

Lo importante es reconocer funciones y relaciones.

---

## 32. Relación aproximada OSI / TCP-IP

| OSI          | TCP/IP       |
| ------------ | ------------ |
| Aplicación   | Aplicación   |
| Presentación | Aplicación   |
| Sesión       | Aplicación   |
| Transporte   | Transporte   |
| Red          | Internet     |
| Enlace       | Acceso a red |
| Física       | Acceso a red |

Esta correspondencia es pedagógica, no una equivalencia perfecta protocolo por
protocolo.

---

## 33. NIC — interfaz de red

La NIC o **Network Interface Controller** proporciona una interfaz física/lógica para
conectar el equipo a una red.

Puede ser:

- Ethernet;
- Wi-Fi;
- virtual;
- otras tecnologías.

Para Linux aparece como una **interfaz de red**.

Observar:

```bash
ip link
```

---

## 34. WSL2 y NIC virtual

En Linux nativo pueden aparecer interfaces asociadas directamente al hardware.

En WSL2 es común que Linux vea interfaces **virtuales** presentadas por el entorno de
virtualización.

Por eso:

```bash
ip link
```

en WSL2 no tiene por qué mostrar exactamente la placa física Wi-Fi o Ethernet del equipo
como la ve Windows.

Esta diferencia es otro ejemplo de virtualización.

---

## 35. Dirección MAC

Una dirección MAC identifica una interfaz dentro de tecnologías de enlace como
Ethernet.

Formato habitual:

```text
a4:c3:f0:1b:2e:9d
```

Puede observarse con:

```bash
ip link
```

Una precisión importante:

> no debe enseñarse como un número necesariamente único, inmutable y grabado para
> siempre en la placa.

Las direcciones MAC pueden ser:

- asignadas por fabricante;
- configuradas por software;
- localmente administradas;
- aleatorizadas por privacidad;
- modificadas en algunos sistemas.

---

## 36. La MAC tiene alcance local

Cuando enviamos un paquete IP a un destino lejano, nuestra computadora no necesita
conocer la MAC del servidor remoto.

Necesita la dirección de enlace del **siguiente salto local**, normalmente el router.

Modelo:

```text
PC
 │ trama: MAC router
 ▼
router
 │ nueva trama: otra MAC
 ▼
siguiente enlace
```

Las direcciones de enlace cambian de un salto a otro.

La dirección IP de destino puede mantenerse a través del recorrido, salvo mecanismos
como NAT.

---

## 37. Tabla de vecinos

Linux mantiene información sobre vecinos locales.

```bash
ip neigh
```

Puede mostrar asociaciones entre:

```text
IP local
↔
MAC
```

En IPv4 este proceso se relaciona con ARP.

En IPv6 existen mecanismos de Neighbor Discovery.

---

## 38. Dirección IP

IP es una dirección lógica utilizada para comunicación entre redes.

Existen actualmente dos versiones importantes:

```text
IPv4
IPv6
```

---

## 39. IPv4

Ejemplo:

```text
192.168.1.25
```

Una dirección IPv4 utiliza:

```text
32 bits
```

No alcanza con mirar solamente la dirección para comprender una red.

También debemos conocer su **prefijo**.

Ejemplo:

```text
192.168.1.25/24
```

---

## 40. Redes IPv4 privadas

Rangos privados definidos para redes internas:

```text
10.0.0.0/8
172.16.0.0/12
192.168.0.0/16
```

No se enrutan directamente a través de Internet pública.

Suelen combinarse con mecanismos como NAT para acceder a Internet.

---

## 41. Loopback

IPv4 reserva:

```text
127.0.0.0/8
```

para loopback.

La dirección más conocida:

```text
127.0.0.1
```

Nombre habitual:

```text
localhost
```

IPv6 utiliza:

```text
::1
```

---

## 42. IPv6

IPv6 utiliza direcciones de:

```text
128 bits
```

Ejemplo:

```text
2001:db8::1234
```

El objetivo de esta materia no es administrar subredes IPv6 en profundidad.

Pero una introducción moderna a redes no debería presentar IP como si solamente
existiera IPv4.

---

## 43. Ver direcciones

```bash
ip addr
```

Resumen:

```bash
ip -br addr
```

Ruta:

```bash
ip route
```

En IPv6:

```bash
ip -6 route
```

---

## 44. ¿Cómo sabe el sistema por dónde enviar un paquete?

Linux consulta su tabla de rutas.

```bash
ip route
```

Podemos encontrar:

```text
default via ...
```

La ruta por defecto se utiliza cuando no existe una ruta más específica.

Modelo:

```text
destino
   ↓
tabla de rutas
   ↓
interfaz / gateway
```

---

## 45. Nombre de dominio

Los humanos preferimos:

```text
www.utn.edu.ar
```

en lugar de recordar direcciones IP.

Un nombre de dominio pertenece a un sistema jerárquico de nombres.

No es correcto reducirlo simplemente a:

```text
nombre = alias de una única IP
```

Un nombre puede relacionarse con:

- varias direcciones;
- otros nombres;
- servidores de correo;
- información de servicios;
- otros tipos de registros.

---

## 46. DNS

DNS — **Domain Name System** — es un sistema distribuido de nombres.

Entre otros registros:

```text
A     → IPv4
AAAA  → IPv6
CNAME → alias
MX    → correo
NS    → servidores autoritativos
```

La resolución puede involucrar:

- cachés;
- resolvedores recursivos;
- servidores autoritativos;
- múltiples consultas.

---

## 47. Resolver nombres desde Linux

Una herramienta disponible casi siempre:

```bash
getent hosts example.com
```

Si `host` está instalado:

```bash
host example.com
```

Si `dig` está instalado:

```bash
dig example.com
```

En Ubuntu `host` y `dig` pueden instalarse con:

```bash
sudo apt install dnsutils
```

No es obligatorio instalarlos para comprender DNS.

---

## 48. Puertos

Una misma computadora puede ejecutar varios servicios.

El protocolo de transporte necesita distinguirlos.

Ejemplos habituales:

```text
22   → SSH
53   → DNS
80   → HTTP
443  → HTTPS
```

Los puertos TCP y UDP utilizan números de:

```text
0 a 65535
```

El mismo número puede existir independientemente en TCP y UDP.

---

## 49. Socket

Una conexión o comunicación de red puede identificarse utilizando información como:

```text
protocolo
IP local
puerto local
IP remota
puerto remoto
```

El concepto de socket conecta esta clase con la [Clase 8](CLASE_08_HILOS_CONCURRENCIA_IPC_SINCRONIZACION.md), donde ya apareció como mecanismo
de IPC.

---

## 50. Observar sockets

```bash
ss -tuln
```

Opciones:

```text
-t → TCP
-u → UDP
-l → listening
-n → no resolver nombres
```

Con procesos, cuando los permisos lo permiten:

```bash
ss -tulpn
```

---

## 51. `ping`

```bash
ping -c 4 example.com
```

`ping` utiliza ICMP.

Sirve para comprobar ciertos aspectos de conectividad y latencia.

Pero:

> que un host no responda a ping no demuestra necesariamente que esté caído.

ICMP puede estar filtrado.

Y:

> que ping funcione tampoco garantiza que un servicio web concreto funcione.

---

## 52. `traceroute`

Si está instalado:

```bash
traceroute example.com
```

Puede mostrar una secuencia aproximada de routers recorridos.

En Ubuntu:

```bash
sudo apt install traceroute
```

Los resultados pueden contener:

```text
*
```

porque algunos routers no responden a las sondas.

Eso no significa necesariamente que el tráfico normal se haya detenido allí.

---

## 53. `curl`

Realizar una solicitud:

```bash
curl https://example.com
```

Solo cabeceras cuando el servidor/protocolo lo permite:

```bash
curl -I https://example.com
```

Más detalle:

```bash
curl -v https://example.com
```

`curl -v` permite observar elementos como:

- resolución;
- conexión TCP;
- TLS;
- solicitud HTTP;
- respuesta.

---

## 54. Laboratorio cliente-servidor local

En una terminal:

```bash
mkdir -p ~/aso/redes/web
cd ~/aso/redes/web
```

Crear:

```bash
echo "<h1>Hola desde Linux</h1>" > index.html
```

Levantar un servidor HTTP local. Usamos un módulo que ya viene con el sistema para
servir archivos de una carpeta; nos sirve como servidor de prueba sin instalar nada:

```bash
python3 -m http.server 8000
```

En otra terminal:

```bash
curl http://localhost:8000
```

También:

```bash
ss -ltn
```

Buscá:

```text
8000
```

---

## 55. ¿Qué acabamos de construir?

```text
curl
cliente HTTP
   │
   │ TCP puerto 8000
   ▼
servidor HTTP local
   │
   ▼
index.html
```

Todo ocurre en la misma computadora usando:

```text
loopback
localhost
127.0.0.1 / ::1
```

Esto demuestra que una red no exige necesariamente dos máquinas físicas para estudiar
el modelo cliente-servidor.

---

## 56. Observar las capas en el laboratorio

### Aplicación

```bash
curl http://localhost:8000
```

### Transporte

```bash
ss -ltn
```

### Red

```bash
ip addr
ip route
```

### Enlace

```bash
ip link
ip neigh
```

La capa física es más difícil de observar directamente desde una VM/WSL2 porque el
hardware puede estar virtualizado.

---

## 57. F12 del navegador

Las herramientas de desarrollo de un navegador permiten inspeccionar solicitudes.

En Chrome, Edge o Firefox:

```text
F12
→ Network / Red
→ recargar
```

Podemos observar:

- URL;
- método;
- estado HTTP;
- tiempos;
- tamaño;
- cabeceras;
- recursos relacionados.

Esto muestra actividad de la capa de aplicación.

---

## 58. WSL2 y redes

WSL2 agrega una capa interesante:

```text
aplicación Linux
      ↓
kernel Linux
      ↓
interfaz virtual
      ↓
infraestructura WSL / Windows
      ↓
NIC física
```

Según la versión y configuración de WSL pueden utilizarse distintos mecanismos de
integración de red.

Por eso:

```bash
ip addr
```

dentro de Ubuntu y:

```powershell
ipconfig
```

en Windows pueden mostrar interfaces y direcciones diferentes.

Esto es esperable.

---

## 59. Internet, Web y navegador: recorrido completo

Supongamos que escribimos:

```text
https://example.com
```

Una descripción muy simplificada:

```text
1. navegador interpreta la URL
2. resuelve el nombre mediante DNS
3. determina una dirección IP
4. consulta rutas
5. establece comunicación de transporte
6. en HTTPS se establece TLS
7. envía una solicitud HTTP
8. routers encaminan paquetes
9. servidor responde
10. navegador procesa la respuesta
```

Entre medio participan:

- interfaces;
- tramas;
- direcciones MAC por cada enlace;
- IP;
- TCP u otro transporte;
- cifrado;
- HTTP;
- múltiples procesos y dispositivos.

---

## 60. No existe una única MAC extremo a extremo

En nuestra red local:

```text
PC → router
```

la trama puede utilizar:

```text
MAC PC
MAC router
```

Después el router crea otra trama para su siguiente enlace.

Así sucesivamente.

Por eso:

```text
IP → direccionamiento entre redes
MAC → direccionamiento sobre el enlace local correspondiente
```

es una primera distinción más útil que:

```text
MAC = quién soy
IP  = dónde estoy
```

La analogía puede ayudar, pero no debe tomarse literalmente.

---

## 61. Actividad práctica

Ejecutá:

```bash
ip -br link
ip -br addr
ip route
ip neigh
getent hosts example.com
ping -c 4 example.com
ss -tuln
curl -I https://example.com
```

Respondé:

1. ¿Cuántas interfaces aparecen?
2. ¿Cuál es la interfaz loopback?
3. ¿Qué direcciones IPv4 aparecen?
4. ¿Aparece alguna IPv6?
5. ¿Cuál es la ruta por defecto?
6. ¿Qué dirección utiliza como gateway?
7. ¿Qué vecinos conoce Linux?
8. ¿A qué direcciones resuelve `example.com`?
9. ¿Qué protocolo utiliza `ping`?
10. ¿Qué sockets aparecen escuchando?
11. ¿Qué estado HTTP devuelve `curl`?
12. Si usás WSL2, ¿qué elementos parecen virtualizados?

---

## 62. Mini desafío Bash

Creá:

```text
red_info.sh
```

Contenido:

```bash
#!/bin/bash

echo "=== RED ==="
echo

echo "Interfaces:"
ip -br link

echo
echo "Direcciones:"
ip -br addr

echo
echo "Rutas:"
ip route

echo
echo "Vecinos:"
ip neigh

echo
echo "Sockets escuchando:"
ss -tuln

echo
echo "Resolucion de example.com:"
getent hosts example.com
```

Permisos:

```bash
chmod +x red_info.sh
```

Ejecutar:

```bash
./red_info.sh
```

---

## 63. Ideas para recordar

```text
Internet
    ≠
Web
```

```text
navegador
    ≠
buscador
```

Y:

```text
Aplicación
    ↓
Transporte
    ↓
Red
    ↓
Enlace
    ↓
Física
```

También:

```text
nombre
   ↓ DNS
IP
   ↓ rutas
siguiente salto
   ↓ enlace
MAC local
   ↓
NIC
```

---

## Glosario

**Internet:** red mundial de redes interconectadas mediante protocolos comunes.

**Web:** sistema de recursos hipervinculados accesibles mediante tecnologías web sobre
Internet.

**Conmutación de paquetes:** transmisión de información dividida en paquetes que
comparten la infraestructura de red.

**ARPANET:** red experimental financiada por ARPA y precursora importante de Internet.

**IP:** protocolo de direccionamiento y entrega de datagramas entre redes.

**TCP:** protocolo de transporte orientado a conexión que ofrece un flujo confiable y
ordenado de bytes.

**UDP:** protocolo de transporte basado en datagramas con mecanismos mínimos.

**HTTP:** protocolo de aplicación utilizado por la Web.

**HTML:** lenguaje de marcado para estructurar documentos web.

**URL:** identificador que describe cómo localizar o acceder a un recurso.

**Navegador:** aplicación cliente utilizada para acceder a recursos web.

**Buscador:** servicio que indexa y permite localizar información.

**OSI:** modelo conceptual de siete capas.

**TCP/IP:** familia de protocolos y modelo utilizado para describir Internet.

**Encapsulación:** agregado de información protocolaria al atravesar capas.

**NIC:** interfaz/controladora de red.

**MAC:** dirección utilizada en determinadas tecnologías de enlace.

**IPv4:** versión de IP con direcciones de 32 bits.

**IPv6:** versión de IP con direcciones de 128 bits.

**Gateway:** siguiente salto utilizado para llegar a otras redes.

**DNS:** sistema distribuido de nombres.

**Puerto:** número utilizado por protocolos de transporte para identificar endpoints de
aplicaciones.

**Socket:** extremo de comunicación utilizado por un proceso.

**Loopback:** interfaz lógica utilizada para comunicación con el propio host.

**ICMP:** protocolo utilizado, entre otras funciones, por `ping`.

---

## Desafiate con preguntas de examen

1. ¿Qué diferencia existe entre Internet y la Web?
2. ¿Qué fue ARPANET?
3. ¿Por qué es una simplificación decir que ARPANET fue construida exclusivamente para
   sobrevivir a una guerra nuclear?
4. ¿Qué es conmutación de paquetes?
5. ¿Por qué 1983 es un año importante en la historia de Internet?
6. ¿Qué función general cumple IP?
7. ¿Qué servicio ofrece TCP a una aplicación?
8. ¿Qué diferencias generales existen entre TCP y UDP?
9. ¿Qué aportó Tim Berners-Lee al desarrollo de la Web?
10. ¿Qué son HTML, HTTP y una URL?
11. ¿Qué significan las etiquetas Web 1.0 y Web 2.0?
12. ¿Por qué Web semántica y Web3 no deben tratarse como exactamente lo mismo?
13. ¿Qué diferencia existe entre navegador y buscador?
14. ¿Por qué utilizamos capas para estudiar redes?
15. ¿Cuáles son las siete capas OSI?
16. ¿Qué función general cumple la capa de transporte?
17. ¿Qué función cumple la capa de red?
18. ¿Qué significa encapsulación?
19. ¿Qué quiere decir que capas pares mantienen una relación lógica?
20. ¿Cómo se relacionan OSI y TCP/IP?
21. ¿Qué es una NIC?
22. ¿Por qué una MAC no debe considerarse necesariamente inmutable?
23. ¿Cuál es el alcance de una dirección MAC?
24. ¿Por qué no necesitamos conocer la MAC del servidor remoto?
25. ¿Qué diferencia existe entre IPv4 e IPv6?
26. ¿Cuáles son los rangos IPv4 privados?
27. ¿Qué es loopback?
28. ¿Qué función cumple una tabla de rutas?
29. ¿Qué es DNS?
30. ¿Por qué un nombre de dominio no es simplemente un alias de una única IP?
31. ¿Para qué sirven los puertos?
32. ¿Qué es un socket?
33. ¿Qué observa `ss`?
34. ¿Por qué un `ping` fallido no demuestra necesariamente que un servidor esté caído?
35. ¿Qué permite observar `curl -v`?
36. ¿Cómo podemos levantar un servidor HTTP local para probar una conexión
    cliente-servidor en la misma computadora?
37. ¿Por qué WSL2 puede mostrar interfaces diferentes de Windows?
38. ¿Qué comandos permiten observar las distintas capas desde Linux?

---

## Cierre de la materia

Durante la materia recorrimos una computadora desde el hardware hasta las redes.

```text
Arquitectura
    ↓
CPU
    ↓
memoria
    ↓
sistema operativo
    ↓
procesos e hilos
    ↓
planificación
    ↓
memoria virtual
    ↓
sistemas de archivos
    ↓
entrada/salida
    ↓
redes
```

En todos los temas apareció una misma idea:

> **la computación se construye mediante capas de abstracción.**

El sistema operativo transforma:

```text
CPU       → procesos e hilos
RAM       → espacios virtuales
discos    → archivos
hardware  → interfaces y dispositivos
red       → sockets y protocolos
```

Bash nos permitió observar esas abstracciones desde un sistema real.

La terminal dejó de ser solamente un lugar donde escribir comandos y se convirtió en
una herramienta para mirar cómo funciona el sistema operativo.
