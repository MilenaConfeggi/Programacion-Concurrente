# Parcial 03-05-2023
## Ejercicio 1 (semáforos)
Para un experimento se tiene una red con 15 controladores de temperatura y dos módulos centrales. Los controladores
cada cierto tiempo toman la temperatura mediante la función medir() y la envía para que alguna de las centrales
le indique que debe hacer (numero del 1 al 10), y luego realiza esa ccion mediante la funcion actuar(). Las centrales
atienden los pedidos de los controles en orden de llegada, usando la funcion determinar() para determinar la acción 
que deberá hacer ese controlador (num del 1 al 10)
```cpp
Sem mutex = 1, hayPedidos = 0, esperando[15] = {[15]0}
int acciones[15]
Process Controlador[id:1..15]{
    while(true){
        t = medir()
        P(mutex)
        cola.push(id,t)
        V(mutex)
        V(hayPedidos)
        P(esperando[id])
        nro = acciones[id]
        actuar(nro)
    }
}
Process Central[id:1..2]{
    while(true){
        P(hayPedidos)
        P(mutex)
        id, t = cola.pop()
        V(mutex)
        nro = medir(id)
        acciones[id] = nro
        V(esperando[id])
    }
}
```
## Ejercicio 2 (monitores)
Hay una boletería virtual que vende en forma online E entradas para un partido de futbol a P personas (P>E) de acuerdo con 
el orden de llegada. Cuando la boleteria atiende a una persona, si aun quedan entradas disponibles le envia el num de entrada
vendida, sino le indica que no hay más entradas.
```cpp
Process Persona[id:1..P]{
    fila.solicitar()
    boleteria.obtener(entrada)
    if(entrada == -1)
        //ta triste
    else
        //ta feliz
}
Process Empleado{
    int nroEntrada = E
    while(nroEntrada > 0){
        fila.siguiente()
        vender()
        entrada--
        boleteria.asignar(entrada)
    }
    entrada == -1
    while(true){
        fila.siguiente()
        boleteria.asignar(entrada)
    }
}
Monitor Fila{
    int esperando = 0
    cond empleado, listo
    Procedure solicitar(){
        esperando++
        signal(empleado)
        wait(listo)
    }
    Procedure siguiente(){
        if(esperando == 0)
            wait(empleado)
        else
            esperando--
        signal(listo)
    }
}
Monitor Boleteria{
    int e
    cond persona, listo
    bool hayEntrada = false
    Procedure asignar(entrada: in int){
        e = entrada
        hayEntrada = true
        signal(persona)
        wait(listo)
    }
    Procedure obtener(entrada: out int){
        if(hayEntrada == false)
            wait(persona)
        entrada = e
        signal(listo)
        hayEntrada = false
    }
}
```
## Ejercicio 3 (monitores)
En un camino turístico hay un puente por donde puede pasar un vehículo a la vez. Hay N autos que deben
pasar por el de acuerdo con el orden de llegada. Solo puede haber procesos auto.
```cpp
Process Vehiculo[id:1..N]{
    puente.llegar()
    //pasar
    puente.irse()
}
Monitor Puente{
    Procedure llegar(){
        if(libre = false){
            esperando++
            wait(liberado)
        }
        else
            libre = false
    }
    Procedure irse(){
        if(esperando == 0)
            libre = true
        else
            esperando--
            signal(liberado)
    }
}
```
