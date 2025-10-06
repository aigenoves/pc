# Alumnos y Profesores

Resolver con **SEMAFOROS** el siguiente problema. En un examen final hay *P alumnos* y *3 profesores*. Cuando todos los alumnos y los profesores han llegado comienza el examen. Para esto uno de los profesores (el primero en llegar) le da el examen a cada alumno. Cada alumno resuelve su examen, lo entrega y espera a que alguno de los profesores lo corrija y le indique la nota. Los profesores corrigen los exámenes respetando el orden en que los alumnos van entregando. **Nota:** maximizar la concurrencia.

```java

sem llega_persona=1;
sem entrega_examen = 1;
sem llega_primer_profe = 0;
sem tiene_examen[P] =([P] 0)
sem tiene_nota[P] =([P] 0)
sem hay_examen_a_corregir = 0;

int primer_profe = -1;

text examenes[P];
text notas[P];

int cantidad_personas = 0;
queue entregas;

process Alumno(id:1..P){
    text mi_examen;
    int mi_nota

    P(llega_persona); // Entro en la SC
    cantidad_personas = cantidad_personas + 1;
    if (cantidad_personas = P + 3) V(llega_primer_profe);
    V(llega_persona); // Salgo de la SC
    P(tiene_examen[id]); //Barrera Privada de cada alumno
    mi_examen = resuelvo_examen(examenes[id]); //Cada alumno puede tardar diferente tiempo.
    P(entrega_examen); // Entro en la SC
    push(entregas, (id, mi_examen));
    V(entrega_examen); // Salgo de la SC
    V(hay_examen_a_corregir); //Avisa que hay un examen para corregir
    P(tiene_nota[id]); //Barrera Privada de los alumnos para saber que ya tienen la nota
    mi_nota = notas[id] //Reviso mi nota
}

process Profesor(id:1..3){
    int i;
    text examen;

    P(llega_persona); // Entro en la SC
    cantidad_personas = cantidad_personas + 1;
    if (cantidad_personas = P + 3) V(llega_primer_profe);
    if (primer_profe == -1) primer_profe = id;
    V(llega_persona); // Salgo de la SC

    if (primer_profe == id) {
        P(llega_primer_profe);
        for (i=0; i<P;i++){
            examen = genero_examen();
            examenes[i] = examen;
            V(tiene_examen[i]);
        }
    }

    while (true) {
        P(hay_examen_a_corregir); //Me entero que hay examen a corregir.
        P(entrega_examen); // Entro en la SC
        pop(entregas, (id_alumno, examen));
        V(entrega_examen); // Salgo de la SC
        nota = corrijo_examen(examen);
        notas[id_alumno] = nota;
        V(tiene_nota[id_alumno]); //Aviso que ya esta corregido.
        }
}

```
