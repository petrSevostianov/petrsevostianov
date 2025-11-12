
Hi everyone!
Welcome to my blog. I’m Petr Sevostianov, Co-founder and **CTO** at [Antilatency](https://antilatency.com/) and the developer of the [CyberGaffer](https://cybergaffer.com) system for virtual production lighting. My background combines **mathematics** — including **vector algebra** and optimization methods such as various **calibration techniques** — with deep experience in 3D computer graphics, **rendering** algorithms, **shaders**, and general **GPU computing**. I’m also a proficient programmer (**C++**, **C#**, **HLSL**, and others) with a strong foundation in **physics**, particularly optics, **light** behavior (excluding interference), and **mechanics**. As the CTO of Antilatency, I lead a talented team of engineers and researchers in developing cutting-edge positional tracking and virtual production solutions.
Fusing that knowledge with **hardware design** expertise allows me to build custom hardware to solve complex research problems.
 My main professional focus is **research** and prototyping, and I approach engineering challenges with a passion for making things work correctly through scientific reasoning.

In this blog, I will explore various problems related to cinematography and, in particular, virtual production. I hope you will find it interesting and useful!



### [CinemaCameraBeautifiers](./CinemaCameraBeautifiers)

This study is focused on exploring nonlinear gamut transforms done by cinema cameras. Those transforms make images non-linear and damage most of the techniques in virtual production such as keying, lighting calibration and LED wall to camera matching. Since these transforms are not publicly documented, the goal is to characterize and model them. The article describes the methodology of measuring, canceling and re-applying such transforms.