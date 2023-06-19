# SPD_RocioGonzalezSegundoParcial

* Rocio González 1B

## Proyecto : Sistema de incendios, parcial

##	Diagramas 
[![segundo-parcial-Rocio-Gonzalez.png](https://i.postimg.cc/c15R5FrX/segundo-parcial-Rocio-Gonzalez.png)](https://postimg.cc/RqKJWQ1H)
[![Parcial-Dos.jpg](https://i.postimg.cc/Pq3bsySX/Parcial-Dos.jpg)](https://postimg.cc/hJmQVLF6)

##	Elementos Utilizados:
• Arduino UNO
• Sensor de temperatura
• Control remoto IR (Infrarrojo)
• Display LCD (16x2 caracteres)
• 1 Servo motor
• Cables y resistencias 
• Protoboard para realizar las conexiones
• Dos leds, uno azul y uno rojo

##	Funcionamiento Integral

<pre lang="cpp">
#include <LiquidCrystal.h>
#include <Servo.h>
#include <IRremote.h>
</pre>
Importo las bibliotecas necesarias para utilizar el LCD, el servo y el control remoto infrarrojo.

<pre lang="cpp">
#define Tecla_1 0xEF10BF00
#define Tecla_2 0xEE11BF00
#define Tecla_3 0xED12BF00
#define Tecla_4 0xEB14BF00

#define LED_ROJO 12
#define LED_AZUL 13
</pre>
Defino constantes que representan los códigos de los botones del control remoto infrarrojo y defino los leds.

<pre lang="cpp">
  const int pinSensor = A0;
  const int pinIR = 11;
  int temperatura;
  int estacionSeleccionada = 0;
  int temperaturaEstacion;
</pre>
Defino el pin analógico del sensor, el pin del infrarrojo, y las variables para la temperatura del sensor y la de la estacion seleccionada, también se almacena el estado de la estación seleccionada.

<pre lang="cpp">
  Servo myservo;
  LiquidCrystal lcd(2, 3, 4, 5, 6, 7);
</pre>

Declaro el servo y los pines a los que conecté el lcd

<pre lang="cpp">
  myservo.attach(9, 400, 2600);
  myservo.write(0);
  lcd.begin(16, 2);
  lcd.print("Temp: ");
  IrReceiver.begin(pinIR, DISABLE_LED_FEEDBACK);
  pinMode(LED_ROJO, OUTPUT);
  pinMode(LED_AZUL, OUTPUT);
  Serial.begin(9600);
</pre> 

Defino el pulso del servo y lo dejo en una posición inicial de 0°, también inicio la comunicación con el lcd especificando su tamaño y muestro la palabra "temp" que aparecerá al iniciar el arduino. 
Siguiendo con el loop, inicio el receptor de infrarrojos con el pin especificado y se desactiva el feedback del LED del receptor, también configuro los leds como salidas y por último hago la comunicación serial.

<pre lang="cpp">
  void loop() {
  // Actualizar la temperatura mapeada con la lectura del sensor
  int lecturaSensor = analogRead(pinSensor);
  if (lecturaSensor >= 0 && lecturaSensor <= 1023) {
    temperatura = map(lecturaSensor, 20, 358, -40, 125);
  } else {
    // Error de lectura del sensor, establecer temperatura a un valor inválido
    temperatura = -1000;
  }

  // Control
  controlar();

  delay(1000);

  // Mostrar la temperatura en el LCD
  mostrarTemperaturas(temperatura);
}
</pre>

En la función loop, leo el valor del sensor de temperatura y se realiza una mapeo para convertir el valor leído en una temperatura en grados C. Si la lectura del sensor está fuera del rango válido, se asigna un valor inválido (-1000) a la variable de temperatura.
También llamo a la función controlar() para procesar las señales del control remoto y actualizar el estado de la estación seleccionada y muestro la temperatura con la funcion mostrarTemperaturas() que también verifica si debo mostrar la alerta de incendio

<pre lang="cpp">
  void mostrarTemperaturas(int temperaturaActual) {
  lcd.setCursor(0, 0);
  lcd.print("Temp: ");
  lcd.print(temperaturaActual);

  lcd.setCursor(0, 1);
  if (estacionSeleccionada == 0) {
    lcd.print("                ");
  } else {
    lcd.print("Temp Est: ");
    lcd.print(temperaturaEstacion);
  }

  if (estacionSeleccionada != 0) {
    if (temperatura > temperaturaEstacion) {
      lcd.clear();
      lcd.setCursor(0, 0);
      lcd.print("INCENDIO");
      encenderApagar(LED_ROJO, LED_AZUL);
      moverServo();
    } else {
      myservo.write(0);
      encenderApagar(LED_AZUL, LED_ROJO);
    }
  }
}
</pre>

La función mostrarTemperaturas() muestra la temperatura actual y la temperatura de la estación seleccionada en el LCD. Si la temperatura actual supera la temperatura de la estación, se muestra el mensaje "INCENDIO" en el LCD y se enciende el LED rojo mientras se apaga el LED azul. Además, llamo a la función moverServo() para mover el servo cuando se inicia el incendio y cuando se desactiva. 

<pre lang="cpp">
  void controlar() {
  if (IrReceiver.decode()) {
    Serial.println(IrReceiver.decodedIRData.decodedRawData, HEX);
    if (IrReceiver.decodedIRData.decodedRawData == Tecla_1) {
      estacionSeleccionada = 1;
      temperaturaEstacion = 24;
    } else if (IrReceiver.decodedIRData.decodedRawData == Tecla_2) {
      estacionSeleccionada = 2;
      temperaturaEstacion = 15;
    } else if (IrReceiver.decodedIRData.decodedRawData == Tecla_3) {
      estacionSeleccionada = 3;
      temperaturaEstacion = 5;
    } else if (IrReceiver.decodedIRData.decodedRawData == Tecla_4) {
      estacionSeleccionada = 4;
      temperaturaEstacion = 30;
    }

    lcd.setCursor(0, 1);
    switch (estacionSeleccionada) {
      case 1:
        lcd.print("Primavera ");
        break;
      case 2:
        lcd.print("Otonio     ");
        break;
      case 3:
        lcd.print("Invierno  ");
        break;
      case 4:
        lcd.print("Verano    ");
        break;
      default:
        lcd.print("          ");
        break;
    }
    lcd.print(temperaturaEstacion);
    lcd.print("C");

    IrReceiver.resume();
  }
  delay(1);
}
</pre>

La función controlar() se encarga de procesar las señales del control remoto infrarrojo. Dependiendo del código de la señal recibida, se actualiza el estado de la estación seleccionada y se asigna la temperatura correspondiente. Luego, se muestra el nombre de la estación y su temperatura en el LCD.


<pre lang="cpp">
  void encenderApagar(int leduno, int ledos) {
  digitalWrite(leduno, HIGH);
  digitalWrite(ledos, LOW);
}
</pre>
La función encenderApagar() se utiliza para encender un LED y apagar el otro.

<pre lang="cpp">
  void moverServo() {
  myservo.write(90);
  delay(1000);
}
</pre>
Por último, la función moverServo() se utiliza para mover el servo a 90 grados.

### Código fuente del proyecto 
https://onlinegdb.com/roKiy5VNt
