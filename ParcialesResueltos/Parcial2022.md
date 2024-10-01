# Parcial 2022
## Ejercicio 1 (semaforos)
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
