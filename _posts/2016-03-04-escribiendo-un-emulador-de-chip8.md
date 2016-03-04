---
layout: post
title: Escribiendo un emulador de Chip8
category: Emuladores
---

Entre mis inquietudes tecnológicas siempre ha estado saber cómo funciona un emulador. Cuando he usado alguno me he quedado maravillado al ver cómo reproducen juegos o programas a la perfección (o casi).

Así que para aprender realmente cuál es su funcionamiento (a un nivel muy básico) decidí informarme sobre cómo debería empezar. Muchos recomendaban empezar escribiendo un emulador de **Chip8**.

Podéis ver y usar como queráis el código fuente de mi emulador, disponible en su [repositorio de Github](https://github.com/ismaro3/Java-chip8-emulator). Cuenta con gran cantidad de comentarios y tests, por lo que es muy didáctico :)

# ¿Qué es Chip8?

Vale, para empezar, Chip8 no es un sistema hardware, es un "lenguaje de programación" interpretado desarrollado para computadores de 8 bits hace 40 años que pretendía facilitar la creación de videojuegos.
Más que un lenguaje de programación es una definición básica de un procesador ficticio: sus registros, su entrada/salida, y su conjunto de instrucciones.
Entre sus características está una pantalla de 64x32 píxeles, 16 registros de propósito general, 4KB de memoria y 16 botones de entrada.

![Ejemplo de juego](/images/emulador-chip8/screenshot.png)

Como vemos no es que sea la leche ni mucho menos. Además de una resolución muy baja, **sólo tiene dos colores**: blanco y negro.
El "procesador" tampoco se queda corto, pues los juegos se ejecutan de **60 a 500 Hz**. Si, Hz, no KHz ni MHz ni GHz. Es por su sencillez por la que es un buen sistema para empezar en este mundo de la emulación.

# Recursos para su programación

Existen una serie de recursos que pueden considerarse la biblia de "Chip-8", en los que se dispone de toda la información para programar un emulador.

* [Cowgod's Chip-8 Technical Reference v1.0](http://devernay.free.fr/hacks/chip8/C8TECH10.HTM) : Si sólo pudiese usar un documento, sin duda sería este. Lo primero que hice fue imprimirlo, y sólo con él programé el 95% de mi emulador. Contiene la descripción de absolutamente todo el sistema, las instrucciones detalladas una por una,... Esencial.
* [Mastering Chip-8](http://devernay.free.fr/hacks/chip8/C8TECH10.HTM) : Este documento complementa al anterior, ya que explica más en detalle algunos aspectos que pueden no quedar del todo claros con el anterior.

# Pasos a seguir

* **Emulación de la memoria:** En primer lugar lo ideal es construir una estructura de datos que pueda representar la memoria y su acceso. Bastará con un array de bytes o similar.

* **Emulación de los registros:** Simular los registros es muy sencillo. Hay registros de 8 y de 16 bits (1 o 2 bytes), por lo que basta crear algunas variables de tipo byte o short según corresponda.

* **Emulación de la pila:** En este sistema, la implementación de la pila se deja libre al usuario, pues no va en la zona de memoria normal. Basta con implementar otro array parecido a la memoria y un registro SP (Stack-Pointer).

* **Emulación de las instrucciones de la CPU:** La parte más costosa no en cuanto a dificultad, sino en cuanto a tiempo. Basta con leer la documentación e ir creando una función por instrucción que realice exactamente lo que dice la función. Una instrucción suele operar con uno o más datos (incrustados en los 2 Bytes que representan la instrucción) y producir alteraciones en los registros del procesador. Mucho ojo si usais Java con los tipos byte, porque no existe el tipo *unsigned byte*. Como consecuencia, para operar correctamente con datos de tipo *byte* tendréis que ir andando aplicando máscaras para eliminar posibles bits añadidos como signo por parte de Java.

* **Emulación de la pantalla:** La pantalla la represento como un array de booleanos, en la que cada píxel puede estar a true (blanco) o false (negro). La operación "draw", que es la única que escribe en la pantalla, modifica dicho array. Cada 1/60 segundos se dibuja el contenido del array en una ventana JPanel de Swing. (Para simular una tasa de refresco de 60Hz).

* **Ciclo principal de emulación:** El bucle principal del emulador debe simular, en cada iteración, un ciclo del procesador. Es similar a lo siguiente (faltan detalles):
{% highlight java %}
while(true) {

    fetch();            //Obtener siguiente instrucción de memoria

    incrementPC();      //Incrementar el contador de programa

    decodeAndExecute(); //Decodificar y ejecutar la instrucción

    //Si han pasado 1/60s toca refrescar la pantalla y actualizar
    //registros de timer (ST y DT).

    if(refreshTime){

        //Si está activo el flag de dibujar, dibujamos y lo desactivamos
        if(memoryDrawFlag){
            drawScreen();
            memoryDrawFlag = false;
        }

        updateDT(); //Actualizar timer DT

        updateST(); //Actualizar timer ST


    }

    //Esperamos a que pase el tiempo correspondiente a un ciclo
    waitForCycleCompletion();


}
{% endhighlight %}

* **Temporización:** Algo importante es hacer que el sistema no vaya demasiado rápido, sino que vaya a la velocidad en la que iría en el sistema normal. Para ello está el método *waitForCycleCompletion()* del bucle anterior. Se encarga de esperar el tiempo necesario para que un ciclo del bucle tarde lo mismo que un ciclo de CPU en el sistema original. Si la frecuencia del procesador original es de 500Hz, cada ciclo debe durar 1/500 (s). Por lo tanto, dicho método se encargará de esperar hasta que ese tiempo se cumpla.

* **Entrada:** Para manejar la entrada se han establecido varios listeners del teclado en Swing para que, al pulsar una tecla, actualice el valor de un ítem en un array. Dicho array representa el estado de las teclas, y es consultado por las instrucciones encargadas de verificar si una tecla está pulsada o no.

* **Sonido:** He usado la clase JavaX sound para sintetizar los sonidos, aunque también se podría usar ficheros .wav.


Sé que la descripción de estos pasos es muy vaga, pero si queréis más detalles, mirad el [código fuente](https://github.com/ismaro3/Java-chip8-emulator) de mi emulador que está muy comentado y cuenta con muchísimos tests automatizados.

# Futuro
El siguiente paso lógico es ampliar la emulación de Chip8 a [SuperChip](http://devernay.free.fr/hacks/chip8/schip.txt), una versión mejorada que incluye mayor resolución, velocidad e instrucciones. En teoría no debería costar mucho, pero no tengo demasiado tiempo. Si lo hago lo pondré por aquí.

Algo que también me gustaría sería escribir un emulador de Game Boy, ya que según he leído es el siguiente paso lógico. No obstante pintá bastante más complicado, pues entran interrupciones, operaciones con distinta duración y otras dificultades.

Si queréis adentraros en el mundo de la emulación, aunque sea brevemente (para decir que habéis escrito un emulador), esta es la manera. Es más, no sé cómo no se propone como trabajo en asignaturas de Hardware de la carrera de Informática, pues es muy didáctico sobre el funcionamiento de un procesador.

Un saludo, Ismael.
