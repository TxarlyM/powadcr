# POWADCR
Grabadora digital en formato TAP/TZX para ordenadores de 8 bits
-----

![plot](./doc/powadcr_sp.jpg)
Este proyecto pretende implementar una grabadora digital (para reproducir ficheros TAP/TZX y grabar archivos en ficheros TAP) para máquinas ZX Spectrum, ZX Next y N-Go y clones compatibles basada en la placa de desarrollo AudioKit ESP32 y usando una pantalla HMI tactil de 3,5".

Ésta placa que se muestra a continuación, ESP32 Audio Kit fabricada por AI-Thinker Technology, va equipada con un microcontrolador ESP32 v3 y un procesador de audio ES8388 y que sirve para éste propósito.

![plot](./doc/audiokit.png)

https://docs.ai-thinker.com/en/esp32-audio-kit

El resumen de especificaciones es el siguiente:
+ CPU 32 bits a 240 MHz
+ 512 KB + 4 MB de SRAM (PSRAM disponible)
+ 2 Núcleos
+ Procesador de audio dedicado ES8388
+ Entrada/salida de audio
+ Bluetooth
+ WiFi
+ 8 botones de conmutación
+ Conectores de E/S
+ Ranura SD
+ Módulo de carga de baterías LiPo

Es una placa de desarrollo con grandes posibilidades, aunque el uso al que se enfocaba era muy distinto al que se propone este proyecto. 

Para empezar es necesario utilizar las librerías de Phil Schatzmann para ESP32 Audio Kit v.0.65 (https://github.com/pschatzmann/arduino-audiokit) donde podremos aprovechar todos los recursos de este kit, para crear un reproductor y grabador digital para ZX Spectrum fácilmente, o esta es la primera idea.

Este proyecto necesita configurar los interruptores pintegrados en la placa de la siguiente manera :
|Switch|Valor|
|---|---|
|1|On|
|2|On|
|3|On|
|4|Off|
|5|Off|


## Pantalla LCD

La pantalla táctil LCD elegida para este proyecto es un módulo de pantalla LCD TFT HMI Touch conectado con 2 pines seriales (TX y RX) a la placa.
+ Marca: TJC
+ Modelo: TJC4832T035_011
+ Tamaño: 3,5".
+ Resolución: 480x320.

## Estructura de ficheros TAP para Sinclair ZX Spectrum.

-----

Proceso de carga en Sinclair ZX Spectrum
-----
Recomiendo el sitio web de Alessandro Grussu con información interesante sobre el proceso de carga y el tiempo de procesamiento para este objetivo. Éste es el Link de su página : https://www.alessandrogrussu.it/tapir/tzxform120.html#MACHINFO

Ahora, me gustaría mostrarles cómo la señal generada a partir del archivo TAP que Sinclair ZX Spectrum puede entender. El mecanismo para leer la señal de audio se basa en el conteo de picos de onda cuadrada, utilizando el tiempo de reloj Z80, luego:

La secuencia para ZX Spectrum, es siempre para carga estándar:

+ TONO GUÍA + SYNC1 + SYNC2 + BLOQUE DE DATOS + SILENCIO

Donde: TONO GUÍA (2168 T-States) tiene dos tipos de longitud.

+ Grande (x 8063 T-States) para el bloque "PROGRAMA" típico (BASIC)
+ Corto (x 3223 T-States) para el bloque "BYTE" típico, código máquina de Z80.

![plot](./doc/squarewave_train.png)

**¿Qué significa T-State?**

Éste concepto puede ser difícil de entender, pero no está lejos de la realidad, ya que resumido, un pulso completo (dos picos, uno alto y otro bajo) tiene un período igual al tiempo "2 x n T-State", donde T-State = 1/3,5 MHz = 0,28571 us, por ejemplo: TONO LÍDER GRANDE.

+ TONO GUÍA = 2168 x 8063 T-States = 17480584 T-States
+ 1 T-State = 1 / 3,5 MHz = 0,28571 us = 0,00000028571 s
+ Duración del TONO LÍDER = 17480584 x 0,00000028571 s = 4,98 s

**¿Cuántos picos tiene el tren de pulsos del TONO GUÍA GRANDE?**

+ El tren de pulsos tiene 2168 picos en ambos casos, pero el tono líder corto tiene una duración diferente (3223 estados T) en comparación con el tono líder grande (8063 estados T)

**¿Cuál es la frecuencia de la señal?**

+ Sabemos que el tren de pulsos del TONO GUÍA GRANDE dura 4,98 s
+ Sabemos que el tren de pulsos del TONO GUÍA CORTO dura 1,99 s
+ La frecuencia de ambos tonos líderes (2168 x 0,00000028571) / 2 = 809,2 Hz

Acerca del dispositivo POWADCR
-----
En esta sección describiremos las piezas necesarias para ensamblar el dispositivo PowaDCR.

Lista de materiales

+ Placa base: ESP32 Audiokit de AI-Thinker technology: https://docs.ai-thinker.com/es/esp32-audio-kit (Posible sitio de compra. Alliexpress)
+ LCD color de 3,5" 480x320 píxeles. Pantalla táctil resistiva - TJC4832T035_011 resistiva (de bajo precio pero posible descontinuación y sustitución por TJC4832T135_011C capacitiva o TJC4832T135_011R resistiva)
+ Cable XH2.5 a dupont para conectar el LCD al puerto extendido del ESP32 Audiokit
+ Batería 2000mAh 3.7v (opcional no necesaria)
+ Tarjeta micro SD formateada FAT32 (para contener todos los juegos del ZX Spectrum en TAP y otros formatos para ser utilizados en PowaDCR en el futuro)
+ Tarjeta Micro SD o interfaz serial FTDI FT232RL para programar el LCD TJC (ambos métodos están disponibles)
+ Cable con conectores estéreo-estéreo macho-macho de 3,5 mm para conectar PowaDCR a Spectrum Next o N-Go o versiones clónicas.
+ Cable con conector XH2.5 y mono de 3,5 mm para conectar desde la salida del amplificador de PowaDCR al conector EAR en las versiones clásicas de ZX Spectrum (teclado de goma 16K, 48K, Spectrum+ y Spectrum 128K Toastrack)
+ Versión del editor HMI en chino [http://filedown.tjc1688.com/USARTHMI/USARTHMIsetup_latest.zip]
+ Entorno Visual Studio Code con las extensiones C/C++, CMAKE y CMAKE Tools

Hackeando la placa ESP32 Audiokit
-----
Esta placa está construida a partir del mismo diseño de la versión con chip de audio AC101, pero con chip ES8388. En este caso, tanto el micrófono como la entrada de línea están mezclados. No es posible seleccionar de forma independiente el micrófono o la entrada de línea, ya que el ruido ambiental entra cuando se captura la señal del ZX Spectrum.

Por lo tanto, se deben quitar ambos micrófonos integrados. desoldándolos o bien arrancandolos con un alicate adecuado.

![image](https://github.com/hash6iron/powadcr/assets/118267564/f47c2810-d573-4a8b-9608-7015e7462f15)


¿Cómo se conectan las piezas de PowaDCR?
-----
Para que la placa y la pantalla funcionen hay que instalar el firmware correspondiente antes de proceder a conectarlos entre sí, por lo que debési saltar éste apartado y realizarlo una vez estén la placa y la pantalla lo tengan instalado.

En primer lugar voy a describir cómo deben conectarse los elementos descritos en la lista de materiales. Si es posible ésto debe hacerse en el orden que se indica aquí : 

+ Conectar el cable incluído con la pantalla HMI de 3,5 en el conector J1 de la pantalla.
+ Desde ese conector salen 4 cables que deben conectarse en le puerto serie o UART del Audio Kit. Es importante conectarlos correctamente ya que es posible que la placa o la pantalla puedan resultar dañadas. El conector UART de la placa está situado en la parte superior del ESP32 AudioKit (un connector tipo Dupont de 14 pines) y en el que solo vamos a usar la fila superior de ese conector. De estos 7 pines sólo vamos a usar 4 : GND, TX, RX y 3V3. Las conexión entre placa y pantalla debe realizarse  según la siguiente tabla :

|Pantalla|Pin AudioKit|
|---|---|
|GND|GND|
|RX|TX|
|TX|RX|
|+5V|3V3|
Como se puede observar hay dos pines en la placa AudioKit marcados con 3V3. Cualquiera de los dos puede conectarse a los 5v del conector de la pantalla.

La placa de AudioKit internamente funciona a 3,3v por lo que se le está suministrando a la pantalla una tensión que no es la que corresponde porque la pantalla funciona a 5v. Ésto no afecta a su funcionamiento, pero en ocasiones puede producir un parpadeo que puede ser molesto.



Hackeando la pantalla HMI
-----
La pantalla por defecto, como se ha mencionado antes, trabaja con un voltaje de 5v. Para poder hacer que la pantalla trabaje exclusivamente a 3,3v hay que hacer una pequeña modificación : Hay que localizar cerca del conector J1 dos contactos marcados como JP2 en los que no hay conectado nada. El fabricante debía haber colocado un conector de Dupont de 2 pines para poder insertar un Jumper si se desea hacer que la pantalla trabaje a 3,3v o no. Si vamos a usar ésta pantalla exclusivamente para el POWADCR lo que debemos hacer es poner un punto de estaño uniendo esos dos contactos ya que los agujeros no están hechos y no es posible poner un conector para el Jumper. Una vez unidos los contactos comprobaremos que la placa no tiene ese parpadeo mencionado antes.




How .bin firmware is uploaded in ESP32-A1S Audiokit? 
-----
1. Download ESP32 Flash Downloading Tool - https://www.espressif.com/en/support/download/other-tools?keys=&field_type_tid%5B%5D=13
2. Unzip file and execute - flash_download_tool_x.x.x.exe file

   See example image below.

   ![image](https://github.com/hash6iron/powadcr_IO/assets/118267564/e7158518-4af8-4e6e-b4ab-eff6b9693307)

   
4. Select ESP32 model.
   - ESP32
   - Develop
     And press "OK" button
     
     Show image below.
     
     ![image](https://github.com/hash6iron/powadcr_IO/assets/118267564/e9348bcb-2879-4872-8998-7e14c02b6f82)

   
5. Setting and begin the flash proccess.
   - Select .bin file or write the path of it (see below lastest stable version)
   - Select all parameters exactly at the image below.
   - Connect ESP32-A1S Audiokit board from UART microUSB port (not power microUSB PORT) at PC USB port.
   - Select the available COM for this connection in COM: field on ESP32 Flash Downloading Tool.
   - Select BAUD: speed at 921600
   - Disconnect the Touch Screen cable in order to release serial port (Audiokit sharing USB and UART communications)
   - Press START button in ESP32 Flash Downloading Tool. Then downloading proccess begin, and wait for FINISH message. Enjoy!
  
     NOTES: If the proccess fail.
      - Try to download again.
      - Try to ERASE before START proccess.

     lastest version: <a href="https://github.com/hash6iron/powadcr_IO/blob/main/powaDCR-ESP32-A1S-v0.3.15.bin">powaDCR-ESP32A1S-v0.3.15.bin</a>
   
   Show image below.
   
   ![image](https://github.com/hash6iron/powadcr_IO/assets/118267564/a595ea33-a1df-406e-b2af-a1d72680fb59)



¿Cómo se carga el firmware personalizado en ESP32-A1S Audiokit? 
-----
Hay dos formas de realizar éste proceso :

1. Usando Visual Studio Code : Este es el método más fácil para compilar los fuentes de éste proyecto y al mismo tiempo subir el firmware en un solo paso. Para ésto debemos hacer lo siguiente :
+ Instalar Visual Studio Code y una vez instalado se deben instalar las extensiones C/C++, CMAKE y CMAKE Tools
+ 

1. Install Arduino IDE 2.0
2. Apply this BOARD repository to Arduino IDE preferences.
   - https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json

3. Connect the ESP32 Audiokit USART USB port to any USB PC port
4. Set board as "ESP32 DEV MOD" and select correct COM port

Required libraries:
- SdFat (https://github.com/greiman/SdFat)
- arduino-audiokit (https://github.com/pschatzmann/arduino-audiokit/tree/main)

How firmware is loaded in TJC LCD?
-----
(in progress)

What PowaDCR beta version is able to do?
-----
(in progress)

If you enjoy with this device and you want to colaborate, please.

<a href="https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=BAWGJFZGXE5GE&source=url"><img src="/doc/paypal_boton.png" /></a>

<a href="https://www.buymeacoffee.com/atamairon"><img src="/doc/coffe.jpg" /></a>

thxs.
