# Pracitica PMA
## Ejercicio 1
Suponga que N clientes llegan a la cola de un banco y que serán atendidos por sus
empleados. Analice el problema y defina qué procesos, recursos y canales/comunicaciones
serán necesarios/convenientes para resolverlo. Luego, resuelva considerando las siguientes
situaciones:
a. Existe un único empleado, el cual atiende por orden de llegada.
```cpp
chan canal(texto)
process Persona[id:1..N]{
    texto solicitud
    send canal(solicitud)
}

Process Empleado{
    texto soli
    while true{
        receive canal(soli)
        atender(soli)
    }

}
```
b. Ídem a) pero considerando que hay 2 empleados para atender, ¿qué debe
modificarse en la solución anterior?
```cpp
chan canal(texto)
process Persona[id:1..N]{
    texto solicitud
    send canal(solicitud)
}

Process Empleado[id:1..2]{
    texto soli
    while true{
        receive canal(soli)
        atender(soli)
    }

}
```
c. Ídem b) pero considerando que, si no hay clientes para atender, los empleados
realizan tareas administrativas durante 15 minutos. ¿Se puede resolver sin usar
procesos adicionales? ¿Qué consecuencias implicaría?
```cpp
chan canal(texto)
chan pedido(int)
chan siguiente[3](texto)

Process Persona[id:1..N]{
    send canal(id)
}

Process Coordinador{
    while true{
        receive pedido(idE)
        if(empty(canal))
            rep = "vacio"
        else
            receive canal(rep)
        send siguiente[idE](rep)
    }
}

Process Empleado[id:1..2]{
    texto soli
    while true{
        send pedido(id)
        receive siguiente[id](respuesta)
        if(respuesta != "vacio")
            atender(respuesta)
        else
            delay()
    }

}
```
## Ejercicio 2
Se desea modelar el funcionamiento de un banco en el cual existen 5 cajas para realizar
pagos. Existen P clientes que desean hacer un pago. Para esto, cada una selecciona la caja
donde hay menos personas esperando; una vez seleccionada, espera a ser atendido. En cada
caja, los clientes son atendidos por orden de llegada por los cajeros. Luego del pago, se les
entrega un comprobante. Nota: maximizar la concurrencia

```cpp
chan solicitudes(int)
chan cajaasignada[P](int)
chan siguiente[3](int)
chan comprobantes[P](texto)
chan listo(int)

Process Cliente[id:1..P]{
    send solicitudes(id)
    send HayPedido(true)
    recive cajaasignada[id](caja)
    send siguiente[caja](id)

    receive comprobantes[id](comp)
    send listo(caja)
    send HayPedido(true)
}

Process Coordinador{
    int cantidades[3]
    while true{
        receive HayPedido(pedido)
        //deberia darle prioridad a sacar de la caja para que sea mas realistael nro?
        if(!empty(listo))
            receive listo(caja)
            cantidades[caja]--
        else if(!empty(solicitudes))
            receive solicitudes(idCl)
            min = cantidades.minimo()
            cantidades[min]++
            send cajaasignada[idCl](min)
    }

}

Process Caja[id:1..5]{
    while true{
        receive siguiente[id](idCl)
        comp = atender(id)
        send comprobantes[idCl](comp)
    }
}

```

## Ejercicio 3
Se debe modelar el funcionamiento de una casa de comida rápida, en la cual trabajan 2
cocineros y 3 vendedores, y que debe atender a C clientes. El modelado debe considerar
que:
- Cada cliente realiza un pedido y luego espera a que se lo entreguen.
- Los pedidos que hacen los clientes son tomados por cualquiera de los vendedores y se
lo pasan a los cocineros para que realicen el plato. Cuando no hay pedidos para atender,
los vendedores aprovechan para reponer un pack de bebidas de la heladera (tardan entre
1 y 3 minutos para hacer esto).
- Repetidamente cada cocinero toma un pedido pendiente dejado por los vendedores, lo
cocina y se lo entrega directamente al cliente correspondiente.
Nota: maximizar la concurrencia.

```cpp
chan pedidos(int)
chan comida[C](texto)
chan solicitud(int)
chan platoacocinar(int)
chan atender[3](int)
Process Cliente[id:1..C]{
    send pedidos(id)
    recive comida[id](plato)
}

Process Coordinador{
    while true{
        receive solicitud(idV)
        if(empty(pedidos))
            idC = -1
        else
            receive pedidos(idC)
        send atender[idV](idC) //le digo al vendedor idV que atienda al cliente idC o que no hay nadie esperando a ser atendido
    }
}

Process Vendedor[id:1..3]{
    while true{
        send solicitud(id)
        receive atender[id](idC)
        if(idC == -1)
            delay()
        else
            send platoacocinar(idC)
    }
}

Process Cocinero[id:1..2]{
    while true{
        receive platoacocinar(idC)
        plato = cocinar()
        send comida[idC](plato)
    }
}
```

## Ejercicio 4
Simular la atención en un locutorio con 10 cabinas telefónicas, el cual tiene un empleado
que se encarga de atender a N clientes. Al llegar, cada cliente espera hasta que el empleado
le indique a qué cabina ir, la usa y luego se dirige al empleado para pagarle. El empleado
atiende a los clientes en el orden en que hacen los pedidos. A cada cliente se le entrega un
ticket factura por la operación.
a) Implemente una solución para el problema descrito.

```cpp
chan solicitud[N](int)
chan cabinaausar[N](int)
chan termine(int, int)
chan tickets[N](text)
Process Cliente[id:1..N]{
    send solicitud(id)
    receive cabinaausar[id](cabina)
    //usar cabina
    send termine(id, cabina)
    receive tickets[id](ticket)
}

Process Empleado{
    int cabinas[10]
    while true{
        if(hayCabinaLibre(cabinas) && !empty(solicitud))
            receive solicitud(idC)
            send cabinaausar[idC](AsignarCabina(cabinas))
        else if (!empty termine)
            receive termine(idC, cabina)
            send tickets[idC](generarTicket())
            liberarCabina(cabina)
    }
}

```
b) Modifique la solución implementada para que el empleado dé prioridad a los que
terminaron de usar la cabina sobre los que están esperando para usarla.
Nota: maximizar la concurrencia; suponga que hay una función Cobrar() llamada por el
empleado que simula que el empleado le cobra al cliente.

```cpp
//La unica diferencia con el a es dar vuelta el if

chan solicitud[N](int)
chan cabinaausar[N](int)
chan termine(int, int)
chan tickets[N](text)
Process Cliente[id:1..N]{
    send solicitud(id)
    receive cabinaausar[id](cabina)
    //usar cabina
    send termine(id, cabina)
    receive tickets[id](ticket)
}

Process Empleado{
    int cabinas[10]
    while true{
        if (!empty termine)
            receive termine(idC, cabina)
            send tickets[idC](generarTicket())
            liberarCabina(cabina)
        else if(hayCabinaLibre(cabinas) && !empty(solicitud))
            receive solicitud(idC)
            send cabinaausar[idC](AsignarCabina(cabinas))
    }
}
```
## Ejercicio 5
Resolver la administración de 3 impresoras de una oficina. Las impresoras son usadas por N
administrativos, los cuales están continuamente trabajando y cada tanto envían documentos
a imprimir. Cada impresora, cuando está libre, toma un documento y lo imprime, de
acuerdo con el orden de llegada.
a) Implemente una solución para el problema descrito.
b) Modifique la solución implementada para que considere la presencia de un director de
oficina que también usa las impresas, el cual tiene prioridad sobre los administrativos.
c) Modifique la solución (a) considerando que cada administrativo imprime 10 trabajos y
que todos los procesos deben terminar su ejecución.
d) Modifique la solución (b) considerando que tanto el director como cada administrativo
imprimen 10 trabajos y que todos los procesos deben terminar su ejecución.
e) Si la solución al ítem d) implica realizar Busy Waiting, modifíquela para evitarlo.
Nota: ni los administrativos ni el director deben esperar a que se imprima el documento.

a)
```cpp
chan pedido(text)
Process Administrativo[id:1..N]{
    send pedido(documento)
}

Process Impresora[id:1..3]{
    while true{
        receive pedido(documento)
        imprimir()
    }

}
```
b)
```cpp
chan pedido(text)
chan pedidoprioritario(text)
chan solicitud(bool)
chan colaimpresion[3](text)
chan estoylibre(int)
Process Administrativo[id:1..N]{
    send pedido(documento)
    send solicitud(true)
}

process director{
    send pedidoprioritario(documento)
    send solicitud(true)
}

process coordinador{
    while true{
        receive estoylibre(idI)
        receive solicitud(solicitud)
        if(!empty(pedidoprioritario))
            receive pedidoprioritario(documento)
        else 
            receive pedido(documento)
        send colaimpresion[idI](documento)
    }
}
Process Impresora[id:1..3]{
    while true{
        send estoylibre(id)
        receive colaimpresion[id](documento)
        imprimir()
    }

}
```

c)
```cpp
chan pedido(text)
chan contador(int)
Process Administrativo[id:1..N]{
    send pedido(documento)
}

Process Impresora[id:1..3]{
    if(id == 1)
        send contador(0)
    while (!listo){
        receive contador(num)
        if (num == N)
            listo= true
            send contador(num)
        else
            num= num +1
            send contador(num)
            receive pedido(documento)
            imprimir(documento)
    }

}
```

d)
```cpp
chan pedido(text)
chan pedidoprioritario(text)
chan solicitud(bool)
chan colaimpresion[3](text)
chan estoylibre(int)
Process Administrativo[id:1..N]{
    for(i=0, i<10, i++)
        send pedido(documento)
        send solicitud(true)
}

process director{
    for(i=0, i<0, i++)
        send pedidoprioritario(documento)
        send solicitud(true)
}

process coordinador{
    int contador = 0
    for(i=0, i<N+10, i++){
        receive estoylibre(idI)
        receive solicitud(solicitud)
        if(!empty(pedidoprioritario))
            receive pedidoprioritario(documento)
        else 
            receive pedido(documento)

        send colaimpresion[idI](documento, false)
    }
    for(i=0,i<3,i++)
        receive estoylibre(algo)
        send colaimpresion[i](documento, true)
}

Process Impresora[id:1..3]{

    while(!booleano){
        send estoylibre(id)
        receive colaimpresion[id](documento)(booleano)
        imprimir()
    }

}
```