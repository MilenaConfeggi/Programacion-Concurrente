# Parcial recuperatorio
## Ejercicio 1 (semaforos)
1. Resolver con SEMÁFOROS el siguiente problema. En un restorán trabajan C cocineros y M mozos. De
forma repetida, los cocineros preparan un plato y lo dejan listo en la bandeja de platos terminados, mientras
que los mozos toman los platos de esta bandeja para repartirlos entre los comensales. Tanto los cocineros
como los mozos trabajan de a un plato por vez. Modele el funcionamiento del restorán considerando que la
bandeja de platos listos puede almacenar hasta P platos. No es necesario modelar a los comensales ni que
los procesos terminen.
NOTA: es el problema de productores/consumidores con tamaño de buffer limitado visto en la página 14
de la teoría 4.
```cpp
buffer platos[P]
int ocupado = 0
int libre = 0
Sem vacio = P
Sem lleno = 0
Sen mutex1 = 1, mutex2 = 0
Process Cocineros[id:1..C]{
    while (true){
        plato = preparar()
        P(vacio) //me fijo que haya lugar
        P(mutex1)
        platos[libre] = plato
        libre = (libre+1) mod P
        V(mutex1)
        V(lleno) //asviso que hay plato
    }
}
Process Mozos[id:1..M]{
    while (true){
        P(lleno)
        P(mutex2)
        plato = platos[ocupado]
        ocupado = (ocupado+1) mod P
        V(mutex2)
        V(vacio)
        repartir(plato)
    }
}
```
## Ejercicio 2 (monitores)
2.Resolver con MONITORES el siguiente problema. En una planta verificadora de vehículos existen 5
estaciones de verificación. Hay 75 vehículos que van para ser verificados, cada uno conoce el número de
estación a la cual debe ir. Cada vehículo se dirige a la estación correspondiente y espera a que lo atiendan.
Una vez que le entregan el comprobante de verificación, el vehículo se retira. Considere que en cada estación
se atienden a los vehículos de acuerdo con el orden de llegada. Nota: maximizar la concurrencia.
```cpp
//Me dijo el ayudante que es mejor tener un proceso aparte que haga el comprobante xq seria mas correcto semanticamente hablando
//los monitores no deberian "hacer" algo sino controlar accesos
Process Vehiculo[id:1..75]{
    int e
    fila[e].llegar()
    estacion[e].obtenerComprobante(comprobante)
    fila[e].irse()
}
Monitor Fila[id:1..5]{
    Procedure llegar(){
        if(!libre){
            esperando++
            wait(vehiculo)
        }
        else{
            libre = false
        }
    }
    Procedure irse(){
        if(esperando == 0)
            libre = true
        else
            signal(vehiculo)
    }

}
Monitor Estacion[id:1..5]{
    Procedure obtenerComprobante(comprobante: out txt){
        //hacer comprobante
        comprobante = hacerComprobante()
    }
}
```
