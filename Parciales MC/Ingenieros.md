# Ejercicios de la práctica de ingeniería
## Ejercicio 1
Se tiene un curso con 40 alumnos, la maestra entrega una tarea distinta a cada alumno, luego
cada alumno realiza su tarea y se la entrega a la maestra para que la corrija, esta revisa la tarea
y si está bien le avisa al alumno que puede irse, si la tarea está mal le indica los errores, el
alumno corregirá esos errores y volverá a entregarle la tarea a la maestra para que realice la
corrección nuevamente, esto se repite hasta que la tarea no tenga errores.
```cpp
Process Alumno[id:1..40]{
    P(mutex)
    cola.push(id)
    V(mutex)
    V(pedirTarea)
    P(esperartarea[id])
    correccion = false
    while(correccion == false){
        tarea = tareas[id].resolver()
        P(mutex2)
        resueltas.push(tarea, id)
        V(mutex2)
        V(pedirCorreccion)
        P(esperarCorrecion[id])
        correcion = correcciones[id]
    }
}
Process Maestra{
    for(i=0,i<40,i++){
        P(pedirTarea)
        P(mutex)
        id = cola.pop()
        V(mutex)
        tareas[id] = tarea.generar()
        V(esperarTarea[id])
    }
    while(true){
        P(pedirCorreccion)
        r, id = resuletas.pop()
        correccion = r.corregir()
        correcciones[id] = correccion
        V(esperarCorrecion[id])
    }
}
```