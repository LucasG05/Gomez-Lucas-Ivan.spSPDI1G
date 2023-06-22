# Gomez Lucas Ivan 1G

## Segundo Parcial SPD

### Descripción
El proyecto consiste en un sistema de deteccion de incendio hecho con una Arduino.
Detecta cambios de temperatura y, mediante los mismos, es capaz de detectar la estación del año,
mostrando todos los datos en un Display LCD. Además, en caso de detectar un incendio activa un
servo motor, simulando una respuesta del sistema.

### Funcion para mostrar estaciones
Esta funcion recibe la temperatura previamente detectada por un sensor y según la misma
indica en el LCD la estación del año en la que se encuentra. Si la temperatura es menor
a 5°, indica invierno. Si es mayor a 5° y menor o igual a 15°, indica otoño. Si es mayor a
15° y menor o igual a 25°, indica Primavera, y si es mayor a 25° y menor o igual a 60°, indica verano. 
~~~
void MostrarEstacion(float temperatura)
{
  if (temperatura <= 5)
  {
    lcd_1.print("Invierno        ");
  }
  else
  {
    if (temperatura > 5 & temperatura <= 15)
    {
      lcd_1.print("Otonio          ");
    }
    else
    {
      if (temperatura > 15 & temperatura <= 25)
      {
        lcd_1.print("Primavera       ");
      }
      else
      {
        if (temperatura > 25 & temperatura <= 60)
        {
          lcd_1.print("Verano          ");
        }
      }
    }
    
  }
}
~~~

### Funcion para mover el servo motor
Esta función mueve el servo motor hasta llegar al máximo de grados indicado,
avanzando de 2 en 2. Al llegar al máximo, retrocede nuevamente hasta llegar a 0.
~~~
void MoverServoMotor(int grados_max)
{
  int pos = 0;
  for (pos = 0; pos <= grados_max; pos += 2) {
    servo_motor.write(pos);
    delay(30); 
  }
  for (pos = grados_max; pos >= 0; pos -= 2) 
  {
    servo_motor.write(pos);
    delay(30);
  }
}
~~~

### Funcion para establecer el estado de alarma
Esta funcion recibe la temperatura y los pines a los qu están conectados un led rojo y
uno verde. En caso de haber más de 60°, activará un estado de alarma, encendiendo y
apagando el led rojo, moviendo el servo motor y mostrando una alerta en el LCD.
~~~
void EstablecerEstadoAlarma(float temperatura, int led, int led_alarma)
{
  if (temperatura > 60)
  {
    lcd_1.print("ALERTA INCENDIO!");
    digitalWrite(led_alarma, HIGH);
    MoverServoMotor(180);
   }
  else
  {
    digitalWrite(led_alarma, LOW);
    EncenderYApagarLed(500, led);
  }
}
~~~

### Funcion para encender y apagar un led
Esta funcion recibe el pin al que está conectado el led y el tiempo de delay que
queremos que tenga entre el encendido y apagado. Hace que el led se encienda y apague
continuamente.
~~~
void EncenderYApagarLed(int tiempo,int pinLed)
{
  digitalWrite(pinLed, HIGH);
  delay(tiempo); 
  digitalWrite(pinLed, LOW);
  delay(tiempo); 
}
~~~

### Funcion para procesar las señales del control remoto
CONTROL_POWER, CONTROL_SISTEMA y CONTROL_ESTACIONES son #define con los numeros hexadecimales
que recibimos al presionar ciertos botones del control remoto.
Esta funcion compara el código numerico recibido al presionar un boton del control remoto,
con los tres define, en caso de recibir alguno de esos 3, cambia la variable "estado" de ese
botón a 1, haciendo que se active el funcionamiento asociado a ese botón.
~~~
void RecibirSenialesIR()
{
  if (IrReceiver.decodedIRData.decodedRawData == CONTROL_POWER)
  {
    if (estado_power == 0)
    {
      estado_power = 1;
    }
    else
    {
      estado_power = 0;
    }
  }
  if (IrReceiver.decodedIRData.decodedRawData == CONTROL_SISTEMA)
  {
    if (estado_sistema_alarma == 0)
    {
      estado_sistema_alarma = 1;
    }
    else
    {
      estado_sistema_alarma = 0;
    }
  }
  if (IrReceiver.decodedIRData.decodedRawData == CONTROL_ESTACIONES)
  {
    estado_deteccion_estaciones = 1; 
  }
}
~~~

### Loop del sistema
Funcionamiento: 
El sistema completo empieza apagado. Al presionar el botón de power del control
remoto encendemos el display LCD y comenzamos a mostrar la temperatura del ambiente. Al
presionarlo nuevamente se apagará.
Al presionar el botón "func/stop" del control remoto, activamos además la detección del
sistema de alarma, para que se active la respuesta del sistema ante una temperatura mayor
a 60°. Al presionarlo nuevamente lo desactivaremos.
Por último, cada vez que queramos detectar la estación del año, debemos precionar el 
número 0 del control.

~~~
void loop()
{
  //Decodificar las señales infrarrojo recibidas
  if (IrReceiver.decode()) {
    RecibirSenialesIR();
    IrReceiver.resume();
  }
    
  if (estado_power == 1)
  {
    lcd_1.display();
    // Leo la temperatura
    int lectura = analogRead(A0);
    float temperatura = map(lectura,20,358,-40,125);

	if (estado_deteccion_estaciones == 1)
    {
      lcd_1.setCursor(0,0); // Posiciono el cursor
      MostrarEstacion(temperatura);
      estado_deteccion_estaciones = 0;
    }
    
    lcd_1.setCursor(0,1); // Posiciono el cursor
    lcd_1.print(temperatura);
    lcd_1.print(" grados.       ");
    
    if(estado_sistema_alarma == 1)
    {
      lcd_1.setCursor(0,0);
      EstablecerEstadoAlarma(temperatura, LED_VERDE, LED_ROJO);
      delay(100);
    }
  }
  else
  {
    lcd_1.noDisplay();
  }
  delay(100);
}
~~~

### Link al proyecto
[Proyecto](https://www.tinkercad.com/things/0efi9vD7Fjd-exquisite-gogo-kieran/editel?sharecode=eNrOsNPXUEcM4oCarl40mb9nDY_nHQnokWXpUwqlb4A) 
