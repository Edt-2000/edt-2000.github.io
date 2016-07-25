---
layout: post
title: Edt-Trak first release!

---

This project started with the idea of modifying a GameTrak controller, and has since then grown in complexity to a whole system for live performances. We are glad to post about our first official 'release'; last weekend we were able to finalize our first 'story' and deliver a 'working product'.

A bit of background: we use [pivotaltracker](https://www.pivotaltracker.com/n/projects/1613837) to track our ideas and progress, each 'story' delivers a next iteration of the Edt-2000 ecosystem. This is a well known project management structure, and we can recommend anyone to use it. [You can read some more about it here](http://www.pivotaltracker.com/help/articles/quick_start)

We want others to be able to build and use what we create, so here is our first guide on how to modify a GameTrak controller! We have been doing this kind of things for a while now, so if there are any things unclear please do not hesitate to ask! Anyone with some electronic experience should be able to do this :)

## The controller

The [GameTrak controller](https://en.wikipedia.org/wiki/Gametrak) is a controller that never really got populair, and sometimes you find them on hobbyist websites but in general they are not often used. 

<iframe width="560" height="315" src="https://www.youtube.com/embed/HFfR_9Wczjc" frameborder="0" allowfullscreen></iframe>

As you can see in this picture we took a while ago, the GameTrak is actually a very simple sensor in a fancy housing.

![GameTrak sensor](https://dl.dropboxusercontent.com/u/4548386/hosted/edt-2000/2016-07-25%20First%20release%20blog/6352091463241700015.jpg)

It consists of a simple joyckstick and a mechanism that measures the retraction of the wire, giving you a total of 3 axis. There must be something we can do with this to control music!?

## How to build your own Edt-Trak

First of all, sorry that we did not took more pictures when taking this thing apart. We just didn't think about it that much. It's also not very hard; just unscrew every screw you see and it will come apart at some point. After you have done that, cut away the original circuit board that you find inside as you can replace that with an Arduino (or similar microcontroller). You will end up with something like we had a while ago.

![GameTrak sensor guts out](https://dl.dropboxusercontent.com/u/4548386/hosted/edt-2000/2016-07-25%20First%20release%20blog/Unknown-9.jpg)

It's a bit hard to see, but each axis of the controller should have 5 wires. It's really nothing more than 3 [voltage dividers](https://en.wikipedia.org/wiki/Voltage_divider) that you can measure: the joystick X and Y axis, and a third potmeter for the Z axis that is driven by a worm-wheel.

The wire colors are like this, but this could differ on your version (we have no idea if they are all the same):

````
red - Z axis
orange - Y axis
yellow - X axis
brown / green - ground / Vin
````

Now you can attach these to your Arduino (or any microcontroller with analog input pins) and start reading the values, like we did a while ago in a hacky way:

![Arduino backpack](https://dl.dropboxusercontent.com/u/4548386/hosted/edt-2000/2016-07-25%20First%20release%20blog/Unknown-3.jpg)

## Building a more permanent prototype

Let's get back to the tie-wrapped Arduino; that's not something we can use on a stage for a live performance as it would break way too easily. One of our goals is to make everything sturdy and foolproof, so we decided to use a standard UTP internet cable to connect the GameTrak to the Arduino. It has 8 internal wires, which is just enough for 2x3 sensor wires and 0-5v wires.

![Open GameTrak](https://dl.dropboxusercontent.com/u/4548386/hosted/edt-2000/2016-07-25%20First%20release%20blog/P7160032.JPG)

We used a cheap UTP breakout on some perfboard, which is far from ideal and not recommended. Get a UTP connector with some terminals or similar! As you can see you really have to fiddle with the wires to get them connected in the right order.

![Wiring horror](https://dl.dropboxusercontent.com/u/4548386/hosted/edt-2000/2016-07-25%20First%20release%20blog/P7160035.JPG)

You will end up with something like this, which should survive for a while but still enables you to replace the cable when someone accidentally steps on it and damages it (which will happen somewhere in the future).

![Internals of Edt-Trak](https://dl.dropboxusercontent.com/u/4548386/hosted/edt-2000/2016-07-25%20First%20release%20blog/P7170043.JPG)

Did we already tell you that we like to use tie-wraps and other methods to make our prototypes a bit more sturdy? This was the result of having to re-insert the wires at least 5 times because they fell out every time anyone moved.

![TieWraps](https://dl.dropboxusercontent.com/u/4548386/hosted/edt-2000/2016-07-25%20First%20release%20blog/P7160037.JPG)

In the end you will have a nice and sturdy GameTrak controller with your own internal wires. The LED that is hidden in the top is attached to the 0V-5V line with a 150 Ohm resistor in series, giving a nice blue ring when there is power. If you got this far, you must be smart enough to get that LED attached yourself (because we also didn't take any pictures..)

![GameTrak on](https://dl.dropboxusercontent.com/u/4548386/hosted/edt-2000/2016-07-25%20First%20release%20blog/P7170044.JPG)

## Measuring and sending Trak-data

We use an Arduino Uno to measure the GameTrak sensors, because it is cheap and easy to use and also because we have a whole bunch of them from previous projects. Go look at [their website](https://www.arduino.cc) if you have never heard or Arduino After some tweaking the following code reads all 6 analog pins and simply sends this to the computer as fast as it can:

```` Arduino
// Define analog pins + UTP colors + GameTrak internal wire colors
#define Left_X A2 // Blue / - yellow left
#define Left_Y A1 // Orange / - orange left
#define Left_Z A0 // Green / - red left

#define Right_X A5 // Brown / - yellow right
#define Right_Y A4 // Orange - orange right
#define Right_Z A3 // Green - red right

// Feedback LED
#define LED 13

// Define value variables initial values
int Left_X_Val = 0;
int Left_Y_Val = 0;
int Left_Z_Val = 0;

int Right_X_Val = 0;
int Right_Y_Val = 0;
int Right_Z_Val = 0;

// Minimal value for Z axis, otherwise send 0's or no values at all (if GameTrak is in resting position, don't send anything)
int gate = 100;
bool active = false;

// Values are divided with a -, instead of println's we give an endpoint :
String divider = "-";
String endPoint = ":";

// Start serial as fast as we can
void setup() {
  Serial.begin(1000000);
  pinMode(LED, OUTPUT);
}

void loop() {
  // Read the values of each pin
  Left_X_Val = analogRead(Left_X);
  Left_Y_Val = 1023 - analogRead(Left_Y);
  Left_Z_Val = analogRead(Left_Z);
  Right_X_Val = analogRead(Right_X);
  Right_Y_Val = analogRead(Right_Y);
  Right_Z_Val = analogRead(Right_Z);
  // Only send when one of the X values is higher than 100
  if(Left_Z_Val > gate || Right_Z_Val > gate) {
    active = true;
    // LED feedback
    digitalWrite(LED, HIGH);
    // When below gate, set to 0
    if(Left_Z_Val <= gate) {
      Left_X_Val = 0;
      Left_Y_Val = 0;
      Left_Z_Val = 0;
    }
    // When below 100, set to 0
    if(Right_Z_Val <= gate) {
      Right_X_Val = 0;
      Right_Y_Val = 0;
      Right_Z_Val = 0;
    }
    // Print to a line seperated by divider
    Serial.print(
      Left_X_Val + 
      divider + 
      Left_Y_Val + 
      divider + 
      Left_Z_Val + 
      divider + 
      Right_X_Val + 
      divider + 
      Right_Y_Val + 
      divider + 
      Right_Z_Val + 
      endPoint
    );
  } else {
    if(active) {
      // Send a last 0 0 0 0 0 0 message
      Serial.print(0 + divider + 0 + divider + 0 + divider + 0 + divider + 0 + divider + 0 + endPoint);
      active = false;
    }
    // Not sending, LED to LOW
    digitalWrite(LED, LOW);
  }
}
````

We added some LED feedback and also some logic to send 0's when the Z axis is below a certain threshold, but you get the idea.

## Receiving Trak-data on a computer

We use [processing](https://processing.org) to receive the values and translate them to OSC messages, because it is reliable, open source and easy to use. The language and editor is very similar to the Arduino IDE and there are a lot of libraries and code examples available online. 

````Processing
import netP5.*;
import oscP5.*;

import processing.serial.*;

Serial myPort;  // Create object from Serial class
int val;      // Data received from the serial port

OscP5 oscP5;
NetAddress myRemoteLocation;

void setup() 
{
  size(200, 200);
  // Set your target IP adress (127.0.0.1 to send to your own PC) + port
  myRemoteLocation = new NetAddress("127.0.0.1", 8000);
  // Listening port is 12000, but we don't use it.
  oscP5 = new OscP5(this, 12000);
  
  // Change this to your own Serial port identifier, this is on a Mac
  // The serial BAUD rate should be the same as in the Arduino sketch
  myPort = new Serial(this, "/dev/cu.usbmodem14131", 1000000);
  // This is our line end, set in the Arduino sketch
  myPort.bufferUntil(':');
}

void draw()
{
  rect(50, 50, 100, 100);
}

void serialEvent (Serial myPort) {
  // Read until our separator (defined in Arduino sketch)
  String input  = myPort.readStringUntil(':');
  // Remove : from the end
  String[] inputs = input.substring(0, input.length() - 1).split("-");
  // Create an int holder
  int[] values = new int[inputs.length];
  
  // Parse to ints, or set to 0 if garbage data (Arduino sends a lot of rubbish when turned on)
  for(int i = 0; i < inputs.length; i++) {
    try {
      values[i] = Integer.parseInt(inputs[i]);
    } catch (NumberFormatException e) {
      values[i] = 0;
    }
  }
  
  // Send over OSC to whoever is listening
  OscMessage myMessage = new OscMessage("/Trak");
  myMessage.add(values);
  oscP5.send(myMessage, myRemoteLocation); 
}
````

We use a Macbook to run the software, so we don't really know how it works on Windows with the Serial port identification but there are enough online resources that can tell you all about that if you don't have a Macbook.

If you have followed along and have everything up and running, the Edt-Trak should now send around 500-1000 [OSC](https://en.wikipedia.org/wiki/Open_Sound_Control) messages to your computer in the following format:

````
/Trak 563 232 534 344 567 863
````

## Doing something with the data

In our growing list of software packages we use we can add one last name: [MaxMSP](https://cycling74.com/products/max). MaxMSP enables us to quickly design functionality and easily control MIDI or anything else through creating 'patches'. It takes some time to learn, but once you get the hang of it you will be wiring everything together in no time.

![MaxMSP Edt-Trak patch](https://dl.dropboxusercontent.com/u/4548386/hosted/edt-2000/2016-07-25%20First%20release%20blog/Screen%20Shot%202016-07-25%20at%2016.17.01.png)

In the git repository you will find an example patch that simply 'unpacks' the 6 values and sends this to the outputs, so you can attach your own logic to control a drum computer or synth through MIDI.

![AirDrum](https://dl.dropboxusercontent.com/u/4548386/hosted/edt-2000/2016-07-25%20First%20release%20blog/Screen%20Shot%202016-07-25%20at%2016.27.58.png)

![Airdrum 2](https://dl.dropboxusercontent.com/u/4548386/hosted/edt-2000/2016-07-25%20First%20release%20blog/Screen%20Shot%202016-07-25%20at%2016.28.04.png)

As this post is getting long, let's just wrap up by asking you to start building yourself and keep us informed of your progress through the comments or through our [GitHub organisation](https://www.github.com/edt-2000). We will try to post updates every once in a while! All the code on this page is also available at the [repository for the Edt-Trak](https://github.com/Edt-2000/Edt-Trak).