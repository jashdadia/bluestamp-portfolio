# Gesture-Controlled Robotic Car
This project is about controlling a robot car with gestures from your wrist. There is a robotic car connnected to its controller via bluetooth, which uses an accelerometer to control movement with hand gestures, all managed by an Arduino Nano and Uno system. My project had lots of complex wiring and some coding, which were tough to get right, but I kept working through the problems with the help of my instructors.


| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|
| Jash D | Dublin High School | Mechanical Engineering | Incoming Senior

<img src = "https://jashdadia.github.io/bluestamp-portfolio/headstone-image.jpeg" width = "400px">

# Final Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/KwhL4cRRagQ" title="Jash D FInal Milestone" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Since my second milestone, I have improved my gesture controlled robotic car by adding a cardboard payload platform mounted with Velcro straps as well as an ultrasonic sensor to detect obstacles within 20 centimeters, instead of the speedometer idea I had last week. The biggest challenges I faced were wiring and programming issues, which I am very glad my instructors helped me get through. They taught me major topics such as connecting HC05 Bluetooth modules using AT commands, using advanced commands from the SoftwareSerial library, as well as using voltage dividers. In the future, I want to be able to use PCBs and soldering irons to make my finished product more polished.



# Second Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/ykYTbxI2UQA" title="Jash D Milestone 2" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Since my first milestone, I integrated gesture control and bluetooth communication into my project, which means that I have completed my base project. I had to sync the bluetooth modules by wiring everything correctly (especially the TX and RX pins), and setting them up using serial commands to assign the Uno to be the slave and the Nano as the master. The nano is the controller, and it sends movement commands based on the accelerometer input data to the Uno on the car via bluetooth, which controls the car's motors. Also, I realized that just one 9V power supply was not enough to meet the demands of the motors, so everything would start heating up, which is why I decided to use two 9V batteries, each one for two motors. There were some programming and wiring issues which Josh helped me fix. Before the final milestone, I am planning on adding a speedometer and a 7-segment display as my modification, and I need to organize everything on the car since everything is currently a mess.



# First Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/YTKSOw8SZiI" title="Jash D Milestone 1" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

So far, I have made progress on my project by completing the car portion, which I can control by uploading code to the Arduino Uno to make it move as I want, but it’s not yet remotely controlled. The setup uses an Arduino Uno to send commands to two motor controllers (H-bridges) which power two motors each for the front and rear motor pairs, all running on a single 9-volt battery. I faced some challenges, like bootloader issues in the Arduino IDE, a faulty rear motor, and the power supply overheating because the motors drew too much current, but my instructors helped me solve these problems. My next steps are to add a Bluetooth module to the car on a breadboard to connect a the car to a motor controller with another bluetooth module on it, using an Arduino Nano with an accelerometer.

# Schematics 

## **Schematic Diagram for Car:**

<img src = "https://jashdadia.github.io/bluestamp-portfolio/car-schematic.png" width = "600px">

## **Schematic Diagram for Controller:**

<img src = "https://jashdadia.github.io/bluestamp-portfolio/controller-schematic.png" width = "600px">

# Code
Here is my final code for my gesture-controlled robot, after adding the ultrasonic sensor:

## **Code for Nano (Controller module):**

```c++
#include <SoftwareSerial.h>
#include <Wire.h>

SoftwareSerial BT_Serial(3, 2);
const int MPU = 0x68;
int16_t AcX, AcY, AcZ;
int flag = 0;
char command = 'S';

void setup() {
  Serial.begin(38400);
  BT_Serial.begin(38400);

  Wire.begin();
  Wire.beginTransmission(MPU);
  Wire.write(0x6B);
  Wire.write(0);
  if (Wire.endTransmission(true) != 0) {
    Serial.println("accelerometer not detected");
    while (1);
  }
  Serial.println("controller ready");
  delay(500);
}

void loop() {
  Read_accelerometer();
  determineGesture();
  delay(100);
}

void Read_accelerometer() {
  Wire.beginTransmission(MPU);
  Wire.write(0x3B);
  if (Wire.endTransmission(false) != 0) {
    Serial.println("accelerometer not connecting");
    return;
  }
  Wire.requestFrom(MPU, 6, true);
  if (Wire.available() == 6) {
    AcX = Wire.read() << 8 | Wire.read();
    AcY = Wire.read() << 8 | Wire.read();
    AcZ = Wire.read() << 8 | Wire.read();

    AcX = map(AcX, -32768, 32767, 0, 180);
    AcY = map(AcY, -32768, 32767, 0, 180);
    AcZ = map(AcZ, -32768, 32767, 0, 180);

    Serial.print("X: "); Serial.print(AcX);
    Serial.print(" Y: "); Serial.print(AcY);
    Serial.print(" Z: "); Serial.println(AcZ);
  } else {
    Serial.println("accelerometer not reading");
  }
}

void determineGesture() {

  if (AcX < 60 && flag == 0) {
    flag = 1;
    command = 'F';
    Serial.println("F");
  } else if (AcX > 130 && flag == 0) {
    flag = 1;
    command = 'B';
    Serial.println("B");
  } else if (AcY < 60 && flag == 0) {
    flag = 1;
    command = 'L';
    Serial.println("L");
  } else if (AcY > 130 && flag == 0) {
    flag = 1;
    command = 'R';
    Serial.println("R");
  } else if (AcX > 70 && AcX < 120 && AcY > 70 && AcY < 120 && flag == 1) {
    flag = 0;
    command = 'S';
    Serial.println("S");
  } else {
    return;
  }

  BT_Serial.write(command);
}
```

## **Code for Uno (Car module):**

```c++
#include <SoftwareSerial.h>
#include <NewPing.h>

#define tx 2
#define rx 3
#define trigPin 12
#define echoPin 13
#define MAX_DISTANCE 200 //200mm max

SoftwareSerial configBt(rx, tx);
NewPing sonar(trigPin, echoPin, MAX_DISTANCE);

const int B_1A_R = 4;
const int B_2A_R = 5;
const int A_1A_R = 6;
const int A_1B_R = 7;

const int B_1A_F = 11;
const int B_2A_F = 10;
const int A_1A_F = 9;
const int A_1B_F = 8;

char c = 'S';
char lastCommand = 'S';

void setup() {
  Serial.begin(38400);
  configBt.begin(38400);
  pinMode(tx, OUTPUT);
  pinMode(rx, INPUT);


  pinMode(B_1A_R, OUTPUT);
  pinMode(B_2A_R, OUTPUT);
  pinMode(A_1A_R, OUTPUT);
  pinMode(A_1B_R, OUTPUT);
  pinMode(B_1A_F, OUTPUT);
  pinMode(B_2A_F, OUTPUT);
  pinMode(A_1A_F, OUTPUT);
  pinMode(A_1B_F, OUTPUT);

  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);

  freeze();
  Serial.println("car is ready");
}

void loop() {

  unsigned int distance = sonar.ping_cm();
  if (distance == 0) distance = MAX_DISTANCE;

  if (distance < 20 && lastCommand == 'F') {
    freeze();
    lastCommand = 'S';
    Serial.println("OBSTACLE STOP");
  }

  if (configBt.available()) {
    c = configBt.read();
    Serial.println(c);

    if (distance < 20) {
      switch (c) {
        case 'B':
          backward();
          lastCommand = 'B';
          break;
        case 'L':
          left();
          lastCommand = 'L';
          break;
        case 'R':
          right();
          lastCommand = 'R';
          break;
        case 'F':
          freeze();
          lastCommand = 'S';
          Serial.println("forward blocked");
          break;
        case 'S':
          freeze();
          lastCommand = 'S';
          break;
      }
    } else {

      switch (c) {
        case 'F':
          forward();
          lastCommand = 'F';
          break;
        case 'L':
          left();
          lastCommand = 'L';
          break;
        case 'R':
          right();
          lastCommand = 'R';
          break;
        case 'B':
          backward();
          lastCommand = 'B';
          break;
        case 'S':
          freeze();
          lastCommand = 'S';
          break;
      }
    }
  }
  delay(50); 
}

void forward() {
  digitalWrite(B_1A_R, HIGH);
  digitalWrite(B_2A_R, LOW);
  digitalWrite(A_1A_R, HIGH);
  digitalWrite(A_1B_R, LOW);
  digitalWrite(B_1A_F, HIGH);
  digitalWrite(B_2A_F, LOW);
  digitalWrite(A_1A_F, HIGH);
  digitalWrite(A_1B_F, LOW);
  Serial.println("F");
}

void backward() {
  digitalWrite(B_1A_R, LOW);
  digitalWrite(B_2A_R, HIGH);
  digitalWrite(A_1A_R, LOW);
  digitalWrite(A_1B_R, HIGH);
  digitalWrite(B_1A_F, LOW);
  digitalWrite(B_2A_F, HIGH);
  digitalWrite(A_1A_F, LOW);
  digitalWrite(A_1B_F, HIGH);
  Serial.println("B");
}

void left() {
  digitalWrite(B_1A_R, HIGH);
  digitalWrite(B_2A_R, LOW);
  digitalWrite(A_1A_R, LOW);
  digitalWrite(A_1B_R, HIGH);
  digitalWrite(B_1A_F, LOW);
  digitalWrite(B_2A_F, HIGH);
  digitalWrite(A_1A_F, HIGH);
  digitalWrite(A_1B_F, LOW);
  Serial.println("L");
}

void right() {
  digitalWrite(B_1A_R, LOW);
  digitalWrite(B_2A_R, HIGH);
  digitalWrite(A_1A_R, HIGH);
  digitalWrite(A_1B_R, LOW);
  digitalWrite(B_1A_F, HIGH);
  digitalWrite(B_2A_F, LOW);
  digitalWrite(A_1A_F, LOW);
  digitalWrite(A_1B_F, HIGH);
  Serial.println("R");
}

void freeze() {
  digitalWrite(B_1A_R, LOW);
  digitalWrite(B_2A_R, LOW);
  digitalWrite(A_1A_R, LOW);
  digitalWrite(A_1B_R, LOW);
  digitalWrite(B_1A_F, LOW);
  digitalWrite(B_2A_F, LOW);
  digitalWrite(A_1A_F, LOW);
  digitalWrite(A_1B_F, LOW);
  Serial.println("S");
}
```

# Bill of Materials

| **Part** | **Note** | **Price** | **Link** |
|:--:|:--:|:--:|:--:|
| Car chassis | frame to hold components of car | $21.29 | <a href="https://www.amazon.com/perseids-Chassis-Encoder-Wheels-Battery/dp/B07DNXBFQN/ref=sr_1_10"> Link </a> |
| L9110S motor drivers | Couple the pairs of motors | $5.99 | <a href="https://www.amazon.com/Ferwooh-Stepper-Controller-2-5-12V-H-Bridge/dp/B0D17PJ2MS/ref=sr_1_1"> Link </a> |
| Elegoo Uno R3 | Logic control for robot component | $14.98 | <a href="https://www.amazon.com/ELEGOO-Board-ATmega328P-ATMEGA16U2-Compliant/dp/B01EWOE0UU/ref=sr_1_2_sspa"> Link </a> |
| Motors | Drive the movement of the car | $11.98 | <a href="https://www.amazon.com/AEDIKO-Motor-Gearbox-200RPM-Ratio/dp/B09N6NXP4H/ref=sr_1_4"> Link </a> |
| Electronics kit | Additional sensors and wiring components | $14 | <a href="https://www.amazon.com/Smraza-Electronics-Potentiometer-tie-Points-Breadboard/dp/B0B62RL725/ref=sxts_b2b_sx_reorder_acb_business"> Link </a> |
| Breadboard Kit | Additional breadboards to connect components | $8.79 | <a href="https://www.amazon.com/Breadboards-Solderless-Breadboard-Distribution-Connecting/dp/B07DL13RZH/ref=sxts_b2b_sx_reorder_acb_business"> Link </a> |
| Arduino Nano | Logic control for the hand component | $15.99 | <a href="https://www.amazon.com/LAFVIN-Board-ATmega328P-Micro-Controller-Arduino/dp/B07G99NNXL"> Link </a> |
| Micro USB cable | Upload code to the microcontrollers | $5 | <a href="https://www.amazon.com/Charging-Transfer-Android-Trustable-MYFON/dp/B098DW7485/ref=sr_1_6"> Link </a> |
| Accelerometer | Detects acceleration (gestures) of hand component  | $9 | <a href="https://www.amazon.com/Pre-Soldered-Accelerometer-Raspberry-Compatible-Arduino/dp/B0BMY15TC4/ref=sr_1_5"> Link </a> |
| HC05 Bluetooth module | Connect the microcontrolelrs between hand and robot | $9 | <a href="https://www.amazon.com/DSD-TECH-HC-05-Pass-through-Communication/dp/B01G9KSAF6/ref=sr_1_3"> Link </a> |
| Breadboard power supply | Enables a 9V alkaline battery to power breadboard | $8 | <a href="https://www.amazon.com/ALAMSCN-Solderless-Breadboard-Battery-Arduino/dp/B08JYPMCZY/ref=sxts_b2b_sx_reorder_acb_business"> Link </a> |
| 9V batteries | Supply power to components | $8.69 | <a href="https://www.amazon.com/Amazon-Basics-Performance-All-Purpose-Batteries/dp/B00MH4QM1S/ref=sr_1_5_pp"> Link </a> |
| Velcro tape | Attach payload platform to car chassis | $8 | <a href="https://www.amazon.com/Art3d-Sticky-Double-Sided-Command-Adhesive/dp/B0B58FGF8H/ref=sr_1_1_sspa"> Link </a> |
| Digital multimeter | debugging tool | $11 | <a href="https://www.amazon.com/AstroAI-Digital-Multimeter-Voltage-Tester/dp/B01ISAMUA6/ref=sxin_17_pa_sp_search_thematic_sspa"> Link </a> |

# Other Resources/Examples

- [Tristan F's BlueStamp Portfolio](https://tristanfabela.github.io/Tristan_BlueStampPortfolio/)
- [Gesture-Controlled Robot Tutorial on Hackster.io](https://www.hackster.io/embeddedlab786/hand-gesture-control-robot-via-bluetooth-94b13d)
- [HC-05 to HC-05 Bluetooth Module Connection Tutorial by Justin Miner](https://docs.google.com/document/d/1EpnEPulXQwPDSK-nKLohqPjpeXNteP2G/edit)
- [Ultrasonic Sensor Integration Guide on Arduino Forum](https://projecthub.arduino.cc/lucasfernando/ultrasonic-sensor-with-arduino-complete-guide-284faf)
