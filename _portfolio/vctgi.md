---
title: "Voxel Cone Tracing Renderer"
excerpt: "My implementation of Voxel Cone Tracing Global Illumination using my custom OpenGL engine (OPEngine). This project was built on top of my Deferred Renderer and it includes all its previous features with the addition of support for real time dynamic revoxelization and mipmapping of the entire scene as well as the conetracing pass."
header:
  image: "assets/images/samples/VCTGI/vct_header.png"
  teaser: "assets/images/samples/VCTGI/vct_teaser.png"
author_profile: true
prioritynum: 3
link: _portfolio/ClassicalPipelines.md
---

[View Source Code](https://github.com/Otaviopeixoto1/OPEngine/tree/main/src/render/VoxelConeTracing){: .btn .btn--primary .btn--x-large}
{: .text-center}
<p>
My implementation of Voxel Cone Tracing Global Illumination using my custom OpenGL engine ([OPEngine]({% link _portfolio/OPEngine.md %})). This project was built on top of [my deferred renderer]({% link _portfolio/ClassicalPipelines.md %}) and some of the code for voxelization and indirect voxel drawing was based on Conrad wahlén's work that can be found <a href="https://liu.diva-portal.org/smash/get/diva2:1148572/FULLTEXT01.pdf">here</a>.
</p>



<!-- https://github.com/BoyBaykiller/IDKEngine/blob/master/IDKEngine/Ressource/Shaders/include/TraceCone.glsl IDK ENGINE FOR REFERENCE -->

# Voxelization
For atomic average function
https://www.icare3d.org/research/OpenGLInsights-SparseVoxelization.pdf

# Conetracing
For crassin paper: https://research.nvidia.com/sites/default/files/pubs/2011-09_Interactive-Indirect-Illumination/GIVoxels-pg2011-authors.pdf

# Diffuse Indirect Lighting
