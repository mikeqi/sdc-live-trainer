# sdc-live-trainer
Live training program for Udacity's SDC Simulator. Created as a part of Udacity Self-Driving Car Project 3: Behavioral Cloning.

## Overview
This program was created to allow the "remote" driving of
Udacity's Self Driving Car (SDC) Simulator using a neural network, while also allowing an option for manual override and live-collection of data from real time fine-tuning of the neural network.

This was idea inspired by [John Chen's Agile Trainer](https://www.github.com/diyjac/AgileTrainer).

Note: If you want to use your own model, make sure you modify the `preprocess_input` method in `live_trainer.py` to do whatever preprocessing your model expects.

### server.py

This is an object-oriented, modular version of the `drive.py` script that Udacity provided. I made a `ControlServer` class that triggers callbacks when the simulator connects and sends telemetry. This plugs into all the three programs listed below.

The main difficulty was in understanding how `Tkinter`'s event loop works and how to make it play nice with the `eventlet` loop that `drive.py` uses to set up the WSGI server. Eventually, I was able to fold in the Tkinter UI loop as a sepearate `eventlet` "greenthread". Check the code for details on how this is implemented.

### manual_driver.py

This was my initial proof of concept to see if it is possible to reliably control the SDC simulator using keyboard input, while the simulator is in "autonomous mode".

The keyboard input is used to increment and decrement a `steering_angle` state. The program also models some restoring-torque dynamics that slowly re-centers the steering when there is no keyboard input. There is also a simple proportional-gain controller that modulates the throttle to achieve a target speed that can be controlled by the user.

You can run it by typing: `python manual_driver.py`

Then start the Udacity SDC Simulator and click "Autonomous Mode". This should bring the program window into focus again once the simulator connects. The controls are:

**<kbd>Up</kbd>/<kbd>Down</kbd>** : Control speed  
**<kbd>Left</kbd>/<kbd>Right</kbd>** : Steer the car

### hybrid_driver.py

This was the second step to implementing the live trainer. This program allows the user to specify a Keras model that will predict a steering angle based on the image data sent by the simulator. It also allows a manual override option to take over control at any time.

Run it by typing:
`python hybrid_driver.py <model>.json`

where `<model>.json` is the Keras model file. The program expects the weights for the model to be stored in `<model>.h5`.

The controls are:

**<kbd>Up</kbd>/<kbd>Down</kbd>** : Control speed  
**<kbd>Left</kbd>/<kbd>Right</kbd>** : Steer the car  
**<kbd>x</kbd>** : Toggle manual override and autonomous mode  
**<kbd>c</kbd>** : Reset steering angle to zero (only in manual mode)  

### live_trainer.py
This is the final implementation of the live trainer. This program combines the functionality of the above two scripts, along with the capability to train the neural network in real-time. Every time a batch is trained, the model weights are saved to `checkpoint.h5`. Live training can be initiated at any time when the car is in manual override.

The controls are:

**<kbd>Up</kbd>/<kbd>Down</kbd>** : Control speed  
**<kbd>Left</kbd>/<kbd>Right</kbd>** : Steer the car  
**<kbd>x</kbd>** : Toggle manual override and autonomous mode  
**<kbd>c</kbd>** : Reset steering angle to zero (only in manual mode)  
**<kbd>z</kbd>** : Toggle live training (only in manual mode)  

The program also displays an "Autonomous Rating" which is the percentage of runtime that the program was in fully autonomous mode.

The learning rate, checkpoint filename and batch size can be adjusted in parameters defined at the beginning of the script.

# Usage
We start with a neural network that was trained on data from the SDC simulator that sort of works. I used NVIDIA's End-to-End Deep Learning Architecture. The live trainer is to be used for fine-tuning this original model. It may also be possible to train a model from scratch using this live trainer, but it might take considerably longer.

The idea is to let the model control the car until we feel like it has deviated too much from the track. Then we take over, and the train it as soon as we have set the steering angle to the right values for recovery. We continue training until we are through the "difficult" area and switch back to autonomous mode. By repeating this several times, the model will eventually learn to navigate any difficult spots it has encountered. It might also be helpful to slow down considerably (to 4-5 mph) while in difficult spots so that the more data can be collected for training.
