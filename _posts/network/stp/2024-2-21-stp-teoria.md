---
title: STP - teoría
author: Arodi
date: 2024-02-21
category: teoría
tags: [network, layer 2, protocolo, stp]
published: true
---

Una característica clave de una buena construcción de una red de comunicación es la resistencia. Una red resistente es capaz de mantener las fallas de dispositivo o de enlaces a través de la redundancia. La redundancia de una topología puede eliminar un simple punto de falla mediante el uso de múltiples enlaces, múltiples dispositivos o ambos. El STP ayuda a prevenir loops en la redundancia de switches de  una red.

![stp](/images/stp/redundancia.png)
_redundancia de switches_

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
STP (Spanning Tree Protocol) es un estandar del comite IEEE definido como 802.1D. STP coloca ciertos puertos en estado Forwarding que actuan normal (reenvían y reciven tramas). Sin embargo, las interfaces en estado Blocking no procesa ninguna trama, excepto los mensajes STP (y algunos mensages generales). Las interfaces bloqueadas no reenvían tramas, no aprenden direcciones MAC de tramas recibidas y ni procesan tramas. STP crea árboles que asegura que exista únicamente un camino en cada segmento de red en cualquier momento. Si cualquier segmento experimenta una caída de conectividad, STP reconstruye un nuevo árbol para activar el previo inactivo, pero redundante camino.

![STP](/images/stp/stp.png)
_STP_

### ¿Qué es STA?
El proceso usado por STP es llamado spanning-tree algorithm (STA), este escoge una interface que debera colocarse en estado de reenvio (forwarding state). Cualquiera interfaz que no sea colocada en estado Forwarding, el STA colocará esa interfaz en estado de bloqueo (blocking state).

Pasos para colocar un puerto en estado forwarding.
1. Seleccionar un root bridge, el cual es el switch con menor BID. Solo puede existir un root bridge por red. Todos los puertos en el root bridge son puertos Forwarding.
2. Seleccionar un puerto root basado en el costo menor del camino root (root path cost). Cada switch nonroot tendra un solo puerto root, el puerto root es el puerto por el cual el nonroot bridge tiene el mejor camino hacia el root bridge.
3. Seleccionar un puerto Designated para cada enlace. Cada enlace tiene un puerto Designated, es el puerto del switch que tiene menor BID.
4. Los puertos root y puertos designated cambian al estado Forwarding y los otros puertos permanecen en estado Blocking.
   
![STP ports](/images/stp/stp_state.png)
_State ports_

## STP
### Mensajes STP
Los switches intercambian los mensajes de configuracion STP cada 2 segundos, por defecto, usa tramas multicast llamado Bridge Protocol Data Unit (BPDU). Los puertos bloqueados escuchan estos BPDUs para  detectar si el otro lado del enlace esta caido, esto requerira una recalculacion del STP. Una pieza de informacion que incluye en el BPDU es el Bridge ID (BID).

Para entender el proceso de seleccion del STA, se tiene que entender los mensajes enviados entre los switches tan bien como el concepto y el formato usado para identificar la identidad unica de cada switch.

#### BID
El BID es un valor unico de cada switch que contine de 2 bytes del bridge ID y 6 bytes del system ID, el system ID esta basado en la direccion MAC. En pocas palabras el BID consta de 8 bytes. Por defecto el bridge ID es de 32,768, el switch con el BID menor es convertido en el root bridge. Sin embargo, si el valor de la prioridad por defecto no es cambiado. El switch con el MAC de menor valor es convertido en el root.

![BID](/images/stp/bid.png)
_Campos en el BID_

#### BPDU
STP envia mensajes BPDU (Bridge Protocol Data Units), el cual cada switch usa para intercambiar informacion. El BPDU mas comun es el Hello BPDU.

Campos mas importantes del Hello BPDU.
- Root Bridge ID: Es el bridge ID (BID) del switch que envia el Hello BPDU, pensando que el es el switch root.
- Bridge ID del emisor: El bridge ID del switch que envia el Hello BPDU.
- Root Cost del emisor: El costo entre el switch y el actual root.
- Valor del timer en el switch root: Este incluye el Hello timer, MaxAge timer y forward delay timer.

### Seleccion del switch root
La seleccion del switch root es realizada a base del BID en el BPDU, el switch root es el switch con el menor valor de prioridad en el BID. Si ocurre un empate a base de la prioridad del BID, la porcion de la direccion MAC en el BID es usada, convirtiendo asi el switch con el menor valor de la direccion MAC en el switch root.

La seleccion del switch root inicia con todos los switches clamando ser el root, asi colocando su propia BID como root BID en el Hello BPDU. Si el switch escucha un BID mejor (menor BID), deja de enviar su propia BID como root y comienza a enviar el mejor BID. Eventualmente a todos los switches le llegara el mejor BID, asi convirtiendo a un solo switch en el switch root y todos sus puertos en puertos designed que entran en estado forwarding.

### Seleccion del puerto root
La selecion del puerto root inicia despues de que se haya seleccionado el switch root. El switch root envia Hello BPDUs con el valor del root cost en 0 a todos los switches, cuando estos mensajes llegan a un switch, el switch le suma un valor, esto dependiendo del costo de la ruta y estos a su vez reenvian el Hello BPDU con el menor root cost a los demas switches. Para asi al final convertir el puerto con menor root cost en el puerto root y cada puerto root entra en estado forwarding.

![Costo de la ruta root](/images/stp/root_path_cost_ex.png)
_Costo de la ruta root_

Los switches necesitan una manera de desempatar, en caso de que el valor del root cost sean iguales en dos o mas interfaces. El switch aplica 3 maneras de desempatar las interfaces (ruta) empatadas
1. Basada en el menor bridge ID del switch vecino.
2. Basada en el menor priority port del switch vecino.
3. Basada en el menor port number interno del switch vecino.

### Seleccion del Puerto designated de cada enlace
El paso final del STA para construir la topologia STP es elegir el puerto designated de cada enlace. El puerto designated (DP) de cada enlace es el puerto con menor costo Hello en el enlace. El no root switch reenvia un Hello con el valor del root path cost, el puerto del switch con el menor root path cost es seleccionado como puerto designated, si se tiene un empate, se desempata selecionando al puerto del switch que tenga menor BID. Para que asi al final todos los puertos designated entren en estado forwarding y los puertos que no son puertos root o puertos designated quedan en estado blocking.

Nota: Los puertos conectados a un dispositivo final se convierten en puerto designated y entran en el estado forwarding, ya que los dispositivos finales no ejecutan el protocolo STP, asi dejando al puerto del switch convertirse en puerto designated.

![stp port decision](/images/stp/stp_port_decision.png)
_Decisión de puertos_

### Configuración de la topologia STP
Generalmente STP trabaja por defecto en los switches, teniendo un BID por defecto basada en el valor de la prioridad y direccion MAC del hardware del switch. Adicionalmente, la interfaz del switch tiene un costo base por defecto con la velocidad actual configurada en esa interfaz. Si un ingeniero de red quiere cambiar la topologia generalmente debe cambiar estas configuraciones.

Para cambiar el BID por defecto, se debe configurar la prioridad del BID, por defecto la prioridad es de 32,768, si se coloca una prioridad menor a todos, este se convertira en el root bridge.
El costo del puerto tiene valores por defecto para cada interfaz y para cada VLAN. Se puede configurar estos valores, que puede impactar en la calculacion del root path cost. Para favorecer un enlace, se debe colocar un valor bajo o de lo contrario evita usar el enlace, ya que tendra un root path cost alto para alcanzar al root bridge.

![Path cost](/images/stp/path_cost.png)
_Costo de la ruta en interfaces_

## STP vs RSTP
### STP
El switch root en STP envia un nuevo Hello BPDU cada 2 segundos por defecto. Cada nonroot  switch reenvia el Hello en todos los puertos designated, pero unicamente despues de cambiar algun valor en el Hello recibido. (Dando como resultado, enviar un solo Hello por cada enlace.)

Cuando un Hello BPDU se reenvia, cada switch coloca el root cost segun el calculo root cost local. El switch tambien coloca su propio bridge ID en el sender bridge ID, el root bridge ID no es cambiado.

Asumiendo que el Hello BPDU esta configurado para enviarse cada 2 segundos en el root switch, cada switch reenvia los Hello recibidos (modificado) a todos los puerto designated, haciendo asi que todos los switches continuen reciendo Hellos cada 2 segundos.
1. El root crea y envia un Hello BPDU, con un root cost de 0 a travez de todas las interfaces.
2. El nonroot switch recibe el Hello BPDU en su  root port. Despues modifica el valor del sender BID con su propio BID y tambien el root cost, para asi reenviar el Hello a traves de todos los puertos designated.
3. El paso 1 y 2 se repite hasta que algo cambie.

Cuando un switch falla en recibir un Hello, este sabe que podria esta ocurriendo un error en la red. Cada switch se basa en recibir Hello desde el root para saber si el camino hasta el root esta funcionando. Cuando un switch deja de recibir Hellos o recibe un Hello con valores diferentes quiere decir que algo fallo, asi que el switch reacciona e inicia el proceso para cambiar la topologia del STP.

#### STP Convergence
STP convergence es el proceso por el cual los switches colectivamente realizan algun cambio a la topologia LAN. Los switches determinan cual de ellos necesitan cambiar los puerto al estado de bloqueo y cuales al estado de reenvio. Este proceso requiere el uso de tres timers, todos los switch usan el timer colocado en el root switch, el cual el root switch envia periodicamente en los mensajes Hello BPDU.

![STP timer](/images/stp/stp_timer.png)
_STP timers_

Si un switch no recibe un Hello BPDU en el tiempo esperado (configurado en el hello timer), el switch continuara trabajando normal. Sin embargo, si el Hello BPDU no llegan a aparecer en el tiempo del MaxAge, el switch reaciona y toma los pasos para cambiar la topologia. El MaxAge esta configurado por defecto en 20 segundos (10 veces el tiempo del hello timer). Esto quiere decir que un switch duriaria 20 segundos en reaccionar si no le llega un Hello.

Despues de que el tiempo de MaxAge expira, el switch vuelve a realizar la seleccion del STP, basado en el Hello recibido de otros switches. Esto reevalua cual switch deberia ser el el root switch. Si el local switch no es el root, este escoge el puerto root y determina cuales son los puertos designated de cada enlace.

#### Estado de interfaces en el STP
STP usa la idea  de roles y estados. Roles, como los puertos root y los puertos designated, relacionado a como STP analiza la topologia LAN. States, como los estados Forwarding y Blocking, le dice al switch quien envia y recibe tramas. Cuando se realiza el STP convergence, el switch escoge un nuevo rol para el puerto, y el rol del puerto determina el estado.

Los switches que usan STP pueden moverse inmediatamente del estado Forwarding al estado Blocking, pero toma un tiempo extra al cambiar del estado Blocking al estado Forwarding.

Cuando un puerto necesita cambiar del estado Blocking al estado Forwarding, el switch coloca primero esta interfaz en dos estados intermedios temporales. Estos estados STP ayuda a prevenir temporalmente loops.
- Listening: Igual al estado Blocking, la interfaz no reenvia tramas. El switch remueve la antigua tabla MAC y no aprendera ninguna direccion MAC durante este periodo. Estas tablas MAC podrian causar problemas de bucle temporales.
- Learning: Las interfaces en este estado aun no reenvian tramas, pero el switch inicia el aprendizaje de las direcciones MAC de las tramas recibidas en la interfaz.

![stp port state](/images/stp/stp_port_state.png)
_Estados de los puertos STP_

STP cambia la interfaz del estado Blocking a Listening, despues a Learning, para asi al final entrar en estado Forwarding. Para cada estado intermedio, STP usa el tiempo de Forward Delay timer, que por defecto es 15 segundos. Dando como resultado que el cambio del estado Blocking a Forwarding tome 30 segundos. En adicional, el switch podria esperar el MaxAge timer (20 segundos por defecto) antes de iniciar a mover una interfaz del estado Blocking al estado Forwarding.

![stp convergence](/images/stp/stp_convergence.png)
_STP convergence_

### RSTP
RSTP (Rapit Spanning-Tree Protocol) trabaja igual que STP en muchas maneras.
- RSTP y STP usan las mismas reglas para seleccionar y desempatar el switch root.
- Los switches con RSTP y STP seleccionan el puerto root de la misma manera.
- RSTP y STP usan las mismas reglas para seleccionar y desempatar los puertos designados de cada enlace.
- RSTP y STP colocan los puertos en estado Forwarding y Blocking, aunque RSTP al estado Blocking lo llama estado Discarding.

Nota: RSTP y STP son tan iguales que podrian trabajar en la misma red, asi desplegando RSTP en switches que soporten ese protocolo y STP en los que solamente soporten STP.

La diferencia entre RSTP y STP radica en STP Convergence, en STP el cambio de topologia puede tomar mucho tiempo (50 segundos con los valores por defecto), en cambio con RSTP el cambio de topologia puede tomar unos pocos segundos y en la condicion mas lenta puede tomar 10 segundos.
RSTP cambia y agrega al STP una manera de evitar la espera del STP timers, resultando en una transicion rapida del estado Forwarding al estado Discarding (Blocking)  y viceversa. RSTP comparado a STP define varios casos en el que evita al switch esperar que el timer expire.
- RSTP agrega un mecanismo por el cual un switch puede remplazar su puerto raiz sin la espera de que el puerto entre en estado Forwarding (en algunas condiciones).
- RSTP agrega un nuevo mecanismo por el cual puede remplazar su puerto Designated sin la espera de que el puerto entre en estado Forwarding (en algunas condiciones).
- RSTP baja el tiempo de espera para los casos en el que RSTP deba esperar un timer.
