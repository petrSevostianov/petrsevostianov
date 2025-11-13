Solution to the LED Wall Calibration problem of Virtual Production.
Hello! I'm Petr Sevostianov, CTO and co-founder of Antilatency. I have written an article about an important problem in virtual production related to cinema cameras and their non-linear color output. 
We discovered that cinema cameras apply non-linear gamut transforms to the images they capture and we're not talking about the transfer functions (like SLog3, ARRI LogC4, Blackmagic Film Gen5).
We have learnt how to measure, characterize and cancel this transform out. 
And then reapply this transform at the end of the pipeline.

This is crucial for accurate LED wall to camera calibration in virtual production.
You can read the full article here:
https://petrsevostianov.github.io/CinemaCameraBeautifiers/

As we all know the cinema cameras use transfer functions. To make an image linear, we have to apply the inverse transfer function to the image. Those transfer functions are publicly documented for most cameras. This is the known approach to linearization. 

However, it turns out that the transfer function is not the only non-linear operation done by cinema cameras. There is also a nonlinear gamut transform happening in the camera before the transfer function is applied. So the pipeline actually looks like this:

Sensor data (linear) -> Gamut transform (non-linear) -> Transfer function (non-linear) -> Encoded image

The first time I encountered this problem was during the CyberGaffer project, where accurate lighting calibration was required. So getting a truly linear image from the camera was essential for the system to work correctly. 
As we dug into the problem, we found work by others before us have also noticed this issue.
For example OpenVPCal @ Netflix have developed a method to calibrate LED walls to cameras by using desaturated colors patches as they too have discovered the camera's non-linear behavior. 

We decided to create a systematic way to measure and characterize this non-linearity.
This is why we have developed our own methodology to test the linearity of a camera (the Two Lights Experiment).
Which utilizes the additive nature of linear images to check if the camera response to stimuli is truly linear or not.

The results of this experiment proved that there is a non-linear gamut transform being applied by every cinema camera we have tested so far.
There is no public information about these transforms as camera manufacturers do not disclose them.

This led us to engineer our own hardware device with precise linear light output to measure and characterize these transforms. The device is capable of producing 4096 different linear light samples. 
We used that device with our camera to measure the samples and constructed a comprehensive map of the way our camera reacts to all of the 4096 different stimuli. 
This allowed us to build a map of the camera's non-linear behavior and the transform it applies to the images.
As well as to create an inverse transform that can be applied to the images to cancel out the non-linearity. 

Full article is attached for you to read and explore the details of our methodology, experiments, and findings, with images, graphs and interactive 3D models. 

We hope this research will move Virtual Production forward. Especially in the areas of keying, light calibration and LED wall calibration.
And we hope it will one day become obsolete as camera vendors choose to share these transforms publicly.