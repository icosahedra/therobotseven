---
layout: post
title:  "Grid Physics"
author: Brian
image: /img/content/2015/08/grid_physics.png
categories: 

images:

  - url: /img/content/2015/08/collision_shapes.png
    alt: Collision Shapes
    title: Collision Shapes

  - url: /img/content/2015/08/tessellation.png
    alt: Shape Tessellation
    title: Shape Tessellation
---

I've been working on creating a physics and AI representation that is both fast to edit and reliable.  A few months ago, I'd prototyped a system using a hexagonal grid.  In addition to the tactical properties, hexagons have the advantage of creating an 'organic' feel.  However hexagonal grids also make collision calculations significantly more difficult.  I've switched to a square grid - but to keep the organic feel and add some variability to the design I've added sub-tile shapes like half-tiles, triangles, and corners.

{% assign image = page.images[0] %} 
{% include image.html image=image %}

Above is the custom editor built in Unity.  A designer can modify tile properties and then generate the 'physics', pathfinding nodes, and a reference OBJ file.  An artist can then use the OBJ as a live object for sculpting detailed final art. 

{% assign image = page.images[1] %} 
{% include image.html image=image %}