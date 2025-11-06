---
layout: page
---

# Cinema Camera Beautifiers

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

## Naming
Since I failed to find any established term for this transformation, I will call it **Beautifier** as we call it internally in [Antilatency](https://antilatency.com) company. Inverse operation will be called **Debeautifier**. This kind of naming addresses the fact that this transform is an artistic color grading, there is no single "correct" way to do it. If a vendor try to opti


## How to find this nonlinearity with a simple experiment?
The experiment called **Two Lights Experiment**.
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

![Two Lights Experiment](./TwoLightsExperiment.png)
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

Here is measured data:
{% include 3DLUTViewer lut="bmpcc6k.cube" %}
{% include Gap %}

First, let's remove transfer function.
{% include 3DLUTViewer lut="bmpcc6k_TransferFunctionRemoved.cube" %}
{% include Gap %}

As you can see on that plot, there are two transformations still present:
1. Matrix3x3 transform that responsible for brightness, color balance and linear gamut shaping.
2. Some other nonlinear transform

Let's remove Matrix3x3 transform as well.
This let us see only nonlinear transform. We will restore Matrix3x3 later in order to keep brightness and color balance and gamut unchanged.

{% include 3DLUTViewer lut="bmpcc6k_Beautifier.cube" %}
This plot is **Beautifier** alone.
{% include Gap %}

## What is the general form of the Beautifier function?
From the **Two Lights Experiment** we already know that all colors between black and white are linear. That means that we are looking for a function that does not change linearity between those colors.
Additionally, we can perform several more experiments for black-somecolor mixtures, and confirm that those mixtures are also linear.

That leads us to the conclusion that the function we are looking for is [Homogeneous function](https://en.wikipedia.org/wiki/Homogeneous_function).
<div align="center">
  <img src="HomogeneousFunctionFormula.svg" alt="f(cu) = cf(u)" height="30">
</div>

That means that if we scale input color by some factor and apply the function, it is equivalent to applying the function first and then scaling output by the same factor.

## Visualizing homogeneous transform
To visualize homogeneous transform, we can slice RGB cube by plane passing through red, green and blue points of the cube. Any distortion of that triangle will represent distortion of the whole cube, because all other colors are laing on scaled versions of this triangle.

To get such slice, we can select samples where indices R+G+B=15 (our cube is 16x16x16 samples).
Since all slices are similarly distorted, we can select any slice, but slice №15 has the highest resolution.

{% include 3DLUTViewer lut="bmpcc6k_Beautifier.cube" showChromaTriangle=16 %}

White triangle on that plot represents undistorted slice.

## In what way can these distortions be represented?
As we can see from the plot above, this distortion is unlike any smooth function such as polynomial or spline. It has almost undistorted area in the center, with significantly distorted areas near the edges. Thus the best way to represent it is using a LUT-based approach. Technically, we can use 3D LUT for that, but the problem of 3D LUT is that resolution of 3D LUT decreases with brightness. If 3D LUT has size 33x33x33 it has 32 steps between red(1,0,0) and green(0,1,0), but only 8 steps between dark-red(0.25,0,0) and dark-green(0,0.25,0). Another problem is memory consumption, for instance, to have 512 steps between red and green, we will need 512x512x512 = 134 million samples, that will take more than 1.6 GB of memory in binary form, and much more in text form.

Instead, we can invent a new type of LUT that is based on homogeneous coordinates. Such LUT will have constant resolution across brightness levels, and will require much less memory.

## Introducing 2D LUT
To have 512 steps between red and green, it uses 1.6Mb (not GB) of memory and has constant resolution across brightness levels. It can be stored as a simple 2D image. Since the shape of the discibed space is a triangle, we can pack it into a rectangle by cutting off a corner and flipping it.

**Beautifier**

![LUT2D](LUT2D.png)

**Debeautifier** (linearizer)

![LUT2DInversed](LUT2DInversed.png)

 



