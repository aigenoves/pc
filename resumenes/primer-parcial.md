# Conceptos Clave y Algoritmos de Concurrencia
---

## Variables Compartidas

El concepto central al trabajar con Variables Compartidas (VC) es la **sincronización** para prevenir la **interferencia** y asegurar la **atomicidad** en las operaciones.

1.  **Atomicidad y Exclusión Mutua (EM):**
    *   Una operación atómica realiza una transformación de estado que es indivisible e invisible a otros procesos. Las operaciones simples (lectura/escritura de tipos básicos) suelen ser atómicas a nivel de *grano fino*.
    *   Sin embargo, operaciones comunes como el incremento de un contador (`x = x + 1`) no son atómicas a nivel de grano fino y deben protegerse.
    *   La **Exclusión Mutua** (EM) es una propiedad de seguridad que garantiza que **a lo sumo un proceso** está accediendo a la *Sección Crítica* (SC) en un momento dado, asegurando estados consistentes.

2.  **La Sentencia `await` de Grano Grueso:**
    *   El mecanismo fundamental para lograr la sincronización con variables compartidas es la sentencia `await`.
    *   **Forma general:** `<await (B) S;>`: el proceso se demora hasta que la condición $B$ es verdadera, momento en el que ejecuta la secuencia de sentencias $S$ de forma **atómica**.
    *   **Sincronización por Condición:** El `await` se utiliza para bloquear un proceso hasta que se cumpla una condición específica ($B$).
    *   **Uso de `< >`:** La notación de corchetes angulares (`<sentencias>`) define una **acción atómica de grano grueso** (coarse-grained). Es vital usarla cuando se accede o modifica variables compartidas cuyas operaciones no son atómicas por sí mismas, como las que componen los contadores.
    *   **Maximización de Concurrencia:** Siempre que sea posible, se deben utilizar **variables locales** para evitar el uso de la atomicidad de grano grueso (`< >`), lo cual reduce la concurrencia.

3.  **Ordenamiento y Administración de Acceso:**
    *   Las soluciones simples de EM con `await` (como la protección de un *lock* booleano) generalmente no garantizan el orden de llegada (FIFO) de los procesos en espera.
    *   Para implementar **sincronización secuencial estricta** o **administración de acceso ordenado** (como en los algoritmos Ticket, o con colas ordenadas), se deben introducir variables de estado compartidas (como `Actual` o `Siguiente`, o `Numero`/`Turno` en Ticket) que dictan explícitamente qué proceso tiene permiso para avanzar.

---

## Semáforos

Los semáforos son una herramienta de sincronización de **más alto nivel** que buscan simplificar los protocolos de concurrencia y evitar el **busy waiting** ineficiente asociado con soluciones puramente basadas en variables compartidas.

1.  **Definición y Operaciones:**
    *   Un semáforo es un entero no negativo con dos operaciones atómicas: **P** y **V**.
    *   **P(s):** Se utiliza para **demorar** un proceso hasta que ocurra un evento (espera a que $s > 0$ y luego lo decrementa). Si el semáforo es 0, el proceso se bloquea (se duerme).
    *   **V(s):** Se utiliza para **señalar** la ocurrencia de un evento (incrementa $s$). Si hay procesos bloqueados en $P(s)$, uno de ellos es despertado.

2.  **Implementación de Exclusión Mutua (EM):**
    *   Un **semáforo binario** (inicialmente 1) se utiliza para proteger la SC.
    *   $P(mutex)$ actúa como protocolo de entrada a la SC, y $V(mutex)$ como protocolo de salida.

3.  **Maximización de Concurrencia (SC mínima):**
    *   Es un concepto crítico que la **Sección Crítica** (el código entre $P$ y $V$) debe ser lo más pequeña posible. Debe incluir **solo** aquellas operaciones que requieren acceso exclusivo al recurso compartido o a las variables de estado que dictan la sincronización. Las tareas individuales no concurrentes deben quedar fuera del par $P/V$.

4.  **Tipos de Uso para Sincronización:**
    *   **Contadores de Recursos (Semáforos Generales):** El valor del semáforo cuenta el número de unidades libres de un recurso. Es común en problemas Productor/Consumidor para contar espacios vacíos y ocupados.
    *   **Señalización de Eventos:** Los semáforos inicializados en 0 se usan para que un proceso espere un evento ($P(s)$) y otro lo señale ($V(s)$).
    *   **Semáforos Privados:** Semáforos donde exactamente un proceso ejecuta la operación $P$. Son esenciales para la comunicación uno a uno (como devolver resultados a clientes específicos en el Problema Productor/Consumidor o Alocación de Recursos).
    *   **Passing the Baton:** Técnica que utiliza Semáforos Binarios Divididos o privados para controlar de manera precisa el orden en que los procesos son despertados, útil para implementar sincronizaciones condicionales complejas y resolver problemas de *scheduling* y prioridad.

5.  **Barreras:**
    *   Una barrera es un punto de sincronización donde todos los procesos deben llegar antes de que cualquiera pueda continuar. Se implementan usando contadores compartidos (para saber cuántos han llegado) y semáforos de señalización (para despertar a todos simultáneamente).

---

## Monitores

Los monitores son una construcción de lenguaje de programación de muy **alto nivel** que encapsulan los datos compartidos y los procedimientos que operan sobre ellos, proporcionando **Exclusión Mutua implícita**.

1.  **Exclusión Mutua Implícita:**
    *   La EM es automática: solo un proceso puede estar ejecutando dentro de un procedimiento del monitor en un momento dado. Esto elimina la necesidad de primitivas de bloqueo explícitas ($P/V$ o *locks*).
    *   El monitor solo se libera si el proceso termina el procedimiento o si el proceso se **bloquea** en una variable de condición.

2.  **Variables de Condición (`cond vc`):**
    *   La sincronización por condición se maneja explícitamente mediante variables `condition` declaradas dentro del monitor.
    *   **`wait(vc)`:** El proceso se duerme en la cola asociada a `vc` y cede el monitor.
    *   **`signal(vc)`:** Despierta al **primer** (más antiguo) proceso dormido en esa cola, el cual pasa a competir para reingresar al monitor.
    *   **`signal_all(vc)`:** Despierta a todos los procesos dormidos en la cola.

3.  **El Rol del Monitor (Administrador vs. Recurso):**
    *   Para maximizar la concurrencia, el monitor generalmente debe actuar como un **administrador** del acceso al recurso (RC) compartido, no como el RC en sí. El trabajo intensivo debe realizarse fuera del monitor.
    *   Por ejemplo, en un cajero, el monitor administra el turno (`Pasar()` y `Salir()`), pero el proceso realiza la función `UsarCajero()` fuera del monitor.

4.  **Administración de Acceso y Prioridad:**
    *   Si se requiere respetar un orden estricto (FIFO) o una **prioridad** (Ej. 3), no basta con una sola variable de condición. El orden de la cola interna del `wait/signal` es FIFO, pero el `signal` despierta al proceso que luego debe competir para reingresar.
    *   Para asegurar un orden específico (como FIFO o prioridad), se suelen usar:
        *   **Variables de Condición Privadas:** Se usa un arreglo de variables de condición (`espera[N]`) para despertar a un proceso *en particular* (por su ID).
        *   **Estructuras de Datos Auxiliares:** Se emplea una cola compartida (ej. `colaOrdenada` o `fila`) dentro del monitor para almacenar los IDs de los procesos en el orden deseado (prioridad, llegada, etc.).

5.  **Técnicas Avanzadas:**
    *   **Passing the Condition (o *Counted Waiting*):** Se utiliza una variable compartida (como `cantLibres` o `esperando`) para llevar la cuenta de los recursos disponibles o los procesos en espera, en lugar de depender únicamente del estado de la cola de condición. Esto permite al proceso que está saliendo determinar explícitamente si debe despertar a alguien o si puede simplemente liberar el recurso. Es clave en la administración de empleados (Ej. 4 - Banco).
    *   **Prevención de DEADLOCK:** En interacciones cliente/empleado (Ej. 4 - Escritorio), la comunicación punto a punto debe manejar la llegada asíncrona de las partes. Se deben utilizar **variables de estado booleanas** (`listo`) para evitar que un proceso que llega antes que el otro ejecute un `signal` que se pierde, resultando en un *deadlock* cuando el segundo proceso llega e intenta hacer un `wait`.
    *   **Re-chequeo de Condiciones:** Cuando hay múltiples procesos que compiten por un recurso (Ej. 7: 2 Servidores), si un servidor es despertado por `signal`, debe re-chequear la condición (ej. `while (empty(C)) wait (HayPedido);`) para asegurarse de que el recurso no fue consumido por otro servidor antes de que pudiera acceder al monitor.
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
