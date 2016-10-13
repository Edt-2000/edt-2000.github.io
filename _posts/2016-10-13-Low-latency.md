# Low latency OSC messages exchange over UDP

In my previous blog post about OSC messages, I primarily focused on getting the
number of messages per second up without looking at, perhaps more important, things
like delay or reliability. In this blog post, I will dive a bit more into getting
things faster, but use the message latency as the primary indicator to see if my code
is faster.

But to test the latency, I'll need a setup which can detect how long it takes for an
OSC message to be transported. To do that, I'll be using two identical Leonardo Ethernet
Arduino's which will be connected via a router, and via their digital ports. One
Leonardo, "Ping", will set one of its digital ports high, causing the other Leonardo,
"Pong", to react and broadcast a OSC message. When Ping gets this message, it will
set another port high. Pong will able to measure how long it took by determining
when the first digital port became high until the second digital port became high. The
time difference between the two events will be the latency.

The setup of the two Arduinos will look something like this:

```
[Arduino 1 "Ping"] D5 ------- D5 [Arduino 2 "Pong"]
  |                D6 ------- D6                |
  |                                             |
  -------------------[Router]--------------------
```

## Benchmark

Before we're doing anything, first start with a benchmark, to know where we're standing
and whether code changes really yield lower latencies. Starting with the current libraries
I have created two new projects: [ping](https://github.com/Edt-2000/Playground/commit/2b6ec9dea928d4140d1a6488eb172a5262c1614d)
and [pong](https://github.com/Edt-2000/Playground/commit/c52ffab50699e2bc24225ee2a649c2502dabb8a9).

These two will exchange signals in order for me to evaluate the latencies. What they
will output will be something like this (the first number is the time between the
current and previous message, in microseconds):

### Ping
```
72: 2: Start cycle.
80: 2: Writing start signal.
1576: 2: OSC Message received.
44: 2: Writing finish signal.
80: 2: Waiting to reset.
1000036: 2: Reset.
```
### Pong
```
36: 2: Start cycle.
500392: 2: Received start.
1792: 2: Received finish.
499960: 2: Received reset.
```

Combining the outputs to a single list will result in something like this:

```
Ping                                        Pong

72: 2: Start cycle.                         36: 2: Start cycle.
80: 2: Writing start signal.                ..
..                                          500392: 2: Received start.
1576: 2: OSC Message received.              ..
44: 2: Writing finish signal.               ..
..                                          1792: 2: Received finish.
80: 2: Waiting to reset.                    ..
1000036: 2: Reset.                          ..
..                                          499960: 2: Received reset.
```

So the time it takes to send a single integer from one Arduino to another takes
about 1.8 milliseconds. Let's get that low as possible.

## Rewrite

I tried to keep track of the changes I made and test whether they are of any influence
on the latency. But I found that I needed a new OSC library, and that optimizing the
one that I used wasn't going to be efficient.

So, [after completely rewriting the OSC library](https://github.com/Edt-2000/Playground/tree/develop/Arduino/libraries/OSC-light)
I managed to get the delay down from 1.8 milliseconds to 1 millisecond. I can probably
spend a whole blog post on why I replaced the library and why it is faster, and I'll
probably do that. But the short version of that blog post is: I replaced everything
and focused on avoiding repetitive memory accesses and tried to reuse variables as
much as possible. Furthermore, I rewrote the OSC address matching logic, which is now
more simple and uses more of the standard C string functions.

This 1 millisecond delay is nice, although this figure is not realistic as the test only sends a single integer.
Changing the test message from 1 integer to 6, which the Trak will send, the delay got up to
roughly 1.3 milliseconds. Every integer causes an extra 60 microseconds delay, which is more
than expected.

```
Ping

68: 2: Start cycle.
72: 2: Writing start signal.
1636: 2: Writing finish signal.
64: 2: Waiting to reset.

Pong

12: 2: Start cycle.
500384: 2: Received start.
1272: 2: Received finish. <-- 1.3 milliseconds round-trip
500308: 2: Received reset.
```

So that's all for this time. Next time I'll probably write more about the new library,
and post some details about some of the choices I've made. Or I'll post about the
Sparkfun Things, which I finally got working. 

























.
