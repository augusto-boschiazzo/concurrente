# Practica 5

1. _Se requiere modelar un puente de un único sentido que soporta hasta 5 unidades de peso. El peso de los vehículos depende del tipo: cada auto pesa 1 unidad, cada camioneta pesa 2 unidades y cada camión 3 unidades. Suponga que hay una cantidad innumerable de vehículos (A autos, B camionetas y C camiones). Analice el problema y defina qué tareas, recursos y sincronizaciones serán necesarios/convenientes para resolver el problema._
    
    1. _Realice la solución suponiendo que no se tiene ningún orden ni prioridad entre los diferentes tipos de vehículos._

        ```ada
        Procedure ejercicio1 is
            TASK TYPE Auto;
            TASK BODY Auto IS
                Puente.Pasar(1);
                AtravesarPuente();
                Puente.Salir(1);
            END Auto;

            TASK Puente IS
                ENTRY Pasar(peso: IN integer);
                ENTRY Salir(peso: IN integer);
            END Puente;

            TASK BODY Puente IS
                
            END Puente;
        ```
    
    2. _Modifique la solución de (a) para que tengan mayor prioridad los camiones que el resto de los vehículos._