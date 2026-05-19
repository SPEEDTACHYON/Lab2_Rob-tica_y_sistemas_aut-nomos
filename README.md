# Lab2_Robotica_y_sistemas_autonomos
En el presente repositorio del laboratorio 2 se encuentra el readme con el seguimiento de la programación y la simulación en WeBots tal como el laboratorio 1, y se presentan los demás directorios con la evidencia del código del controlador y el escenario utilizado 


## Laboratorio 2 : Navegación reactiva con filtrado y fusión de sensores en Webots

## Integrantes : 

Joaquín Rojas (Rol:Documentador,Github : "SPEEDTACHYON")

Juan Rodríguez (Rol:Programador del robot, Github : " ")

## Objetivos del trabajo :

### Objetivo del Laboratorio 2 :

Implementar un sistema b´asico de navegación reactiva en Webots para un
robot móvil diferencial, utilizando sensores de distancia y encoders de rueda,
aplicando filtrado sobre las mediciones y empleando un filtro de Kalman para
estimar la distancia frontal a obstáculos y mejorar la toma de decisiones.

### Objetivo del grupo ;

Nuestro objetivo como grupo es asignarnos roles y dividirnos el trabajo correctamente en partes equitativas, lograr una mayor comprensión sobre el filtrado y fusión de sensores en un simulador de un entorno del robot real (Webots), y lograr entregar un trabajo de laboratorio que cubra todos los requerimientos de entrega presentados en el PDF "Laboratorio 2" del aula virtual.

## Descripción del robot y sensores utilizados:

Nuestro robot será un robot móvil diferencial con una locomoción por ruedas el cual fue estpeticamente hecho en e-puck.wbt en Webots, los sensores que utilizaremos en el robot serán : 

### 1. Sensores de distancia y proximidad :

**(a) sensores infrarrojos (IR Distance Sensors, DistanceSensors en Webots)  :**

Para  los sensores frontales de distancia, y los sensores laterales.

**(b) sensores de ultrasonido (Sonar o Ultrasonic sensors) :**

Para usarlos configurar DistanceSensors configurados como tipo "sonar", Por qué? Porque dan mayor rango de apertura y de distancia que los infrarrojos

### 2. Encoders de rueda : Sensores de posición angular (PositionSensors)

Se usan estos sensores porque se asocian directamente a los motores de las ruedas (RotationalMotor)

Encoder de la rueda izquierda : Un sensor PositionSensor asociado a motor izquierdo (left wheel motor)

Encoder de la rueda derecha : Un sensor PositionSensor asociado a motor derecho (right wheel motor)


## Frecuencia de muestreo empleada:

La frecuencia de muestro permite  registrar la lectura de los sensores y encoders, se usará una frecuencia de muestreo fija, para usar esta frecuencia de muestreo debemos definir:

**tiempo de muestreo (Ts) :** También conocido como periodo, es el intervalo que transcurre entre dos lecturas del sensor se mide generalmente en segundos (s) o milisegundos (ms).

**frecuencia de muestreo :** La cantidad de lectura que realiza el sensor por cada segundo, es inverso al tiempo de muestro fs= 1/Ts.

**cantidad de muestras regstradas por cada experimento** : es la cantidad total de ejecucipones o iteraciones que se aplica la frecuencia de muestreo en cada experimento.

Ejemplo :

Ts = 4.16 ms.

fs = 1000ms(1s) / Ts = 1000 / 4.16 = 240 Hz

El experimento duró 20 segundos, entonces la cantidad de muestras registradas en el experimento fueron 20 * fs = 4800 ejecuciones.

## Análisis de las señales registradas:

Aquí se deben analizar las señales registradas en la frecuencia de muestreo fija :

Consistencia (Desviación estándar matemática) : mide si cada señal registrada coincide en el tiempo teórico establecido, por ejemplo, si el tiempo de muestreo Ts es 10 ms, qué tan cerca están las señales de ese valor, y si presentan algún sesgo (error sistemático) o un error aleatorio, y en base a eso qué filtrado simple podríamos aplicar , ejemplo : filtrado del promedio, calibración.

Valores atípicos en señales registradas : Señales que se desvién excesivamente del tiempo de muestreo.


## Estimación del avance mediante encoders:

Primero debemos establecer la variable a estimar.

**Primera variable :** "distancia frontal  al obstáculo más cercanos.

Y ahora debemos establecer las fuentes de información, es decir, en qué basarnos para estimar la variable principal :

**Predicción :** Estima los valores antes de ser registrados, se obtendra a partir de los encoders de las ruedas, estimando cuanto avanzo el robot entre dos instantes consecutivos.

**Corrección :** Rectifica los valores registrados que estén alejados del valor real, se realizara con las lecturas de los sensores frontales de distancia, los cuales entregan una medicion directa, pero ruidosa, de la cercana de obstaculos.

## Filtro simple aplicado:

## Implementación del filtro de Kalman:

## Descripción de las etapas de predicción y corrección: 

## Lógica de navegación reactiva implementada:

## Gráfico de señales crudas, filtradas y estimadas:

## Resultados obtenidos en los escenarios de prueba:

## Instrucciones para ejecutar la simulación:







