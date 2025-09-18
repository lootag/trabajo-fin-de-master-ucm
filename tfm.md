<style>
  body {
    font-family: serif;
    font-size: 11pt;
    line-height: 1.5;
  }
  .title-page {
    text-align: center;
    padding-top: 15%;
    max-width: 800px;
    margin: 0 auto;
  }
  .title {
    font-size: 2.5em;
    font-weight: bold;
    margin-bottom: 0.5em;
    color: #333;
  }
  .subtitle {
    font-size: 1.5em;
    color: #666;
    margin-bottom: 1em;
  }
  .author, .date, .institution {
    font-size: 1.2em;
    margin-bottom: 0.5em;
    color: #555;
  }
  .institution {
    margin-top: 1.5em;
  }
</style>

<div class="title-page">
  <div class="institution">
    Universidad Complutense de Madrid
  </div>
  <div class="title">
    Compresión Híbrida de Imágenes con Redes Neuronales
  </div>
  <div class="subtitle">
    Un estudio comparativo de algorítmos de compresión residual tradicionales y basados en deep learning por la compresión de imágenes
  </div>
  <div class="author">
    Luigi Taglialatela
  </div>
  <div class="date">
    Septiembre 2025
  </div>
</div>

<div style="page-break-after: always; visibility: hidden;">
\pagebreak
</div>

### Tabla de Contenidos
1. [Introducción](#1-introducción)  
2. [Presentación de las Metodologías Empleadas](#2-presentación-de-las-metodologías-empleadas)  
3. [Conjunto de datos y estadísticas descriptivas](#3-conjunto-de-datos-y-estadísticas-descriptivas)  
4. [Conclusiones](#4-conclusiones)  
5. [Posibles desarrollos futuros](#5-posibles-desarrollos-futuros)  
6. [Referencias](#6-referencias)


<div style="page-break-after: always; visibility: hidden;">
\pagebreak
</div>


### 1. Introducción
La compresión de datos es un proceso fundamental en la era digital, esencial para la gestión eficiente y el almacenamiento de la vasta cantidad de información que generamos. Aunque se ha convertido en una herramienta omnipresente en campos que van desde las telecomunicaciones hasta el almacenamiento en la nube, presenta desafíos significativos, como la búsqueda constante de un equilibrio entre la reducción del tamaño del archivo y la calidad del dato. Dentro de este campo, es muy importante distinguir entre dos enfoques principales: la compresión sin pérdidas (lossless), que permite reconstruir el archivo original de manera idéntica, y la compresión con pérdidas (lossy), que sacrifica parte de la información para lograr una mayor reducción de tamaño, una técnica común en archivos de audio, video e imágenes.

En este estudio, vamos a explorar exclusivamente técnicas lossy, es decir técnicas que no permiten reconstruir los datos originales. En particular, vamos a enfocarnos en la compresión de imágenes con algoritmos residuales. En este contexto, un autoencoder se utiliza para aprender a representar la imagen de manera eficiente, generando un residuo que contiene la información que no pudo ser capturada por la codificación inicial. Este residuo es posteriormente cuantizado, y comprimido utilizando Zstandard (Zstd), un algoritmo lossless (sin pérdida) desarrolado por Facebook, conocido por su alta velocidad y buena relación de compresión. Para establecer una base de comparación, se contrastarán los resultados de esta aproximación con los de JPEG, un estándar de compresión de imagen tradicional, que nos servirá como línea base (baseline). De esta manera, podremos evaluar las ventajas del autoencoder y el enfoque residual frente a un método consolidado y ampliamente utilizado en la industria.


### 2. Presentación de las Metodologías Empleadas

Para lograr esto, empleamos dos métodos principales, cada uno evaluado contra un punto de referencia común de compresión JPEG pura.

* **Autoencoder estándar con residuos comprimidos a través de zstd**: Este enfoque utiliza un autoencoder clásico para generar una reconstrucción base de una imagen. Esta reconstrucción captura la información más dominante y de baja frecuencia. La diferencia entre la imagen original y esta reconstrucción base, conocida como el **residuo**, contiene los detalles más finos y la información de alta frecuencia. Luego, este residuo se cuantiza y se comprime utilizando el algoritmo de compresión de datos Zstandard (`zstd`), que es muy eficaz para datos redundantes. El archivo comprimido final es la combinación del código latente del autoencoder y los datos del residuo comprimido. 

* **Autoencoder Variacional (VAE) con residuos comprimidos a través de zstd**: Similar al autoencoder estándar, el VAE genera una reconstrucción base y residuos. Sin embargo, el espacio latente del VAE está regularizado, lo que lo obliga a seguir una distribución de probabilidad conocida. Esto hace que la representación latente sea altamente estructurada y más fácil de comprimir. Los residuos se procesan de la misma manera. Nuestra hipótesis es que la representación latente más eficiente del VAE conducirá a una mejor relación de compresión general.

Vamos a evaluar el algoritmo de compresión JPEG estándar en varios niveles de calidad para establecer un punto de referencia. Esto proporciona un claro punto de comparación para el rendimiento de la relación distorsión-tasa de todos los métodos híbridos, lo que nos permite evaluar de manera definitiva sus ventajas. Las métricas principales para la comparación son:

* **PSNR (Relación Señal-Ruido Pico)** : métrica de ingeniería que mide la calidad de reconstrucción de una imagen. Compara una imagen original con su versión comprimida y reconstruida para cuantificar cuánto ruido o distorsión se introdujo durante el proceso de compresión. Se expresa en decibelios (dB), donde un valor más alto indica una reconstrucción de mayor calidad y menos distorsión. El PSNR se basa en la diferencia de píxeles, lo que lo hace sensible a errores de píxeles individuales, pero no necesariamente refleja cómo el ojo humano percibe la calidad de la imagen.

* **SSIM (Índice de Similitud Estructural)**: métrica perceptiva que mide la similitud estructural entre dos imágenes. A diferencia del PSNR, que se centra en los errores de píxeles, el SSIM evalúa las similitudes en la luminancia, el contraste y la estructura de la imagen. Su objetivo es imitar mejor la forma en que el sistema visual humano percibe la distorsión. El valor de SSIM varía de -1 a 1, donde un valor de 1.0 indica una similitud perfecta (sin distorsión). El SSIM es a menudo un indicador más fiable de la calidad visual percibida que el PSNR.

* **Tasa de compresión (compression rate)**: medida de la eficiencia de un algoritmo de compresión. Se define como la relación entre el tamaño del archivo original y el tamaño del archivo comprimido. Una tasa de compresión más alta indica una reducción más significativa en el tamaño del archivo, lo que es deseable para ahorrar espacio de almacenamiento y banda de red. Sin embargo, una tasa de compresión muy alta puede llevar a una pérdida significativa de calidad en métodos con pérdida (lossy).  

El dataset fue dividido en un conjunto de entrenamiento (80%) y un conjunto de prueba (20%). Los modelos se entrenaron utilizando el optimizador Adam con una tasa de aprendizaje inicial de 0.001 y un tamaño de lote de 16. El entrenamiento se llevó a cabo durante 100 épocas, con una monitorización continua de la función de pérdida para evitar el sobreajuste. Todos los modelos se evaluaron utilizando el conjunto de prueba, y las métricas PSNR, SSIM y tasa de compresión se calcularon para cada imagen reconstruida.

Todos los métodos se implementaron utilizando PyTorch, una biblioteca de aprendizaje profundo ampliamente utilizada. El entrenamiento de los modelos se realizó con el backend MPS (Metal Performance Shaders) de Apple, aprovechando la GPU integrada en los dispositivos Mac para acelerar el proceso de entrenamiento.

### 3. Conjunto de datos y estadísticas descriptivas

El conjunto de datos utilizado para este estudio se compone de 271 imágenes de naturaleza provenientes de la base de datos abierta RAISE (https://loki.disi.unitn.it/RAISE/). Se realizó un análisis exhaustivo de sus características intrínsecas para comprender los factores que probablemente influirán en el rendimiento de la compresión. Estas sono las métricas principales que vamos a utilizar para evaluar el conjunto de datos:

* **Aspect ratio**: La relación entre el ancho y la altura de una imagen. Un aspect ratio uniforme en un conjunto de datos simplifica el preprocesamiento y puede ayudar a un modelo a aprender relaciones espaciales más consistentes.

* **Desviación Estándar de la Intensidad de Píxeles**: Una medida de la **textura y el detalle**. Una desviación estándar baja significa que los píxeles se agrupan alrededor del valor medio, lo que indica áreas suaves y con pocos detalles. Una desviación estándar alta significa que los píxeles están muy dispersos, lo que indica una imagen con mucha textura y muy ocupada.

* **Varianza Laplaciana**: Esta es una medida directa de la **nitidez de la imagen**. El operador laplaciano resalta los bordes y los detalles finos, y la varianza de este resultado cuantifica la nitidez de la imagen. Una varianza alta indica una imagen nítida y con muchos detalles, mientras que una varianza baja sugiere una imagen borrosa o suave.

* **Distribución de la Intensidad de Píxeles**: Esto muestra la frecuencia de cada valor de brillo (0-255) en todo el conjunto de datos. Revela el balance tonal general y la entropía de las imágenes. Una **distribución uniforme** indica una alta entropía, ya que cada valor de brillo es igualmente probable, lo que sugiere un alto grado de detalle y hace que las imágenes sean más difíciles de comprimir.

Las imágenes exhiben un alto grado de consistencia geométrica, con relaciones de aspecto agrupadas principalmente en torno a 1.5 (3:2) y 0.7 (2:3), lo que es típico para la fotografía digital tanto en orientación horizontal como vertical.


![Distribución de los aspect ratios](descriptive_stats/images/aspect_ratio_distribution.png)

La complejidad de la textura del conjunto de datos muestra un patrón más variado pero predecible. La distribución de la desviación estándar de la intensidad de píxeles es aproximadamente normal, lo que indica que la mayoría de las imágenes tienen un nivel similar de textura y detalle, centrado en un valor medio para todo el conjunto.  

![Distribución de la desviación estandard de los píxeles](descriptive_stats/images/pixel_std_dev_distribution.png)

Sin embargo, la distribución de la varianza laplaciana, una medida directa de la nitidez, está fuertemente sesgada a la izquierda. Esto sugiere que, a pesar de su variada textura, la mayoría de las imágenes son relativamente suaves y carecen de bordes nítidos y de alta frecuencia que suelen ser difíciles de comprimir.

![Distribución varianza laplaciana](descriptive_stats/images/laplacian_variance_distribution.png)

Por concluir, la distribución de la intensidad de píxeles es notablemente uniforme en todo el rango tonal (0-255), con algunos outliers entre los valores más altos. Esta uniformidad indica una alta entropía, ya que cada valor de brillo es igualmente probable. En términos de compresibilidad, esto sugiere que las imágenes son tonalmente complejas y contienen una amplia variedad de valores tonales, lo que generalmente dificulta la compresión.

![Distribución de la intensidad de píxeles](descriptive_stats/images/pixel_intensity_distribution.png)

Esta dualidad se ve aún más clara en las correlaciones entre estas métricas y el tamaño del archivo. Un gráfico del tamaño del archivo frente a la desviación estándar de la intensidad de píxeles revela solo una correlación lineal positiva débil, lo que sugiere que la textura por sí sola no es un predictor fuerte de la compresibilidad para este conjunto de datos.  
![Tamaño del archivo vs desviación estándar de la intensidad de píxeles](descriptive_stats/images/file_size_vs_texture.png)

En contraste, un gráfico similar del tamaño del archivo frente a la varianza laplaciana muestra una fuerte relación positiva, confirmando que la nitidez es el factor dominante que influye en el tamaño final del archivo.  Estos hallazgos combinados sugieren que el modelo de compresión debe centrarse en codificar de manera eficiente el contenido suave y de baja frecuencia de las imágenes, ya que esta es la característica más prevalente.
![Tamaño del archivo vs varianza laplaciana](descriptive_stats/images/file_size_vs_sharpness.png)


Resumiendo los resultados del análisis descriptivo, la distribución de la intensidad de píxeles fue **uniforme**, lo que normalmente indica una alta entropía y **baja compresibilidad**. Una distribución uniforme significa que una amplia variedad de valores tonales está presente, ofreciendo poca redundancia para la compresión. Sin embargo, la varianza laplaciana estaba **sesgada a la izquierda**, lo que indica que la mayoría de las imágenes son **suaves y carecen de detalles nítidos** y de alta frecuencia. La falta de información de alta frecuencia sugiere una alta compresibilidad. Esta contradicción significa que las imágenes son tonalmente complejas pero estructuralmente simples.

### Punto de referencia: JPEG

JPEG (Joint Photographic Experts Group) es un algoritmo de compresión con pérdida ampliamente utilizado para imágenes digitales. Funciona convirtiendo los datos de la imagen del espacio de color RGB al espacio de color YCbCr, que separa la luma (brillo) de la croma (color). Luego submuestrea los componentes de la croma, ya que el ojo humano es menos sensible a los detalles de color. Los datos se dividen luego en bloques de 8x8 y se aplica una **Transformada de Coseno Discreta (DCT)** a cada bloque. Esto convierte los datos de píxeles espaciales en componentes de frecuencia. Finalmente, se aplica la **cuantización**, que descarta la información de alta frecuencia redondeando los coeficientes de frecuencia según una configuración de calidad. Las configuraciones de mayor calidad dan como resultado una cuantización menos agresiva. 

Vamos a ulizar JPEG con nivel de calidad 90 como punto de referencia para evaluar los métodos híbridos. Se escoge este nivel porque produce resultados cuya diferencia con la imagen original es basicamente imperceptible al ojo humano, logrando un buen equilibrio entre calidad y tamaño del archivo. Es además el nivel que muchos softwares de edición de imágenes, comom por ejemplo GIMP, utilizan por defecto. 

Aquí una imagen de ejemplo del conjunto de datos original y su compresión a través de JPEG con calidad 90:
![Reconstrucción JPEG](raw_jpeg_results/examples/jpeg_example_07.png)

Y aquí los boxplots de las métricas de evaluación de la compresión JPEG:
![Metrícas de compresión JPEG](boxplots/raw_jpeg_boxplots.png)

### Autoencoder estándar con residuos comprimidos a través de zstd

Este método se basa en un **autoencoder estándar** para realizar una compresión con pérdida de los datos de la imagen principal. Un autoencoder es un tipo de red neuronal diseñada para el aprendizaje no supervisado de la codificación de datos. Consiste en dos partes principales: un **codificador** (encoder) y un **decodificador** (decoder). El codificador toma una imagen de entrada y la comprime en una representación pequeña y densa llamada **vector latente**. Este proceso es una forma de reducción de dimensionalidad, ya que obliga a la red a aprender las características más esenciales de la imagen. Luego, el decodificador reconstruye la imagen a partir de este vector latente. La diferencia entre la imagen original y la imagen reconstruida, conocida como el **residuo**, captura la información de alta frecuencia y los detalles finos que el autoencoder descarta. Vamos a cuantizar el residuo a 4 bits (por lo que el algoritmo resulta ser lossy), y luego lo comprimimos utilizando el algoritmo de compresión de datos sin pérdidas **Zstandard (zstd)**. El archivo comprimido final es una combinación del vector latente del autoencoder y los datos del residuo comprimido. Aquí podemos ver el diagrama de la arquitectura del autoencoder:
![Autoencoder estándar](autoencoder_architecture.png)

Se eligió esta arquitectura específica con el fin de lograr una compresión de imágenes eficiente y efectiva. El diseño se basa en algunos principios clave comunes en visión artificial.

1. Encoder: Capas Convolucionales
Se utilizaron capas convolucionales (nn.Conv2d) en el codificador para reducir progresivamente el tamaño de la imagen. Las capas convolucionales son excelentes para esta tarea porque aprenden automáticamente a extraer características espaciales y patrones de los datos de la imagen. Al usar un stride de 2, cada capa reduce las dimensiones de la imagen a la mitad mientras aumenta el número de canales. Esto efectivamente "canaliza" la información de la imagen hacia una representación más abstracta y comprimida. El uso de múltiples capas permite a la red aprender una jerarquía de características, desde bordes simples en la primera capa hasta texturas y formas complejas en las capas más profundas.

2. Decoder: Capas Convolucionales Transpuestas
El decodificador refleja la estructura del codificador utilizando capas convolucionales transpuestas (nn.ConvTranspose2d), también conocidas como capas de "desconvolución". Estas capas realizan lo opuesto a una convolución, aumentando gradualmente el tamaño del mapa de características para volver a las dimensiones originales de la imagen. Al reflejar la arquitectura del codificador, el decodificador aprende a revertir el proceso de compresión, reconstruyendo la imagen a partir de la representación latente comprimida.

3. Bottleneck y Compresión Latente
El bottleneck es la parte más crítica del autoencoder para la compresión. Después de la última capa convolucional, los datos siguen siendo un tensor 3D (16x16x256). Se utilizó una capa de aplanamiento (nn.Flatten) para convertir este tensor en un vector 1D y luego una capa completamente conectada (nn.Linear) para comprimirlo aún más en una latent_dim mucho más pequeña de 128. Esto obliga al modelo a aprender las características de alto nivel más esenciales de la imagen, ya que debe descartar una cantidad significativa de datos para que quepan en este pequeño espacio latente. Este paso es donde se produce la compresión principal.

4. Funciones de Activación
ReLU (nn.ReLU): Las funciones de activación de Unidad Lineal Rectificada se utilizan en las capas ocultas tanto del codificador como del decodificador. ReLU es una opción estándar porque es computacionalmente eficiente y ayuda a prevenir el problema del desvanecimiento del gradiente durante el entrenamiento.

La capa final del decodificador utiliza una función de activación sigmoide. Dado que los valores de píxeles de su imagen se normalizan a un rango de [0, 1] antes de ser alimentados a la red, la función sigmoide asegura que los valores de píxeles de la imagen reconstruida también estén restringidos a este rango. Esto garantiza que la salida sea una representación de imagen válida.


Aquí podemos ver el diagrama de la función de pérdida del autoencoder durante el entrenamiento:
![Autoencoder estándar](autoencoder_results/training_loss.png)



La ventaja principal de esta arquitectura es su simplicidad computacional y de implementación. Además, ofrece una cierta flexibilidad para ajustar la tasa de compresión modificando la dimensión latente. Sin embargo, una limitación es que los autoencoders estándar pueden tener dificultades para capturar la variabilidad completa en datos complejos, lo que puede resultar en reconstrucciones borrosas o faltantes. A pesar de esto, para el propósito de la compresión híbrida con residuos comprimidos, esta arquitectura proporciona un buen equilibrio entre eficiencia y calidad. Aquí podemos ver una imágen de ejemplo del conjunto de datos original y su reconstrucción a través del autoencoder:
![](autoencoder_results/examples/example_001_r062cab38t.png)

Como se puede observar, el autoencoder logra capturar las características principales de la imagen, como la luminosidad y los colores, pero pierde muchos detalles, lo que es esperado dado el alto nivel de compresión. El residuo, que contiene estos detalles, es luego comprimido por separado para mejorar la calidad final de la imagen reconstruida.

Aquí tenemos la misma imagen de ejemplo, pero esta vez con la compresión híbrida completa:
![](auto_encoder_hybrid_results/examples/hybrid_example_02.png)

A seguir, tenemos los boxplots de las métricas de evaluación de la compresión híbrida:
![Metrícas de compresión híbrida](boxplots/auto_encoder_hybrid_boxplots.png)

Como se puede observar, la compresión híbrida con el autoencoder estándar parece lograr una pequeña mejora en las métricas de calidad (PSNR y SSIM) en comparación con JPEG solo, aunque alcance tasas de compresión mucho menores.

### Autoencoder Variacional (VAE) con residuos comprimidos a través de zstd

El VAE es un modelo generativo que introduce un elemento probabilístico en el autoencoder estándar. A diferencia de un autoencoder estándar, el codificador del VAE no produce un solo vector latente. En cambio, produce dos vectores: una **media ($\mu$)** y una **desviación estándar ($\sigma$)** para cada dimensión del espacio latente. Esto define una distribución de probabilidad (típicamente una distribución normal) a partir de la cual se muestrea un vector latente. Esta regularización del espacio latente obliga al VAE a aprender una representación de los datos continua y bien estructurada, lo que hace que el espacio latente sea más organizado y fácil de trabajar. Luego, el decodificador reconstruye la imagen a partir de este vector latente muestreado. El proceso híbrido es idéntico al del método del autoencoder estándar: la diferencia entre la imagen original y la reconstruida por el VAE se calcula como el residuo, que luego se cuantiza a 4 bits y se comprime sin pérdidas con Zstandard. 

Aquí podemos ver el diagrama de la arquitectura del VAE:
![VAE](vae_architecture.png)

La arquitectura es una extensión lógica del autoencoder simple que discutimos, pero con componentes clave para la funcionalidad probabilística del VAE.

1. El Codificador con Ramificación (Bottleneck)
La arquitectura del codificador es casi idéntica a la del autoencoder simple, utilizando capas convolucionales para reducir las dimensiones espaciales y aumentar los canales. Sin embargo, hay una importante diferencia en el bottleneck: en lugar de una única capa lineal que comprime el tensor a un solo vector latente, este VAE utiliza dos capas lineales separadas: una para la media, y una para la log-varianza. La razón por la que se utiliza el logaritmo de la varianza en lugar de la varianza directamente es para garantizar que la varianza sea siempre positiva, ya que la varianza no puede ser negativa. Esta ramificación en el cuello de botella es lo que transforma un autoencoder simple en un VAE, permitiéndole aprender una distribución en lugar de un único punto latente.

2. Muestreo
El muestreo es el corazón del VAE: en lugar de simplemente muestrear una variable aleatoria de la distribución, la técnica de muestreo crea un vector latente $z$ de la siguiente manera: $z=μ+ϵ⋅σ$
Donde ϵ (eps) es un vector muestreado aleatoriamente de una distribución normal estándar (media 0, varianza 1). Esto permite que el flujo de gradientes pase a través de la operación de muestreo, lo cual es necesario para optimizar la red durante el entrenamiento. Este diseño hace que el VAE sea capaz de aprender la estructura del espacio latente y no solo una representación determinista, lo que teoricamente conduce a una mejor generalización.

3. El Decodificador
El decodificador es casi idéntico al del autoencoder simple. Su función es tomar el vector latente muestreado (z) y reconstruir la imagen. Al reflejar la estructura del codificador (utilizando capas convolucionales transpuestas), aprende a expandir la representación comprimida de vuelta a las dimensiones originales. El uso de la función de activación sigmoide en la última capa asegura que los valores de los píxeles de la imagen reconstruida se mantengan en el rango [0, 1].

La ventaja principal de esta arquitectura es su capacidad para aprender una representación latente más estructurada y continua de los datos, lo cual hace que el VAE sea más efectivo en generalizar a imágenes que no ha visto en la fase de entrenamiento. Sin embargo, los VAE pueden ser más difíciles de entrenar debido a la naturaleza probabilística del muestreo y la necesidad de equilibrar la reconstrucción con la regularización del espacio latente. Además, la calidad de las reconstrucciones puede ser ligeramente inferior a la de los autoencoders simples debido a la naturaleza estocástica del muestreo. 

Aquí tenemos siempre la misma imagen de ejemplo del conjunto de datos original y su reconstrucción a través del VAE:
![](vae_results/examples/example_001_r062cab38t.png)

Y aquí la misma imagen de ejemplo, pero esta vez con la compresión híbrida completa:
![](vae_hybrid_results/examples/hybrid_example_02.png)

Estos son los boxplots de las métricas de evaluación de la compresión híbrida con el VAE:
![Metrícas de compresión híbrida con VAE](boxplots/vae_hybrid_boxplots.png)

Como se puede observar, los resultados son muy similares a los del autoencoder estándar, con una ligera mejora en las métricas de calidad (PSNR y SSIM) en comparación con JPEG solo, aunque alcanzando tasas de compresión mucho menores.

### 4. Conclusiones
Este estudio exploró la viabilidad de un enfoque de compresión de imágenes híbrido, comparando dos arquitecturas de redes neuronales (un autoencoder estándar y un autoencoder variacional) contra un método de compresión JPEG puro. Los resultados del análisis descriptivo del conjunto de datos RAISE revelaron una dualidad: las imágenes eran tonalmente complejas (entropía alta) pero estructuralmente simples (nitidez baja), una combinación que presenta un desafío único para la compresión. El análisis comparativo de los boxplots de PSNR, SSIM y tasa de compresión mostró que, si bien las metodologías híbridas, tanto la basada en el autoencoder estándar como la basada en el VAE, lograron una calidad de imagen ligeramente superior (PSNR y SSIM más altos), lo hicieron a expensas de tasas de compresión significativamente menores en comparación con JPEG-90. A pesar de que la hipótesis de que el VAE obtendría una mejor relación de compresión general no se demostró, sus resultados fueron comparables a los del autoencoder estándar, confirmando que ambos enfoques aprenden representaciones de la capa base de manera eficiente. En conclusión, si bien los métodos híbridos con autoencoders ofrecen una calidad de reconstrucción marginalmente mejor, JPEG-90 sigue siendo un punto de referencia superior en cuanto a la eficiencia de la compresión.

### 5. Posibles desarrollos futuros

Hay varias direcciones prometedoras para futuras investigaciones en el ámbito de la compresión híbrida de imágenes utilizando redes neuronales:

* Cuantización Adaptativa: En lugar de una cuantización fija de 4 bits, el modelo podría aprender a aplicar un número variable de bits a diferentes partes de la imagen residual. Por ejemplo, podría asignar más bits a los bordes afilados y texturas detalladas (donde el ojo humano es más sensible a la pérdida) y menos bits a las áreas suaves y uniformes. Esto mejoraría tanto la calidad percibida como la eficiencia de la compresión.

* Optimización Tasa-Distorsión: Un enfoque más avanzado sería entrenar todo el sistema híbrido de principio a fin para optimizar tanto la tasa (BPP) como la distorsión (PSNR/SSIM). Esto podría implicar agregar un término de pérdida de tasa-distorsión al objetivo de entrenamiento, lo que permitiría al modelo encontrar el equilibrio óptimo entre la calidad y el tamaño del archivo.

* Generative Adversarial Networks  (GANs): Se podría utilizar una GAN en lugar del autoencoder. El generador de una GAN actuaría como el codificador-decodificador, y un discriminador sería entrenado para distinguir entre imágenes originales y reconstruidas. Esto obligaría al generador a producir reconstrucciones que sean perceptualmente muy similares a la original, lo que podría generar mejores puntuaciones SSIM.

### 6. Referencias
[1] D. P. Kingma and M. Welling, "Auto-Encoding Variational Bayes" in International Conference on Learning Representations (ICLR), 2014.

[2] G. E. Hinton and R. R. Salakhutdinov, "Reducing the Dimensionality of Data with Neural Networks" Science, vol. 313, no. 5786, pp. 504-507, 2006.

[3] G. K. Wallace, "The JPEG still picture compression standard," in IEEE Transactions on Consumer Electronics, vol. 38, no. 1, pp. xviii-xxxiv, Feb. 1992, doi: 10.1109/30.125072.
keywords: {Transform coding;Image coding;Digital images;Image storage;Standards development;ISO standards;Gray-scale;Displays;Costs;Facsimile},

[4] Y. Collet and M. Kucherawy. 2021. RFC 8878: Zstandard Compression and the 'application/zstd' Media Type. RFC Editor, USA.

[5] "D.-T. Dang-Nguyen, C. Pasquini, V. Conotter, G. Boato, RAISE – A Raw Images Dataset for Digital Image Forensics, ACM Multimedia Systems, Portland, Oregon, March 18-20, 2015". 

### 7. Anexos
El código completo del proyecto está disponible en este repositorio de GitHub: https://github.com/lootag/trabajo-fin-de-master-ucm.