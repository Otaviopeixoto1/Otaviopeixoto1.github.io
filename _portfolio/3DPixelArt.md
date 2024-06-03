---
title: "3D Pixel Art Renderer (Unity)"
excerpt: "This project was made with the aim to replicate the pixel art style when rendering 3D meshes. By customizing Unity's rendering pipeline and using a series of postprocessing techniques, the effect is achieved and, from crude simple 3D models, we can produce a stable pixelization effect that gives a stylized look to our rendererd scene."
header:
  image: "assets/images/samples/3DPixelart/3DPixlart_big.png"
  teaser: "assets/images/samples/3DPixelart/pixelation.png"
author_profile: true
prioritynum: 6
---

[View Source Code](https://github.com/Otaviopeixoto1/Project-Howl/tree/main/Assets/3DPixelArt){: .btn .btn--primary .btn--x-large}
{: .text-center}

<p>
This project was made with the aim to replicate the pixel art style when rendering 3D meshes. By customizing Unity's rendering pipeline and using a series of postprocessing techniques, the effect is achieved and, from crude simple 3D models, we can produce a stable pixelization effect that gives a stylized look to our rendererd scene.</p>


# Pixelization Effect
<p>Pixelization is achieved by rendering the initial image to a small render target and then upscaling it to the final screen resolution with Nearest texture filtering. However, this approach alone will cause some terrible artifacts arround the edge of objects when we move the camera arround: </p>


{% include video id="cpYk-ANneAI " provider="youtube" %}




# Stable Camera
<p>In order to stabilize the camera, we must make sure that it only moves at discrete steps of 1 pixel. This can be done by converting the the camera target position into view space coordinates. Then, using the total amout of pixels per unit, we can convert from view space to pixel coordinates and snap to the closest integer:</p>


{% highlight c# %}
void Update()
{
  cameraHeight = mainCamera.orthographicSize * 2;
  cameraWidth = cameraHeight * (16/9f);

  float xPixelsPerUnit = (renderResolution.x/cameraWidth); 
  float yPixelsPerUnit = renderResolution.y/cameraHeight;
  
  Vector3 targetCoord = target.transform.position - origin;

  // pixel coordinates
  float xc =  Vector3.Dot(targetCoord, transform.right) * xPixelsPerUnit;
  float yc =  Vector3.Dot(targetCoord, transform.up) * yPixelsPerUnit;
  float z =  Vector3.Dot(targetCoord, transform.forward);
  
  //pixel snap
  float _xc = Mathf.RoundToInt(xc);
  float _yc = Mathf.RoundToInt(yc);
  
  
  Vector3 correctedPos = (_xc/xPixelsPerUnit) * transform.right  
                          + (_yc/yPixelsPerUnit) * transform.up
                          + (z - zdist) * transform.forward ;

  
  transform.position = correctedPos + origin;
  
  // ...
}
{% endhighlight %}

<p> This results in stable pixels but we can see the camera snapping, causing a slight stuttering effect:</p>
{% include video id="SYyWcx-E5rI " provider="youtube" %}

<p> The final stable and smooth result can be obtained by rendering the previous result into a quad and then setting up a secondary camera to render that quad with one pixel of extra margin and then offset this camera by the difference between original and snapped coordinate:</p>

{% highlight c# %}
void Update()
{
  // ...
  Vector2 pixelOffset = new Vector2(_xc - xc,  _yc - yc);
  secondaryCamera.SetPixelOffset(pixelOffset); 
}

// On the secondaryCamera class we define:
public void SetPixelOffset(Vector2 offset)
{
    transform.localPosition = cameraCenter - new Vector3(offset.x * 16.0f/9.0f, offset.y, 0);
}

{% endhighlight %}




{% include video id="YOhI5M1Ac4Q" provider="youtube" %}


# Outlines
<p> The outlines are a final touch that gives life to the pixel art style. In this case, I simply sampled the depth and normal textures at the center and at the 4 neighboring pixels and calculating the gradients. The total sum of the gradient components is used as the outline strength: </p>


{% highlight c++ %}

float2 GetDepthStrength(float depthSamples[5],float uvy)
{
    float biasedDiff = 0;
    float diff = 0;
    for(int i = 1; i < 3 ; i++)
    {
        diff += (depthSamples[i] - depthSamples[0]);
        biasedDiff += clamp(-(depthSamples[i] - depthSamples[0]),0,1);
    }
    for(int i = 3; i < 5 ; i++)
    {
        diff += (depthSamples[i] - depthSamples[0]); // *0.5 perspective
        biasedDiff += clamp(-(depthSamples[i] - depthSamples[0]),0,1); //  *0.5 perspective
    }
    
    diff = diff < 0.0;

    float2 depthStrength = float2(smoothstep(_DLower , _DUpper , biasedDiff), diff);
    //float2 depthStrength = float2(smoothstep(_DLower , _DUpper , biasedDiff), diff);
    return depthStrength; 
}

float GetNormalStrength(float3 normalSamples[5], float depthIndicator, float3 directionBias)
{
    float normalIndicator = 0;
    for(int i = 1; i < 5 ; i++)
    {
        float sharpness = 1 - dot(normalSamples[0], normalSamples[i]);
        float3 normaldiff = normalSamples[0] - normalSamples[i];

        float normalBias = smoothstep(-0.01f, 0.01f, dot(normaldiff, directionBias));
        normalIndicator += sharpness * normalBias;
    }

    float normalStrength = smoothstep(_NLower,_NUpper, normalIndicator * depthIndicator);
    return normalStrength;
}

// ...

float4 frag()
{

  // ...

  float strength = depthStrength.x * (1 - 0.5 * depthStrength.x) 
                  +(1-depthStrength.x) * (1 + 0.5 * normalStrength);

  return color * strength;
}
{% endhighlight %}


<p>Resulting in the final render:</p>


![image-center](/assets/images/samples/3DPixelart/finalrender3dpxlart.png){: .align-center}