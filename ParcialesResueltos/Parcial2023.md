# Parcial 2023

## Ejercicio 1 (Semáforos)
1. Resolver con SEMÁFOROS los problemas siguientes:
a) En una estación de trenes, asisten P personas que deben realizar una carga de su tarjeta SUBE en la terminal
disponible. La terminal es utilizada en forma exclusiva por cada persona de acuerdo con el orden de llegada.
Implemente una solución utilizando sólo emplee procesos Persona. Nota: la función UsarTerminal() le permite cargar
la SUBE en la terminal disponible. 

```cpp
Sem mutex1 = 1
Sem waiting[P] = {[P]0}
Cola cola
Boolean vacio = true
Process Persona[id: 1..P]{
    p(mutex)
    if(vacio)
        cola.push(id)
        v(mutex)
        p(waiting[id])
    else
        vacio = false
        v(mutex)
    UsarTerminal()
    p(mutex)
    if(cola.empty)
        vacio = true
    else
        v(waiting[cola.pop()])
    v(mutex)
}
```
b) Resuelva el mismo problema anterior pero ahora considerando que hay T terminales disponibles. Las personas
realizan una única fila y la carga la realizan en la primera terminal que se libera. Recuerde que sólo debe emplear
procesos Persona. Nota: la función UsarTerminal(t) le permite cargar la SUBE en la terminal t.
```cpp
Sem mutex1 = 1, mutexTerminales = 1
Sem waiting[P] = {[P]0}
Cola cola
Cola terminales
Boolean vacio = true
int T
int libres = T 
Process Persona[id: 1..P]{
    p(mutex)
    if(libres = 0)
        cola.push(id)
        v(mutex)
        p(waiting[id])
    else
        libres--
        v(mutex)
    P(mutexTerminales)
    t = terminales.pop()
    v(mutexTerminales)
    UsarTerminal(t)
    P(mutexTerminales)
    terminales.push(t)
    v(mutexTerminales)
    p(mutex)
    if(cola.empty())
        libres++
    else
        v(waiting[cola.pop()])
    v(mutex)
}
```

## Ejercicio 2 (Monitores)
2.Resolver con MONITORES el siguiente problema: En una elección estudiantil, se utiliza una máquina para voto
electrónico. Existen N Personas que votan y una Autoridad de Mesa que les da acceso a la máquina de acuerdo con el
orden de llegada, aunque ancianos y embarazadas tienen prioridad sobre el resto. La máquina de voto sólo puede ser
usada por una persona a la vez. Nota: la función Votar() permite usar la máquina.
```cpp
Process Persona[id:1..N]{
    int prioridad 
    maquina.llegar(id, prioridad)
    votar()
    maquina.dejar()
}
Process Autoridad{
    for(i=0, i<N, i++){
        maquina.darAcceso()
    }
}
Monitor Máquina{
    cond autoridad, maquinaLibre
    cond esperando [N]
    Procedure llegar(id, prioridad: in int){
        cola.insertarPorPrioridad(id, prioridad)
        signal(autoridad)
        wait(esperando[id])
    }
    Procedure darAcceso(){
        if(llegas.empty())
            wait(autoridad)
        id = cola.pop()
        signal(esperando[id])
        wait(maquinaLibre)
    }
    Procedure dejar(){
        signal(maquinaLibre)
    }
}
```