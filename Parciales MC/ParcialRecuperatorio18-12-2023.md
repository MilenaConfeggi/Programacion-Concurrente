# Parcial resuperatorio 18-12-2023
## Ejercicio 1 (semáforos)
Resolver con SEMÁFOROS el siguiente problema. Se debe simular el uso de una máquina expendedora de gaseosas
con capacidad para 100 latas por parte de U usuarios. Además existe un repositor encargado de reponer las latas de
la máquina. Los usuarios usan la máquina según el orden de llegada. Cuando les toca usarla, sacan una lata y luego
se retiran. En el caso de que la máquina se quede sin latas, entonces le debe avisar al repositor para que cargue
nuevamente la máquina en forma completa. Luego de la recarga, saca una lata y se retira. Nota: maximizar la
concurrencia; mientras se reponen las latas se debe permitir que otros usuarios puedan agregarse a la fila.
```cpp
Sem mutex = 1, despertarRepositor = 0, esperarLatas = 0, personas[U] = {[U]0}
bool libre = true
Cola cola
Process Usuarios[id:1..U]{
    P(mutex)
    if(!libre){
        cola.push(id)
        V(mutex)
        P(personas[id])
    }
    else{
        libre = false
        V(mutex)
    }
    if(latas == 0)
        V(despertarRepositor)
        P(esperarLatas)
    latas--
    P(mutex)
    if(cola.empty())
        libre = true
    else
        V(personas[cola.pop()])
    V(mutex)
}
Process Repositor{
    while(true){
        P(despertarRepositor)
        latas = 100
        V(esperarLatas)
    }
}
```
## Ejercicio 2 (monitores)
Resolver el siguiente problema con MONITORES. En una montaña hay 30 escaladores que en una parte de la subida
deben utilizar un único paso de a uno a la vez y de acuerdo al orden de llegada al mismo. Nota: sólo se pueden
utilizar procesos que representen a los escaladores; cada escalador usa sólo una vez el paso.
```cpp
Process Escalador[id:1..30]{
    //escalar
    paso.llegar()
    //pasar
    paso.terminar()
}

Monitor Paso{
    Procedure llegar(){
        if(!libre){
            esperando++
            wait(pasar)
        }
        else
            libre = false
    }
    Procedure terminar(){
        if(esperando == 0)
            libre = true
        else
            signal(pasar)
            esperando--
    }
}
```