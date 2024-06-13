---
title: "Voxel Cone Tracing Renderer"
excerpt: "My implementation of Voxel Cone Tracing Global Illumination using my custom OpenGL engine (OPEngine). This project was built on top of my Deferred Renderer and it includes all its previous features with the addition of support for real time dynamic revoxelization and mipmapping of the entire scene as well as the conetracing pass."
header:
  image: "assets/images/samples/VCTGI/vct_header.png"
  teaser: "assets/images/samples/VCTGI/vct_teaser.png"
author_profile: true
prioritynum: 3

gallery1:
  - url: /assets/images/samples/VCTGI/vct_ao.png
    image_path: /assets/images/samples/VCTGI/vct_ao.png
    alt: "placeholder image 1"
    title: "Ambient Occlusion"
  - url: /assets/images/samples/VCTGI/vct_direct.png
    image_path: /assets/images/samples/VCTGI/vct_direct.png
    alt: "placeholder image 2"
    title: "Direct Illumination"
  - url: /assets/images/samples/VCTGI/vct_indirect.png
    image_path: /assets/images/samples/VCTGI/vct_indirect.png
    alt: "placeholder image 2"
    title: "Indirect Illumination"

gallery2:
  - url: /assets/images/samples/VCTGI/vct_final.png
    image_path: /assets/images/samples/VCTGI/vct_final.png
    alt: "placeholder image 1"
    title: "Final rendered scene"

gallery3:
  - url: /assets/images/samples/VCTGI/vct_nogi.png
    image_path: /assets/images/samples/VCTGI/vct_nogi.png
    alt: "placeholder image 1"
    title: "Scene with direct illumination and no ambient lighting"
  - url: /assets/images/samples/VCTGI/vct_gion.png
    image_path: /assets/images/samples/VCTGI/vct_gion.png
    alt: "placeholder image 2"
    title: "Voxel cone traced scene with one bounce of light"
---

[View Source Code](https://github.com/Otaviopeixoto1/OPEngine/tree/main/src/render/VoxelConeTracing){: .btn .btn--primary .btn--x-large}
{: .text-center}

<p>
My implementation of Voxel Cone Tracing Global Illumination using my custom OpenGL engine (<a href="https://otaviopeixoto1.github.io/portfolio/OPEngine/">OPEngine</a>). This project was built on top of <a href="https://otaviopeixoto1.github.io/portfolio/ClassicalPipelines/">my deferred renderer</a> and some of the code for voxelization and indirect voxel drawing was based on Conrad Wahlén's work that can be found <a href="https://liu.diva-portal.org/smash/get/diva2:1148572/FULLTEXT01.pdf">here</a>. By addapting Conrad's approach, I can voxelize the final scene in real time while generating a sparse voxel list containing all voxels that are filled. This list is then used for drawing the voxelized scene and for unpacking the voxels into a final RGBA texture format.
</p>



<!-- https://github.com/BoyBaykiller/IDKEngine/blob/master/IDKEngine/Ressource/Shaders/include/TraceCone.glsl IDK ENGINE FOR REFERENCE -->

# Voxelization
<p> The scene voxelization is done using a geometry shader in order to output the final triangles projected along their dominant axis to maximize the amount of fragments generated to avoid holes on our objects. The geometry shader for dominant axis selection is outlined below:  </p>


{% highlight c++ %}
float maxComponent = max(normal.x, max(normal.y, normal.z));
uint ind = maxComponent == normal.x ? 0 : maxComponent == normal.y ? 1 : 2;
domInd = ind;

for (int i = 0; i < 3; i++)
{
  worldPosition = vWorldPos[i];
  viewPosition = viewMatrix * worldPosition;
  TexCoords = vTexCoords[i];
  voxelTexCoord = (gl_in[i].gl_Position.xyz + vec3(1.0f)) * 0.5f;

  if (ind == 0) gl_Position = vec4(gl_in[i].gl_Position.zyx, 1);		
  else if (ind == 1) gl_Position = vec4(gl_in[i].gl_Position.xzy, 1);
  else if (ind == 2) gl_Position = vec4(gl_in[i].gl_Position.xyz, 1);

  EmitVertex();
}
{% endhighlight %}

<p>
In addition, the NV_conservative_raster extension was used to guarantee that evert voxel intesected by a triangle is filled.
</p>

<p> In order to revoxelize the scene in real time without artifacts, we must average the fragment colors for the final output. However, this can be problematic since atomic operations are not available by default for the floating point values stored in RGB texture formats. One approach to this is to rely on another extension called NV_shader_atomic_float, but for learning purposes I chose to use a more manual approach like the one mentioned in <a href="https://www.icare3d.org/research/OpenGLInsights-SparseVoxelization.pdf"> OpenGL Insights: Octree-Based Sparse Voxelization Using the GPU Hardware Rasterizer</a> by Cyril Crassin and Simon Green. In their approach, the voxel texture is chosen to have an unsigned integer format where the colors are packed. We then use the imageAtomicCompSwap() function to average all fragment contributions: </p>


{% highlight c++ %}
uint nextUint = packUnorm4x8(vec4(color.rgb, 1.0 / 255.0f));
uint prevUint = 0;
uint curUint = imageAtomicCompSwap(voxel3DData, voxelCoord, prevUint, nextUint);

// atomically average the color
while(curUint != prevUint) 
{
  prevUint = curUint;

  // Unpack newly read value, and counter.
  vec4 avg = unpackUnorm4x8(curUint);
  uint count = uint(avg.a * 255.0f);
  // Calculate and pack new average.
  avg = vec4((avg.rgb * count + color.rgb) / float(count + 1), float(count + 1) / 255.0f);
  nextUint = packUnorm4x8(avg);

  curUint = imageAtomicCompSwap(voxel3DData, voxelCoord, prevUint, nextUint);
}
{% endhighlight %}

<p>
Before the final color is stored, a sparse voxel list is also filled in order to issue indirect draw calls when rendering the voxelized scene. Another use of the sparse voxel list is for unpacking the final texture into RGBA8 format. In order to reduce the amount of threads used, we also fill an indirect command buffer for the compute shader so we can only read and write into voxels that are occupied.</p>

<figure class="align-center">
  <a>
  <img src="/assets/images/samples/VCTGI/vct_voxelization.png" alt=""></a>
  <figcaption>Voxelized scene with direct Illumination</figcaption>
</figure>


# Conetracing

After Voxelizing and mipmapping the scene on a simple box filter compute shader, the conetracing pass begins by sampling the Gbuffer to find the starting positions and the direction of the cones. Finally, the accumulated color as well as an estimation of abient occlusion can be obtained as follows:

{% highlight c++ %}
// ...
for(float dist = voxelSize ; dist < maxConeDistance && accum.a < accumThr;) 
{
  //coneDiameter = 2.0 * tan(60) = 3.464101615
  coneDiameter = 3.464101615 * dist;
  sampleDiameter = max(voxelSize, coneDiameter);
  sampleLod = log2(sampleDiameter / voxelSize);

  samplePos = startPos + dir * dist;

  sampleValue = textureLod(voxelColorTex, samplePos, sampleLod).rgba;

  accum += (1.0f - accum.a) * sampleValue;
  opacity = (dist < aoDist) ? accum.a : opacity; 

  dist += sampleDiameter * stepMultiplier;
}

return vec4(accum.rgb, 1.0f - opacity);

// ...
{% endhighlight %}



{% include gallery id="gallery1" caption="On the left we have the ambient occlusion from the conetracing, while on the the middle there is the direct illumination results from deferred shading and on right is the conetraced accumulated color" %}


# Diffuse Indirect Lighting
The final combination of the direct illumination with the one bounce of diffuse indirect illumination and the ambient occlusion can be seen below:

{% include gallery id="gallery2" caption="Final result" %}

{% include gallery id="gallery3" caption="Scene rendered without Global illumination and no ambient lighting (left) versus scene with one bounce of diffuse indirect illumination from VCTGI" %}