---
layout: post
title:  "Volumetrics"
author: Brian
image: "/img/content/2015/04/volumetrics.png"
categories: 
---
This is a volumetic rendering test using path-tracing, and simplified mie scattering.  It runs on an Intel HD 5000 integrated graphics card, shader model 3.0 (DX 9) at about 30fps.  After getting this working, I've decided to move entirely to DX11 and use volumetric textures instead.  While this path tracing method is neat, and the results are high quality, it's simply too slow.

The resolution of this technique is directly related to the sampling distance.  You can dramatically reduce artifacts by randomly offsetting (dithering) each path (so if you sample along each ray at 1m intervals, start each ray offset between 0-0.5 ).  One interesting discovery - if you randomize the offset every frame, you end up with an effect that looks sort of like film grain.  
