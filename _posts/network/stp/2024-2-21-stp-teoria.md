---
title: STP, teoría
author: AK
date: 2024-02-21
category: network
tags: [layer 2, protocolo, stp]
published: true
---

Una característica clave de una buena construcción de una red de comunicación es la resistencia. Una red resistente es capaz de mantener las fallas de dispositivo o de enlaces a través de la redundancia. La redundancia de una topología puede eliminar un simple punto de falla mediante el uso de múltiples enlaces, múltiples dispositivos o ambos. El STP ayuda a prevenir loops en la redundancia de switches de  una red.

![stp](/images/stp/redundancia.png)

## Introducción
### ¿Cuáles son los problemas que puede tener una red de switches redundantes?
Broadcast storms: Cada switch transmite broadcasts sin cesar, consumiendo significativamente parte de la capacidad de un enlace.

![broadcast storms](/images/stp/broadcast_storms.png)
_Broadcast storms_

Multiple-frame transmission: Se envía multiples copias de tramas unicast al destino, confundiendo al host y causando errores irrecuperables.

![Multiple-frame transmission](/images/stp/multiple-frame_transmission.png)
_Multiple-frame transmission_

MAC database instability: Se produce inestabilidad en el contenido de la tabla de direcciones MAC, el switch continuamente actualiza la tabla de direcciones MAC, resultando en tramas enviadas a lugares equivocados.

![MAC database instability](/images/stp/mac_database_instability.png)
_MAC database instability_

### ¿Qué es STP?
STP (Spanning Tree Protocol) es un estandar del comite IEEE definido como 802.1D. STP coloca ciertos puertos en estado Forwarding que actuan normal (reenvían y reciven tramas). Sin embargo, las interfaces en estado Blocking no procesa ninguna trama, excepto los mensajes STP (y algunos mensages generales). Las interfaces bloqueadas no reenvían tramas, no aprenden direcciones MAC de tramas recibidas y ni procesan tramas. STP crea arboles que asegura que exista únicamente un camino en cada segmento de red en cualquier momento. Si cualquier segmento experimenta una caída de conectividad, STP reconstruye un nuevo arbol para activar el previo inactivo, pero redundante camino.

![STP](/images/stp/stp.png)
_STP_

### ¿Qué es STA?

El proceso usado por STP es llamado spanning-tree algorithm (STA), este escoge una interface que debera colocarse en estado de reenvio (forwarding state). Cualquiera interfaz que no sea colocada en estado Forwarding, el STA colocará esa interfaz en estado de bloqueo (blocking state).

Pasos para colocar un puerto en estado forwarding.
1. Seleccionar un root bridge, el cual es el switch con menor BID. Solo puede existir un root bridge por red. Todos los puertos en el root bridge son puertos Forwarding.
2. Seleccionar un puerto root basado en el costo menor del camino root (root path cost). Cada switch nonroot tendra un solo puerto root, el puerto root es el puerto por el cual el nonroot bridge tiene el mejor camino hacia el root bridge.
3. Seleccionar un puerto Designated para cada enlace. Cada enlace tiene un puerto Designated, es el puerto del switch que tiene menor BID.
4. Los root ports y designated ports cambian al estado Forwarding y los otros puertos permanecen en estado Blocking.

![STP ports]({{ site.baseurl }}/images/stp/state_ports.png)
_State ports_