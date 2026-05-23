# Lab2_Robotica_y_sistemas_autonomos
En el presente repositorio del laboratorio 2 se encuentra el readme con el seguimiento de la programación y la simulación en WeBots tal como el laboratorio 1, y se presentan los demás directorios con la evidencia del código del controlador y el escenario utilizado 


## Laboratorio 2 : Navegación reactiva con filtrado y fusión de sensores en Webots

## Integrantes : 

Joaquín Rojas (Rol:Documentador,Github : "SPEEDTACHYON")

Juan Rodríguez (Rol:Programador del robot, Github : " ")

## Objetivos del trabajo :

### Objetivo del Laboratorio 2 :

Implementar un sistema básico de navegación reactiva en Webots para un
robot móvil diferencial, utilizando sensores de distancia y encoders de rueda,
aplicando filtrado sobre las mediciones y empleando un filtro de Kalman para
estimar la distancia frontal a obstáculos y mejorar la toma de decisiones.

### Objetivo del grupo ;

Nuestro objetivo como grupo es asignarnos roles y dividirnos el trabajo correctamente en partes equitativas, lograr una mayor comprensión sobre el filtrado y fusión de sensores en un simulador de un entorno del robot real (Webots), y lograr entregar un trabajo de laboratorio que cubra todos los requerimientos de entrega presentados en el PDF "Laboratorio 2" del aula virtual.

## Descripción del robot y sensores utilizados:

Nuestro robot será un robot móvil diferencial con una locomoción por ruedas el cual fue estpeticamente hecho en e-puck.wbt en Webots, los sensores que utilizaremos en el robot serán : 

### 1. Sensores de distancia y proximidad :

**(a) sensores infrarrojos (IR DistanceSensors, DistanceSensors en Webots)  :**

Para  los sensores frontales de distancia, y los sensores laterales.

**(b) sensores de ultrasonido (Sonar o UltrasonicSensors) :**

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

**Consistencia (Desviación estándar matemática) :** mide si cada señal registrada coincide en el tiempo teórico establecido, por ejemplo, si el tiempo de muestreo Ts es 10 ms, qué tan cerca están las señales de ese valor, y si presentan algún sesgo (error sistemático) o un error aleatorio, y en base a eso qué filtrado simple podríamos aplicar , ejemplo : filtrado del promedio, calibración.

**Valores atípicos en señales registradas :** Señales que se desvién excesivamente del tiempo de muestreo.


## Estimación del avance mediante encoders:

Primero debemos establecer la variable a estimar.

**Primera variable :** "distancia frontal  al obstáculo más cercano."

Y ahora debemos establecer las fuentes de información, es decir, en qué basarnos para estimar la variable principal :

**Predicción :** Estima los valores antes de ser registrados, se obtendra a partir de los encoders de las ruedas, estimando cuanto avanzo el robot entre dos instantes consecutivos.

**Corrección :** Rectifica los valores registrados que estén alejados del valor real, se realizara con las lecturas de los sensores frontales de distancia, los cuales entregan una medicion directa, pero ruidosa, de la cercana de obstaculos.


## Filtro simple aplicado:

**Usaremos 2 filtro simple principales :** el filtro de umbral ciego (Hard thresholding) y el filtro de promedio móvil suavizado

### 1) Filtro de umbral ciego (Hard thresholding) :

**Utilidad en este contexto :** Actúa como barrera de seguridad ante valores fuera de rango, como los sensores de distancia (LiDAR o ultrasónicos) son propensos a lecturas saturadas de 0.0 m o picos infinitos pérdida de eco, este algoritmo permite validar que los valores entrantes estén dentro de los límites físicos reales del hardware definidos entre 0.2 m y 4 m, cuando el sensor marca un valor fuera de este rango el filtro lo descarta de inmediato al ser físicamente imposible. 

**Impacto de control en el Filtro de Kalman :** Bloquea valores extremos directos, elimina el riesgo de que el Filtro de Kalman experimente saltos abruptos, lo que previene de forma absoluta la ejecución de frenados fantasma o las maniobras de evasión erróneas que pueden ser provocadas por la saturación de ruido acumulado.

### 2) Filtro de promedio móvil suavizado :

**Utilidad en este contexto :** A diferencia del filtro anterior, ya no necesitamos eliminar errores de lectura por fallas en el sensor, sino disminuir el ruido blanco  Gaussiano de alta frecuencia. Si un vehículo se encuentra en movimiento o los motores generan vibraciones mecánicas, la distancia medida puede oscilar de manera microscópica con un sesgo aleatorio (por ejemplo, fluctuando rápidamente entre 1.50 m y 1.53 m. El filtro toma las últimas N mediciones ya validadas y calcula el promedio o media aritmética.

**Impacto de control en el filtro de Kalman :** Este segundo filtro nos entrega una salida suavizada y continua con ruido blanco Gaussiano atenuado, y esta salida es la que finalmente se entrega como valor de salida oficial del sensor $z_k$ para las ecuaciones del Filtro de Kalman.


## Implementación del filtro de Kalman:

Para implementar el filtro de Kalman, hay que definir 6 parámetros iniciales clave :

**x̂_pred :** predicción de la posición x.

**Incertidumbre inicial(P0) :** nuestra seguridad sobre la posición inicial, generalmente 1.

**Variable de control (u) :** distancia frontal al obstáculo más cercano.

**Ruido del proceso (Q) :** Las pequeñas perturbaciones del entorno, generalmente oscila el valor 0.1 si es estable.

**Ruido de la medición (R) :** Este es el error propio del sensor, los rangos comunes son entre 0.25 (poco errado) a 1 (considerablemente errado)

El filtro de Kalman funciona por iteraciones (t) , y en cada iteración, el filtro de Kalman se acerca más al valor verdadero.

<img width="623" height="396" alt="image" src="https://github.com/user-attachments/assets/cb0c7510-d4d5-4985-ba79-4563eb599c17" />

### Ejemplo físico aplicado del filtro de Kalman :

x̂₀ = 25 m

P₀ = 0.9 (confiamos lo justo en nuestra posición inicial)

u = 1 m/s en un ∆t=1 (en este caso la variable de control es la rapidez)

Q = 0.2 (medianamente estable)

R = 0.8 (considerablemente errado)

(t=1)

en la primera iteración se nos entrega un valor de posición z₁ = 25.4 m

x̂₁pred = x̂₀ + u * ∆t = 25 + 1*1 = 26 m

P₁pred = P₀+ Q = 0.9 + 0.2 = 1.1

K₁= P₁pred / (P₁pred + R) = 1.1 / 1.9 = 0.579

x̂₁ = x̂₁pred + K₁ (ẑ₁ - x̂₁pred) = 26 + 0.579 (25.4 - 26) = 25.65

P₁ = (1 - K₁) * P₁pred = (1 - 0.579) * 1.1  = 0.4631

error porcentual de x̂₁ con respecto a z₁ = (x̂₁/z₁)*100% - 100% = 0.9 %

(t=2)

en la segunda iteración se nos entrega un valor de posición z₂ = 26.4 m

x̂₂pred = x̂₁ + u * ∆t = 25.65 + 1*1 = 26.65 m

P₂pred = P₁+ Q = 0.4631 + 0.2 = 0.6631

K₂= P₂pred / (P₂pred + R) = 0.6631 / 1.4631 = 0.4532

x̂₂ = x̂₂pred + K₁ (ẑ₂ - x̂₂pred) = 26.65 + 0.4532 (26.4 - 26.65) = 26.5367

P₂ = (1 - K₂) * P₂pred = (1 - 0.4532) * 0.6631  = 0.363

error porcentual de x̂₂ con respecto a z₂ = (x̂₂/z₂)*100% - 100% = 0.518 %

el error porcentual de x̂ con respecto a z siempre irá bajando por cada iteración.

hasta (t=k)

Para el caso de estudio, se debe aplicar el filtro de Kalman en k iteraciones como en el ejemplo, solo hay algo importante, en nuestro caso de estudio la variable de control es la distancia frontal al obstáculo más cercano, por tanto su notación no es $\hat{x}_k$ sino $\hat{d}_k$

## Descripción de las etapas de predicción y corrección: 

## Lógica de navegación reactiva implementada:

## Gráfico de señales crudas, filtradas y estimadas:

## Resultados obtenidos en los escenarios de prueba:

## Instrucciones para ejecutar la simulación:







