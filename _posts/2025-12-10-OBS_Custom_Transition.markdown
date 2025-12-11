---
layout: post
title:  "OBS Custom transition with a Shader"
date:   2025-12-11 19:00:00 +0200
categories: ["Shader"]
---

This is something I wrote for my own model debut, as I knew I won't be able to afford a custom stinger transition at the time of the debut.
Plus I wanted the transition to be a bit out of ordinary - stinger transitions are great, but they do not allow you to modify pixel positions from scene A/B - only fade between A/B.
Shaders don't have that limitation. This initially started with me planning out everythin in Unity's shader graph to have a better view on each stage of the transition and once I had a general concept, I started translating shader graph to shader language used OBS Shader component.
The original transition was suppose to be using sampling from scene A and B, logo and a something similar to a normal texture to produce a much more interesting and complex transition. However mid-project I have run into an issue that seems to be related to a limitation of using only 3 textures that can be sampled. With 4, the results would just break with OBS Shader plugin not providing any error message.

As a workaround, I was planning on looking up how to create hexagons using pixel shaders, however due to the debut being set in stone, I have eventually decided to just simplify it to a simple wave pattern (which I am not exactly happy with).

{% assign preview = "2025-12-11-TransitionPreview.png" %}
[![{{ preview }}](/images/articles/{{ preview }})](https://www.youtube.com/watch?v=2xrZEjVzFks)
If you want, you can always follow me on [Twitch](https://www.twitch.tv/sui_vt) or [YouTube](https://www.youtube.com/@Sui_VT).


While I won't provide you with a logo, you are free to use this code with your own logo if you find it interesting:
```hlsl
// Glass-shield transition
// written by SuiMachine / Sui_VT.
// You are free to modify it as you please.
// Credit is not required but appriciated.
//
// There may be a few sections in here that seem like they are overcomplicated but may be intentional
// For example I initially wrote a shader so that it built UVs for a logo and sampleed image_a or image_b
// but then run into an issue where sampling background with normal coorinates would result
// in old data being overriden, so I had to rewrite it in a way where image_a or image_b are sampled only once!

uniform texture2d image_a;
uniform texture2d image_b;
uniform float transition_time = 0.5;
uniform float Aspect_ratio = 1.77777777778;
uniform bool convert_linear = true;

uniform texture2d LogoTexture = "SuiLogo.png";
uniform float2 LogoColorDistortionXY = { 0.2, 0.05 }; //Made it look cooler, though
uniform float LogoColorDistortion_Strength = 0.51;

uniform float3 Logo_Shine_Line = {1, 1, 1};

uniform float Logo_scale = 0;
uniform float2 Logo_positionXY = {0, 0};

uniform float Start_Logo_scale = 10;
uniform float2 Start_Logo_positionXY = {0, 0};

uniform float Distortion_wave_strength = 1;

uniform float phase2Start = 1;
uniform float phase3Start = 1;

float2 GetCenteredUV(float2 uvIn) { return uvIn - float2(0.5, 0.5); }

float2 GetCenteredUVScaled(float2 uvIn, float X_scale)
{
  float2 result = uvIn - float2(0.5, 0.5);
  return result * float2(X_scale, 1.0);
}

float2 RestoreNormalUVCoordinate(float2 uvIn) { return uvIn + float2(0.5, 0.5); }
float2 MirrorUV(float2 uv) { return 1.0 - abs(frac(uv * 0.5) * 2.0 - 1.0); }

//Based on https://realtimevfx.com/t/collection-of-useful-curve-shaping-functions/3704
float SmoothSlowdownCurve(float x) { return 1.0 - pow(abs(x - 1.0), 3.5); }

uniform float4 background_color = {0.0, 0.0, 0.0, 1.0};

void GetTransformedLogo(float2 screenUV, float2 uv, float distortionStrength, out float3 logoPixel, out float2 distortedUV, out float logoAlpha)
{
  float4 sampledLogo = LogoTexture.Sample(textureSampler, uv);
  logoAlpha = sampledLogo.a;
  logoPixel = sampledLogo.rgb;
  
  float2 logoUVDistortion = sampledLogo.rg - LogoColorDistortionXY; //This may need to be modified for blue-ish logos
  logoUVDistortion = logoUVDistortion * distortionStrength;
  
  float2 distoredSceneAUV = screenUV + logoUVDistortion;
  distortedUV = MirrorUV(distoredSceneAUV);  
}

float GetGradient(in float2 screenUV, float percent)
{
    //Create diagnal gradient
    float gradientX = screenUV.x * Aspect_ratio;
    float gradientY = screenUV.y;

    gradientX = gradientX - 0.88;
    gradientY = gradientY - 0.5;
    
    float fullGradient = gradientX + gradientY;

    float tVal = lerp(1.5, -2.5, percent); //this might need to be adjusted on wider aspect ratios :-/
    fullGradient = fullGradient - tVal;
    fullGradient *= 10; //sharper edge
    return fullGradient;
}

void GetWaveyParttern(in float2 centeredScaledUV, in float2 centeredUV, float percent,  out float ramp,  out float visibility)
{
  float len = length(centeredScaledUV);
  float tValue = lerp(-2, 2, percent);
  visibility = len + tValue;

  float secondaryValue = 1 - visibility;
  secondaryValue = secondaryValue * secondaryValue;
  secondaryValue = 1 - secondaryValue;
  ramp = secondaryValue;

  visibility = clamp(visibility, 0, 1);
}

float4 mainImage(VertData v_in) : TARGET
{
  float2 squareImageScaledUVs = GetCenteredUVScaled(v_in.uv, Aspect_ratio);
  float2 recenteredUVs = GetCenteredUV(v_in.uv);

  if(transition_time < phase2Start)
  {
    float tempT = transition_time / phase2Start;

    //Transform start and UVs
    float2 logoPosition = squareImageScaledUVs / float2(Logo_scale, Logo_scale);
    logoPosition += float2(Logo_positionXY.x * -1.0, Logo_positionXY.y);

    float2 logoPositionStart = squareImageScaledUVs / float2(Start_Logo_scale, Start_Logo_scale);
    logoPositionStart += float2(Start_Logo_positionXY.x * -1.0, Start_Logo_positionXY.y);

    //Use linear interpoation over time and sample pixel
    float2 finalPosition = lerp(RestoreNormalUVCoordinate(logoPositionStart), RestoreNormalUVCoordinate(logoPosition), SmoothSlowdownCurve(tempT));

    float3 logoPixel;
    float2 distortedLogoPixel;
    float logoAlpha;
    GetTransformedLogo(v_in.uv, finalPosition, LogoColorDistortion_Strength, logoPixel, distortedLogoPixel, logoAlpha);

    //Mix with background
    float2 combinedUV = lerp(v_in.uv, distortedLogoPixel, logoAlpha);
    float3 result = image_a.Sample(textureSampler, combinedUV).rgb;

    if(convert_linear)
      result = srgb_nonlinear_to_linear(result);
    return float4(result, 1.0);
  }
  else if(transition_time < phase3Start)
  {
    float tempT = (transition_time - phase2Start) / (phase3Start - phase2Start);
    //Transform start and UVs
    float2 logoPosition = squareImageScaledUVs / float2(Logo_scale, Logo_scale);
    logoPosition += float2(Logo_positionXY.x * -1.0, Logo_positionXY.y);
    logoPosition = RestoreNormalUVCoordinate(logoPosition);

    float sceneVisiblity;
    float rampVisibility;
    GetWaveyParttern(squareImageScaledUVs, recenteredUVs, tempT, rampVisibility, sceneVisiblity);

    float2 distortedUV2 = recenteredUVs *  (1- clamp(rampVisibility, 0, 1));
    distortedUV2 = distortedUV2 + 0.5;
    distortedUV2 = v_in.uv + distortedUV2;
    distortedUV2 = distortedUV2 * 0.5;

    //Mix with background
    float3 logoPixel;
    float2 logoDistoredUV;
    float logoAlpha;
    GetTransformedLogo(distortedUV2, logoPosition, LogoColorDistortion_Strength, logoPixel, logoDistoredUV, logoAlpha);

    float2 sampledBackgroundUV = lerp(v_in.uv, distortedUV2, rampVisibility);
    sampledBackgroundUV = lerp(sampledBackgroundUV, logoDistoredUV, logoAlpha);
    float3 distoredBackgroundA = image_a.Sample(textureSampler, sampledBackgroundUV).rgb;
    float3 distoredBackgroundB = image_b.Sample(textureSampler, sampledBackgroundUV).rgb;

    float gradientValue = GetGradient(v_in.uv, tempT);
    float clampedGadient = clamp(gradientValue, 0, 1);
    float appearLine = clamp(1 - (gradientValue * gradientValue), 0, 1);
    appearLine = appearLine * logoAlpha;
    
    float3 blendedBackgrounds = lerp(distoredBackgroundA, distoredBackgroundB, sceneVisiblity);
    float3 blendedWithLogo = lerp(blendedBackgrounds, logoPixel, logoAlpha);
    float3 result = lerp(blendedBackgrounds, blendedWithLogo, clampedGadient);
    result += appearLine * Logo_Shine_Line;

    if(convert_linear)
      result = srgb_nonlinear_to_linear(result);
    return float4(result, 1.0);
  }
  else
  {
    float tempT = (transition_time - phase3Start) / (1 - phase3Start);

    float tempSub1 = clamp(tempT * 2, 0, 1);
    float tempSub2 = clamp((tempT * 2) - 1, 0, 1);

    float2 finalPosition = squareImageScaledUVs / float2(Logo_scale, Logo_scale);
    finalPosition += float2(Logo_positionXY.x * -1.0, Logo_positionXY.y);

    float3 logoPixel;
    float2 distortedLogoPixel;
    float logoAlpha;
    GetTransformedLogo(v_in.uv, RestoreNormalUVCoordinate(finalPosition), LogoColorDistortion_Strength, logoPixel, distortedLogoPixel, logoAlpha);

    float2 combinedUV = lerp(v_in.uv, distortedLogoPixel, logoAlpha);
    float3 distortedBackground = image_b.Sample(textureSampler, combinedUV);
    float3 blendedWithLogo = lerp(distortedBackground, logoPixel, logoAlpha);
    float3 clearBackground = image_b.Sample(textureSampler, v_in.uv);

    float gradientValue = GetGradient(v_in.uv, tempSub1);
    float clampedGadient = clamp(gradientValue, 0, 1);
    float appearLine = clamp(1 - (gradientValue * gradientValue), 0, 1);

    float3 result = lerp(blendedWithLogo, clearBackground, clampedGadient);
    result = lerp(result, distortedBackground, appearLine);

    if(convert_linear)
      result = srgb_nonlinear_to_linear(result);
    return float4(result, 1.0); 
  }
}
```