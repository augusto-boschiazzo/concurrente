# Parcial 2025 Fecha 1

## Ejercicio 1:

```c
Monitor Cabina {
    int esperando = 0, usando = 0;
    cond cola[N];
    Queue espera;

    PedirFoto(int id, int edad) {
        if (usando > 0) {
            InsertarOrdenado(espera, (id, edad));
            esperando++;
            wait(cola[id]);
        }
        else {
            usando++;
        }
    }

    LiberarCabina() {
        if (espera > 0) {
            int sig = pop(espera);
            signal(cola[sig]);
            espera--;
        }
        else {
            usando--;
        }
    }
}

Process Cliente[id: 0..49] {
    int edad = ObtenerEdad();
    Cabina.PedirFoto(id, edad);
    TomarFoto();
    Cabina.LiberarCabina();
}
```

---

## Ejercicio 2

```c
sem sem_cola = 1;
sem sem_imp = 0;
sem sem_ver [N] = [N] {0};
Queue pedidos;
Pieza piezas[N];

Process Cliente[id: 0..N]{
    Pieza pieza = ObtenerPedido(id);
    P(sem_cola);
    push(pedidos, (id, pedido));
    V(sem_cola);
    V(sem_imp);
    P(sem_ver[id]);
    bool resultado = VerificarPieza(piezas[id]);
    if (resultado) {
        Print("Cumple");
    }
    else {
        Print("No cumple");
    }
}

Process Impresora{
    int id;
    Pieza pieza;
    for (int i = 0; i < N; i++){
        P(sem_imp);
        P(sem_cola);
        (id, pedido) = pop(pedidos);
        V(sem_cola);
        piezas[id] = ImprimirPieza(pedido);
        V(sem_ver[id]); 
    }
}
```

# Parcial 2026 - MC - Fecha 1

_Esto es más o menos lo que recuerdo de las consignas_ 

1. Usando **semáforos** modele: 22 autos de fórmula 1 corren una carrera, van llegando y ubicándose. Esperan a estar todos, cuando llega el último el comisario da la órden de salida. Dan 50 vueltas, y cuando completan la carrera le piden al comisario que revise su auto. Si está en buenas condiciones, les da el puesto. Si no, los descalifica. Funciones: Ubicarse(), DarVuelta(), Revisar(id_auto) que dice si está en condiciones o no.

```c
sem aviso = 0, largada = 0, ubicacion = 1, completado = 1;
sem revision[22] = [22] {0};
int ubicados = 0;
int puestos[22];
Queue autos;

Process Auto[id: 0..22-1]{
    int puesto;
    Ubicarse();
    P(ubicacion);
    ubicados++;
    if (ubicados == 22) {
        V(aviso);
    }
    V(ubicacion);
    P(largada);
    for (int i = 0; i < 50; i++) {
        DarVuelta();
    }
    P(completado);
    push(autos, id);
    V(completados);
    V(aviso);
    P(revision[id]);
    puesto = puestos[id];
}

Process Comisario{
    int puesto = 1, id_auto;
    bool cumple;
    P(aviso);
    for (int i = 0; i < 22; i++) {
        V(largada);
    }
    for (int i = 0; i < 22; i++) {
        P(aviso);
        P(completado);
        id_auto = pop(autos);
        V(completado);
        cumple = Revisar(id_auto);
        if (cumple) {
            puestos[id_auto] = puesto++;
        }
        else {
            puestos[id_auto] = -1; // valor de descalificación
        }
        V(revision[id]);
    }
}
```

2. Usando **monitores** modele: Se quieren probar 50 autos (22 F1 y 28 F2) en circuitos. Existen 3 circuitos. Cada uno ya sabe qué circuito quiere usar. Sólo pueden probar uno a la vez, y tienen prioridad los autos de F1. Funciones: Usar().

```c
Monitor Circuito[0..2]{
    bool ocupado = false;
    int esperando_f1 = 0, esperando_f2 = 0;
    cond cola_f1, cola_f2;

    PedirUsarF1() {
        if (ocupado || esperando_f1 > 0) {
            esperando_f1++;
            wait(cola_f1);
        } else {
            ocupado = true;
        }
    }

    PedirUsarF2() {
        if (ocupado || esperando_f2 > 0 || esperando_f1 > 0) {
            esperando_f2++;
            wait(cola_f2);
        } else {
            ocupado = true;
        }
    }

    Liberar() {
        if (esperando_f1 > 0) {
            esperando_f1++;
            signal(cola_f1);
        } else if (esperando_f2 > 0) {
            esperando_f2++;
            signal(cola_f2);
        } else {
            ocupado = false;
        }
    }
}

Process AutoF1[id: 0..22] {
    int circuito = ...; // se conoce.
    Circuito[circuito].PedirUsarF1();
    Usar();
    Circuito[circuito].Liberar();
}

Process AutoF2[id: 0..28] {
    int circuito = ...; // se conoce.
    Circuito[circuito].PedirUsarF2();
    Usar();
    Circuito[circuito].Liberar();
}
```

# Parcial 2025 - MD - Fecha 1

1. 
    ```c
    chan llegada();
    chan inicio();
    chan entrega(int, text);
    chan correccion[E](int);

    process Estudiante[id: 0..E-1] {
        text examen;
        int resultado;
        
        send llegada();
        receive inicio();

        examen = ResolverExamen();
        send entrega(id, examen);
        receive correcion[id](resultado);
    }

    process Profesor {
        text examen;
        int alumno;
        int resultado;

        for (i: 0..E-1) {
            receive llegada();
        }

        for (i: 0..E-1) {
            send inicio();
        }

        for (i: 0..E-1) {
            receive entrega(alumno, examen);
            resultado = CorregirParcial();
            send correcion[alumno](resultado);
        }
    }
    ```
2. 

    ```c
    process Paciente[id: 0..15-1] {
        text sintomas;
        text tratamiento;

        Admin!llegada(id);
        Medico?atencion(tratamiento);
    }

    process Admin {
        text sintomas;
        int idPaciente;
        cola pacientes;

        while(true) {
            do Pacientes[*]?llegada(idPaciente, sintomas) -> push(pacientes, (idPaciente, sintomas));
            [] not empty(pacientes); Medico?pedirTrabajo() -> Medico!enviarPaciente(pop(pacientes));
            od
        }
    }

    process Medico {
        text sintomas;
        text tratamiento;
        int idPaciente;

        while (true) {
            Admin!pedirTrabajo();
            Admin?enviarPaciente(idPaciente, sintomas);
            tratamiento = atender(sintomas);
            Paciente[idPaciente]!atencion(tratamiento);
        }
    }
    ```

3. 

    ```ada
    Procedure punto3
        TASK TYPE Programador IS
            ENTRY RecibirProblema(problema: IN text);
            ENTRY RecibirId(id: IN integer);
            ENTRY RecibirCalificacion(resultado, puesto: IN integer);
        END Programador;

        TASK BODY Programador IS
            problemilla: text;
            resolucioncilla: text;
            puestillo: integer;
            resultadillo: integer;

            Accept RecibirId(id: IN integer) do
                idecilla:= id;
            end RecibirId;

            Accept RecibirProblema(problema: IN text) do
                problemilla:= problema;
            END RecibirProblema;

            resolucioncilla:= ResolverProblema(problemilla);
            Admin.RecibirResolucion(idecilla, resolucioncilla);
            Accept RecibirCalificacion(resultado, puesto: IN integer) do
                resultadillo:= resultado;
                puestillo:= puesto;
            end RecibirCalificacion;
        END Programador;

        arrayProgramadores: array (1..P) of Programador;

        TASK Profesor IS
            ENTRY RecibirResolucion(id: IN integer; resolucion: IN text; resultado, puesto: OUT integer);
        END Profesor;

        TASK BODY Profesor
            puesto, idAlumno, resultado: integer;
            problema, resolucion: text;
            puesto:= 1;

            for i in 1..E loop
                arrayProgramadores(i).RecibirProblema(problema);
            end loop;

            loop
                Admin.PedidoExamen(idAlumno, resolucion)
                resultado:= Corregir(resolucion);
                arrayProgramadores(idAlumno).RecibirCalificacion(puesto, resultado);
                puesto:= puesto + 1;
            end loop;
        END Profesor;

        TASK Admin IS
            ENTRY RecibirResolucion(id: IN integer; resolucion: IN text);
            ENTRY PedidoExamen(id: OUT integer; resolucion: OUT text);
        END Admin;

        TASK BODY Admin IS
                Accept PedidoExamen(id: OUT integer; resolucion: OUT integer) is
                    Accept RecibirResolucion(idecilla: IN integer; resolucioncilla: IN text) is
                        id:= idecilla;
                        resolucion:= resolucioncilla;
                    end RecibirResolucion;
                end PedidoExamen;
        END Admin;

    Begin
        for i in 1..E loop
            arregloProgramadores(i).RecibirId(i);
        end loop;
    End punto3;
    ```