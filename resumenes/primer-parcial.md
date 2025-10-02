Este resumen abarca los conceptos fundamentales de Variables Compartidas, los desafíos de la concurrencia, y las soluciones algorítmicas clave presentadas en las fuentes, incluyendo Sentencias `await`, Semáforos y Monitores.

---

## Resumen de la Clase de Variables Compartidas

El tema de Variables Compartidas aborda los problemas de concurrencia que surgen cuando múltiples procesos acceden y modifican datos simultáneamente.

### 1. Las Bases de la Concurrencia y la Interferencia

Cuando se ejecuta un programa concurrente, las sentencias de alto nivel que modifican una variable compartida (como `x = x + y;`) no son atómicas a nivel de grano fino. Estas se descomponen en múltiples acciones atómicas básicas: **Cargar** el valor de la memoria a un registro, **Sumar/Modificar** en el registro acumulador, y **Almacenar** el resultado de vuelta en la posición de memoria.

**El Problema:** El sistema operativo puede interrumpir un proceso y cambiar el contexto entre cualquiera de estas acciones atómicas. Las diferentes secuencias de ejecución (llamadas "historias") resultan en posibles valores finales distintos para la variable compartida, lo que se conoce como una condición de carrera o interferencia.

**Ejemplo 1: Análisis de Historias**
Dado `int x = 1;`, `Process A { x = x + 5; }`, y `Process B { x = x + 3; }`.

| Acción | Descripción de Grano Fino | Proceso |
| :---: | :---: | :---: |
| 1 | Load Pos Memoria x, Reg Acumulador | A |
| 2 | Add Pos Memoria y (5), Reg Acumulador | A |
| 3 | Store Reg Acumulador, Pos Memoria x | A |
| 4 | Load Pos Memoria x, Reg Acumulador | B |
| 5 | Add Pos Memoria z (3), Reg Acumulador | B |
| 6 | Store Reg Acumulador, Pos Memoria x | B |

Si las acciones se ejecutan en orden secuencial (1-2-3-4-5-6 o 4-5-6-1-2-3), el resultado final de $x$ es **9**.
Sin embargo, si las acciones se entrelazan, el resultado puede ser inconsistente. Por ejemplo, en la historia **1-2-4-5-3-6**, el resultado de $x$ es **4**. Este comportamiento no deseado es la razón por la que se necesita sincronización y exclusión mutua.

### 2. Algoritmos y Mecanismos de Sincronización

Para evitar la interferencia, se utilizan mecanismos que garantizan que las secciones críticas (SC) que acceden a variables compartidas se ejecuten con **Exclusión Mutua (EM)**. Siempre que sea posible, se deben usar **variables locales** a cada proceso para evitar el uso de mecanismos de sincronización (como `<>`), que pueden reducir la concurrencia.

#### A. Soluciones de Grano Grueso con Sentencias `await`

Las sentencias `await` permiten definir secciones de código que se ejecutan de manera atómica, logrando exclusión mutua y/o sincronización por condición.

1.  **Forma General para Exclusión Mutua:**
    `< sentencias >`
    *   **Explicación:** El proceso ejecuta las sentencias de forma atómica. **Solo un proceso a la vez** puede estar ejecutando esa Sección Crítica (SC). Los procesos que esperan acceder a la SC no lo hacen por orden de llegada, sino que compiten una vez que se libera.
    *   **Ejemplo de Exclusión Mutua:** Para garantizar que el incremento de la variable compartida `Total` en el Ejemplo 2 sea atómico, se usa: `<Total = Total + 1>;`.

2.  **Forma General para Sincronización por Condición y EM:**
    `< await (B); sentencias >`
    *   **Explicación:** El proceso se detiene hasta que la condición $B$ es verdadera, momento en el cual ejecuta las `sentencias` de forma atómica. La atomicidad asegura que no hay posibilidad de que alguien modifique $B$ entre el momento en que se encuentra verdadera y la ejecución de las sentencias.
    *   **Ejemplo de Sincronización por Condición (Ejemplo 3 - Examen):** Para asegurar que un `Process Alumno` espere a ser llamado, se usa una variable compartida `Actual` y la sentencia: `<await (Actual == id)>;`.

#### B. Semáforos

Los Semáforos son un mecanismo de sincronización que debe inicializarse en la declaración (ej: `sem mutex = 1;`).

1.  **Operaciones Básicas:**
    *   **P(s):** Es atómico. Equivalente a `< await (s > 0) s = s-1; >`. El proceso se demora hasta que el semáforo `s` es mayor que cero, y luego lo decrementa.
    *   **V(s):** Es atómico. Equivalente a `< s = s+1; >`. Incrementa el semáforo `s`, despertando potencialmente a un proceso bloqueado.

2.  **Uso para Exclusión Mutua (Mutex):**
    *   **Explicación:** Un semáforo binario inicializado en 1 (`sem mutex = 1;`) se usa para proteger la SC. El proceso ejecuta `P(mutex)` al inicio de la SC y `V(mutex)` al final. Si un proceso intenta entrar a la SC mientras otro la ocupa, se bloquea en `P(mutex)`.
    *   **Ejemplo (Maximización de Concurrencia):** En el Ejemplo 1 (caramelos), la SC solo debe proteger el acceso al recurso compartido (tomar caramelo e incrementar `cant`), dejando las acciones no críticas (comer caramelo) fuera, entre `V(mutex)` y la siguiente iteración, para maximizar la concurrencia.
        ```
        Process Chico[id: 0..C-1] {
            while (true) {
                P(mutex);
                -- tomar caramelo
                cant = cant + 1;
                V(mutex);
                -- comer caramelo
            }
        }
        ```

3.  **Uso para Sincronización (Contador de Recursos):**
    *   **Explicación:** Un semáforo inicializado en 0 o en el número de recursos disponibles se usa para señalizar eventos o contar recursos.
    *   **Ejemplo (Productor/Consumidor - Ejemplo 4):** Para que el `Process Servidor` espere hasta que haya un pedido en la cola, se usa un semáforo contador `pedidos = 0;`. El `Cliente` (productor) avisa con `V(pedidos)` después de encolar, y el `Servidor` (consumidor) se demora con `P(pedidos)` antes de intentar sacar.

#### C. Monitores

Los Monitores son estructuras de alto nivel diseñadas para la sincronización, que encapsulan variables permanentes y procedimientos.

1.  **Características Clave:**
    *   **No existen variables compartidas** fuera del monitor.
    *   **Exclusión Mutua (EM):** Es **implícita**. Solo se permite un proceso ejecutando dentro de un monitor a la vez. El monitor se libera solo cuando el procedimiento termina o el proceso se duerme en una variable condición.
    *   **Acceso:** Los procesos compiten por acceder al monitor; **NO acceden de acuerdo al orden de llegada** cuando el monitor está libre.

2.  **Sincronización por Condición:**
    Se utilizan variables `cond` (variables condition) declaradas dentro del monitor.
    *   **`wait (vc)`:** Duerme al proceso en la cola asociada a la variable condición `vc`.
    *   **`signal (vc)`:** Despierta al primer proceso dormido en `vc` para que compita por reingresar al monitor.
    *   **`signal_all (vc)`:** Despierta a todos los procesos dormidos en `vc` para que compitan por reingresar.

3.  **Ejemplo (Administración de Acceso con Orden - Cajero Ejemplo 2):**
    Cuando el monitor necesita respetar un orden (Ejemplo: orden de llegada al cajero), el monitor debe **administrar** el acceso en lugar de representar directamente el recurso.
    *   Para implementar el orden, se necesita una variable booleana (`libre`) y una variable condition para la espera (`cola`). Además, se necesita contar cuántos procesos están esperando (`int esperando`) porque la función `empty()` no se puede usar en variables condition.

    *Código Simplificado del Monitor Cajero (que respeta el orden usando `esperando`):*
    ```
    Monitor Cajero{
        bool libre = true;
        cond cola;
        int esperando = 0;

        Procedure Pasar () {
            if (not libre) {
                esperando ++;
                wait (cola);
            }
            else libre = false;
        }

        Procedure Salir () {
            if (esperando > 0 ) {
                esperando --;
                signal (cola);
            }
            else libre = true;
        }
    }
    ```
    El proceso `Persona` llama a `Pasar()` (solicita acceso), ejecuta `UsarCajero()` (fuera del monitor), y luego llama a `Salir()` (libera el recurso).


# Algoritmos


Este listado presenta los algoritmos y la sincronización implementada para cada tema, seguido del código fuente asociado, excluyendo las explicaciones detalladas y el formato tabular, según lo solicitado.

---

## Variables Compartidas

### Algoritmo: Exclusión Mutua (EM) para Contadores (Ejemplo 2)

Se utiliza EM para asegurar la atomicidad del incremento de la variable compartida `Total`. La variable local `Parcial` no necesita protección.

```
int Total = 0;
Process Puerta[id: 0..3]
{  int Parcial = 0;
   while (true)
    { esperar llegada          Parcial = Parcial + 1;
      <Total= Total+ 1>;
    }
}
```

### Algoritmo: Sincronización Secuencial Estricta (Ejemplo 3)

Se sincroniza al Docente para que atienda a los alumnos en estricto orden por ID (`Actual`) y se usan variables booleanas (`Ok`, `Listo`) para sincronizar el inicio y el final del examen.

```
int Actual = -1;  bool Listo = False, Ok = false;
Process Alumno [id: 0..29]
{  <await (Actual == id)>;
   Ok = true;
   //Rinde el examen
   <await (Listo)>;
   Listo = false;
}
Process Docente
{   for i = 0..29
    {  Actual = i
       <await (Ok)>; Ok = false;
       //Toma el examen
       Listo = true;
       <await (not Listo)>;
    }
}
```

### Algoritmo: Sincronización Secuencial con Cola Ordenada (Ejemplo 4)

El Docente atiende por orden de ID entre aquellos alumnos que ya han llegado. Se usa una `colaOrdenada Espera` para almacenar los ID de los alumnos presentes. La extracción de la cola se protege con `await` para esperar a que la cola no esté vacía.

```
int Actual = -1;  bool Listo = False, Ok = false;   colaOrdenada Espera;
Process Alumno [id: 0..29]
{  <agregar (Epera, id)>;
   <await (Actual == id)>;
   Ok = true;
   //Rinde el examen
   <await (Listo)>;
   Listo = false;
}
Process Docente
{   for i = 0..29
    {  <await (not empty (Espera)); sacar(Espera, Actual)>;
       <await (Ok)>; Ok = false;
       //Toma el examen
       Listo = true;
       <await (not Listo)>;
    }
}
```

### Algoritmo: Administración de Acceso con Prioridad (Ejemplo 5)

Controla el acceso al cajero, permitiendo que una persona se auto-asigne el turno si está libre, o se encole (con prioridad para ancianos) si está ocupado. Utiliza la variable `Siguiente` para el turno.

```
colaEspecial C;
int Siguiente = -1;
Process Persona [id: 0..N-1]
{  int edad = ….;
   <if  (Siguiente = -1) Siguiente = id          else Agregar(C, edad, id)>;
   <await (Siguiente == id)>;
   //Usa el cajero
   <if  (empty(C)) Siguiente = -1          else Siguiente = Sacar(C)>;
}
```

### Algoritmo: Algoritmo "Ticket" (Orden de Llegada, Ejemplo 6 – versión 2)

Implementación para respetar estrictamente el orden de llegada sin usar una cola explícita, basándose en la asignación de tickets (`Numero`) y el seguimiento del turno actual (`Siguiente`).

```
int Numero = 0;
int Siguiente = 0;
Process Persona [id: 0..N-1]
{  int turno;
   <turno = Numero; Numero++ >;
   <await (Siguiente == turno)>;
   //Usa el cajero
   <Siguiente++>;
}
```

---

## Semáforos

### Algoritmo: Exclusión Mutua y Maximización de Concurrencia (Ejemplo 1)

Se usa un semáforo `mutex` inicializado en 1 para proteger el recurso compartido (tomar caramelo e incrementar `cant`), manteniendo la acción de `comer caramelo` fuera de la Sección Crítica para **maximizar la concurrencia**.

```
int cant = 0;
sem mutex = 1;
Process Chico[id: 0..C-1]
{ while (true)
   {  P(mutex);
      -- tomar caramelo
      cant = cant + 1;
      V(mutex);
      -- comer caramelo
   }
}
```

### Algoritmo: EM con Control de Límite de Recursos (Ejemplo 2 – versión 2)

Se limita la cantidad de caramelos (`N`). La condición de límite se rechequea dentro de la Sección Crítica para evitar que múltiples procesos accedan a la bolsa si solo queda un recurso.

```
int cant = 0;
sem mutex = 1;
Process Chico[id: 0..C-1]
{ while (cant < N)
   {  P (mutex);
      if  (cant < N)
          { -- tomar caramelo
            cant = cant + 1;
            V(mutex);
            -- comer caramelo
          }
      else V (mutex);
   }
}
```

### Algoritmo: Productor/Consumidor con Semáforo Contador (Ejemplo 4)

El Cliente (Productor) usa la cola `C` (protegida por `mutex`) para dejar pedidos y señaliza al Servidor (Consumidor) mediante el semáforo contador `pedidos`. El Servidor usa semáforos privados (`espera[id]`) para notificar individualmente al Cliente que el resultado está listo.

```
sem mutex = 1,   pedidos = 0,    espera[N] = ([N] 0);
int resultados[N];
cola C;
Process Cliente[id: 0..N-1]
{ secuencia S;
  while (true)
  { --generar secuencia S
    P(mutex);
    push(C, (id, S));
    V(mutex);
    V(pedidos);
    P(espera[id]);
    --ver resultado de resultados[id]
  }
}
Process Servidor
{ secuencia sec;   int aux;
  while (true)
  { P(pedidos);
    P(mutex);
    pop(C, (aux, sec));
    V(mutex);
    resultados[aux] = resolver(sec);
    V(espera[aux]);
  }
}
```

### Algoritmo: Alternancia de Servidores Basada en Tiempo (Ejemplo 5)

Dos Servidores alternan la atención cada 5 horas. Se utiliza un arreglo de semáforos de turno (`turno`) y un proceso `Reloj`. El semáforo `pedidos` se usa como contador de recursos y como señal de fin de tiempo enviada por el `Reloj` (`V(pedidos)` en el reloj).

```
sem mutex = 1,   pedidos = 0,    espera[N] = ([N] 0),   inicio = 0,    turno = (1, 0);
int resultados[N];     cola C;      bool finTiempo = false;
Process Servidor[id: 0..1]
{ secuencia sec;   int aux;     bool ok;
  while (true)
  { P(turno[id]);  finTiempo = false;  V(inicio);
    ok = true;
    while (ok)
    { P(pedidos);
      if  (finTiempo) { ok = false;
        V(turno[1-id]);
      }
      else { P(mutex);  pop(C, (aux, sec));  V(mutex);
        resultados[aux] = resolver(sec);
        V(espera[aux]);
      }
    }
  }
}
Process Reloj
{  while (true)
  { P(inicio); delay(5 hs); finTiempo = true;
    V(pedidos);
  }
}
```

### Algoritmo: Implementación de Barrera (Ejemplo 7)

Se implementa una barrera para `C` chicos usando un contador (`contador`) y un semáforo de señalización (`barrera`). Es crucial que el proceso libere la EM (`V(mutex)`) antes de demorarse en la barrera (`P(barrera)`).

```
int contador = 0;
sem mutex = 1;
sem barrera = 0;
Process Chico[id: 0..C-1]
{  int i;
  P(mutex);
  contador = contador + 1;
  if  (contador < C) { V(mutex);
    P(barrera);
  }
  else {  for i = 1..C-1  →   V(barrera);
    V (mutex);
  }
}
```

---

## Monitores

### Algoritmo: Exclusión Mutua Simple (Ejemplo 1)

El monitor representa el Recurso Compartido (Cajero) y asegura la Exclusión Mutua implícita mientras el proceso ejecuta el procedimiento.

```
Monitor Cajero{
  Procedure PasarAlCajero ()
      { UsarCajero ();
       }
}
Process Persona [id: 0..N-1]
{ ….
    Cajero.PasarAlCajero();
    ….
}
```

### Algoritmo: Administración de Acceso Ordenado (FIFO, Ejemplo 2)

El monitor administra el acceso. Usa una variable `bool libre` y una variable `int esperando` para contar los procesos dormidos, ya que la función `empty()` no se puede usar sobre variables `condition`.

```
Monitor Cajero{
  bool libre = true;
  cond cola;
  int esperando = 0;
  Procedure Pasar ()
      { if  (not libre) { esperando ++;
                                wait (cola);
                              }
         else  libre = false;
      }
 Procedure Salir ()
      { if  (esperando > 0 ) { esperando --;
                                           signal (cola);
                                        }
         else  libre = true;
       }
}
Process Persona [id: 0..N-1]
{  Cajero.Pasar ();
   UsarCajero();
   Cajero.Salir();
}
```

### Algoritmo: Administración de Acceso con Prioridad (Ejemplo 3)

Introduce prioridad usando variables `condition` **privadas** (`espera[N]`) para despertar específicamente al proceso que corresponda según la prioridad de la cola ordenada `fila`.

```
Monitor Cajero{
bool libre = true;
cond espera[N];
int idAux, esperando = 0;   colaOrdenada fila;
Procedure Pasar (idP, edad: in int)
{ if  (not libre) { insertar(fila, idP, edad);
  esperando ++;
  wait (espera[idP]);
}
else  libre = false;
}
Procedure Salir ()
{ if  (esperando > 0 ) { esperando --;
  sacar (fila, idAux);
  signal (espera[idAux]);
}
else  libre = true;
}
}
Process Persona [id: 0..N-1]
{  bool edad = ….;
   Cajero.Pasar (id, edad);
   UsarCajero();
   Cajero.Salir();
}
```

### Algoritmo: Administración de Empleados (Passing the Condition, Ejemplo 4 - Banco)

El monitor `Banco` gestiona la asignación de clientes a empleados. Utiliza la técnica *Passing the Condition* mediante la variable `cantLibres`, que se comprueba en `Llegada` para determinar si el cliente debe esperar en `esperaC`.

```
Monitor Banco {
cola elibres;
cond esperaC;
int esperando = 0, cantLibres = 0;
Procedure Llegada(idE: out int)
{ if  (cantLibres == 0) { esperando ++;
  wait (esperaC);
}
else cantLibres--;
pop(elibres, idE);
}
Procedure Próximo(idE: in int)
{  push(elibres, idE);
  if  (esperando > 0 ) { esperando --;
    signal (esperaC);
  }
  else cantLibres++;
}
}
```

### Algoritmo: Interacción Cliente/Empleado (Prevención de DEADLOCK, Ejemplo 4 - Escritorio)

El monitor `Escritorio` permite la comunicación bidireccional entre Cliente y Empleado. Usa la variable `listo` para evitar que el Empleado se bloquee permanentemente (`DEADLOCK`) si el Cliente llega antes y hace `signal` sin que haya nadie esperando.

```
Monitor Escritorio[id: 0..2] {
cond vcCliente, vcEmpleado;
text datos, resultados;
boolean listo = false;
Procedure Atención(D: in text; R: out text)
{ datos = D;
  listo = true;
  signal (vcEmpleado);
  wait (vcCliente);
  R = resultados;
  signal (vcEmpleado);
}
Procedure Esperardatos(D: out text)
{  if  (not listo) wait (vcEmpleado);
  D = datos;
}
Procedure EnviarResultados(R: in text)
{  resultados = R;
  signal (vcCliente);
  wait (vcEmpleado);
  listo = false;
}
}
```

### Algoritmo: Barrera de Procesos Centralizada (Ejemplo 5)

Implementa una barrera para 22 jugadores coordinada por el proceso `Partido`. Los jugadores esperan en `espera` hasta que el proceso `Partido` (despertado por el último jugador en `inicio`) ejecuta el *delay* y los libera.

```
Monitor Cancha
{  int cant = 0;
  cond espera, inicio;
  Procedure llegada ()
  { cant ++;
    if  (cant == 22) signal (inicio);
    wait (espera);
  }
  Procedure Iniciar ()
  { if  (cant < 22) wait (inicio);
  }
  Procedure Terminar ()
  { signal_all(espera);
  }
}
Process Jugador[id: 0..21]
{  Cancha.llegada();
}
Process Partido
{  Cancha. Iniciar();
  delay (90minutos); //se juega el partido
  Cancha.Terminar();
}
```

### Algoritmo: Productor/Consumidor (2 Servidores con VC Privadas, Ejemplo 7)

Adaptación para múltiples servidores. Se requiere la variable `HayPedido` para sincronizar al Servidor cuando la cola está vacía, la condición se rechequea usando `while (empty (C))`, y se usan variables `condition` privadas (`espera[N]`) para notificar al cliente específico, ya que el orden de finalización del trabajo puede variar entre servidores.

```
Monitor Admin {
Cola C;
Cond espera[N];
Cond HayPedido;
text res[N];
Procedure Pedido (IdC: in int; S: in text; R: out text)
{ push (C, (IdC,S) );
  signal (HayPedido);
  wait (espera[IdC]);
  R = res[IdC];
}
Procedure Sig (IdC: out int; S: out text)
{ while (empty (C)) wait (HayPedido);
  pop (C, (IdC, S));
}
Procedure Resultado (IdC: in int; R: in text)
{ res[IdC] = R;
  signal (espera[IdC]);
}
}
Process Servidor [id: 0..1]
{ text sec, res;
  int aux;
  while (true)
  { Admin.Sig(aux, sec);
    res = AnalizarSec(sec);
    Admin.Resultado(aux, res);
  }
}
```
