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
Sem mutex1 = 1
Sem waiting[P] = {[P]0}
Cola cola
Cola terminales
int T[P]
int libres = T 
Process Persona[id: 1..P]{
    p(mutex)
    if(libres = 0)
        cola.push(id)
        v(mutex)
        p(waiting[id])
    else
        libres--
        t[id] = terminales.pop()
        v(mutex)
    UsarTerminal(t)
    p(mutex)
    if(cola.empty())
        libres++
        terminales.push(t)
    else
        sig = cola.pop()
        t[sig] = t[id]
        v(waiting[sig])
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
    bool prioridad 
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
    cond esperaP, esperaS
    int cantP == 0, cantS == 0
    Procedure llegar(id, prioridad: in bool){
        if(prioridad){
            cantP++
            signal(autoridad)
            wait(esperaP)
        }
        else{
            cantS++
            signal(autoridad)
            wait(esperaS)
        }
    }
    Procedure darAcceso(){
        if(cantP == 0 && cantS == 0)
            wait(autoridad)
        elif(cantP > 0)
            cantP--
            signal(esperaP)
        else
            cantS--
            signal(esperaS)
        wait(maquinalibre)
    }
    Procedure dejar(){
        signal(maquinaLibre)
    }
}
```