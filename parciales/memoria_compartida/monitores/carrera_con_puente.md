# Carrera con puente

Resolver con ***MONITORES*** el siguiente problema. Se debe simular una carrera a campo traviesa con *C corredores* donde en a mitad del recorrido hay un puente colgante que puede ser usado por una única persona a la vez. Cuando los *C corredores* han llegado al punto de partida comienza la carrera. Cuando un corredor llega al puente espera su turno (respetando el orden de llegada al mismo *Passing the Condition*) y lo cruza (suponga que tarda un par de minutos en cruzarlo); y luego continua su carrera hasta llegar a la meta. ***Nota:*** cada correrdor pasa sólo una vez por el puente. Sólo se pueden usar procesos que representen a los corredores.

```java

Process Corredor(id: 0 ..C-1){
    Largada.EsperarInicio(); //Barrera
    //Correr;
    Puente.Solicitar();
    //Cruzar el puente;
    AdminPuente.Dejar();
    //Correr;

}


MONITOR AdminPuente{
    bool libre = true;
    int cant = 0;
    cond espera;

    // Passing the Condition
    Procedure Solicitar(){
        if (libre){
            libre = false
        }
        else { 
            cant=cant+1;
            wait(espera);
        }
    }

    Procedure Dejar(){

        if (cant == 0){
            libre = true;
        }
        else {
            cant=cant-1;
            signal(espera);
        }
    }
    // Fin Passing the Condition
}

// monitor barrera
MONITOR Largada{
    int cantidad = 0;
    cond espera;

    Procedure EsperarInicio(){
        cantidad=cantidad+1;
        if (cantidad<C) wait(espera)
        else signal_all(espera);
    }

}

```
