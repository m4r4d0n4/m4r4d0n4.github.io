# Web Visión Robótica 

Página web de Juan Montes Cano para la asignatura de Visión Robótica del MUVA de la URJC

## Práctica 1 Fórmula 1

### 18/02

Primeras pruebas filtrando la imagen y obteniendo el color rojo, de donde sacamos el contorno de la franja y obtenemos su centroide. Lo usaremos para controlar cuanto hay que rotar el coche. La primera aproximación que se da es ajustar con setW una cantidad fija si el centroide se desplaza más de nuestro umbral, esto para rectas funciona pero para curvas cerradas no, por lo que vamos a transformar la velocidad de la rotación en funcion del desplazamiento del centroide.

Imagen del centroide
![image](https://github.com/m4r4d0n4/m4r4d0n4.github.io/assets/58432330/6176f941-5035-42a4-9ad6-c0a94a912d0a)

Tras ajustar la velocidad de rotación con el desplazamiento del centroide logramos que el coche ya sea capaz de realizar una vuelta limpia, pero actualmente está la velocidad fija, se producen balanceos y el coche no es rápido (110s por vuelta en circuito simple).

### 19/02 

Actualmente el ajuste de la rotación se hace según un umbral dependiendo de la posición en el eje x del centroide. Sin embargo, vamos a probar a controlar su desplazamiento, este desplazamiento es la diferencia entre la ultima coordenada x y la nueva coordenada x, estos valores se obtienen por cada frame renderizado, el tiempo entre frames no podemos saberlo de momento, por lo que calcularemos un deltatime para poder tenerlo y calcular una rotación más precisa.


