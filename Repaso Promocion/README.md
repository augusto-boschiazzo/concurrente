# Parcial Teórico - Segundo semestre 2025

1. _Conceptos generales_

    1. _Definir los conceptos de concurrencia y paralelismo._

        La concurrencia es la capacidad para ejecutar múltiples tareas de forma simultanea o en paralelo. Éste concepto de software no se ve limitado a una arquitectura de hardware específica, y es independiente del número de procesadores. El paralelismo consiste de la ejecución de tareas en paralelo a través de múltiples elementos de procesamiento. De esta forma, un sistema paralelo es concurrente, pero uno concurrente no es necesariamente paralelo.

        Los procesos en sistemas concurrentes que no son paralelos comparten tiempo de CPU a través de time slicing u otras técnicas, y el sistema operativo se encarga de controlar y planificar los procesos (cuando el slice expira, el proceso bloquea y el SO realiza el context switch).

    2. _Definir el concepto de sincronización entre procesos y explicar cuáles son los dos tipos de sincronización en que se clasifica._

        Los procesos deben sincronizarse durante su ejecución, y esta sincrinización puede ser de dos tipos: por condición, y por exclusión mutua.

        La sincronización por exclusión mútua consiste en asegurar que un único proceso tenga acceso a un recurso compartido en un instante determinado de tiempo. Permite evitar que dos procesos se encuentren en una sección crítica al mismo tiempo.

        La sincronización por condición permite bloquear un proceso hasta que se de una situación determinada.

        En un problema de productor consumidor en el que se comparte un buffer de memoria, la exclusión mutua permite evitar que los procesos accedan al buffer al mismo tiempo, mientras que la sincronización por condición permite que el consumidor no lea el buffer hasta que el productor no termine de escribirlo.

2. _Problema de la sección crítica_

    1. _Indicar y explicar las propiedades que deben garantizarse en la administración de una sección crítica entre procesos concurrentes._

        Las propiedades que se deben garantizar para administrar la sección crítica son:

        - Exclusión mutua: A lo sumo un proceso se encuentra en su sección crítica
        - Ausencia de Deadlock: Si dos o más procesos tratan de entrar en su sección crítica, al menos uno lo hará
        - Ausencia de demora innecesaria: Si sólo un proceso intenta entrar en su sección crítica (los demás están en su sección no crítica o terminaron), entonces no debe estar impedido a hacerlo
        - Eventual entrada: Los procesos que intentan entrar en su sección crítica tienen posibilidades de hacerlo (eventualmente lo harán)

    2. _Dados los siguientes 5 ejemplos de soluciones a este problema, para cada uno indicar si es correcto, y en caso contrario indicar cuál/es propiedades no cumple y ¿por qué? (suponga que SC y SNC terminan su ejecución)._

    ```c
    bool aviso[2] = ([2] false);

    Process Worker[id: 0..1] {
        while (true) 
        {
            SNC;
            aviso[id] = true;
            while (aviso[1-id]) skip;
            SC;
            aviso[id] = false;
        }
    }
    ```

    Éste ejemplo no cumple con la ausencia de deadlocks, ya que puede ocurrir que ambos procesos hayan logrado inicializar la variable aviso en true al momento de llegar al while que comprueba la condición, quedanose ambos atascados. 

    ---

    ```c
    cola c;

    Process Worker[id: 0..99] {
        int aux;
        while (true)
        {
            SNC;
            push(c,id);
            while(top(c) != id) skip;
            SC;
            pop(c,aux);
        }
    }
    ```

    No cumple la eventual entrada, ya que puede ocurrir que un proceso que llegó inicialmente a la pila deba esperar a que los demás terminen, pero nunca llegue a entrar porque los procesos que vienen después se ponen adelante de él.

    ---

    ```c
    int turno = 0;

    Process Worker[id: 0..99] 
    {
        SNC;
        while (turno != id) skip;
        SC;
        turno = turno + 1;
    }
    ```

    Éste ejemplo cumple con todas las propiedades. El proceso con el id igual al turno entrará, no hay deadlock; eventualmente todos los procesos tendran el turno; no se demoran una vez que tienen su turno; sólo el proceso que tiene su turno estará en la sección crítica, y avanzará el turno cuando salga.

    ---

    ```c
    bool ocupado = false;

    Process Worker[id: 0..99]
    {
        while (true)
        {
            SNC;
            while(ocupado) skip;
            ocupado = true;
            SC;
            ocupado = false;
        }
    }
    ```

    No cumple la exclusión mútua porque puede ocurrir que un proceso pase el while y salga del procesador, y luego otro proceso pase también el while. De esta forma, ambos estarían en la sección crítica.

    ---

    ```c
    bool aviso[100] = ([100] false);
    int act = -1;

    Process Worker[id: 0..99]
    {
        while(true)
        {
            SNC;
            aviso[id] = true;
            while(aviso[id]) skip;
            SC;
            act = -1;
        }
    }

    Process Admin
    {
        while(true)
        {
            act = 0;
            while (not aviso[act])
                act = (act + 1) mod 100;
            aviso[act] = false;
            while (act != -1) skip;
        }
    }
    ```

    No cumple la eventual entrada. Por cómo se usa la variable act, en cada repetición el admin vuelve a comprobar siempre desde el primer proceso, los últimos procesos podrían no alcanzar su turno nunca.

3. _Semáforos y monitores_

    1. _Indicar cuál es la diferencia entre los semáforos generales (los usados en la práctica) y los binarios en cuanto a su funcionalidad. Muestre cómo se define el funcionamiento de ambos por medio de las sentencias AWAIT._

        Los semáforos son una tipo de datos abstracto u objeto que permite usar exclusión mútua. Poseen una variable interna con un valor enterno no negativo que lleva cuenta de cuántas veces más se puede desbloquear el semáforo usando la operación P ("proberen" - testear en holandés) y la operación que permite liberarlo una vez terminada la SC, con V ("verhogen" - incrementar). 

        Los semáforos binarios sólo pueden tomar un valor de 1 o 0, bloqueado y desbloqueado. Se usan para la exclusión mútua, sólo un proceso lo bloquea por vez.

        Los semáforos generales o counting semaphores toman valores superiores a 1, con lo cual se pueden bloquear más de una vez, y se les puede realizar una incrementación una cantidad arbitraria de veces.

        Para implementar las operaciones de los semáforos se usa el siguiente código:

        P:
        ```c
        <await (free > 0); free = free - 1;>
        ```

        V:
        ```c
        <free = free + 1;>
        ```

        Éste código se usa para semáforos generales. En su lugar, para los binarios se puede usar un valor booleano en la variable free y que tome el valor falso cuando se supera el await de P, y verdadero cuando se llama a V.

    2. _Indicar qué diferencias existen entre las disciplinas de señalización "Signal and wait" y "Signal and continue" en monitores._

        La disciplina "Signal and wait" implica que cuando un proceso llama a la operación signal o signal_all pasa a competir por el uso del monitor, y le da el control del monitor al proceso despertado. En su lugar, con monitores "signal and continue", el proceso que realiza el signal continúa con el uso del monitor hasta que termina o realiza un wait, y es entonces que el proceso despertado compite por el monitor para continuar ejecutando. En ambas, el proceso despertado continúa por la instrucción siguiente a la del wait en la que se durmió.

4. _Programación distribuida_

    1. _Rendezvous y RPC: indicar cuál es la principal característica de estas dos herramientas que lo diferencian de los pasajes de mensajes. Indicar qué cosas de la comunicación guardada en rendezvous "conceptual" no se tienen en el rendezvous provisto por ADA._

        Las características que diferencian RPC y Rendezvous de pasaje de mensajes son la comunicación bidireccional, que permite que en una misma comunicación se envíen y reciban datos en el proceso llamador y el que recibe la invocación, y que los procesos llamadores quedan bloqueados al realizar la comunicación hasta que se ejecuta y retorna un resultado, como si se realizara una llamada a un proceso local.

        La comunicación guardada en rendezvous conceptual hace uso de la condición (con forma in proc (formales) and B) que pueden hacer uso de los parámetros formales, y de una expresión de scheduling que permite que se atiendan los llamados con un órden explícito. En ADA, todos los llamados se atienden por orden de llegada y se debe manejar de forma explícita todo orden distinto.

    2. _Suponga n^2 procesos organizados en forma de grilla cuadrada. Cada proceso puede comunicarse sólo con los vecinos izquierdo, derecho, de arriba y de abajo (los procesos de las esquinas tienen sólo 2 vecinos, y los otros en los bordes tienen 3 vecinos). Cada proceso tiene inicialmente un valor local v. Escriba un algoritmo heartbeat que calcule el máximo de los n^2 valores. Al terminar el programa, cada proceso debe conocer ese valor._

        Se tienen los n^2 procesos. Cada uno realizará 2(n-1) iteraciones en las que envía su máximo local (inicialmente es el mismo valor del propio proceso) a todos sus vecinos, y luego recibe los máximos de los mismos. La cantidad de iteraciones es porque representa la máxima distancia entre procesos opuestos, y por lo tanto, es la máxima cantidad de iteraciones que el valor máximo va a tardar en llegar a cualquier parte de la grilla.

        ```c
        chan valores[n][n](int);

        Process Vecino[i: 0..n][j: 0..n]
        {
            int v;
            int aux;
            int max = v;

            for (int k = 0; k < 2 * n - 1; k++)
            {
                if (i > 0)
                    send valores[i-1][j](max);
                if (i < n)
                    send valores[i+1][j](max);
                if (j > 0)
                    send valores[i][j-1](max);
                if (j < n)
                    send valores[i][j+1](max);

                
                if (i > 0)
                {
                    receive valores[i-1][j](aux);
                    if (aux > max)
                        max = aux;
                }
                if (i < n)
                {
                    receive valores[i+1][j](aux);
                    if (aux > max)
                        max = aux;
                }
                if (j > 0)
                {
                    send valores[i][j-1](aux);
                    if (aux > max)
                        max = aux;
                }
                if (j < n)
                {
                    send valores[i][j+1](aux);
                    if (aux > max)
                        max = aux;
                }
            }
        }
        ```

5. _Métricas en sistemas paralelos_

    _Dado el siguiente código usando P procesos en paralelo con variables compartidas, y suponiendo que se cuentan con P procesadores (cada procesador ejecutará un proceso)._ 
    
    ```c
    int datos[n];

    Process Worker[id: 0..P-1]
    {
        int desde = id * (n/P);
        int hasta = (id + 1) * (n/P);

        for (int i = desde; i < hasta; i++)
            for (int j = 0; j < n; j++)
                ProcesarDato(i,j,datos[i]);
    }
    ```

    _Suponiendo que n = 1000, que P = 5 y el tiempo para ejecutar la función ProcesarDato en cada procesador es de 2 minutos (P0 y P1), 1 minuto (P2) y 4 minutos (P3 y P4). Se debe calcular (para hacer los cálculos sólo tenga en cuenta el tiempo de ejecución de ProcesarDato):_
    
    1. _Realizar paso a paso el cálculo del tiempo secuencial y el tiempo paralelo._

        Tiempo secuencial:

        ```
        Cada worker: (hasta - desde) * n llamados a ProcesarDato -> para id 0: desde = 0 * (1000/5), hasta = 1 * 1000 / 5 -> 200 * 1000 = 200000 llamados de cada proceso
        En total: 200000 * 5 = 1000000 de llamados.
        Tiempo: Usando el mejor procesador (1 minuto por llamada) = 1000000 minutos Ts
        ```

        Tiempo Concurrente

        ```
        Con la cantidad de llamados por proceso definida anteriormente, tomamos el procesador más lento (ésto es porque el algoritmo divide de forma igualitaria la cantidad de operaciones para todos los procesadores), por lo tanto suponemos 4 minutos.
        Tiempo: 200000 * 4 minutos = 800000 minutos Tp
        ```

    2. _Realizar paso a paso el cálculo del speedup y la eficiencia._

        El speedup se define como el tiempo del algoritmo secuencial dividido por el tiempo del paralelo:

        ```
        S = Ts/Tp = 1000000 / 800000 = 1.25
        ```

        Sin embargo, el speedup óptimo es de la forma:

        ```
        S óptimo = Sum(i=0, P, PotenciaCalculo(i)/potenciaCalculo(mejor)) = (1/2 + 1/2 + 1 + 1/4 + 1/4) / 1 = 2.5
        ```

        La eficiencia es la siguiente:

        ```
        E = S / S óptimo = 1.25 / 2.5 = 0.5
        ```

    3. _Explicar detalladamente cómo modificaría la solución para lograr una mejor eficiencia, recalculando la misma._

        La eficiencia actualmente es de un 50%. Se puede mejorar redistribuyendo la carga de trabajo según la potencia de cada procesador. Para ello:

        ```
        Cantidad de iteraciones para P2 = 1 / 2.5 * 1000000 = 400000
        Cantidad de iteraciones para P0 y P1 = 0.5 / 2.5 * 1000000 = 200000
        Cantidad de iteraciones para P3 y P4 = 0.25 / 2.5 * 1000000 = 100000
        ```

        La idea es que todos los procesadores terminen a la vez y ninguno esté ocioso. De esta forma, el Tp para este algoritmo es de 400000, que es el tiempo que a todos los procesadores les va a tomar terminar.