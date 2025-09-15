# Notas de clase

## Defectos de la sincronización por *Busy Waiting*

* Protocolos "Busy-waiting":
  * Complejos
  * Sin clara Separación entre variables de sincronización y las usadas para computar resultados
* Diícil de diseñar para probar corrección
  * verificación compleja al incrementar el número de procesos
* Técnica ineficiente en multiprogramación
  * Un proceso ***spinning*** reduce la productividad del procesador

## Semáforos

Fueron descriptos por Dijkstra en 1968. Permiten realizar sincronización entre procesos concurrentes.

Es una instancia de un tipo de datos abstracto (o un objeto) con sólo 2 operaciones (métodos) ***atómicas***: P y V.

Internamente el valor de un semáforo es un entero *no negativo* ($\geq0$):

* V: Señala la **ocurrencia de un evento** (incrementa).
* P: Se usa para **demorar** un proceso **hasta que ocurra un evento** (decrementa).

> Permiten proteger *Secciones Críticas* (SC) y pueden usarse para implementar *Sincronización por Condición* (SxC).

### Operaciones Básicas

* #### Declaraciones

   ```java
   sem s; // NO!. Si o si se deben inicializar en la declaración.
   sem mutex = 1;
   sem fork[5] = ([5] 1)
   ```

* #### Semáforo general (*counting semaphore*)

   Son aquellos donde el valor interno que toma el semaforo es $\geq0$ sin tener una cota superior.

   Son no deterministas, por lo tanto no hay orden en la espera sobre la operacion ***P***, si hubiese muchos procesos que estan esperando en la operacion ***P*** a que el valor interno del semáforo sea $\geq0$ cuando este, a traves de alguna llamada a la operacion ***V*** de otro proceso se incremente en 1 y tome el valor 1, cualquiera de los que estaban esperando puede pasar. Prodia suceder que, si no se tiene en cuenta esta cuestion, no se respete la propiedad de ***Eventual Entrada***.

   Hay herramientas (lenguajes o librerias) que pueden tener implementados los semáforos los cuales implementen esa la espera en colas, en esos casos se garantiza la ***Eventual Entrada***. Pero **¡no los vamos a manejar en la materia!**

   **¡Son los que vamos a usar en la práctica, y en la meteria en general, como herramienta!**

   ```java
   P(s): <await(s>0) s = s-1;>
   V(s): <s = s+1;>
   ```

   **¿Cómo actuan los semáforos generales?**

   La operacion ***P*** va a demorar al proceso que hace el llamado hasta que se indique que ha ocurrido algún evento (semáforo $\geq0$) si el semáforo $=0$ el proceso se tiene que quedar dormido hasta que pueda pasar y ahí, de forma atomica decrementa en 1 al semáforo.

   La operación ***V***, cuando ocurre un evento, incrementa el contador interno del semáforo en 1 de manera atómica. No es bloqueante, solo realiza el incremento.

* #### Semáforo binario

   Tambien limita el valor superior pero pueden tomar 0 $\lor$ 1 como valores. Por lo tanto al hacer la declaración del semáforo binario se lo debe inicializar en 0 o 1.

   **¡Este tipo de semáforos no los vamos a uasr como herramienta en la cátedra!**

   ```java
   P(b): <await(b>0) s = s-1;>
   V(b): <await(b<10) s = s+1;>
   ```

   **¿Cómo actuan los semáforos generales?**

   En este tipo de semáforos lo que cambia es la operación ***V*** que en este caso es *bloqueante* ya que debe esperar a que el valor del semáforo sea $\leq1$ (más precisamente $=0$).

   La operación ***P*** es igual a la del semáforo general.

### Problemas básicos y técnicas

#### Problema de la Sección crítica (PEM): *Exclusión Mutua (EM)*

   Es más simple que cualquiera de los algoritmos con ***busy waiting*** lo cuales utilizan ***variables compartidas***. La complejidad va a estar, en todo caso, en el código de la ***Sección Crítica (SC)***  y en el de la ***Sección no Crítica (SNC)*** y no en la forma de entrar o salir a la **SC**.

   ```java
   sem free=1 //Si o si en 1 porque si no puede entrar ningún proceso.
   process SC[I=1 to n]
   {
       while(true)
       {
           P(free);
           ...SC...;
           V(free);
           ...SNC...;
        }
   }
   ```

#### Barreras: Señalización de Eventos

   La idea de esta técnica de ***Señalizacion de Eventos*** es tener un semáforo para cada *flag* de sincrinización (**inicializados en 0**) luego un proceso setea el *flag* ejecutando la operación ***V***, y espera a que un *flag* sea seteado t luego lo limpia ejecutando la operacion ***P***.

   Barrera para dos procesos: Necesitamos saber cada vez que un proceso llega o parte de la barrera. Hay que relacionar los estados de los procesos.

   Hay que tener cuidado con el orden en el que ejecutamos las operaciones ***V*** y ***P***. Siempre es ***V(llega1); P(llega2);*** y en el otro proceso ***V(llega2); P(llega1);***