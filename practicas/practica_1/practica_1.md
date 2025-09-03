# Resolución

## Ejercicios

1. Para el siguiente programa concurrente suponga que todas las variables están inicializadas en 0 antes de empezar. Indique cual/es de las siguientes opciones son verdaderas:

    a) En algún caso el valor de x al terminar el programa es 56. [✓]

    b) En algún caso el valor de x al terminar el programa es 22. [✓]

    c) En algún caso el valor de x al terminar el programa es 23. [✓]

    ```java
    P1::
    if (x = 0) then
        y:= 4 * 2
        x:= y + 2  
            1- Load Pos Mem y, Reg Acum
            2- Add 2, Reg Acum
            3- Store Reg Acum, Pos Mem x
    ```

    ```java
    P2::
    if (x>0) then
        x:= x + 1
            1- Load Pos Mem x, Reg Acum
            2- Add 1, Reg Acum
            3- Store Reg Acum, Pos Mem x
    ```

    ```java
    P3::
    x:= (x*3) + (x*2) + 1
            (x*3)
                1- Load Pos Mem x, Reg Acum
                2- Mul 3, Reg Acum
                3- Store Reg Acum, Pos Mem T1 //reg aux T1
            (x*2)
                4- Load Pos Mem x, Reg Acum
                5- Mul 2, Reg Acum
                6- Store Reg Acum, Pos Mem T1 //reg aux T2
            (x*3) + (x*2) + 1
                7- Load Pos Mem T1, Reg Acum
                8- Add Pos Mem T2, Reg Acum
                9- Add 1. Reg Acum
            10- Store Reg Acum, Pos Mem x
    ```

    >a: 56 Si con la historia P1<sub>1</sub> P1<sub>2</sub> P1<sub>3</sub> P2<sub>1</sub> P2<sub>2</sub> P2<sub>3</sub> P3<sub>1</sub> P3<sub>2</sub> P3<sub>3</sub> P3<sub>4</sub> P3<sub>5</sub> P3<sub>6</sub> P3<sub>7</sub> P3<sub>8</sub> P3<sub>9</sub> P3<sub>10</sub>

    >b: 22 Si con la historia P3<sub>1</sub> P3<sub>2</sub> P3<sub>3</sub> P1<sub>1</sub> P1<sub>2</sub> P1<sub>3</sub>P3<sub>4</sub> P3<sub>5</sub> P3<sub>6</sub> P3<sub>7</sub> P3<sub>8</sub> P3<sub>9</sub> P3<sub>10</sub> P2<sub>1</sub> P2<sub>2</sub> P2<sub>3</sub>

    >c: 23 Si con la historia P3<sub>1</sub> P3<sub>2</sub> P3<sub>3</sub> P1<sub>1</sub> P1<sub>2</sub> P1<sub>3</sub> P2<sub>1</sub> P2<sub>2</sub> P2<sub>3</sub> P3<sub>4</sub> P3<sub>5</sub> P3<sub>6</sub> P3<sub>7</sub> P3<sub>8</sub> P3<sub>9</sub> P3<sub>10</sub>

2. Realice una solución concurrente de grano grueso (utilizando <> y/o <await B; S>) para el siguiente problema. Dado un número N verifique cuántas veces aparece ese número en un arreglo de longitud M. Escriba las pre-condiciones que considere necesarias.

    Solución 1 con M procesos: [^1]

    ```java
    integer N := ...
    integer x := 0
    arreglo [M]

    process Contar[id:0..M-1] {
        if arreglo[id] == N {
            <x:=x+1>
        }
    }
    ```

    Solución 2 con 10 procesos: [^2]

    Precondicones: arreglo de 1000 y 10 procesos contar

   ```java
    integer N := ...
    integer x := 0
    arreglo [1000]
    integer bloque := 1000 div 10

    process Contar[id:0..9] {
        integer inicio := id*bloque
        integer fin := (id+1) * bloque - 1
        contador := 0
        for i = inicio..fin {
            if arreglo[i] == N {
                contador := contador + 1
            }
        }
        // SC
        <x:=x+contador>
    }
    ```

3. Dada la siguiente solución de grano grueso:  

    a) Indicar si el siguiente código funciona para resolver el problema de Productor/Consumidor  con  un  buffer  de  tamaño  N.  En  caso  de  no  funcionar,  debe hacer las modificaciones necesarias.

    ```java
    int cant = 0;
    int pri_ocupada = 0;
    int pri_vacia = 0;
    int buffer[N];
    Process Productor::  
    { while (true) 
        { produce elemento 
            <await (cant < N); cant++> 
            buffer[pri_vacia] = elemento; 
            pri_vacia = (pri_vacia + 1) mod N; 
        } 
    } 
    Process Consumidor::  
    { while (true) 
        { <await (cant > 0); cant-- > 
            elemento = buffer[pri_ocupada]; 
            pri_ocupada = (pri_ocupada + 1) mod N; 
            consume elemento 
        } 
    }
    ```

    Modificación[^3]:

    ```java
    int cant = 0;
    int pri_ocupada = 0;
    int pri_vacia = 0;
    int buffer[N];
    Process Productor[i:0..P-1]::  
    { while (true) 
        { produce elemento 
            <await (cant < N); cant++ 
            buffer[pri_vacia] = elemento;>
            pri_vacia = (pri_vacia + 1) mod N; 
        } 
    } 
    Process Consumidor[i:0..C-1]::  
    { while (true) 
        { <await (cant > 0); cant-- 
            elemento = buffer[pri_ocupada];>
            pri_ocupada = (pri_ocupada + 1) mod N; 
            consume elemento 
        } 
    }
    ```

    b) Modificar el código para que funcione para C consumidores y P productores. [^4]

    ```java
        int cant = 0;
        int pri_ocupada = 0;
        int pri_vacia = 0;
        int buffer[N];
        Process Productor[i:0..P-1]::  
        { while (true) 
            { produce elemento 
                <await (cant < N); cant++ 
                buffer[pri_vacia] = elemento;
                pri_vacia = (pri_vacia + 1) mod N;>
            } 
        } 
        Process Consumidor[i:0..C-1]::  
        { while (true) 
            { <await (cant > 0); cant-- 
                elemento = buffer[pri_ocupada];
                pri_ocupada = (pri_ocupada + 1) mod N>
                consume elemento 
            } 
        }
    ```

    [^1]: Esta solución es correcta pero podemos pensarla con una cantidad finita de procesos Contar.

    [^2]: Esta solución es para una cantidad de 1000 elementos del arreglo y una cantidad de 10 prcesos Contar. Se peude generalizar.

    [^3]: En este caso se debe proteger el buffer en la posición pri_vacia y pre_ocupada ya que otro proceso puede estar utilizandola.

    [^4]: En este caso se debe proteger la asignacion de pri_vacia y pre_ocupada que que otro proceso puede estar calculandola.

4. Resolver con SENTENCIAS AWAIT (<> y <await B; S>). Un sistema operativo mantiene 5  instancias de un recurso almacenadas en una cola, cuando un proceso necesita usar una instancia del recurso la saca de la cola, la usa y cuando termina de usarla la vuelve a depositar.

    ```java
        colaRecursos colaRec[5];
        int cant = 0;
                
        Process So[i:0..P-1]::  
        Recurso recurso;
        { while (true) 
            {  
                <await (cant < 5); cant++ 
                recurso = colaRec.pop();>
                // Utilizo el recurso
                <colaRec.push(recurso);
                cant--;>
            } 
        } 
    ```

5. En cada ítem debe realizar una solución concurrente de grano grueso (utilizando <> y/o <await B; S>) para el siguiente problema, teniendo en cuenta las condiciones indicadas en el item. Existen N personas que deben imprimir un trabajo cada una.

   a. Implemente una solución suponiendo que existe una única impresora compartida por todas las personas, y las mismas la deben usar de a una persona a la vez, sin importar el orden. Existe una función Imprimir(documento) llamada por la persona que simula el uso de la impresora. Sólo se deben usar los procesos que representan a las Personas.

    ```java
        Process Persona[id: 0..N-1]::
        {
            Documento doc;
            <Imprimir(doc);>
        } 
    ```

   b. Modifique la solución de (a) para el caso en que se deba respetar el orden de llegada.

    ```java
        int numero = 0;
        int proximo = 0;
        array[0..N-1]  ordenLlegada = -1 // Aca quiero representar que asigno -1 a todas las pos.
        
        Process Persona[id: 0..N-1]::
        {
            Documento doc;
            <ordenLlegada = numero; numero++;> 
            <await turno[i] == proximo>
            Imprimir(doc); // Creo que no necesito que sea de grano grueso
            proximo++; // porque como proximo no aumenta hasta el final entonces no avanza.
        } 
    ```

   c. Modifique la solución de (a) para el caso en que se deba respetar el orden dado por el identificador del proceso (cuando está libre la impresora, de los procesos que han solicitado su uso la debe usar el que tenga menor identificador).

    ```java

        colaOrdenada Espera;
        int siguiente = -1;
        Process Persona[id: 0..N-1]::
        {
            Documento doc;
            <if siguiente == -1 { // Nadie esta usando la impresora para que me voy a agregar. La uso.
                siguiente = id;
            } else {
                agregar(Espera, id); // La estan usando me agrego en la cola.
            }>
            <await siguiente == id;> // Espero a que me toque
            Imprimir(doc);
            
            <if empty(Espera) { // Si esta  vacia es porque soy el ultimo que la uso. Reinicio el contador.
                siguiente = -1;
            } else {
                siguiente = sacar(Espera) // No soy el último, le digo al siguiente que es su turno.
            }>
        } 
    ```

   d. Modifique la solución de (b) para el caso en que además hay un proceso Coordinador que le indica a cada persona que es su turno de usar la impresora.
