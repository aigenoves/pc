# Clase 2 

## MC - Variables Compartidas

En memoria compartida hay un problema: El problema de la seccion crítica. Necesitamos resolverlo
Necesitamos algunos algoritmos, complejos y que no dan la soluciones.

Tambien debemos resolver las barreras

Técnicas (Pag 5)
 - Busy Waiting
   - Ventajas ...
   - Ineficiente ...
   - Aceptable ....


Como solucion al problema de la secc crit tenemos que implementar un protocolo de entrada y un protocolo de salida.

Propiedades de los protocolos de entrada y salida:

1. Exclusión mutua: A lo sumo un proc ejecuntando su SC.
2. Ausencia de deadlocks: Si 2 o más proceso tratan de entrar a sus SC, al menos uno tendrá éxito.
3. Ausencia de demora innecesaria: si n proc trata de entrar a su SC y los otros están en sus SNC o terminaron, el primero no está impidiendo entrar a su SC. Un ejemplo de falla en esta propiedad es cuando queremos que los procesos se ejecuten en orden.  
4. Eventual entrada: un proceso que intenta entrar a su SC tiene posibilidades de hacerlo (eventualmente lo hará).

* Solución trivial < SC >.

Implementación de sentencias *await?* (las sentencias *await* no existen)

### Posibles soluciones a la SC

1. Solución de "grano grueso"
   1. La eventual entrada formalmente no se puede confirmar que se cumpla ya que depende de la politica de scheduling
2. Solución de "grano fino" *Spin Locks*
   1. Test & Set (Atómica)
   2. No se puede comprobar la **Eventual Entrada**
   3. Los procesos se quedan iterando (spinning) mientras esperan qie se limpie *lock*.
3. Solución Fair
   1. Ordena el orden en el que los procesos entran a sus SC.
      1. Alg. Tie-Breaker: Bien para 2 procesos, complejo para *n* procesos.
         1. En la solución de *"Grano Fino"* recordar que siempre asignar true a mi variable y después a último.
      2. Alg. Ticket: Se reparten números y se espera a que sea el turno.
         1. Solución de *"Grano Grueso"*
         2. Solución de *"Grano Fino"*.:  
            Utilizar **Fetch & Add**.
      3. Alg. Bakery

### Sincronizacion ***Barrier***

1. Contador Compartido
   1. Solo una iteracion. No podemos reiniciar a 0 la cantiad porque no sabemos que procesos lo vieron o no.
   2. Si se va a utilizar una sola vez es la barrera más sencilla.
2. Flags y Coordinadores
3. Barreras Simétrica
   1. Permite ejecutar varias etapas de la barrera para sincronizar con otro proceso.
   2. Ejemplo concreto: *Butterfly Barrier*
      1. Deberia ser más rapido mientras mas procesos hay.
      2. log<sub>2</sub>n etapas

### Defectos de la sincronización por Busy Waiting
*[completar]*