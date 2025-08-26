# Resolución

## Ejercicios

1. Para el siguiente programa concurrente suponga que todas las variables están inicializadas en 0 antes de empezar. Indique cual/es de las siguientes opciones son verdaderas:

    a) En algún caso el valor de x al terminar el programa es 56.

    b) En algún caso el valor de x al terminar el programa es 22.

    c) En algún caso el valor de x al terminar el programa es 23.

    ```plaintext
    P1::
    if (x = 0) then
        y:= 4 * 2
        x:= y + 2  
            1- Load Pos Mem y, Reg Acum
            2- Add 2, Reg Acum
            3- Store Reg Acum, Pos Mem x
    ```

    ```plaintext
    P2::
    if (x>0) then
        x:= x + 1
            1- Load Pos Mem x, Reg Acum
            2- Add 1, Reg Acum
            3- Store Reg Acum, Pos Mem x
    ```

    ```plaintext
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

    > a: 56 Si con la historia P1.1, P1.2, P1.3, P2.1, P2.2, P2.3, P3.1, P3.2, P3.3, P3.4, P3.5, P3.6, P3.7, P3.8, P3.9, P3.10
    >b: 22 Si con la historia P3.1, P3.2, P3.3, P1.1, P1.2, P1.3,P3.4, P3.5, P3.6, P3.7, P3.8, P3.9, P3.10, P2.1, P2.2, P2.3
    >c: 23 Si con la historia P3.1, P3.2, P3.3, P1.1, P1.2, P1.3, P2.1, P2.2, P2.3, P3.4, P3.5, P3.6, P3.7, P3.8, P3.9, P3.10

2. Realice una solución concurrente de grano grueso (utilizando <> y/o <await B; S>) para el siguiente problema. Dado un número N verifique cuántas veces aparece ese número en un arreglo de longitud M. Escriba las pre-condiciones que considere necesarias.

    ```plaintext
    integer N
    integer x := 0
    arreglo [M]

    process Contar[id:0..M-1] {
        if arreglo[id] == N {
            <x:=x+1>
        }
    }
    ```