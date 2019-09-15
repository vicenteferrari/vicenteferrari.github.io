---
layout: post
date: 2018-12-21
abstract: |
    Haremos el tamaño de archivos de audio más pequeños usando métodos
    numéricos para implementar algoritmos de compresión.
author:
- Nelson Curihual
- Vicente Ferrari
- Sebastián Opazo
- Sebastián Pardo
title: 'Compresión de audio con perdida.'
---

Introducción
============

Hoy en día la mayoría de los datos son guardados en medios digitales y
es de gran interés para las empresas informáticas reducir la cantidad de
gasto en almacenamiento y transmisión de la información (ancho de
banda). En este problema de compresión de datos hay dos grandes
paradigmas:

-   La compresión sin pérdida, o lossless compression, en inglés, reduce
    el tamaño de un archivo digital reduciendo su entropía. Es decir la
    redundancia de datos. Esto básicamente significa traducir patrones
    repetidos en la información por unos patrones igual de reconocibles
    pero de menor tamaño. Permite que la información comprimida sea
    descomprimida sin perdida en la fidelidad de la información.

-   La compresión con pérdida, o lossy compression, va un paso más allá
    que la sin pérdida. Esta depende drásticamente en el dominio
    problema. La compresión para audio es distinta la de imagenes, etc.
    En el caso de audio, se aprovechan las imperfecciones del aparato
    auditivo humano para transformar datos indeseados a 'patrones nulos'
    que bajan la entropía de los datos y son trivialmente comprimidos
    con los métodos lossless tradicionales.

Física del sonido
-----------------

El sonido es una serie de oscilaciones en la presión del aire. Este
sonido origina de todo tipo de perturbaciones en los materiales, por
ejemplo, los golpes fuertes entre objetos producen mucho movimiento
interno en las partículas que forman el objeto, estas rápidamente
comienzan a vibrar en sus frecuencias de resonancia natural y el sonido
se propaga por el material y por medios adyacentes. Otro ejemplo es el
de un violín donde la fricción entre el arco y las cuerdas del
instrumento hacen que estas cuerdas vibren a las frecuencias naturales
del material y a sus frecuencias de resonancia. Luego cuando estos
objetos vibrantes chocan con el aire alrededor de ellos, transmiten la
energía mecánica a la misma frecuencia. Y es así como se transmite el
sonido.

Para medir esto, utilizamos aparatos con membranas delgadas que resuenan
con las ondas de sonido y producen un voltaje que grabamos con respecto
al tiempo. Para convertir una señal análoga y continua de presión de
aire a una secuencia de datos digitales tenemos que tomar muestras
periódicas del voltaje de nuestros aparatos medidores (micrófonos). Como
se observa en la Figura 1.

![Ejemplo de muestreo de una señal análoga](/assets/compresion/sampling.png)

Figura 1: Ejemplo de muestreo de una señal análoga.

Luego, lo más común es querer volver reproducir los datos de manera
física a través de un parlante, por ejemplo.

El primer problema que se nos ocurre es el de cuantas muestras hay que
tomar para poder imitar la señal original lo más fielmente posible. Es
muy posible, submuestrear la señal original, y reconstruir una señal
completamente distinta a la original. Como se observa en la Figura
2.

![Ejemplo de submuestreo de una señal análoga.](/assets/compresion/undersample.png)

Figura 2: Ejemplo de submuestreo de una señal análoga.

Para esto nos volvemos al teorema de muestreo de Nyquist-Shannon cuya
conclusión es:

Dado una señal cuya frecuencia presente más alta es $n$, se necesitan
tomar muestras por lo menos $2n$ veces por segundo para poder
caracterizar completamente la señal original.

Psicoacústica
-------------

Las ondas de sonido son muy variadas. Los objetos resuenan naturalmente
a distintas frecuencias y a sus frecuencias de resonancia.

El primer límite de nuestro aparato auditivo es que en promedio no
logramos sentir frecuencias más bajas que 20 Hz o más altas de 22 kHz.

Cuando estas ondas llegan a nuestros oídos resuenan con una membrana
llamada cóclea, esta comienza a vibrar en distintos lugares dependiendo
de la frecuencia del sonido que está llegando. Al vibrar, la cóclea
estimula nervios que la rodean y así se manda la información correcta al
cerebro.

Una de las falencias de la cóclea es que al vibrar en cierto lugar a
grandes amplitudes, las partes adyacentes se transforman menos
sensibles. Por ejemplo, si llega un ruido y este tiene una amplitud alta
en la frecuencia 4,5 kHz, y al mismo tiempo está vibrando en el lugar
adyacente a 4,6 kHz casi no la percibiremos.

Otro problema de nuestra percepción es que para diferentes frecuencias
tenemos diferentes umbrales de activación, estos se ven graficados en la
Figura 3.

![Umbral de sonido de los oídos.](/assets/compresion/hearing.png)

Figura 3: Umbral de sonido de los oídos.

Sonidos más bajos que el umbral para esa frecuencia no son percibidos
por el usuario.

Todo esto lo tenemos que tomar en consideración al momento de formar un
modelo psicoacústico que nos permita eliminar información innecesaria de
nuestros datos de audio.[1]

Archivos de audio digital
-------------------------

Cuando se tiene un micrófono (u otro aparato para medir sonido) que
produce una señal continua de voltaje y ocupamos un artefacto
electrónico para medir la amplitud de este voltaje en muestras discretas
podemos guardar estos datos en archivos digitales dentro del
almacenamiento de dispositivos electrónicos como computadores,
reproductores MP3, etc.

Estos archivos de audio tienen 3 características que nos importan:

-   La frecuencia de muestreo es tal vez la métrica más importante de un
    archivo de audio. Como mencionamos anteriormente, para caracterizar
    una señal completamente la frecuencia de muestreo debe ser al menos
    el doble de la frecuencia presente más alta en la señal. Ya que los
    humanos en promedio no escuchamos más allá de 20 kHz, varias
    estándares internacionales se han formulado siendo una frecuencia de
    muestreo de 44.1 kHz la más común.

-   Otro parámetro con un rol importante es la profundidad de bits. Es
    decir cuantos bits ocupamos para cada muestra. Esto no nos indica
    que tan alta puede llegar la señal sino nos indica cuanta resolución
    tenemos disponible, por ejemplo, ocupando solo 4 bits tendríamos
    $2^4$ o 16 pasos distintos de volumen, siendo -7 el más bajo y 8 el
    más alto. En cambio con una profundidad de $2^{16}$ tenemos 65536
    distintos valores posibles, con -32767 como el valor más bajo y
    32768 el más alto.

    En la práctica los profesionales de sonido normalmente ocupan una
    profundidad de 32 bits para grabar sonido en estudios, etc.

-   Al tenerlos dos datos anteriores un parámetro emergente es la tasa
    de bits, es decir a cuantos bits por segundo estamos almacenando o
    transmitiendo nuestra señal digital. Si tenemos una frecuencia de
    muestreo de 44.1 kHz y una profundidad de 16 bits, utilizaremos
    705600 bits por segundo de información o 705 kbps. Si tomamos en
    consideración que los módems antiguos solo podían transmitir a 128
    kbps, nos podemos dar cuenta lo importante que fue desarrollar
    métodos de compresión.

Modelo psicoacústico
====================

Tomando en consideración todo lo anteriormente explicado podemos
proceder a explicar modelos psicoacústicos que nos permiten determinar
que información es indeseada en nuestros datos.

Dependiendo de los distintos casos de uso que podríamos tener se ocupan
distintos modelos. Vamos a explicar algunos a continuación:

El más simple será una simple detección de 'silencio', cuando
determinamos que la información presente en un determinado instante
temporal no será escuchada por el usuario simplemente la eliminamos.

Podemos también utilizar modelos de predicción lineal, estos ajustan las
curvas originales a modelos de habla preestablecidos que permite
transmitir a muy bajas tasas de bits, como se muestra en la Figura 4.

![Ejemplo de un modelo de predicción lineal.](/assets/compresion/LPC.png)

Figura 4: Ejemplo de un modelo de predicción lineal. Señal azul es la original,
señal roja es el ajuste.

Modelos más sofisticados se aprovechan de los defectos auditivos de las
personas y los distintos tipos de enmascaramiento sonoro.

Enmascaramiento de frecuencia, es cuando una componente del sonido (a
una frecuencia particular) enmascara el las componentes vecinas.

Enmascaramiento temporal es cuando dos tonos tocados juntos en el
tiempo, uno puede hacer que el otro sea difícil de escuchar. Si
escuchamos un sonido fuerte y luego este cesa, nos toma un pequeño
tiempo en poder volver a escuchar tonos suaves.

El estándar dorado de compresión de audio con perdida son los métodos de
codificación desarrollados por el Moving Pictures Experts Group (MPEG),
de estos el más común es el MPEG-2 Audio Layer III o MP3, este método de
compresión implementa todo lo explicado anteriormente.[2]

Métodos numéricos
=================

En este texto implementaremos un modelo de compresión con perdida
rudimentario. Sale del alcance de este informe desarrollar modelos
psicoacústicos sofisticados ya que es un trabajo enorme. Nos
concentraremos en la solución numérica del problema matemático central.

El proceso de describe a continuación.

Todo comienza con una señal discreta de audio normalizada, representada
en MatLab por un vector. Estos son arreglos de datos extensos ya que
normalmente muestreamos a 44.1 kHz. Por ejemplo, 10 segundos de audio
serían 440.000 datos.

A continuación dividimos nuestra señal en varias 'ventanas' o en
distintos vectores en MatLab. Esto es para limitar el error generado por
la transformada. La cantidad de ventanas dependerá de cuanto queremos
comprimir el archivo.

Luego, usando la transformada discreta de coseno (DCT-I), una
transformada derivada de la transformada de Fourier, podemos pasar las
ventanas del dominio temporal al dominio frecuencial. El resultado de la
transformada es la representación de la señal original hecha como una
suma de distintos cosenos y los coeficientes de estos. Al tener los
coeficientes de la transformada podemos volver a la secuencia original
con la transformada discreta de coseno inversa.

Entonces, después de usar la transformada, los coeficientes pequeños
resultan en cosenos pequeños, los cuales son menos probable de ser
escuchados, en vez de almacenar estos, los eliminamos y guardamos el
resto. Haciendo esto almacenamos menos datos y bajamos el tamaño del
archivo.

El algoritmo de descompresión será simple, simplemente tomamos la
transformada inversa de lo que guardamos anteriormente y reproducimos
eso. Nos faltará algo de la señal original pero una de las propiedades
de la DCT es que unos pocos de los coeficientes altos son responsables
por la mayoría de la potencia de la señal original.[3]

Para poder lograr esto, necesitaremos calcular la transformada discreta
de coseno numéricamente en MatLab. Esto requiere poder solucionar
numéricamente la transformada de Fourier en MatLab. Este método se
observa en el archivo adjunto dft.m.

Resultados
==========

Adjunto a este informe va un archivo de audio crudo de formato .wav
llamado audio.wav que ocuparemos para demostrar el algoritmo. El archivo
original tiene una frecuencia de muestreo de 44.1 kHz, una profundidad
de 16 bits, una tasa de bits de 1411 kbps (ya que es audio estéreo, osea
tiene dos canales, uno izquierdo y uno derecho) y dura 23 segundos.
Dando un tamaño total de 3.86 MB.

Primero comprimimos el archivo con ventanas de 4096 datos, manteniendo
1500 coeficientes y los 16 bits de profundidad. Al algoritmo nos
devuelve un $strcut$ de MatLab con todos los datos necesarios para
reconstruir el audio. MatLab nos deja guardar este archivo y pesa tan
solo 1.1 MB para un coeficiente de compresión de 3.45. Luego usamos la
descompresión para transformar estos datos (coeficientes de cosenos)
devuelta a señales y lo podemos escuchar.

Luego hacemos lo mismo con distintos parámetros, ventanas más chicas y
manteniendo menos coeficientes. Otorgando los resultado en la Figura 5.

![Grafico con los tamaños resultante de los archivos.](/assets/compresion/sizes.png)

Figura 5: Grafico con los tamaños resultante de los archivos.

Comparación
===========

El método elegido para este informe claramente no es el mejor
disponible. Este ámbito ha sido sujeto de gran estudio por décadas y por
buen motivo, llegar a buenos resultados que entreguen un archivo pequeño
manteniendo la mejor calidad posible es difícil. Sin embargo utilizando
modelos simples como el presentado en este informe se pueden obtener
resultados muy satisfactorios. No obstante intentaremos comparar nuestro
método con otros codificadores estándar tales como FLAC[4]
(Lossless), LAME[5] (MP3) y Vorbis[5]. Cabe mencionar que
entre distintos codificadores con perdida los métodos numéricos
utilizados son prácticamente los mismos, lo que cambia bastante entre
estos son los modelos psicoacústicos que determinan que datos son y no
son necesarios.

En la Figura 6 podemos ver comparaciones subjetivas de calidad
entre estos codificadores.

![Eje vertical indica calidad, eje horizontal tamaño en MB.](/assets/compresion/chart.png)

Figura 6: Eje vertical indica calidad, eje horizontal tamaño en MB.

Como podemos ver, el archivo original tiene excelente calidad pero el
archivo es muy grande, la codificación FLAC al ser lossless otorga una
compresión moderada sin perder nada de calidad. Por otro lado los codecs
con perdida como MP3 o Vorbis entregan muy buena calidad a casi un
décimo del tamaño original. Nuestro método en cambio otorga mala calidad
con una compresión moderada.

Conclusión
==========

Utilizando la transformada discreta de coseno podemos eliminar
frecuencias indeseadas de nuestros datos de audio originales logrando
tamaños de archivos mucho menores. La mayoría del esfuerzo para lograr
buenos resultados apuntan al desarrollo de un buen modelo psicoacústico
que aproveche al máximo las falencias del sistema auditivo humano. Aun
así con modelos rudimentarios se pueden lograr resultados decentes.


[1] The Sense of Hearing, Christopher J. Plack, Lawrence Erlbaum Associates,
2005

[2] The audio/mpeg Media Type, M. Nilsson,
<https://tools.ietf.org/html/rfc3003>

[3] Data Compression, The Complete Reference, Forth Edition, David Salomon,
2007

[4] FLAC, Free Lossless Audio Codec, <https://xiph.org/flac/>

[5] LAME, Lame Ain't an Mp3 Encoder, <http://lame.sourceforge.net/>

[6] Vorbis, <https://xiph.org/vorbis/>
