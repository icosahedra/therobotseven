---
layout: post
title:  "Level Editor Updates"
author: Brian
image: "/img/content/2015/09/improved-tessellation.png"
image-width: 435
image-height: 316
description: Fixing editor shapes
categories: 
---
I've been busy making editor improvements - one of the biggest issues was procedurally tessellating the physics geometry so that it's nicely editable in Maya.  I've done this by making sure the vertices always align between arbitrary shapes, so that it's easy to merge vertices and generate a clean mesh.  It wastes a few triangles, and it took a while to write all the cases, but the meshes are much nicer to work with.  Also it just looks nicer.
