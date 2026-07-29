---
layout: page
title: SNN_Arduino
description: An Arduino library for building spiking neural network "brains" for robots — LIF neurons, arbitrary network motifs, and sensor/actuator interfacing in the spirit of liquid state machines.
importance: 4
github: https://github.com/RishabhMalviya/SNN_Arduino
---

An early project, from my undergraduate years at IIT Bombay: a small C++ library that lets you build **spiking neural network brains for your Arduino robots**, largely following the paradigms of liquid state machines with **leaky integrate-and-fire (LIF)** neurons.

Rather than running a trained network on a host machine and streaming commands to the robot, the network is simulated directly on the microcontroller. Neurons integrate real analog and asynchronous digital input from sensors, and "motor" neurons drive L293D ICs and other actuation — so the whole sensorimotor loop is neuromorphic and on-board.

## What it does

The library is built around two classes:

- **`Neuron`** — simulates a LIF membrane potential in code, with configurable capacitance, leak conductance, resting potential, threshold, and spike amplitude. Each neuron can be bound to an input pin (sensory drive) and an output pin (spike output, e.g. to an LED or a motor driver).
- **`Network`** — holds up to 6 neurons and an explicit **connectivity matrix**, which lets you wire arbitrary network motifs. Rows correspond to the spiking neuron and columns to the neurons on the receiving end, so you can implement anything up to a fully-connected network.

Sensory neurons are declared by ID; digital sensors are interfaced through pin-change interrupts that inject current into the corresponding neuron. A `Visualize()` helper dumps spiking behaviour so you can watch the network run on LEDs wired to the default output pins.

## A four-neuron example

This network has two sensory and two motor neurons. The sensory neurons aren't used directly as motor drivers — instead, the connectivity matrix implements a linear combination of the sensory spikes (mutual inhibition, in this case) that drives the motor neurons:

```cpp
#include <Neurons.h>
#include <Network.h>

int sensoryNeuronIDs[2] = {0, 1};
int connectivity[16] = { 0,  0,  100, -800,
                         0,  0, -800,  100,
                         0,  0,    0,    0,
                         0,  0,    0,    0 };

Network testNetwork(4, connectivity);

void setup() {
  Serial.begin(9600);
  testNetwork.dt = 5;
  testNetwork.setSensoryNeurons(sensoryNeuronIDs, 2);
}

void loop() {
  testNetwork.updateNeurons();
  testNetwork.issueSpikes();
  testNetwork.Visualize();
  delay(testNetwork.dt);
}

ISR(PCINT1_vect) {
  if (digitalRead(A0) != pins[0]) {
    testNetwork.Neurons[0].inputCurrent += 2000;
    pins[0] = digitalRead(A0);
  }
  // ... likewise for A1
}
```

The repository ships four worked examples — `singleNeuronNetwork`, `twoNeuronNetwork`, `connectivityMatrix`, and `analogIn` — which are the fastest way to get a feel for the library.

This work grew out of the same interest in neuromorphic computing that led to my undergraduate research on [neuromorphic hardware applications](https://general-vision.com/pub3rdparty/3P_FaceReco_Voice_Manan.pdf).
