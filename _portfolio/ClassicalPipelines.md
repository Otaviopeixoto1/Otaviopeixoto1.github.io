---
title: "Classical Rendering Pipelines"
excerpt: "The classical Deferred and Forward rendering pipelines implemented on my custom engine (OPEngine). They come with features such as Shadow Mapping, Normal Mapping, multiple point and directional lights, Anti-Aliasing (MSAA and FXAA), hdr and tonemapping, skybox..."
header:
  image: "assets/images/samples/classicpipelines/classic_pipelines.png"
  teaser: "assets/images/samples/classicpipelines/shadows_deferredLights_cropped.png"
author_profile: true
prioritynum: 1


gallery1:
  - url: /assets/images/samples/classicpipelines/nonorm.png
    image_path: /assets/images/samples/classicpipelines/nonorm.png
    alt: "placeholder image 1"
    title: "Flat normals"
  - url: /assets/images/samples/classicpipelines/normalMapped.png
    image_path: /assets/images/samples/classicpipelines/normalMapped.png
    alt: "placeholder image 2"
    title: "Normal mapped"

gallery2:
  - url: /assets/images/samples/classicpipelines/albedo.png
    image_path: /assets/images/samples/classicpipelines/albedo.png
    alt: "placeholder image 1"
    title: "Flat normals"
  - url: /assets/images/samples/classicpipelines/normal.png
    image_path: /assets/images/samples/classicpipelines/normal.png
    alt: "placeholder image 2"
    title: "Normal mapped"
  - url: /assets/images/samples/classicpipelines/pos.png
    image_path: /assets/images/samples/classicpipelines/pos.png
    alt: "placeholder image 2"
    title: "Normal mapped"

gallery3:
  - url: /assets/images/samples/classicpipelines/lighting.png
    image_path: /assets/images/samples/classicpipelines/lighting.png
    alt: "placeholder image 1"
    title: "Flat normals"
  - url: /assets/images/samples/classicpipelines/tonemap.png
    image_path: /assets/images/samples/classicpipelines/tonemap.png
    alt: "placeholder image 2"
    title: "Normal mapped"
---

[View Source Code](https://github.com/Otaviopeixoto1/OPEngine/tree/main/src/render/Classical){: .btn .btn--primary .btn--x-large}
{: .text-center}

<p>The classical Deferred and Forward rendering pipelines implemented on my custom engine (OPEngine). They come with features such as Cascaded and Filtered Shadow Mapping, Normal Mapping, multiple point and directional lights, Anti-Aliasing (MSAA and FXAA), hdr and tonemapping, skybox and more.</p>



![image-center]( /assets/images/samples/classicpipelines/renderForward.png){: .align-center}


# Deferred Rendering
<p> For Deferred Rendering the G-buffers structure I chose is the one shown below</p>
{% include gallery id="gallery2" caption="From left to right: albedo, view space normals and view space position" %}

# Normal Mapping
<p>Normal maps can be turned on and off for both the classical pipelines</p>
{% include gallery id="gallery1" caption="Sponza Atrium wall without vs with Normal Mapping" %}

# Tonemapping
<p>For the final rendered images, I chose to apply the gamma correction alongside a simple exposure tonemapping: </p>
{% highlight c++ %}
{
  // ...
  vec3 mapped = vec3(1.0) - exp(-hdrColor * exposure);

  // gamma correction 
  mapped = pow(mapped, vec3(1.0 / gamma));

  FragColor = vec4(mapped, 1.0);

}
{% endhighlight %}


{% include gallery id="gallery3" caption="On the left we have the original image with colors in hdr, while on the right we have the gamma corrected and tonemapped final rendered image" %}

