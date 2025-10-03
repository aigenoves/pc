# Resolución

## Ejercicios

1. ddxswdsx
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

4. No es una solución del todo correcta, supongamos que entran 4 procesos de prioridad alta, esto deja a la cantidad de usuarios totales en 2 (6-4), por lo tanto si llegan otros dos procesos de prioridad alta antes de procesos de prioridad baja (que hasta ahora no llego ninguno) y suponiendo que los 4 proceso de prioridad alta estan en ls SC por mucho tiempo, los 2 procesos nuevos de prioridad alta estarian esperando que se haga V(alta) dejando fura de posibilidad el uso de la base de datos por usuarios de prioridad baja.

5. En  una  empresa  de  logística  de  paquetes  existe  una  sala  de contenedores  donde  se preparan las entregas. Cada contenedor puede almacenar un paquete y la sala cuenta con capacidad para N contenedores. Resuelva considerando las siguientes situaciones:  

   a) La empresa cuenta con 2 empleados:  un empleado Preparador que se ocupa de preparar  los  paquetes  y  dejarlos  en  los contenedores;  un  empelado  Entregador que  se  ocupa  de  tomar  los  paquetes  de  los  contenedores  y  realizar  la  entregas. Tanto el Preparador como el Entregador trabajan de a un paquete por vez.  
  
    ```java
    Array contenedores[5] = [(5) 0]; // N=5
    int posLibre = 0;
    int posOcupada = 0;
    sem cantContLibres = 5;
    sem cantContOcupados = 0;

    Process Preparador {
      while(true){
        Paquete paq;
        // prepara paquete
        P(cantContLibres);
          contenedores[posLibre] = paquete;
          posLibre = (posLibre + 1) MOD 5;
        V(cantContOcupados);

      }

    }

    Process Entregador {
      while(true){
        Paquete paq;
        P(cantContOcupados);
          paq = contenedores[posOcupada];
          posOcupada = (posOcupada + 1) MOD 5;
        V(CantContLibres);
        // Entregar Paquete
      }
      
    }

    ```

   b) Modifique la solución a) para el caso en que haya P empleados Preparadores.  

    ```java
    Array contenedores[5] = [(5) 0]; // N=5
    int posLibre = 0;
    int posOcupada = 0;
    sem cantContLibres = 5;
    sem cantContOcupados = 0;
    sem puedeDepositar = 1;

    Process Preparador[id: 0 .. P-1] {
      while(true){
        Paquete paq;
        // prepara paquete
        P(cantContLibres);
        P(puedeDepositar)
          contenedores[posLibre] = paquete;
          posLibre = (posLibre + 1) MOD 5;
        V(puedeDepositar)
        V(cantContOcupados);

      }

    }

    Process Entregador {
      while(true){
        Paquete paq;
        P(cantContOcupados);
          paq = contenedores[posOcupada];
          posOcupada = (posOcupada + 1) MOD 5;
        V(cantContLibres);
        // Entregar Paquete
      }
      
    }

    ```

   c) Modifique la solución a) para el caso en que haya E empleados Entregadores.  

    ```java
    Array contenedores[5] = [(5) 0]; // N=5
    int posLibre = 0;
    int posOcupada = 0;
    sem cantContLibres = 5;
    sem cantContOcupados = 0;
    sem puedeRetirar = 1;

    Process Preparador {
      while(true){
        Paquete paq;
        // prepara paquete
        P(cantContLibres);
          contenedores[posLibre] = paquete;
          posLibre = (posLibre + 1) MOD 5;
        V(cantContOcupados);

      }

    }

    Process Entregador {
      while(true){
        Paquete paq;
        P(cantContOcupados);
        P(puedeRetirar)
          paq = contenedores[posOcupada];
          posOcupada = (posOcupada + 1) MOD 5;
        V(puedeRetirar)
        V(cantContLibres);
        // Entregar Paquete
      }
      
    }

    ```

   d) Modifique la solución a) para el caso en que haya P empleados Preparadores y E empleadores Entregadores.  

    ```java
    Array contenedores[5] = [(5) 0]; // N=5
    int posLibre = 0;
    int posOcupada = 0;
    sem cantContLibres = 5;
    sem cantContOcupados = 0;
    sem puedeDepositar = 1;
    sem puedeRetirar = 1;

    Process Preparador[id: 0 .. P-1] {
      while(true){
        Paquete paq;
        // prepara paquete
        P(cantContLibres);
        P(puedeDepositar)
          contenedores[posLibre] = paquete;
          posLibre = (posLibre + 1) MOD 5;
        V(puedeDepositar)
        V(cantContOcupados);

      }
    }

    Process Entregador {
      while(true){
        Paquete paq;
        P(cantContOcupados);
        P(puedeRetirar)
          paq = contenedores[posOcupada];
          posOcupada = (posOcupada + 1) MOD 5;
        V(puedeRetirar)
        V(cantContLibres);
        // Entregar Paquete
      }
    }
   ```

6. Existen N personas que deben imprimir un trabajo cada una. Resolver cada ítem usando semáforos.

   a) Implemente una solución suponiendo que existe una única impresora compartida por todas las personas, y las mismas la deben usar de a una persona a la vez, sin importar el orden. Existe una función Imprimir(documento) llamada por la persona que simula el uso de la impresora. Sólo se deben usar los procesos que representan a las Personas.

   ```java
   sem impresoraLibre = 1;

   Process Persona(id: 0..N-1){
    Documento documento = Documento()
    P(impresoraLibre)
    Imprimir(documento)
    V(impresoraLibre)
   }
   ```

   b) Modifique la solución de (a) para el caso en que se deba respetar el orden de llegada.

   ```java
   sem impresoraLibre = 1;

   Process Persona(id: 0..N-1){
    Documento documento = Documento()
    P(impresoraLibre)
    Imprimir(documento)
    V(impresoraLibre)
   }
   ```