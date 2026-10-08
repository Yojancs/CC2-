# Preguntas y respuestas: Rasterización de polígonos y transformaciones geométricas

## 1. ¿Qué diferencia existe entre rasterizar la frontera de un polígono y rellenar su interior?

La rasterización de la frontera de un polígono consiste en determinar qué píxeles deben dibujarse para representar sus aristas o bordes. Para ello, se utilizan algoritmos que calculan los píxeles que mejor aproximan las líneas que conectan los vértices del polígono en la pantalla.

Por otro lado, el relleno del interior consiste en identificar los píxeles que se encuentran dentro de la frontera del polígono para colorearlos. En el algoritmo de relleno mediante líneas de barrido (*Scanline*), se recorren las filas de la imagen y se calculan las intersecciones de cada fila con las aristas del polígono. Luego, se rellenan los píxeles comprendidos entre pares de intersecciones.

Por ejemplo, si tenemos un polígono cuadrado, la rasterización de la frontera dibuja únicamente sus cuatro lados, mientras que el relleno colorea toda la región comprendida entre ellos.

En conclusión, **la frontera define el contorno del polígono y el relleno representa su superficie interior**. Ambos procesos son complementarios, pero cumplen funciones diferentes dentro de la representación gráfica.

## 2. ¿Por qué las aristas horizontales no se incorporan a la ET en el algoritmo trabajado?

Las aristas horizontales no se incorporan a la Tabla de Aristas (ET, *Edge Table*) porque sus coordenadas inicial y final tienen el mismo valor en el eje Y. Por lo tanto, la variación vertical es cero.

La variación vertical se calcula mediante la siguiente expresión:

$$
\Delta y = y_{\max} - y_{\min}
$$

En una arista horizontal:

$$
y_{\max} = y_{\min}
$$

Por lo tanto:

$$
\Delta y = 0
$$

El algoritmo de relleno mediante líneas de barrido utiliza la pendiente inversa de cada arista para calcular cómo cambia la coordenada X cuando aumenta la coordenada Y:

$$
m^{-1} = \frac{\Delta x}{\Delta y}
$$

Si se incorpora una arista horizontal, se produciría una división entre cero, lo cual no está definido matemáticamente.

Además, una arista horizontal no atraviesa diferentes filas de barrido, ya que todos sus puntos tienen la misma coordenada Y. Por ello, no es necesario incluirla en la ET para calcular las intersecciones utilizadas en el relleno interior.

**En conclusión**, se excluyen las aristas horizontales porque no aportan intersecciones entre distintas filas de barrido y porque su inclusión provocaría problemas al calcular la pendiente inversa.

## 3. ¿Por qué una arista deja de estar activa al alcanzar su ymax?

Una arista deja de estar activa cuando la línea de barrido alcanza su coordenada máxima en Y (\(y_{\max}\)), porque a partir de ese punto ya no debe participar en el cálculo de las intersecciones de las siguientes filas.

En el algoritmo, cada arista se registra en la ET con información como su coordenada mínima en Y, su coordenada máxima en Y, la coordenada X inicial de la intersección y su pendiente inversa. Cuando la línea de barrido llega al extremo superior de una arista, esta debe retirarse de la Tabla de Aristas Activas (EAT, *Active Edge Table*).

La condición de eliminación se expresa habitualmente de la siguiente manera:

$$
y \geq y_{\max}
$$

Donde:

- \(y\): coordenada actual de la línea de barrido.
- \(y_{\max}\): coordenada vertical máxima de la arista.

Esta condición permite trabajar con un intervalo vertical que incluye el extremo inferior, pero excluye el extremo superior. De esta manera, se evita contar dos veces los vértices compartidos por dos aristas consecutivas, especialmente cuando una línea de barrido pasa exactamente por un vértice.

**En conclusión**, eliminar una arista al alcanzar su \(y_{\max}\) permite mantener actualizada la EAT, evitar intersecciones duplicadas y realizar el relleno del polígono de manera consistente.

## 4. ¿Qué representa Δx/Δy durante la actualización de la EAT?

El valor \(\Delta x/\Delta y\) representa la pendiente inversa de una arista, es decir, la variación horizontal de la arista por cada unidad de variación vertical. Este valor permite actualizar de manera incremental la coordenada X de la intersección sin tener que calcular nuevamente la ecuación completa de la recta en cada fila.

Se calcula mediante la siguiente fórmula:

$$
\frac{\Delta x}{\Delta y}
=
\frac{x_{\max}-x_{\min}}{y_{\max}-y_{\min}}
$$

Donde:

- \(x_{\max}\) y \(x_{\min}\): coordenadas horizontales de los extremos de la arista.
- \(y_{\max}\) y \(y_{\min}\): coordenadas verticales de los extremos de la arista.

Durante el recorrido de las líneas de barrido, la coordenada X se actualiza mediante la expresión:

$$
x_{\text{nuevo}}
=
x_{\text{actual}}+\frac{\Delta x}{\Delta y}
$$

Por ejemplo, si una arista tiene una variación horizontal de 6 unidades y una variación vertical de 3 unidades, su pendiente inversa será:

$$
\frac{\Delta x}{\Delta y}=\frac{6}{3}=2
$$

Esto significa que, por cada fila que avanza la línea de barrido, la coordenada X de la intersección aumenta en 2 unidades.

Este procedimiento hace que el algoritmo sea más eficiente, ya que reutiliza la información calculada previamente en lugar de resolver nuevamente la ecuación de la recta en cada iteración.

**En conclusión**, \(\Delta x/\Delta y\) indica cuánto debe desplazarse horizontalmente una intersección al pasar de una línea de barrido a la siguiente.

## 5. ¿Por qué ET y EAT deben calcularse a partir de las coordenadas actuales después de transformar un polígono?

Las transformaciones geométricas, como la traslación, la rotación y el escalamiento, modifican las coordenadas de los vértices del polígono. Estos cambios pueden alterar la posición, la orientación, el tamaño y las pendientes de sus aristas.

Por este motivo, la Tabla de Aristas (ET) debe construirse utilizando las coordenadas actualizadas después de aplicar las transformaciones. A partir de esta tabla, se organiza la información necesaria para inicializar y actualizar la Tabla de Aristas Activas (EAT) durante el relleno.

Por ejemplo, la rotación de un polígono puede modificar las pendientes de sus aristas, mientras que la traslación cambia sus coordenadas mínimas y máximas. El escalamiento, por su parte, puede modificar tanto sus dimensiones como las coordenadas de intersección.

Si se utilizaran las tablas calculadas antes de la transformación, el algoritmo trabajaría con datos que ya no corresponden a la geometría actual del polígono. Esto podría producir un relleno incorrecto, intersecciones fuera de lugar o aristas activas en filas que no corresponden.

El procedimiento correcto es:

1. Obtener las coordenadas originales de los vértices.
2. Aplicar las transformaciones geométricas necesarias.
3. Actualizar las coordenadas de todos los vértices.
4. Construir nuevamente la ET con las coordenadas transformadas.
5. Inicializar y actualizar la EAT durante el proceso de relleno.

**En conclusión**, las tablas ET y EAT deben reflejar siempre la geometría actual del polígono para garantizar que las intersecciones y el relleno correspondan a su posición, tamaño y orientación finales.

## 6. ¿Qué ventaja ofrecen las coordenadas homogéneas para integrar traslación, rotación y escalamiento?

Las coordenadas homogéneas permiten representar las transformaciones geométricas mediante matrices de tamaño \(3 \times 3\) en dos dimensiones. Su principal ventaja es que permiten expresar la traslación, la rotación y el escalamiento utilizando una misma estructura matemática.

Un punto bidimensional se representa mediante tres componentes:

$$
P =
\begin{bmatrix}
x \\
y \\
1
\end{bmatrix}
$$

### Traslación

La traslación permite desplazar un punto una distancia \(t_x\) en el eje X y una distancia \(t_y\) en el eje Y. Su matriz es:

$$
T =
\begin{bmatrix}
1 & 0 & t_x \\
0 & 1 & t_y \\
0 & 0 & 1
\end{bmatrix}
$$

Al multiplicar la matriz por el punto, se obtiene:

$$
P' = TP =
\begin{bmatrix}
x+t_x \\
y+t_y \\
1
\end{bmatrix}
$$

### Rotación

La rotación permite girar un punto un ángulo \(\theta\) alrededor del origen. Su matriz es:

$$
R =
\begin{bmatrix}
\cos\theta & -\sin\theta & 0 \\
\sin\theta & \cos\theta & 0 \\
0 & 0 & 1
\end{bmatrix}
$$

### Escalamiento

El escalamiento permite modificar el tamaño de un objeto mediante los factores \(s_x\) y \(s_y\). Su matriz es:

$$
S =
\begin{bmatrix}
s_x & 0 & 0 \\
0 & s_y & 0 \\
0 & 0 & 1
\end{bmatrix}
$$

### Combinación de transformaciones

Una de las ventajas más importantes de las coordenadas homogéneas es que permiten combinar varias transformaciones en una sola matriz compuesta. Por ejemplo, para aplicar primero el escalamiento, después la rotación y finalmente la traslación, se utiliza:

$$
P' = T R S P
$$

En este caso, las matrices se multiplican en el orden indicado y la matriz resultante permite transformar el punto mediante una sola multiplicación.

Esto simplifica la implementación del programa, evita escribir procedimientos independientes para cada combinación de transformaciones y facilita aplicar las mismas operaciones a todos los vértices de un polígono.

**En conclusión**, las coordenadas homogéneas unifican las transformaciones geométricas en una representación matricial común, facilitan su combinación y permiten implementar el movimiento, la rotación y el cambio de tamaño de los polígonos de forma más organizada y eficiente.
