# Parcial 2020
## Ejercicio 1 (semáforos)
Resolver con SEMÁFOROS el siguiente problema. En una empresa hay UN Coordinador y 30 Empleados que formarán 3 grupos de 10 empleados cada
uno. Cada grupo trabaja en una sección diferente y debe realizar 345 unidades de un producto. Cada empleado al llegar se dirige al coordinador para que
le indique el número de grupo al que pertenece y una vez que conoce este dato comienza a trabajar hasta que se han terminado de hacer las 345 unidades
correspondientes al grupo (cada unidad es hecha por un único empleado). Al terminar de hacer las 345 unidades los 10 empleados del grupo se deben
juntar para retirarse todos juntos. El coordinador debe atender a los empleados de acuerdo al orden de llegada para darle el número de grupo (a los 10
primeros que lleguen se le asigna el grupo 1, a los 10 del medio el 2, y a los 10 últimos el 3). Cuando todos los grupos terminaron de trabajar el coordinador
debe informar (imprimir en pantalla) el empleado que más unidades ha realizado (si hubiese más de uno con la misma cantidad máxima debe informarlos
a todos ellos). Nota: maximizar la concurrencia; suponga que existe una función Generar que simula la elaboración de una unidad de un producto.
```cpp
Sem mutexcola = 1, despertarCoordinador = 0, esperarGrupo[30] = {[3]0}, mutexcantidad[3] = {[3]1}, mutexTerminaron[3] = {[3]1}, dormir[30] = {[3]0}, mutexgrupos = 1, despertarEmpleado = 0
Cola cola
int cantidades[3]
int personales[30]
int terminaron[3]
bool losmascapos[30]
int total
Process Empleado[id:1..30]{
    P(mutexcola)
    cola.push(id)
    V(mutexcola)

    V(despertarCoordinador)
    P(esperarGrupo[id])
    grupo = asignacion[id]

    P(mutexCantidad[grupo])
    while(cantidades[grupo] < 345){
        cantidades[grupo]++
        V(mutexcantidad[grupo])
        //hacer unidad
        personales[id]++
        P(mutexCantidad[grupo])
    }
    V(mutexcantidad[grupo])

    P(mutexTerminaron[grupo])
    terminaron[grupo]++
    if(terminaron[grupo] == 10){
        for(i=0,i<10,i++)
            V(dormir[grupo])
        P(mutexGrupos)
        total++ //sumo a la cant de grupos terminados
        if(total == 3) //si ya terminaron todos se despierta al coordinador
            V(despertarCoordinador)
        V(mutexGrupos)
    }
    V(mutexTerminaron[grupo])
    P(dormir[grupo]) //esto lo tenia que hacer para todos incluso el que despertaba no?

    P(despertarEmpleado) //espero que el coordinador vea quienes son los mas capos
    if(losmascapos[id] == true)
        SoyElMasCapo()
}

Process Coordinador{
    llegaron = 0
    grupo = 1
    for(i=0,i<30,i++){
        P(despertarCoordinador)

        P(mutexcola)
        id = cola.pop()
        V(mutexcola)

        asignacion[id] = grupo
        V(esperarGrupo[id])

        llegaron++
        if(llegaron == 10){
            llegaron = 0
            grupo++
        }
    }
    P(despertarCoordinador)
    losmascapos = personales.max() //asumo que en este vector pone un booleano true para los id que tenian el maximo (menos ganas de laburar tengo)
    for(i=0,i<30,i++){
        V(despertarEmpleado)
    }
}
//en otra solución es el coordinador quien cuenta cuantos grupos terminaron haciendo un for de P(terminaron), esta mal lo que hice yo??
```
## Ejercicio 2 (monitores)
Resolver con MONITORES la siguiente situación. En la guardia de traumatología de un hospital trabajan 5 médicos y una enfermera. A la guardia
acuden P Pacientes que al llegar se dirigen a la enfermera para que le indique a que médico se debe dirigir y cuál es su gravedad (entero entre 1 y 10);
cuando tiene estos datos se dirige al médico correspondiente y espera hasta que lo termine de atender para retirarse. Cada médico atiende a sus pacientes
en orden de acuerdo a la gravedad de cada uno. Nota: maximizar la concurrencia.
```cpp

```

# Ejercicio 3 (monitores)
Se tienen N procesos que comparten el uso de una CPU.
Un proceso, cuando necesita utilizar la CPU, le pide el uso, le proporciona el trabajo que va a ejecutar y el tiempo que insume ejecutar el trabajo.
Luego, el proceso se queda dormí do hasta que la CPU haya finalizado de ejecutar su trabajo momento en que es despertado y prosigue su ejecución
normal.
La CPU funciona de la siguiente manera. Cada vez que el proceso i (i:1..N) pide CPU esta lo pone en el lugar iesimo de una lisia de procesos esperando
para ejecutar. La CPU recorre Circularmente la lista de procesos esperando a ser ejecutados (del 1 al N) T sí él proceso esta lista lo saca de ía lisia y
lo ejequia durante un lapso de 10 mis (o un lapso menor, si el tiempo de ejecución del proceso es.menor a 10 mis). Una vez que ejecutó un proceso, si
todavía le queda al proceso tiempo de.ejecución lo vuelve a poner en su lugar en Ía lista y pasa al siguiente: Si el proceso terminó de ejecutar su trabajo,
la CPU lo despierta para que continúe su ejecución y continúa con el siguiente proceso de la lista. Cuando la lisia está vacío, ía CPU no debe hacer nada,
simplemente debe esperar a que algún proceso ingrese a la lista de procesos en espera de ejecución, para empezar a trabajar.
Tenga en cuenta que existe la función delay(x) que retarda un proceso durante x mis. La ejecución de un proccsd puede ser modelizada utilizando esta
función,
No hagan suposiciones acerca de ios tiempos de ejecución de los procesos Modelice utilizando Monitores.
```cpp

```

# Ejercicio 4 (semáforos)
Suponga un juego donde hay 30 competidores. Cuando los jugadores llegan avisan al encargado, una vez que están ios 30 el encargado del juego
les entrega un numero aleatorio del 1 al 15 de tal manera que dos compelí dores tendrán el mismo numero. (Suponga que existe una función DarNumeroQ que 
devuelve en forma aleatoria un numero de! 1 aí 15, el encargado no guarda el numero que les asigna a los competidores). Una vez que
ya se entregaron los 30 números/los competidores buscaran concurrentemente su compañero que tenga el mismo numero (tenga en cucntn que pueden
empezar a buscar cuando lodos los competidores tengan cf numero no antes; Además la búsqueda de un jugador no interfiere con la búsqueda de otros
que tengan distinto numero). Cuando los competidores se encuentran permanecen en una snla durante 15 minutos y dejan de jugar. Luego cada uno
de los competidores avisa al encargado que termino de jugar y espero a que su compañero (el que tenia ci mismo numero) también avise que finalizo
para luego irse ambos; el encargado cuando llega ei segundo competidor Ies devuelve a ambos el resultado que obtuvieron que es el orden en que so van,
(Los primeros en irse; tendrán como resultado 1, los últimos i 5). Para modelizar el tiempo utilice la función Delay(x) que produce un retardo de x minutos.
```cpp

```
# Ejercicio 5 (monitores)
Se debe controlar el acceso a una impresora. Existen dos tipos de procesos trabajadores, A procesos de tipo 1 y B procesos de tipo 2 que trabajan y
al final del día por única vez intentan imprimir un resumen de la siguiente manera: Proceso tipo 1: espera hasta poder imprimir el resumen. Proceso
tipo 2: intenta imprimir su resumen, si no lo logro en 2 minutos, se retira sin imprimirlo. Además existen E técnicos encargados del mantenimiento de la
impresora, donde cada uno podrá acceder a la impresora si no hay ningún otro proceso (trabajador o técnico) usándola. Al acceder detecta si hay algún
error y lo resuelve (tarda 2 minutos en realizar esto). Para que un proceso trabajador pueda imprimir debe asegurarse de que la impresora esté libre
(ningún trabajador o técnico debe estar usando la impresora). Al acceder a la impresora imprime durante cierto tiempo.
```cpp

```
