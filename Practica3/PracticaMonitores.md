# Practica 3 Monitores

## Ejercicio 1

a) Si, el código funciona correctamente
b) Si, se podría hacer mas simple ya que este ejercicio solo pide exclusion mutua y haciendo que el monitor represente el cruce ya la tendrías.

```cpp
Monitor Puente {
    Procedure cruzarPuente (){
        //cruzar puente
    }
}

Process Auto [a:1..M]{
    Puente.cruzarPuente();
}
```

c) No, no respeta el orden de llegada ya que cuando hay mas de 1 auto dormido todos compiten por el puente. La solución planteada en b) tampoco respeta el orden. 

## Ejercicio 2


```cpp
Monitor BaseDeDatos {
    int cant = 0; 
    cond cv;

    Procedure pedirAcceso(){
        while(cant == 5) wait(cv); 
        cant++;
    }

    Procedure salir(){
        cant--;
        signal(cv);
    }
}

Process Lector [p:1..N]{
    BaseDeDatos.pedirAcceso();
    //leer bd
    BaseDeDatos.salir();
}
```

## Ejercicio 3

a)

```cpp
Monitor Fotocopiadora {
    Procedure usar (){
        //fotocopiar
    }
}

Process Persona [id:1..N]{
    Fotocopiadora.usar();
}
```
b)

```cpp
Monitor Fotocopiadora {
    bool libre = true;
    cond cola;
    int esperando = 0;
    Procedure usar (){
        if (not libre){
            esperando++
            wait(cola)
        }
        else libre = false        
    }
    Procedure salir(){
        if (esperando > 0){
            esperando--
            signal(cola)
        }
        else libre = true
    }
}

Process Persona [id:1..N]{
    int edad
    Fotocopiadora.usar(id, edad);
    //fotocopiar
    Fotocopiadora.salir();
}
```
c)

```cpp
Monitor Fotocopiadora {
    bool libre = true;
    cond cola[N];
    cola colaOrdenada[N]
    Procedure usar (idP, edad: in int){
        if (not libre){
            insertar(colaOrdenada, idP, edad)
            wait(cola[idP])
        }
        else libre = false        
    }
    Procedure salir(){
        if (esperando > 0){
            sacar(colaOrdenada, aux)
            signal(cola[aux])
        }
        else libre = true
    }
}

Process Persona [id:1..N]{
    Fotocopiadora.usar();
    //fotocopiar
    Fotocopiadora.salir();
}
```
d)

```cpp
Monitor Fotocopiadora {
    int siguiente = 1
    cond cola[N];
    Procedure usar (idP, edad: in int){
        if (idP != siguiente)
            wait(cola[idP])     
    }
    Procedure salir(){
        siguiente++
        signal(cola[siguiente])
    }
}

Process Persona [id:1..N]{
    Fotocopiadora.usar();
    //fotocopiar
    Fotocopiadora.salir();
}
```

e)

```cpp
Monitor Fotocopiadora {
    cond persona;
    cond empleado;
    cond termine
    int esperando = 0;
    Procedure asignar(){
        if (esperando == 0)
            wait(empleado)
        esperando--
        signal(persona)
        wait(termine)
    }
    Procedure usar (){
        signal(empleado)
        esperando++
        wait(persona)
    }
    Procedure salir(){
        signal(termine)
    }
}

Process Persona [id:1..N]{
    fotocopiadora.usar()
    //fotocopiar
    fotocopiadora.dejar()
}

 Process Empleado {
    for (i=1; i<=N; i++)
        fotocopiadora.asignar()
}
```
f)

```cpp
Monitor Fotocopiadora{
    Cola libres = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    Cola asignada = [N][0]
    Cola llegados
    cond impresora;
    cond personas [n] 
    cond empleado;
    Procedure asignar(){
        if (llegados.empty()) 
            wait(empleado)
        aux= llegados.pop()
        if (libres.empty())
            wait(impresora)
        asignada[aux] = libres.pop()
        signal(personas[aux])
    }
    Procedure usar (idP: in int; fotocop: out int){
        llegados.push()
        signal(empleado)
        wait(personas[idP])
        fotocop = asignada[idP]
    }
    Procedure salir(fotocop: in int){
        libres.push(fotocop)
        signal(impresora)
    }
}

Process Persona [id:1..N]{
    fotocopiadora.usar(id)
    //fotocopiar
    fotocopiadora.dejar(fotocop)
}

Process Empleado {
    for (i=1; i<=N; i++)
        fotocopiadora.asignar()    
}
```
## Ejercicio 4
En este ejercicio necesito el booleano "Libre" y no puedo usar esperando!=0 para saber si quedarme en el wait o seguir. Ya que sino puede pasar que un auto pase en esperando=0 pero
luego se quede trabado en el while x su peso, la var esperando nunca se aumento por lo que puede pasar otro auto por el if y pasar el while xq su peso es menor. De esa forma no se 
respetaria el orden.

```cpp
Process Auto [id:1..N]{
    int peso;
    Puente.pasar(peso)
    //cruzar puente
    Puente.salir(peso)
}

Monitor Puente{
    int kg = 0
    int esperando = 0
    bool libre = true
    cond reintentar
    cond proximo

    Procedure Pasar(peso: in int,){
        if(!libre)
            esperando++
            wait(proximo)
        else
            libre = false
        while(kg + peso > 5000)
            wait(reintentar)
        kg = kg + peso
        if (esperando > 0)
            signal(proximo)
            esperando--
        else
            libre = true
    }

    Procesure Salir(peso: in int){
        kg = kg - peso
        signal(reintentar)
    }
}

```
## Ejercicio 5
a)
```cpp
Process Cliente[id:1..N]{
    text lista
    corralon.llegar(lista, comprobante)
}

Process Empleado{
    for (i=0, i<N, i++)
        corralon.llamarCliente(lista)
        comprobante = generarComprobante(lista) //funcion inventadisima
        corralon.entregarComprobante(comprobante)
}

Monitor Corralon{
    cond empleado
    cond cliente
    cond datos
    cond comprobante
    cond terminar
    int esperando = 0
    text auxlista
    text auxComprobante

    Procedure llegar(lista: in text, comprobante: out text){
        esperando++
        signal(empleado)
        wait(cliente)
        auxlista= lista
        signal(datos)
        wait(comprobante)
        comprobante = auxComprobante
        signal(terminar)
    }
    Procedure llamarCliente(lista: out text){
        if (esperando == 0)
            wait(empleado)
        esperando--
        signal(cliente)
        wait(datos)
        lista = auxlista
    }
    Procedure entregarComprobante(comprobante: in text){
        auxComprobante = comprobante
        signal(comprobante)
        wait(terminar)
    }
}

```

b)
```cpp
Process Cliente[id:1..N]{
    text lista
    corralon.llegar(idE) //espera a que se le asigne empleado
    mostrador[idE].atencion(lista, comprobante) //va al mostrador del empleado
}

Process Empleado[id:1..E]{
    text lista
    while(true)
        corralon.llamarCliente(id)
        mostrador[id].esperarLista(lista)
        comprobante = generarComprobante(lista) 
        mostrador[id].entregarComprobante(comprobante)
}

Monitor Corralon{
    cola elibres;
    cond esperaC;
    int esperando = 0, cantlibres = 0

    Procedure llegar(idE: out int){
        if(cantLibres == 0) //si no hay empleados libres espero a que haya
            esperando++
            wait(esperaC)
        else cantlibres-- 
        elibres.pop() //obtengo el id del empleado libre y se lo mando al cliente (out)
    }
    Procedure llamarCliente(idE: in int){
        elibres.push(idE) //agrego al empleado como libre
        if(esperando > 0) //si hay clientes esperando se los atiende
            esperando--
            signal(esperaC)
        else cantlibres++ //si no hay clientes esperando se aumenta la cant de empleados libres
    }
}

Monitor Mostrador[id:1..E]{
    cond cliente
    cond empleado
    text lis, comp
    boolean listo = false
    Procedure Atencion(lista: in text, comprobante: out text){
        list = lista
        listo = true //marca que llegó cliente
        signal(empleado)
        wait(cliente)
        comprobante = comp
        signal(empleado)
    }
    Procedure esperarLista(lista: out text){
        if(!listo) //solo se duerme si el cliente no llegó
            wait(empleado)
        lista = list
    }
    Procedure entregarComprobante(comprobante: in text){
        comp = comprobante
        signal(cliente)
        wait(empleado)
        listo = false //resetea para atender prox cliente
    }
}
```
c) 
```cpp
Process Empleado[id:1..E]{
    text lista
    termino= false
    while(!termino)
        corralon.llamarCliente(id, termino)
        if (!termino)
            mostrador[id].esperarLista(lista)
            comprobante = generarComprobante(lista) 
            mostrador[id].entregarComprobante(comprobante)
}
Monitor Corralon{
    cola elibres;
    cond esperaC;
    int esperando = 0, cantlibres = 0, atendidos = 0 //agrego var atendidos

    Procedure llegar(idE: out int){
        if(cantLibres == 0) 
            esperando++
            wait(esperaC)
        else cantlibres-- 
        elibres.pop() 
    }
    Procedure llamarCliente(idE: in int, termino: out bool){
        if (atendidos == N)
            termino = true //si atendidos = cant de clientes termino
        else
            atendidos++ //sino sumo atendidos
            elibres.push(idE) 
            if(esperando > 0) 
                esperando--
                signal(esperaC)
            else cantlibres++ 
    }
}

```

## Ejercicio 6
Uso 2 monitores para maximizar concurrencia
```cpp
Process Alumno[id: 1..50]{
    manejoGrupos.esperarTarea(id, tarea)
    //realiza tarea
    manejoNotas.terminar(tarea)
}
Process JTP{
    nota = 25
    list grupos[25][0]//lista para saber x cada tarea cuantos alumnos la completaron
    manejoGrupos.esperarAlumnos()
    for (i=0, i<50, i++){
        manejoGrupos.asignar(AsignarNroGrupo())
    }
    for(i=0, i<50, i++){
        manejoNotas.recibir(tarea)
        grupos[tarea]++ //aumento la cant de alumnos que terminaron la tarea
        if (grupos[tarea] == 2) //si los 2 alumnos con la misma tarea terminaron se les da la nota
            manejoNotas.entregarNota(nota)
            nota--
    }
}

Monitor manejoGrupos{
    int cant = 0
    list tareas [50] //lista con tarea x alumno
    Cola fila [50] //fila de alumnos
    cond espera [50] //alumnos esperan que ya le hayan dado su tarea
    Procedure esperarTarea(idA: in int, tarea: out int){
        cant++
        fila.push(idA)
        if(cant == 50) signal(jtp) //si ya llegaron todos los alumnos se despierta el jtp
        wait(espera[idA]) 
        tarea = tareas[idA]
    }
    Procedure esperarAlumnos(){
        if(cant < 50) wait(jtp)  //si todavia no llegaron todos los alumnos el jtp tiene q esperar q lo despierten     
    }
    Procedure asignar(tarea: in int){
        idA = fila.pop() //sig alumno
        tareas[idA]= tarea //le asigno la tarea
        signal(espera[idA]) //despierto alumno
    }
}
Monitor manejoNotas{
    Cola finalizadas [50] //cola de tareas finalizadas
    list puntajes [25][0] //lista de puntajes por tarea
    cond notas [25]
    cond completas
    Procedure terminar(tarea: in int, nota: out int){
        finalizadas.push(tarea)
        signal(completas) //aviso que hay tareas completas
        wait(notas[tarea]) 
        nota = puntajes[tarea]
    }
    Procedure recibir(tarea: out int){
        if (finalizadas.lenght() = 0)
            wait(completas) //si no hay tareas completas jtp espera a que haya
        tarea= finalizadas.pop() //saco la tarea para que el jtp la corrija (out)
    }
    Procedure entregarNota(nota: in int, tarea in int){
        puntajes[tarea]= nota //asigna nota a la tarea
        siganlall(notas[nota]) //despierto a los 2 alumnos que estaban esperando por su nota
    }
}
```
## Ejercicio 7
```cpp
Process Corredores[id:1..C]{
    carrera.esperar()
    carrera.iniciar()
    //Correr
    filaDispenser.esperarAgua()
    dispenser.obtenerAgua()
    filaDispenser.salirMaquina()
}

Process Repositor{
    while true
        dispenser.reponer()
}
Monitor Carrera{
    int cant = 0
    int esperando
    bool libre
    cond buscar
    cond iniciar
    Cola corredores [C]
    Procedure esperar(){
        cant++
        if(cant == C)
            signal_all(iniciar)
    }
    Procedure iniciar(){
        if (cant < C) wait(iniciar)
    }
}
Monitor filaDispenser{
    Procedure esperarAgua(){
        if(libre = false)
            esperando++
            wait(buscar)
        else
            libre = true
    }
    Procedure salirMaquina(){
        if (esperando > 0)
            signal(buscar)
            esperando--
        else
            libre = true
    }
}
Monitos Dispenser{
    int botellas = 20
    Procedure obtenerAgua(){
        if (botellas = 0)
            signal(pedirRecarga)
            wait(recarga)
        botellas--
    }
    Procedure reponer(){
        wait(reponer)
        botellas = 20
        signal(recarga)
    }

}
```
## Ejercicio 8

```cpp
Process Jugador[id:1..20]{
    equipo[DarEquipo()].esperar(c)
    cancha[c].esperar()
}

Process Partido[id:1..2]{
    cancha[id].jugar()
    dalay(50)
    cancha[id].terminar()
}

Monitor Equipo[id:1..4]{
    int cantJugadores = 0
    cond jugadores, listo
    Procedure llegar(cancha: out int){
        cantJugadores++
        if(cantJugadores == 5)
            asignar.equipoListo(cancha)
            signal_all(jugadores)
        else
            wait(jugadores)
    }    
}

Monitor Asignar{
    int num = 1
    int esperando = 0
    cond otroEquipo
    Procedure equipoListo(cancha: out int){
        if(esperando == 1)
            signal(otroEquipo)
        else
            esperando++
        if (esperando <= 1)
            wait(otroEquipo)
        esperando--
        cancha = num
        num++
    }
}

Monitor Cancha[id:1..2]{
    int cantJugadores = 0
    Procedure esperar(){
        canJugadores++
        if (cantJugadores == 10)
            signal(inicio)
        else
            wait(jugadores)          
    }    
    Procedure jugar(){
        if(cantJugadores < 10)
            wait(inicio)
    }
    Procedure terminar(){
        signal_all(jugadores)
    }   
}

```

## Ejercicio 9
```cpp
Process preceptor{
    repartir.esperarAlumnos(enunciado)
}

Process profesora{
    for(i=0, i<45, i++)
        correccion.obtenerExamen(examen)
        nota = corregirExamen(examen)
        notas.darCalificacion(nota)
}

Process alumno [id:1..45]{
    repartir.esperarExamen(e)
    //resolver examen
    correccion.terminarExamen()
    notas.obtenerCalificacion(e, nota)
    correccion.liberarProfe()
}

Monitor Repartir{
    int enunciado
    int cantA = 0
    cond darEnunciado, preceptor
    Procedure esperarAlumnos(e: in int){
        if (cantA < 45) wait(darEnunciado)
        enunciado = e
        signal_all(preceptor)
    }
    Procedure esperarExamen(e: out int){
        cantA++
        if(cantA == 45)
            signal(darEnunciado)
        wait(preceptor)
        e = enunciado
    }
}

Monitor Correccion{
    int esperando = 0
    text examen
    cond profesora, turno
    Procedure liberarProfe(e: out text){
        if (esperando > 0)
            signal(turno)
            esperando--
        else
            profeLibre = true
    }
    Procedure terminarExamen(){
        if(!profeLibre)
            esperando++
            wait(turno)
        else
            profeLibre = false       
    }
}

Monitor Notas{
    int correccion
    bool entrega = false
    cond entregar, calificacion
    Procedure obtenerExamen(){
        if(!entrega)
            wait(entregar)
        e = examen
    }
    Procedure obtenerCalificacion(e: in text, nota: out int){
        examen = e
        entrega= true
        signal(entregar)
        wait(calificacion)
        nota = correccion
    }
    Procedure darCalificacion(nota: in int){
        correccion = nota
        signal(calificacion)
        entrega = false //vuelvo a poner entrega en falso para el prox
    }
}
```
## Ejercicio 10
El enunciado me hizo entender que el empleado esperaba a que la persona llegue para recien limpiarlo (alto garca) y no lo voy a cambiar
```cpp
Process Persona [id:1..N]{
    juego.solicitar()
    Usar_juego()
    juego.dejar()
}

Process Empleado{
    while(true)
        juego.esperarPersona()
        Desinfectar_juego()
        juego.entregar()
}

Monitor Juego{
    cond juego, empleado, desinfeccion
    bool juegoLibre
    int esperando
    Procedure solicitar(){
        if(!juegoLibre)
            esperando++
            wait(juego)
        else
            juegoLibre = false
        signal(empleado)
        wait(desinfeccion)
    }
    Procedure dejar(){
        if (esperando == 0)
            juegoLibre = true
        else
            signal(juego)
            esperando--
    }
    Procedure esperarPersona(){
        if(juegoLibre) 
            wait(empleado)      
    }
    Procedure entregar(){
        signal(desinfeccion)
    }
}
```
