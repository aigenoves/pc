# Resolución

## Ejercicios

1. dd

2. C pero cada proceso debe ocuparse de contar los fallos de un nivel de
   gravedad determinado.

  ```java
  sem mutex = 1; int cant = 0; colaFallos c[N]; array fallos[4] = ([4], 0); array sem[4] = (1 0 0 0);

  process colab [id: 0...3] {
    Fallo actual;
    int Nivel;
    P(sem[id]);
    P(mutex);
    while (cant < N) {
      nivel = c.top(),getNivel();
      if (nivel != id){
        V(sem[nivel]);
        V(mutex);
        P(sem[id]);
      } else {
        actual = c.pop();
        fallos[id] = fallos[id] + 1;
        cant = cant + 1;
        P(mutex);
      }
    }
    for (i = 1 to 4){
      V(sem[i]);
    }

  process colab [id: 0..3] {
    colaFallos cola_local[N]
    Fallo actual;
    P(mutex);
    cola_local[N] = c[N]
    V(mutex);
    for (i = 1 to N){
      actual = cola_local.pop();
      if (actual.getNivel() == id){
        fallos[id] = fallos[id] + 1;
      }
    }
  }
  ```

  Esta es una solucnion con un poco de semantica de colas

  ```java
     sem mutex=1; colaFallos colaF=([N] Fallo); array cantFallos = ([4], 0)

     process Contador[id: 0..3]{
        colaFallos cola_local;
        Fallo fallo;
        P(mutex);
        cola_local = colaF.clone(); // asumo que hace una copia sin apuntar a la original.
        V(mutex);

        while (!cola_local.isEmpty()) {
            fallo = cola_local.pop();
            if (fallo.getErrorLevel()==id) {
                cantFallos[id]++;
            }
        }
     }
  ```

3. Un  sistema  operativo  mantiene  5  instancias  de  un  recurso  almacenadas  en  una  cola. Además, existen P procesos que necesitan usar una instancia del recurso. Para eso, deben sacar la instancia de la cola antes de usarla. Una vez usada, la instancia debe ser encolada nuevamente para su reúso.

  ```java
   ColaRecurso instRecurso =([5] Recurso); // Preguntar como definir esta cola
   sem mutex = 1;
   sem cantProcDisp = 5

   process Proceso[id:0..P]{
    Recurso recurso;
    P(cantProcDisp)
    P(mutex);
       recurso = instRecurso.pop() // S.C.
    V(mutex)

    // SE UTILIZA EL RECURSO

    P(mutex);
       recurso = instRecurso.push() // S.C.
    V(mutex)
    V(cantProcDisp)

   } 
   ```

   ```java
   cola colaR = ([5] Recurso)
   sem mutex = 1

   process Proceso[id:0..P]{
    Recurso actual;
    P(mutex) 
    if (colaR.notEmpty()) {
      actual = colaR.pop() 
      V(Mutex)
         // Processa el recurso 
      P(Mutex) 
         colaR.push(actual)
      V(Mutex) 
    }
    else {
      V(mutex) 
    }
   } 
  ```

4. No es una solución del todo correcta, supongamos que entran 4 procesos de prioridad alta, esto deja a la cantidad de usuarios totales en 2 (6-4), por lo tanto si llegan otros dos procesos de prioridad alta antes de procesos de prioridad baja (que hasta ahora no llego ninguno) y suponiendo que los 4 proceso de prioridad alta estan en ls SC por mucho tiempo, los 2 procesos nuevos de prioridad alta estarian esperando que se haga V(alta) dejando fura de posibilidad el uso de la base de datos por usuarios de prioridad baja.

5. En  una  empresa  de  logística  de  paquetes  existe  una  sala  de contenedores  donde  se preparan las entregas. Cada contenedor puede almacenar un paquete y la sala cuenta con capacidad para N contenedores. Resuelva considerando las siguientes situaciones:  
a) La empresa cuenta con 2 empleados:  un empleado Preparador que se ocupa de preparar  los  paquetes  y  dejarlos  en  los contenedores;  un  empelado  Entregador que  se  ocupa  de  tomar  los  paquetes  de  los  contenedores  y  realizar  la  entregas. Tanto el Preparador como el Entregador trabajan de a un paquete por vez.  
b) Modifique la solución a) para el caso en que haya P empleados Preparadores.  
c) Modifique la solución a) para el caso en que haya E empleados Entregadores.  
d) Modifique la solución a) para el caso en que haya P empleados Preparadores y E empleadores Entregadores.  
