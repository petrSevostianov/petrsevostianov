This study is focused on exploring nonlinear gamut transforms done by cinema cameras. Those transforms make image non-linear and damage the most of techniques in virtual production such as keying, lighting calibration and led wall to camera matching. Since these transforms are not publicly documented, the goal is to reverse-engineer them. The article describes the methodology of measuring, canceling and re-applying such transforms.

## Known approach to linearization.
Cinema camera row profiles are packed with transfer functions. To make image linear, we need to apply inverse transfer function to the image. 
Those transfer functions are publicly documented for most of cameras.

Here is some references:
- [SLog3](TransferFunctions/SLog3_SGamut3Cine_SGamut3.pdf)
- [ARRI LogC4](TransferFunctions/ARRI_LogC4.pdf)
- [Blackmagic Film Gen 5](TransferFunctions/BlackmagicGen5.pdf)

However, it turns out that transfer function is not the only non-linear operation done by cinema cameras. There is also a **nonlinear gamut transform** happening in camera before transfer function is applied. So pipeline looks like this:

`Sensor data (linear)` ->  `Gamut transform (non-linear)` ->  `Transfer function (non-linear)`  ->  `Encoded image`

## What is gamut nonlinearity?
Since light is additive, all mixtures of any two colors should lay on a straight line between those colors. However, due to gamut nonlinearity, those mixtures are bent away from the straight line.
In simple words, **Half-Yellow**(0.5 0.5 0) will not be between **Red**(1 0 0) and **Green**(0 1 0). There is a reason why this nonlinear gamut transform exists, but is a topic for another article.

## How to find this nonlinearity with a simple experiment?
This experiment relies on the fact that light is additive. So to get image of a scene illuminated by two lights, we can sum images of each light on separately.

You will need:
- Cinema camera
- 2 RGB lights
- White object of any shape

We can capture 4 images where each light is in one of two states. By choosing those states(colors) differently, we can measure how mixtures of those colors are bent.

So we will do 4 experiments with the following color pairs:

|Experiment | State A | State B |
|-----------|---------|---------|
|1          |  Black  |  White  |
|2          |  Red    |  Green  |
|3          |  Green  |  Blue   |
|4          |  Blue   |  Red    |

For each experiment, we will capture 4 images:

| Image | Light 1 | Light 2 |
|-------|---------|---------|
|   I1  |   A     |    A    |
|   I2  |   B     |    B    |
|   I3  |   A     |    B    |
|   I4  |   B     |    A    |

After **removing transfer function**, `I1`+`I2` should be equal to `I3`+`I4` if images are linear. Any difference shows nonlinearity.

![Two Lights Experiment](TwoLightsExperiment.png)
On that image we can see the results of the experiments.
Each row corresponds to one of the experiments above.
Left column is `I1+I2`, middle column is `I3+I4`, right column is difference (positive and negative) between those sums.

Experiment 1 shows very small difference (noise scale). That is what we expect.
However, experiments 2, 3 and 4 show significant differences. That means that camera is nonlinear in gamut plane.

## Measuring gamut nonlinearity
The experiment above can only show that nonlinearity exists, but does not provide enough data to measure and reverse-engineer it. In order to do that, we've designed a measurement device that is able to precisely display linear colors. This device is based on RGB LEDs with excellent voltage stability and state-of-the-art PWM control.
This device is able to display temperature-balanced color sequences with high accuracy, allowing us to probe the camera's response to various color stimuli.

![Camera Calibration Device Frame](CameraCalibratorFrame.png)
*here is a frame from the calibration sequence. Central area shows the color to be averaged later to reduce noise, four dots at the corners act as reference points for alignment and as strobe to split the sequence into samples.*

12 minutes long calibration video contains 16x16x16=4096 color samples covering the entire RGB cube uniformly.

This is measured data:
<iframe id="viewer" src="./Tools/3DLUTViewer.html?lut=../bmpcc6k.cube" width="100%" height="400px"></iframe>