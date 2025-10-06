# Competencia Culinaria

En una competencia culinaria se juntam **1 jurado** y **C concursantes.** Una vez que todos los concursantes llegaron, el jurado les asigna la receta que deben realizar. A continuación, los concursantes cocinan el plato pedido (a cada uno le lleva un tiempo variable) y lo exhiben ante el jurado en el orden en que van terminando. El jurado asigna un puntaje a cada concursante, el cual lo guarda para su CV.

Solución:

```java

sem mutex = 1;
sem mutex_2 = 1;
sem barrera = 0;
sem espera = 0;

sem esperar_receta[C] = ([C] 0); // Semaforo privado.
sem esperar_nota[C] = ([C] 0); // Semaforo privado.
sem presentar_plato = 0; //semáforo de señalización para que el jurado no haga busy waiting.


recetas recetas[C]; //Arreglo de recetas
platos platos[C]; //Arreglo de platos terminados
queue terminaron;
int notas[C];
int cant_concursantes = 0;

process Concursante[id: 0..C]{
    int i;
    int mi_nota;

    P(mutex);
    cant_concursantes = cant_concursantes + 1;
    if (cant_concursantes < C){
        V(mutex)
        P(espera)
    } else {
        V(mutex);
        V(barrera);
        for (i = 0; i < C-1; i++) {
            V(espera)
        }
    }
    P(esperar_receta[id]); //Me quedo esperando que se me asigne la receta.
    plato = cocino(receta[id]); // Cocino lo que se me asigna.
    platos[id] = plato // Una vez que termino de cocinar deposito el plato en el arrelgo de platos.
    P(mutex_2); // Accedo a la sección crítica porque tengo que usar la cola.
        push(terminaron, id);
    V(mutex_2);  // Libero la sección crítica.
    V(presentar_plato); // Aviso en el semaforo que hay un nuevo plato a examinar.
    P(esperar_nota[id]); // Espero a que me den la nota.
    mi_nota = notas[id]; // Agrego la nota a mi CV.

}

process Jurado{
    int i, idC
    P(barrera);
    for (i=0; i<C;i++){
        recetas[i] = genero_receta();
        V(esperar_receta[i])
    }

    for (i=0; i<C; i++) { //Se que hay C concursantes por lo tanto puedo recorrerlos. 
        P(presentar_plato); //Me quedo aca hasta que haya algun plato presentado
        P(mutex_2); // Accedo a la sección crítica porque tengo que usar la cola.
            idC = pop(terminaron);
        V(mutex_2);
        notas[idC] = poner_nota(platos[idC]);
        V(esperar_nota[idC]); // Actualizo el semáforo privado.
    }

}
```
