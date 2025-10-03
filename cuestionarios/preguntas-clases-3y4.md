# Cuestionario guía - Clases Teóricas 3 y 4

1 a) Explique ña semántica de un semáforo

Un semáforo es una instancia de un tipo de datos abstracto (TDA) o un objeto. Fue descrito por Dijkstra en 1968.
Posee un Valor Interno: Internamente, el valor de un semáforo es un entero no negativo.
Posee dos operaciones atómicas P v Y:
    Operacion P: Demorar un proceso hasta que ocurra un evento(decrementa el valor interno del semaforo)
    Operacion V: Señala la ocurrecnia de un evento (incrementa el valor interno del semáforo)
