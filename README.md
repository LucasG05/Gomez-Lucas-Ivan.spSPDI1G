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

