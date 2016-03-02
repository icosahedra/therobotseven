---
layout: post
title:  "Light Falloff"
author: Brian
image: "/img/content/2016/03/spotlight.png"
image-width: 1406
image-height: 747
description: Some inverse square law
categories: 

images:

    - url: /img/content/2016/03/standard_inverse_square_light.png
      alt: standard inverse square law graph
      title: Inverse Square Law

    - url: /img/content/2016/03/modified_inverse_square_light.png
      alt: modified inverse square law graph
      title: Inverse Square Law 2

    - url: /img/content/2016/03/modified_inverse_square_light_converge.png
      alt: converging inverse square law graph
      title: Inverse Square Law 3

    - url: /img/content/2016/03/spotlight_2.png
      alt: spotlight image
      title: Spotlight

---
When I'm not working on physics or AI, I'm trying to spend a little bit of time working on lighting.  It's how I reward myself for finishing a system I didn't really want to be working on.

I've been looking at the way we calculate light 'falloff'. The most important component of light falloff is simply the geometric loss of intensity, e.g. the [inverse square law](https://en.wikipedia.org/wiki/Inverse-square_law).  This is easy to understand - as light travels through space, it spreads out with the square of the distance.  Geometric loss and attenuation frequently get rolled into one shader equation, often mis-labeled 'attenuation' (attenuation deals with light absorption and scattering), but I'm just talking about the geometry for now.

Why am I doing this?  I'm working on an HDR workflow, I've written some tonemapping I'm happy with, but the old lighting equations just didn't quite look right, and I wanted to figure out why.  So - lets begin.

The actual physical equations for geometric intensity loss would look like this in GLSL:
{% highlight glsl %}
vec3 lightVector = worldPosition.xyz - lightPosition.xyz;
float sqrDistance = dot(lightVector, lightVector);
light_power = 1.0;
intensity = light_power / (sqrDistance);
{% endhighlight %}

Okay, but what if the distance is really small, or zero?  Our light intensity goes to infinity?  Of course not - which is why the most common version I've seen is this:
{% highlight glsl %}
light_power = 1.0;
intensity = light_power / (1 + sqrDistance);
{% endhighlight %}

Graphing (and blogging) helps me prevent a lot of silly mistakes, so let's see what that looks like.
{% assign image = page.images[0] %} 
{% include image.html image=image %}

Okay - so a few things here.  First, you can see right at the start a funny 'knee' at the left edge the graph.  Why?  Well - it's because we added that 1 to keep the function from exploding when the distance light and target became really small. This was a useful approximation when lights really needed a very small range because we were rendering to 8 bit targets - but it's 2016 and we have HDR buffers.  Maybe I want those highlights back.  Maybe I want to put my characters really close to a light.  Of course, we still don't want the light to go to infinity at very close distances, but let's be a bit more ambitious.

{% highlight glsl %}
light_power = 1.0
intensity = light_power / (0.01 + sqrDistance);
{% endhighlight %}

{% assign image = page.images[1] %} 
{% include image.html image=image %}

Boom! That's a highlight - and also a much different looking curve.  In case you can't see it, this function will continue rising to a value of 100, instead of capping out at 1 and lazily becoming less intense.  Maybe 100 is a bit much, maybe 10 is enough.  In any case, we can adjust it easily.

But we still have another problem - see that long tail on the graph?  Well, according to this function, this light will gradually lose intensity... forever.  In a pure vacuum this is true - in a game engine where we always need fewer lights affecting objects - this is bad.  Now we have some choices:

 * Don't cull lights.  Usually a bad choice, unless you only have 1 light
 * Cull lights normally, pretending this isn't a problem.  This is one cause of light popping. 
 * Cull lights when they are reeeeeealy far away.  Probably won't have popping, but it's almost the same as not culling them.
 * Fix it.

Fixing it turns out to be easy.  We can solve this in one of two ways - either by applying a fixed range, or a fixed intensity cutoff.  I'm going to calculate the range from a minimum intensity.  Why not just subtract a minimum intensity?  Well, we might want that range for shadow calculations or other optimizations, but just subtracting a small value works, too

{% highlight c# %}

float minDistanceSqr = 0.01f;  /// the same value from our shader
float minLightIntensity = 0.01f;  /// light gets culled at this point

/**** Derivation ****/
// intensity = light_power / (minDistanceSqr + sqrDistance)
// minLightIntensity = light_power / (0.01 + sqrDistance)
// 0.01 + sqrDistance = light_power / minLightIntensity
// sqrDistance = (light_power / minLightIntensity) - 0.01
// maxLightRadius = Mathf.Sqrt( (light_power / minLightIntensity) - 0.01)

float maxLightRadius = Mathf.Sqrt( (light_power / minLightIntensity) - minDistanceSqr );
{% endhighlight %}

Finally, we can fix the shader, and see that our graph does converge.  Since we subtracting, we now need to make sure our values don't become negative.
{% highlight glsl %}
light_power = 1.0
intensity = max(0, (light_power / (0.01 + sqrDistance)) - (distance / maxLightRadius) * minLightIntensity);
{% endhighlight %}

{% assign image = page.images[2] %} 
{% include image.html image=image %}

I didn't quite get to attenuation - I'll leave that for next time.  One more image for the road.

{% assign image = page.images[3] %} 
{% include image.html image=image %}

