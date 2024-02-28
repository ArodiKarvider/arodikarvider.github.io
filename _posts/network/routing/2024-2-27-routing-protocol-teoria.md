---
title: Protocolo de enrutamiento
date: 2024-02-27
category: teoría
tags: [network, layer 3, router, draft]
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

## Protocolo de enrutamiento
Un protocolo de enrutamiento (Routing protocol) es un conjunto de mensajes, reglas, y algoritmos usados para aprender rutas. Este proceso incluye el intercambio y analisis  de la informacion del enrutamiento. Cada router escoge la mejor ruta de cada subred y finalmente coloca estas mejores rutas IP en su tabla de enrutamiento. Ejemplo de estos protocolos son, RIP, EIGRP, IS-IS, OSPF y BGP.
Por otro lado un protocolo enrutado y protocolo enrutable (Routed protocol & routable protocol) se refieren a un protocolo que define una estructura de paquete y direccion logico, permite al router reenviar o enrutar paquetes. Los routers reenvian paquetes definidos por los protocolos enrutados y enrutables, estos protocolos son los protocolos IPv4 y IPv6.

Incluso cuando el protocolo de enrutamiento es muy diferente al protocolo enrutable, ellos trabajan juntos de manera muy cercana. El proceso de enrutamiento, reenvia paquetes IP, pero si un router no tiene ninguna ruta en su tabla de enrutamiento IP que haga match al paquete de direccion de destino, el router descarta el paquete. El router necesita al protocolo de enrutamiento para poder aprender toda la posible ruta y poder agregarla a su tabla de enrutamiento, asi al final poder realizar el proceso de enrutamiento y reenviar los protocolos enrutables como la IPv4.

Funcion de los protocolos de enrutamiento
- Aprender la informacion del enrutamiento sobre subredes IP de los router vecinos.
- Anunciar la informacion del enrutamiento sobre subredes IP a los router vecinos.
- Si mas de una posible ruta existe para alcanzar una subred, escoge la mejor ruta basada en una metrica.
- Si la topologia de red cambia (por ejemplo, cuando falla un enlace) reacciona anunciando que algunas rutas han fallado y escoge una nueva mejor ruta (este proceso es llamado, convegence).

## Protocolos de enrutamiento interior y exterior
Los protocolos de enrutamiento se dividen en dos categorias: Interior Gateway Protocols (IGP) y Exterior Gateway Protocols (EGP).
- IGP: Un protocolo de enrutamiento que fue diseñado y pensado para su uso dentro de un unico Autonomous System (AS).
- EGP: Un protocolo de enrutamiento que fue diseñado y pensado para el uso entre diferentes Autonomous Systems (AS).

### Autonomous System
Un Autonomous System (AS) es una colleccion de routers bajo una administraccion en comun que presenta una politica de enrutamiento en comun y claramente definida hacia internet, por ejemplo la red interna de una empresa grande y la red de una ISP es una AS. Las redes pequeñas de una empresas no son Autonomous System; en la mayoria de los casos, la red de una empresa pequeña forma parte de una red de un Autonomous System de una ISP.
Por diseño algunos protocolos de enrutamiento trabajan mejor dentro de un AS, estos protocolos de enrutamiento son llamados IGP. En cambio, los protocolos de enrutamiento diseñados para intercambiar rutas entre routers de diferentes Autonomous Systems son llamados EGP. Actualmente, Border Gateway Protocol (BGP) es el unicos EGP usado.

Cada AS puede serle asignado un AS number (ASN). Como las direcciones IP publicas, IANA controla la asignacion de las ASNs. Esta delega la autoridad a otra organizacion alrededor del mundo, la misma organizacion  que asigna direcciones IP publicas (LACNIC en latinoamerica) para que tambien asigne ASNs.

## IGP
Las organizaciones tienen varias opciones al elegir un IGP para su red empresarial, pero mayormente las empresas usan OSPF o EIGRP.
