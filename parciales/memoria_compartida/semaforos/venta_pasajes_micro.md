# Resolver con Semáforos

Una empresa de turismo posee **UN** micro con capacidad para 50 personas. Hay un único *vendedor* que atiende a los C *clientes* (C>50) que intentan comprar un pasaje de acuerdo al orden de llegada (suponga que la atención de un cliente tarda un par de minutos), si aún hay lugares disponibles le indica el número de asiento que le tocó. Cada cliente, luego de ser atendido por el vendedor, se dirige al micro para subir en caso de que le hayan dado un asiento, y en caso contrario se retira sin viajar. El micro espera a que los 50 pasajeros hayan subido para realizar el viaje. **Nota:** Maximizar la concurrencia.

Solución:

```java
sem mutex = 1; //Administro el acceso a la cola Clientes
sem hay_cliente = 0; // Semaforo que controla que el Vendedor puede atender a un cliente.
sem cliente_micro = 0 // Semáforo para avisar que hay un pasajero en el micro.
sem vendidos[C] = ([C] 0); //Semaforo privado que le dice al cliente si se le vendio un pasaje o no.

int pasaje[C] = ([C] 0); // Semáforo que indica que, para el cliente en su posición, pudo comprar o no pasajes.
queue Clientes; // Cola de clientes que necesitan pasajes.

process Cliente(id:0..C-1){
	P(mutex); //Voy a usar la sección crítica
	push(Clientes, id); // Me agrego a la cola
	V(mutex) // Salgo de la sección crítica.
	V(hay_cliente); //Le aviso al vendedor que llego un cliente.
	P(vendidos[id]); // Me quedo en el semáforo privado hasta que el vendedor me atienda.
	if (pasajes[id] > 0) { // Aca pregunto si el vendedor me pudo vender un pasaje
		V(cliente_micro); // Le aviso al chofer que entro al micro
	};
};

process Vendedor(){
int cant_pasajes_vendidos = 0;
int i, id_cliente

for (i=0; i<C; i++){ //Recorro lo C clientes
	P(hay_cliente); // Pregunto si hay clientes en la cola
	P(mutex); //Accedo a la secció crítica
		pop(Clientes, id_cliente); // Saco al cliente de la cola por orden de llegada.
	V(mutex); // Libero la sección crítica
	if (cant_pasajes_vendidos < 50) { // Si aun quedan pasajes para vender
		cant_pasajes_vendidos = cant_pasajes_vendidos + 1; //incremento la cantidad de pasajes vendidos
		pasajes[idC] = asignar_asiento(); // Agrego el asiento en el semaforo. 
	};
	V(vendidos[idC]); // Agrego el id del cliente al semaforo privado
};

};

proccess Micro(){
int i;

for (i = 0; i<50;i++) P(cliente_micro); // Recorro los C clientes preguntando si ya estan en el micro. Si hay 50 comienzo el viaje.
// Empieza el viaje.

};
```
