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
                    V(mutex);
                }

                cantFallos[id]++;

                P(mutex);
            }
            V(mutex);
        }
        ```

---
