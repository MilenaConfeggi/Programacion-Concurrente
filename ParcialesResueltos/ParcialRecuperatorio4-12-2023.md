# Parcial resuperatorio 2023
## Ejercicio 1 (semáforos)
Un sistema debe validar un conjunto de 10000 transacciones que se encuentran disponibles en una estructura de
datos. Para ello, el sistema dispone de 7 workers, los cuales trabajan colaborativamente validando de a 1
transacción por vez cada uno. Cada validación puede tomar un tiempo diferente y para realizarla los workers
disponen de la función Validar(t), la cual retorna como resultado un número entero entre 0 al 9. Al finalizar el
procesamiento, el último worker en terminar debe informar la cantidad de transacciones por cada resultado de la
función de validación. Nota: maximizar la concurrencia.
```cpp
Sem cantDisponibles = 1, cantTransacciones = 1, llegados = 1
int trabajadores = 0
int resultados[9] = {[9]0}
Process Workers[id:1..7]{
    P(cantDisponibles)
    while(cant < 10000){
        cant++
        t = transacciones.pop()
        V(cantDisponibles)
        nro = Validar(t)
        P(cantTransacciones)
        resultados[nro]++
        V(cantTransacciones)
        P(cantDisponibles)
    }
    V(cantDisponibles)
    P(llegados)
    trabajadores++
    if(trabajadores == 7){
        for(i=0, i<10, i++){
            Informar(resultados[i])
        }
    }
    V(llegada)
}
```
## Ejercicio 2 (monitores)
En una empresa trabajan 20 vendedores ambulantes que forman 5 equipos de 4 personas cada uno (cada vendedor
conoce previamente a que equipo pertenece). Cada equipo se encarga de vender un producto diferente. Las
personas de un equipo se deben juntar antes de comenzar a trabajar. Luego cada integrante del equipo trabaja
independientemente del resto vendiendo ejemplares del producto correspondiente. Al terminar cada integrante del
grupo debe conocer la cantidad de ejemplares vendidos por el grupo. Nota: maximizar la concurrencia.
```cpp
Process Vendedores[id:1..20]{
    equipo[id].reunirse()
    cantVendidos = vender()
    equipo[id].terminar(cantVendidos, total)
}
Monitor Equipos[id:1..5]{
    int esperando = 0, terminados = 0, result = 0
    cond grupo
    Procedure reunirse(){
        esperando++
        if(esperando < 4){
            wait(grupo)
        }
        else{
            signal_all(grupo)
        }
    }
    Procedure terminar(cantVendidos: in int, total: out int){
        terminados++
        result += cantVendidos
        if(terminados < 4){
            wait(grupo)
        }
        else{
            signal_all(grupo)
        }
        total = result
    }
}
```
