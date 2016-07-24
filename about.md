---
layout: page
title: About
subtitle: About the Edt-2000 project
---
The Edt-2000 is a collective name for a number of components that will form an expressive midi controller for live performances. If you've ever been to a live electronics music show you know how non-interactive most of those performances are; it's a guy/girl standing over a laptop or keyboard or you-name-it turning some knobs. 

The Edt-2000 aims to be different. Based on an old GameTrak controller and 2 geeks this projects wants to make electronic music more interactive and expressive. We invite you to get one yourself too and help us build a cool new instrument! 

We use [pivotaltracker](https://www.pivotaltracker.com/n/projects/1613837) to capture new ideas and our progress, so have a look if you're interested.

## Components

As Thomas and Edwin are both programmers they are used to create separate components that communicate and work together through a clear interface; the OSC messaging layer. The idea is that each component knows as little as possible about the rest of the system, and just listens or sends OSC messages (or MIDI/DMX). The Edt-Cetera, a fancy name for a computer with MaxMSP, will orchestrate the components and determine which output mechanism is controlled by which sensor. As we progress with more functionality, each component can get more responsibilities and for instance control an LED directly, but for now almost everything goes through this routing mechanism.

### Edt-Trak & Edt-Pedal

This is the starting point with the original idea: a modified GameTrak controller that can control midi instruments. Because we want to be able to switch between presets we are making a control pedalboard, so that it becomes easy to control stuff on stage with some simple foot switches. We haven't really figured out which switch does what, but hey; switches! Always fun!

### Edt-Strobe & Edt-TOP & Edt-FadeLight

A good live show is nothing without cool lighting. We came up with 3 important elements: a stroboscope or bright LED that can be triggered, an Edt-Top (Tower of Power) that is simply a strip of LED's that can be placed easily anywhere on the stage and the Edt-FadeLights; a few simple bright RGB LED's (or DMX dimmers) that fade light in or out when you raise your hands (or do anything like that).

### Edt-Chuck & Edt-Suit

We want the performers to have a cool suit, with lights and EL-Wire, just because we can. The Edt-Chuck will be a nunchuck controller that wirelessly communicates with the rest; and can be used to give more fine control over which parameters you control and what kind of things happen with the music and lights.

### Edt-MOSCidi

We finish with the Edt-MOSCidi, which is also one of the most important components to create a flexible system. Through experiments we discovered that OSC is a fast and reliable protocol to use for our communications; but we do need a way to bring OSC messages into a midi device like the synth or the drum computer. A powerful Arduino (or maybe Netduino) will become a bridge between OSC messages and Midi messages, so that you could control your synth with only the Edt-Trak and the Edt-MOSCidi.

# You want to help?

We would love to know what people think, and encourage others to help build a cool instrument/controller. Please play around with the code, leave comments on github or submit an issue through Git when you find a bug.
