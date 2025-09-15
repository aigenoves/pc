# Resolución

## Ejercicios

2. c) Ídem b) pero cada proceso debe ocuparse de contar los fallos de un nivel de
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
