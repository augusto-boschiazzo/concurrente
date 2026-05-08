# Practica 3

### Monitores

---

1. _Se dispone de un puente por el cual puede pasar un solo auto a la vez. Un auto pide permiso para pasar por el puente, cruza por el mismo y luego sigue su camino._

---

2. _Existen N procesos que deben leer información de una base de datos, la cual es administrada por un motor que admite una cantidad limitada de consultas simultáneas._
    
    1. _Analice el problema y defina qué procesos, recursos y monitores/sincronizaciones serán necesarios/convenientes para resolverlo._


    2. _Implemente el acceso a la base por parte de los procesos, sabiendo que el motor de base de datos puede atender a lo sumo 5 consultas de lectura simultáneas._

    
    ```c
    Monitor::AccesoDB
    {
        int nw = 0, int nr = 0, dr = 0;
        cond ok_leer, ok_escribir;

        procedure pedidoLeer()
        {
            if (nw > 0 OR nr == 5){
                dr++;
                wait(ok_leer);
            }
            else
                nr++;
        }

        procedure liberarLeer()
        {
            nr--;
            if (nr == 0 AND dw > 0)
            {
                dw--;
                signal(ok_escribir);
                nw++;
            }
        }

        procedure pedidoEscribir()
        {
            if (nr > 0 OR nw > 0)
            {
                dw++;
                wait(ok_escribir);
            }
            else
                nw++;
        }

        procedure liberarLeer()
        {
            if (dw > 0)
            {
                dw--;
                signal(ok_escribir);
            }
            else
            {
                nw--;
                if (dr > 0)
                {
                    nr = dr;
                    dr = 0;
                    signal_all(ok_leer);
                }
            }
        }
    }
    ```

    > _Ignorar lo que hice arriba, pensé que decía que tenía que haber escritores también. Lo dejo porque es una solución razonable al lector-escritor._

    ```c
    Monitor::AccesoDB
    {
        int nr = 0;
        cond ok_leer;

        procedure pedidoLeer()
        {
            if (nr == 5) wait(ok_leer);
            nr++;
        }

        procedure liberarLeer()
        {
            nr--;
            signal(ok_leer);
        }
    }

    Process::Lector[id: 0..N-1]
    {
        AccesoDB.pedidoLeer();
        leer();
        AccesoDB.liberarLeer();
    }
    ```

---

3. _Existen N personas que deben fotocopiar un documento. La fotocopiadora sólo puede ser usada por una persona a la vez. Analice el problema y defina qué procesos, recursos y monitores serán necesarios/convenientes, además de las posibles sincronizaciones requeridas para resolver el problema. Luego, resuelva considerando las siguientes situaciones:_

    1. _Implemente una solución suponiendo no importa el orden de uso. Existe una función Fotocopiar() que simula el uso de la fotocopiadora._

    ```c
    Monitor::Fotocopiadora
    {
        procedure fotocopiar(file archivo)
        {
            Fotocopiar(archivo);
        }
    }

    Process::Persona[id: 0..N-1]
    {
        file archivo;
        Fotocopiadora.Fotocopiar(archivo);
    }
    ```

    2. _Modifique la solución de (a) para el caso en que se deba respetar el orden de llegada._

    ```c
    Monitor::Fotocopiadora
    {
        bool ocupada = false;
        cond usarFotocopiadora;
        int esperando = 0;

        procedure usarFotocopiadora()
        {
            if (ocupada)
            {   
                esperando++;
                wait(usarFotocopiadora);
            }
            else
            {
                ocupada = true;
            }
        }

        procedure liberarFotocopiadora()
        {
            if (esperando > 0)
            {
                esperando--;
                signal(usarFotocopiadora);
            }
            else
                ocupada = false;
        }
    }

    Process::Persona[id: 0..N-1]
    {
        file archivo;

        Fotocopiadora.usarFotocopiadora();
        Fotocopiar(archivo);
        Fotocopiadora.liberarFotocopiadora();
    }
    ```

    3. _Modifique la solución de (b) para el caso en que se deba dar prioridad de acuerdo con la edad de cada persona (cuando la fotocopiadora está libre la debe usar la persona de mayor edad entre las que estén esperando para usarla)._

    ```c
    Monitor::Fotocopiadora
    {
        bool ocupada = false;
        cond usarFotocopiadora[N];
        int esperando = 0, idSiguiente;
        cola colaPrioritaria;

        procedure usarFotocopiadora(in int id, edad)
        {
            if (ocupada)
            {   
                esperando++;
                insertarOrdenado(colaPrioritaria, id, edad);
                wait(usarFotocopiadora[id]);
            }
            else
            {
                ocupada = true;
            }
        }

        procedure liberarFotocopiadora()
        {
            if (esperando > 0)
            {
                esperando--;
                idSiguiente = colaPrioritaria.pop();
                signal(usarFotocopiadora[idSiguiente]);
            }
            else
                ocupada = false;
        }
    }

    Process::Persona[id: 0..N-1]
    {
        file archivo;
        int edad;
        
        Fotocopiadora.usarFotocopiadora(id, edad);
        Fotocopiar(archivo);
        Fotocopiadora.liberarFotocopiadora();
    }
    ```

    4. _Modifique la solución de (a) para el caso en que se deba respetar estrictamente el orden dado por el identificador del proceso (la persona X no puede usar la fotocopiadora hasta que no haya terminado de usarla la persona X-1)._

    ```c
    Monitor::Fotocopiadora
    {
        int proximo = 0;
        cond usarFotocopiadora[N];

        procedure usarFotocopiadora(id)
        {
            if (id != proximo)
                wait(usarFotocopiadora[id]);
        }

        procedure liberarFotocopiadora()
        {
            proximo++;
            signal(usarFotocopiadora[proximo]);
        }
    }

    Process::Persona[id: 0..N-1]
    {
        file archivo;
        
        Fotocopiadora.usarFotocopiadora(id);
        Fotocopiar(archivo);
        Fotocopiadora.liberarFotocopiadora();
    }
    ```

    5. _Modifique la solución de (b) para el caso en que además haya un Empleado que le indica a cada persona cuando debe usar la fotocopiadora._
    
    ```c
    Monitor::Fotocopiadora
    {
        bool ocupada = false;
        cond usarFotocopiadora;
        int esperando = 0;

        procedure usarFotocopiadora()
        {
            if (ocupada)
            {   
                esperando++;
                wait(usarFotocopiadora);
            }
            else
            {
                ocupada = true;
            }
        }

        procedure liberarFotocopiadora()
        {
            if (esperando > 0)
            {
                esperando--;
                signal(usarFotocopiadora);
            }
            else
                ocupada = false;
        }
    }

    Process::Persona[id: 0..N-1]
    {
        file archivo;

        Fotocopiadora.usarFotocopiadora();
        Fotocopiar(archivo);
        Fotocopiadora.liberarFotocopiadora();
    }
    ```

    6. _Modificar la solución (e) para el caso en que sean 10 fotocopiadoras. El empleado le indica a la persona cuál fotocopiadora usar y cuándo hacerlo._