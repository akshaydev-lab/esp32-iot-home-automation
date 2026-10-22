# esp32-iot-home-automation
# ESP32-Based IoT Home Automation System

A prototype exploring home automation and safety monitoring using an ESP32, relay-based appliance control, and environmental sensors.

## Project Overview

This project brings together home appliance control and basic environmental monitoring. The ESP32 acts as the main controller, while a 4-channel relay module is used for switching connected loads. A DHT sensor is intended for temperature/humidity readings, and an MQ-2 sensor is used for gas/smoke detection experiments.

The goal is to explore how connected electronics can support convenient appliance control and provide alerts for potentially unsafe conditions.

## Components Discussed

- ESP32 development board
- 4-channel relay module
- DHT temperature/humidity sensor
- MQ-2 gas/smoke sensor
- Motor/fan loads for demonstration
- Buzzer for alerts
- Exhaust fan control
- Additional solar panel / solar-tracking elements, if included in the version being documented

> Confirm the exact components in your final build before treating this list as a complete bill of materials. Solar-tracking hardware may be a separate module or project version.

## Intended / Reported Features

- Relay-based control of connected appliances
- Temperature and humidity sensing
- MQ-2 sensor-based gas/smoke detection experiments
- Buzzer alert for selected conditions
- Exhaust fan control
- ESP32-based wireless connectivity

Features such as automatic gas shutoff, emergency calling, cloud dashboard integration, or solar tracking should only be described as completed if they were implemented and tested in the version you publish.

## System Overview

1. Sensors provide environmental readings to the ESP32.
2. The ESP32 processes sensor readings and control commands.
3. Relay outputs switch compatible demonstration loads.
4. A buzzer may provide an alert when a configured threshold is reached.

Exact wiring, thresholds, app/dashboard setup, and control logic depend on the final implementation and should be added after checking the working circuit and source code.

## Safety Notes

- Do not connect mains-voltage appliances unless the circuit is properly isolated, enclosed, fused, and reviewed by a qualified person.
- MQ-2 readings are not a certified gas-safety system and should not be relied on as the only protection against gas leaks or fire.
- Test with safe, low-voltage loads where possible.
- Never publish Wi-Fi passwords, API keys, or other credentials in this repository.

## Repository Structure

```text
esp32-iot-home-automation/
├── README.md
├── src/       # Add your actual ESP32 source code
├── docs/      # Add verified wiring diagrams and notes
└── images/    # Add real project photos when available
```

## Current Documentation Status

This README is a starting point based on the project details currently available. Source code, verified circuit diagrams, test results, and project photos have not been included yet.

## Future Improvements

- Add a verified circuit diagram and pin mapping
- Document the final source code and setup steps
- Record sensor calibration and test results
- Add a dashboard/app guide if used in the final build
- Improve enclosure, electrical isolation, and fail-safe behavior

## License

No license selected yet. Choose a license before inviting others to reuse or redistribute the project.
ESP32 source code:
/**********************************************************************************
 *  TITLE: Arduino IoT Cloud + IR + Manual (Switch) control 4 Relays with DHT11 & Real-Time feedback using ESP32
 *  Click on the following links to learn more. 
 *  YouTube Video: https://youtu.be/UlT72MMbPu8
 *  Related Blog : 
 *  by Tech StudyCell
 *  Preferences--> Aditional boards Manager URLs : 
 *  https://dl.espressif.com/dl/package_esp32_index.json, http://arduino.esp8266.com/stable/package_esp8266com_index.json
 *  
 *  Download Board ESP32 : https://github.com/espressif/arduino-esp32
 *  Download the libraries
 *  ArduinoIoTCloud Library (Version 2.0.1) with all the dependencies: https://github.com/arduino-libraries/ArduinoIoTCloud
 *  IRremote Library (Version 3.3.0): https://github.com/Arduino-IRremote/Arduino-IRremote
 *  DHT Library (Version 1.4.6): https://github.com/adafruit/DHT-sensor-library
 **********************************************************************************/

#include <ArduinoIoTCloud.h>
#include <Arduino_ConnectionHandler.h>

#include <IRremote.h>

const char DEVICE_LOGIN_NAME[]  = "8e94ed46-baf4-4b57-8f12-92ac0bea1ced"; //Enter DEVICE ID

const char SSID[]               = "Akshaywifi";    //Enter WiFi SSID (name)
const char PASS[]               = "akshay172";    //Enter WiFi password
const char DEVICE_KEY[]         = "?AcGBrNdjw9B4dlA7wyuVeFRE";    //Enter Secret device password (Secret Key)


#define IR_RECV_PIN         35 //D35 pin connected with IR Receiver IC

// define the GPIO connected with Relays and switches
#define RelayPin1 4  //D4
#define RelayPin2 5  //D5
#define RelayPin3 14  //D14
#define RelayPin4 18 //D18

#define SwitchPin1 13  //D13
#define SwitchPin2 12  //D12
#define SwitchPin3 19  //D19
#define SwitchPin4 27  //D27

#define wifiLed    2   //D2

// Uncomment whatever type you're using!

//#define DHTTYPE DHT22   // DHT 22, AM2302, AM2321
//#define DHTTYPE DHT21   // DHT 21, AM2301


IRrecv irrecv(IR_RECV_PIN);
decode_results results;

int toggleState_1 = 0; //Define integer to remember the toggle state for relay 1
int toggleState_2 = 0; //Define integer to remember the toggle state for relay 2
int toggleState_3 = 0; //Define integer to remember the toggle state for relay 3
int toggleState_4 = 0; //Define integer to remember the toggle state for relay 4

// Switch State
bool SwitchState_1 = LOW;
bool SwitchState_2 = LOW;
bool SwitchState_3 = LOW;
bool SwitchState_4 = LOW;



int   reconnectFlag = 0;

void onexustChange();
void onfanChange();
void onlightChange();
void onmotorChange();

CloudSwitch exust;
CloudSwitch fan;
CloudSwitch light;
CloudSwitch motor;


void initProperties(){

  ArduinoCloud.setBoardId(DEVICE_LOGIN_NAME);
  ArduinoCloud.setSecretDeviceKey(DEVICE_KEY);
  ArduinoCloud.addProperty(exust, READWRITE, ON_CHANGE, onexustChange);
  ArduinoCloud.addProperty(fan, READWRITE, ON_CHANGE, onfanChange);
  ArduinoCloud.addProperty(light, READWRITE, ON_CHANGE, onlightChange);
  ArduinoCloud.addProperty(motor, READWRITE, ON_CHANGE, onmotorChange);
  
}

WiFiConnectionHandler ArduinoIoTPreferredConnection(SSID, PASS);



     
 
  
 





void ir_remote_control(){
  if (irrecv.decode(&results)) {
    
      switch(results.value){
          case 0x80BF49B6:  relayOnOff(1); exust  = toggleState_1; break; //update the HEX-code
          case 0x80BFC936:  relayOnOff(2); fan = toggleState_2; break; //update the HEX-code
          case 0x80BF33CC:  relayOnOff(3); light = toggleState_3; break; //update the HEX-code
          case 0x80BF718E:  relayOnOff(4); motor = toggleState_4; break; //update the HEX-code
          default : break;         
        }   
        //Serial.println(results.value, HEX);    
        irrecv.resume();   
  } 
}

void relayOnOff(int relay) {

  switch (relay) {
    case 1:
      if (toggleState_1 == 0) {
        digitalWrite(RelayPin1, LOW); // turn on relay 1
        toggleState_1 = 1;
        Serial.println("Device1 ON");
      }
      else {
        digitalWrite(RelayPin1, HIGH); // turn off relay 1
        toggleState_1 = 0;
        Serial.println("Device1 OFF");
      }
      delay(100);
      break;
    case 2:
      if (toggleState_2 == 0) {
        digitalWrite(RelayPin2, LOW); // turn on relay 2
        toggleState_2 = 1;
        Serial.println("Device2 ON");
      }
      else {
        digitalWrite(RelayPin2, HIGH); // turn off relay 2
        toggleState_2 = 0;
        Serial.println("Device2 OFF");
      }
      delay(100);
      break;
    case 3:
      if (toggleState_3 == 0) {
        digitalWrite(RelayPin3, LOW); // turn on relay 3
        toggleState_3 = 1;
        Serial.println("Device3 ON");
      } else {
        digitalWrite(RelayPin3, HIGH); // turn off relay 3
        toggleState_3 = 0;
        Serial.println("Device3 OFF");
      }
      delay(100);
      break;
    case 4:
      if (toggleState_4 == 0) {
        digitalWrite(RelayPin4, LOW); // turn on relay 4
        toggleState_4 = 1;
        Serial.println("Device4 ON");
      }
      else {
        digitalWrite(RelayPin4, HIGH); // turn off relay 4
        toggleState_4 = 0;
        Serial.println("Device4 OFF");
      }
      delay(100);
      break;
    default : break;
  }
}

void manual_control()
{
  if (digitalRead(SwitchPin1) == LOW && SwitchState_1 == LOW) {
    digitalWrite(RelayPin1, LOW);
    toggleState_1 = 1;
    SwitchState_1 = HIGH;
    exust = toggleState_1;
    Serial.println("Switch-1 on");
  }
  if (digitalRead(SwitchPin1) == HIGH && SwitchState_1 == HIGH) {
    digitalWrite(RelayPin1, HIGH);
    toggleState_1 = 0;
    SwitchState_1 = LOW;
    exust = toggleState_1;
    Serial.println("Switch-1 off");
  }
  if (digitalRead(SwitchPin2) == LOW && SwitchState_2 == LOW) {
    digitalWrite(RelayPin2, LOW);
    toggleState_2 = 1;
    SwitchState_2 = HIGH;
    fan = toggleState_2;
    Serial.println("Switch-2 on");
  }
  if (digitalRead(SwitchPin2) == HIGH && SwitchState_2 == HIGH) {
    digitalWrite(RelayPin2, HIGH);
    toggleState_2 = 0;
    SwitchState_2 = LOW;
    fan = toggleState_2;
    Serial.println("Switch-2 off");
  }
  if (digitalRead(SwitchPin3) == LOW && SwitchState_3 == LOW) {
    digitalWrite(RelayPin3, LOW);
    toggleState_3 = 1;
    SwitchState_3 = HIGH;
    light = toggleState_3;
    Serial.println("Switch-3 on");
  }
  if (digitalRead(SwitchPin3) == HIGH && SwitchState_3 == HIGH) {
    digitalWrite(RelayPin3, HIGH);
    toggleState_3 = 0;
    SwitchState_3 = LOW;
    light = toggleState_3;
    Serial.println("Switch-3 off");
  }
  if (digitalRead(SwitchPin4) == LOW && SwitchState_4 == LOW) {
    digitalWrite(RelayPin4, LOW);
    toggleState_4 = 1;
    SwitchState_4 = HIGH;
    motor = toggleState_4;
    Serial.println("Switch-4 on");
  }
  if (digitalRead(SwitchPin4) == HIGH && SwitchState_4 == HIGH) {
    digitalWrite(RelayPin4, HIGH);
    toggleState_4 = 0;
    SwitchState_4 = LOW;
    motor = toggleState_4;
    Serial.println("Switch-4 off");
  }
}  

void doThisOnConnect(){
  /* add your custom code here */
  Serial.println("Board successfully connected to Arduino IoT Cloud");
  digitalWrite(wifiLed, HIGH); //Turn off WiFi LED
}
void doThisOnSync(){
  /* add your custom code here */
  Serial.println("Thing Properties synchronised");
}

void doThisOnDisconnect(){
  /* add your custom code here */
  Serial.println("Board disconnected from Arduino IoT Cloud");
  digitalWrite(wifiLed, LOW); //Turn off WiFi LED
}

void setup() {
  // Initialize serial and wait for port to open:
  Serial.begin(9600);
  // This delay gives the chance to wait for a Serial Monitor without blocking if none is found
  delay(1500);

  // Defined in thingProperties.h
  initProperties();

  irrecv.enableIRIn(); // Start the receiver
  // Connect to Arduino IoT Cloud
  ArduinoCloud.begin(ArduinoIoTPreferredConnection);

  ArduinoCloud.addCallback(ArduinoIoTCloudEvent::CONNECT, doThisOnConnect);
  ArduinoCloud.addCallback(ArduinoIoTCloudEvent::SYNC, doThisOnSync);
  ArduinoCloud.addCallback(ArduinoIoTCloudEvent::DISCONNECT, doThisOnDisconnect);

  setDebugMessageLevel(2);
  ArduinoCloud.printDebugInfo();

  pinMode(RelayPin1, OUTPUT);
  pinMode(RelayPin2, OUTPUT);
  pinMode(RelayPin3, OUTPUT);
  pinMode(RelayPin4, OUTPUT);

  pinMode(wifiLed, OUTPUT);

  pinMode(SwitchPin1, INPUT_PULLUP);
  pinMode(SwitchPin2, INPUT_PULLUP);
  pinMode(SwitchPin3, INPUT_PULLUP);
  pinMode(SwitchPin4, INPUT_PULLUP);

  //During Starting all Relays should TURN OFF
  digitalWrite(RelayPin1, HIGH);
  digitalWrite(RelayPin2, HIGH);
  digitalWrite(RelayPin3, HIGH);
  digitalWrite(RelayPin4, HIGH);
}

void loop() {

  ArduinoCloud.update();
  
  manual_control();     //Manual Control
  ir_remote_control();  //IR Remote Control
          
}

void onexustChange() {
  //Control the device
  if (exust == 1)
  {
    digitalWrite(RelayPin1, LOW);
    Serial.println("Device1 ON");
    toggleState_1 = 1;
  }
  else
  {
    digitalWrite(RelayPin1, HIGH);
    Serial.println("Device1 OFF");
    toggleState_1 = 0;
  }
}

void onfanChange() {
  if (fan == 1)
  {
    digitalWrite(RelayPin2, LOW);
    Serial.println("Device2 ON");
    toggleState_2 = 1;
  }
  else
  {
    digitalWrite(RelayPin2, HIGH);
    Serial.println("Device2 OFF");
    toggleState_2 = 0;
  }
}

void onlightChange() {
  if (light == 1)
  {
    digitalWrite(RelayPin3, LOW);
    Serial.println("Device3 ON");
    toggleState_3 = 1;
  }
  else
  {
    digitalWrite(RelayPin3, HIGH);
    Serial.println("Device3 OFF");
    toggleState_3 = 0;
  }
}

void onmotorChange() {
  if (motor == 1)
  {
    digitalWrite(RelayPin4, LOW);
    Serial.println("Device4 ON");
    toggleState_4 = 1;
  }
  else
  {
    digitalWrite(RelayPin4, HIGH);
    Serial.println("Device4 OFF");
    toggleState_4 = 0;
  }
}

