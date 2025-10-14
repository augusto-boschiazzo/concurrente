# Practica 1

### Variables compartidas

---

2. _Realice una solución concurrente de grano grueso (utilizando <> y/o <await B; S>) para el siguiente problema. Dado un número N verifique cuántas veces aparece ese número en un arreglo de longitud M. Escriba las pre-condiciones que considere necesarias._

    ```c
    int cant = 0; int arreglo[0..M-1];

    Process::Comprobador[i: 0..M-1]
    {
        if arreglo[i] == N <cant++;>
    }
    ```

---

3. _Dada la siguiente solución de grano grueso:_

   1. _Indicar si el siguiente código funciona para resolver el problema de Productor/Consumidor con un buffer de tamaño N. En caso de no funcionar, debe hacer las modificaciones necesarias_

        ```c
        int cant = 0; int pri_ocupada = 0; int pri_vacia = 0; int buffer[N];
    
        Process::Productor
        { 
            while (true)
            { // produce elemento
                <await (cant < N); cant++>
                buffer[pri_vacia] = elemento;
                pri_vacia = (pri_vacia + 1) mod N;
            }
        }

        Process::Consumidor
        { 
            while (true)
            { 
                <await (cant > 0); cant-- >
                elemento = buffer[pri_ocupada];
                pri_ocupada = (pri_ocupada + 1) mod N;
                // consume elemento
            }
        }

        ```

        No es correcta:

        ```c
        Process::Productor
        {
            while(true)
            {
                <await(cant < N); buffer[pri_vacio] = elemento; cant++;>
                pri_vacio = (pri_vacio + 1) mod N;
            }
        }

        Process::Consumidor
        {
            while(true)
            {
                <await(cant > 0); elemento = buffer[pri_ocupado]; cant--;>
                pri_ocupado = (pri_ocupado + 1) mod N;
            }
        }   
        ```

    2. _Modificar el código para que funcione para C consumidores y P productores._
    
        Se deberían agregar las operaciones de aumento de pri\_ocupado y pri\_vacio en la sentencia de grano grueso.

---

4. _Resolver con SENTENCIAS AWAIT (<> y <await B; S>). Un sistema operativo mantiene 5 instancias de un recurso almacenadas en una cola, cuando un proceso necesita usar una instancia del recurso la saca de la cola, la usa y cuando termina de usarla la vuelve a depositar_

    ```c
    int cant = 0; colaRecurso c[5];
    
    Process::Consumidor[id: 0..N-1]
    {
        Recurso recurso;
        while(true)
        {
            <await(cant < 5); recurso = c.pop(); cant++;>
            // uso recurso
            <c.push(recurso); cant--;>
        }
    }
    ```

---

5. _En cada ítem debe realizar una solución concurrente de grano grueso (utilizando <> y/o <await B; S>) para el siguiente problema, teniendo en cuenta las condiciones indicadas en el item. Existen N personas que deben imprimir un trabajo cada una._

    1. _Implemente una solución suponiendo que existe una única impresora compartida por todas las personas, y las mismas la deben usar de a una persona a la vez, sin importar el orden. Existe una función Imprimir(documento) llamada por la persona que simula el uso de la impresora. Sólo se deben usar los procesos que representan a las Personas._

        ```c
        Impresora impresora;
        
        Process::persona[id: 0..N-1]
        {
            while(true)
            {
                <Imprimir(documento);>
            }
        }
        ```

    2. _Modifique la solución de (a) para el caso en que se deba respetar el orden de llegada._

        ```c
        int turno = 0; int proximo = 0; int cola[0..N-1] = ([n] = 0);

        Process::Persona[id: 0..N-1]
        {
            <cola[id] = turno; turno++>
            <await(cola[id] == proximo);>
            Imprimir(documento);
            turno++;
        }
        ```

    3. _Modifique la solución de (a) para el caso en que se deba respetar el orden dado por el identificador del proceso (cuando está libre la impresora, de los procesos que han solicitado su uso la debe usar el que tenga menor identificador)._

        ```c
        int turno = 0;

        Process::Persona[id: 0..N-1]
        {
            <await(id == turno);>
            Imprimir(documento);
            turno++;
        }
        ```

    4. _Modifique la solución de (b) para el caso en que además hay un proceso Coordinador que le indica a cada persona que es su turno de usar la impresora._

        ```c
        int turno = 0, proximo = 0; bool seguir = false; int cola[0..N-1] = ([n] = 0);

        Process::Persona[id: 0..N-1]
        {
            while(true)
            {
                <cola[id] = turno; turno++;>
                <await(cola[id] == proximo);>
                Imprimir(documento);
                seguir = true;
            }
        }

        Process::Coordinador
        {
            while(true)
            {
                <await(seguir);>
                seguir = false;
                proximo++;
            }
        }

        ```

---

6. _Dada la siguiente solución para el Problema de la Sección Crítica entre dos procesos (suponiendo que tanto SC como SNC son segmentos de código finitos, es decir que terminan en algún momento), indicar si cumple con las 4 condiciones requeridas:_

    ```c
    int turno = 1;
    
    Process::SC1
    { 
        while (true)
        {
            while (turno == 2) skip;
            SC;
            turno = 2;
            SNC;
        }
    }
    
    Process::SC2
    { 
        while (true)
        { 
            while (turno == 1) skip;
            SC;
            turno = 1;
            SNC;
        }
    }
    ```

    Condiciones de la sección crítica:
        * **Exclusión mútua**: _A lo sumo un proceso está en su sección crítica_. Cumple, ya que la condición para entrar a la sección crítica sólo se llega a cumplir cuando el otro proceso sale.
        * **Ausencia de Deadlock**: _Si dos o más procesos tratan de entrar a la sección crítica, al menos uno tendrá éxito_. Cumple, ya que cuando inicia la ejecución del programa, SC2 tiene las condiciones para entrar a la SC, y luego se alternan infinitamente.
        * **Ausencia de demoras innecesarias**: _Si un proceso trata de entrar a la SC y los demás están en sus SNC o terminaron, el primero no está impedido a entrar a su SC_. Cumple, ya que apenas los procesos salen de su sección no crítica, cambian la condición, permitiéndole al otro entrar a su SC.
        * **Eventual entrada**: _Un proceso que intenta entrar a su SC tiene posibilidades de hacerlo (eventualmente lo hará)_. Cumple, las SC son finitas, y las condiciones de entrada para cada proceso son verdaderas con infinita recurrencia, o sea, siempre pueden volver a entrar eventualmente.

---

7. _Desarrolle una solución de grano fino usando sólo variables compartidas (no se puede usar las sentencias await ni funciones especiales como TS o FA). En base a lo visto en la clase 3 de teoría, resuelva el problema de acceso a sección crítica usando un proceso coordinador. En este caso, cuando un proceso SC\[i\] quiere entrar a su sección crítica le avisa al coordinador, y espera a que éste le dé permiso. Al terminar de ejecutar su sección crítica, el proceso SC\[i\] le avisa al coordinador. Nota: puede basarse en la solución para implementar barreras con “Flags y coordinador” vista en la teoría 2._

    ```c
    bool seguir = true; int continuar[0..N-1] = ([N] = 0); int turno = 0;

    Process::Worker[id: 0..N-1]
    {
        while(true)
        {
            while (continuar[id] == 0) skip;
            continuar[id] = 0;
            // Realiza tarea 
            seguir = true;
        }
    }

    Process::Coordinador
    {
        while(true)
        {
            while (not seguir) skip;
            seguir = false;
            continuar[turno] = 1;
            turno = (turno + 1) mod (N - 1);
        }
    }
    ```
