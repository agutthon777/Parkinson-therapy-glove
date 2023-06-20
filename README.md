# Raven Media Co. 
69H Kilobiro Road Email: neal.kelley@ravenemediastudio.com
Kampala, Uganda

# Parkinson's Disease Vibrotherapy Glove

This repository contains the source code and design files for a Parkinson's disease vibrotherapy glove. The glove utilizes vibration therapy to help manage the symptoms of Parkinson's disease. 

The project employs an Arduino Nano to control coin motors, which are placed strategically in the glove's fingers. The motors are programmed to vibrate in a specific sequence and duration based on therapeutic research findings. The goal is to provide a cost-effective and wearable solution for individuals affected by Parkinson's disease.

## Features:

- Utilizes Arduino Nano for control.
- Employs 4 coin motors for vibrotherapy.
- Customizable vibration patterns and duration.
- Low power requirements for portability and convenience.

## Code:
The provided Arduino sketch controls the coin motors, allowing for customizable vibration sequences and durations.

## Hardware:
The main components of the project include an Arduino Nano, coin vibration motors, and a power supply. The repository contains information about the circuit used and how to build the glove. 

Please note, while the project's aim is therapeutic, it does not replace professional medical advice. Always consult with a healthcare provider for treatment options. 

Contributions to improve the design or the code are very welcome.

## Future work:
- Adding wireless control to customize vibration patterns.
- Including sensors to monitor effectiveness.
- Improving power management for longer battery life.

---

## Code with detailed description:

#include <SoftwareSerial.h>
#include <ArduinoJson.h>

SoftwareSerial BTSerial(10, 11); // Bluetooth module connection (TX, RX)

const int motorPins[8] = {2, 3, 4, 5, 6, 7, 8, 9}; // Motor control pins
const int numMotors = sizeof(motorPins) / sizeof(motorPins[0]); // Number of motors

int vibrationDuration = 166; // Duration of motor vibration in milliseconds

int sequence[] = {1, 3, 2, 4, 1, 3, 2, 4}; // Default motor activation sequence
int seqLen = sizeof(sequence) / sizeof(sequence[0]); // Length of the sequence

long lastMotorStart = 0; // Timestamp of the last motor activation
int currentMotor = 0; // Index of the current motor in the sequence
bool defaultPatternActive = false; // Flag for default pattern activation
bool countdownActive = true; // Flag for countdown activation
unsigned long countdownStartTime = 0; // Start time of the countdown
const unsigned long countdownDuration = 8000; // Duration of the countdown in milliseconds

void setup() {
  for (int i = 0; i < numMotors; i++) {
    pinMode(motorPins[i], OUTPUT); // Set motor control pins as OUTPUT
  }

  BTSerial.begin(9600); // Start serial communication with Bluetooth module
  Serial.begin(9600); // Start serial communication with the computer (for debugging)

  countdownStartTime = millis(); // Record the start time of the countdown

  defaultPatternActive = true; // Activate the default pattern
}

void loop() {
  if (countdownActive) {
    unsigned long elapsedTime = millis() - countdownStartTime; // Calculate elapsed time since the countdown started

    if (elapsedTime < countdownDuration) {
      int motorNumber = elapsedTime / 1000; // Determine the motor number based on the elapsed time

      if (motorNumber >= 0 && motorNumber < numMotors) {
        digitalWrite(motorPins[motorNumber], HIGH); // Activate the current motor

        if (motorNumber > 0) {
          digitalWrite(motorPins[motorNumber - 1], LOW); // Deactivate the previous motor
        }
      }
    } else {
      countdownActive = false; // Finish the countdown
    }
  }

  if (!countdownActive && defaultPatternActive && millis() - lastMotorStart >= vibrationDuration) {
    if (currentMotor > 0) {
      digitalWrite(motorPins[sequence[currentMotor - 1] - 1], LOW); // Deactivate the previous motor
    }

    digitalWrite(motorPins[sequence[currentMotor] - 1], HIGH); // Activate the current motor
    lastMotorStart = millis(); // Record the timestamp of the current motor activation

    currentMotor = (currentMotor + 1) % seqLen; // Move to the next motor in the sequence
    if (currentMotor == 0) {
      currentMotor = 1; // Reset the current motor index if all motors have been activated once
    }
  }

  if (BTSerial.available()) {
    defaultPatternActive = false; // Disable the default pattern when receiving Bluetooth commands

    String message = BTSerial.readString(); // Read the incoming message from Bluetooth

    DynamicJsonDocument jsonDoc(256); // Create a JSON document to store the parsed data
    DeserializationError jsonError = deserializeJson(jsonDoc, message); // Parse the JSON message

    if (jsonError == DeserializationError::Ok) {
      int motorNumber = jsonDoc["motor"].as<int>(); // Extract the motor number from the JSON message
      String command = jsonDoc["command"].as<String>(); // Extract the command from the JSON message

      if (command == "start") {
        digitalWrite(motorPins[motorNumber - 1], HIGH); // Activate the specified motor
        Serial.print("Motor ");
        Serial.print(motorNumber);
        Serial.println(" started");
      } else if (command == "stop") {
        digitalWrite(motorPins[motorNumber - 1], LOW); // Deactivate the specified motor
        Serial.print("Motor ");
        Serial.print(motorNumber);
        Serial.println(" stopped");
      } else if (command == "duration") {
        int newDuration = jsonDoc["duration"].as<int>(); // Extract the new duration from the JSON message
        if (newDuration > 0 && newDuration <= 5000) {
          vibrationDuration = newDuration; // Update the vibration duration
          Serial.print("New vibration duration: ");
          Serial.println(vibrationDuration);
        }
      }
    } else {
      Serial.println("Failed to parse JSON message");
    }
  }
}

Parts list

Arduino Nano
Coin Vibration Motors
Jumper Wires
Power Source (e.g., USB cable, battery pack, or power adapter)
Glove or Wearable Platform
Resistors (if necessary)
Diodes (if necessary)
Motor Driver (if necessary)



