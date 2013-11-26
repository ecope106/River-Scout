River-Scout
===========

Ultrasonic sensor, two servos, dc motor, and led
/*
 HC-SR04 Ping distance sensor:
 VCC to arduino 5v 
 GND to arduino GND
 Echo to Arduino pin 12 
 Trig to Arduino pin 12
 */

#include <Servo.h>

Servo SensorHeadServo;   // Create Servo object for sensor head servo
Servo MotorServo;        // Create Servo object for second servo

const int echoPin = 11;   //control pin for echo
const int trigPin = 12;   //control pin for trigger
const int LEDPin = 13;    //control pin for led indicator
const int DcMotorPin = 8; //control in for propulsion

const int minimumRange = 40;    // Minimum range needed before course needs to change

// Set up initial conditions by:
// 1. Set serial port to 9600 baud
// 2. Configure trigger pin as an output
// 3. Configure the echo pin as an input
// 4. Configure the LED pin as an output
// 5. Configure the DC motor pin as an output
// 6. Attach to the sensor head and motor servos
// 7. Point both servos straight ahead
void setup() {
  Serial.begin (9600);
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  pinMode(LEDPin, OUTPUT);       // Use LED indicator (if required)
  pinMode(DcMotorPin, OUTPUT);
  SensorHeadServo.attach(10);
  MotorServo.attach(9);

  // Point both servos straight ahead
  SensorHeadServo.write(90);
  MotorServo.write(90);
}

// Stop the DC motor
void stopMotor() {
  Serial.println("Stopping motor");
  digitalWrite(DcMotorPin, LOW);
  digitalWrite(LEDPin, HIGH);
}

// Start the DC motor
void startMotor() {
  Serial.println("Starting motor");
  digitalWrite(DcMotorPin, HIGH);
  digitalWrite(LEDPin, LOW);
}

// Turn the sensor head to the right
void lookRight() {
  Serial.println("Looking right");
  SensorHeadServo.write(45);
}

// Trun the sensor head to the left
void lookLeft() {
  Serial.println("Looking left");
  SensorHeadServo.write(135);
}

// Turn the object to the right and then point the servo straight ahead
void turnRight() {
  Serial.println("Turning right");
  MotorServo.write(30);
  digitalWrite(DcMotorPin, HIGH);
  delay(200);
  MotorServo.write(90);
}

// Turn the object to the left and then point the servo straight ahead
void turnLeft() {
  Serial.println("Turning left");
  MotorServo.write(160);
  digitalWrite(DcMotorPin, HIGH);
  delay(2000);
  MotorServo.write(90);
}

bool somethingInFrontOfUs() {

  long duration;
  bool tooClose;

  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);

  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);

  digitalWrite(trigPin, LOW);
  duration = pulseIn(echoPin, HIGH);
  
  Serial.print("Duration in front of us is ");
  Serial.println(duration);

  // Return the distance (in cm) is too close based on the speed of sound.
  tooClose = (duration/58.2) <= minimumRange;
  
  Serial.print("The value of tooClose is ");
  Serial.println(tooClose);
  return (tooClose);
}

void loop() {
  
  // Look to see if there is anything in front of us
  if (somethingInFrontOfUs()) {
    // Yes, something is in front of us, stop the motor
    stopMotor();
    // Look to the right
    lookRight();
    // Is there something in front of us there?
    if (!somethingInFrontOfUs()) {
      // No, nothing to the right so turn right
      turnRight();
    } else {
      // Yes there was something to the right so look left
      lookLeft();
      // Is there something in front of us there?
      if (!somethingInFrontOfUs()) {
        // No, nothing to the left so turn left
        turnLeft();
      }
    }
  } 
  else {
    // Nothing in front of us, start motor
    startMotor();
  }
}
