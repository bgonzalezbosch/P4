PAV - P4: reconocimiento y verificación del locutor
===================================================

Obtenga su copia del repositorio de la práctica accediendo a [Práctica 4](https://github.com/albino-pav/P4)
y pulsando sobre el botón `Fork` situado en la esquina superior derecha. A continuación, siga las
instrucciones de la [Práctica 2](https://github.com/albino-pav/P2) para crear una rama con el apellido de
los integrantes del grupo de prácticas, dar de alta al resto de integrantes como colaboradores del proyecto
y crear la copias locales del repositorio.

<<<<<<< HEAD
<<<<<<< HEAD
También debe descomprimir, en el directorio `PAV/P4`, el fichero [db_8mu.tgz](https://atenea.upc.edu/mod/resource/view.php?id=3508877?forcedownload=1)
=======
También debe descomprimir, en el directorio `PAV/P4`, el fichero [db_8mu.tgz](https://atenea.upc.edu/pluginfile.php/3145524/mod_assign/introattachment/0/spk_8mu.tgz?forcedownload=1)
>>>>>>> Ficheros iniciales de P4
=======
También debe descomprimir, en el directorio `PAV/P4`, el fichero [db_8mu.tgz](https://atenea.upc.edu/mod/resource/view.php?id=3508877?forcedownload=1)
>>>>>>> f8f86880296c2b221da50c6c2b028c2d4c10a90c
con la base de datos oral que se utilizará en la parte experimental de la práctica.

Como entrega deberá realizar un *pull request* con el contenido de su copia del repositorio. Recuerde
que los ficheros entregados deberán estar en condiciones de ser ejecutados con sólo ejecutar:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh
  make release
  run_spkid mfcc train test classerr verify verifyerr
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Recuerde que, además de los trabajos indicados en esta parte básica, también deberá realizar un proyecto
de ampliación, del cual deberá subir una memoria explicativa a Atenea y los ficheros correspondientes al
repositorio de la práctica.

A modo de memoria de la parte básica, complete, en este mismo documento y usando el formato *markdown*, los
ejercicios indicados.

## Ejercicios.

### SPTK, Sox y los scripts de extracción de características.

- Analice el script `wav2lp.sh` y explique la misión de los distintos comandos involucrados en el *pipeline*
  principal (`sox`, `$X2X`, `$FRAME`, `$WINDOW` y `$LPC`). Explique el significado de cada una de las 
  opciones empleadas y de sus valores.

<<<<<<< HEAD
<<<<<<< HEAD
=======
>>>>>>> f8f86880296c2b221da50c6c2b028c2d4c10a90c
  * `sox` permite realizar la conversión de una señal de entrada sin cabecera a una del fromato adecuado. Además, también permite la conversión de señales guradadas en un programa externo. Al fiechero de entrada se le pueden aplicar las siguientes opciones:
    - `-t`: Tipo de fichero de audio.
    - `e`: Indica la codificación que se quiere aplicar (signed-integer, unsigned-integer, floating-point, mu-law, a-law, ima-adpcm, ms-adpcm, gsm-full-rate). 
    - `b`: Indica el numero de bits.
    - `-`: Redirección del output (pipeline)

  * `$X2X` es un programa de SPTK que permite la conversión entre distintos formatos de datos. Por ejemplo pasar de un formato short (2 bytes9 a un formato float (4 bytes)
  
  * `$FRAME` divide la señal de entrada en tramas de `l` muestras con desplazamiento de ventana de `p` muestras. También puedes se puede elegir el punto de comienzo entre centrado o no centrado.
  
    - `-l`: Indica el número de muestras de cada trama. Su valor máximo es de 256.
    - `-p`: Indica el número de muestras de desplazamineto. Su valor máximo es de 100.

  * `$WINDOW` pondera cada trama por una ventana. Por defecto esta es la Blackman. Hay 6 tipos de ventanas.
    - `-l`: Tamaño de la ventana de entrada. Su valor máximo es 256.
    - `-L`: Tamaño de la ventana de salida. Su valor máximo es 256.
    
  * `$LPC` realiza un análisis de la señal, usando el métodode Levinson-Durbin. Calcula los coeficientes LPC de cada trama enventanada del fichero de entrada. Tanto la señal de entrada como la señal de salida tienen un formato float
    - `-l`: Tamaño de la ventana de entrada. Máximo 256.
    - `-m`: Númerode coeficientes LPC. Cmomo mucho 25.
    - `-f`: Valor mínimo del determinante. Como mucho 10^-6.
  
- Explique el procedimiento seguido para obtener un fichero de formato *fmatrix* a partir de los ficheros de
  salida de SPTK (líneas 45 a 47 del script `wav2lp.sh`).
  
  En las filas de la 45 a la 47 procedemos a calcular el número de columnas y filas.
  Las columnas las calculamos con el orden del predictor y les sumamos 1 debidoa auqe en el primer elemento del vector de prediccioón se almacena la ganancia de dicho predictor.
  Para determinar el número de filas, hemos de tener en cuenta la longitud de la señal y la lomgitud y desplazamiento de ventana que le aplicamos a dicha señal. Con el comando `sox` realizamos la conversión de datos del tipo float al tipo ascii y a continuación, contamos las líneas con el comando wc-1.
  Una vez creada la matriz, la imprimimos.

  * ¿Por qué es conveniente usar este formato (u otro parecido)? Tenga en cuenta cuál es el formato de
    entrada y cuál es el de resultado.
    
    permite identificar de n modo fácil, sencillo y correcto los coeficientes por trama. De este modo tenemos disponemos de los coeficientes en diferentes columnas y podemos mostrarlos y elegir los coeficientes 2 y 3 (que son los que se nos pide)

- Escriba el *pipeline* principal usado para calcular los coeficientes cepstrales de predicción lineal
  (LPCC) en su fichero <code>scripts/wav2lpcc.sh</code>:
  
  Main command for feature extration
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh
  sox $inputfile -t raw -e signed -b 16 - | $X2X +sf | $FRAME -l 240 -p 80 | $WINDOW -l 240 -L 240 |
	$LPC -l 240 -m $lpc_order | $LPCC -m $lpc_order -M $lpcc_order > $base.lpcc
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Escriba el *pipeline* principal usado para calcular los coeficientes cepstrales en escala Mel (MFCC) en su
  fichero <code>scripts/wav2mfcc.sh</code>:
  
  Main command for feature extration
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh
  sox $inputfile -t raw -e signed -b 16 - | $X2X +sf | $FRAME -l 180 -p 100 | $WINDOW -l 180 -L 180 |
	$MFCC -s $fm -l 180 -m $mfcc_order -n $melbank_order > $base.mfcc
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
<<<<<<< HEAD
=======
- Explique el procedimiento seguido para obtener un fichero de formato *fmatrix* a partir de los ficheros de
  salida de SPTK (líneas 45 a 47 del script `wav2lp.sh`).

  * ¿Por qué es conveniente usar este formato (u otro parecido)? Tenga en cuenta cuál es el formato de
    entrada y cuál es el de resultado.

- Escriba el *pipeline* principal usado para calcular los coeficientes cepstrales de predicción lineal
  (LPCC) en su fichero <code>scripts/wav2lpcc.sh</code>:

- Escriba el *pipeline* principal usado para calcular los coeficientes cepstrales en escala Mel (MFCC) en su
  fichero <code>scripts/wav2mfcc.sh</code>:
>>>>>>> Ficheros iniciales de P4
=======
>>>>>>> f8f86880296c2b221da50c6c2b028c2d4c10a90c

### Extracción de características.

- Inserte una imagen mostrando la dependencia entre los coeficientes 2 y 3 de las tres parametrizaciones
  para todas las señales de un locutor.
  
  + Indique **todas** las órdenes necesarias para obtener las gráficas a partir de las señales 
    parametrizadas.
<<<<<<< HEAD
<<<<<<< HEAD
=======
>>>>>>> f8f86880296c2b221da50c6c2b028c2d4c10a90c
    
    - Orden para obtener el fichero de texto LP:
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh
      fmatrix_show work/lp/BLOCK01/SES017/*.lp | egrep '^\[' | cut -f4,5 > lp_2_3.txt
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

    - Orden para obtener el fichero de texto LPCC:
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh
      fmatrix_show work/lpcc/BLOCK01/SES017/*.lpcc | egrep '^\[' | cut -f4,5 > lpcc_2_3.txt
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

    - Orden para obtener el fichero de texto MFCC:
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh
      fmatrix_show work/mfcc/BLOCK01/SES017/*.mfcc | egrep '^\[' | cut -f4,5 > mfcc_2_3.txt
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    
    A partir de los fichers generados, representamos las gráficas a aprtir de siguente código python:
    
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh
    import matplotlib.pyplot as plt

    # LP
    X, Y = [], []
    for line in open('lp_2_3.txt', 'r'):
      values = [float(s) for s in line.split()]
      X.append(values[0])
      Y.append(values[1])
    plt.figure(1)
    plt.plot(X, Y, 'rx', markersize=4)
    plt.savefig('lp_2_3.png')
    plt.title('LP',fontsize=20)
    plt.grid()
    plt.xlabel('a(2)')
    plt.ylabel('a(3)')
    plt.savefig('lp_2_3.png')
    plt.show()

    # LPCC
    X, Y = [], []
    for line in open('lpcc_2_3.txt', 'r'):
      values = [float(s) for s in line.split()]
      X.append(values[0])
      Y.append(values[1])
    plt.figure(2)
    plt.plot(X, Y, 'rx', markersize=4)
    plt.savefig('lpcc_2_3.png')
    plt.title('LPCC',fontsize=20)
    plt.grid()
    plt.xlabel('c(2)')
    plt.ylabel('c(3)')
    plt.savefig('lpcc_2_3.png')
    plt.show()

    # MFCC
    X, Y = [], []
    for line in open('mfcc_2_3.txt', 'r'):
      values = [float(s) for s in line.split()]
      X.append(values[0])
      Y.append(values[1])
    plt.figure(3)
    plt.plot(X, Y, 'rx', markersize=4)
    plt.savefig('mfcc_2_3.png')
    plt.title('MFCC',fontsize=20)
    plt.grid()
    plt.xlabel('mc(2)')
    plt.ylabel('mc(3)')
    plt.savefig('mfcc_2_3.png')
    plt.show()
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    
    <img width="640" alt="lp" src="https://github.com/bgonzalezbosch/P4/blob/Bernat-Roger/lp.png"> 
    <img width="640" alt="lpcc" src="https://github.com/bgonzalezbosch/P4/blob/Bernat-Roger/lpcc.png"> 
    <img width="640" alt="mfcc" src="https://github.com/bgonzalezbosch/P4/blob/Bernat-Roger/mfcc.png"> 
 
  + ¿Cuál de ellas le parece que contiene más información?
  
  Para saber cual contiene más información, debemos comparar el nivel de correlación entre coeficientes próximos. Si la incorrelación es alta, hay más información. En la representación de los coeficientes 2 y 3 se observan diferencias entre dichos coeficientes, sobretodo en el caso de LP.
  
  Si miramos cada gráfica vemos lo siguente:
  
  - LP: obtenemos poca información debido a que se ve una marcada correlación lineal entre los coeficientes
  
  - LPCC y MFCC: Ambas son parecidas. Los puntos están distribuidos en el espacio en orma de elipse. En el caso de MFCC hay un rango mayor de valores con lo que los coeficientes serán más incorrelados.

- Usando el programa <code>pearson</code>, obtenga los coeficientes de correlación normalizada entre los
  parámetros 2 y 3 para un locutor, y rellene la tabla siguiente con los valores obtenidos.
  
  Para obtener los valores de &rho;<sub>x</sub> hemos ejecutado estos comandos ene el terminal:
  
    - LP:
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh
    pearson work/lp/BLOCK01/SES017/*.lp
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    - LPCC:
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh
    pearson work/lpcc/BLOCK01/SES017/*.lpcc
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    - MFCC:
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh
    pearson work/mfcc/BLOCK01/SES017/*.mfcc
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    
  El resultado de dichas ejecuciones es el siguiente:
    
  |                        | LP      | LPCC    | MFCC    |
  |------------------------|:-------:|:-------:|:-------:|
  | &rho;<sub>x</sub>[2,3] |-0.872284|0.148411 |-0.175672|
  
  + Compare los resultados de <code>pearson</code> con los obtenidos gráficamente.
  
  Primero debemos comprender el significado de los valores de &rho;<sub>x</sub>[2,3]. Cuanto más cercano sea a 1 valor absoluto quiere decir que los dos coeficientes que comparamos están muy correlados entre sí. Siguiendo esto, observamos como en LP el valor está muy cerca del -1 com lo que podemos afirmar que los parámetros son correlados o, dicho de otra forma, que la información que nos proporcionan los dos parámetros es la misma que la que proporciona un solo.
  
  De esta misma manera, LPCC y MFCC distan mucho del 1, por tanto podemos decir que sus parámetros son incorrelados entre sí. La información que nos proporcionan ambos parámetros es distinta a la que nos proporciona uno solo.
  
  Si nos fijamos en las gráficas y lo que emos comentado previamente sobre ellas, vemos que coinciden. en ambos casos LP son correlados y LPCC y MFCC son incorrelados siendo LPCC más incorrelado (más cercano al cero) que MFCC
  
- Según la teoría, ¿qué parámetros considera adecuados para el cálculo de los coeficientes LPCC y MFCC?

  Partiendo de LP, el orden que se usa y hemos usado es 8 (lp_order = 8). En el caso de LPCC, hemos definido el número de cepstrum en 13 coeficientes después de hacer pruebas con valores rondando entre 2*lp_order y 3*lp_order/2. Para el caso MFCC, se usan os 13 primeros coeficientes*1.25 = 16 y el número de filtros suele ir de 24 a 40. En nuestro caso 24.

  - LPCC: línea 108 de run_spkid.sh (compute_lpcc())
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh
  EXEC="wav2lpcc 8 13  $db_sen1/$filename.wav $w/$FEAT/$filename.$FEAT" #orden LPC = lp y orden cepstrum
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  - MFCC: línea 118 de run_spkid.sh (compute_mfcc())
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh
  EXEC="wav2mfcc 8 16 24 $db_sen2/$filename.wav $w/$FEAT/$filename.$FEAT"  #orden MFCC = lp, orden del MFCC y num de filtros
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


<<<<<<< HEAD
=======
  + ¿Cuál de ellas le parece que contiene más información?

- Usando el programa <code>pearson</code>, obtenga los coeficientes de correlación normalizada entre los
  parámetros 2 y 3 para un locutor, y rellene la tabla siguiente con los valores obtenidos.

  |                        | LP   | LPCC | MFCC |
  |------------------------|:----:|:----:|:----:|
  | &rho;<sub>x</sub>[2,3] |      |      |      |
  
  + Compare los resultados de <code>pearson</code> con los obtenidos gráficamente.
  
- Según la teoría, ¿qué parámetros considera adecuados para el cálculo de los coeficientes LPCC y MFCC?

>>>>>>> Ficheros iniciales de P4
=======
>>>>>>> f8f86880296c2b221da50c6c2b028c2d4c10a90c
### Entrenamiento y visualización de los GMM.

Complete el código necesario para entrenar modelos GMM.

- Inserte una gráfica que muestre la función de densidad de probabilidad modelada por el GMM de un locutor
  para sus dos primeros coeficientes de MFCC.
  
<<<<<<< HEAD
<<<<<<< HEAD
=======
>>>>>>> f8f86880296c2b221da50c6c2b028c2d4c10a90c
  Para obtener las gráficas hemos ejecutado las siguientes ordenes en el terminal:
  
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh
  gmm_train -d work/mfcc -e mfcc -g SES017.gmm lists/class/SES017.train
  plot_gmm_feat work/gmm/mfcc/SES017.gmm
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  
  <img width="640" alt="gmm_mfcc_SES017" src="https://github.com/bgonzalezbosch/P4/blob/Bernat-Roger/gmm_mfcc_SES017.png">

- Inserte una gráfica que permita comparar los modelos y poblaciones de dos locutores distintos (la gŕafica
  de la página 20 del enunciado puede servirle de referencia del resultado deseado). Analice la capacidad
  del modelado GMM para diferenciar las señales de uno y otro.
  
  Modificando el fichero plot_gmm_feat.py y usando los siguientes comandos, hemos obtenido la grafica que permite comparar los modelos y poblaciones de SE012 y SE017.
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh
  plot_gmm_feat -p 99,50,10 work/gmm/mfcc/SES017.gmm work/mfcc/BLOCK01/SES017/SA017S*
  plot_gmm_feat -p 99,50,10 -f blue work/gmm/mfcc/SES017.gmm work/mfcc/BLOCK01/SES012/SA012S*
  plot_gmm_feat -p 99,50,10 -g blue work/gmm/mfcc/SES012.gmm work/mfcc/BLOCK01/SES017/SA017S*
  plot_gmm_feat -p 99,50,10 -g blue -f blue work/gmm/mfcc/SES012.gmm work/mfcc/BLOCK01/SES012/SA012S*
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  la gráfica es la siguiente:

  <img width="777" alt="modelos y poblaciones GMM" src="https://github.com/bgonzalezbosch/P4/blob/Bernat-Roger/gmm1234.png">

  En la gráfica podemos ver las regiones con el 99%, 50% y 10% de la masa de probabilidad para los GMM de los locutores SES017 (en rojo, arriba) y SES012 (en azul, abajo); también se muestra la población del usuario SES017 (en rojo, izquierda) y SES012 (en azul, derecha). 

  Se debe tener en cuanta a la hora de compara que los límites de las gráficas superiores e inferiores no coinciden con tal de poder ver toda la informació con más detalle.

  Para determinar que locutor y población coinciden, las regiones y las poblaciones deben coincidir. Dicho de otra manera, donde hay más concentración de población debe conicidir con los circulos de porcentage más pequeños. Analizando las gráficas vemos como en el caso en el que locutor y población coincidenm se cumple la condición que hemos dicho. En cambio, para GMM:SES012 LOC:SES017 y  GMM:SES017 LOC:SES012 las regiones de la masa de probabilidad de los GMM está desplazada respecto las zonas de más densidad de población. Sobretodo en el cas0 GMM:SES017 LOC:SES012.
<<<<<<< HEAD
=======
- Inserte una gráfica que permita comparar los modelos y poblaciones de dos locutores distintos (la gŕafica
  de la página 20 del enunciado puede servirle de referencia del resultado deseado). Analice la capacidad
  del modelado GMM para diferenciar las señales de uno y otro.
>>>>>>> Ficheros iniciales de P4
=======
>>>>>>> f8f86880296c2b221da50c6c2b028c2d4c10a90c

### Reconocimiento del locutor.

Complete el código necesario para realizar reconociminto del locutor y optimice sus parámetros.

<<<<<<< HEAD
<<<<<<< HEAD
=======
>>>>>>> f8f86880296c2b221da50c6c2b028c2d4c10a90c
Una vez completado el código y optimizado, hemos ejecutado las siguentes ordenes en el terminal:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh
FEAT=lp run_spkid train classerr
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh
FEAT=lpcc run_spkid train classerr
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh
FEAT=mfcc run_spkid train classerr
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Inserte una tabla con la tasa de error obtenida en el reconocimiento de los locutores de la base de datos
  SPEECON usando su mejor sistema de reconocimiento para los parámetros LP, LPCC y MFCC.
  
  |                        | LP   | LPCC | MFCC |
  |------------------------|:----:|:----:|:----:|
  | Número de errores      |77    |27    |9     |
  | Número total           |785   |785   |785   |
  | Tasa de Error (%)      |9.81  |3.44  |1.15  |
<<<<<<< HEAD
=======
- Inserte una tabla con la tasa de error obtenida en el reconocimiento de los locutores de la base de datos
  SPEECON usando su mejor sistema de reconocimiento para los parámetros LP, LPCC y MFCC.
>>>>>>> Ficheros iniciales de P4
=======
>>>>>>> f8f86880296c2b221da50c6c2b028c2d4c10a90c

### Verificación del locutor.

Complete el código necesario para realizar verificación del locutor y optimice sus parámetros.

- Inserte una tabla con el *score* obtenido con su mejor sistema de verificación del locutor en la tarea
  de verificación de SPEECON. La tabla debe incluir el umbral óptimo, el número de falsas alarmas y de
  pérdidas, y el score obtenido usando la parametrización que mejor resultado le hubiera dado en la tarea
  de reconocimiento.
<<<<<<< HEAD
<<<<<<< HEAD
=======
>>>>>>> f8f86880296c2b221da50c6c2b028c2d4c10a90c
  
  - LP:  
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh 
  run_spkid lp train test classerr trainworld verify verifyerr
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh
  =================================
  THR:  0.286750956464601
  Missed:     64/250=0.2560
  FalseAlarm: 15/1000=0.0150
  ---------------------------------
  ==> CostDetection: 39.1
  =================================
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  - LPCC:  
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh 
  run_spkid lpcc train test classerr trainworld verify verifyerr
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh
  =================================
  THR:  0.0315311813148026
  Missed:     19/250=0.0760
  FalseAlarm: 15/1000=0.0150
  ---------------------------------
  ==> CostDetection: 21.1
  =================================
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
- MFCC:  
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh 
  run_spkid mfcc train test classerr trainworld verify verifyerr
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh
  =================================
  THR:  0.200692851725595
  Missed:     19/250=0.0760
  FalseAlarm: 3/1000=0.0030
  ---------------------------------
  ==> CostDetection: 10.3
  =================================
  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

  Como podemos ver en la siguiente tabla  simplificada, nuestro mejor sistema de verificación del locutor es con MFCC.

  |                        | LP   | LPCC | MFCC |
  |------------------------|:----:|:----:|:----:|
  | CostDectection         |39.1  |21.1  |10.3  |


<<<<<<< HEAD
=======
 
>>>>>>> Ficheros iniciales de P4
=======
>>>>>>> f8f86880296c2b221da50c6c2b028c2d4c10a90c
### Test final

- Adjunte, en el repositorio de la práctica, los ficheros `class_test.log` y `verif_test.log` 
  correspondientes a la evaluación *ciega* final.

<<<<<<< HEAD
<<<<<<< HEAD
=======
>>>>>>> f8f86880296c2b221da50c6c2b028c2d4c10a90c
Los ficheros correspondientes a la evaluación *ciega* final los hemos creado a partir de ejecutar los siguientes comandos en el terminal:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh 
FEAT=mfcc run_spkid finalclass
FEAT=mfcc run_spkid finalverif  
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
<<<<<<< HEAD
=======
### Trabajo de ampliación.

- Recuerde enviar a Atenea un fichero en formato zip o tgz con la memoria (en formato PDF) con el trabajo 
  realizado como ampliación, así como los ficheros `class_ampl.log` y/o `verif_ampl.log`, obtenidos como 
  resultado del mismo.
>>>>>>> Ficheros iniciales de P4
=======
>>>>>>> f8f86880296c2b221da50c6c2b028c2d4c10a90c
