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

<iframe width="560" height="315" src="https://www.youtube.com/embed/dQw4w9WgXcQ" title="Rick Astley - Never Gonna Give You Up (Official Video) (4K Remaster)" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

For your final milestone, explain the outcome of your project. Key details to include are:
- What you've accomplished since your previous milestone
- What your biggest challenges and triumphs were at BSE
- A summary of key topics you learned about
- What you hope to learn in the future after everything you've learned at BSE



# Second Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/ykYTbxI2UQA" title="Jash D Milestone 2" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Since my first milestone, I integrated gesture control and bluetooth communication into my project, which means that I have completed my base project. I had to sync the bluetooth modules by wiring everything correctly (especially the TX and RX pins), and setting them up using serial commands to assign the Uno to be the slave and the Nano as the master. The nano is the controller, and it sends movement commands based on the accelerometer input data to the Uno on the car via bluetooth, which controls the car's motors. Also, I realized that just one 9V power supply was not enough to meet the demands of the motors, so everything would start heating up, which is why I decided to use two 9V batteries, each one for two motors. There were some programming and wiring issues which Josh helped me fix. Before the final milestone, I am planning on adding a speedometer and a 7-segment display as my modification, and I need to organize everything on the car since everything is currently a mess.



# First Milestone

<iframe width="560" height="315" src="https://www.youtube.com/embed/YTKSOw8SZiI" title="Jash D Milestone 1" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

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
| Car chassis | frame to hold components of car | $21.29 | <a href="https://www.amazon.com/perseids-Chassis-Encoder-Wheels-Battery/dp/B07DNXBFQN/ref=sr_1_10?crid=26TUUFVPI4E3P&dib=eyJ2IjoiMSJ9.b6A_uqlNY7c_XNSLuXfmFx3lnf3nSeLGG7KgC7ZRfC9FAK22FT6M83V1dTBEAvnkHgRE0NNpo0oADYqyV4P2HpY5BFGlLS5OXcRD4aEW4oZsKRGeNyx6VCcRs7hoENdwnlQ8hLuKGPpRNYNJns2n3xydphLJvzrAHjoARmRiwPmFpghbM1R-1qsX5oLcwUgeikl74r8tSpjraJ1ymDeFdq6Kf9PpSFMZnd112Ga4ex0Q4MCaQT605Nzcs1spfnEG27m1GZgqWH8y7CDjJa2srdlHjoSkiJWC8MTTn3ug0Zg.7oE32LVlD_UTGvu8buwQxem0Dpe5zyabMMu1Q39WiQs&dib_tag=se&keywords=robot%2Bchassis&qid=1715357415&s=toys-and-games&sprefix=robot%2Bchassi%2Ctoys-and-games%2C95&sr=1-10&th=1&qty=1&sbo=RZvfv%2F%2FHxDF%2BO5021pAnSA%3D%3D"> Link </a> |
| Screwdriver Kit | Screwdriver with bits to fasten components onto chassis | $5.94 | <a href="https://www.amazon.com/Small-Screwdriver-Set-Mini-Magnetic/dp/B08RYXKJW9/"> Link </a> |
| L9110S motor drivers | Couple the pairs of motors | $5.99 | <a href="https://www.amazon.com/Ferwooh-Stepper-Controller-2-5-12V-H-Bridge/dp/B0D17PJ2MS/ref=sr_1_1?crid=2OQ7UJ1VJLUHU&dib=eyJ2IjoiMSJ9.xPgxMG6cmxZRuRbSqT3QxSr9zBhiCzp3WpnCeGbZjJfW2wU1eHonQ9Yw7yZi2k6Q3PhHd4uR1wLLWBETfHe0SF_wYRGvOug585fW0fsZTX6ImNTMLCJR3VH7MrRlnR7uQ5g0XrAXnzyVOSTEAmuNKyuiUk_vhsIuCNv1HCMrPUyPtn7qKFCwz7vVMcvEXx5Ddy4TPQJlpbS_voU9at8F85yJM5O9Hp5bbg_xuIHUsuE2ePCbv4lATgHmgHzENtlSRiU4laurwSqTAEgEnv9gNIbmb5d2HT5qBLfNChqSyio.9Fh1mUFHx48E8QZCOAX5T2ZJxzbHHdu93PJ63MLUqpM&dib_tag=se&keywords=L9110S+DC&qid=1716940953&s=electronics&sprefix=l9110s+dc%2Celectronics%2C89&sr=1-1"> Link </a> |
| Elegeoo Uno R3 | Logic control for robot component | $14.98 | <a href="https://www.amazon.com/ELEGOO-Board-ATmega328P-ATMEGA16U2-Compliant/dp/B01EWOE0UU/ref=sr_1_2_sspa?crid=3A6NCD2X9JEMJ&dib=eyJ2IjoiMSJ9.AcWZy-Yg4mDTnhzEHozxzPZdVC5-KUL2tW-OQewDKpBB4brSpD-p4bn74WcXiW3KarYertgpNaLJ0VHKx0qsPqolKAhiz1GRG5BwJQl73cEvrlXIXNmqlpSvU7uu2aRVSwAZi9Gj2AjSPLM3esW1Gzy9xEiQ9oiR5LCNjh4MlYDx5mTm5sI4rsD4CFTipJnF572qXlickl35FRcCj8oMXQotumgqI4yEIq0HobOtIlEnNhtVB51JMBHhqtmmF_PC9WeHJ4ySUVVcv_gq3_VeG1aAEbdm4NXmmT6NOYPw4Qo.1PFdgFT22oqO5Mg6-6j_aUL_EV8tUPuaFrB5N9oaEX0&dib_tag=se&keywords=elegoo+arduino&qid=1716856465&s=electronics&sprefix=elegoo+arduino%2Celectronics%2C99&sr=1-2-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&psc=1"> Link </a> |
| Motors | Drive the movement of the car | $11.98 | <a href="https://www.amazon.com/AEDIKO-Motor-Gearbox-200RPM-Ratio/dp/B09N6NXP4H/ref=sr_1_4?crid=1JP29NIWBLH2M&dib=eyJ2IjoiMSJ9.Wq3jKgOLbqtEP772vMD4pV5f-w3PLBdEpKqguykXOb0JFO14f4Dq0m_VDVUMUFtR8WFINUEticI3GXcoGqwXPqK9yIh04PhCktgccMz9zAUiKXMJPwmOTUp_6av3XuFD0lXo9WngN9iKI6YgZrhEEs9qnqbcB1GnvgntCdKz8Q1dFuNu61NgSE6Z8vBk3FRpaNcr1lCI7FApTiNi0Qce8gbfmMn6oUggZQHpIOKKZ6s.M7WsZ_ZZtm3rm93kKgw0NOxt1McVBYX6m55oGxu1xxI&dib_tag=se&keywords=dc+motor+with+gearbox&qid=1715911706&sprefix=dc+motor+with+gearbox%2Caps%2C126&sr=8-4"> Link </a> |
| Electronics kit | Additional sensors and wiring components | $14 | <a href="https://www.amazon.com/Smraza-Electronics-Potentiometer-tie-Points-Breadboard/dp/B0B62RL725/ref=sxts_b2b_sx_reorder_acb_business?content-id=amzn1.sym.f63a3b0b-3a29-4a8e-8430-073528fe007f%3Aamzn1.sym.f63a3b0b-3a29-4a8e-8430-073528fe007f&crid=2IC3T44H3U3WG&cv_ct_cx=breadboard+kit&dib=eyJ2IjoiMSJ9.TUd5tu2T8rmms7ZuJ0UzmbtpLL1zsu93bQM0PzwnP4E.sT0V0vL_QtbYv8ymVTCcRkhFNgBtRvRiT7G4FT1oGTE&dib_tag=se&keywords=breadboard+kit&pd_rd_i=B0B62RL725&pd_rd_r=67e1f4ff-e3b9-44e4-b441-b4ae282f036b&pd_rd_w=UjFaP&pd_rd_wg=0xRoC&pf_rd_p=f63a3b0b-3a29-4a8e-8430-073528fe007f&pf_rd_r=BFGP77H27ZN31W4PZAW6&qid=1715911733&sbo=RZvfv%2F%2FHxDF%2BO5021pAnSA%3D%3D&sprefix=breadboard+kit%2Caps%2C109&sr=1-2-9f062ed5-8905-4cb9-ad7c-6ce62808241a"> Link </a> |
| Breadboard Kit | Additional breadboards to connect component | $8.79 | <a href="https://www.amazon.com/Breadboards-Solderless-Breadboard-Distribution-Connecting/dp/B07DL13RZH/ref=sxts_b2b_sx_reorder_acb_business?content-id=amzn1.sym.f63a3b0b-3a29-4a8e-8430-073528fe007f%3Aamzn1.sym.f63a3b0b-3a29-4a8e-8430-073528fe007f&crid=1RAL6PA1TZ81Q&cv_ct_cx=breadboard+kit&dib=eyJ2IjoiMSJ9.TUd5tu2T8rmms7ZuJ0UzmbtpLL1zsu93bQM0PzwnP4E.sT0V0vL_QtbYv8ymVTCcRkhFNgBtRvRiT7G4FT1oGTE&dib_tag=se&keywords=breadboard+kit&pd_rd_i=B07DL13RZH&pd_rd_r=1e3e6f57-5578-4452-b230-90d43c79b5d3&pd_rd_w=rFN6B&pd_rd_wg=3mMuA&pf_rd_p=f63a3b0b-3a29-4a8e-8430-073528fe007f&pf_rd_r=JC9D7T4VYRDQ9HJVY5X8&qid=1715912837&s=electronics&sbo=RZvfv%2F%2FHxDF%2BO5021pAnSA%3D%3D&sprefix=breadboard+kit%2Celectronics%2C102&sr=1-1-9f062ed5-8905-4cb9-ad7c-6ce62808241a"> Link </a> |
| Arduino Micro | Logic control for the hand component | $20 | <a href="https://www.amazon.com/Arduino-Micro-Headers-A000053-Controller/dp/B00AFY2S56/ref=sxts_b2b_sx_reorder_acb_business?content-id=amzn1.sym.44ecadb3-1930-4ae5-8e7f-c0670e7d86ce%3Aamzn1.sym.44ecadb3-1930-4ae5-8e7f-c0670e7d86ce&cv_ct_cx=arduino%2Bmicro&keywords=arduino%2Bmicro&pd_rd_i=B00AFY2S56&pd_rd_r=3c265d26-c144-45b4-b645-a19f57187069&pd_rd_w=ZWCox&pd_rd_wg=dgTyS&pf_rd_p=44ecadb3-1930-4ae5-8e7f-c0670e7d86ce&pf_rd_r=SRN3W01Y55A8M3VF2PXJ&qid=1686186926&sbo=RZvfv%2F%2FHxDF%2BO5021pAnSA%3D%3D&sr=1-1-62d64017-76a9-4f2a-8002-d7ec97456eea&th=1"> Link </a> |
| Micro USB cable | Upload code to the microcontrollers | $5 | <a href="https://www.amazon.com/Charging-Transfer-Android-Trustable-MYFON/dp/B098DW7485/ref=sr_1_6?crid=3USJU0DMSZB2S&keywords=micro+usb&qid=1686187078&s=electronics&sprefix=micro+usb%2Celectronics%2C106&sr=1-6"> Link </a> |
| Accelerometer | Detects acceleration(gestures) of hand component  | $9 | <a href="https://www.amazon.com/Pre-Soldered-Accelerometer-Raspberry-Compatible-Arduino/dp/B0BMY15TC4/ref=sr_1_5?crid=8EDYBVQQY7E2&dib=eyJ2IjoiMSJ9.ID40hq0zMYWtG7Um61yZ63xnujgA2opJZN4n7Ear4a7PVz0kChoZQvMielgIQHXUTy4_yuQvwgK7S5aC7H8U6s5ChRMOd0Iba7IZDg_ySpKnO5uemH-09l_GS1vcaiACgMnHA4JltsdzdfsSBwKgUFAhFhLuvIKnY6G3lrVGfynAdqGHpq4kg53C83MmKTRP8583zcZvMNE8N9pGZr9m2_ctic429UEwmpvof0hrhug.bBXCol9-0Y3cd8LQBcW01jRrDORIYOXF6HAJOn6LUjY&dib_tag=se&keywords=accelerometer+arduino&qid=1715912788&sprefix=accelerometer+arduino%2Caps%2C110&sr=8-5"> Link </a> |
| HC05 Bluetooth module | Connect the microcontrolelrs between hand and robot | $9 | <a href="https://www.amazon.com/DSD-TECH-HC-05-Pass-through-Communication/dp/B01G9KSAF6/ref=sr_1_3?crid=2J833J7AYQJA&keywords=hc05&qid=1686187263&sprefix=hc0%2Caps%2C112&sr=8-3"> Link </a> |
| Breadboard power supply | Enables a 9V alkaline battery to power breadboard | $8 | <a href="https://www.amazon.com/ALAMSCN-Solderless-Breadboard-Battery-Arduino/dp/B08JYPMCZY/ref=sxts_b2b_sx_reorder_acb_business?content-id=amzn1.sym.f63a3b0b-3a29-4a8e-8430-073528fe007f%3Aamzn1.sym.f63a3b0b-3a29-4a8e-8430-073528fe007f&crid=Z2S8NZU0KN1S&cv_ct_cx=breadboard+power+supply&dib=eyJ2IjoiMSJ9.nJ_euybTOUu9E6yyDpnEqg.NgztCYPGkG96eXyyFxpvxOVw5ykdTUq6oziUQnvf51E&dib_tag=se&keywords=breadboard+power+supply&pd_rd_i=B08JYPMCZY&pd_rd_r=f2beb6df-6d77-44a3-8b72-83255f19ca20&pd_rd_w=r1wmq&pd_rd_wg=ToFNq&pf_rd_p=f63a3b0b-3a29-4a8e-8430-073528fe007f&pf_rd_r=R5ZMMGW4CXRBP3PWAYMA&qid=1715912515&s=electronics&sbo=RZvfv%2F%2FHxDF%2BO5021pAnSA%3D%3D&sprefix=breadboard+power+s%2Celectronics%2C114&sr=1-1-9f062ed5-8905-4cb9-ad7c-6ce62808241a"> Link </a> |
| 9V batteries | Supply power to components | $8.69 | <a href="https://www.amazon.com/Amazon-Basics-Performance-All-Purpose-Batteries/dp/B00MH4QM1S/ref=sr_1_5_pp?crid=3TQ7ANPH958JM&dib=eyJ2IjoiMSJ9.bmcV2Upj_vpB6G9CFlPPxYAryat512da7ekZjc52HecXSTmtx7PbJ50EgQFPCMqlAxjOUq-tL4vQTpozlHvH89bMwx-HJoyGcdz6EY8HrMxahTiqOXkoP7ewkDcgHoMhmHamdlQfW6FBHO0Gm-DYZZnnMuvEU3qOpemA8PGEvRhEx4-lGaBZhrvls039G1-9SizAW-YRGXZ2fFrdVDlREyyOhAuxXZaE5QqUxWesRQgP9UfGOYaInRWTTPwhDbXFa-RPzGbU1C_u4wq-NMqKBtWEQqR9-cA8O3FYOx3icEY.dtKJmI2T-iCmMM_bYnbiHUWzhKpJDRxS-bBmZIwYFKM&dib_tag=se&keywords=9v+batteries&qid=1720651326&rdc=1&s=electronics&sprefix=9v+batteries%2Celectronics%2C105&sr=1-5"> Link </a> |
| Velcro tape | attach components to hand component | $8 | <a href="https://www.amazon.com/Art3d-Sticky-Double-Sided-Command-Adhesive/dp/B0B58FGF8H/ref=sr_1_1_sspa?crid=2N0JOMEZLJ2DS&dib=eyJ2IjoiMSJ9.qGUGB_MXfmbL0MW7bqNJbxvZC9pzliDJ9KYyRNNrctnh03kCcUXONRrcPYdGeo7Jwzrm83HyF8Jsb1RkcdlLPAw-8RkxbTCMiW6UI1Fpnjv9GjXUg9VBOLxmLVUbmMp5J7gFXKKLTWQ-w_L4Q9rykEUqKmjv-v6GRykMMZLY2cVt__lLxMIlwr6qBnQLWpHiklifUJwjiURxO--TTt2VReYgmN0z7118ifSucrkvRrg.mwA0L4zMSlJP2RO8IBba7dVqwa1Lkr8KvY1JmeQEfCg&dib_tag=se&keywords=velcro+tape+pieces&qid=1716734034&sprefix=velcro+tape+piece%2Caps%2C89&sr=8-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&psc=1"> Link </a> |
| Digital multimeter | debugging tool | $11 | <a href="https://www.amazon.com/AstroAI-Digital-Multimeter-Voltage-Tester/dp/B01ISAMUA6/ref=sxin_17_pa_sp_search_thematic_sspa?content-id=amzn1.sym.e8da13fc-7baf-46c3-926a-e7e8f63a520b%3Aamzn1.sym.e8da13fc-7baf-46c3-926a-e7e8f63a520b&cv_ct_cx=digital+multimeter&dib=eyJ2IjoiMSJ9.5LQumrfBR8l0mKnJCJlRg73dxpou0gqYD_ffU3srgs0Utegwth8GcQCSVXVzeZeLSJx5J3itz5TLdmJHsrVITQ.-00jRPoT-bBy26YC4LzQ-S4cYdztgmSMGb83_WEm6HY&dib_tag=se&keywords=digital+multimeter&pd_rd_i=B01ISAMUA6&pd_rd_r=e1ff2570-7e4a-4906-bc55-6f819d48d1bc&pd_rd_w=h7HgL&pd_rd_wg=0ZcFH&pf_rd_p=e8da13fc-7baf-46c3-926a-e7e8f63a520b&pf_rd_r=R6YKX3NXTDQ1PQP4H8RM&qid=1715911879&sbo=RZvfv%2F%2FHxDF%2BO5021pAnSA%3D%3D&sr=1-1-7efdef4d-9875-47e1-927f-8c2c1c47ed49-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9zZWFyY2hfdGhlbWF0aWM&psc=1"> Link </a> |

# Other Resources/Examples

- [Example 1](https://trashytuber.github.io/YimingJiaBlueStamp/)
- [Example 2](https://sviatil0.github.io/Sviatoslav_BSE/)
- [Example 3](https://arneshkumar.github.io/arneshbluestamp/)
