# Gesture-Controlled Robot
This project is about controlling a robot car with gestures from your wrist. There is a robotic car connnected to its controller via bluetooth, which uses an accelerometer to control movement with hand gestures, all managed by an Arduino Nano and Uno system. My project had lots of complex wiring and some coding, which were tough to get right, but I kept working through the problems with the help of my instructors. So far, I have managed to complete my base project, and am now working on modifications!

You should comment out all portions of your portfolio that you have not completed yet, as well as any instructions:
```html
In Progress:
- Third milestone
- Headstone image
- Schematics
- Modifications
- Bill of Materials
- Other resources
```

| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|
| Jash D | Dublin High School | Mechanical Engineering | Incoming Senior

**Replace the BlueStamp logo below with an image of yourself and your completed project. Follow the guide [here](https://tomcam.github.io/least-github-pages/adding-images-github-pages-site.html) if you need help.**

![Headstone Image](logo.svg)
  
# Final Milestone

<iframe width="1811" height="891" src="https://www.youtube.com/embed/dQw4w9WgXcQ" title="Rick Astley - Never Gonna Give You Up (Official Video) (4K Remaster)" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

For your final milestone, explain the outcome of your project. Key details to include are:
- What you've accomplished since your previous milestone
- What your biggest challenges and triumphs were at BSE
- A summary of key topics you learned about
- What you hope to learn in the future after everything you've learned at BSE



# Second Milestone

<iframe width="1811" height="891" src="https://www.youtube.com/embed/ykYTbxI2UQA" title="Jash D Milestone 2" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Since my first milestone, I integrated gesture control and bluetooth communication into my project, which means that I have completed my base project. I had to sync the bluetooth modules by wiring everything correctly (especially the TX and RX pins), and setting them up using serial commands to assign the Uno to be the slave and the Nano as the master. The nano is the controller, and it sends movement commands based on the accelerometer input data to the Uno on the car via bluetooth, which controls the car's motors. Also, I realized that just one 9V power supply was not enough to meet the demands of the motors, so everything would start heating up, which is why I decided to use two 9V batteries, each one for two motors. There were some programming and wiring issues which Josh helped me fix. Before the final milestone, I am planning on adding a speedometer and a 7-segment display as my modification, and I need to organize everything on the car since everything is currently a mess.



# First Milestone

<iframe width="1811" height="891" src="https://www.youtube.com/embed/YTKSOw8SZiI" title="Jash D Milestone 1" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

So far, I have made progress on my project by completing the car portion, which I can control by uploading code to the Arduino Uno to make it move as I want, but it’s not yet remotely controlled. The setup uses an Arduino Uno to send commands to two motor controllers (H-bridges) which power two motors each for the front and rear motor pairs, all running on a single 9-volt battery. I faced some challenges, like bootloader issues in the Arduino IDE, a faulty rear motor, and the power supply overheating because the motors drew too much current, but my instructors helped me solve these problems. My next steps are to add a Bluetooth module to the car on a breadboard to connect a the car to a motor controller with another bluetooth module on it, using an Arduino Nano with an accelerometer.

# Schematics 
My schematic diagrams are currently in progress.

# Code
Here is my final code for my gesture-controlled robot, after adding the ultrasonic sensor:

## **Code for Nano:**

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

## **Code for Uno:**

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
| Item Name | Why? | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| Item Name | Why? | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |
| Item Name | Why? | $Price | <a href="https://www.amazon.com/Arduino-A000066-ARDUINO-UNO-R3/dp/B008GRTSV6/"> Link </a> |

# Other Resources/Examples

- [Example 1](https://trashytuber.github.io/YimingJiaBlueStamp/)
- [Example 2](https://sviatil0.github.io/Sviatoslav_BSE/)
- [Example 3](https://arneshkumar.github.io/arneshbluestamp/)
