// C++ code
//
#include <Servo.h>
#include <DHT11.h>

#include "DFRobot_PH.h"
#include <EEPROM.h>

#define PH_PIN A1
float voltage,phValue,temperature = 25;
DFRobot_PH ph;

DHT11 dht11(2);

int pos = 0;

int cm = 0;

long readUltrasonicDistance(int triggerPin, int echoPin)
{
  pinMode(triggerPin, OUTPUT);  // Clear the trigger
  digitalWrite(triggerPin, LOW);
  delayMicroseconds(2);
  // Sets the trigger pin to HIGH state for 10 microseconds
  digitalWrite(triggerPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(triggerPin, LOW);
  pinMode(echoPin, INPUT);
  // Reads the echo pin, and returns the sound wave travel time in microseconds
  return pulseIn(echoPin, HIGH);
}


Servo servo_9;

void setup()
{
  Serial.begin(9600);
  servo_9.attach(9);

  // RAINDROP SENSOR 
  pinMode(A0,  INPUT);
}

void loop()
{
  for (pos = 0; pos <= 180; pos += 3) {
    servo_9.write(pos);
    digitalWrite(LED_BUILTIN, HIGH);
    delay(10); // Wait for 15 millisecond(s)
  }
  for (pos = 180; pos >= 0; pos -= 3) {
    servo_9.write(pos);
    digitalWrite(LED_BUILTIN, LOW);
    delay(10); // Wait for 15 millisecond(s)
  }
    // measure the ping time in cm
  cm = 0.01723 * readUltrasonicDistance(7, 7);
  Serial.print("Distance: ");
  Serial.println(cm);
  delay(15); // Wait for 100 millisecond(s)

  // TEMP + HUMIDITY SENSOR
  int temperature = dht11.readTemperature();
  Serial.print("Temperature: ");
  Serial.println(temperature);

  // RAINDROP SENSOR
  int sensorRead = digitalRead(10);
  Serial.print("Raindrop Sensitivity: ");
  Serial.println(sensorRead);
  static unsigned long timepoint = millis();
  if(millis()-timepoint>1000U){                  //time interval: 1s
      timepoint = millis();
      //temperature = readTemperature();         // read your temperature sensor to execute temperature compensation
      voltage = analogRead(PH_PIN)/1024.0*5000;  // read the voltage
      phValue = ph.readPH(voltage,temperature);  // convert voltage to pH with temperature compensation
      Serial.print("pH:");
      Serial.println(phValue,2);
  }
  ph.calibration(voltage,temperature);           // calibration process by Serail CMD

}

