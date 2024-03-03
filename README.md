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

La primera aproximación al error es `error = centroide_objetivo_x-centroide_observado_x`, sin embargo, como el centroide objetivo no está exactamente en medio de la pantalla y para obtener un error en [-1,1] llevamos a cabo una normalización, por tanto, `error = (centroide_objetivo_x-centroide_observado_x)/centroide_objetivo_x`

Centroide observado en la imagen original
![image](https://github.com/m4r4d0n4/m4r4d0n4.github.io/assets/58432330/27112050-a154-4acb-8530-e0beef1091c6)

Centroide observado en la imagen umbralizada y recortada
![image](https://github.com/m4r4d0n4/m4r4d0n4.github.io/assets/58432330/aae038bc-ef0c-4d09-aab3-32599fee8bd7)



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

El error que pasamos a nuestros PIDs es el mencionado previamente, pero para el caso de la velocidad pasamos un valor absoluto del error puesto que no nos interesa su signo sino su magnitud, es decir, cómo de cerca está el vehículo del centroide. 

La tarea de ajuste de parámetros de los PIDs se ha llevado a cabo manualmente, lo que implica el ajuste de 8 parámetros. (Hablar de q esta manera no es optima y tal tal tal en alguan seccion de mejroas y eso) (nuestros valores de PIDs)


### Máquina de estados

Se han descrito los PIDs y cómo según la posición en el trazado actúan unos u otros, pero hasta el momento no se tiene manera de distinguir si el vehículo se encuentra en una curva o en una recta. Este proyecto ha mezclado dos formas de obtener si estamos ante una recta o una curva. La primera es el uso de la función `cv2.convexityDefects` para obtener los defectos (es decir, partes donde no hay convexidad) de la convex hull de la línea roja. La línea roja cuando es recta es convexa pero al aparecer una curva surgen defectos. Experimentalmente se han encontrado los números de defectos necesarios para categorizar una curva.

Otra manera con la que aseguramos que estamos ante una curva, ha sido tomar la máscara de la curva y hacer cortes horizontales para quedarnos segmentos de la curva. En estas etapas, se encuentra su punto promedio, de manera que obtenemos al final una línea segmentada que tiene la estructura de la curva. Para saber si estos puntos forman una recta o no, usamos la ecuación de la recta y = mx + b, calcularemos la pendiente y la b con dos puntos y veremos si el resto de puntos está cerca o no de la recta calculada. Se establece una tolerancia hallada experimentalmente que nos indicará si el vehículo está en una curva o un recta.

En la parte superior de la imagen se aprecian los puntos que calculamos, los proyectamos en el cielo para no ensuciar la imagen de la cámara y visualizar claramente la curva. 
![image](https://github.com/m4r4d0n4/m4r4d0n4.github.io/assets/58432330/f4420cc7-d320-4fb6-93ac-9d3ef0e6a07a)

En el caso de una recta
![image](https://github.com/m4r4d0n4/m4r4d0n4.github.io/assets/58432330/fe879a27-f6df-494a-ba0a-cf2188c148f8)

En cuanto al convex hull tenemos esta visualización donde se aprecian los defectos(por defecto la tenemos quitada porque ensucia mucho la imagen)

![image](https://github.com/m4r4d0n4/m4r4d0n4.github.io/assets/58432330/4b1dfc60-6e7c-4502-aa6e-3f88b718ffa0)


Con la detección de curvas hecha, la máquina de estados resultante es muy simple, si se está en una curva se usan los PIDs de curvas, si se está en recta se usarán los PIDs de recta y si se está en un estado desconocido se rotará el coche hasta encontrar la línea roja.


### Resultados

Carpeta con los vídeos, solo accesible con usuario @urjc.es
[Carpeta con los vídeos](https://urjc-my.sharepoint.com/:f:/g/personal/juan_montes_urjc_es/EkIDjlBM_zFDg9y-xqpzB2ABRTc5QFR2A4z2gQ9MTQYldw?e=GBHwh3)

Se han grabado usando la herramiento de grabación de Gazebo.

Se ha comprobado el funcionamiento del vehículo en todos los circuitos y en el circuito simple en sentido opuesto. Además, se ha trasladado el coche con el simulador para que mire la pared y ver si es resiliente a cambios imprevistos.

### Alternativas intentadas

Dentro de cada una de las posibilidades vistas se han probado varias opciones. El detector de curvas tuvimos otros dos candidatos, la primera idea fue obtener los puntos como los obtenemos actualmente pero simplemente comparar las pendientes entre los puntos dos a dos, esta pendiente debe ser similar si estamos en una línea. El problema es que la línea muchas veces esta muy en horizontal y la pendiente es muy grande y en las curvas a veces no lo es es tanto, intenté de alguna manera normalizar las diferencias pero sin éxito y opté por el método de ver las diferencias con la línea creada por dos puntos, que además es un método que nos da un resultado cuantitativo que tiene en cuenta a todos los puntos. También se intentó otro método analizando únicamente la región donde se ve la curva, es decir, el horizonte de la pista. Se intentó trazar una línea usando la transformada de Hough y si esa línea era muy larga es que estábamos en una recta y si era corta sería una curva, sin embargo, al ponerlo en práctica no hubo éxito.

En cuanto a la calibración de los PIDs, se pensó que existiría alguna manera de ajustarlos automáticamente, forzando la simulación automáticamente y variandolos poco a poco siguiendo algún algoritmo y reiniciando la simulación, teniendo el error del centroide y el tiempo obtenido como objetivo a conseguir. Se intentó montar un prototipo pero no se encontró la manera de reiniciar la simulación por código.

También se lograron buenos resultados usando dos PIDs e incluso uno. Si se pone un PID para la rotación del coche y escogemos la velocidad según la rotación se obtienen resultados muy buenos, y estos resultados se pueden mejorar mucho porque únicamente se necesitan calibrar 2 parámetros.

### Mejoras posibles

La principal mejora que se puede obtener es un ajuste mucho más fino de los 8 parámetros de los PIDs, aunque actualmente sigue la línea en ciertas ocasiones hay un pequeño balanceo que podría mitigarse. Y el coche no va todo lo rápido que podría ir, sin embargo, el ajuste de parámetros es tedioso y la velocidad afecta a la rotación por lo que, en ocasiones, tocar un solo parámetro llega a afectar los otros tres relacionados.

Se podría aprovechar mejor la máquina de estados que tenemos con nuestra información cuantitativa de las curvas, puesto que se puede decir cómo de cerrada es una curva, se podría clasificar las curvas en varios tipos (o tener esta información a la hora de realizar la curva). 


### Problemas encontrados

El principal problema que se ha encontrado al realizar la práctica es el ajuste de los PIDs, puesto que al tener 4 es altamente complejo. A pesar de que nuestros PIDs no son los mejores, se han invertido dos días únicamente en ello.

También ha sido importante entender que prácticamente todos los elementos de nuestro robot están interconectados, por lo que el malfuncionamiento de un elemento provoco el malfuncionamiento de otro. En este caso concreto, al principio los umbrales para la detección de curvas no estaban bien puestos y eso daba la sensación de que nuestros PIDs no funcionaban correctamente. Una vez arreglado el problema de detección, los PIDs funcionaron como estaba previsto.

### Conclusión

Este proyecto ha sido interesante e incluso un poco adictivo, el hecho de querer superar tus marcas de tiempo y ver tu sistema lo más robusto posible es emocionante. Sin embargo, es un proyecto que puede causar frustración porque al más mínimo ajuste en la velocidad hay que retocar todos los valores de rotación, lo que no permitía evolucionar el vehículo fluidamente.

Desde el punto de vista personal, me ha gustado el proyecto y me interesaría ver cosas como qué hacer si hay un obstáculo en la carretera, como hay que girar, frenar, bordear el objeto... Me he dado cuenta de que problemas como ese, que a priori nos pueden parecer sencillos realmente no lo son y tienen su complejidad para un funcionamiento fiable, robusto y resiliente.







