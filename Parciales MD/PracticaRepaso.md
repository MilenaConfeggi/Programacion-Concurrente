# Practica de repaso
## Pasaje de mensajes
## Ejercicio 1
En una oficina existen 100 empleados que envían documentos para imprimir en 5 impresoras
compartidas. Los pedidos de impresión son procesados por orden de llegada y se asignan a la primera
impresora que se encuentre libre:
a) Implemente un programa que permita resolver el problema anterior usando PMA.
b) Resuelva el mismo problema anterior pero ahora usando PMS.

```cpp
chan documentos(text)
Process Empleado[id:1..100]{
    documento = generarDocumento()
    send documentos(documento)
}

Process Coordinador{
    while(true)
        receive libres(idI)
        receive documentos(documento)
        send impresion[idI](documento)
}

Process Impresora[id:1..5]{
    while(true)
        send libres(id)
        receive impresion[id](documento)
        imprimir(documento)
}
```

```cpp
Process Empleado[id:1..100]{
    documento = generarDocumento()
    Coordinador!pedidos(documento)
}

Process Coordinador{
    do Empleado[*]?pedidos(documento) -> cola.push(documento)
    [] !cola.empty; Impresora[*]?libre(idI) -> Impresora[idI]!documentos(cola.pop())
}

Process Impresora[id:1..5]{
    while(true)
        Coordinador!libre(id)
        Coordinador?documentos(documento)
        imprimir(documento)
}
```

## Ejercicio 2
Resolver el siguiente problema con PMS. En la estación de trenes hay una terminal de SUBE que
debe ser usada por P personas de acuerdo con el orden de llegada. Cuando la persona accede a la
terminal, la usa y luego se retira para dejar al siguiente. Nota: cada Persona usa sólo una vez la
terminal. 

```cpp
Process Persona[id:1..P]{
    Terminal!pedidos(id)
    Terminal?acceso()
    //usar
    Terminal!liberar()
}

Process Terminal{
    bool libre = false
    do Persona[*]?pedidos(idP) -> 
        if libre
            Persona[idP]!acceso()
        else
            cola.push(idP)

    [] Persona[*]?liberar(); -> 
        if(cola.empty)
            libre = true
        else
            Persona[cola.pop()]!acceso()
}

```

## Ejercicio 3
Resolver el siguiente problema con PMA. En un negocio de cobros digitales hay P personas que
deben pasar por la única caja de cobros para realizar el pago de sus boletas. Las personas son
atendidas de acuerdo con el orden de llegada, teniendo prioridad aquellos que deben pagar menos
de 5 boletas de los que pagan más. Adicionalmente, las personas embarazadas tienen prioridad sobre
los dos casos anteriores. Las personas entregan sus boletas al cajero y el dinero de pago; el cajero les
devuelve el vuelto y los recibos de pago.

```cpp
chan menorPrioridad(int, text)
chan mediaPrioridad(int, text)
chan altaPrioridad(int, text)
chan pedido()
chan docs[P](text, text)
Process Persona[id:1..P]{
    int prioridad
    if(prioridad == 3)
        send menorPrioridad(id, boleta)
    elif(prioridad == 2)
        send mediaPrioridad(id, boleta)
    else
        send altaPrioridad(id, boleta)
    send pedido()
    receive docs[id](vuelto, recibo)
}

Process Caja{
    while(true)
        receive pedido()
        if(!altaPrioridad.empty())
            receive altaPrioridad(idP, boleta)
        elif(!mediaPrioridad.empty())
            receive mediaPrioridad(idP, boleta)
        else
            receive menorPrioridad(idP, boleta)

        vuelto, recibo = resolver(boleta)
        send docs[idP](vuelto, recibo)
        
}
```

## ADA
## Ejercicio 1
Resolver el siguiente problema. La página web del Banco Central exhibe las diferentes cotizaciones
del dólar oficial de 20 bancos del país, tanto para la compra como para la venta. Existe una tarea
programada que se ocupa de actualizar la página en forma periódica y para ello consulta la cotización
de cada uno de los 20 bancos. Cada banco dispone de una API, cuya única función es procesar las
solicitudes de aplicaciones externas. La tarea programada consulta de a una API por vez, esperando
a lo sumo 5 segundos por su respuesta. Si pasado ese tiempo no respondió, entonces se mostrará
vacía la información de ese banco.
```java
Procedure Pagina is
TASK Tarea
Task body Tarea is
guardar: array(1..20) of text;
Begin
    loop
        FOR i in 1..20 loop
            SELECT
                bancos[i].pedido(rta)
                guardar[i]:=rta
            OR DELAY 5
                guardar[i]:=null
        end loop;
        actualizar(guardar)
    end loop;
end tarea;

TASK TYPE Banco is
    ENTRY pedido(respuesta: OUT text)
end banco;
bancos: array(1..20) of Banco
TASK body Banco
respuesta: text
Begin
    loop
        ACCEPT pedido(respuesta: OUT text) do
            respuesta:= procesar()
        end pedido;
    end loop;
end banco

Begin
    null
end pagina;
```

## Ejercicio 2
Resolver el siguiente problema. En un negocio de cobros digitales hay P personas que deben pasar
por la única caja de cobros para realizar el pago de sus boletas. Las personas son atendidas de acuerdo
con el orden de llegada, teniendo prioridad aquellos que deben pagar menos de 5 boletas de los que
pagan más. Adicionalmente, las personas ancianas tienen prioridad sobre los dos casos anteriores.
Las personas entregan sus boletas al cajero y el dinero de pago; el cajero les devuelve el vuelto y los
recibos de pago. 
```java
Procedure Negocio is
TASK Caja is
    ENTRY bajaprioridad(pedido: IN text, vuelto: OUT text, recibo: OUT text)
    ENTRY mediaprioridad(pedido: IN text, vuelto: OUT text, recibo: OUT text)
    ENTRY altaprioridad(pedido: IN text, vuelto: OUT text, recibo: OUT text)
end caja;
TASK BODY Caja is
Begin
    loop
        SELECT
            when(altaprioridad.count = 0 && mediaprioridad.count = 0) -> ACCEPT bajaprioridad(pedido: IN text, vuelto: OUT text, recibo: OUT text) do
                vuelto, recibo:= procesar(pedido)                 
            end bajaprioridad;
        OR
            when(altaprioridad.count = 0) -> ACCEPT mediaprioridad(pedido: IN text, vuelto: OUT text, recibo: OUT text)
            do
                vuelto, recibo:= procesar(pedido) 
            end mediaprioridad;
        OR
            ACCEPT altaprioridad(pedido: IN text, vuelto: OUT text, recibo: OUT text)
            do
                vuelto, recibo:= procesar(pedido)
            end altaprioridad;
        end select;
    end loop;
end caja;

TASK TYPE Persona;
personas: array(1..P) of Persona
TASK BODY Persona is
prioridad: int
Begin
    if(prioridad == 1)
        Caja.altaprioridad(pedido, vuelto, recibo)
    elif(prioridad == 2)
        Caja.mediaprioridad(pedido, vuelto, recibo)
    else
        Caja.bajaprioridad(pedido, vuelto, recibo)
    end if;
end persona;
Begin
    null
end negocio;

```

## Ejercicio 3
Resolver el siguiente problema. La oficina central de una empresa de venta de indumentaria debe
calcular cuántas veces fue vendido cada uno de los artículos de su catálogo. La empresa se compone
de 100 sucursales y cada una de ellas maneja su propia base de datos de ventas. La oficina central
cuenta con una herramienta que funciona de la siguiente manera: ante la consulta realizada para un
artículo determinado, la herramienta envía el identificador del artículo a las sucursales, para que cada
una calcule cuántas veces fue vendido en ella. Al final del procesamiento, la herramienta debe
conocer cuántas veces fue vendido en total, considerando todas las sucursales. Cuando ha terminado
de procesar un artículo comienza con el siguiente (suponga que la herramienta tiene una función
generarArtículo() que retorna el siguiente ID a consultar). Nota: maximizar la concurrencia. Existe
una función ObtenerVentas(ID) que retorna la cantidad de veces que fue vendido el artículo con
identificador ID en la base de la sucursal que la llama.
```java
Procedure VentaDeIndumentaria is

    Task Central is
        Entry recibirVentas(ventas: IN integer);
    End Central;
    Task body Central is
        articulo: integer;
        total: integer;
    Begin
        loop
            total:= 0;
            articulo:= generarArticulo();
            for i in 1..100 loop
                sucursales[i].conseguirVentas(articulo);
            end loop;
            for i in 1..100 loop
                Accept recibirVentas(ventas: IN integer) do
                    total:= total + ventas;
                end recibirVentas;
            end loop;
        end loop;
    End Central;

    Task Type Sucursal is
        Entry conseguirVentas(articulo: IN integer);
    End Sucursal;

    sucursales: array (1..100) of Sucursal;

    Task body Sucursal 
        id: integer;
        ventas: integer;
    Begin
        loop
            Accept conseguirVentas(articulo: IN integer) do
                id:= articulo;
            end conseguirVentas;
            ventas:= ObtenerVentas(id);
            Central.recibirVentas(ventas);
        end loop;
    End Sucursal;
    
Begin
    null;
End VentaDeIndumentaria;
```
