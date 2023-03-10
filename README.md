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
    
    
## Unity

In order for Unity to send the haptic feedback to the server motors, we estimate the curl of each finger from a skeleton passed in in the skeleton poser from SteamVR.

     public void SetForceFeedbackFromSkeleton(Hand hand, SteamVR_Skeleton_Pose_Hand skeleton)
     {
        SteamVR_Skeleton_Pose_Hand openHand;
        SteamVR_Skeleton_Pose_Hand closedHand;
        
        if (hand.handType == SteamVR_Input_Sources.LeftHand)
        {
            openHand = ((SteamVR_Skeleton_Pose) Resources.Load("ReferencePose_OpenHand")).leftHand;
            closedHand = ((SteamVR_Skeleton_Pose) Resources.Load("ReferencePose_Fist")).leftHand;
        }
        else
        {
            openHand = ((SteamVR_Skeleton_Pose) Resources.Load("ReferencePose_OpenHand")).rightHand;
            closedHand = ((SteamVR_Skeleton_Pose) Resources.Load("ReferencePose_Fist")).rightHand;
        }
        
        List<float>[] fingerCurlValues = new List<float>[5];
        
        for(int i = 0; i < fingerCurlValues.Length; i++) fingerCurlValues[i] = new List<float>();
        
        for (int boneIndex = 0; boneIndex < skeleton.bonePositions.Length; boneIndex++)
        {
            //calculate open hand angle to poser animation
            float openToPoser = Quaternion.Angle(openHand.boneRotations[boneIndex], skeleton.boneRotations[boneIndex]);
            
            //calculate angle from open to closed
            float openToClosed =
                Quaternion.Angle(openHand.boneRotations[boneIndex], closedHand.boneRotations[boneIndex]);
            
            //get the ratio between open to poser and open to closed
            float curl = openToPoser / openToClosed;
            
            //get the finger for the current bone
            int finger = SteamVR_Skeleton_JointIndexes.GetFingerForBone(boneIndex);
            
            if (!float.IsNaN(curl) && curl != 0 && finger >= 0)
            {
                //Add it to the list of bone angles for averaging later
                fingerCurlValues[finger].Add(curl);
            }
        }
        //0-1000 averages of the fingers
        short[] fingerCurlAverages = new short[5];

        for (int i = 0; i < 5; i++)
        {
            float enumerator = 0;
            for (int j = 0; j < fingerCurlValues[i].Count; j++)
            {
                enumerator += fingerCurlValues[i][j];
            }
            
            //The value we to pass is where 0 is full movement flexibility, so invert.
            fingerCurlAverages[i] = Convert.ToInt16(1000 - (Mathf.FloorToInt(enumerator / fingerCurlValues[i].Count * 1000)));
            
            Debug.Log(fingerCurlAverages[i]);
        }
        
        _SetForceFeedback(hand, new VRFFBInput(fingerCurlAverages[0], fingerCurlAverages[1], fingerCurlAverages[2], fingerCurlAverages[3], fingerCurlAverages[4]));
    }
