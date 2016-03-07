---
layout: post
title:  "Triplanar Texturing with Derivative Maps"
author: Brian
image: "/img/content/2016/03/triplanar.gif"
image-width: 600
image-height: 600
description: Triplanar Texturing with Derivative Maps
categories: 

images:

    - url: /img/content/2016/03/triplanar_world.png
      alt: triplanar texturing with primatives
      title: Triplanar texturing with primatives


---

We're building a fairly large world, and want to be able to work with modular parts.  One of the biggest tech-art challenges with modularity is texturing.  How do you ensure the textures fit together without having obvious tiles?  With a small team, we can't really afford to manually texture every scene.  Alternatively, we could make sure each asset is self-contained, but then the visual quality suffers as the tiling becomes visually apparent.  

Enter tri-planar projection.  

Here is the excellent paper describing the core of this technique (as well as a few other tricks I haven't yet implemented).
[NVidia GPUGems3: Generating Complex Procedural Terrains Using the GPU](http://http.developer.nvidia.com/GPUGems3/gpugems3_ch01.html)

By projecting a texture from the X, Y, and Z axes in world space - we can create a somewhat seamless world.  This is the method many voxel engines use to procedurally texture the environment.  Where the textures intersect, there is some linear blending between the textures.  

{% highlight hlsl %}
float3 vNormal = normalize(fragment.normal);

float3 triblend = vNormal*vNormal;
triblend= triblend / (triblend.x + triblend.y + triblend.z);

//_TextureScale is the tiling of the texture in world space
float3 uvs = _TextureScale * fragment.worldPosition.xyz;

float3 albedoY = tex2D(_MainTex, uvs.xz); 
float3 albedoX = tex2D(_MainTex, uvs.zy); 
float3 albedoZ = tex2D(_MainTex, uvs.xy); 

float3 albedo = (triblend.y * albedoY) + (triblend.x * albedoX) + (triblend.z * albedoZ);
{% endhighlight %}

Linear blending is fine for color, AO, and spec maps - but how do we blend the normals?

Stephen Hill posted a great article detailing how one might go about blending two normal maps together.  In short, it can be done, but you of course need to renormalize.
[Blending in Detail](http://blog.selfshadow.com/publications/blending-in-detail/)

My preferred method is what Stephen refers to as 'whiteout blending' - from the AMD Ruby: Whiteout demo.
{% highlight hlsl %}
float3 r = normalize(float3(n1.xy + n2.xy, n1.z*n2.z));
{% endhighlight %}

But we still have another problem - since we are using world space UV coordinates - tangent space normal maps won't work directly - we don't have tangents.  Okay, okay - we can define a tangent basis using the projection axes, but we still end up with visual artifacts.  I soon found there was a better way.

Derivative maps.  I might never create a tangent space normal map again.  Why?

 * Derivative Maps don't require tangents or binormals.
 * Derivative Maps only need two channels.
 * Derivative Maps don't suffer from tangent seams.
 * Derivative Maps can be blended using alpha blending, without renormalization.

 Want to know more?

 Morton Mikkelsen has a paper, which is not exactly a light read - and a blog which helps a bit.
 * [Bump Mapping Unparametrized Surfaces on the GPU](https://dl.dropboxusercontent.com/u/55891920/papers/mm_sfgrad_bump.pdf)
 * [Morten Mikkelsen Blog](http://mmikkelsen3d.blogspot.com/2011/07/derivative-maps.html)

It's great work - it's just also a bit, er, academic, which is to say, heavy on mathematical jargon.  Rory Driscoll's two pages on derivative maps were extremely helpful for implementation.
 * [Rory Driscoll Derivative Maps](http://www.rorydriscoll.com/2012/01/11/derivative-maps/)
 * [Rory Driscoll Derivative Maps vs Normal Maps](http://www.rorydriscoll.com/2012/01/15/derivative-maps-vs-normal-maps/)

The gist of it is we can calculate a local basis for bump mapping by using the partial derivative functions ddx() and ddy().  In the original paper, Mikkelsen is using 8-bit height maps and ddx_fine() on DX11.  However you can reduce the shader complexity, and support SM3.0 by instead precalculating a derivative map.


Here is where it really gets awesome.  We're evaluating 3 derivative maps per-pixel, which sounds bad - but since we are calculating the UV basis in world space:

{% highlight hlsl %}
float3 ws_derivative_x = ddx(fragment.worldPosition)
float3 ddx_UV =  ws_derivative_x * _TextureScale;
float2 ddx_UV_X = ddx_UV.zy;
float2 ddx_UV_Y = ddx_UV.xz;
float2 ddx_UV_Z = ddx_UV.xy;
{% endhighlight %}

Since the world position doesn't change between maps, we only need to calculate ddx(worldPosition) and ddy(worldPosition).  We can swizzle our way into all the UV partials we need.  It gets better if you keep optimizing - most of the math from one derivative map can be reused by the other two - so aside from the two additional texture lookups and blending - it's almost free - no need to continually renormalize.

And lets not forget - here is what the VTF struct looks like now:

{% highlight hlsl %}
struct VertexToFragment {
  float4 position : SV_POSITION;
  float3 normal : TEXCOORD0;
  float3 worldPosition : TEXCOORD1;
};
{% endhighlight %}

No UVs.  No Tangents.  No Binormals.  That's a lot of spare interpolators for other fun things.  Also, meshes can be significantly smaller, and I never have to worry about whether normal maps were generated with the correct tangent basis.  I'm considering abandoning tangent space normal maps everywhere.

As a final example - here is a seamlessly textured object built out of standard primitives.

{% assign image = page.images[0] %} 
{% include image.html image=image %}

Helpful Links

 * [OpenGL Tutorial on Normal Mapping](http://www.opengl-tutorial.org/intermediate-tutorials/tutorial-13-normal-mapping/)
 * [Unity Shaderlab Properties](http://docs.unity3d.com/Manual/SL-PropertiesInPrograms.html)

 


