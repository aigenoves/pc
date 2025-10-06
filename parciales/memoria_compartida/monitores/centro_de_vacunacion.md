# Centro de vacunación

Resolver el siguiente problema con **MONITORES**.
Simular la atención en un Centro de Vacunación con 8 puestos para vacunar contra el coronavirus. Al Centro acuden ***200 pacientes*** para ser vacunados, cada uno de ellos ya conoce el puesto al que se debe dirigir. En cada puesto hay ***UN empleado*** para vacunar a los pacientes asignados a dicho puesto, y lo hace de acuerdo al orden dado por la edad del paciente (cuando está libre atiende al paciente de mayor edad que esté esperando en ese momento en ese puesto).

Cada paciente al llegar al puesto que tenía asignado espera a que lo llamen y se dirige a la silla para que el empleado lo vacune, y luego se retira.

***Nota***: suponer que existe una función Vacunar() que simula la atención del paciente. Suponer que cada puesto tiene asignado 25 pacientes. Todos los procesos deben terminar.

```java

Process Paciente(id:0..199){
    int num_puesto = ....;
    int edad = ....;

    Puesto[num_puesto].Esperar(id, edad);
    Silla[num_puesto].EsperarAtencion();

}

Process Empleado(id:0..7){
    int cant;
    for (cant=0; cant < 25; cant++){ //Atiendo solo a 25 personas
        Puesto[id].Siguiente(); //Le pido a mi puesto que le avise al siguiente que vaya a la silla.
        Silla[id].EsperarPaciente(); //Espero a que el paciente este en la silla.
        Vacunar(); //Vacuno al paciente
        Silla[id].TerminarVacunacion(); //Le aviso al paciente que ya se se termino la vacunacion que se levante de la silla.
    }

}

MONITOR Puesto(id:0..7){
    cond esperaPac[200], emp;
    colaOrdenada cola;
    int id_paciente, edad_paciente;

    Procedure Esperar(id, edad){
        agregar(cola, (id_paciente, edad)); //agrego en la cola, de manera ordenada por edad, al pciente
        signal(emp); //Por las dudas despierto al empleado
        wait(esperarPac[id_paciente]) // duermo al paciente en la condicion esperarPac en la posicion de su id hasta que le indique el empleado que se despierte.
    }

    Procedure Siguiente(){
        if (empty(cola)) wait(emp); // Si no hay nadie en la cola duermo al empleado
        sacar(cola, (id_paciente, edad_paciente)); // saco de la cola un paciente
        signal(esperarPac[id_paciente]) // despierto al paciente.
    }

}

MONITOR Silla(id:0..7){
    bool llego_paciente = false;
    cond emp, pac;

    Procedure EsperarAtencion(){
        llego_paciente = true;
        signal(emp);
        wait(pac);
        signal(emp);
    }

    Procedure EsperarPaciente(){
        if (not llego_paciente) wait(emp): // Si el paciente no llego
        llego_paciente = false; // Si ya estaba el paciente o llego el paciente
    }

    Procedure TerminarVacunacion(){
        signal(pac); // le dice al paciente que termino de vacunarlo y que se vaya
        wait(emp);
    }
}

```
