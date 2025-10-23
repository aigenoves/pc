# Programación concurrente en memoria distribuida

## Conceptos generales

- Arq. de memoria distribuida &rarr; procesadores (pueden ser multicore) + mem. local + red de comunicaciones + ***mecanismo de comunicación / sincronización*** &rarr;***intercambio de mensajes***
- ***Programa distribuido:*** Programa concurrente comunicado por mensajes. Supone la ejecución sobre una arq. de memoria distribuida (MD), aunque puedan ejecutarse sobre una de memoria compartida (MC) (o híbrida).
- ***Primitibas de pasaje de mensajes:*** interfaz con el sistema de comunicaciones &rarr; semáforos + datos + sincronización.
- Los procesos ***SOLO comparten canales*** (físicos o lógicos).
Variantes para los canales:
  - Mailbox, input port, link.
  - Uni o bidireccionales.
  - Sincrónicos o asincrónicos.

  ### Detalles de los criterios
  
  #### Criterio: Cuantos o quienes pueden enviar o recibir datos por estos canales

  ***Mailbox:*** Canales totalmente compartidos. Todos los procesos conoces estos canales y todos pueden enviar y recibir por este canal.
  ***input port:*** Por estos canales un único proceso puede ser receptor, pero cualquier otro proceso puede comunicarse (enviar) por este tipo de canal. Se tienen en cuenta en app cliente/servidor.
  ***link:*** Canal punto a punto. En este tipo de canal un único proceso puede enviar y un único proceso puede recibir.

  #### Criterio: Sentido de la comunicación

  ***Unidireccionales:***: La comunicación va en un solo sentido, solo se puede enviar.
  ***Bidireccionales:***: La comunicación va en ambos sentidos, se puede enviar y recibir.

  #### Criterio: Sincronicidad

  ***Síncronicos:*** Se necesita, si o si, que ambos procesos llegen al mismo punto para que la comunicación finalmente se haga.
  ***Asíncronicos:*** Si un proceso necesita enviar información a otro proceso lo hace y continua con su trabajo, no le interesa si el otro proceso llego al punto de recepción. Queda guardado el mensaje hasta que el otro proceso llege al punto de la comunicación y toma el dato guardado.

### Características

- Los canales son lo único que corpanten los procesos
  - Variables locales a un proceso "cuidador"
  El proceso es el encargado administrador de esa variable en particular.
  - Le exclusión mutua no requiere un mecanismo especial
  Al ser las variables privadas (no existen variables compartidas) no se necesita un mecanismo que administre el acceso a las mismas.
  - Los procesos intereactúan comunicándose
  la única manera en que los procesos van a interactuar es comunicandose entre ellos, pasandose datos, hasta la sincronización es a través de estas comunicaciones.
  - Accedidos por primitivas de envío y recepción
  Las comunicaciones se realizan a través de primitivas de envío y recepción

- Mecanismos para el Procesamiento Distribuido
  - Pasaje de Mensajes Asíncronicos (PMA)
    - Canales de tipo ***Mailbox***
    - Asincrónicos
    - Comunicación unidireccional
  - Pasaje de Mensajes Síncronicos (PMS)
    - Canales de tipo ***link***
    - Sincrónicos
    - Comunicación unidireccional
  - Llamado a Procedimientos Remotos (RPC) &rarr; Solo teórico
    - Canales de tipo ***link***
    - Sincrónicos
    - Comunicación bidireccional
  - Rendezvous (ADA)
    - Canales de tipo ***link***
    - Sincrónicos
    - Comunicación bidireccional
- La sincronización de la comunicación interproceso depende del patrón de interacción.
  - Productores y consumidores (Filtros o Pipes)
  Vamos a tener procesos, llamados filtros, que lo que hacen es recibir info. de otro proceso la procesasan, modifican o actualizan y devuelven el resultado a otro proceso para continuar con el trabajo.
  - Clientes y servidores
  Hacemos referencia a que tenemos un monton de procesos clientes que no pueden resolver algun procesamiento y necesitan que lo resuelva uno o más servidores. El servidor es un proceso que sabe resolver algún procesamiento. El cliente envia al servidor y el servidor recibe del cleinte, lo procesa, y envía el resultado a ese mismo cliente.
  - Pares que interactúan
  Basicamente son todos procesos con las mismas características, suelen ser procesos idénticos que hacen el mismo trabajo sobre conjuntos de datos distintos, cada uno va a estar procesando lo que tienen que hacer y cada tanto va a interactuar con algun otro proceso para comunicacion de resultados parcial, sincronizarse, esperarse, etc.

### Relación entre mecanismos de sincronización

- ***busy waiting*** &rarr; Primer herramienta vista, es ineficiente, compleja
- ***Semáforos (MC)*** &rarr; Mejora respecto de ***busy waiting***. Reduce la complejidad. Nos permiten resolver los problemas de sincronización por exclusión mutua o por condición
- ***Monitores (MC)*** &rarr; Es una continuación, por el lado de MC, de semáforos que brinda una herramienta de mayor abstracción. Nos permiten resolver, de manera implícita, la exclusión mutua y ,de manera explícita, resolver la exclusión por condición

*Terminar*
- ***PMA*** &rarr; Tiene una forma de sincronzar parecida a la que tienen los semáforos. Un proceso que necesita recibir una información, a través de un cierto canal, se va a quedar esperando hasta que en ese canal haya al menos un mensaje. Es el P de los semáforos
- ***PMS*** &rarr; Es el V de los semáforos
- ***RPC*** &rarr;
- ***Rendezvous*** &rarr;
