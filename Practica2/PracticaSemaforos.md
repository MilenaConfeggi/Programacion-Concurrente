# Practica 2 Semaforos
## Ejercicio 1

a y b)
```cpp
Sem libre = 1
Process Persona[id: 0..N]{
    P(libre)
    delay
    V(libre)
}
```
c)
```cpp
Sem libre = 3
Process Persona[id: 0..N]{
    P(libre)
    delay
    V(libre)
}
```
d)
```cpp
Sem libre = 3
Process Persona[id: 0..N]{
    for(i=1, i< N, i++){
        P(libre)
        delay
        V(libre)
    }
}
```
## Ejercicio 2

a)
```cpp
list historial[N]
int ocupado = 0
sem mutex = 1
Process chequear[id: 1..4]{
    P(mutex)
    while(ocupado < N){
        nro = historial[ocupado].getnro()
        id = historial[ocupado].getid()
        ocupado++
        V(mutex)
        if(nro == 3)
            imprimir(id)
        P(mutex)
    }
    V(mutex)
}
```
b)
```cpp
list historial[N]
int ocupado = 0
sem mutex1 = 1
sem mutex2 = 1
list contador[4][0]
Process chequear[id: 1..4]{
    P(mutex1)
    while(ocupado < N){
        nro = historial[ocupado].getnro()
        ocupado++
        V(mutex1)
        p(mutex2)
        contador[nro]++
        v(mutex2)
        P(mutex1)
    }
    V(mutex1)
}
```
c)
```cpp
cola historial[N]
int ocupado = 0
sem mutex = 1
list contador[4][0]
Process chequear[id: 0..3]{
    P(mutex)
    while(ocupado < N){
        fallo = historial.pop()
        ocupado++
        v(mutex)
        if(fallo.getNivel() == id)
            contador[id]++
        else
            p(mutex)
            ocupado--
            historial.push(fallo)
            v(mutex)
        P(mutex)
    }
    V(mutex)
}
```
## Ejercicio 3
```cpp
cola recursos
Sem e = 5, b = 1
int esperando = 0
Process Proceso[id:1..P]{
    P(e)
    P(b)
    dato = recursos.pop()
    v(b)
    //usar recurso
    p(b)
    recursos.push(dato)
    v(b)
    v(e)
}
```

## Ejercicio 4
```cpp
Sem total = 6, alta = 4, baja = 5 
Process Usuario_Alta[id:1..L]{
    P(alta)
    P(total)
    //usa bd
    V(alta)
    V(total)
}
Process Usuario_Baja[id:1..K]{
    P(baja)
    P(total)
    //usa bd
    V(baja)
    V(total)
}
```

## Ejercicio 5

a)
```cpp
Cola contenedores
sem lleno = 0, vacio = 1
Process Productor{
    while true{
        //preparar paquete
        P(vacio)
        contenedores.push()
        V(lleno)
    }
}
Process Entregador{
    while true{
        P(lleno)
        contenedores.pop()
        V(vacio)
        //entregar paquete
    }
}
```
b)
```cpp
Cola contenedores
sem lleno = 0, vacio = 1
sem mutex = 1
Process Productor[id: 1..P]{
    while true{
        //preparar paquete
        P(mutex)
        P(vacio)
        contenedores.push(paquete)
        V(lleno)
        V(mutex)
    }
}
Process Entregador{
    while true{
        P(lleno)
        paquete = contenedores.pop()
        V(vacio)
        //entregar paquete
    }
}
```
c)
```cpp
Cola contenedores
sem lleno = 0, vacio = 1
sem mutex = 1
Process Productor{
    while true{
        //preparar paquete
        P(vacio)
        contenedores.push()
        V(lleno)
    }
}
Process Entregador[id: 1..E]{
    while true{
        P(mutex)
        P(lleno)
        contenedores.pop()
        V(vacio)
        V(mutex)
        //entregar paquete
    }
}
```

d)
```cpp
Cola contenedores
sem lleno = 0, vacio = 1
sem mutex1 = 1, mutex2 = 1
Process Productor[id: 1..P]{
    while true{
        //preparar paquete
        P(mutex1)
        P(vacio)
        contenedores.push(paquete)
        V(lleno)
        V(mutex1)
    }
}
Process Entregador[id: 1..E]{
    while true{
        P(mutex2)
        P(lleno)
        contenedores.pop()
        V(vacio)
        V(mutex2)
        //entregar paquete
    }
}
```
## Ejercicio 6

a)
```cpp
sem mutex = 1
Process Persona[id:1..N]{
    P(mutex)
    imprimir(documento)
    V(mutex)
}
```

b)
```cpp
Cola c
Sem waiting[P] = {[P]0}
Sem mutex = 1
bool vacio = true
Process Persona[id:1..N]{
    P(mutex)
    if (!vacio)
        c.push(id)
        V(mutex)
        P(waiting[id])
    else
        vacio = false
        V(mutex)    
    Imprimir(documento)
    P(mutex)
    if(c.empty())
        vacio = true
    else
        V(waiting[c.pop()])
    V(mutex)
}
```
c)
```cpp
int proc = 1
Sem waiting[N] = {[N]0}
Process Persona[id:1..N]{
    if(prox != id)
        P(waiting[id])
    Imprimir(documento)
    prox++
    V(waiting[prox])
}
```

d)
```cpp
Sem mutex = 1, libre = 0, contenido = 0
Sem waiting[N] = {[N]0}
Cola c
Process Persona[id:1..N]{
    P(mutex)
    c.push(id)
    V(contenido)
    V(mutex)
    P(waiting[id])
    imprimir(documento)
    V(libre)
}
Process Coordinador{
    for (i=0, i<N, i++){
        P(contenido)
        P(mutex)
        V(waiting[c.pop()])
        V(mutex)
        P(libre)
    }
}
```

e)
```cpp
Sem mutex = 1, contenido = 0, mutexImpresoras = 1, cantimp = 5
Sem waiting[N] = {[N]0}
Cola c
Cola impresoras
List personas[N]
Process Persona[id:1..N]{
    P(mutex)
    c.push(id)
    V(contenido)
    V(mutex)
    P(waiting[id])
    imprimir(documento)
    P(mutexImpresoras)
    impresoras.push(personas[id])
    V(mutexImpresoras)
    V(cantImp)
}
Process Coordinador{
    for (i=0, i<N, i++){
        P(contenido)
        P(mutex)
        id = c.pop()
        V(mutex)
        P(cantImp)
        P(mutexImpresoras)
        imp = impresoras.pop()
        V(mutexImpresoras)
        Personas[id] = imp
        V(waiting[id])
    }
}
```

## Ejercicio 7
```cpp
Cola c
int cant = 0
Sem mutex1 = 1, mutex2 = 1, hayTrabajo = 0, barreraAlumnos = 0
Sem puntaje[10]
List nota[10]
Process Alumno[id:1..50]{
    int tarea, nota
    tarea = elegir()
    P(Mutex)
    cant++
    if(cant == 50){
        for (i=0, i<50, i++){
            V(barreraAlumnos)
        }
    }
    V(mutex)
    P(barreraAlumnos)
    //hacer tarea
    P(mutex2)
    c.push(tarea)
    V(mutex2)
    V(hayTrabajo) //despierto profesor
    P(puntaje[tarea])
    nota = nota[tarea]
}
Process Profesor{
    int nota = 10
    int tarea
    list grupos[10] = {[10]0}
    for (i=0, i<50, i++)
        P(hayTrabajo)
        P(mutex2)
        tarea = c.pop()
        V(mutex2)
        grupos[tarea]++ //cant de alumnos que terminaron x cada tarea
        if(grupos[tarea] == 5) //si llegó a 5 se les da la nota
            nota[tarea] = nota
            for (i=0, i<50, i++){
                V(puntaje[tarea])//despierto a todos los que tenian la tarea
            }
            nota--
}
```

## Ejercicio 8
```cpp
int empleados = 0, finalizados = 0, e
Sem mutex1 = 1, mutex2 = 1, mutex3 = 1, barreraEmpleados = 0, despertarEmpresa = 0, listo = 0
list empleados[E] = {[E]0}
Process Empleado[id:1.E]{
    //llega
    P(mutex)
    empleados++
    if (empleados == E)
        for (i=0, i<E, i++){
            V(barreraEmpleados)
        }
    V(mutex)
    P(barreraEmpleados)
    P(mutex2)
    while(piezas < T){
        piezas++
        V(mutex2)
        //labura
        P(mutex2)
        empleados[id]++ //para saber cuanto trabajó cda empleado
    }
    V(mutex2)
    P(mutex3)
    finalizados++
    if (finalizados == E)
        V(despertarEmpresa)
    V(mutex3)
    P(listo)
    if(id == e)
        //soy el mas capo
    else
        //no soy el mas capo
}
Process Empresa{
    P(despertarEmpresa)
    e = empleados.mayor()
    for (i=0, i<E, i++){
        V(listo)
    }
}
```
## Ejercicio 9
```cpp
Sem hayEspacioM = 30, hayEspacioV = 50, mutexMarcos = 1, mutexVidrios = 1, hayMarcos = 0, hayVidrios = 0
Process Carpintero[id:1..4]{
    while (true){
        //armar marco
        P(hayEspacioM)
        P(mutexMarcos)
        marcos.push(marco)
        V(mutexMarcos)
        V(hayMarcos)
    }
}
Process Vidriero{
    while (true){
        //armar vidrio
        P(hayEspacioV)
        P(mutexVidrios)
        vidrios.push(vidrio)
        V(mutexVidrios)
        V(hayVidrios)
    }
}
Process Armador[id:1..2]{
    while(true){
        P(hayMarcos)
        P(mutexMarcos)
        marco = marcos.pop()
        V(mutexMarcos)
        V(hayEspacioM)
        P(hayVidrios)
        P(mutexVidrios)
        vidrio = vidrios.pop()
        V(mutexVidrios)
        V(hayEspacioV)
        //armar ventana
    }
}
```
## Ejercicio 10

a) No esta aclarado en el enunciado pero tiene que hacerse la descarga de manera ordenada (re dificil sino)
```cpp
Sem cantmaxcamiones = 7, cantmaxtrigo = 5, cantmax = 5, mutexcola = 1, despertarcoordinador = 0, arrancardescarga[T+M] = {[T+M]0}
Cola cola
Process CamionT[id:1..T]{
    //llegar
    P(mutexcola)
    cola.push(id, tipo)
    V(mutexcola)
    V(despertarcoordinador)
    P(arrancardescarga)
    //descargar
    V(cantmaxcamiones)
    V(cantmaxtrigo)

}
Process CamionM[id:T+1..M+T]{
    //llegar
    P(mutexcola)
    cola.push(id, tipo)
    V(mutexcola)
    V(despertarcoordinador)
    P(arrancardescarga[id])
    //descargar
    V(cantmaxcamiones)
    V(cantmaxmaiz)
}
Process Coordinador{
    while (true){
        P(despertarcoordinador)
        P(cantmaxcamiones)
        P(mutexcola)
        camion = cola.pop()
        V(mutexcola)
        if (camion.tipo() == trigo)
            P(cantmaxtrigo)
        else
            P(cantmaxmaiz)
        V(arrancardescarga[camion.id()])
    }
}
```
b)
```cpp
Proces camionT[id:1..T]{
    //llega
    P(hayTrigo)
    P(hayLugar)
    //descargar
    V(hayTrigo)
    V(hayLugar)
}
Proces camionM[id:1..M]{
    //llega
    P(hayMaiz)
    P(hayLugar)
    //descargar
    V(hayMaiz)
    V(hayLugar)
}
```
## Ejercicio 11
```cpp
Sem despertarCoordinador = 0, mutexcola = 1, vacunando[50]
Cola cola
Process Coordinador{
    for(i=0,i<10,i++){
        P(despertarCoordinador)
        for(i=0,i<5,i++){
            id = cola.pop()
            VacunarPersona(id)
            V(vacunado[id])
        }
    }
}
Process Persona[id:1..50]{
    P(mutexcola)
    cola.push(id)
    if(cola.lenght() == 5)
        V(despertarcoordinador)
    V(mutexcola)
    P(vacunado[id])
}
```
## Ejercicio 12
a)
```cpp
Sem mutexcola = 1, despertarRecepcionista = 0, hisopados[150] = {[150]0}, mutexenfermera[3] = {[3]1}, mutexvector = 1
Cola cola
Cola enfermeras[3]
int vector[3] = {[3]0}
Process pasajero[id:1..150]{
    P(mutexcola)
    cola.push(id)
    V(mutexcola)
    V(despertarRecepcionista)
    P(hisopados[id])
}
Process recepcionista{
    for(i=0,i<150,i++){
        P(despertarRecepcionista)
        P(mutexcola)
        id = cola.pop()
        V(mutexcola)
        
        P(mutexvector)
        idEnfermera = min(vector) //mantengo un vector con las cantidades en cada cola para no tener q hacer mutex de todas las colas
        vector[idenfermera]++
        V(mutexvector)

        P(mutexenfermera[idenfermera])        
        enfermeras[idEnfermera].push(id)
        V(mutexenfermera[idenfermera])

        V(despertarenfermera[idenfermera])
    }
}
Process enfermera[id:1..3]{
    while (true){
        P(despertarenfermera[id])
        P(mutexenfermera[id])
        id = enfermeras[id].pop()
        V(mutexenfermera[id])
        id.Hisopar()
        V(hisopados[id])

        P(mutexvector)
        vector[id]--
        V(mutexvector)
    }
}
```
b)
```cpp
Sem hisopados[150] = {[150]0}, mutexenfermera[3] = {[3]1}, mutexvector = 1
Cola enfermeras[3]
int vector[3] = {[3]0}
Process pasajero[id:1..150]{
    P(mutexvector)
    idEnfermera = min(vector) 
    vector[idenfermera]++
    V(mutexvector)

    P(mutexenfermera[idenfermera])        
    enfermeras[idEnfermera].push(id)
    V(mutexenfermera[idenfermera])

    V(despertarenfermera[idenfermera])
    P(hisopados[id])
}

Process enfermera[id:1..3]{
    while (true){
        P(despertarenfermera[id])
        P(mutexenfermera[id])
        id = enfermeras[id].pop()
        V(mutexenfermera[id])
        id.Hisopar()
        V(hisopados[id])

        P(mutexvector)
        vector[id]--
        V(mutexvector)
    }
}
```