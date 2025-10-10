# Practica 1

### Variables compartidas

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
        Process Productor:
        {
            while(true)
            {
                <await(cant < N); buffer[pri_vacio] = elemento; cant++;>
                pri_vacio = (pri_vacio + 1) mod N;
            }
        }

        Process Consumidor:
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
    
    Process Consumidor[id: 0..N-1]
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
        
        Process persona[id = 0..N-1]
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

        Process Persona[id = 0..N-1]
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

        Process Persona[id = 0..N-1]
        {
            <await(id == turno);>
            Imprimir(documento);
            turno++;
        }
        ```

    4. _Modifique la solución de (b) para el caso en que además hay un proceso Coordinador que le indica a cada persona que es su turno de usar la impresora._
