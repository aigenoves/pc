# Pasaje de Mensajes Asincrónicos (PMA)

## Uso de canales en PMA

* ***Canales***  
Son colas de mensajes (**FIFO**) que han sido enviados y aún no han sido recibidos. Un canal sirve para enviar mensajes de un mismo tipo. El acceso a los mismos es ***Atómico***. En principio los canales son ilimitados, aunque en las implementaciones reales tendrán un tamaño de buffer asignado. Los mensajes **NO** se pierden ni modifican y todo mensaje enviado en algún momento puede ser "leído". posee 3 primitivas: send, receive, empty.
Son declarados globales a los procesos ya que pueden ser compartidos. Según la forma en que se usan podría ser:  

  * Cualquier proceso puede enviar o recibir por alguno de los canales declarados. En este caso suelen denominarse ***mailboxes***.
  * En algunos casos un canal tiene un solo receptor y muchos emisores(***input port***).
  * Si el canal tiene un único emisor y un único receptor se lo denomina ***link***: provee un “camino” entre el emisor y sus receptores.
  
* ***Declaración de canales***  
  * **Declaración:** chan ch (id<sub>1</sub>: tipo<sub>1</sub>,...,id<sub>n</sub>: tipo<sub>n</sub>)
  * Ejemplos:
    * **chan entrada(char);**  
    En este ejemplo se envia por el canal caracteres de a 1.
    * **chan acceso_disco (INT cilindro, INT bloque, INT cant, CHAR * biffer);**  
    En este ejemplo donde cada mensaje va a tener un registro con 4 datos, 3 enteros (número de cilindro, número de bloque y cantidad de bytes a leer/escribir) y un 4 elemento que es un string (puntero a un conjunto de caracteres).

    * **chan resultado[n] (INT);**  
    En este ejemplo lo que se declara es un arreglo de canales llamado resultado. Son ***n*** canales resultado donde cada mensaje que va en cada uno de ellos es un entero.

* ***Operación Send***  
Es para que un proceso agregue un mensaje al final de la cola (*"ilimitada"*) de un canal ejecutando *send*, que no bloquea al emisor. Esta operacion es atómica.
  * **send ch(expr<sub>1</sub>,...,expr<sub>n</sub>)**

* ***Operación Receive***  
Un proceso recibe un mensaje desde un canal con *receive*, que demora ("bloquea") al receptor hasta que en el canal haya al menos un mensaje; luego toma el primero y lo almacena en variables locales:  
  * **receive ch(var<sub>1</sub>,...,var<sub>n</sub>)**  

  *Las var del receive deben tener los mismos tipos que la declaración del canal.*

  Receive es una primitiva ***bloqueante***, ya que produce un delay. **Semántica**: el procesp NO hace nada hasta recibir un mensaje en la cola correspondiente al canal. ***NO*** es necesario hacer polling, se duerme no usa el procesador hasta que no pueda sacar algo de la cola.
  
* ***empty(ch)***  
Determina si la cola de un canal está vacía. ütil cuando el proceso puede hacer trabajo productivo mientras espera un mensaje, ***pero debe usarse con cuidado***. Si hay dos procesos que pueden hacer receive por ese canal, si ambos procesos llaman a la función *empty* a los dos les dice que hay mensajes, supongamos uno solo, como a los dos procesos el *empty* les dijo que habia mensajes, ambos procesos intentan hacer el *receive*, solo uno logra recibir el mensaje y el otro se va a quedar en el *receive* (recordar la atomicidad).
Tambien podría ocurrir lo contrario, se chquea el *empty* de un canal, me dice que esta vacio y cuando yo voy a continuar trabajando podría ser que otro proceso ya haya dejado un mensaje.