# Practica 1

### Variables compartidas

---

3. _Dada la siguiente solución de grano grueso:_

   1. _Indicar si el siguiente código funciona para resolver el problema de Productor/Consumidor con un buffer de tamaño N. En caso de no funcionar, debe hacer las modificaciones necesarias_

        ```c
        int cant = 0; int pri_ocupada = 0; int pri_vacia = 0; int buffer[N];
    
        Process Productor::
        { 
            while (true)
            { // produce elemento
                <await (cant < N); cant++>
                buffer[pri_vacia] = elemento;
                pri_vacia = (pri_vacia + 1) mod N;
            }
        }

        Process Consumidor::
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
    
    
