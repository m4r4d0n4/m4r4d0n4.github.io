# Web Visión Robótica 

Página web de Juan Montes Cano para la asignatura de Visión Robótica del MUVA de la URJC

## Práctica 1 Fórmula 1

### Introducción

El problema a resolver es el control de un Fórmula 1 que contiene una cámara y podrá observar el asfalto. En medio del asfalto hay una línea roja, que nos servirá como apoyo para el desarrollo de un algoritmo reactivo de control visual.

### Planteamiento

Para solucionar el problema nos apoyaremos en controladores PID. Se distinguen dos grados de libertad: el ángulo de giro y la velocidad del vehículo. Y dentro del trazado se distinguen dos zonas claramente diferenciadas, las curvas y las rectas. Por tanto, se diseñaran y ajustarán 4 controladores PIDs, 2 para las velocidades en rectas y curvas y dos para el giro del vehículo en rectas y curvas.

Por otra parte, puesto que se usan controladores PID se tiene que establecer el error a usar, por lo que se tendrá en cuenta la línea roja para obtener una referencia de la pista.

Por último, se diseñará una máquina de estados que nos indicará si estamos en una curva, recta o un estado desconocido.

### Medida de error

Para obtener una referencia que nos sirva para medir el error se toma la línea roja. A partir de aquí, se aplica un procesado de la imagen que incluye un filtro de color HSV, la obtención de la máscara que da los puntos de la línea roja y el cálculo de su contorno con el que calcularemos el centroide. Lo primero que se realiza es obtener un centroide como referencia si el coche estuviese encima de la línea, para ello, se situa el coche en la línea y se toma el centroide de referencia. A partir de aquí, se puede tomar como medida de error la diferencia entre el centroide observado y el centroide objetivo. En este caso, se toma la diferencia respecto de la coordenada x, y no se analizará la parte inferior de la línea roja que será previamente recortada, debido a que la información del centroide de la parte superior de la máscara varía menos y nos proporciona un error más estable.

La primera aproximación al error es ´error = centroide_objetivo_x-centroide_observado_x´, sin embargo, como el centroide objetivo no está exactamente en medio de la pantalla y para obtener un error en [0,1] llevamos a cabo una normalización, por tanto, ´error = (centroide_objetivo_x-centroide_observado_x)/centroide_objetivo_x´

Meter foto centroide

### Controladores PID

Los controladores PID sirven para el diseño de los sistemas de control automático. Cada componente de un controlador PID cumple una función específica:

- P (Proporcional) : Ofrece una salida proporcional al error del sistema y está controlado por la constante `K_p`
- D (Derivativo) : El término derivativo predice la tendencia futura del error, basándose en su tasa de cambio actual. Está controlado por la constante `K_d`
- I (Integral) : El término integral tiene como objetivo eliminar el error acumulado en el tiempo. Está controlado por la constante `K_i`

```math
- k_p \cdot e - K_D \cdot \frac{de}{dt} - k_i \cdot \int e \, dt
```

En este problema no tiene sentido utilizar el controlador integral, puesto que el error acumulado es la suma de las diferencias entre centroides a lo largo del circuito y no es deseable alterar, por ejemplo, el trazado de una curva en un instante por errores ocurridos 3 curvas atrás. (Hablar de q se intento usarlo reseteandolo cada 5s etc.)

Nuestro sistema consta de 4 PIDs:
- Curvas : Un PID para velocidad y un PID para girar el vehículo
- Rectas : Un PID para velocidad y un PID para girar el vehículo

La tarea de ajuste de parámetros de los PIDs se ha llevado a cabo manualmente, lo que implica el ajuste de 8 parámetros. (Hablar de q esta manera no es optima y tal tal tal en alguan seccion de mejroas y eso) (nuestros valores de PIDs)


### Máquina de estados

Se han descrito los PIDs y cómo según la posición en el trazado actúan unos u otros, pero hasta el momento no se tiene manera de distinguir si el vehículo se encuentra en una curva o en una recta. Este proyecto ha mezclado dos formas de obtener si estamos ante una recta o una curva. La primera es el uso de la función `cv2.convexityDefects` para obtener los defectos (es decir, partes donde no hay convexidad) de la convex hull de la línea roja. La línea roja cuando es recta es convexa pero al aparecer una curva surgen defectos. Experimentalmente se han encontrado los números de defectos necesarios para categorizar una curva.

Otra manera con la que aseguramos que estamos ante una curva, ha sido tomar la máscara de la curva y hacer cortes horizontales para quedarnos segmentos de la curva. En estas etapas, se encuentra su punto promedio, de manera que obtenemos al final una línea segmentada que tiene la estructura de la curva. Para saber si estos puntos forman una recta o no, usamos la ecuación de la recta y = mx + b, calcularemos la pendiente y la b con dos puntos y veremos si el resto de puntos está cerca o no de la recta calculada. Se establece una tolerancia hallada experimentalmente que nos indicará si el vehículo está en una curva o un recta.

Fotos de las movidas

Con la detección de curvas hecha, la máquina de estados resultante es muy simple, si se está en una curva se usan los PIDs de curvas, si se está en recta se usarán los PIDs de recta y si se está en un estado desconocido se rotará el coche hasta encontrar la línea roja.


### Resultados

Video 

### Alternativas intentadas



### Mejoras posibles














### 18/02

Primeras pruebas filtrando la imagen y obteniendo el color rojo, de donde sacamos el contorno de la franja y obtenemos su centroide. Lo usaremos para controlar cuanto hay que rotar el coche. La primera aproximación que se da es ajustar con setW una cantidad fija si el centroide se desplaza más de nuestro umbral, esto para rectas funciona pero para curvas cerradas no, por lo que vamos a transformar la velocidad de la rotación en funcion del desplazamiento del centroide.

Imagen del centroide
![image](https://github.com/m4r4d0n4/m4r4d0n4.github.io/assets/58432330/6176f941-5035-42a4-9ad6-c0a94a912d0a)

Tras ajustar la velocidad de rotación con el desplazamiento del centroide logramos que el coche ya sea capaz de realizar una vuelta limpia, pero actualmente está la velocidad fija, se producen balanceos y el coche no es rápido (110s por vuelta en circuito simple).

### 19/02 

Actualmente el ajuste de la rotación se hace según un umbral dependiendo de la posición en el eje x del centroide. Sin embargo, vamos a probar a controlar su desplazamiento, este desplazamiento es la diferencia entre la ultima coordenada x y la nueva coordenada x, estos valores se obtienen por cada frame renderizado, el tiempo entre frames no podemos saberlo de momento, por lo que calcularemos un deltatime para poder tenerlo y calcular una rotación más precisa.

Ha surgido un nuevo problema y es que, además del balanceo, al aumentar la velocidad llegamos a unos puntos límites donde la curva se aleja lo suficiente como para que el centroide "parpadee". No obtenemos un centroide preciso por lo que vamos a intentar obtener un centroide más preciso sobre el que construir. Según he leído, hay varias aproximaciones, usar la media de las posiciones (algo que descarto porque en cambios bruscos de curva no creo que sea óptimo), interpolación, que puede ser una primera aproximación sobre la que evaluar su funcionamiento aunque dudo que sea la mejor opción y un filtro de Kalman que nos permite predicir la posición actual del centroide basado en datos anteriores. El filtro de Kalman, que desconozco actualmente su funcionamiento y su implementación, creo que puede ser la mejor opción.

### 22/02

Se va a hacer un PID para rectas y otro para curvas (tanto para rotacion como velocidad), donde el error sera la distancia al centroide en el eje x. Tambien voy a intentar analizar la curva, es decir, con el trazado que percibimos proyectar dicho trazado como si lo viesemos en planta (homografía), de manera que puedo analizar la curva en 2D sobre la carretera. Mi objetivo es poder obtener la curvatura y asi poder decidir si es mas cerrada o mas abierta y poder predecir lo antes posible el cambio de PID, ademas de evaluar la posibilidad de usar el parametro de la curvatura para dar un mejor ajuste en la velocidad. 

En resumen, se va a implemenar un controlador PID generico y a partir de este, como tenemos 2 grados de libertad: rotacion y velocidad, generamos 4 PIDs para rectas y curvas.

Por otra parte, se va a generar una automata de estados, de manera que podamos capturar los casos mas limites, como la no visualizacion de la recta, y podamos acotar el numero de situaciones inesperadas que podamos recibir. La idea del 19/02 del delta_centroide la dejamos de tener en cuenta y con ello el filtro de kallman, solo precisamos de la distancia que separa el punto ideal del centroide y el punto actual.

Por último, se va a realizar un calibrado del punto de referencia del centroide, actualmente el centroide objetivo se encuentra en la mitad de la pantalla, pero lo ideal es que esté levemente desplazado. Encontraremos experimentalmente este punto.

También he pensado en alguna manera de detectar la cercanía con las paredes como posible parámetro para ajustar el error del PID de las velocidades. Probablemente analizando la intensidad del color de las paredes sea una buena aproximación. También me he fijado que al acercarnos una curva se renderiza las paredes del fondo, de manera que si vemos una pared oscura delante en una región determinada de la pantalla podemos asegurar que nos acercamos a una curva.


## 24/02 

Ya tenemos los 4 controladores y los valores para los controladores PID de rectas ajustados, ahora tenemos que ajustar los PIDs para curvas. Para ello, primero necesitamos encontrar una manera fiable de detectar si estamos en una curva o una recta. Tambien hemos calibrado el cntroide del eje x que es 370. Para detectar la curva vamos a usar la convex hull delc ontorno de la curva y segun los defectos que tenga, mientras mas defectos mayor es la curva, experimental se esperan tener mas de 12 defectos si la curva no es recta. Estan los 4 PIDs calibrados, aun requieren de optimizar pero ya completa todos los circuitos aunque los tiempos son muy mejorables, sin embargo, esta siendo dificil aumentar la velocidad sin afectar al balanceo, al aumentar el u_b del controlador P en los PID de velocidad tenemos un balanceo muy grande que reuqiere de un reajuste de las K en las rotaciones. Por otra parte, en ocasiones el centroide x se mantenia pero el centroide y subia en la pantalla de manera que el error que nos daba no era del todo correcto, esto se producia en curvas abiertas y largas, por ello ahora en nuestro error tenemos en cuenta dicha coordenada, nuestro error sera la norma del vector que separa el centroide y nuestro centroide objetivo, el signo vendrá determinado por el signo del centroide x.

## 25/02

Contamos con un prototipo funcional que ya recorre el circuito, pero tenemos información que no estamos teniendo en cuenta como las distnacias a las paredes. A intentar ajustar la información dada por el centroide, evaluando solo considerar un fragmento de la pantalla. Para evitar las rotaciones bruscas vamos a limitar el PDI que puede dar, para rotacion será (2,-2) y para velocidad 30. El método actual a veces no detecta las curvas que son muy abiertas, por lo que se acompañara de otro metodo que mire como de ancho es el trazado como manera auxiliar para asegurarnos de que estamos en uan curva o una recta

