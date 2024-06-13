---
title: "GPU boids (Unity)"
excerpt: "A GPU driven boid simulation and rendering made in Unity. With the aim to learn and extract the most out of GPU Driven Rendering, I have used Unity's compute shader capabilities and indirect rendering commands to simulate and draw up to one million boids."
header:
  image: "assets/images/samples/GPUBoids/hd400kBoids.png"
  teaser: "assets/images/samples/GPUBoids/boidsTeaser.png"
author_profile: true
prioritynum: 4

feature_row:
  - image_path: /assets/images/samples/GPUBoids/Rule_alignment.png
    alt: "Rule alignment"
    title: "Alignment"
    excerpt: "Alignement forces push nearby boids into movint at the same direction."
  - image_path: /assets/images/samples/GPUBoids/Rule_cohesion.png
    alt: "Rule cohesion"
    title: "Cohesion"
    excerpt: "Cohesion forces push the boids towards the central position of the nearby boids, causing agglomeration."
  - image_path: /assets/images/samples/GPUBoids/Rule_separation.png
    title: "Separation"
    excerpt: "Separation forces avoid collision between the boids inside the agglomerate"

---

[View Source Code](https://github.com/Otaviopeixoto1/UnityComputeExperiments/tree/main/Assets/GPUBoids){: .btn .btn--primary .btn--x-large}
{: .text-center}

A GPU driven boid simulation and rendering made in Unity. With the aim to learn and extract the most out of GPU Driven Rendering, I have used Unity's compute shader capabilities and indirect rendering commands to simulate and draw up to one million boids.


![image-center](/assets/images/samples/GPUBoids/200kboidsvid.gif){: .align-center}

# Boids

<p> Developed by Craig Reynolds in 1986, Boids is an algorith that tries to simulate the flocking behaviour of birds. Each boid represents an individual inside the flock, with their movements being determined by <a href="https://en.wikipedia.org/wiki/Boids"> three simple rules</a>: </p>

{% include feature_row %}

<p>The simplicity of the algorithm makes it very simple to implement, even in parallel. However, the catch is that we must check for nearest neighbors, making it scale in complexity as O(N^2) </p>

# Space Partitioning

<p> In order to solve the O(N^2) scaling and inspired by Rama C. Hoetzlein presentation on <a href="https://on-demand.gputechconf.com/gtc/2014/presentations/S4117-fast-fixed-radius-nearest-neighbor-gpu.pdf"> Fast Fixed-Radius Nearest Neighbors approach</a>, I have used a uniform spatial grid as an acceleration data structure: </p>



<figure style="width: 60%" class="align-center">
  <a href="https://on-demand.gputechconf.com/gtc/2014/presentations/S4117-fast-fixed-radius-nearest-neighbor-gpu.pdf">
  <img src="/assets/images/samples/GPUBoids/boid_space_part.png" alt=""></a>
  <figcaption>Grid based space partitioning.</figcaption>
</figure>

With this space partitioning scheme, each boid is firstly assigned into a grid cell. Then, based on Rama's presentation and using nvidia's GPU gems <a href="https://developer.nvidia.com/gpugems/gpugems3/part-vi-gpu-computing/chapter-39-parallel-prefix-sum-scan-cuda"> Work Efficient Parallel Prefix Sum (Scan)</a> We improve the memory access pattern on the GPU by sorting the boid array with counting sort. After sorting, we search for nearby boids by iterating over contiguous slices of the array.


