# Practica 4

### PMA

1. _Suponga que N clientes llegan a la cola de un banco y que serán atendidos por sus empleados. Analice el problema y defina qué procesos, recursos y comunicaciones serán necesarios/convenientes para resolver el problema. Luego, resuelva considerando las siguientes situaciones:_
   
   1. _Existe un único empleado, el cual atiende por orden de llegada._

      ```c
      chan atencion(string);

      process Cliente[id: 0..N-1] {
        string solicitud = ...;
        send atencion(solicitud);
      }

      process Empleado {
        string solicitud;
        while (true) {
            solicitud = receive atencion(solicitud);
            AtenderSolicitud(solicitud);
        }
      }
      ```

   2. _Ídem a) pero considerando que hay 2 empleados para atender, ¿qué debe modificarse en la solución anterior?_

      ```c
      chan atencion(string);

      process Cliente[id: 0..N-1] {
        string solicitud = ...;
        send atencion(solicitud);
      }

      process Empleado[id: 0..2-1] {
        string solicitud;
        while (true) {
            solicitud = receive atencion(solicitud);
            AtenderSolicitud(solicitud);
        }
      }
      ```

   3. _Ídem b) pero considerando que, si no hay clientes para atender, los empleados realizan tareas administrativas durante 15 minutos. ¿Se puede resolver sin usar procesos adicionales? ¿Qué consecuencias implicaría?_

      ```c
      chan atencion(string);
      chan solicitarTrabajo(int);
      chan recibirTrabajo[2](string);

      process Cliente[id: 0..N-1] {
        string solicitud = ...;
        send atencion(solicitud);
      }

      process Coordinador {
        int idEmpleado;
        string solicitud;
        while (true) {
            receive solicitarTrabajo(idEmpleado);
            if (not empty(atencion))
                receive atencion(solicitud);
            else
                solicitud = "VACIO";
            send recibirTrabajo[idEmpleado](solicitud);
        }
      }

      process Empleado[id: 0..2-1] {
        string solicitud;
        while (true) {
            send solicitarTrabajo(id);
            receive recibirTrabajo[id](solicitud);
            if (solicitud != "VACIO") 
                AtenderSolicitud(solicitud);
            else
                delay(900);
        }
      }
      ```

      Se requiere un coordinador para evitar problemas a la hora de que los empleados realicen un ```empty``` sobre el canal, ya que no es una operación que garantice que a la hora de realizar las siguientes instrucciones el canal se mantenga lleno. O sea, la operación ```empty``` sólo es segura cuando un único proceso recibe mensajes por ese canal. Para eso, se introduce un nuevo proceso coordinador que recibe las solicitudes (es un único proceso recibiendo los mensajes), que les envía las solicitudes a los empleados en canales propios.

2. _Se desea modelar el funcionamiento de un banco en el cual existen 5 cajas para realizar pagos. Existen P clientes que desean hacer un pago. Para esto, cada uno selecciona la caja donde hay menos personas esperando; una vez seleccionada, espera a ser atendido. En cada caja, los clientes son atendidos por orden de llegada por los cajeros. Luego del pago, se les entrega un comprobante. **Nota**: maximizar la concurrencia._

    ```c
    chan caja[5](int, string);
    chan llegada(int);
    chan cajaAsignada[P](int);
    chan comprobantes[P](int);
    chan salida(int);

    process Cliente[id: 0..P-1] {
        int cajaId;
        string solicitud = ...;
        string comprobante;

        send llegada(id);
        receive cajaAsignada[id](cajaId);
        send caja[cajaId](id, solicitud);
        receive comprobantes[id](comprobante);
    }

    process Coordinador {
        int idCliente;
        int idCaja;
        int genteEnCola[5] = ([5] 0);

        while (true) {
            if (not empty(llegada) && empty(salida)) {
                receive llegada(idCliente);
                idCaja = min(genteEnCola);
                genteEnCola[idCaja]++;
                send cajaAsignada[idCliente](idCaja);
            }
            else if (not empty(salida)) {
                receive salida(idCaja);
                genteEnCola[idCaja]--;
            }
        }
    }

    process Caja[id: 0..5-1] {
        int idCliente;
        string solicitud;
        string comprobante;

        while (true) {
            receive caja[id](idCliente, solicitud);
            comprobante = AtenderSolicitud(solicitud);
            send comprobantes[idCliente](comprobante);
            send salida(id);
        }
    }
    ```

3. _Se debe modelar el funcionamiento de una casa de comida rápida, en la cual trabajan 2 cocineros y 3 vendedores, y que debe atender a C clientes. El modelado debe considerar que:_

    - _Cada cliente realiza un pedido y luego espera a que se lo entreguen._
    - _Los pedidos que hacen los clientes son tomados por cualquiera de los vendedores y se lo pasan a los cocineros para que realicen el plato. Cuando no hay pedidos para atender, los vendedores aprovechan para reponer un pack de bebidas de la heladera (tardan entre 1 y 3 minutos para hacer esto)._
    - _Repetidamente cada cocinero toma un pedido pendiente dejado por los vendedores, lo cocina y se lo entrega directamente al cliente correspondiente._

    > _Nota: maximizar la concurrencia._

    ```c
    chan pedidos(int, string);
    chan pedirTrabajo(int);
    chan antencionPedido[3](int, string);
    chan cocinaPedido(int, string);
    chan recibirPedido[C](string);

    process Cliente[id: 0..C-1] {
        string pedido = ...;
        string resultado;

        send pedidos(id, pedido);
        receive recibirPedido[id](resultado);
    }

    process SistemaPedidos {
        int idCliente;
        int idVendedor;
        string pedido;

        while (true) {
            receive pedirTrabajo(idVendedor);
            if (not empty(pedidos))
                receive pedidos(idCliente, pedido);
            else {
                pedido = "VACIO";
                idCliente = -1;
            }
            send atencionPedido[idVendedor](idCliente, pedido);
        }
    }

    process Vendedor[id: 0..3-1] {
        int idCliente;
        string pedido;

        while (true) {
            send pedirTrabajo(id);
            receive atencionPedido[id](idCliente, pedido);
            if (pedido != "VACIO")
                send cocinaPedido(idCliente, pedido);
            else
                delay(((rand() % 3) + 1) * 60);
        }
    }

    process Cocinero[id: 0..2-1] {
        int idCliente;
        string pedido;
        string resultado;

        while (true) {
            receive cocinaPedido(idCliente, pedido);
            resultado = cocinarPedido(pedido);
            send recibirPedido[idCliente](resultado);
        }
    }
    ```

4. _Simular la atención en un locutorio con 10 cabinas telefónicas, el cual tiene un empleado que se encarga de atender a N clientes. Al llegar, cada cliente espera hasta que el empleado le indique a qué cabina ir, la usa y luego se dirige al empleado para pagarle. El empleado atiende a los clientes en el orden en que hacen los pedidos, pero siempre dando prioridad a los que terminaron de usar la cabina. A cada cliente se le entrega un ticket factura por la operación._

    > _Nota: maximizar la concurrencia; suponga que hay una función Cobrar() llamada por el empleado que simula que el empleado le cobra al cliente._

    ```c
    chan llegada(int);
    chan entrarACabina[N](int);
    chan salida(int, int);
    chan recibirTicket[N](string);
    chan cobro(string);

    process Cliente[id: 0..N-1] {
        int idCabina;
        string ticket;

        send llegada(id);
        receive entrarACabina[id](idCabina);
        // usa cabina
        send salida(id, idCabina);
        receive recibirTicket[id](ticket);
        send cobro(ticket);
    }

    process Empleado {
        int idCliente;
        int idCabina;
        bool cabinaLibre [10] = ([10] true);
        string ticket;

        while (true) {
            if (empty(salida) && not empty(llegada) && any(cabinaLibre)) -> // devuelve true si alguno es true
                receive llegada(idCliente);
                idCabina = first(cabinaLibre); // first() retorna la primera que está libre
                cabinaLibre[idCabina] = false;
                send entrarACabina[idCliente](idCabina);
            [] (empty(llegada)) -> 
                if (not empty(salida)) ->
                    receive salida(idCliente, idCabina);
                    cabinaLibre[idCabina] = true;
                    ticket = generarTicket(idCabina);
                    send recibirTicket[idCliente](ticket);
                [] (not empty(cobro))  ->
                    receive cobro(ticket);
                    Cobrar(ticket);
        }
    }
    ```

5. _Resolver la administración de 3 impresoras de una oficina. Las impresoras son usadas por N administrativos, los cuales están continuamente trabajando y cada tanto envían documentos a imprimir. Cada impresora, cuando está libre, toma un documento y lo imprime, de acuerdo con el orden de llegada._

    > _Nota: ni los administrativos ni el director deben esperar a que se imprima el documento._

    1. _Implemente una solución para el problema descrito._

        ```c
        chan imprimir(string);
        
        process Administrativo[id: 0..N-1] {
            string documento;

            while (true) {
                documento = Trabajar();
                send imprimir(documento);
            }
        }

        process Impresora[id: 0..3-1] {
            string documento; 

            while (true) {
                receive imprimir(documento);
                Imprimir(documento);
            }
        }
        ```
    
    2. _Modifique la solución implementada para que considere la presencia de un director de oficina que también usa las impresas, el cual tiene prioridad sobre los administrativos._

        ```c
        chan imprimir(string);
        chan imprimirDirector(string);
        
        process Administrativo[id: 0..N-1] {
            string documento;

            while (true) {
                documento = Trabajar();
                send imprimir(documento);
            }
        }

        process Director {
            string documento;

            while (true) {
                documento = Trabajar();
                send imprimirDirector(documento);
            }
        }

        process Impresora[id: 0..3-1] {
            string documento; 

            while (true) {
                if (empty(imprimirDirector) && not empty(imprimir)) {
                    receive imprimir(documento);
                    Imprimir(documento);
                }
                else if (not empty(imprimirDirector)) {
                    receive imprimirDirector(documento);
                    Imprimir(documento);
                }
            }
        }
        ```
    
    3. _Modifique la solución (a) considerando que cada administrativo imprime 10 trabajos y que todos los procesos deben terminar su ejecución._

    ```c
    
    ```

    4. _Modifique la solución (b) considerando que tanto el director como cada administrativo imprimen 10 trabajos y que todos los procesos deben terminar su ejecución._

    5. _Si la solución al ítem d) implica realizar Busy Waiting, modifíquela para evitarlo._

---

### PMS

1. _Suponga que existe un antivirus distribuido que se compone de R procesos robots Examinadores y 1 proceso Analizador. Los procesos Examinadores están buscando continuamente posibles sitios web infectados; cada vez que encuentran uno avisan la dirección y luego continúan buscando. El proceso Analizador se encarga de hacer todas las pruebas necesarias con cada uno de los sitios encontrados por los robots para determinar si están o no infectados._

    1. _Analice el problema y defina qué procesos, recursos y comunicaciones serán necesarios/convenientes para resolver el problema._

    2. _Implemente una solución con PMS sin tener en cuenta el orden de los pedidos._

        ```c
        process Examinador[id: 0..R-1] {
            string sitio;
            while(true) {
                    sitio = BuscarSitioInfectado();
                    Analizador!(sitio);
            }
        }

        process Analizador {
            string sitio;
            while(true) {
                Examinador?(sitio);
                AnalizarSitio(sitio);
            }
        }
        ```

        El problema de esta solución es que, más allá del orden de llegada, los examinadores pasan mucho tiempo esperando para enviar su sitio y seguir buscando otros.

    3. _Modifique el inciso (b) para que el Analizador resuelva los pedidos en el orden en que se hicieron._

        ```c
        process Examinador[id: 0..R-1] {
            string sitio;
            while(true) {
                sitio = BuscarSitioInfectado();
                Coordinador!(sitio);
            }
        }

        process Coordinador {
            cola sitios;
            string sitio;
            while(true) {
                do Examinador[*]?(sitio) -> push(sitios, sitio);
                [] not empty(sitios); Analizador?() -> Analizador!(pop(sitios));
                od
            }
        }

        process Analizador {
            string sitio;
            while(true) {
                Coordinador!(); //pide trabajo
                Coordinador?(sitio); // Los distintos mensajes no requieren un port porque se turnan siempre. 
                                     // Si hubiera más mensajes de recepción o envío posibles, entonces sí se 
                                     // debería tener en cuenta el port.
                AnalizarSitio(sitio)
            }
        }
        ```

        El coordinador permite que se respete el órden de llegada de los examinadores ya que no deben quedarse esperando a que termine el análisis para que su mensaje sea recibido. De esta manera, el coordinador agrega los sitios a la cola y los mensajes se reciben con poco tiempo de espera, permitiendo que los examinadores continúen rápidamente con su trabajo, y que sean agregados a la cola apenas envían su mensaje.

2. _En un laboratorio de genética veterinaria hay 3 empleados. El primero de ellos continuamente prepara las muestras de ADN; cada vez que termina, se la envía al segundo empleado y vuelve a su trabajo. El segundo empleado toma cada muestra de ADN preparada, arma el set de análisis que se deben realizar con ella y espera el resultado para archivarlo. Por último, el tercer empleado se encarga de realizar el análisis y devolverle el resultado al segundo empleado._

    ```c
    process Preparador {
        string muestra;
        
        while (true) {
            muestra = PrepararMusetra();
            Admin!preparacion(muestra);
        }
    }

    process Armador {
        string muestra;
        string set;
        string resultado;

        while (true) {
            Admin!pedidoMuestra();
            Admin?preparacion(muestra);
            set = PrepararSet(muestra);
            Analizador!(set);

            Analizador?(resultado)
            ArchivarResultado(resultado);
        }
    }

    process Analizador {
        string set;
        string resultado;

        while (true) {
            Armador?(set);

            resultado = AnalizarSet(set);
            Armador!(resultado);
        }
    }

    process Admin {
        string muestra;
        string set;
        string resultado;
        cola muestras;
        cola sets;
        cola resultados;

        while (true) {
            do Preparador?preparacion(muestra) -> push(muestras, muestra);
            [] not empty(muestras); Armador?pedidoMuestra() -> Armador!preparacion(pop(muestras));
            od
        }
    }
    ```

3. _En un examen final hay N alumnos y P profesores. Cada alumno resuelve su examen, lo entrega y espera a que alguno de los profesores lo corrija y le indique la nota. Los profesores corrigen los exámenes respetando el orden en que los alumnos van entregando._

    > _Nota: maximizar la concurrencia; no generar demora innecesaria; todos los procesos deben terminar su ejecución_

    1. _Implemente una solución con PMS considerando que P=1._
    
        ```c
        process Alumno[id: 0..N-1] {
            string examen;
            string resultado;

            examen = ResolverExamen();
            Mesa!(id, examen);
            Profesor?(resultado);
        }

        process Profesor {
            string examen;
            string resultado;
            int alumno;

            while (true) {
                Mesa!();
                Mesa?(alumno, examen);
                resultado = CorregirExamen(examen);
                Alumno[alumno]!(resultado);
            }
        }

        process Mesa {
            int alumno;
            string examen;
            cola examenes;

            while (true) {
                do Alumno[*]?(alumno, examen) -> push(examenes, (alumno, examen));
                [] not empty(examenes); Profesor?() -> Profesor!(pop(examenes));
                od
            }
        }
        ```

    2. _Implemente una solución con PMS considerando que P>1._
        
        ```c
        process Alumno[id: 0..N-1] {
            string examen;
            string resultado;

            examen = ResolverExamen();
            Mesa!(id, examen);
            Profesor[*]?(resultado);
        }

        process Profesor[id: 0..P-1] {
            string examen;
            string resultado;
            int alumno;

            while (true) {
                Mesa!();
                Mesa?(alumno, examen);
                resultado = CorregirExamen(examen);
                Alumno[alumno]!(resultado);
            }
        }

        process Mesa {
            int alumno;
            string examen;
            cola examenes;

            while (true) {
                do Alumno[*]?(alumno, examen) -> push(examenes, (alumno, examen));
                [] not empty(examenes); Profesor[*]?() -> Profesor!(pop(examenes));
                od
            }
        }
        ```
    
    3. _Modifique (b) considerando que los alumnos no comienzan a realizar su examen hasta que todos hayan llegado al aula._
            
        ```c
        process Alumno[id: 0..N-1] {
            string examen;
            string resultado;

            Mesa!llegada();
            Mesa?inicio();

            examen = ResolverExamen();
            Mesa!(id, examen);
            Profesor[*]?(resultado);
        }

        process Profesor[id: 0..P-1] {
            string examen;
            string resultado;
            int alumno;

            Mesa!llegada();

            while (true) {
                Mesa!();
                Mesa?(alumno, examen);
                resultado = CorregirExamen(examen);
                Alumno[alumno]!(resultado);
            }
        }

        process Mesa {
            int alumno;
            string examen;
            cola examenes;

            for (int i = 0; i < N + P; i++) {
                do Alumno[*]?llegada() -> ;
                [] Profesor[*]?llegada() -> ;
                od
            }

            for (int i = 0; i < N; i++) {
                Alumno[i]!inicio();
            }

            while (true) {
                do Alumno[*]?(alumno, examen) -> push(examenes, (alumno, examen));
                [] not empty(examenes); Profesor[*]?() -> Profesor!(pop(examenes));
                od
            }
        }
        ```
    
4. _En una exposición aeronáutica hay un simulador de vuelo (que debe ser usado con exclusión mutua) y un empleado encargado de administrar su uso. Hay P personas que esperan a que el empleado lo deje acceder al simulador, lo usa por un rato y se retira._

    > _Nota: cada persona usa sólo una vez el simulador._

    1. _Implemente una solución donde el empleado sólo se ocupa de garantizar la exclusión mutua (sin importar el orden)._

        ```c
        process Persona[id: 0..P-1] {
            Empleado?();
            UsarSimulador();
            Empleado!();
        }

        process Empleado {
            for (int i = 0; i < P; i++) {
                Persona[*]!();
                Persona[*]?();
            }
        }
        ```
    
    2. _Modifique la solución anterior para que el empleado los deje acceder según el orden de su identificador (hasta que la persona i no lo haya usado, la persona i+1 debe esperar)._

        ```c
        process Persona[id: 0..P-1] {
            Empleado?();
            UsarSimulador();
            Empleado!();
        }

        process Empleado {
            for (int i = 0; i < P; i++) {
                Persona[i]!();
                Persona[i]?();
            }
        }
        ```

    3. _Modifique la solución a) para que el empleado considere el orden de llegada para dar acceso al simulador._ 

        ```c
        process Persona[id: 0..P-1] {
            Empleado!llegada(id);
            Empleado?();
            UsarSimulador();
            Empleado!();
        }

        process Empleado {
            int persona;
            bool libre;
            cola personas;

            while (true) {
                do Persona[*]?llegada(persona) -> 
                    if (not libre) {
                        push(personas, persona);
                    }
                    else {
                        libre = false;
                        Persona[persona]!();
                    };
                [] Persona[*]?() ->}
                    if (not empty(personas)) {
                        persona = pop(personas, persona);
                        Persona[persona]!();
                    }
                    else {
                        libre = true;
                    }
                od
            }
        }
        ```

5. _En un estadio de fútbol hay una máquina expendedora de gaseosas que debe ser usada por E Espectadores de acuerdo al orden de llegada. Cuando el espectador accede a la máquina en su turno usa la máquina y luego se retira para dejar al siguiente. Nota: cada Espectador una sólo una vez la máquina._

    ```c
    process Espectador[id: 0..E-1] {
        Maquina!(id);
        Maquina?();
        UsarMaquina();
        Maquina!();
    }

    process Maquina {
        int espectador;
        bool libre;
        cola espectadores;

        while (true) {
            do Espectador[*]?(espectador) ->
                if (not libre) {
                    push(espectadores, espectador);
                }
                else {
                    libre = false;
                    Espectador[espectador]!();
                }
            [] Espectador[*]?() ->
                if (not empty(espectadores)) {
                    espectador = pop(espectadores);
                    Espectador[espectador]!();
                }
                else {
                    libre = true;
                }
            od
        }
    }
    ```