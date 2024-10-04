# Parcial 2022
## Ejercicio 1 (semaforos)
En una planta verificadora de vehículos, existen 7 estaciones donde 
se dirigen 150 vehículos para ser verificados. Cuando un vehículo llega a la planta, el coordinador de la planta le 
indica a qué estación debe dirigirse. El coordinador selecciona la estación que tenga menos vehículos asignados en ese 
momento. Una vez que el vehículo sabe qué estación le fue asignada, se dirige a la misma y espera a que lo llamen 
para verificar. Luego de la revisión, la estación le entrega un comprobante que indica si pasó la revisión o no. Más allá 
del resultado, el vehículo se retira de la planta. Nota: maximizar la concurrencia.
```cpp
Sem mutexcola = 1, despertarcoordinador = 0, mutexvector = 1
Sem vehiculos[150] = {[150]0} //lo uso para esperar que le asignen estacion al vehiculo
Sem mutexEstacion[7] = {[7]1} //lo uso para agregar a la cola de estacion
Sem despertar[7] = {[7]0} //lo uso para que la estacion sepa que tiene un vehiculo esperando
Sem listo[150] = {[150]0} //lo uso para que el vehiculo sepa que ya tiene su comprobante
Cola cola
int estacionaignada[150]
Cola estaciones[7]
int comprobantes[150]
int vector[7] = {[7]0}
Process Vehiculo[id:1..150]{
    //llega
    P(mutexcola)
    cola.push(id)
    V(mutexcola)
    V(despertarcoordinador)

    P(vehiculos[id])
    estacion = estacionasignada[id]

    P(mutexestacion[estacion])
    estaciones[estacion].push(id)
    V(mutexestacion[estacion])

    V(despertar[estacion])

    P(listo[id])
    comprobante = comprobantes[id]

    P(mutexvector)
    vector[estacion]--
    V(mutexvector)
}
Process Coordinador{
    for(i=0,i<150,i++){
        P(despertarcoordinador)

        P(mutexcola)
        id = cola.pop()
        V(mutexcola)

        P(mutexvector)
        idestacion = min(vector)
        vector[idestacion]++
        V(mutexvector)

        estacionasignada[id] = idestacion
        V(vehiculos[id])
    }
}
Process Estacion[id:1..7]{
    while (true){
        P(despertar[id])

        P(mutexestacion[id])
        idvehiculo = estaciones[id].pop()
        V(mutexestacion[id])

        comprobante = idvehiculo.revisar()
        comprobantes[idvehiculo] = comprobante
        V(listo[idvehiculo])
    }
}
```
## Ejercicio 2 (monitores)
Resolver con MONITORES el siguiente problema. En un sistema operativo se ejecutan 20 procesos que 
periódicamente realizan cierto cómputo mediante la función Procesar(). Los resultados de dicha función son 
persistidos en un archivo, para lo que se requiere de acceso al subsistema de E/S. Sólo un proceso a la vez puede hacer 
uso del subsistema de E/S, y el acceso al mismo se define por la prioridad del proceso (menor valor indica mayor 
prioridad).
```cpp
Process Proceso[id:1..20]{
    while (true)
        result = Procesar()
        E/S.pediracceso(result)
        //usar
        E/S.salir()
}
Monitor E/S{
    Procedure perdiracceso(){
        if(libre = false){
            esperando++
            cola.push(result, prioridad)
            wait(acceso[result])
        }
        else
            libre = false        
    }
    Procedure salir(){
        if(esperando = 0)
            esperando--
            libre = true
        else
            signal(acceso[cola.pop()])
    }

}
```
