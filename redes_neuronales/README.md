name: inverse
layout: true
class: center, middle, inverse
---
template: inverse
![RN](../imagenes/logo_horizontal.png)
# Redes Neuronales
***
---
layout: false
# REDES NEURONALES
***
Cuándo se empieza a programar en un lenguaje nuevo existe **"Hello World"**, en Deep Learning se empieza por crear un modelo de reconocimiento de números escritos a mano.

Mediante este ejemplo, se presentarán algunos conceptos básicos de las redes neuronales, reduciendo todo lo posible conceptos teóricos, con el objetivo de ofrecer al lector una visión global de un caso concreto para facilitar la lectura de los capítulos posteriores donde se entrará en más detalle de diferentes temas del área.

En este capítulo también se mostrará cómo se codifica este ejemplo con Keras para ofrecer al lector un primer contacto con esta librería y entender la estructura que tiene la implementación de este ejemplo con Keras.
---
## Caso de estudio: reconocimiento de dígitos

En este apartado presentamos los datos que usaremos para nuestro primer ejemplo de redes neuronales: el conjunto de datos MNIST, que contiene imágenes de dígitos escritos a mano.

El conjunto de datos MNIST, que se pueden descargar de la página The MNIST database[1], está formado por imágenes de dígitos hechos a mano.

Este conjunto de datos contiene 60 000 elementos para entrenar el modelo y 10 000 adicionales para testearlo, y es ideal para adentrarse por primera vez en técnicas de reconocimiento de patrones sin tener que dedicar mucho tiempo al preproceso y formateado de datos, ambos pasos muy importantes y costosos en el análisis de datos[2] y de especial complejidad cuando se está trabajando con imágenes.

Este conjunto de imágenes originales en blanco y negro han sido normalizadas a 20×20 píxeles conservando su relación de aspecto.

En este caso, es importante notar que las imágenes contienen niveles de grises como resultado de la técnica de anti-aliasing[3], usada en el algoritmo de normalización (reducir la resolución de todas las imágenes a la de resolución más baja).

Posteriormente, las imágenes se han centrado en una de 28×28 píxeles, calculando el centro de masa de estos y trasladando la imagen con el fin de posicionar este punto en el centro del campo de 28×28.

Estilo de las imágenes:

![RN](imagenes/001.png)

Además, el conjunto de datos dispone de una etiqueta (label) por cada una de las imágenes que indica qué dígito representa, tratándose pues de un aprendizaje supervisado.

Esta imagen de entrada se representa en una matriz con las intensidades de cada uno de los 28×28 píxeles con valores entre [0, 255].

Por ejemplo esta imagen (la octava del conjunto de entrenamiento):

![RN](imagenes/002.png)

Se representa con esta matriz de puntos de 28×28.

![RN](imagenes/003.png)

Por otro lado, recordemos que tenemos las etiquetas, que en nuestro caso son números entre 0 y 9 que indican qué dígito representa la imagen, es decir, a qué clase se asocia.

En este ejemplo, vamos a representar esta etiqueta con un vector de 10 posiciones, donde la posición correspondiente al dígito que representa la imagen contiene un 1 y el resto son 0. Este proceso de transformar las etiquetas en un vector de tantos ceros como el número de etiquetas distintas, y poner un 1 en el índice que le corresponde la etiqueta, se conoce como **one-hot encoding**.
---
## Perceptron

Antes de avanzar, una breve explicación intuitiva de cómo funciona una sola neurona para cumplir con su cometido de aprender del conjunto de datos de entrenamiento.

### Algoritmos de regresión

Los algoritmos de regresión modelan la relación entre distintas variables de entrada (features) utilizando una medida de error, la loss, que se intentará minimizar en un proceso iterativo para poder realizar predicciones “lo más acertadas posibles”. Hablaremos de dos tipos:
- regresión logística
- regresión lineal.

La diferencia principal entre regresión logística y lineal es en el tipo de salida de los modelos; cuando nuestra salida sea discreta, hablamos de regresión logística, y cuando la salida sea continua hablamos de regresión lineal.

La ___regresión logística___ es un algoritmo con aprendizaje supervisado y se utiliza para clasificar.

El ejemplo que usaremos a continuación, que consiste en identificar a qué clase pertenece cada ejemplo de entrada asignándole un valor discreto de tipo 0 o 1, se trata de una ___clasificación binaria___.
---
### Una neurona artificial simple

Para mostrar cómo es una neurona básica, supongamos un ejemplo simple, donde tenemos un conjunto de puntos en un plano de dos dimensiones, y cada punto ya se encuentra etiquetado como _“cuadrado”_ o _“redonda”_:

![RN](imagenes/004.png)

---
Dado un nuevo punto “X”, queremos saber qué etiqueta le corresponde:

![RN](imagenes/005.png)

Una aproximación habitual es dibujar una línea que separe los dos grupos y usarla como clasificador:

![RN](imagenes/006.png)

En este caso, los datos de entrada serán representados por vectores de la forma (x1, x2) que indican sus coordenadas en este espacio de dos dimensiones, y nuestra función retornará ‘0’ o ‘1’ (encima o debajo de la línea) para saber si se debe clasificar como “cuadrado” o “círculo”.

Como hemos visto, se trata de un caso de regresión lineal, donde “la línea” (el clasificador) puede ser definida por la recta

![RN](imagenes/007.png)

De manera más generalizada, podemos expresar la recta como:

![RN](imagenes/008.png)

Para clasificar elementos de entrada X, en nuestro caso de dos dimensiones, debemos aprender un vector de peso W de la misma dimensión que los vectores de entrada, es decir, el vector (w1, w2) y un sesgo b.

Con estos valores calculados, ahora ya podemos construir una neurona artificial para clasificar un nuevo elemento X.

Básicamente la neurona aplica este vector W de pesos calculado, de manera ponderada sobre los valores en cada dimensión del elemento X de entrada, le sumará el sesgo b, y el resultado lo pasará a través de una función de “activación” no lineal para producir un resultado de ‘0’ o ‘1’.

La función de esta neurona artificial que acabamos de definir puede expresarse de una manera más formal, cómo:

![RN](imagenes/009.png)

En lo que respecta a aprender los parámetros W y b a partir de los datos de que ya disponemos etiquetados cómo “cuadrados” o “círculos”; se trata de un proceso iterativo para todos los ejemplos etiquetados conocidos, comparando el valor de su label obtenida a través del modelo, con el valor esperado de la label de cada elemento.

Después de cada iteración, se ajustan los pesos de los parámetros W y b de tal manera que se minimice la función de loss definida anteriormente.

Una vez tenemos los parámetros W y b podemos calcular el valor de z. Entonces, necesitaremos una función que aplique una transformación a esta variable para qué se convierta en ‘0’ o ‘1’.

Aunque hay varias funciones, para este ejemplo usaremos una conocida como función sigmoid[4] que retorna un valor real de salida entre 0 y 1 para cualquier valor de entrada:

![RN](imagenes/010.png)

Si se piensa un poco en la fórmula, veremos que tiende siempre a dar valores próximos al 0 o al 1.

Si la entrada z es razonablemente grande y positiva, “e” a la menos z es cero y, por tanto, la y toma el valor de 1.

Si z tiene un valor grande y negativo, resulta que para “e” elevado a un número positivo grande, el denominador resultará ser un número grande y por lo tanto el valor de y será próximo a 0.

Hasta aquí hemos presentado cómo se puede definir una neurona artificial, la arquitectura más simple que puede tener una red neuronal.

En concreto esta arquitectura se conoce como Perceptron[5] (llamado también linear threshold unit (LTU)), inventada en 1957 por Frank Rosenblatt, y que visualmente se resume de manera general con el siguiente esquema:

![RN](imagenes/011.png)

El Perceptron es la versión más simple de red neuronal porque consta de una sola capa que contiene una sola neurona. Pero lo normal hoy en día es que nos encontremos con redes neuronales compuestas de numerosas capas y que cada una de ellas contenga muchas neuronas que se comunican con las de la capa anterior para recibir información, y estas a su vez comunican su información a las neuronas de la capa siguiente.

Hay varias funciones de activación además de la sigmoid, cada una con propiedades diferentes, por ejemplo otra función de activación llamada softmax[6], que será útil para presentar un ejemplo de red neuronal mínima para clasificar en más de dos clases.

Podemos considerar a la función softmax como una generalización de la función sigmoid que permite clasificar más de dos clases.

### Multi-Layer Perceptron

Antes de avanzar con el ejemplo, introduciremos brevemente la forma que toman habitualmente las redes neuronales construidas a partir de perceptrones como el que acabamos de presentar.

En la literatura del área nos referimos a un Multi-Layer Perceptron (MLP) cuando nos encontramos con redes neuronales que tienen una capa de entrada (input layer), una o más capas compuestas por perceptrones, llamadas capas ocultas (hidden layers) y una capa final con varios perceptrones llamada la capa de salida (output layer).

En general nos referimos a Deep Learning cuando el modelo basado en redes neuronales está compuesto por múltiples capas ocultas. Visualmente se puede presentar con el siguiente esquema:

![RN](imagenes/012.png)

Los MLP son a menudo usados para clasificación, y en concreto cuando las clases son exclusivas, como es el caso de la clasificación de imágenes de dígitos que nos ocupa (en clases de 0 hasta 9), la capa de salida es una función softmax en la que la salida de cada neurona corresponde a la probabilidad estimada de la clase correspondiente. Visualmente lo podríamos representar de la siguiente forma:

![RN](imagenes/013.png)

### Función de activación softmax

Vamos a resolver el problema de manera que, dada una imagen de entrada, obtendremos las probabilidades de que sea cada uno de los 10 posibles dígitos.

De esta manera tendremos un modelo que, por ejemplo, podría predecir en una imagen un nueve, pero solo estar seguro en un 80% de que sea un nueve,  ya que debido al dudoso bucle inferior piensa que podría llegar a ser un ocho en un 5% de posibilidades e incluso podría dar una cierta probabilidad a cualquier otro número.

Aunque en este caso concreto consideraremos que la predicción de nuestro modelo es un 9, pues es el que tiene mayor probabilidad, esta aproximación de usar una distribución de probabilidades nos puede dar una mejor idea de cuán confiados estamos de nuestra predicción. Esto es bueno en este caso, donde los números son hechos a mano, y seguramente en muchos otros no podemos reconocer los dígitos con un 100% de seguridad.

Por tanto, para este ejemplo de clasificación de MNIST, para cada ejemplo de entrada obtendremos como vector de salida de la red neuronal una distribución de probabilidad sobre un conjunto de etiquetas mutuamente excluyentes, es decir, un vector de 10 probabilidades cada una correspondiente a un dígito y que todas estas 10 probabilidades sumen 1 (las probabilidades se expresarán entre 0 y 1).

Esto se logra mediante el uso de una capa de salida en nuestra red neuronal con la función de activación softmax, en la que cada neurona en esta capa softmax depende de las salidas de todas las otras neuronas de la capa, puesto que la suma de la salida de todas ellas debe ser 1.

La función softmax se basa en calcular “las evidencias” de que una determinada imagen pertenece a una clase en particular y luego se convierten estas evidencias en probabilidades de que pertenezca a cada una de las posibles clases.

Para medir la evidencia de que una determinada imagen pertenece a una clase en particular, una aproximación consiste en realizar una suma ponderada de la evidencia de pertenencia de cada uno de sus píxeles a esa clase.

Supongamos que disponemos ya del modelo aprendido para el número cero. Por el momento, podemos considerar un modelo como “algo” que contiene información para saber si un número es de una determinada clase. En este caso, para el número cero, supongamos que tenemos un modelo como el que presentamos a continuación:

![RN](imagenes/014.png)

Con una matriz de 28×28 píxeles, donde los píxeles en rojo representa pesos negativos (es decir, reducir la evidencia de que pertenece), mientras que los píxeles en azul representan pesos positivos (aumenta la evidencia de que pertenece). El color negro representa el valor neutro.

Imaginemos una hoja en blanco encima sobre la que trazamos un cero. En general, el trazo de nuestro cero caería sobre la zona azul (recordemos que estamos hablando de imágenes que han sido normalizadas a 20×20 píxeles y posteriormente centradas a una imagen de 28×28).

Resulta bastante evidente que si nuestro trazo pasa por encima de la zona roja lo más probable es que no estemos escribiendo un cero; por tanto, usar una métrica basada en sumar si pasamos por zona azul y restar si pasamos por zona roja, parece razonable.

Para confirmar que es una buena métrica imaginemos ahora que trazamos un tres; está claro que la zona roja del centro del anterior modelo que usábamos para el cero va a penalizar la métrica antes mencionada puesto que como podemos ver en la parte izquierda de esta figura al escribir un tres pasamos por encima:

Pero en cambio, si el modelo de referencia es el correspondiente al 3 como el mostrado en la parte derecha de la anterior figura, podemos observar que, en general, los diferentes posibles trazos que representan un tres se mantienen mayormente en la zona azul.

![RN](imagenes/015.png)

En la siguiente figura se muestra los pesos de un ejemplo concreto de modelo aprendido para cada una de estas diez clases del MNIST:

![RN](imagenes/016.png)

Recordemos que hemos escogido el rojo en esta representación visual para los pesos negativos, mientras que usaremos el azul para representar los positivos[8].

Una vez se ha calculado la evidencia de pertenencia a cada una de las 10 clases, estas se deben convertir en probabilidades cuya suma de todos sus componentes sume 1.

Para ello softmax usa el valor exponencial de las evidencias calculadas y luego las normaliza de modo que sumen uno, formando una distribución de probabilidad. La probabilidad de pertenencia a la clase i es:

![RN](imagenes/017.png)

Intuitivamente, el efecto que se consigue con el uso de exponenciales es que una unidad más de evidencia tiene un efecto multiplicador y una unidad menos tiene el efecto inverso. Lo interesante de esta función es que una buena predicción tendrá una sola entrada en el vector con valor cercano a 1, mientras que las entradas restantes estarán cerca de 0. En una predicción débil tendrán varias etiquetas posibles, que tendrán más o menos la misma probabilidad.

Datos para alimentar una red neuronal
A continuación pasemos a un nivel más práctico en el ejemplo de reconocimiento de dígitos MNIST, pero antes aprovechemos para explicar algunos detalles interesantes sobre los datos disponibles.
Conjunto de datos para entrenamiento, validación y prueba
Antes de presentar la implementación en Keras del ejemplo anterior, recordemos cómo debemos repartir los datos disponibles para poder configurar y evaluar el modelo correctamente.
Para la configuración y evaluación de un modelo en Machine Learning, y por ende Deep Learning, habitualmente se dividen los datos disponibles en tres conjuntos: datos de entrenamiento (training), datos de validación (validation) y datos de prueba (test). Los datos de entrenamiento son los que se usan para que el algoritmo de aprendizaje obtenga los parámetros del modelo. Si el modelo no acaba de adaptarse a los datos de entrada (por ejemplo, si presentara overfitting), en este caso modificaríamos el valor de ciertos hiperparámetros y después de entrenarlo nuevamente con los datos de entrenamiento volveríamos a evaluarlo con los de validación. Podemos ir haciendo estos ajustes de los hiperparámetros guiados por los datos de validación hasta que obtenemos unos resultados de validación que consideremos correctos (en la segunda parte del libro entraremos en detalle en este tema de validación).  Si hemos seguido este procedimiento, debemos ser conscientes de que, en realidad, los datos de validación han influido en el modelo para que se ajustara también a los datos de validación. Por este motivo reservamos siempre un conjunto de datos de prueba para evaluación final del modelo que solo se usarán al final de todo el proceso, cuando consideremos que el modelo está acabado de afinar y ya no modificaremos más ninguno de sus hiperparámetros. Veremos en más detalle este funcionamiento en futuros ejemplos de la segunda parte del
libro.
Precarga de los datos en Keras
En Keras el conjunto de datos MNIST se encuentra precargado en forma de cuatro arrays Numpy y se pueden obtener con el siguiente código:
import keras
from keras.datasets import mnist
(x_train, y_train), (x_test, y_test) = mnist.load_data()
x_train y  y_train conforman el conjunto de entrenamiento, mientras que x_test y y_test contienen los datos para el test. Las imágenes se encuentran codificadas como arrays Numpy y sus correspondientes etiquetas (labels) que van desde 0 hasta 9.  Siguiendo la estrategia dellibro de ir introduciendo gradualmente los conceptos del tema, dejamos para más adelante cómo separar una parte de los datos de entrenamiento para guardarlos como los datos validación, y de momento solo tendremos en cuenta los datos de entrenamiento y de prueba.
Si queremos comprobar qué valores hemos cargado, podemos elegir cualquiera de las imágenes del conjunto MNIST, por ejemplo la imagen 8,  y usando el siguiente código Python:
import matplotlib.pyplot as plt plt.imshow(x_train[8], cmap=plt.cm.binary)
Obtenemos la siguiente imagen:

Y si queremos ver su correspondiente etiqueta (label) podemos hacerlo mediante:
print(y_train[8])
1
Que como vemos nos devuelve el valor de “1”,  como era de esperar.
Representación de los datos en Keras
Keras, que como hemos visto usa un array multidimensional de Numpy como estructura básica de datos, le llama a esta estructura de datos tensor.
De manera resumida podríamos decir que un tensor tiene tres atributos principales:

Número de ejes (Rank o ndim): un tensor que contiene un solo número lo llamaremos scalar (o un tensor 0-dimensional, o tensor 0D). Un array de números lo llamamos vector, o tensor 1D. Un array de vectores será una matriz (matrix), o tensor 2D. Si empaquetamos esta matriz en un nuevo array, obtenemos un tensor 3D, que podemos interpretarlo visualmente como un cubo de números. Empaquetando un tensor 3D en un array, podemos crear un tensor 4D, y así sucesivamente. En la librería Numpy de Python esto se llama ndim del tensor.
Forma (shape): se trata de una tupla de enteros que describen cuántas dimensiones tiene el tensor en cada eje. Un vector tiene un shape con un único elemento, por ejemplo “(5,)”, mientras que un escalar tiene un shape vacío “( )”. En la librería Numpy este atributo se llama shape. Tipo de datos (data type): este atributo indica el tipo de datos que contiene el tensor, que pueden ser por ejemplo uint8, float32, float64, etc. En raras ocasiones tenemos, en nuestro contexto, tensores de tipo char (nunca string). En la librería Numpy este atributo se llama dtype.
Les propongo que obtengamos el número de ejes y dimensiones del tensor train_images de nuestro ejemplo anterior:
print(x_train.ndim)
3
print(x_train.shape)
(60000, 28, 28)
Y si queremos saber qué tipo de datos contiene:
print(x_train.dtype) uint8
Normalización de los datos de entrada
Estas imágenes de MNIST de 28×28 píxeles se representan como una matriz de números cuyos valores van entre [0, 255] de tipo uint8. Perocomo veremos en posteriores capítulos, es habitual escalar los valores de entrada de las redes neuronales a unos rangos determinados. En el ejemplo de este capítulo los valores de entrada conviene escalarlos a valores de tipo float32 dentro del intervalo [0, 1]:
Por otro lado, para facilitar la entrada de datos a nuestra red neuronal (veremos que en convolucionales no hace falta) debemos hacer una transformación del tensor (imagen) de 2 dimensiones (2D) a un vector de una dimensión (1D). Es decir, la matriz de 28×28 números se puede representar con un vector (array) de 784 números (concatenando fila a fila), que es el formato que acepta como entrada una red neuronal densamente conectada como la que veremos en este capítulo.
En Python, convertir cada imagen del conjunto de datos MNIST a un vector con 784 componentes se puede hacer de la siguiente manera:
x_train = x_train.reshape(60000, 784) x_test = x_test.reshape(10000, 784)
Después de ejecutar estas instrucciones Python, podemos comprobar que x_train.shape toma la forma de (60000, 784) y x_test.shape toma la forma de (10000, 784), donde la primera dimensión indexa la imagen y la segunda indexa el píxel en cada imagen (ahora la intensidad del píxel es un valor entre 0 y 1):
print(x_train.shape) print(x_test.shape)
(60000, 784)
(10000, 784)
Además tenemos las etiquetas (labels) para cada dato de entrada, (recordemos que en nuestro caso son números entre 0 y 9 que indican qué dígito representa la imagen, es decir, a que clase se asocia). En este ejemplo, y como ya hemos avanzado, vamos a representar esta etiqueta con un vector de 10 posiciones, donde la posición correspondiente al dígito que representa la imagen contiene un 1 y el resto de posiciones del vector contienen el valor 0.
En este ejemplo usaremos lo que se conoce como one-hot encoding, que ya hemos indicado  que explicaremos más adelante en la segunda parte del libro: en resumen, consiste en transformar las etiquetas en un vector de tantos ceros como el número de etiquetas distinta, y que contiene el valor de 1 en el índice que le corresponde al valor de la etiqueta. Keras ofrece muchas funciones de soporte, y entre ellas to_categorical para realizar esta transformación, que la podemos importar de keras.utils:
from keras.utils import to_categorical
Para ver el efecto de la transformación podemos ver los valores antes y después de aplicar to_categorical :
print(y_test[0])
x_train = x_train.astype('float32') x_test = x_test.astype('float32')
x_train /= 255 x_test /= 255
7
print(y_train[0])
5
print(y_train.shape)
(60000,)
print(x_test.shape) (10000, 784)
y_train = to_categorical(y_train, num_classes=10) y_test = to_categorical(y_test, num_classes=10)
print(y_test[0])
[0. 0. 0. 0. 0. 0. 0. 1. 0. 0.]
print(y_train[0])
[0. 0. 0. 0. 0. 1. 0. 0. 0. 0.]
print(y_train.shape) (60000, 10)
print(y_test.shape)
(10000, 10)
Ahora ya tenemos los datos preparados para ser usados en nuestro ejemplo de modelo simple que vamos a programar en Keras en la próxima sección.
Redes densamente conectadas en Keras
En este apartado vamos a presentar cómo se especifica en Keras el modelo que hemos definido en los apartados anteriores.
Clase Sequential en Keras
La estructura de datos principal en Keras es la clase Sequential, que permite la creación de una red neuronal básica. Keras ofrece también una API[9] que permite implementar modelos más complejos en forma de grafo que pueden tener múltiples entradas, múltiples salidas, con conexiones arbitrarias en medio, pero no lo presentaremos hasta el capítulo octavo del libro.
La clase Sequential[10]  de la librería de Keras es una envoltura para el modelo de red neuronal secuencial que ofrece Keras y se puede crear  de la siguiente manera:
from keras.models import Sequential model = Sequential()
En este caso, el modelo en Keras se considera como una secuencia de capas que cada una de ellas va “destilando” gradualmente los datos de entrada para obtener la salida deseada. En Keras podemos encontrar todos los tipos de capas requeridas y se pueden agregar fácilmente al modelo mediante el método add( ).
Definición del modelo
La construcción en Keras de nuestro modelo para reconocer las imágenes de dígitos podría ser el siguiente:
from keras.models import Sequential from keras.layers.core import Dense, Activation
model = Sequential() model.add(Dense(10, activation='sigmoid', input_shape=(784,))) model.add(Dense(10, activation='softmax'))
Aquí, la red neuronal se ha definido como una secuencia de dos capas que están densamente conectadas, es decir, que todas las neuronas de cada capa están conectadas con todas las neuronas de la capa siguiente. Visualmente podríamos representarlo de la siguiente manera:

En este código expresamos explícitamente en el argumento input_shape  de la primera capa cómo son los datos de entrada: un tensor que indica que tenemos 784 features del modelo (en realidad el tensor que se está definiendo es de (None, 784,) como veremos más adelante).
Una característica muy interesante de la librería de Keras es que esta deducirá automáticamente la forma de los tensores entre capas después de la primera. Esto significa que el programador solo tiene que establecer esta información para la primera de ellas. Además, para cada capa indicamos el número de nodos que tiene y la función de activación que aplicaremos en ella (en este caso, sigmoid).
La segunda capa es una capa softmax de 10 neuronas, lo que significa que devolverá una matriz de 10 valores de probabilidad que representan a los 10 dígitos posibles (en general, la capa de salida de una red de clasificación tendrá tantas neuronas como clases, menos en una clasificación binaria, en donde solo necesita una neurona). Cada valor será la probabilidad de que la imagen del dígito actual pertenezca a cada una de ellas.
Un método muy útil que proporciona Keras para comprobar la arquitectura de nuestra modelo es summary():
model.summary()
_________________________________________________________________ Layer (type)                 Output Shape             Param # ================================================================= dense_1 (Dense)             (None, 10)                7850 _________________________________________________________________ dense_2 (Dense)             (None, 10)                110
=================================================================
Total params: 7,960
Trainable params: 7,960
Non-trainable params: 0
Más adelante entraremos en más detalle con la información que nos retorna el método summary(), pues resulta muy valioso este cálculo de parámetros y tamaños de los datos que tiene la red neuronal cuando empezamos a construir modelos de redes muy grandes.  Para nuestro ejemplo simple, vemos que indica que se requieren 7 960 parámetros (columna Param #), que corresponden a los 7 850 parámetros para la primera capa y 110 para la segunda.
En la primera capa por cada neurona i (entre 0 y 9) requerimos 784 parámetros para los pesos wij y por tanto 10 x 784 parámetros para almacenar los pesos de las 10 neuronas. Además de los 10 parámetros adicionales para los 10 sesgos bj correspondientes a cada una de ellas. En la segunda capa, al ser una función softmax, se requiere conectar todos sus 10 nodos con los 10 nodos de la capa anterior, y por tanto se requieren 10×10 parámetros wi además de los correspondientes 10 sesgos bj correspondientes a cada nodo.
En el manual de Keras se puede encontrar los detalles de los argumentos que podemos indicar para la capa Dense[11]. En nuestro ejemplo aparecen los más relevantes, donde el primer argumento indica el número de neuronas de la capa; el siguiente es la función de activación que usaremos en ella. En el siguiente capítulo hablaremos en más detalle de otras posibles funciones de activación más allá de las dos presentadas aquí: sigmoid y
softmax.
También a menudo se indica la inicialización de los pesos como argumento de las capas Dense. Los valores iniciales deben ser adecuados para que el problema de optimización converja tan rápido como sea posible. En el manual de Keras se puede encontrar las diversas opciones de inicialización[12].
Pasos para implementar una red neuronal en Keras
A continuación vamos a presentar una breve descripción de los pasos que debemos realizar para implementar una red neuronal básica, y en los siguientes capítulos iremos introduciendo gradualmente más detalles de cada uno de estos pasos.
Configuración del proceso de aprendizaje
A partir del modelo Sequential, podemos  definir las capas de manera sencilla con el método add(), tal como hemos avanzado en el apartado anterior. Una vez que tengamos nuestro modelo definido, podemos configurar cómo será su proceso de aprendizaje con el método compile( ), con el que podemos especificar algunas propiedades a través de argumentos del método.
El primero de estos argumentos es la función de loss que usaremos para evaluar el grado de error entre salidas calculadas y las salidas deseadas de los datos de entrenamiento.  Por otro lado, se especifica un optimizador que, como veremos, es la manera que tenemos de especificar el algoritmo de optimitzaación que permite a la red neuronal calcular los pesos de los parámetros a partir de los datos de entrada y de la función de loss definida.
Más detalle del  propósito exacto de la función de loss y el optimizador se presentarán en el siguiente capítulo.
Y finalmente debemos indicar la métrica que usaremos para monitorizar el proceso de aprendizaje (y prueba) de nuestra red neuronal. En este primer ejemplo solo tendremos en cuenta la accuracy (fracción de imágenes que son correctamente clasificadas). Por ejemplo, en nuestro caso podemos especificar los siguientes argumentos en método compile( ) para probarlo en nuestro ordenador:
model.compile(loss="categorical_crossentropy", optimizer="sgd", metrics = ['accuracy'])
Donde especificamos que la función de loss es categorical_crossentropy, el optimizador usado es el stocastic gradient descent (sgd) y la métrica es accuracy, con la que evaluaremos el porcentaje de aciertos averiguando dónde el modelo predice la etiqueta correcta.
Entrenamiento del modelo
Una vez definido nuestro modelo y configurado su método de aprendizaje, este ya está listo para ser entrenado. Para ello podemos entrenar o “ajustar” el modelo a los datos de entrenamiento de que disponemos invocando al método fit( ) del modelo:
model.fit(x_train, y_train, batch_size=100, epochs=5)
Donde en los dos primeros argumentos hemos indicado los datos con los que entrenaremos el modelo en forma de arrays Numpy. Con el argumento batch_size se indica el número de datos que usaremos para cada actualización de los parámetros del modelo y con epochs estamos indicando el número de veces que usaremos todos los datos en el proceso de aprendizaje.  Estos dos últimos argumentos se explicarán con mucho más detalle en el próximo capítulo.
Este método encuentra el valor de los parámetros de la red mediante el algoritmo iterativo de entrenamiento que presentaremos con un poco más de detalle en el siguiente capítulo. A grandes rasgos, en cada iteración de este algoritmo, este coge datos de entrenamiento de x_train, los pasa a través de la red neuronal (con los valores que en aquel momento tengan sus parámetros), compara el resultado obtenido con el esperado (indicado en y_train) y calcula la loss para guiar el proceso de ajuste de los parámetros del modelo, que intuitivamente consiste en aplicar el optimizador especificado anteriormente en el método compile() para calcular un nuevo valor de cada uno de los parámetros (pesos y sesgos) del modelo en cada iteración de tal forma de que se reduzca el de la loss.
Este es el método que, como veremos, puede llegar a tardar más tiempo y Keras nos permite ver su avance usando el argumento verbose (por defecto, igual a 1), además de indicar una estimación de cuánto tarda cada epoch:
Epoch 1/5
60000/60000 [==================] - 1s 15us/step - loss: 2.1822 - acc: 0.2916 Epoch 2/5
60000/60000 [==================] - 1s 12us/step - loss: 1.9180 - acc: 0.5283 Epoch 3/5
60000/60000 [==================] - 1s 13us/step - loss: 1.6978 - acc: 0.5937 Epoch 4/5
60000/60000 [==================] - 1s 14us/step - loss: 1.5102 - acc: 0.6537 Epoch 5/5
60000/60000 [==================] - 1s 13us/step - loss: 1.3526 - acc: 0.7034
10000/10000 [==================] - 0s 22us/step
Este es un ejemplo simple para que el lector al acabar el capítulo haya podido programar ya su primera red neuronal, pero como veremos  el método fit( ) permite muchos más argumentos que tienen un impacto muy importante en el resultado del aprendizaje. Además, este método retorna un objeto History que hemos omitido en este ejemplo. Su atributo History.history es el registro de los valores de loss para los datos de entrenamiento y resto de métricas en sucesivas epochs, así como otras métricas para los datos de validación si se han especificado. Más adelante, en el capítulo 5 de la segunda parte del libro, veremos lo valioso de esta información para evitar por ejemplo el sobreajuste del modelo.
Evaluación del modelo
En este punto ya se ha entrenado la red neuronal y ahora se puede evaluar cómo se comporta con datos nuevos de prueba (test) con el método evaluate().  Este devuelve dos valores:
test_loss, test_acc = model.evaluate(x_test, y_test)
Que indican cómo de bien o mal se comporta nuestro modelo con datos nuevos que nunca ha visto (que hemos almacenado en x_test y y_test cuando hemos realizado el mnist.load_data( ) ).  De momento fijémonos solo en uno de ellos, la accuracy:
print('Test accuracy:', test_acc)
Test accuracy: 0.9018
Que nos está indicando que el modelo que hemos creado en este capítulo aplicado sobre datos que nunca ha visto anteriormente, clasifica el 90% de ellos correctamente.
Debe el lector fijarse que, en este ejemplo, para evaluar este  modelo solo nos hemos centrado en su accuracy, es decir la proporción entre las predicciones correctas que ha hecho el modelo y el total de predicciones. Sin embargo, aunque en ocasiones resulta suficiente, otras veces es necesario profundizar un poco más y tener en cuenta los tipos de predicciones correctas e incorrectas que realiza el modelo en cada una de sus categorías.
En el mundo de Machine Learning una herramienta para evaluar modelos es la matriz de confusión (confusion matrix),  una tabla con filas y columnas que contabilizan las predicciones en comparación con los valores reales. Usamos esta tabla para entender mejor cómo el modelo se comporta de bien  y es muy útil para mostrar de forma explícita cuando una clase es confundida con otra. Una matriz de confusión para un clasificador binario como el explicado al principio del capítulo tiene esta estructura

En la que:
VP es la cantidad de positivos que fueron clasificados correctamente como positivos por el modelo.
VN es la cantidad de negativos que fueron clasificados correctamente como negativos por el modelo.
FN es la cantidad de positivos que fueron clasificados incorrectamente como negativos.
FP es la cantidad de negativos que fueron clasificados incorrectamente como positivos.
Con esta matriz de confusión, la accuracy se puede calcular sumando los valores de la diagonal y divido por el total:
Accuracy = (VP + VN) / (VP + FP + VN + FN)
Ahora bien, la accuracy puede ser engañosa en la calidad del modelo porque al medirla para el modelo concreto no distinguimos entre los errores de tipo falso positivo y falso negativo, como si ambos tuvieran la misma importancia. Por ejemplo, piensen en un modelo que predice si una seta es venenosa. En este caso, el coste de un falso negativo, es decir, una seta venenosa dada por comestible podría ser dramático. En cambio al revés, un falso positivo, tiene un coste muy diferente.
Por ello tenemos otra métrica llamada Sensitivity (o recall) que nos indica cómo de bien el modelo evita los falsos negativos:
Sensitivity = VP / (VP + FN)
Es decir, del total de observaciones positivas (setas venenosas), cuantas detecta el modelo.
A partir de la matriz de confusión se pueden obtener diversas métricas para focalizar otros casos como se muestra en este enlace[13], pero queda fuera del alcance de este libro entrar más detalladamente en este tema. La conveniencia de usar una métrica u otra dependerá de cada caso en particular y, en concreto, del “coste” asociado a cada error de clasificación del modelo.
Pero el lector se preguntará cómo es esta matriz de confusión en nuestro clasificador, donde tenemos 10 posibles valores. En este caso les propongo usar el paquete  Scikit-learn[14] (que ya hemos mencionado anteriormente) para evaluar la calidad del modelo calculando la matriz de confusión[15], presentada de la figura siguiente:

En este caso los elementos de la diagonal representan el número de puntos en que la etiqueta que predice el modelo coincide con el valor real de la etiqueta, mientras que los otros valores nos indican los casos en que el modelo ha clasificado incorrectamente. Por tanto, cuanto más altos son los valores de la diagonal mejor será la predicción. En este ejemplo, si el lector calcula la suma de los valores de la diagonal dividido por el total de valores de la matriz, observará que coincide con la accuracy que nos ha retornado el método evaluate().
En el GitHub del libro pueden encontrar el código usado para calcular esta matriz de confusión.
Generación de predicciones
Finalmente, nos queda el paso de usar el modelo creado en los anteriores apartados para realizar predicciones sobre qué dígito representan nuevas imáges. Para ello Keras ofrece el método predict() de un modelo que ya ha sido previamente entrenado.
Para probar este método podemos elegir un elemento cualquiera, por ejemplo uno del conjunto de datos de test x_test. Por ejemplo elijamos el  elemento 11 de este conjunto de datos x_test y veamos a que clase corresponde según el modelo entrenado de que disponemos.
Antes vamos a ver la imagen para poder comprobar nosotros mismos  si el modelo está haciendo una predicción correcta (antes de hacer el reshape anterior):
plt.imshow(x_test[11], cmap=plt.cm.binary)

Creo que el lector estará de acuerdo que en este caso se trata del número 6.
Ahora comprobemos que el método predict() del modelo, ejecutando el siguiente código, nos predice correctamente el valor que acabamos de estimar nosotros que debería predecir.
Para ello ejecutamos la siguiente línea de código:
predictions = model.predict(x_test)
Una vez calculado el vector resultado de la predicción para este conjunto de datos podemos saber a qué clase le da más probabilidad de
pertenencia  mediante la función argmax de Numpy, que retorna el índice de la posición que contiene el valor más alto del vector. En concreto, para el elemento 11:
np.argmax(predictions[11])
6
Podemos comprobar imprimiendo el array:
print(predictions[11])
[0.06 0.01 0.17 0.01 0.05 0.04 0.54 0.   0.11 0.02]
Vemos que nos ha devuelto el índice 6, correspondiente a la clase “6”, la que habíamos estimado nosotros.
También podemos comprobarlo que el resultado de la predicción es un vector cuya suma de todos sus componentes es igual a 1, como era de esperar. Para ello podemos usar:
np.sum(predictions[11])
1.0
Hasta aquí el lector ha podido crear su primer modelo en Keras que clasifica correctamente los dígitos MNIST el 90% de las veces. En el siguiente capítulo vamos a presentar cómo funciona el proceso de aprendizaje  y varios de los hiperparámetros que podemos usar en una red neuronal para mejorar estos resultados. En el capítulo 4 veremos cómo podemos mejorar estos resultados de clasificación usando redes neuronales convolucionales para el mismo ejemplo.