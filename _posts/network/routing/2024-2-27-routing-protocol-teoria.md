---
title: Protocolo de enrutamiento - teoría
date: 2024-02-27
category: teoría
tags: [network, layer 3, router, unedited]
published: true
---
El reenvío de paquetes por parte del router se logra a través del aprendizaje de rutas y el funcionamiento de los switches en la LAN. El aprendizaje de rutas es el proceso de un router que usa para determinar un camino que se usara para reenviar paquetes. Para determinar el mejor camino, el router busca en su tabla de enrutamiento la direccion IP de red que matchee con la direccion IP del paquete de destino.

La busqueda del router en la tabla de enrutamiento da como resultado tres caminos:
- Red directamente conectada: Si el destino de la direccion IP pertenece a un dispositivo que esta en una red que esta directamente conectado a una de las interfaces del mismo router, ese paquete es reenviado directamente a ese dispositivo.
- Red remota: Si el destino de la direccion IP pertenece a una red remota, el paquete es reenviado a otro router. Las redes remotas unicamente son alcanzadas por el reenvio del paquete de otro router.
- Ruta no determinada: Si el destino de la direccion IP del paquete no pertenece a una red directamente conectada o una red remota y el router no tiene configurada una ruta por defecto, el paquete es descartada. El router envia un Internet Control Message Protocol (ICMP) Unreachable mesaage a la direccion IP de origen del paquete.

Los routers agregan IP routes a su tabla de enrutamiento usando tres metodos. IP routers conectados directamente, IP routers estaticos, e IP routers aprendidos de forma dinamica.
- IP routers conectadas directamente son redes directamente conectadas a la interfaz del mismo router.
- IP routers estaticos son redes remotas configuradas manualmente.
- IP routers dinamicos son redes remotas aprendidas mediante un protocolo de enrutamiento.

![Routing methods](/images/roupro/routing_methods.png)
_Enrutamiento dinamico vs estatico_

## Protocolo de enrutamiento
Un protocolo de enrutamiento (Routing protocol) es un conjunto de mensajes, reglas, y algoritmos usados para aprender rutas. Este proceso incluye el intercambio y analisis  de la informacion del enrutamiento. Cada router escoge la mejor ruta de cada subred y finalmente coloca estas mejores rutas IP en su tabla de enrutamiento. Ejemplo de estos protocolos son, RIP, EIGRP, IS-IS, OSPF y BGP.
Por otro lado un protocolo enrutado y protocolo enrutable (Routed protocol & routable protocol) se refieren a un protocolo que define una estructura de paquete y direccion logico, permite al router reenviar o enrutar paquetes. Los routers reenvian paquetes definidos por los protocolos enrutados y enrutables, estos protocolos son los protocolos IPv4 y IPv6.

Incluso cuando el protocolo de enrutamiento es muy diferente al protocolo enrutable, ellos trabajan juntos de manera muy cercana. El proceso de enrutamiento, reenvia paquetes IP, pero si un router no tiene ninguna ruta en su tabla de enrutamiento IP que haga match al paquete de direccion de destino, el router descarta el paquete. El router necesita al protocolo de enrutamiento para poder aprender toda la posible ruta y poder agregarla a su tabla de enrutamiento, asi al final poder realizar el proceso de enrutamiento y reenviar los protocolos enrutables como la IPv4.

Funcion de los protocolos de enrutamiento
- Aprender la informacion del enrutamiento sobre subredes IP de los router vecinos.
- Anunciar la informacion del enrutamiento sobre subredes IP a los router vecinos.
- Si mas de una posible ruta existe para alcanzar una subred, escoge la mejor ruta basada en una metrica.
- Si la topologia de red cambia (por ejemplo, cuando falla un enlace) reacciona anunciando que algunas rutas han fallado y escoge una nueva mejor ruta (este proceso es llamado, convegence).

![routing protocol example](/images/roupro/routing_protocol_exa.png)
_Funcion de un protocolo de enrutamiento_

## Protocolos de enrutamiento interior y exterior
Los protocolos de enrutamiento se dividen en dos categorias: Interior Gateway Protocols (IGP) y Exterior Gateway Protocols (EGP).
- IGP: Un protocolo de enrutamiento que fue diseñado y pensado para su uso dentro de un unico Autonomous System (AS).
- EGP: Un protocolo de enrutamiento que fue diseñado y pensado para el uso entre diferentes Autonomous Systems (AS).

![Routing protocol evolution](/images/roupro/routing_protocol_evolution.png)
_Protocolos de enrutamiento_

### Autonomous System
Un Autonomous System (AS) es una colleccion de routers bajo una administraccion en comun que presenta una politica de enrutamiento en comun y claramente definida hacia internet, por ejemplo la red interna de una empresa grande y la red de una ISP es una AS. Las redes pequeñas de una empresas no son Autonomous System; en la mayoria de los casos, la red de una empresa pequeña forma parte de una red de un Autonomous System de una ISP.
Por diseño algunos protocolos de enrutamiento trabajan mejor dentro de un AS, estos protocolos de enrutamiento son llamados IGP. En cambio, los protocolos de enrutamiento diseñados para intercambiar rutas entre routers de diferentes Autonomous Systems son llamados EGP. Actualmente, Border Gateway Protocol (BGP) es el unicos EGP usado.

Cada AS puede serle asignado un AS number (ASN). Como las direcciones IP publicas, IANA controla la asignacion de las ASNs. Esta delega la autoridad a otra organizacion alrededor del mundo, la misma organizacion  que asigna direcciones IP publicas (LACNIC en latinoamerica) para que tambien asigne ASNs.

![Autonomous system](/images/roupro/as.png)
_Autonomous System_

## IGP
Las organizaciones tienen varias opciones al elegir un IGP para su red empresarial, pero mayormente las empresas usan OSPF o EIGRP.

### Algoritmo del protocolo de enrutamiento IGP
El termino de Algoritmo del protocolo de enrutamiento simplemente se refiere a la logica y el proceso usado por diferentes protocolos de enrutamiento para resolver el problema de aprender todas las rutas, escoger la mejor ruta para cada subred y reaccionar a los cambios de la red. Existen tres principales ramas para el algoritmo de protocolo de enrutamiento para los protocolos de enrutamiento IGP.
- Vector Distancia (Distance vector)
- Vector Distancia Avanzado (Advanced distance vector)
- Estado de enlace (Link-state)
Historicamente hablando, el protocolo vector distancia fue inventado primero a principios de 1980. RIP (Routing Information Protocol) fue el primero en usar el protocolo IP vector distancia, siendo introducido un poco mas tarde IGRP propietaria de Cisco (Interior Gateway Routing Protocol).
A principios de 1990, el protocolo vector distancia algo lenta en el cambio de rutas y con potencial de bucles de enrutamiento impulsaron el desarrollo de protocolos de enrutamientos alternos que utilizaran nuevos algoritmos. En particular los protocos Link-state, OSPF (Open Shortest Path First) y IS-IS (Integrated Intermediate System to Intermediate System) resolvieron los problemas principales. Vinieron con un precio, estos requieren CPU y memoria extra en los routers, con mas planificacion requeridos por los ingenieros de redes.

Alrededor del tiempo que se introdujo OSPF, Cisco creo un protocolo de enrutamiento propietario llamado EIGRP (Enhanced Interior Gateway Routing Protocol), el cual usa algunas caracteristicas del protocolo IGRP. EIGRP resuelve el mismo problema como lo hace los protocolos de enrutamiento Link-state, pero EIGRP requiere menos planeamiento cuando se implementa la red. EIGRP es clasificado como un protocolo de enrutamiento unico, sin embargo este usa mas carateristicas de vector distancia que link-state, asi que es mas comunmente clasificado como protocolo de enrutamiento vector distancia avanzado.

![IGP metrics](/images/roupro/igp_metric.png)
_Metricas de IGPs_

![Metric compared](/images/roupro/metric_compared.png)
_Ejemplo de metrica RIP y OSPF_

#### Protocolos de enrutamiento Vector distancia
Vector distancia quiere decir que el router anuncia como vector la distancia y direccion. Distancia es definido en terminos de metrica como numero de saltos, y direccion es el router del siguiente salto o interfaces. Protocolos de Vector distancia tipicamente usa el algortimo Bellman-Ford para determinar el mejor camino.

Algunos protocolos vector distancia periodicamente envia todas las tablas de enrutamineto a todos sus vecinos. En una red grande, que puede tener tablas de enrutamientos enormes puede causar traficos  signicativos en los enlaces.

Aunque el algoritmo Bellman-Ford eventualmente acumula suficiente conocimiento para mantener una base de dato de redes accesibles, el algoritmo no permite a un router conocer la topologia exacta de una red. El router unicamente conoce la informacion del enrutamiento recibida a traves de los vecinos.

Los protocolos Vector distancia usa a los router como señal a lo largo del camino hacia el destino final. La unica informacion que el router sabe sobre las rutas remotas es la distancia o la metrica para alcanzar la red y cual camino o interfaces son usados para llegar ahi. Un protocolo de enrutamiento vector distancia no tiene un mapa de la topologia de red.

Los protocolos vector distancia trabajan mejor en las siguientes situaciones.
- Cuando la red es simple y plana, y no requiere diseño jerarquico
- CUando el administrador de red no tiene el conocimiento suficiente para configurar y arrglar problemas de los protocolos link-state
- Cuando se esta utilizando tipos de red especificos, como una implementacion de red hub-and-spoke.
- Cuando el tiempo para  los cambios de camino en una red no son una inconveniencia.

#### Protocolo de enrutamiento Link-state
En contraste al protocolo de vector distancia, el protocolo de enrutamiento link-state puede crear un vista completa o topologia de una red mediante la recopilacion de informacion de todos los demas enrutadores. Piense en que un protocolo de link-state tiene un mapa completo de la topologia de red. Las señales a los largo del camino de origen a destino no son necesarias porque todos los router link-state usan un mapa identico a la red. Un router link-state usa la informacion link-state para crear una topologia y seleccionar el mejor camino para cada destino de red.

En algunos protocolos de enrutamiento vector distancia los routers envian informacion periodicamente actualizando sus enrutamientos en los vecinos. Los protocolos de enrutamiento no se actualizan periodicamente. Despues de que la red ha pasado la etapa de convergence, link-state unicammente actualiza cuando la topologia cambia.

Los protocolos link-state trabajan mejor en estas situaciones:
- Cuando el diseño de red es herarquico, el cual es un caso tipico en una red grande.
- Cuando el administrador tiene buenos conocimientos para implementar un protocolo de enrutamiento de link-state
- Cuando el tiempo para los cambios de camino en una red es crucial

#### Metrica
Los protocolos de enrutamiento escogen la mejor ruta para alcanzar una subred, la mejor ruta es la ruta con la metrica mas baja. Por ejemplo, RIP cuenta los numeros de routers (hops) entre un router y la subred destino. OSPF asocia el costo total de cada interfaz de un extremo a otro del router, con el costo basado en ancho de bando del enlace.

#### Distancia Administrativa
La distancia administrativa (Administrative Distance) define la preferencia de la funte de la ruta. Cada fuente de enrutamiento - incluyendo protocolos de enrutamiento, rutas estaticas e incluso redes directamente conectada - esta preferiferido en orden de la mas preferible a la menos preferible, usando un valor AD. Cada router usa las caracteristicas AD para seleccionar el mejor camindo cuando se aprende el mismo camino de destino de red de dos o mas protocolos de enrutamiento.

El valor AD es un a valor entero de 0 a 255. El valor menor es el origen de ruta mas preferida. Una distancia administrativa de o es la mas preferida. Solamente las redes directamentes conectadas tienen un AD de 0, el cual no cambia. Una distancia administratica de 255 quiere decir que el router no le hara caso al origen de ruta y no la agregara a la tabla de enrutamiento.

![Administrative distance](/images/roupro/administrative_distance.png)
_Distancia administrativa_

#### Protocolo de enrutamiento Classful
Los protocolos de enrutamiento Classful no envian mascara de subred en las actualizaciones del enrutamiento. Los primeros protocolos dde enrutaminento, como RIP, son classful. Cuando estos protocolos fueron creados, las direcciines de red fueron asignadas segun la clase (Clase A, B, C). El protocolo de enrutamiento no requeria incluir la mascara de red en las actualizaciones del enrutamiento , porque la mascara de red era determinada basandose en los primeros octetos de la direccion de red.

Los protocolos de enrutamiento classful aun se siguen usando hoy en dia, pero como ellos no incluyen la mascara de red, no se pueden utilizar en todas las situaciones. Los protocolos de enrutamiento classful no pueden ser usadas cuando la red son subredes que usan mas de una mascara de red. En otras palabras, el protocolo de enrutamiento classful no soporta Variable-Lengh Subnet Mask (VLSM).

Otra limitacion de los protocolos de enrutamiento classful incluye la imposibilidad de soportar redes discontinuas y superredes. Los protocolos de enrutamineto que son classful son los protocolos RIPv1 e IGRP

#### Protocolo de enrutamiento Classless
Los protocolos de enrutamiento Classless incluyen mascaras de subred con la direccion de red en actualizaciones de enrutamiento. Las redes actuales ya no se asignan segun su clase, y las mascaras de red no pueden ser determinadas segun el valor del primer octeto. Los protocolos de enrutamiento classless son requeridas en mas redes hoy en dia porque estos soportan VLSM y redes discontinuas, asi tambien superredes. Los protocolos de enrutamiento que son classless son los protocolos RIPv2, EIGRP, OSPF, IS-IS y BGP.
