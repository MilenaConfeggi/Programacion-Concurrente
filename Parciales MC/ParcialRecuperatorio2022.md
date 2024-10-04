# Parcial 2022 recuperatorio
## Ejercicio 1 (semaforo)
1. Resolver con SEMÁFOROS el siguiente problema. En una fábrica de muebles trabajan 50 empleados. A llegar, los empleados
forman 10 grupos de 5 personas cada uno, de acuerdo al orden de llegada (los 5 primeros en llegar forman el primer grupo, los 5
siguientes el segundo grupo, y así sucesivamente). Cuando un grupo se ha terminado de formar, todos sus integrantes se ponen a
trabajar. Cada grupo debe armar M muebles (cada mueble es armado por un solo empleado); mientras haya muebles por armar en
el grupo los empleados los irán resolviendo (cada mueble es armado por un solo empleado). Nota: Cada empleado puede tardar
distinto tiempo en armar un mueble. Sólo se pueden usar los procesos “Empleado”, y todos deben terminar su ejecución.
Maximizar la concurrencia.
```cpp
int nro = 1
int muebles[10] = {[10]0}
Sem mutexcola = 1
Sem arrancar[50] = {[50]0}
Sem mutexgrupo[10] = {[10]1}
Process Empleado[id:1..50]{
    int grupo
    P(mutexcola)
    cola.push(id)
    if(cola.lenght() == 5){
        for(i=0,i<5,i++){
            V(arrancar[cola.pop()])
            grupo = nro
        }
        nro++
    }
    V(mutexcola)
    P(arrancar[id])
    P(mutexgrupo[grupo])
    while (muebles[grupo] < M){
        muebles[grupo]++
        V(mutexgrupo[grupo])
        //hacer mueble
        P(mutexgrupo[grupo])
    }
    V(mutexgrupo[grupo])
}
```
## Ejercicio 2 (monitores)
2. Resolver con MONITORES el siguiente problema. En un comedor estudiantil hay un horno microondas que debe ser usado por E
estudiantes de acuerdo con el orden de llegada. Cuando el estudiante accede al horno, lo usa y luego se retira para dejar al
siguiente. Nota: cada Estudiante una sólo una vez el horno; los únicos procesos que se pueden usar son los “estudiantes”.
```cpp
Process Estudiante[id:1..E]{
    microondas.llegar(id)
    //usar microondas
    microondas.salir()
}
Monitor Microondas{
    bool libre = true
    int esperando = 0
    cond estudiante
    Procedure llegar(id: in int){
        if (!libre){
            esperando++
            wait(estudiante)
        }
        else
            libre = false
    }
    Procedure salir(){
        if(esperando == 0){
            libre = true
        }
        else{
            signal(estudiante)
            esperando--
        }
    }

}
```