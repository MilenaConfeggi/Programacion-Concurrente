# Practica 5
La sintaxis no es la mejor!!!
## Ejercicio 1
1. Se requiere modelar un puente de un único sentido que soporta hasta 5 unidades de peso.
El peso de los vehículos depende del tipo: cada auto pesa 1 unidad, cada camioneta pesa 2
unidades y cada camión 3 unidades. Suponga que hay una cantidad innumerable de
vehículos (A autos, B camionetas y C camiones). Analice el problema y defina qué tareas,
recursos y sincronizaciones serán necesarios/convenientes para resolverlo.
a. Realice la solución suponiendo que todos los vehículos tienen la misma prioridad.
b. Modifique la solución para que tengan mayor prioridad los camiones que el resto de los
vehículos.
a)
```java
Procedure Puente is
Task Trabajador is
    Entry pedidoAuto();
    Entry pedidoCamioneta();
    Entry pedidoCamiones();
    Entry salir(Peso: IN Integer)
end Trabajador;
Task Type Auto();
Autos: array (1..A) of Auto;
Task body Auto is
    trabajador.pedidoAuto()
    //pasar
    trabajador.salir(1)
end Auto;

Task Type Camioneta();
Camionetas: array (1..B) of Camioneta;
Task body Camioneta is
    trabajador.pedidoCamioneta()
    //pasar
    trabajador.salir(2)
end Camioneta;

Task Type Camion();
Camiones: array (1..C) of Camion;
Task body Camion
    trabajador.pedidoCamion()
    //pasar
    trabajador.salir(3)
end Camioneta;

Task body Trabajador
    int cant = 5
    Begin
        LOOP
            SELECT
                when(cant-1 => 0 AND Salir.count=0) -> ACCEPT pedidoAuto() do 
                cant--;
                end;
            OR
                when (cant-2=>0 AND Salir.count=0) -> ACCEPT pedidoCamioneta() do 
                cant=cant-2;
                end;
            OR
                when (cant-3=>0 AND Salir.count=0) -> ACCEPT pedidoCamion() do 
                cant=cant-3;
                end;
            OR
                accept Salir(IN peso) do
                    cant=cant+peso;
                end;
            end select;
        end loop;
    end;
end Trabajador;

Begin
    null
end Puente;
```
b)
```java
Procedure Puente is
Task Trabajador is
    Entry pedidoAuto();
    Entry pedidoCamioneta();
    Entry pedidoCamiones();
    Entry salir(Peso: IN Integer)
end Trabajador;
Task Type Auto();
Autos: array (1..A) of Auto;
Task body Auto is
    trabajador.pedidoAuto()
    //pasar
    trabajador.salir(1)
end Auto;

Task Type Camioneta();
Camionetas: array (1..B) of Camioneta;
Task body Camioneta is
    trabajador.pedidoCamioneta()
    //pasar
    trabajador.salir(2)
end Camioneta;

Task Type Camion();
Camiones: array (1..C) of Camion;
Task body Camion
    trabajador.pedidoCamion()
    //pasar
    trabajador.salir(3)
end Camioneta;

Task body Trabajador
    int cant = 5
    Begin
        LOOP
            SELECT
                when(cant-1 => 0 AND Salir.count=0 AND pedidoCamion.count = 0) -> ACCEPT pedidoAuto() do 
                cant--;
                end;
            OR
                when (cant-2=>0 AND Salir.count=0  AND pedidoCamion.count = 0) -> ACCEPT pedidoCamioneta() do 
                cant=cant-2;
                end;
            OR
                when (cant-3=>0 AND Salir.count=0) -> ACCEPT pedidoCamion() do 
                cant=cant-3;
                end;
            OR
                accept Salir(IN peso) do
                    cant=cant+peso;
                end;
            end select;
        end loop;
    end;
end Trabajador;

Begin
    null
end Puente;
```

## Ejercicio 2
Se quiere modelar el funcionamiento de un banco, al cual llegan clientes que deben realizar
un pago y retirar un comprobante. Existe un único empleado en el banco, el cual atiende de
acuerdo con el orden de llegada.
a) Implemente una solución donde los clientes llegan y se retiran sólo después de haber sido
atendidos.
b) Implemente una solución donde los clientes se retiran si esperan más de 10 minutos para
realizar el pago.
c) Implemente una solución donde los clientes se retiran si no son atendidos
inmediatamente.
d) Implemente una solución donde los clientes esperan a lo sumo 10 minutos para ser
atendidos. Si pasado ese lapso no fueron atendidos, entonces solicitan atención una vez más
y se retiran si no son atendidos inmediatamente.

a)
```java
Procedure Banco is
Task Empleado is
    Entry pedido()
end Empleado;

Task type Cliente
clientes: array(1..N) of Cliente
Task body Cliente is
    comprobante: texto
Begin
    empleado.pedido("datos", comprobante)
end Cliente;

Task body Empleado is
Begin
    loop
        ACCEPT pedido(D: IN texto, C: OUT texto) do
            C= resolverpedido()
        end pedido;
    end loop;
end Empleado;
Begin
    null
end Banco;
```

b)
```java
Procedure Banco is
Task Empleado is
    Entry pedido()
end Empleado;

Task type Cliente
clientes: array(1..N) of Cliente
Task body Cliente is
    comprobante: texto
Begin
    SELECT
        empleado.pedido("datos", comprobante)
    OR DELAY 600
        null
    end select;
end Cliente;

Task body Empleado is
Begin
    loop
        ACCEPT pedido(D: IN texto, C: OUT texto) do
            C= resolverpedido()
        end pedido;
    end loop;
end Empleado;
Begin
    null
end Banco;
```
c)
```java
Procedure Banco is
Task Empleado is
    Entry pedido()
end Empleado;

Task type Cliente
clientes: array(1..N) of Cliente
Task body Cliente is
    comprobante: texto
Begin
    SELECT
        empleado.pedido("datos", comprobante)
    ELSE 
        null
    end select;
end Cliente;

Task body Empleado is
Begin
    loop
        ACCEPT pedido(D: IN texto, C: OUT texto) do
            C= resolverpedido()
        end pedido;
    end loop;
end Empleado;
Begin
    null
end Banco;
```

d)
```java
Procedure Banco is
Task Empleado is
    Entry pedido()
end Empleado;

Task type Cliente
clientes: array(1..N) of Cliente
Task body Cliente is
    comprobante: texto
Begin
    SELECT
        empleado.pedido("datos", comprobante)
    OR DELAY 600
        SELECT
            empleado.pedido("datos", comprobante)
        ELSE
            null
    end select;
end Cliente;

Task body Empleado is
Begin
    loop
        ACCEPT pedido(D: IN texto, C: OUT texto) do
            C= resolverpedido()
        end pedido;
    end loop;
end Empleado;
Begin
    null
end Banco;
```
## Ejercicio 3
Se dispone de un sistema compuesto por 1 central y 2 procesos periféricos, que se
comunican continuamente. Se requiere modelar su funcionamiento considerando las
siguientes condiciones:
- La central siempre comienza su ejecución tomando una señal del proceso 1; luego
toma aleatoriamente señales de cualquiera de los dos indefinidamente. Al recibir una
señal de proceso 2, recibe señales del mismo proceso durante 3 minutos.
- Los procesos periféricos envían señales continuamente a la central. La señal del
proceso 1 será considerada vieja (se deshecha) si en 2 minutos no fue recibida. Si la
señal del proceso 2 no puede ser recibida inmediatamente, entonces espera 1 minuto y
vuelve a mandarla (no se deshecha).
```java
Procedure sistema is
Task central is
    Entry senialuno()
    Entry senialdos()
    Entry fin()
end central;

Task timer is
    Entry iniciar()
end timer;
Task body timer is
Begin
    ACCEPT iniciar()
    delay 60
    central.fin()
end timer;

Task type procesouno;
Task body procesouno is
Begin
    loop
        SELECT
            central.senialuno()
        OR DELAY 120
            null
        end select;
    end loop;
end procesouno;

Task type procesodos;
Task body procesodos is
Begin
    loop
        SELECT
            central.senialdos()
        ELSE
            DELAY 60
            central.senialdos()
    end loop;
end procesodos;

Task body central is
int seguir;
Begin
    ACCEPT senialuno() 
    loop
        SELECT
            ACCEPT senialuno()
        OR
            ACCEPT senialdos()
            seguir = true
            timer.iniciar()
            while(seguir)
                SELECT
                    ACCEPT fin() do seguir = false
                OR
                    when fin.count = 0 -> ACCEPT senialdos()
                end select;
            end while;
        end select;
    end loop;
end central;
Begin
    null
end sistema;
```

## Ejercicio 4
En una clínica existe un médico de guardia que recibe continuamente peticiones de
atención de las E enfermeras que trabajan en su piso y de las P personas que llegan a la
clínica ser atendidos.
Cuando una persona necesita que la atiendan espera a lo sumo 5 minutos a que el médico lo
haga, si pasado ese tiempo no lo hace, espera 10 minutos y vuelve a requerir la atención del
médico. Si no es atendida tres veces, se enoja y se retira de la clínica.
Cuando una enfermera requiere la atención del médico, si este no lo atiende inmediatamente
le hace una nota y se la deja en el consultorio para que esta resuelva su pedido en el
momento que pueda (el pedido puede ser que el médico le firme algún papel). Cuando la
petición ha sido recibida por el médico o la nota ha sido dejada en el escritorio, continúa
trabajando y haciendo más peticiones.
El médico atiende los pedidos dándole prioridad a los enfermos que llegan para ser atendidos.
Cuando atiende un pedido, recibe la solicitud y la procesa durante un cierto tiempo. Cuando
está libre aprovecha a procesar las notas dejadas por las enfermeras.
```java
Procedure clinica is
Task medico is
    Entry enfermos()
    Entry pedidoEnfermera()
    Entry notaEnfermera(nota: IN texto)
end medico;
Task bodymedico is
Begin
    loop
        SELECT 
            ACCEPT enfermos()
        OR
            when(enfermos.count = 0) -> ACCEPT pedidoEnfermera()
        ELSE
            ACCEPT notaEnfermera(nota) do
                procesar(nota)
            end;
        end select;
    end loop;
end medico;

Task type enfermera
Task body enfermera is
Begin
    loop
        SELECT
            medico.pedidoEnfermera()
        ELSE
            escritorio.dejarnota(nota)
        end select;
    end loop;
end enfermera;

Task escritorio is
    Entry dejarnota(nota: IN texto)
end escritorio;
Task body escritorio is
Cola cola
Begin
    loop
        SELECT
            when(!cola.empty()) -> medico.notaEnfermera(cola.pop())
        OR
            ACCEPT dejarnota(nota) do
                cola.push(nota)
            end;
    end loop;
end escritorio;

Task type persona 
task body persona is
int intentos
Begin
    SELECT
        medico.enfermos()
    OR DELAY 300
        DELAY 600
        intentos = 0
        while(intentos <= 3)
            SELECT
                medico.enfermos()
            ELSE
                when(intentos<=2) -> DELAY 600; intentos++ //hago esto para que no espere 10min antes de irse
            end select;
        end while;
    end select;
end persona;

Begin
    null
end clinica;
```

## Ejercicio fantasma
En un sistema para acreditar carreras universitarias, hay UN Servidor que atiende pedidos
de U Usuarios de a uno a la vez y de acuerdo con el orden en que se hacen los pedidos.
Cada usuario trabaja en el documento a presentar, y luego lo envía al servidor; espera la
respuesta de este que le indica si está todo bien o hay algún error. Mientras haya algún error,
vuelve a trabajar con el documento y a enviarlo al servidor. Cuando el servidor le responde
que está todo bien, el usuario se retira. Cuando un usuario envía un pedido espera a lo sumo
2 minutos a que sea recibido por el servidor, pasado ese tiempo espera un minuto y vuelve a
intentarlo (usando el mismo documento).

```java
Procedure sistema is

Task type Usuario 
usuarios: array (1..A) of Usuario;
Task body usuario is
texto documento
bool rta
Begin
    rta = false
    documento = realizardocumento()
    while(rta == false)
        SELECT
            servidor.enviardoc(documento, rta)
            if(rta == false)
                documento = documento.arreglar()
            end if;
        OR DELAY 120
            delay 60
        end select;
    end while;
end usuario;

Task servidor is
    ENTRY enviardoc(documento: IN texto; rta: OUT bool)
end servidor;
Task body servidor is
    loop
        ACCEPT enviardoc(documento, rta) do
            rta = realizarrta(documento)
        end;
    end loop,
end servidor;
Begin
    null
end sistema;
```

## Ejercicio 5
En una playa hay 5 equipos de 4 personas cada uno (en total son 20 personas donde cada
una conoce previamente a que equipo pertenece). Cuando las personas van llegando
esperan con los de su equipo hasta que el mismo esté completo (hayan llegado los 4
integrantes), a partir de ese momento el equipo comienza a jugar. El juego consiste en que
cada integrante del grupo junta 15 monedas de a una en una playa (las monedas pueden ser
de 1, 2 o 5 pesos) y se suman los montos de las 60 monedas conseguidas en el grupo. Al
finalizar cada persona debe conocer el grupo que más dinero junto. Nota: maximizar la
concurrencia. Suponga que para simular la búsqueda de una moneda por parte de una
persona existe una función Moneda() que retorna el valor de la moneda encontrada.

```java
Procedure juego is
Task type Persona is
    ENTRY cantidadTotal(total: IN int)
    ENTRY gruposuperior(idgrupo: IN int)
end persona;
personas: array[1..20] of Persona
Task body Persona is
    int equipo = equipo()
    int monedas = 0
    int T 
Begin
    equipos[equipo].llegada()
    equipos[equipo].salida()
    FOR i IN 1..15 LOOP
        monedas:= juntarmoneda()
    end loop;
    equipos[equipo].cantidadMonedas(monedas)
    ACCEPT cantidadTotal(total) do
        T = total
    ACCEPT gruposuperior(idgrupo)
    end;
end persona;

Task type Equipo is
    ENTRY conseguirid(id: IN int)
    ENTRY llegada()
    ENTRY salida()
    ENTRY cantidadmonedas(monedas: IN int)
end equipo;
equipos: array[1..5] od Equipo
Task body equipo is
    int idE
Begin
    ACCEPT conseguirid(id) do
        idE = id
    end;
    FOR i IN 1..4 LOOP
        ACCEPT llegada()
    end loop;
    FOR i IN 1..4 LOOP
        ACCEPT salida()
    end loop;
    FOR i IN 1..4 LOOP
        ACCEPT cantidadmonedas(monedas) do
            total = total + monedas
        end;
    end loop;
    organizador.totalEquipo(total, idE)
end equipo;

Task Organizador is
    ENTRY totalequipo(monedas: IN int)
end organizador;
Task body organizador is
int maximo = 0
int idmaximo 
Begin
    FOR i IN 1..4 LOOP
        ACCEPT totalEquipo(monedas, idE) do
            if(monedas > maximo)
                maximo = monedas
                idmaximo = idE
            end if;
        end;
    end loop;
    FOR i IN 1..20 LOOP
        personas[i].gruposuperior(idmaximo)
        end;
    end loop;
end organizador;

Begin
    FOR i IN 1..4 LOOP
        equipos[i].conseguirid(i)
    end loop;
end juego;
```

## Ejercicio 6
Se debe calcular el valor promedio de un vector de 1 millón de números enteros que se
encuentra distribuido entre 10 procesos Worker (es decir, cada Worker tiene un vector de
100 mil números). Para ello, existe un Coordinador que determina el momento en que se
debe realizar el cálculo de este promedio y que, además, se queda con el resultado. Nota:
maximizar la concurrencia; este cálculo se hace una sola vez.

```java
Procedure contador is

Task type Worker is
    ENTRY comenzar()
end worker;
workers: array[1..10] of Worker
Task body Worker is
    vector: array (1..100000) of Float;
    res: Integer := 0;
Begin
    ACCEPT comenzar()
    res = sacarpromedio(vector)
    coordinador.resultado(res)
end worker;

Task Coordinador is
   ENTRY resultado(res: IN int) 
end coordinador;
Task body Coordinador is
resultados: array (1..10) of Float;
Begin
    FOR i IN 1..10 LOOP
        workers[i].comenzar()
    end loop;
    FOR i IN 1..10 LOOP
        ACCEPT resultado(res: IN int) do
            resultados[i] = res 
        end;
    end loop;

end coordinador;

Begin
    null
end contador;
```

## Ejercicio 7
Hay un sistema de reconocimiento de huellas dactilares de la policía que tiene 8 Servidores
para realizar el reconocimiento, cada uno de ellos trabajando con una Base de Datos propia;
a su vez hay un Especialista que utiliza indefinidamente. El sistema funciona de la siguiente
manera: el Especialista toma una imagen de una huella (TEST) y se la envía a los servidores
para que cada uno de ellos le devuelva el código y el valor de similitud de la huella que más
se asemeja a TEST en su BD; al final del procesamiento, el especialista debe conocer el
código de la huella con mayor valor de similitud entre las devueltas por los 8 servidores.
Cuando ha terminado de procesar una huella comienza nuevamente todo el ciclo. 
Nota: suponga que existe una función Buscar(test, código, valor) que utiliza cada Servidor donde
recibe como parámetro de entrada la huella test, y devuelve como parámetros de salida el
código y el valor de similitud de la huella más parecida a test en la BD correspondiente.
Maximizar la concurrencia y no generar demora innecesaria.
```java
Procedure Sistema is
Task type Servidor is
    ENTRY enviarInfo(test: IN int)
end servidor;
servidores: array[1..8] of Servidor
Task body Servidor is
    int test
    int cod
    int val
Begin
    ACCEPT enviarInfo(test: IN int) do
        test:= test
    end;
    Buscar(test, codigo, valor)
    cod:= codigo
    val:= valor
    especialista.resultados(cod, val)
end servidor;

Task Especialista is
    ENTRY resultados(codigo: IN int; valor: IN int)
end especialista;
Task body Especialista
    int test
    int valoractual = 0
    int codigoactual
Begin
    loop
        test:= tomarImagen()
        FOR i IN 1..8 LOOP
            servidores[i].enviarInfo(test)
        end loop;
        FOR i IN 1..8 LOOP
            ACCEPT resultados(codigo: IN int; valor: IN int) do
                if(valor > valoractual)
                    valoractual:= valor
                    codigoactual:= codigo
                end if;
            end;
        end loop;
    end loop;
end especialista;

Begin
    null
end sistema;
```

## Ejercicio 8 
Una empresa de limpieza se encarga de recolectar residuos en una ciudad por medio de 3
camiones. Hay P personas que hacen reclamos continuamente hasta que uno de los 
camiones pase por su casa. Cada persona hace un reclamo y espera a lo sumo 15 minutos a
que llegue un camión; si no pasa, vuelve a hacer el reclamo y a esperar a lo sumo 15
minutos a que llegue un camión; y así sucesivamente hasta que el camión llegue y recolecte
los residuos. Sólo cuando un camión llega, es cuando deja de hacer reclamos y se retira.
Cuando un camión está libre la empresa lo envía a la casa de la persona que más reclamos
ha hecho sin ser atendido. Nota: maximizar la concurrencia.
```java
Procedure Ciudad is
Task Empresa is
    ENTRY realizarReclamo(idP: IN int)
    ENTRY atenderReclamo(idP: OUT int)
end empresa;
Task body Empresa is
vec: array[1..P] of int //vector contador de reclamos
int cant = 0 //cantidad de reclamos pendientes
int maximo = 0
int idmax
Begin
    loop
        SELECT
            when(cant > 0) -> ACCEPT atenderReclamo(idP: OUT int) do //camion pide un reclamo
                cant-- //disminuyo la cant de pedidos pendientes
                FOR i IN 1..P LOOP
                    if(vec[i] > maximo) 
                        maximo:= vec[i]
                        idP:= i  //saco el id con mas reclamos                    
                end loop;
                vec[idP]:= -1 //pongo la posicion del vector del id antendido en -1
                maximo = 0 
            end;
        OR
            ACCEPT realizarReclamo(idP: IN int) do //persona hace reclamo
                if(vec[idP] != -1) //si la posicion en el vector es -1 es que ya se le envio al camion el reclamo
                    if(vec[idP] == 0) //si es la primera vez que hace reclamo aumento la cant de pendientes
                        cant++
                    end if;
                    vec[idP]++ //aumento la cant de reclamos
                end;
        end select;
    end loop;
end empresa;


Task type Camion is

end camion;
camiones: array[1..3] of Camion
Task body Camion is

Begin
    loop
        empresa.atenderReclamo(idP: OUT int) //pide atender un reclamo
        personas[idP].esperarCamion() //atiende, puede ser que tenga que esperar ya que la persona salio del select y volvio a hacer el reclamo
    end loop;
end camion;

Task type Persona is
    ENTRY recibirid(id: IN int)
    ENTRY esperarCamion()
end persona;
personas: array[1..P] of Persona
Task body Persona is
    bool atendido = false
    int idP
Begin
    ACCEPT recibirid(id: IN int) do
        idP:= id
    end;
    While(not atendido)
        empresa.realizarReclamo(idP)
        SELECT
            ACCEPT esperarCamion()
            atendido = true
        OR DELAY 900
            null
        end select;
    end loop;
end persona;

Begin
    FOR i IN 1..P LOOP
        personas[i].recibirid(i)
    end loop;
end ciudad;
```
