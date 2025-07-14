# Gesture-Controlled Robot
This project is basically about controlling a robot car with your wrist. There is a robotic car connnected to its controller via bluetooth, which uses an accelerometer to control movement with hand gestures, all managed by an Arduino Nano and Uno system. My project had lots of complex wiring and some coding, which were tough to get right, but I kept working through the problems with the help of my instructors. So far, I have managed to get the code to work with the Arduino Uno and the motor controllers, so my car does move!

You should comment out all portions of your portfolio that you have not completed yet, as well as any instructions:
```html
In Progress:
- Second and third milestones
- Headstone image
- Schematics
- Final code
- Bill of Materials
- Other resources
```

| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|
| Jash D | Dublin High School | Mechanical Engineering | Incoming Senior

**Replace the BlueStamp logo below with an image of yourself and your completed project. Follow the guide [here](https://tomcam.github.io/least-github-pages/adding-images-github-pages-site.html) if you need help.**

![Headstone Image](logo.svg)
  
# Final Milestone

<iframe width="560" height="315" src="https://www.youtube.com/watch?v=WBYdMrUd0w0" title="Final Milestone" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

For your final milestone, explain the outcome of your project. Key details to include are:
- What you've accomplished since your previous milestone
- What your biggest challenges and triumphs were at BSE
- A summary of key topics you learned about
- What you hope to learn in the future after everything you've learned at BSE



# Second Milestone

<iframe width="560" height="315" src="https://www.youtube.com/watch?v=WBYdMrUd0w0" title="Second Milestone" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

For your second milestone, explain what you've worked on since your previous milestone. You can highlight:
- Technical details of what you've accomplished and how they contribute to the final goal
- What has been surprising about the project so far
- Previous challenges you faced that you overcame
- What needs to be completed before your final milestone 

# First Milestone

<iframe width="560" height="315" src="https://www.youtube.com/watch?v=YTKSOw8SZiI" title="First Milestone" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

So far, I have made progress on my project by completing the car portion, which I can control by uploading code to the Arduino Uno to make it move as I want, but it’s not yet remotely controlled. The setup uses an Arduino Uno to send commands to two motor controllers (H-bridges) which power two motors each for the front and rear motor pairs, all running on a single 9-volt battery. I faced some challenges, like bootloader issues in the Arduino IDE, a faulty rear motor, and the power supply overheating because the motors drew too much current, but my instructors helped me solve these problems. My next steps are to add a Bluetooth module to the car on a breadboard to connect a the car to a motor controller with another bluetooth module on it, using an Arduino Nano with an accelerometer.

# Schematics 
My schematic diagrams are currently in progress.

# Code
Here is my current code prior to adding the bluetooth modules:

```c++
// rear motor controller pins
const int B_1A_R = 4;
const int B_2A_R = 5;
const int A_1A_R = 6;
const int A_1B_R = 7;

// front motor controller pins
const int B_1A_F = 11;
const int B_2A_F = 10;
const int A_1A_F = 9;
const int A_1B_F = 8;

void setup() {
  pinMode(B_1A_R, OUTPUT);
  pinMode(B_2A_R, OUTPUT);
  pinMode(A_1A_R, OUTPUT);
  pinMode(A_1B_R, OUTPUT);
  pinMode(B_1A_F, OUTPUT);
  pinMode(B_2A_F, OUTPUT);
  pinMode(A_1A_F, OUTPUT);
  pinMode(A_1B_F, OUTPUT);

  Serial.begin(9600);
  Serial.println("full movement test for milestone 1");
  delay(2000);
}

void loop() {
  Serial.println("forward");
  digitalWrite(B_1A_R, HIGH);
  digitalWrite(B_2A_R, LOW);
  digitalWrite(A_1A_R, HIGH);
  digitalWrite(A_1B_R, LOW);
  digitalWrite(B_1A_F, HIGH);
  digitalWrite(B_2A_F, LOW);
  digitalWrite(A_1A_F, HIGH);
  digitalWrite(A_1B_F, LOW);
  delay(2000);

  Serial.println("stop");
  digitalWrite(B_1A_R, LOW);
  digitalWrite(B_2A_R, LOW);
  digitalWrite(A_1A_R, LOW);
  digitalWrite(A_1B_R, LOW);
  digitalWrite(B_1A_F, LOW);
  digitalWrite(B_2A_F, LOW);
  digitalWrite(A_1A_F, LOW);
  digitalWrite(A_1B_F, LOW);
  delay(1000);

  Serial.println("left");
  digitalWrite(B_1A_R, HIGH);
  digitalWrite(B_2A_R, LOW);
  digitalWrite(A_1A_R, LOW);
  digitalWrite(A_1B_R, HIGH);
  digitalWrite(B_1A_F, LOW);
  digitalWrite(B_2A_F, HIGH);
  digitalWrite(A_1A_F, HIGH);
  digitalWrite(A_1B_F, LOW);
  delay(1000);

  Serial.println("stop");
  digitalWrite(B_1A_R, LOW);
  digitalWrite(B_2A_R, LOW);
  digitalWrite(A_1A_R, LOW);
  digitalWrite(A_1B_R, LOW);
  digitalWrite(B_1A_F, LOW);
  digitalWrite(B_2A_F, LOW);
  digitalWrite(A_1A_F, LOW);
  digitalWrite(A_1B_F, LOW);
  delay(1000);

  Serial.println("right");
  digitalWrite(B_1A_R, LOW);
  digitalWrite(B_2A_R, HIGH);
  digitalWrite(A_1A_R, HIGH);
  digitalWrite(A_1B_R, LOW);
  digitalWrite(B_1A_F, HIGH);
  digitalWrite(B_2A_F, LOW);
  digitalWrite(A_1A_F, LOW);
  digitalWrite(A_1B_F, HIGH);
  delay(1000);

  Serial.println("stop");
  digitalWrite(B_1A_R, LOW);
  digitalWrite(B_2A_R, LOW);
  digitalWrite(A_1A_R, LOW);
  digitalWrite(A_1B_R, LOW);
  digitalWrite(B_1A_F, LOW);
  digitalWrite(B_2A_F, LOW);
  digitalWrite(A_1A_F, LOW);
  digitalWrite(A_1B_F, LOW);
  delay(1000);

  Serial.println("back");
  digitalWrite(B_1A_R, LOW);
  digitalWrite(B_2A_R, HIGH);
  digitalWrite(A_1A_R, LOW);
  digitalWrite(A_1B_R, HIGH);
  digitalWrite(B_1A_F, LOW);
  digitalWrite(B_2A_F, HIGH);
  digitalWrite(A_1A_F, LOW);
  digitalWrite(A_1B_F, HIGH);
  delay(2000);

  Serial.println("stop");
  digitalWrite(B_1A_R, LOW);
  digitalWrite(B_2A_R, LOW);
  digitalWrite(A_1A_R, LOW);
  digitalWrite(A_1B_R, LOW);
  digitalWrite(B_1A_F, LOW);
  digitalWrite(B_2A_F, LOW);
  digitalWrite(A_1A_F, LOW);
  digitalWrite(A_1B_F, LOW);
  delay(1000);
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
