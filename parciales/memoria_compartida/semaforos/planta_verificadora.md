# Planta Verificadora

```java

sem mutex = 1;
sem pagar = 0;
sem verificar = 0;
sem salir[N] = ([N] 0);

int recibos[N] = ([N] 0);

queue cola_caja;
prioritary_queue cola_verificacion;


process Vehiculo(id: 0..N+1){
    text tipo;
    P(mutex)
        push(cola_caja, (id, tipo));
    V(mutex)
    if (tipo == "Auto"){
        V(pagar);
        P(espera_recibo);
    }
    P(mutex);
        push(cola_verificacion, (v));
    V(mutex)

    V(verificar);
    P(salir[id]);
    // Salgo de la planta
}

process Caja(){
    Vehiculo v;
    Recibo recibo
    
    while(true){
        vehiculo v;
        P(pagar);
        P(mutex)
            v = pop(cola_caja, (id, tipo))
        V(mutex);
        recibo = cobro();
        recibos[v.id] = recibo;
        P(espera_recibo[v.id]);
    }
}

process Verificacion(){
    vehiculo v;

    while(true){
        P(verificar);
        P(mutex)
            v = pop(v, (id, tipo))
        V(mutex);
        // verificando vehiculo
        V(salir[v.id]);
    }
}

```
