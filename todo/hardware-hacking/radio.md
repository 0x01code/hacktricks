# Radio

<details>

<summary><strong>Aprende a hackear AWS de cero a héroe con</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (Experto en Equipo Rojo de AWS de HackTricks)</strong></a><strong>!</strong></summary>

Otras formas de apoyar a HackTricks:

* Si quieres ver tu **empresa anunciada en HackTricks** o **descargar HackTricks en PDF** Consulta los [**PLANES DE SUSCRIPCIÓN**](https://github.com/sponsors/carlospolop)!
* Obtén la [**merchandising oficial de PEASS & HackTricks**](https://peass.creator-spring.com)
* Descubre [**La Familia PEASS**](https://opensea.io/collection/the-peass-family), nuestra colección exclusiva de [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Únete al** 💬 [**grupo de Discord**](https://discord.gg/hRep4RUj7f) o al [**grupo de telegram**](https://t.me/peass) o **síguenos** en **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Comparte tus trucos de hacking enviando PRs a los** [**HackTricks**](https://github.com/carlospolop/hacktricks) y [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) repositorios de github.

</details>

## SigDigger

[**SigDigger** ](https://github.com/BatchDrake/SigDigger)es un analizador de señales digitales gratuito para GNU/Linux y macOS, diseñado para extraer información de señales de radio desconocidas. Admite una variedad de dispositivos SDR a través de SoapySDR, y permite la demodulación ajustable de señales FSK, PSK y ASK, decodificar video analógico, analizar señales intermitentes y escuchar canales de voz analógicos (todo en tiempo real).

### Configuración Básica

Después de instalar, hay algunas cosas que podrías considerar configurar.\
En la configuración (el segundo botón de la pestaña) puedes seleccionar el **dispositivo SDR** o **seleccionar un archivo** para leer y a qué frecuencia sintonizar y la tasa de muestreo (se recomienda hasta 2.56Msps si tu PC lo soporta)\\

![](<../../.gitbook/assets/image (245).png>)

En el comportamiento de la GUI se recomienda habilitar algunas cosas si tu PC lo soporta:

![](<../../.gitbook/assets/image (472).png>)

{% hint style="info" %}
Si te das cuenta de que tu PC no está capturando cosas, intenta deshabilitar OpenGL y reducir la tasa de muestreo.
{% endhint %}

### Usos

* Solo para **capturar un tiempo de una señal y analizarla** mantén presionado el botón "Presionar para capturar" todo el tiempo que necesites.

![](<../../.gitbook/assets/image (960).png>)

* El **Sintonizador** de SigDigger ayuda a **capturar mejores señales** (pero también puede degradarlas). Idealmente comienza con 0 y sigue **aumentándolo hasta** que encuentres que el **ruido** introducido es **mayor** que la **mejora de la señal** que necesitas).

![](<../../.gitbook/assets/image (1099).png>)

### Sincronización con canal de radio

Con [**SigDigger** ](https://github.com/BatchDrake/SigDigger)sincronízate con el canal que deseas escuchar, configura la opción "Vista previa de audio de banda base", configura el ancho de banda para obtener toda la información enviada y luego ajusta el Sintonizador al nivel antes de que el ruido realmente comience a aumentar:

![](<../../.gitbook/assets/image (585).png>)

## Trucos interesantes

* Cuando un dispositivo está enviando ráfagas de información, generalmente la **primera parte será un preámbulo** por lo que **no** necesitas **preocuparte** si **no encuentras información** allí **o si hay algunos errores**.
* En tramas de información generalmente deberías **encontrar diferentes tramas bien alineadas entre sí**:

![](<../../.gitbook/assets/image (1076).png>)

![](<../../.gitbook/assets/image (597).png>)

* **Después de recuperar los bits es posible que necesites procesarlos de alguna manera**. Por ejemplo, en la codificación Manchester un arriba+abajo será un 1 o 0 y un abajo+arriba será el otro. Por lo tanto, pares de 1s y 0s (arribas y abajos) serán un 1 real o un 0 real.
* Incluso si una señal está utilizando la codificación Manchester (es imposible encontrar más de dos 0s o 1s seguidos), podrías **encontrar varios 1s o 0s juntos en el preámbulo**!

### Descubriendo el tipo de modulación con IQ

Hay 3 formas de almacenar información en señales: Modulando la **amplitud**, **frecuencia** o **fase**.\
Si estás revisando una señal hay diferentes formas de intentar averiguar qué se está utilizando para almacenar información (encuentra más formas abajo) pero una buena es revisar el gráfico IQ.

![](<../../.gitbook/assets/image (788).png>)

* **Detectando AM**: Si en el gráfico IQ aparecen por ejemplo **2 círculos** (probablemente uno en 0 y otro en una amplitud diferente), podría significar que esta es una señal AM. Esto se debe a que en el gráfico IQ la distancia entre el 0 y el círculo es la amplitud de la señal, por lo que es fácil visualizar diferentes amplitudes siendo utilizadas.
* **Detectando PM**: Como en la imagen anterior, si encuentras pequeños círculos no relacionados entre sí probablemente signifique que se está utilizando una modulación de fase. Esto se debe a que en el gráfico IQ, el ángulo entre el punto y el 0,0 es la fase de la señal, lo que significa que se utilizan 4 fases diferentes.
* Ten en cuenta que si la información está oculta en el hecho de que se cambia una fase y no en la fase en sí, no verás diferentes fases claramente diferenciadas.
* **Detectando FM**: IQ no tiene un campo para identificar frecuencias (la distancia al centro es la amplitud y el ángulo es la fase).\
Por lo tanto, para identificar FM, deberías **ver básicamente solo un círculo** en este gráfico.\
Además, una frecuencia diferente se "representa" en el gráfico IQ por una **aceleración de velocidad a través del círculo** (así que en SysDigger seleccionando la señal el gráfico IQ se llena, si encuentras una aceleración o cambio de dirección en el círculo creado podría significar que esto es FM):

## Ejemplo de AM

{% file src="../../.gitbook/assets/sigdigger_20220308_165547Z_2560000_433500000_float32_iq.raw" %}

### Descubriendo AM

#### Revisando el sobre

Revisando la información AM con [**SigDigger** ](https://github.com/BatchDrake/SigDigger)y solo mirando el **sobre** puedes ver diferentes niveles de amplitud claros. La señal utilizada está enviando pulsos con información en AM, así es como se ve un pulso:

![](<../../.gitbook/assets/image (590).png>)

Y así es como se ve parte del símbolo con la forma de onda:

![](<../../.gitbook/assets/image (734).png>)

#### Revisando el Histograma

Puedes **seleccionar toda la señal** donde se encuentra la información, seleccionar el modo **Amplitud** y **Selección** y hacer clic en **Histograma**. Puedes observar que solo se encuentran 2 niveles claros

![](<../../.gitbook/assets/image (264).png>)

Por ejemplo, si seleccionas Frecuencia en lugar de Amplitud en esta señal AM encontrarás solo 1 frecuencia (no hay forma de que la información modulada en frecuencia esté utilizando solo 1 frec).

![](<../../.gitbook/assets/image (732).png>)

Si encuentras muchas frecuencias, potencialmente esto no será FM, probablemente la frecuencia de la señal fue modificada debido al canal.
#### Con IQ

En este ejemplo puedes ver cómo hay un **gran círculo** pero también **muchos puntos en el centro.**

![](<../../.gitbook/assets/image (222).png>)

### Obtener Tasa de Símbolos

#### Con un símbolo

Selecciona el símbolo más pequeño que puedas encontrar (para asegurarte de que es solo 1) y verifica la "Frecuencia de selección". En este caso sería 1.013kHz (así que 1kHz).

![](<../../.gitbook/assets/image (78).png>)

#### Con un grupo de símbolos

También puedes indicar la cantidad de símbolos que vas a seleccionar y SigDigger calculará la frecuencia de 1 símbolo (probablemente cuanto más símbolos selecciones, mejor). En este escenario seleccioné 10 símbolos y la "Frecuencia de selección" es 1.004 Khz:

![](<../../.gitbook/assets/image (1008).png>)

### Obtener Bits

Habiendo encontrado que es una señal **modulada en AM** y la **tasa de símbolos** (y sabiendo que en este caso algo hacia arriba significa 1 y algo hacia abajo significa 0), es muy fácil **obtener los bits** codificados en la señal. Entonces, selecciona la señal con información y configura el muestreo y la decisión y presiona muestrear (verifica que se haya seleccionado **Amplitud**, que se haya configurado la **Tasa de símbolos descubierta** y que se haya seleccionado la **Recuperación de reloj de Gadner**):

![](<../../.gitbook/assets/image (965).png>)

* **Sincronizar con intervalos de selección** significa que si previamente seleccionaste intervalos para encontrar la tasa de símbolos, esa tasa de símbolos se utilizará.
* **Manual** significa que se utilizará la tasa de símbolos indicada.
* En **Selección de intervalo fijo** indicas el número de intervalos que se deben seleccionar y calcula la tasa de símbolos a partir de ello.
* **Recuperación de reloj de Gadner** suele ser la mejor opción, pero aún así necesitas indicar una tasa de símbolos aproximada.

Al presionar muestrear, aparece esto:

![](<../../.gitbook/assets/image (644).png>)

Ahora, para hacer que SigDigger entienda **dónde está el rango** del nivel que lleva información, necesitas hacer clic en el **nivel más bajo** y mantener presionado hasta el nivel más alto:

![](<../../.gitbook/assets/image (439).png>)

Si hubiera, por ejemplo, **4 niveles diferentes de amplitud**, deberías haber necesitado configurar los **Bits por símbolo en 2** y seleccionar desde el más pequeño hasta el más grande.

Finalmente, **aumentando** el **Zoom** y **cambiando el tamaño de la fila** puedes ver los bits (y puedes seleccionar todo y copiar para obtener todos los bits):

![](<../../.gitbook/assets/image (276).png>)

Si la señal tiene más de 1 bit por símbolo (por ejemplo 2), SigDigger no tiene **forma de saber cuál símbolo es** 00, 01, 10, 11, por lo que usará diferentes **escalas de grises** para representar cada uno (y si copias los bits usará **números del 0 al 3**, tendrás que tratarlos).

Además, el uso de **codificaciones** como **Manchester**, y **arriba+abajo** puede ser **1 o 0** y un abajo+arriba puede ser un 1 o 0. En esos casos necesitas **tratar los arribas (1) y abajos (0)** obtenidos para sustituir los pares de 01 o 10 como 0s o 1s.

## Ejemplo de FM

{% file src="../../.gitbook/assets/sigdigger_20220308_170858Z_2560000_433500000_float32_iq.raw" %}

### Descubriendo FM

#### Verificación de las frecuencias y la forma de onda

Ejemplo de señal enviando información modulada en FM:

![](<../../.gitbook/assets/image (725).png>)

En la imagen anterior puedes observar claramente que se utilizan **2 frecuencias**, pero si **observas** la **forma de onda** es posible que **no puedas identificar correctamente las 2 frecuencias diferentes**:

![](<../../.gitbook/assets/image (717).png>)

Esto se debe a que capturé la señal en ambas frecuencias, por lo tanto una es aproximadamente la otra en negativo:

![](<../../.gitbook/assets/image (942).png>)

Si la frecuencia sincronizada está **más cerca de una frecuencia que de la otra**, puedes ver fácilmente las 2 frecuencias diferentes:

![](<../../.gitbook/assets/image (422).png>)

![](<../../.gitbook/assets/image (488).png>)

#### Verificación del histograma

Al verificar el histograma de frecuencia de la señal con información, puedes ver fácilmente 2 señales diferentes:

![](<../../.gitbook/assets/image (871).png>)

En este caso, si verificas el **histograma de amplitud** encontrarás **solo una amplitud**, por lo que **no puede ser AM** (si encuentras muchas amplitudes puede ser porque la señal ha ido perdiendo potencia a lo largo del canal):

![](<../../.gitbook/assets/image (817).png>)

Y este sería el histograma de fase (lo que deja claro que la señal no está modulada en fase):

![](<../../.gitbook/assets/image (996).png>)

#### Con IQ

IQ no tiene un campo para identificar frecuencias (la distancia al centro es la amplitud y el ángulo es la fase).\
Por lo tanto, para identificar FM, deberías **ver básicamente solo un círculo** en este gráfico.\
Además, una frecuencia diferente es "representada" en el gráfico IQ por una **aceleración de velocidad a través del círculo** (así que en SysDigger seleccionando la señal, el gráfico IQ se llena, si encuentras una aceleración o cambio de dirección en el círculo creado, podría significar que se trata de FM):

![](<../../.gitbook/assets/image (81).png>)

### Obtener Tasa de Símbolos

Puedes utilizar la **misma técnica que la utilizada en el ejemplo de AM** para obtener la tasa de símbolos una vez que hayas encontrado las frecuencias que transportan los símbolos.

### Obtener Bits

Puedes utilizar la **misma técnica que la utilizada en el ejemplo de AM** para obtener los bits una vez que hayas **encontrado que la señal está modulada en frecuencia** y la **tasa de símbolos**.
