# Lab2_Robotica_y_sistemas_autonomos
En el presente repositorio del laboratorio 2 se encuentra el readme con el seguimiento de la programación y la simulación en WeBots tal como el laboratorio 1, y se presentan los demás directorios con la evidencia del código del controlador y el escenario utilizado 


## Laboratorio 2 : Navegación reactiva con filtrado y fusión de sensores en Webots

## Integrantes : 

Joaquín Rojas (Rol:Documentador,Github : "SPEEDTACHYON")

Juan Rodríguez (Rol:Programador del robot y creador de entornos utilizados, Github : "Ing-loop")

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

**tiempo de muestreo ($T_s$) :** También conocido como periodo, es el intervalo que transcurre entre dos lecturas del sensor se mide generalmente en segundos (s) o milisegundos (ms).

**frecuencia de muestreo ($f_s$) :** La cantidad de lectura que realiza el sensor por cada segundo, es inverso al tiempo de muestro $f_s$= 1/$T_s$.

**cantidad de muestras regstradas por cada experimento** : es la cantidad total de ejecucipones o iteraciones que se aplica la frecuencia de muestreo en cada experimento.

Ejemplo :

$T_s$ = 4.16 ms.

$f_s$ = 1000ms(1s) / $T_s$ = 1000 / 4.16 = 240 Hz

El experimento duró 20 segundos, entonces la cantidad de muestras registradas en el experimento fueron 20 * $f_s$ = 4800 ejecuciones.

## Análisis de las señales registradas:

Aquí se deben analizar las señales registradas en la frecuencia de muestreo fija :

**Consistencia (Desviación estándar matemática) :** mide si cada señal registrada coincide en el tiempo teórico establecido, por ejemplo, si el tiempo de muestreo Ts es 10 ms, qué tan cerca están las señales de ese valor, y si presentan algún sesgo (error sistemático) o un error aleatorio, y en base a eso qué filtrado simple podríamos aplicar , ejemplo : filtrado del promedio, calibración.

**Valores atípicos en señales registradas :** Señales que se desvién excesivamente del tiempo de muestreo.


## Estimación del avance mediante encoders:

Primero debemos establecer la variable a estimar.

**Primera variable :** "distancia frontal  al obstáculo más cercano."

Y ahora debemos establecer las fuentes de información, es decir, en qué basarnos para estimar la variable principal :

**Predicción :** Estima los valores antes de ser registrados, se obtendrá a partir de los encoders de las ruedas, estimando cuanto avanzo el robot entre dos instantes consecutivos.

**Corrección :** Rectifica los valores registrados que estén alejados del valor real, se realizará con las lecturas de los sensores frontales de distancia, los cuales entregan una medicion directa, pero ruidosa, de la cercana de obstaculos.


## Filtro simple aplicado:

**Usaremos 2 filtro simple principales :** el filtro de umbral ciego (Hard Thresholding) y el filtro de promedio móvil suavizado

### 1) Filtro de umbral ciego (Hard Thresholding) :

**Utilidad en este contexto :** Actúa como barrera de seguridad ante valores fuera de rango, como los sensores de distancia (LiDAR o ultrasónicos) son propensos a lecturas saturadas de 0.0 m o picos infinitos pérdida de eco, este algoritmo permite validar que los valores entrantes estén dentro de los límites físicos reales del hardware definidos entre 0.05 m y 1.414 m (fuera de 1.414 es una distancia del obstáculo fuera del tamaño máximo posible en una arena de tamaño 1 m *1 m) , cuando el sensor marca un valor fuera de este rango el filtro lo descarta de inmediato al ser físicamente imposible. 

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

### 1) La predicción de resultados

**Primero , recordemos qué debíamos hacer en la predicción :**

**Predicción :** Estima los valores antes de ser registrados, se obtendrá a partir de los encoders de las ruedas, estimando cuanto avanzo el robot entre dos instantes consecutivos.

**Ahora debemos aplicar la predicción :**

La fase de predicción nos plantea cuál debería ser nuestra nueva distancia hasta el obstáculo antes de leer los sensores. Se fundamenta principalmente en la odometría diferencial (odometría ; estudio y medición de las variables internas del robot) : los sensores de posición angular (PositionSensor) miden cuántos radianes giró cada una de las ruedas en el último periodo de tiempo de muestreo $T_s$.A partir del radio de la rueda ($R_{rueda}$).

**Calculamos el avance lineal de cada rueda:**

$$\Delta s_{izq} = R_{rueda} \cdot (\theta_{izq, k} - \theta_{izq, k-1})$$

$$\Delta s_{der} = R_{rueda} \cdot (\theta_{der, k} - \theta_{der, k-1})$$

El desplazamiento neto del centro del robot es el promedio de ambos avances: $\Delta s = \frac{\Delta s_{izq} + \Delta s_{der}}{2}$. Como el robot avanza hacia el obstáculo frontal, la distancia estimada disminuye en esa misma proporción. Recordando la aplicación del Filtro de Kalman,  las ecuaciones de predicción para el Filtro de Kalman son:


**Predicción del Estado (Distancia) :**


$$\hat{d}_{k|k-1} = \hat{d}_{k-1|k-1} - \Delta s$$


**Predicción de la Incertidumbre:**


$$P_{k|k-1} = P_{k-1|k-1} + Q$$



### 2) La corrección de resultados

**Primero , recordemos qué debíamos hacer en la corrección :**

**Corrección :** Rectifica los valores registrados que estén alejados del valor real, se realizará con las lecturas de los sensores frontales de distancia, los cuales entregan una medicion directa, pero ruidosa, de la cercana de obstaculos.

**Ahora debemos aplicar la corrección :**

La fase de corrección ajusta la predicción teórica utilizando la observación del entorno real. 

Sin embargo, para evitar que ruidos atípicos rompan el filtro, la medición física se somete a los filtrados simples que  diseñamos :

**Filtro de Umbral Ciego (Hard Thresholding) :** Si la lectura del sonar/IR está fuera de $[0.2\text{ m}, 4.0\text{ m}]$, se descarta y se asume el último estado válido para evitar frenados fantasma.

**Promedio móvil suavizado:** Las lecturas válidas por el Hard Tresholding entran para promediar el ruido blanco gaussiano de los motores, generando la medición corregida oficial $z_k$ para usar en las ecuaciones de Kalman.

Finalmente, calculamos la ganancia de Kalman ($K_k$), la cual decide si confiar más en la predicción de los encoders o en la lectura del sonar (según los valores de $Q$ y $R$).


**Ganancia de Kalman :** 


$$K_k = \frac{P_{k|k-1}}{P_{k|k-1} + R}$$


**Corrección del Estado :**


$$\hat{d}_{k|k} = \hat{d}_{k|k-1} + K_k \cdot (z_k - \hat{d}_{k|k-1})$$


**Corrección de la Incertidumbre :**


$$P_{k|k} = (1 - K_k) \cdot P_{k|k-1}$$



## Lógica de navegación reactiva implementada:

El algoritmo de navegación reactiva que diseñamos se basa principalmente en estímulo-respuesta, procesando la información del entorno para modificar la velocidad diferencial de los motores de manera inmediata. Para garantizar un desplazamiento fluido, la toma de decisiones críticas (avanzar, desacelerar o girar de emergencia) se ejecuta utilizando la variable de control del Filtro de Kalman ($\hat{d}_k$), mientras que los ajustes de dirección fina se apoyan en los sensores laterales. En este diagrama de flujo creado se explica el comportamiento del algoritmo frente a los dos escenarios de pruena utilizados (para "Escenario_utilizado_simple" y "Escenario_utilizado_desafiante") :

```

                                  ┌──────────────────────────┐
                                  │   Lectura de Sensores    │
                                  └────────────┬─────────────┘
                                               │
                                               ▼
                                  ┌──────────────────────────┐
                                  │ Pipeline Filtro / Kalman │
                                  └────────────┬─────────────┘
                                               │
                                               ▼
                         ┌───────────────────────────────────────────┐
                         │    ¿Distancia al obstáculo frontal d̂_k?   │
                         └─────────────────────┬─────────────────────┘
                                               │
                 ┌─────────────────────────────┼─────────────────────────────┐
                 │ (< 0.4m)                    │ (0.4m a 0.8m)               │ (> 0.8m)
                 ▼                             ▼                             ▼
   ┌───────────────────────────┐ ┌───────────────────────────┐ ┌───────────────────────────┐
   │     GIRO DE EMERGENCIA    │ │     EVITACIÓN GRADUAL     │ │      CRUCERO MÁXIMO       │
   │ (Giro sobre su propio eje)│ │     (Frenado y giro)      │ │   (Avance lineal libre)   │
   └───────────────────────────┘ └───────────────────────────┘ └───────────────────────────┘

```


## Gráfico de señales crudas, filtradas y estimadas:

<img width="989" height="590" alt="image" src="https://github.com/user-attachments/assets/b8ba6146-20a4-43ad-b280-64023fb20661" />

**Explicación del gráfico :**

Como vimos, en la sección del filtrado de Kalman, el filtro de Kalmaqn se va acercando consistentemente al valor real a lo largo del tiempo hasta tener un error prácticamente nulo al ajustarse a los datos reales, mientras el filtro promedio demora un poco más en ajustarse a la medición cruda, la medición cruda se mantiene prácticamente completamente consistente en un valor, solamente constando de micro-variaciones por ruido mínimo en el sensor.


## Resultados obtenidos en los escenarios de prueba:

Finalmente, en los resultados obtenidos en los escenarios de prueba, obtuvimos que, para el escenario_utilizado_simple se evidenció el siguiente comportamiento :

[0.1s] AVANZANDO | Dist. Kalman al obstaculo: 0.998 m | Sensor crudo: 1.1
[3.5s] AVANZANDO | Dist. Kalman al obstaculo: 0.540 m | Sensor crudo: 0.559
[6.8s] AVANZANDO | Dist. Kalman al obstaculo: 0.100 m | Sensor crudo: 0.106
[7.3s] AVANZANDO | Dist. Kalman al obstaculo: 0.045 m | Sensor crudo: 0.044
[7.4s] ESTADO: COLISION EVITADA! Esquivando en coord (X:0.96, Y:0.00)
[8.2s] ESTADO: Camino despejado, retomando avance...

**Resumen :** El robot avanza hasta que está lo suficientemente cerca del obstáculo, es detectado por los sensores a tiempo y se detiene la colisión, luego se demora un poco menos de un segundo en girar y retomar el avance ya habiendo evitado la colisión.

El principal problema suele presentarse cuando el entorno es más desafiante, y tiene múltiples obstáculos, para esto usamos múltiples obstáculos, porque si tiene múltiples obstáculos a la vez a distancia similar, puede caer en un "estado trampa", intenta evitar un obstáculo, pero al instante se encuentra con otro, si dificultabamos tanto el entorno en algunas situaciones puntuales el sensor indicaba que la colisión se evitó pero realmente el robot alcanza a impactar con el obstáculo minimamente, ejemplo :

[0.1s] AVANZANDO | Dist. Kalman al obstaculo: 0.998 m | Sensor crudo: 1.064
[3.5s] AVANZANDO | Dist. Kalman al obstaculo: 0.540 m | Sensor crudo: 0.559
[6.8s] AVANZANDO | Dist. Kalman al obstaculo: 0.100 m | Sensor crudo: 0.106
[7.3s] AVANZANDO | Dist. Kalman al obstaculo: 0.045 m | Sensor crudo: 0.044
[7.4s] ESTADO: COLISION EVITADA! Esquivando en coord (X:0.96, Y:0.22)
[8.2s] ESTADO: Camino despejado, retomando avance...
[8.3s] AVANZANDO | Dist. Kalman al obstaculo: 0.057 m | Sensor crudo: 0.0
[8.3s] ESTADO: COLISION EVITADA! Esquivando en coord (X:0.97, Y:0.23)
[9.1s] ESTADO: Camino despejado, retomando avance...

En este caso, el filtrado de Kalman no le da tiempo suficiente para ajustarse al valor exacto del sensor crudo y no le da tiempo suficiente para esquivar el obstáculo en la realidad porque la distancia cruda es de 0.0 m.

## Conclusiones analíticas

**1)** El filtrado de Kalman es un filtro que se aproxima al valor crudo correctamente a lo largo del tiempo, funciona bien para variables de control que no cambian abruptamente, sin embargo, en el caso de la distancia frontal a un obstáculo si hay saturación de obstáculos el filtro de Kalman puede ser insuficiente en algunas ocasiones.

**2)** Antes de utilizar el filtro de Kalman es esencial implementar filtros simples anteriores que comprueben que las distancias son realistas, como es el caso del filtrado simple Hard Thresholding, nos permite establecer rangos sin que salgan fuera de la realidad física del entorno.

**3)** Al implementar escenarios con obstáculos complejos en Webots, aprendimos que la complejidad del escenario afecta considerablemente en la precisión y certidumbre del robot, mientras más obstáculos tiene el entorno, las opciones de error del robot se disparan, la cantidad de rutas posibles se reduce considerablemente y el movimiento fluído en el entorno es cada vez más restringido.


## Instrucciones para ejecutar la simulación:

**1)** Inicializar la aplicación de webots

**2)** Empezar un nuevo entorno, y seleccionar el entrono predeterminado de e-puck.wbt.

**3)** Copiar y pegar el código de programación del robot en la parte derecha de código para Webots (lenguaje de programación C) que se encuentra en el directorio "Programación_del_robot"

**4)** Utilizar las lógicas de entorno e-puck.wbt tal cual, y copiar y pegar en entorno utilizado, los dos entornos utilizados en los directorios "Entrono_utilizado_simple" y "Entorno_utilizado_desafiante"

**5)** Listo, entonces ahora se puede simplemente revisar el correcto funcionamiento del robot y los registros de frecuencia de muestreo y filtrados que hace recurrentemente cada segundo.

**6)** Si surge un error en la simulación, háganos saber.







