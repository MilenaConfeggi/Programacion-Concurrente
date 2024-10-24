# Pracitica PMS
## Ejercicio 1
Suponga que existe un antivirus distribuido que se compone de R procesos robots
Examinadores y 1 proceso Analizador. Los procesos Examinadores están buscando
continuamente posibles sitios web infectados; cada vez que encuentran uno avisan la
dirección y luego continúan buscando. El proceso Analizador se encarga de hacer todas las 
pruebas necesarias con cada uno de los sitios encontrados por los robots para determinar si
están o no infectados.
a) Analice el problema y defina qué procesos, recursos y comunicaciones serán
necesarios/convenientes para resolverlo.
b) Implemente una solución con PMS sin tener en cuenta el orden de los pedidos.
c) Modifique el inciso (b) para que el Analizador resuelva los pedidos en el orden
en que se hicieron.

b)
```cpp
Process Robot[id:1..R]{
    while(true){
        sitioweb = buscarbug()
        coordinador!aviso(sitioweb)
    }

}
Process Coordinador{
    cola avisos
    do robot[*]?aviso(sitioweb) -> avisos.push(sitioweb)
    [] notempty(avisos); analizador?pedido() -> analizador!aviso(avisos.pop())
}
Process Analizador{
    while(true){
        admin!pedido()
        admin?aviso(sitioweb)
        analizar(sitioweb)
    }
}
```
## Ejercicio 2
En un laboratorio de genética veterinaria hay 3 empleados. El primero de ellos
continuamente prepara las muestras de ADN; cada vez que termina, se la envía al segundo
empleado y vuelve a su trabajo. El segundo empleado toma cada muestra de ADN
preparada, arma el set de análisis que se deben realizar con ella y espera el resultado para
archivarlo. Por último, el tercer empleado se encarga de realizar el análisis y devolverle el
resultado al segundo empleado. 
```cpp
Process EmpleadoUno{
    while(true){
        muestra = generarMuestra()
        Coordinador!muestra(muestra)
    }
}
Process EmpleadoDos{
    muestra, analisis
    while(true){
        Coordinador!recibir()
        Coordinador?muestras(muestra)
        EmpleadoTres!muestra(armarKit(muestra))
        EmpleadoTres?analisis(analisis)
        archivar(analisis)
    }
}
Process EmpleadoTres{   
    text muestra
    while(true){
        EmpleadoDos?muestra(muestra)
        analisis = analizar(muestra)
        EmpleadoDos!analisis(analisis)
    }
}
Process Coordinador{
    cola muestras
    do
        EmpleadoUno?muestra(muestra) -> muestra.push(muestra)
        [] !muestras.empty; EmpleadoDos?recibir() -> EmpleadoDos!muestra(muestras.pop())
    od
}
```

## Ejercicio 3
En un examen final hay N alumnos y P profesores. Cada alumno resuelve su examen, lo
entrega y espera a que alguno de los profesores lo corrija y le indique la nota. Los
profesores corrigen los exámenes respetando el orden en que los alumnos van entregando.
a) Considerando que P=1.
b) Considerando que P>1.
c) Ídem b) pero considerando que los alumnos no comienzan a realizar su examen hasta
que todos hayan llegado al aula.
Nota: maximizar la concurrencia; no generar demora innecesaria; todos los procesos deben
terminar su ejecución

a)
```cpp
Process Alumno[id:1..N]{
    //Hace el examen
    Coordinador!examenes(examen,id);
    Profesor?notas(nota);
}
Process Coordinador{
    cola examenes
    int contadorExamenes = 0
    do
        contadorExamenes != N; Alumno[*]?examenes(examen,idA) -> examenes.push(examen,idA)
        [] !examenes.empty; Profesor?recibir() -> Profesor!pedidos(examenes.pop())
    od
}
Process Profesor{
    for (int i =0; i< N; i++){
        Coordinador!recibir();
        Coordinador?pedidos(examen,idA);
        int nota = corregirExamen(examen);
        Alumno[idA]!notas(nota);
    }
}
```

b)
```cpp
Process Alumno[id:1..N]{
    //Hace el examen
    Coordinador!examenes(examen,id);
    Profesor[*]?notas(nota);
}
Process Coordinador{
    cola examenes
    int contadorExamenes = 0
    do
        contadorExamenes != N; Alumno[*]?examenes(examen,idA) -> examenes.push(examen,idA)
        [] !examenes.empty; Profesor?recibir() -> Profesor!pedidos(examenes.pop())
    od
    for(i=0, i<P, i++)
        profesor[*]?recibir(idP)
        profesor[idP]!pedidos(null)
}
Process Profesor[id:1..P]{
    examen = ""
    while(examen != null){
        Coordinador!recibir(id);
        Coordinador?pedidos(examen,idA);
        if(examen != null)
            int nota = corregirExamen(examen);
            Alumno[idA]!notas(nota);
    }
}
```
c)
```cpp
Process Alumno[id:1..N]{
    //Hace el examen
    Barrera!llegue()
    Barrera?activar()
    Coordinador!examenes(examen,id);
    Profesor[*]?notas(nota);
}
Process Barrera{
    for(i=1, i<= N, i++)
        Alumno[*]?llegue()
    for(i=1, i<= N, i++)
        Alumno[i]!activar()
}
Process Coordinador{
    cola examenes
    int contadorExamenes = 0
    do
        contadorExamenes != N; Alumno[*]?examenes(examen,idA) -> examenes.push(examen,idA)
        [] !examenes.empty; Profesor?recibir() -> Profesor!pedidos(examenes.pop())
    od
    for(i=0, i<P, i++)
        profesor[*]?recibir(idP)
        profesor[idP]!pedidos(null)
}
Process Profesor[id:1..P]{
    examen = ""
    while(examen != null){
        Coordinador!recibir(id);
        Coordinador?pedidos(examen,idA);
        if(examen != null)
            int nota = corregirExamen(examen);
            Alumno[idA]!notas(nota);
    }
}
```

## Ejercicio 4
En una exposición aeronáutica hay un simulador de vuelo (que debe ser usado con
exclusión mutua) y un empleado encargado de administrar su uso. Hay P personas que
esperan a que el empleado lo deje acceder al simulador, lo usa por un rato y se retira.
a) Implemente una solución donde el empleado sólo se ocupa de garantizar la exclusión
mutua.
b) Modifique la solución anterior para que el empleado considere el orden de llegada para
dar acceso al simulador.
Nota: cada persona usa sólo una vez el simulador. 

a)
```cpp
Process Empleado{
    while(true){
        persona[*]?solicitar(idP)
        persona[idP]!recibir()
        persona[idP]?liberar()
    }

}
Process Persona[id:1..P]{
    empleado!solicitar(id)
    empleado?recibir()
    usar()
    empleado!liberar()
}
```
b)
```cpp
Process Empleado{
    bool libre = true
    do persona[*]?solicitar(idP) -> if libre libre=false; persona[idP]!recibir() else cola.push(idP)
    [] persona[idP]?liberar(); -> if cola.empty() libre=true else persona[cola.pop()]!recibir() 
}

Process Persona[id:1..P]{
    empleado!solicitar(id)
    empleado?recibir()
    usar()
    empleado!liberar()
}
```

## Ejercicio 5
En un estadio de fútbol hay una máquina expendedora de gaseosas que debe ser usada por
E Espectadores de acuerdo con el orden de llegada. Cuando el espectador accede a la
máquina en su turno usa la máquina y luego se retira para dejar al siguiente. Nota: cada
Espectador una sólo una vez la máquina.

```cpp
Process Espectador[id:1..E]{
    coordinador!solicitar(id)
    coordinador?recibir()
    usar()
    coordinador!liberar()
}
Process Coordinador{
    bool libre = true
    do espectador[*]?solicitar(idE) -> if libre libre=false; espectador[idE]!recibir() else cola.push(idE)
    [] espectador[idE]?liberar(); -> if cola.empty() libre=true else espectador[cola.pop()]!recibir() 
}
```