# Salita Médica

```java
process Paciente(id:0..30){
    int turno = ...;

    Salita.Esperar(turno);
    Consultorio.EsperarAtencion();

}

process Enfermera(){
    int cant;
    for (cant = 0; cant<30; cant++){
        Salita.Siguiente();
        Consultorio.EsperarPaciente();
        Vacunar()
        Consultorio.TerminarVacunacion();
    }

}

MONITOR Salita {
    int turno_actual = -1;
    cond espera[30], enf;
    bool presente[30] = ([30] false);

    Procedure Esperar(int turno) {
        presente[turno] = true;
        if (turno_actual == turno) signal (enf); // Si la enfermera esta dormida esperando que llegue el paciente con el turno actual, la despierta
        wait(espera[turno]); // Me tengo que demorar en mi variable condicion para que la enfermera me despierte.
    }

    Procedure Siguiente() {
        turno_actual = turno_actual + 1;
        if (not presente[turno_actual]) wait (enf);
        signal(esperar[turno_actual]);
    }

}

MONITOR Consultorio {
    bool llego_paciente = false;
    cond enf, pac;

    Procedure EsperarAtencion() {
        llego_paciente = true;
        signal(enf);
        wait(pac):
        signal(enf);
    }

    Procedure EsperarPaciente() {
        if (not llego_paciente) wait(enf);
        llego_paciente = false;
    }

    Procedure TerminarVacunacion(){
        signal(pac);
        wait(enf);
    }

}

```
