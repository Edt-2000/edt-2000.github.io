A few weeks back we were setting up the first version of the Edt-2000 system and tried to make some music with it and see what effects we can make with it. We used a Arduino Leonardo with Ethernet for reading the Gametrak and a Arduino with an Ethernet shield for handling incoming OSC messages and converting them into MIDI. Using it this way allows us to broadcast the OSC messages over the network, and have wireless suits with Sparkfun Things and stuffed with LEDs and EL-wires that react to the incoming data from the Gametrak. [This map](https://github.com/Edt-2000/Playground/blob/develop/LAYOUT.md) could make some things more clear.

In order to make the Gametrak a useful instrument, and be able to make music with it, the delay between the Gametrak and the resulting MIDI signal must be minimal. Something in the range of 1-2 milliseconds. But during testing the performance of the OSC message exchange was not very good. There was quite some delay between each message, it looked like somewhere something chocked and could not keep up with the pace we wanted it to have. So, I started some testing and tried to find out what was causing the delays and see whether I could fix it and make the code work faster.

### The benchmark

To have something of a reference, and see how fast the network can be, I have made a simple [OSC message generator](https://github.com/Edt-2000/Playground/commit/9e24bb9aa58dfcc1a13b9a00f0f7ff2ca20108c3) and [OSC message monitor](https://github.com/Edt-2000/Playground/commit/cd1a083703238cd7bb030d4d3baa86d241f90542). The generator just generates as much as OSC messages as possible, sending them to an IP address and the monitor listens to a specific port, and displays the amount of messages received per second. When I tested it on my computer, I got the following results:

````
Messages received: 9007.
Messages received: 8807.
..
Messages received: 9026.
Messages received: 9034.
````

This is a part that I copied from the message log of my OSC monitor. Each line takes about a second, so my simple OSC generator is writing a message every 0.11 ms.

Since I am planning on using a broadcasting IP to broadcast every UDP message to everything on the network, I changed the IP to which the generator is sending to 10.0.0.255, which is the broadcasting address when using a subnet of 255.255.255.0.

````
Messages received: 7714.
Messages received: 7773.
..
Messages received: 7942.
Messages received: 7324.
````

The drop in messages per second is significant, but still, a message per 0.14 ms is still overkill. This little benchmark shows how fast the messages can be send back and forth over the network.

### The first test

Knowing the network can transmit large amounts of messages per second quite effortlessly, let's see how [my application for an Arduino Leonardo with Ethernet](https://github.com/Edt-2000/Playground/releases/tag/Trak_v1) is performing.

````
Messages received: 176.
Messages received: 176.
..
Messages received: 176.
Messages received: 176.
````

Well, that is quite bad.

This is only 40 times as bad as the benchmark, considering that this application is also writing its messages to the broadcasting address. Off course, the processing power of an Arduino and its onboard network controller are nothing compared to a laptop, but I think I can do better that 176 messages per second.

### Reduce some extras

First, I want my program to be as lean as possible, so [I have removed some things](https://github.com/Edt-2000/Playground/commit/0195423943f4625f9508f42405f4d099c188cddd) I do not need, and changed the broadcasting address to the IP address of my computer, but both changes did not result in any improvement. So I need to look elsewhere.

### AnalogRead

Using the Arduino library to program a Leonardo has its advantages, even though I build and upload using PlatformIO, since the community behind Arduino is quite productive and has libraries available for almost anything, including the ones I am using for OSC and UDP. But the methods that are available in the Arduino library are known to be slow in some cases, especially the methods concerning IO. So [let's just remove the whole analogRead method](https://github.com/Edt-2000/Playground/commit/c9fa2af1236cd7fb485e65afc60c618a11aa6252) and see what we get.

````
Messages received: 209.
Messages received: 209.
..
Messages received: 209.
Messages received: 209.
````

Okay. That improves the message rate from 176 mps to 209 mps. Roughly 15%. So some improvement can be made here. But as we need the sensory data, perhaps we can replace the analogRead with something faster or tweak the current one. But when that only results in a 10% improvement, the total amount of messages would only improve 1 - 2%. We keep this in mind and use the default analogRead for now.

### Juggling OSCMessages

Internally, we pass the OSCMessage from the Trak instance to the OSC instance by value. The Trak instance deals with reading the Trak sensor and creating OSC messages, and the OSC instance handles the sending of OSC messages. This could perhaps be made faster by [passing the OSCMessage by reference](https://github.com/Edt-2000/Playground/commit/14530e560c08b757ca8b08486435498014057070). After fixing [the memory leak](https://github.com/Edt-2000/Playground/commit/43c5784ab8861fce99d81649ceb17f605093f682), the results were not very encouragingly:

````
Messages received: 176.
Messages received: 176.
..
Messages received: 176.
Messages received: 175.
````

So that did not make any difference; so let's focus on the [OSC library](https://github.com/Edt-2000/Playground/tree/develop/Arduino/libraries/OSC-master) I have been using. This library is very nice, but it could contain time consuming features which we do not require.

### OSC library

First, let's just stop reading any OSC information that we receive and [flush the buffer instead](https://github.com/Edt-2000/Playground/commit/cd23fe1d7a5a06be5293b176a287022910b0c510) and while we're at it, [use the better flushing method](https://github.com/Edt-2000/Playground/commit/f538de2d1d93f368a4dfdec77c89484be42666c8) I found on the internet:

````
Messages received: 177.
Messages received: 177.
..
Messages received: 177.
Messages received: 177.
````

Yes. Result! But not very significantly. Not quite unexpected, since we weren't really receiving a lot of data anyway. So let's dig deeper.

When I was going through the OSC library, I spotted a suggestion from Paul about putting the complete message on the stack to improve performance. The downside of this is that it could exhaust the memory for complex messages, but since we do not have complex messages (assuming 6 floats and an integer is not complex), this should be fine. [So I enabled the suggestion](https://github.com/Edt-2000/Playground/commit/ed8401a8b2f412a661c909ad6006d46f996dd359):

````
Messages received: 214.
Messages received: 214.
..
Messages received: 214.
Messages received: 214.
````

That proved to be quite a good suggestion, as the rate improved by 20% by sending larger chunks at once. Going through that same send method, we could strip out some more and buffer the complete message and transmit it in one single go. While we are at it, we could also strip out the support for any data type but floats and integers, as TouchOSC only supports floats and we want to be compatible with TouchOSC. So, [I have made some changes to the code](https://github.com/Edt-2000/Playground/commit/1a62a721ff405293e9f31fe0fe7a17f8e5c4e747) and tested it:

````
Messages received: 442.
Messages received: 442.
..
Messages received: 442.
Messages received: 442.
````

When looking closer into that method, the only data types we were using were located at the end of a lengthy if statement and a lot of ```p.write()``` calls were made in various parts of the method. In the new method the complete message, being about 40 bytes big, is send in a single ```p.write()``` call. And it improves the performance by 200%.

### AnalogRead, part two

Since the sending part of the OSC message is quite a bit faster, let's see what difference [removing the analogRead method](https://github.com/Edt-2000/Playground/commit/c9fa2af1236cd7fb485e65afc60c618a11aa6252) makes now.

````
Messages received: 738.
Messages received: 738.
..
Messages received: 742.
Messages received: 734.
````

So the previous time it changed from roughly 175 to 210, now the it's 442 to 740. That is quite significant. So perhaps looking further into speeding up analogRead could have quite some effect. On the internet I found an interesting article: http://www.microsmart.co.za/technical/2014/03/01/advanced-arduino-adc/. This article demonstrates that you can sacrifice some of the accuracy of the ADC of the Arduino in order to get faster readouts. Let's see what happens if I [configure the prescalar to 16](https://github.com/Edt-2000/Playground/commit/2f5b868bd7fbf4b13d8968abb1e2ccc3588d90e7) to get the fastest readouts.

````
Messages received: 591.
Messages received: 590.
..
Messages received: 591.
Messages received: 591.
````

The improvement is almost as big as the improvement of OSC send method, so I am quite pleased. Off course, I have sacrificed some of the accuracy of the analogRead, but a quick check of the transmitted data showed that the data was still looking good. If the accuracy proves to be too low, I could always go with a prescalar of 32, which gives me roughly 560 messages per second.

So, overall, I managed to get an improvement of 330% on sending OSC messages over UDP. By simply removed extra functionality and streamlining some IO I managed to get the speed to something probably more acceptable. Next time I'll be looking into receiving OSC messages on a Arduino and a ESP8266, see how fast that is and how fast it can be made.
