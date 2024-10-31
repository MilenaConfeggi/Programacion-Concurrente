# MD del 08-11-22
## PMA
1. Resolver con PMA el siguiente problema. Se debe modelar el funcionamiento de una casa de venta de repuestos 
automotores, en la que trabajan V vendedores y que debe atender a C clientes. El modelado debe considerar que: 
(a) cada cliente realiza un pedido y luego espera a que se lo entreguen; y (b) los pedidos que hacen los clientes son 
tomados por cualquiera de los vendedores. Cuando no hay pedidos para atender, los vendedores aprovechan para 
controlar el stock de los repuestos (tardan entre 2 y 4 minutos para hacer esto). Nota: maximizar la concurrencia.
```cpp
chan pedidos(int)
chan libre(int)
chan entrega[C](text)
chan siguiente[V](int)

Process Cliente[id:1..C]{
    send pedidos(id)
    receive entrega[id](rta)
}

Process Coordinador{
    while true(){
        receive libre(idV)
        if(pedidos.empty())
            idP = -1
        else
            receive pedidos(idP)
        send siguiente[idV](idP)
    }
}

Process Vendedor[id:1..V]{
    while true{
        send libre(id)
        receive siguiente[id](idC)
        if(respuesta != -1)
            rta = atender(idC)
            send entrega[idC](rta)
        else
            controlarStock()
    }
}
```

## ADA
2. Resolver  con  ADA  el  siguiente  problema.  Se  quiere  modelar  el  funcionamiento  de  un  banco,  al  cual  llegan 
clientes  que  deben  realizar  un  pago  y  llevarse  su  comprobante.  Los  clientes  se  dividen  entre  los  regulares  y  los 
premium,  habiendo  R  clientes  regulares  y  P  clientes  premium.  Existe  un  único  empleado  en  el  banco,  el  cual 
atiende  de  acuerdo  al  orden  de  llegada,  pero  dando  prioridad  a  los  premium  sobre  los  regulares.  Si  a  los  30 
minutos de llegar un cliente regular no fue atendido, entonces se retira sin realizar el pago. Los clientes premium 
siempre esperan hasta ser atendidos.
```java
Procedure banco is
TASK Empleado is
    ENTRY altaPrioridad(comprobante: OUT int)
    ENTRY bajaPrioridad(comprobante: OUT int)
end empleado;
TASK BODY Empleado is
comprobante: int
Begin
    loop
        SELECT
            ACCEPT altaPrioridad(comprobante: OUT int) do
                comprobante:= generarComrpobante()
            end altaPrioridad;
        OR
            when(AltaPrioridad.count = 0) -> ACCEPT bajaPrioridad(comprobante: OUT int) do
                comprobante:= generarComrpobante()
            end bajaPrioridad;
        end select;
    end loop;
end empleado;

TASK TYPE Premium 
premiums: array(1..P) of Premium

TASK BODY Premium is
Begin
    empleado.altaPrioridad(comprobante)
end premium;

TASK TYPE Regular
regulares: array(1..P) of Regular

TASK BODY Regular is
Begin
    SELECT
        empleado.bajaPrioridad(comprobante)
    OR DELAY(1800)
        null
end premium;

Begin
    null
end banco;
```


# Programación Concurrente ATIC – Redictado de Programación Concurrente 
##  Segunda Fecha - 01/07/2021 

### PMA
2. Resolver  el  siguiente  problema  con  Pasaje  de  Mensajes  Asincrónicos  (PMA).  En  una  empresa  de
software  hay  3  prog ra madores  que  deben  arreglar  errores  informados  por  N  clientes.  Los  clientes
continuamente están trabajando, y cuando encuentran un error envían un reporte a la empresa para que lo
corrija (no tienen que esperar a que se resuelva). Los programadores resuelven los reclamos de acuerdo al
orden de llegada, y si no hay reclamos pendientes trabajan durante una hora en otros programas. Nota: los
procesos  no  deben  terminar  (trabajan  en  un  loop  infinito);  suponga  que  hay  una  función ResolverError  que
simula que un programador está resolviendo un reporte de un cliente, y otra Programar que simula que está
trabajando en otro programa
```cpp
chan libre(int)
chan pedido(text)
chan reclamo[3](text)

Process Programador[id:1..3]{
    while(true){
        send libre(id)
        receive reclamo[id](reporte)
        if(reporte != "vacio")
            resolverError(reporte)
        else
            programar()
    }
}

Process Coordinador{
    while(true){
        receive libre(idP)
        if(pedido.empty())
            reporte = "vacio"
        else
            receive pedido(reporte)
        send reclamo[idP](reporte)
    }
}

Process Cliente[id:1..3]{
    while(true)
        send pedido(reporte)
}
```

### ADA
3. Resolver  el  siguiente  problema  con  ADA.  Hay  un  sitio  web  para  identificación  genética  que  resuelve
pedidos  de  N  clientes.  Cada  cliente  trabaja  continuamente  de  la  siguiente  manera:  genera  la  secuencia  de
ADN, la envía al sitio web para evaluar y espera el resultado; después de esto puede comenzar a generar la
siguiente secuencia de ADN. Para resolver estos pedidos el sitio web cuenta con 5 servidores idénticos que
atienden los pedidos de acuerdo al orden de llegada (cada pedido es atendido por un único servidor). Nota: 
maximizar  la  concurrencia.  Suponga  que  los  servidores  tienen  una  función  ResolverAnálisis  que  recibe  la
secuencia de ADN y devuelve un entero con el resultado
```java
Procedure sitioWeb is
TASK TYPE Servidor
servidores: array(1..5) of Servidor
TASK BODY Servidor is
    resultado: int
Begin
    loop
        coordinador.pedido(p, idC) 
        resultado:= ResolverAnálisis(p)
        clientes[idC].recibirResultado(resultado)
    end loop;
end servidor;

TASK TYPE Cliente is
    ENTRY recibirResultado(resultado: IN text)
    ENTRY recibirId(id: IN int)
end cliente;
clientes: array(1..N) of Cliente
TASK BODY Cliente is
id: int
Begin
    ACCEPT recibirId(id: IN int) do
        id:= id
    end recibirId;
    loop
        adn: generaradn()
        coordinador.recibirADN(adn, id)
        ACCEPT recibirResultado(resultado: IN text)
    end loop;
end cliente;

TASK Coordinador is
    ENTRY recibirADN(adn: IN text; id: IN int)
    ENTRY pedido(p: OUT text; idC: OUT int)
end coordinador;
TASK BODY Coordinador is
cola
Begin
    loop
        SELECT
            when cola.lenght not 0 -> ACCEPT pedido(p: OUT text; idC: OUT int) do
                p, idC:= cola.pop()
            end pedido;
        OR 
            ACCEPT recibirADN(adn: IN text; id: IN int) do
                cola.push(adn, id)
            end recibiradn;
        end select;
    end loop;

end coordiandor;

Begin
    FOR i in 1..N loop
        clientes[i].recibirId(i)
    end loop;
end sitioWeb;
```

# Programación Concurrente ATIC – Redictado de Programación Concurrente 
## Tercera Fecha - 15/07/2021

### PMS
2. Resolver el siguiente problema con Pasaje de Mensajes Sincrónicos (PMS). En una excursión hay una 
tirolesa que debe ser usada por 20 turistas. Para esto hay un g uía y un empleado. El empelado espera a 
que  todos  los  turistas  hayan  llegado  para  darles  una  charla explicando  las  medidas  de  seguridad.  Cuando  
termina  la  charla  los  turistas  piden  usar  la  tirolesa  y  esperan  a  que  el  guía  les  vaya  dando  el  permiso  de  
tirarse.  El  guía  deja  usar  la  tirolesa  a  un  cliente a  la  vez y  de  acuerdo  al  orden  en  que  lo  van  solicitando. 
Nota: todos los procesos deben terminar;  suponga que el empleado tienen una función DarCharla() que 
simula que el empleado está dando la charla, y los turistas tienen una función UsarTitolesa() que simula que 
está usando la tirolesa. 
```cpp
Process Turista[id:1..20]{
    empleado!llegar()
    empleado?terminoCharla()
    coordinador!solicitarTirolesa(id)
    guia?turno()
    //usar
    guia!liberar()
}

Process Coordinador{
    int cant = 0
    do cant< 20; turista[*]?solicitarTirolesa(idT) -> cola.push(idT)        
    [] !cola.empty(); guia?libre() -> guia!siguiente(cola.pop())
}

Process Empleado{
    for(i=1; i<=20; i++)
        turista[*]?llegar()
    for(i=1; i<=20; i++)
        turista[i]!terminoCharla()
}

Process Guia{
    for(i=1; i<=20; i++)
        coordinador!libre()
        coordinador?siguiente(idT)
        Turista[idT]!turno()
        turista[idT].liberar()
}
```

### ADA
3. Resolver el siguiente problema con ADA. En un negocio hay un empleado para atender los pedidos de 
clientes  de  N  clientes;  algunos  clientes  son  ancianos  o  embarazadas.  El  empleado  atiende  los  pedidos  de  
acuerdo a la siguiente prioridad: primero las embarazadas, luego los ancianos, y por último el resto. Cada 
cliente  hace  sólo  un  pedido  de  la  siguiente  manera:  en  el  caso  de  las  embarazadas,  si  no  son  atendidas  
inmediatamente se retiran; en el caso de los ancianos, esperan a los sumo 5 minutos a ser atendidos, y si 
no se retira; cualquier otro cliente espera si o si hasta ser atendido. El empleado atiende a los clientes de 
acuerdo al orden de llegada pero manteniendo las siguientes prioridades: primero las embarazadas, luego 
los  ancianos  y  luego  el  resto.  Nota:  suponga  que  existe  una  función  AtenderPedido()  que  simula  que  el  
empleado está atendiendo a un cliente.
```java
Procedure Negocio is
TASK Empleado is
    ENTRY altaPrioridad()
    ENTRY bajaPrioridad()
    ENTRY mediaPrioridad()
end empleado;
TASK BODY Empleado is
Begin
    loop
        SELECT
            ACCEPT altaPrioridad() do
                AtenderPedido()
            end altaPrioridad;
        OR
            when (altaPrioridad().count = 0) -> ACCEPT mediaPrioridad() do
                AtenderPedido()
            end mediPrioridad;
        OR
            when (altaPrioridad().count = 0 && mediaprioridad().count = 0) -> ACCEPT bajaPrioridad() do
                AtenderPedido()
            end bajaPrioridad;
        end select;
    end loop;
end empleado;

TASK TYPE Cliente 
clientes: array(1..N) of CLiente
TASK BODY Cliente is
tipo: text
Begin
    tipo:= obtenerTipo()
    if(tipo == "embarazada")
        SELECT
            empleado.altaPrioridad()
        ELSE
            null
        end select;
    else
        if(tipo == "anciano")
            SELECT
                empleado.mediaPrioridad()
            OR DELAY 300
                null
            end select;
        else
            empleado.bajaPrioridad()
end cliente;

Begin
    null
end negocio;
```

#  parcial práctico del 13-12-22

### PMS
1. Resolver con Pasaje de Mensajes Sincrónicos (PMS) el siguiente problema. En un comedor estudiantil hay un horno microondas 
que debe ser usado por E estudiantes de acuerdo con el orden de llegada. Cuando el estudiante accede al horno, lo usa y luego se 
retira para dejar al siguiente. Nota: cada Estudiante una sólo una vez el horno.
```cpp
Process Estudiante[id:1..E]{
    coordinador!llegar(id)
    coordinador?turno()
    //usar
    coordinador!liberar()
}

Process Coordinador{
    do estudiante[*]?llegar(idE) -> 
        if libre
            libre = false
            estuidante[idE]!turno()
        else
            cola.push(idE)
    [] estudiante[*]?liberar; ->
        if cola.empty()
            libre = true
        else
           estuidante[cola.pop()]!turno() 
}
```

### ADA
2. Resolver  con  ADA  el  siguiente  problema.  Se  debe  controlar  el  acceso  a  una  base  de  datos.  Existen  L  procesos  Lectores  y  E 
procesos Escritores que trabajan indefinidamente de la siguiente manera: 
• Escritor:  intenta  acceder  para  escribir,  si  no  lo  logra  inmediatamente,  espera  1  minuto  y  vuelve  a  intentarlo  de  la  misma 
manera. 
• Lector: intenta acceder para leer, si no lo logro en 2 minutos, espera 5 minutos y vuelve a intentarlo de la misma manera. 
Un proceso Escritor podrá acceder si no hay ningún otro proceso  usando la base de datos; al acceder escribe y sale de la BD. Un 
proceso Lector podrá acceder si no hay procesos Escritores usando la base de datos; al acceder lee y sale de la BD. Siempre se le 
debe dar prioridad al pedido de acceso para escribir sobre el pedido de acceso para leer. 
```java
Procedure BD is
TASK TYPE Lector 
lectores: array(1..L) OF Lector
TASK BODY Lector is

Begin
    loop
        SELECT 
            controlador.accesoLector()
            //leer
            controlador.salirLector()
        OR DELAY 120
            DELAY 300
    end loop;
end lector;

TASK TYPE Escritor
escritores: array(1..E) OF Escritor
TASK BODY Escritor is

Begin
    loop
        SELECT 
            controlador.accesoEscritor()
            //escribir
            controlador.salirEscritor()
        OR DELAY 60
            null
    end loop;
end escritor;

TASK Controlador is
    ENTRY accesoLector()
    ENTRY accesoEscritor()
    ENTRY salirLector()
    ENTRY salirEscritor()
end controlador;

TASK BODY Controlador is
int cantLectores = 0
bool hayEscritor = False
Begin
    loop
        SELECT 
            ACCEPT salirLector() 
            cantLectores--           
        OR
            ACCEPT salirEscritor() 
            hayEscritor = false
            end salirLector;
        OR
            when(hayEscritor == false; accesoEscritor.count == 0) -> ACCEPT accesoLector() 
            cantLectores++
        OR
            when(hayEscritor == false && cantLectores == 0) -> ACCEPT accesoEscritor()
            hayEscritor == True
        end select;
    end loop;
end controlador;

Begin
    null
end BD;

```
 
# primer recuperatorio del parcial práctico

### PMS
1. Resolver con Pasaje de Mensajes Sincrónicos (PMS) el siguiente problema. En un torneo de programación 
hay 1 organizador, N competidores y S supervisores. El organizador comunica el desafío a resolver a cada 
competidor. Cuando un competidor cuenta con el desafío a resolver, lo hace y lo entrega para ser evaluado. 
A continuación, espera a que alguno de los supervisores lo corrija y le indique si está bien. En caso de tener 
errores, el competidor debe corregirlo y volver a entregar, repitiendo la misma metodología hasta que llegue 
a la solución esperada. Los supervisores corrigen las entregas respetando el orden en que los competidores 
van entregando. Nota: maximizar la concurrencia y no generar demora innecesaria.
```cpp
Process Competidor[id:1..N]{
    bool bien = false
    organizador!llegar(id)
    oganizador?arrancar()
    while(!bien)
        coordinador!entregar(idC, resolucion)
        supervisor[*]?correccion(bien)
}

Process Organizador{
    for(i=0; i<N; i++)
        competidor[*]?llegar(idC)
        competidor[idC]!arrancar()
}

Process Coordinador{
    do competidor[*]?entregar(idC, resolucion) -> cola.push(idC, resolucion)
    [] !cola.empty(); supervisor[*]?libre(idS); -> supervisor[idS]!atender(cola.pop())
}

Process Supervisor[id:1..S]{
    while(true){
        coordinador!libre(id)
        coordinador?atender(idC, resolucion)
        bien = corregir(resolucion)
        competidor[idC]!correccion(bien)
    }
}
```


