# Haptic Feedback Gloves

This project was built upon the [LucidGloves](https://github.com/LucidVR/lucidgloves) project.

The glove consists of an ESP32 dev board, 10K WL potentiometers, MG 90 Micro Servo Motors, multiple 3D printed parts and the spools and thread of badge reels.

![enter image description here](https://camo.githubusercontent.com/5fdc308881d8162ef857c3a5de539e3d302f1c2ff6c40ee94bce6969845236e1/68747470733a2f2f63646e2e646973636f72646170702e636f6d2f6174746163686d656e74732f3738353133353634363038323939303132302f3837353837383032363036323230343932382f756e6b6e6f776e2e706e67)



## Wiring

We wired the ESP32 board directly to the potentiometers and servo motors. For the pententiometers we were able to use the 3.3V pin from the dev board, for the servo motors however, at least 5V were needed. We decided to use an old USB cable and wire a splitter to have a high enough voltage. We connected all the pins using a split GND pin.

![wiring diagram](https://github.com/DeniseBischof/haptic_feedback_gloves/blob/main/Screenshots/wiring%20diagram.png?raw=true)

## Arduino IDE

In order for us to access the potentiometer values, we directly read the input and map it to a minimum and maximum value. 

    analogValue = analogRead(36);
    value = map(analogValue, 0, 4095, VALUE_MIN, VALUE_MAX);
   
We can then send the values directly to SteamVR via the serial port and estimate the movement calibration of our hand.

After the estimation, SteamVR sends a haptic feedback to the serial port, which then will be directly sent to the servo motors.

    void  writeServoHaptics(int* hapticLimits){
    
    float  scaledLimits[5];
    
    scaleLimits(hapticLimits, scaledLimits);
    
    if(hapticLimits[0] >= 0) thumbServo.write(scaledLimits[0]);
    
    if(hapticLimits[1] >= 0) indexServo.write(scaledLimits[1]);
    
    if(hapticLimits[2] >= 0) middleServo.write(scaledLimits[2]);
    
    if(hapticLimits[3] >= 0) ringServo.write(scaledLimits[3]);
    
    if(hapticLimits[4] >= 0) pinkyServo.write(scaledLimits[4]);
    
    }
    
    


