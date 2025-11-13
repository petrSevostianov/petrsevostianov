## Solution to the LED Wall Calibration problem of Virtual Production.

Hello! I'm Petr Sevostianov, CTO and co-founder of Antilatency. I have written an article about an important problem in virtual production related to cinema cameras and their non-linear color output.

Besides the well-known transfer functions (e.g. SLog3, LogC4 ..) applied by cinema cameras, there is another transform operation: a **non-linear gamut transform**. This transform distorts colors in gamut space and breaks linearity.

Linearity is crucial for virtual production tasks such as lighting calibration, keying, and LED wall to camera matching.

We have learnt how to **measure**, **characterize** this transform.
So we can not only **cancel** this non-linearity out, but also **re-apply** it back at the end of the pipeline to get the original look.

You can read the full article here:
[https://petrsevostianov.github.io/CinemaCameraBeautifiers/](https://petrsevostianov.github.io/CinemaCameraBeautifiers/)

In this article, I describe our methodology, experiments, and findings in detail, complete with images and interactive 3D visualizations.

I hope you find this research useful and comprehensive. Feel free to reach out if you have any questions or would like to discuss further. Join me in my quest to push virtual production towards transparent and reproducible workflows!