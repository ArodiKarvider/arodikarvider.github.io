---
title: STP - teoría
date: 2024-02-21
category: teoría
tags: [network, layer 2, protocolo, stp, unedited]
published: true
---
> UNEDITED
{: .prompt-info }

Una característica clave de una buena construcción de una red de comunicación es la resistencia. Una red resistente es capaz de mantener las fallas de dispositivo o de enlaces a través de la redundancia. La redundancia de una topología puede eliminar un simple punto de falla mediante el uso de múltiples enlaces, múltiples dispositivos o ambos. El STP ayuda a prevenir loops en la redundancia de switches de  una red.

![stp](/images/stp/redundancia.png)
_redundancia de switches_

## Introducción
### ¿Cuáles son los problemas que puede tener una red de switches redundantes?
Broadcast storms: Cada switch transmite broadcasts sin cesar, consumiendo significativamente parte de la capacidad de un enlace.

![broadcast storms](/images/stp/broadcast_storms.png)
_Broadcast storms_

Multiple-frame transmission: Se envía múltiples copias de tramas unicast al destino, confundiendo al host y causando errores irrecuperables.

![Multiple-frame transmission](/images/stp/multiple-frame_transmission.png)
_Multiple-frame transmission_

MAC database instability: Se produce inestabilidad en el contenido de la tabla de direcciones MAC, el switch continuamente actualiza la tabla de direcciones MAC, resultando en tramas enviadas a lugares equivocados.

![MAC database instability](/images/stp/mac_database_instability.png)
_MAC database instability_

### ¿Qué es STP?
STP (Spanning Tree Protocol) es un estándar del comite IEEE definido como 802.1D. STP coloca ciertos puertos en estado Forwarding que actuan normal (reenvían y reciven tramas). Sin embargo, las interfaces en estado Blocking no procesa ninguna trama, excepto los mensajes STP (y algunos mensages generales). Las interfaces bloqueadas no reenvían tramas, no aprenden direcciones MAC de tramas recibidas y ni procesan tramas. STP crea árboles que asegura que exista únicamente un camino en cada segmento de red en cualquier momento. Si cualquier segmento experimenta una caída de conectividad, STP reconstruye un nuevo árbol para activar el previo inactivo, pero redundante camino.

![STP](/images/stp/stp.png)
_STP_

### ¿Qué es STA?
El proceso usado por STP es llamado spanning-tree algorithm (STA), este escoge una interface que deberá colocarse en estado de reenvío (forwarding state). Cualquiera interfaz que no sea colocada en estado forwarding, el STA colocará esa interfaz en estado de bloqueo (blocking state).

Pasos para colocar un puerto en estado forwarding.
1. Seleccionar un root bridge, el cual es el switch con menor BID. Sólo puede existir un root bridge por red. Todos los puertos en el root bridge son puertos forwarding.
2. Seleccionar un puerto root basado en el costo menor del camino root (root path cost). Cada switch nonroot tendrá un sólo puerto root, el puerto root es el puerto por el cual el nonroot bridge tiene el mejor camino hacia el root bridge.
3. Seleccionar un puerto designated para cada enlace. Cada enlace tiene un puerto designated, es el puerto del enlace que tiene menor BID.
4. Los puertos root y puertos designated cambian al estado dorwarding y los otros puertos permanecen en el estado blocking.
   
![STP ports](/images/stp/stp_state.png)
_State ports_

## STP
### Mensajes STP
Los switches intercambian los mensajes de configuracion STP cada 2 segundos, por defecto, usa tramas multicast llamado Bridge Protocol Data Unit (BPDU). Los puertos bloqueados escuchan estos BPDUs para  detectar si el otro lado del enlace esta caído, esto requerirá una recalculación del STP. Una pieza de información que incluye en el BPDU es el Bridge ID (BID).

Para entender el proceso de selección del STA, se tiene que entender los mensajes enviados entre los switches tan bien como el concepto y el formato usado para identificar la identidad única de cada switch.

#### BID
El BID es un valor único de cada switch que contine de 2 bytes del bridge ID y 6 bytes del system ID, el system ID esta basado en la direccion MAC. En pocas palabras el BID consta de 8 bytes. Por defecto el bridge ID es de 32,768, el switch con el BID menor es convertido en el root bridge. Sin embargo, si el valor de la prioridad por defecto no es cambiada. El switch con el MAC de menor valor es convertido en el root.

![BID](/images/stp/stp_bid.png)
_Campos en el BID_

#### BPDU
STP envía mensajes BPDU (Bridge Protocol Data Units), el cual cada switch usa para intercambiar información. El BPDU más común es el Hello BPDU.

Campos más importantes del Hello BPDU.
- Root Bridge ID: Es el bridge ID (BID) del switch que envía el Hello BPDU, pensando que él es el switch root.
- Bridge ID del emisor: El bridge ID del switch que envía el Hello BPDU.
- Root Cost del emisor: El costo entre el switch y el actual root.
- Valor del timer en el switch root: Este incluye el Hello timer, MaxAge timer y forward delay timer.

### Selección del switch root
La selección del switch root es realizada a base del BID en el BPDU, el switch root es el switch con el menor valor de prioridad en el BID. Si ocurre un empate a base de la prioridad del BID, la porción de la dirección MAC en el BID es usada, convirtiendo así el switch con el menor valor de la direccion MAC en el switch root.

La selección del switch root inicia con todos los switches clamando ser el root, asi colocando su propia BID como root BID en el Hello BPDU. Si el switch escucha un BID mejor (menor BID), deja de enviar su propia BID como root y comienza a reenviar el mejor BID. Eventualmente a todos los switches le llegara el mejor BID, así convirtiendo a un solo switch en el switch root y todos sus puertos en puertos designated que entran en estado forwarding.

### Selección del puerto root
La selección del puerto root (RP) inicia después de que se haya seleccionado el switch root. El switch root envía Hello BPDUs con el valor del root cost en 0 a todos los switches, cuando estos mensajes llegan a un switch, el switch le suma un valor, esto dependiendo del costo de la ruta y estos a su vez reenvían el Hello BPDU sumado menor a los demás switches. Para así al final convertir el puerto con menor root cost en el puerto root y cada puerto root entra en estado forwarding.

![Costo de la ruta root](/images/stp/root_path_cost_ex.png)
_Costo de la ruta root_

Los switches necesitan una manera de desempatar, en caso de que el valor del root cost sean iguales en dos o más interfaces. El switch aplica 3 maneras de desempatar las interfaces (ruta) empatadas
1. Basada en el menor bridge ID del switch vecino.
2. Basada en el menor priority port del switch vecino.
3. Basada en el menor port number interno del switch vecino.

### Selección del Puerto designated de cada enlace
El paso final del STA para construir la topología STP es elegir el puerto designated de cada enlace. El puerto designated (DP) de cada enlace es el puerto con menor costo Hello en el enlace. El no root switch reenvia un Hello con el valor del root path cost menor, el puerto del switch con el menor root path cost es seleccionado como puerto designated, si se tiene un empate, se desempata selecionando al puerto del switch que tenga menor BID. Para que así al final todos los puertos designated entren en estado forwarding y los puertos que no son puertos root o puertos designated quedan en estado blocking.

Nota: Los puertos conectados a un dispositivo final se convierten en puerto designated y entran en el estado forwarding, ya que los dispositivos finales no ejecutan el protocolo STP, así dejando al puerto del switch convertirse en puerto designated.

![stp port decision](/images/stp/stp_port_decision.png)
_Decisión de puertos_

### Configuración de la topologia STP
Generalmente STP trabaja por defecto en los switches, teniendo un BID por defecto basado en el valor de la prioridad y dirección MAC del hardware del switch. Adicionalmente, la interfaz del switch tiene un costo base por defecto con la velocidad actual configurada en esa interfaz. Si un ingeniero de red quiere cambiar la topología generalmente debe cambiar estas configuraciones.

Para cambiar el BID por defecto, se debe configurar la prioridad del BID, por defecto la prioridad es de 32,768, si se coloca una prioridad menor a todos, este se convertira en el root bridge.
El costo del puerto tiene valores por defecto para cada interfaz o enlace. Se puede configurar estos valores, que puede impactar en la calculacion del root path cost. Para favorecer un enlace, se debe colocar un valor bajo o de lo contrario evita usar el enlace, ya que tendra un root path cost alto para alcanzar al root bridge.

![Path cost](/images/stp/path_cost.png)
_Costo de la ruta en interfaces_
<!-- editando -->
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

RSTP también difiere de STP en algunas otras cosas. Como por ejemplo, en STP el root switch es el único que envia los Hello BPDU y los otros switches sólo actualizan y reenvían estos Hello BPDU. En cambio en RSTP cada switch independientemente si sea root o no genera su propio Hello BPDU, adicionalmente en RSTP se permite consultas entre vecinos, en lugar de esperar que el MaxAge expire. Este tipo de cambio en el protocolo ayuda a que el switch reaccione rápidamente y cambie la topologia RSTP.

#### Estados y procesos
STP y RSTP usan estados de puertos, pero con algunas diferencias. RSTP mantine los estados learning y forwarding, sin embargo RSTP no usa el estado listening definido por STP y por ultimo RSTP renombra el estado blocking al estado discarding, redefiniendo ligeramente su uso.
RSTP usa el estado discarding que define dos estados, los estados disabled y blocking. Blocking puede trabajar fisicamente, pero STP/RSTP no reenvia trafico por esa interfaz para evitar loops, disabled significa que la interfaz estada administrativamente deshabilitado. RSTP combina estos dos estados en un simple estado llamado discarding.

![Comparation port states](/images/stp/comparation_port_states.png)
_Comparacion de los estados de puertos_

RSTP tiene algunos cambios en el proceso y en el contenido del mensaje para aumentar la velocidad del STP convergence. Por ejemplo, STP espera un tiempo (forward delay) en los estados listening y learning para cambiar al estado forwarding. La razon por la que STP tarda, es que a todos los switches se les dice que cronometren el tiempo de descarte de las tablas MAC en el estado listening con los mensajes BPDU cuando la topologia cambia, ya que cuando la topologia cambia, las tablas MAC existentes pueden causar loops. Luego el estado cambia al estado learning para aprender las nuevas entradas MAC, para asi al final entrar al estado forwarding. Remover las entradas de la tabla MAC es buena, pero esto puede causar la necesidad de esperar el tiempo de Forward Delay timer que por defecto es de 15 segundos, dando un total de 30 segundos para alcanzar el estado forwarding.

RSTP convergence es mas rapido, ya que evita depender de temporizadores. Los switches en RSTP se comunican entre si cuando la topologia cambia, estos mensajes le indican a los switches vecinos remover las entradas MAC que podrian causar loops pontenciales sin la neceisdad de esperar un temporizador. Como resultado se tiene mas escenarios en el que un puerto en estado discarding puede pasar inmediatamente a un estado forwarding, sin la necesidad de esperar y sin pasar a usar el estado learning.

#### Rol del puerto Alternate (Root)
![rstp_port_roles.png](/images/stp/rstp_port_roles.png)
_Roles de puerto en RSTP_

Con STP cada nonroot switch elige un puerto Root. RSTP sigue las mismas reglas para escoger el puerto Root y el puerto Designated, pero luego el RSTP toma otro paso para escoger otros posibles puertos root, llamados puertos Alternate, que es el puerto que no es ni Root, ni Designated o Backup.
Para ser un puerto Alternate, ambos, el puerto Root y el puerto Alternate deben recibir Hellos que identifiquen al mismo switch root.
Un puerto Alternate basicamente actua como el segundo mejor puerto Root. Un puerto Alternate puede reemplazar un puerto Root muy rapido sin requerir la espera de estados intermedios de RSTP. Por ejemplo, cuando el puerto root cae o cuando deja de recibir mensajes Hellos, el switch cambia el rol y el estado del puerto anterior de la siguiente maneara.
- El rol del puerto root cambia a un puerto disabled.
- El estado pasa de Forwarding a Discarding (equivalente al estado Blocking en STP)
- El puerto Alternate cambia el rol a puerto root, y al estado Forwaring sin la espera de un timer.

El nuevo puerto root no necesita pasar tiempos en otros estados, como en el estado learning, es movido inmediatamente al estado forwarding.

#### Rol del puerto Backup (Designated)
Puerto Backup es como un nuevo rol en RSTP, mientras que un puerto alternate fue creado para reemplazar de forma rapida a un puerto root, el puerto backup fue creado para  reemplazar de forma rapida a un puerto designated en un mismo dominio de colision (cuando se usa un hub) sin la necesidad de esperar mucho tiempo moverse desde el estado discarding al estado forwarding.

![rstp_backup_port.png](/images/stp/rstp_backup_port.png)
_Puerto Backup en RSTP_

#### Tipos de puertos
RSTP usa algunos terminos para referirse a los diferentes enlaces y puertos conectados. Los puertos de los enlaces conectados entre dos switches son considerados poin-to-point, ya que son conectados unicamente entre dos dispositivos. Sin embargo RSTP clasifica point-to-point en dos categorias. Puertos point-to-point que conectan dos switches y no son el borde de la red, simplemente son llamados point-to-point. Los puertos que estan conectados a dispositivos finales de la red, como una PC o servidor son llamados puertos point-to-point edge o simplemente puerto edge. 
RSTP usa el termino shared para describir los puertos conectados a un hub, el termino compartido viene del hecho de que comparte Ethernet, el hub tambien fuerza a que el puerto conectado en los switches sea half-duplex. RSTP asume que todos los puertos half-duplex sean puertos shared. RSTP convergence es mas lenta cuando es un puerto shared a comparacion a un puerto point-to-point.

![rstp_link_types.png](/images/stp/rstp_link_types.png)
_Tipos de enlace en RSTP_

- Tipo de enlace point-to-point: RSTP mejora el STP convergence en los enlaces full-duplex. RSTP reconoce la perdida del camino al root bridge a traves del puerto root en 6 segundos, basada en el valor de 3 Hello timer de 2 segundos.
- Tipo de enlace edge/PortFast: RSTP mejora el STP convergence en los puerto edge, pasando inmediatamente de blocking a forwarding cuando un dispositivo final es conectado.  
- Tipo de enlace shared: RSTP no hace nada diferente al STP en los enlaces shared. Esto no importa mucho ya que en la actualidad todos los switch son full-duplex.

### Variedades de STP
Se tiene muchas variedades del protocolo STP
- STP: Especificacion original de STP, definido en IEEE 802.1D, previene  topologias con loops en redes con enlaces redundantes.
- PVST+: Per-VLAN Spanning Tree Plus  (PVST+) es una mejora de Cisco que proporciona la separacion de STP para cada instancia de VLAN configurada en una red.
- RSTP: RSPT o IEEE 802.1w, es una evolucion que mejora la velocidad STP convergence. Sin embargo RSTP solo proporciona una unica instacia STP.
- Rapid PVST+: Rapit PVST+ es una mejora de Cisco para RSTP que usa PVST+, proporcionando instancias separadas de 802.1w por VLAN.
- MSTP y MST: Multiple Spanning Tree Protocol (MSTP) es un estandar IEEE inspirado por la anterior implementacion propietaria de Cisco, Multiple Instance STP (MISTP). MSTP mapea multiples VLAN en la misma instancia spanning thee. La implementacion de Cisco para MSTP es Multiple Spanning Tree (MST), el cual proporciona hasta 16 instancias RSTP y combina muchos VLAN de la topologia fisica y logica en una instancia RSTP en comun. 

#### Operacion PVST+
En el entorno PVST+ uno puede configurar los parametros del spanning-tree para que las tramas se reenvien a una sola VLAN. Se puede hacer esto configurando un switch para que se seleccione como root para una VLAN y que otro switch sea seleccionado como root para otra VLAN diferente.

![PVST+](/images/stp/pvstplus.png)

PVST+ tiene las siguientes caracteristicas:
- Se puede configurar PVST+ por VLAN, permitiendo el uso completo de los enlaces redundantes.
- Cada instancia spanning tree por VLAN agrega mas ciclos de uso al CPU de los switches de la red.

##### System ID extendido
PVST+ requiere separar las instancias del spanning tree para cada VLAN. El campo BID en el BPDU debe llevar la informacion del VLAN ID.

![pvstplus_bid.png](/images/stp/pvstplus_bid.png)

Campos del BID en PVST+
- Bridge Priority: Un campo de 4 bit es usado para el bridge priority. Sin embargo los valores aumentan de 4096 de 4096 y no de 1 en 1, ya que usa los primeros 4 bits y no todos los 16 bits. Los 12 bits sobrantes se usa para las VLANs.
- Extended Systen ID: Un campo de 12 bits que es usado para el VID en PVST+.
- MAC Address: Un cambo de 6 bytes con la direccion MAC del switch.

Nota: Rapid PVST+ es simplemente la implementacion de PVST+ en RSTP, asi que tiene la misma caracteristica de RSTP, pero con instancias separadas para cada VLAN.

## Caracteristicas opcionales del STP
### EtherChannel
Una de las mejores formas de bajar el tiempo de STP convergence es evitarlo por completo. EtherChannel es una manera de evitar el STP convergence cuando se produce una falla en el puerto o en el cable.

EtherChannel combina multiples segmentos paralelos (hasta ocho) de igual velocidades entre pares de switches para trabajar al mismo tiempo. Los switches tratan al EtherChannel como una sola interfaz STP, dando como resultado de que no ocurra el STP convergence si por lo menos uno de los enlaces en el EtherChannel sigue funcionando. Si no se tiene configurado EtherChannel a los enlaces paralelos en los switches, STP bloquea todos los enlaces excepto uno.

![EtherChannel](/images/stp/etherchannel.png)

Nota: EtherChannel se puede configurar tanto en un switch como en un router, llamado EtherChannel capa 2 y EtherChannel capa 3 respectivamente.

### PortFast para PVST+
PortFast le permite a un puerto switch pasar del estado blocking al estado forwarding de forma inmediata, sin antes pasar por los estados intermedios (listening y learning). Sin embar, no se debe habilitar en los puertos que se vayan a conectar a un bridge, switch u otro dispositivo STP, habilitar PortFast en los puertos que se vayan a conectar a estos dispositivos puede crear loops que los estados listening y learning intentan evitar.

Es más apropiado habilitar PortFast en las conexiones con dispositivos finales. Si se conecta un dispositivo final a un puerto con PortFast el switch movera el puerto del estado blocking al estado forwarding para el envio de tramas de forma inmediata. Sin PortFast el switch debe esperar la confirmacion de que el puerto es un puerto designated y pasar por los estados temporales, listening y learning, esto en STP.

RSTP incluye PortFast, esto en los tipos de puerto, para ser mas especifico en los puerto point-to-point edge, RSTP convergece es mas rapido en estos puertos, ya que no pasa por el estado learning, el cual es la misma idea que el PortFast de Cisco. Los switches Cisco habilitan los puertos RSTP point-to-point edge mediante PortFast.

### BPDU Guard
STP y RSTP abren diferentes tipos de exposicion de seguridad en la LAN.
- Un atacante podria conectar un switch malicioso a uno de los switches que pertenezca a la red, un switch con la prioridad baja se convertiria en el switch root, así cambiando la topologia. La nueva topologia podria bajar el rendimiento de la red existente.
- Un atacante podría conectase a multiples puertos, a multiples switches, convertise en root y reenviar mucho trafico a la LAN. Sin que personal de la red se de cuenta, el atacante podria usar un analizador de LAN para copiar una gran cantidad de tramas enviadas a traves de la LAN.
- Los usuarios podrian dañar la LAN al comprar y usar un switch economico (uno que no utilice STP/RSTP) como un switch redundante, ya que si no se tiene la funcion STP/RSTP ningun puerto elegiria bloquearse y causaria un loop.

La funcion Cisco BPDU Guard ayuda a defenderse de este tipo de problemas desabilitando el puerto si se recibe un mensaje BPDU en ese puerto. Esta caracteristica es util usarla en puertos que solamente sean de acceso y que nunca vayan a conectarse a otro switch.

Además, la funcion BPDU Guard ayuda a prevenir problemas con PortFast, ya que PortFast deberia ser habilidado solamente en los puertos de acceso que se conectan a dispositivos finales y no a otros switches. Usar BPDU Guard junto con PortFast ayuda a que si se conecta por error otro switch, el puerto con BPDU Guard configurado desahabilite el puerto antes de crear loops.
