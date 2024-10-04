# Parcial recuperatorio 2024
## Ejercicio 1 (semáforos)
En una planta verificadora de vehículos existe un puesto de atención para atender a 30 vehiculos que deben ser verificados. En
el puesto de atencion hay un empleado que atiende a los vehículos de acuerdo al orden de llegada, realiza la verificación y le
entrega el comprobante, luego espera que el vehículos abandone el puesto para poder atender al siguiente. Todos los procesos
deben terminar
```cpp
Sem mutex = 1, despertarEmpleado = 0, terminalLibre = 0, esperar[20] = {[20]0}
Cola cola
comprobantes[20]
Proces Vehiculo[id:1..20]{
    P(mutex)
    cola.push(id)
    V(mutex)
    V(despertarEmpleado)
    P(esperar[id])
    c = comprobantes[id]
    V(terminalLibre)
}
Process Empleado{
    for(i=0, i<20, i++){
        P(despertarEmpleado)
        P(mutex)
        id = cola.pop()
        V(mutex)
        c = verificar(id)
        comprobantes[id] = c
        V(esperar[vehiculo])
        P(terminalLibre)
    }
}
```
## Ejercicio 2 (monitores)
En una oficina hay un empleado para atender solicitudes de N personas que vienen a realizar un trámite de acuerdo al orden
de llegada. Cada persona espera a que el empleado atienda su trámite y le de el resultado. Cuando el empleado está libre atiende
el sig pedido pendiente y si no hay ninguno lee un libro por 10min. Todos los procesos deben terminar.
```cpp
//debería haber hecho 2 monitores???
Process Persona[id:1..N]{
    oficina.llegar(id, r)
}
Process Empleado{
    while(atendidos < N){
        oficina.siguiente(id)
        if(id == -1)
            delay(10)
        else
            atendidos++
            r = resolverTramite(id)
            oficina.atender(id, r)
    }
}
Monitor oficina{
    Cola cola
    cond persona
    int resultados[N]
    Procedure llegar(id: in int, r: out int){
        cola.push(id)
        wait(persona)
        r = resultados[id]
    }
    Procedure siguiente(id: out int){
        if(cola.empty())
            id = 1
        else
            id = cola.pop()
    }
    Procedure atender(id: in int, t: in int){
        resultados[id] = t
        signal(persona)
    }
}
```