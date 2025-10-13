# Practica 2

### Semáforos

---

1. _Existen N personas que deben ser chequeadas por un detector de metales antes de poder ingresar al avión._

    1. _Analice el problema y defina qué procesos, recursos y semáforos/sincronizaciones serán necesarios/convenientes para resolverlo._

        Se usará un semáforo para permitir el ingreso al detector, al igual que un proceso persona que accede al detector con el semáforo. La persona pide acceso al detector con P, usa el detector cuando el semáforo está en uno, y luego lo libera.

    2. _Implemente una solución que modele el acceso de las personas a un detector (es decir, si el detector está libre la persona lo puede utilizar; en caso contrario, debe esperar)._

        ```c
        sem s = 1;

        Process::Persona[id: 0..N-1]
        {
            P(s);
            //Pasar por detector;
            V(s);
        }
        ```

    3. _Modifique su solución para el caso que haya tres detectores._

        ```c
        sem s = 3;

        Process::Persona[id: 0..N-1]
        {
            P(s);
            //Pasar por detector;
            V(s);
        }
        ```

    4. _Modifique la solución anterior para el caso en que cada persona pueda pasar más de una vez, siendo aleatoria esa cantidad de veces._

        ```c
        sem s = 3;

        Process::Persona[id: 0..N-1]
        {
            int cant = rand() mod 10 + 1;
            for (int i = 0; i < cant; i++)
            {
                P(s);
                //Pasar por detector;
                V(s);
            }
        }
        ```

---

2. _Un sistema de control cuenta con 4 procesos que realizan chequeos en forma colaborativa. Para ello, reciben el historial de fallos del día anterior (por simplicidad, de tamaño N). De cada fallo, se conoce su número de identificación (ID) y su nivel de gravedad (0=bajo, 1=intermedio, 2=alto, 3=crítico). Resuelva considerando las siguientes situaciones:_

    1. _Se debe imprimir en pantalla los ID de todos los errores críticos (no importa el orden)._

        ```c
        sem mutex = 1; colaFallos c[N]; int cant = 0;

        Process::Checker[id: 0..4]
        {
            Fallo fallo;
            P(mutex);
            while (cant < N)
            {
                fallo = c.pop();
                cant++;
                V(mutex);
                if (fallo.gravedad == 3) stdout(fallo.id);
                P(mutex);
            }
            V(mutex);
        }
        ```

    2. _Se debe calcular la cantidad de fallos por nivel de gravedad, debiendo quedar los resultados en un vector global._

        ```c
        sem mutex = 1; colaFallos c[N]; int cant = 0; int cantFallos[4] = ([4] = 0); sem mutexGravedad[4] = ([4] = 0);

        Process::Checker[id: 0..4]
        {
            Fallo fallo;
            P(mutex);
            while (cant < N)
            {
                fallo = c.pop();
                cant++;
                V(mutex);

                P(mutexGravedad[fallo.gravedad]);
                cantFallos[fallo.gravedad]++;
                V(mutexGravedad[fallo.gravedad]);

                P(mutex);
            }
            V(mutex);
        }
        ```

    3. _Ídem b) pero cada proceso debe ocuparse de contar los fallos de un nivel de gravedad determinado._

        ```c
        sem mutex = 1; colaFallos c[N]; int cant = 0; int cantFallos[4] = ([4] = 0);

        Process::Checker[id: 0..4]
        {
            Fallo fallo;
            P(mutex);
            while (cant < N)
            {
                fallo = c.pop();
                cant++;
                V(mutex);

                if (fallo.gravedad != id)
                {
                    P(mutex);
                    c.push(fallo);
                    cant--;
                    V(mutex);
                }
                else
                {
                    cantFallos[id]++;
                }

                P(mutex);
            }
            V(mutex);
        }
        ```

---

3.  _Un sistema operativo mantiene 5 instancias de un recurso almacenadas en una cola. Además, existen P procesos que necesitan usar una instancia del recurso. Para eso, deben sacar la instancia de la cola antes de usarla. Una vez usada, la instancia debe ser encolada nuevamente para su reúso._

    ```c
    sem recurso = 5, acceso = 1; colaRecurso c[5];

    Process::Proceso[0..N-1]
    {
        Recurso recurso;
        while(true)
        {
            P(recurso);
            P(acceso);
            recurso = c.pop();
            V(acceso);
            // usa recurso
            P(acceso);
            c.push(recurso);
            V(acceso);
            V(recurso);
        }
    }
    ```

---

4.  _Suponga que existe una BD que puede ser accedida por 6 usuarios como máximo al mismo tiempo. Además, los usuarios se clasifican como usuarios de prioridad alta y usuarios de prioridad baja. Por último, la BD tiene la siguiente restricción:_
    * _no puede haber más de 4 usuarios con prioridad alta al mismo tiempo usando la BD._
    * _no puede haber más de 5 usuarios con prioridad baja al mismo tiempo usando la BD._ </br>
    _Indique si la solución presentada es la más adecuada. Justifique la respuesta._

    ```c
    sem total = 6, alta = 4, baja= 5;

    Process Usuario-Alta [I:1..L]::
    {
        P (total);
        P (alta);
        //usa la BD
        V(total);
        V(alta);
    }
    Process Usuario-Baja [I:1..K]::
    {
        P (total);
        P (baja);
        //usa la BD
        V(total);
        V(baja);
    }
    ```

    La solución está mal ya que se debería primero esperar a tener disponible uno de los lugares correspondientes al proceso de alta o baja prioridad, porque de no ser así, podrían estar bloqueándole el acceso a procesos que sí podrían entrar. O sea, ya hay cuatro procesos de alta prioridad, y sólo se encuentra uno de baja prioridad. Cuando uno de alta prioridad hace P(total), está quitando uno de los lugares que podría usar uno de baja prioridad. Pero luego, se queda esperando a que uno de los otros usuarios de alta prioridad libere una instancia del recurso, y en ese tiempo, uno de baja prioridad podría haber accedido al recurso sin problemas. Se soluciona haciendo primero P(alta), que hace que sólo intente acceder si puede hacerlo, y luego P(total), que bloquea una instancia para efectivamente acceder a ella. Lo mismo para los usuarios de baja prioridad.

---

5. _En una empresa de logística de paquetes existe una sala de contenedores donde se preparan las entregas. Cada contenedor puede almacenar un paquete y la sala cuenta con capacidad para N contenedores. Resuelva considerando las siguientes situaciones:_

    1. _La empresa cuenta con 2 empleados: un empleado Preparador que se ocupa de preparar los paquetes y dejarlos en los contenedores; un empelado Entregador que se ocupa de tomar los paquetes de los contenedores y realizar la entregas. Tanto el Preparador como el Entregador trabajan de a un paquete por vez._

        ```c
        Contenedor contenedores[0..N-1]; sem dejar[0..N-1] = ([n] = 1), tomar[0..N-1] = ([n] = 0);

        Process::Preparador
        {
            int i = 0;
            Paquete paquete;
            while(true)
            {
                paquete = PrepararPaquete();
                P(dejar[i]);
                contenedores[i] = paquete;
                V(tomar[i]);
                i = (i + 1) mod N;
            }
        }

        Process::Entregador
        {
            int i = 0;
            Paquete paquete;
            while(true)
            {
                P(tomar[i]);
                paquete = contenedores[i];
                V(dejar[i]);
                i = (i + 1) mod N;
                EntregarPaquete(paquete);
            }
        }

        ```

    2. _Modifique la solución a) para el caso en que haya P empleados Preparadores._

        ```c
        Contenedor contenedores[0..N-1]; sem dejar[0..N-1] = ([n] = 1), tomar[0..N-1] = ([n] = 0), dejando = 1; int iP = 0;

        Process::Preparador[id: 0..P-1]
        {
            Paquete paquete;
            while(true)
            {
                paquete = PrepararPaquete();
                P(dejando);
                P(dejar[iP]);
                contenedores[iP] = paquete;
                V(tomar[iP]);
                iP = (iP + 1) mod N;
                V(dejando)
            }
        }

        Process::Entregador
        {
            int i = 0;
            Paquete paquete;
            while(true)
            {
                P(tomar[i]);
                paquete = contenedores[i];
                V(dejar[i]);
                i = (i + 1) mod N;
                EntregarPaquete(paquete);
            }
        }

        ```

    3. _Modifique la solución a) para el caso en que haya E empleados Entregadores._

        ```c
        Contenedor contenedores[0..N-1]; sem dejar[0..N-1] = ([n] = 1), tomar[0..N-1] = ([n] = 0), tomando = 1; int iE = 0;

        Process::Preparador
        {
            int i = 0;
            Paquete paquete;
            while(true)
            {
                paquete = PrepararPaquete();
                P(dejar[i]);
                contenedores[i] = paquete;
                V(tomar[i]);
                i = (i + 1) mod N;
            }
        }

        Process::Entregador[id: 0..E-1]
        {
            Paquete paquete;
            while(true)
            {
                P(tomando);
                P(tomar[iE]);
                paquete = contenedores[iE];
                V(dejar[iE]);
                iE = (iE + 1) mod N;
                V(tomando);
                EntregarPaquete(paquete);
            }
        }

        ```

    4. _Modifique la solución a) para el caso en que haya P empleados Preparadores y E empleadores Entregadores._

    ```c
        Contenedor contenedores[0..N-1]; sem dejar[0..N-1] = ([n] = 1), tomar[0..N-1] = ([n] = 0), dejando = 1, tomando = 1; int iP = 0, iE = 0;

        Process::Preparador[id: 0..P-1]
        {
            Paquete paquete;
            while(true)
            {
                paquete = PrepararPaquete();
                P(dejando);
                P(dejar[iP]);
                contenedores[iP] = paquete;
                V(tomar[iP]);
                iP = (iP + 1) mod N;
                V(dejando)
            }
        }

        Process::Entregador[id: 0..E-1]
        {
            Paquete paquete;
            while(true)
            {
                P(tomando);
                P(tomar[iE]);
                paquete = contenedores[iE];
                V(dejar[iE]);
                iE = (iE + 1) mod N;
                V(tomando);
                EntregarPaquete(paquete);
            }
        }

        ```

---

6. _Existen N personas que deben imprimir un trabajo cada una. Resolver cada ítem usando semáforos:_
    
    1. _Implemente una solución suponiendo que existe una única impresora compartida por todas las personas, y las mismas la deben usar de a una persona a la vez, sin importar el orden. Existe una función Imprimir(documento) llamada por la persona que simula el uso de la impresora. Sólo se deben usar los procesos que representan a las Personas._
    
        ```c
        sem impresora = 1;

        Process::Persona[id: 0..N-1]
        {
            Documento documento;
            while(true)
            {
                P(impresora);
                ImprimirDocumento(documento);
                V(impresora);
            }
        }
        ```

    2. _Modifique la solución de (a) para el caso en que se deba respetar el orden de llegada._
    
        ```c
        sem impresora[0..N-1] = ([n] = 0), continuar = 0, encolando = 1, llena = 0; colaInt cola;

        Process::Persona[id: 0..N-1]
        {
            Documento documento;
            while(true)
            {
                P(encolando);
                cola.push(id);
                V(encolando);
                V(llena);

                P(impresora[id]);
                ImprimirDocumento(documento);
                V(continuar);
            }
        }

        Process::Coordinador
        {
            int siguiente;
            while(true)
            {
                P(llena);
                P(encolando);
                siguiente = cola.pop();
                V(encolando);

                P(continuar);
                V(impresora[siguiente]);
            }
        }
        ```

    3. _Modifique la solución de (a) para el caso en que se deba respetar estrictamente el orden dado por el identificador del proceso (la persona X no puede usar la impresora hasta que no haya terminado de usarla la persona X-1)._
    
        ```c
        sem impresora[0..N-1] = ([n] = 0), continuar = 1;

        Process::Persona[id: 0..N-1]
        {
            Documento documento;
            while(true)
            {
                P(impresora[id]);
                ImprimirDocumento(documento);
                V(continuar);
            }
        }

        Process::Coordinador
        {
            int id = 0;
            while(true)
            {
                P(continuar);
                V(impresora[id]);
                id = (id + 1) mod N;
            }
        }
        ```

    4. _Modifique la solución de (b) para el caso en que además hay un proceso Coordinador que le indica a cada persona que es su turno de usar la impresora._

        (ya lo resolví de esa manera en b)
    
    5. _Modificar la solución (d) para el caso en que sean 5 impresoras. El coordinador le indica a la persona cuando puede usar una impresora, y cual debe usar_

        ```c
        sem impresora[0..N-1] = ([n] = 0), continuar = 0, encolando = 1, llena = 0, comunicando = 1; colaInt cola; int nImpresora;

        Process::Persona[id: 0..N-1]
        {
            Documento documento;
            while(true)
            {
                P(encolando);
                cola.push(id);
                V(encolando);
                V(llena);

                P(impresora[id]);
                ImprimirDocumento(documento, nImpresora);
                V(comunicando);
                V(continuar);
            }
        }

        Process::Coordinador[id: 0..4]
        {
            int siguiente;
            while(true)
            {
                P(continuar);

                P(llena);
                P(encolando);
                siguiente = cola.pop();
                V(encolando);

                P(comunicando)
                nImpresora = id;
                V(impresora[siguiente]);
            }
        }
        ```
