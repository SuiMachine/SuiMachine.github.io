---
layout: post
title:  "VTube Texture Compression - why it should be implemented and what are its problems"
date:   2026-01-31 10:00:00 +0200
categories: ["BCn" ]
---

This blog entry is written in a hope that it can convince Denchi to implement BCn texture compression into VTube Studio. It should hopefully gather all necessity keypoints in a single spot with a few references to external sources to prove the viability of BC7 texture compression in VTube Studio.

## The problem BCn helps with
As it stands VTube Studio is a resource hog in terms of VRAM usage. Almost all textures provided to users are in PNG format that then Unity decompresses to [ARGBA32][ARGB32]. This a problem as Live2D requires many layers for user's toggles and a decent pixel density, which can result in Live2D models using extreme amounts of VRAM. This is not a given, as talking with some Vtubers I know some of their models are light to a point where they can use as little as 85.33 MB of VRAM for textures, but as the model's complexity increases, so does their VRAM size - to a point, where their textures can take over 3 GB of VRAM.

With the current DRAM market focusing on quick profits and targeting datacenters over regular users over the last 2 months (as of the moment of writing) we have seen a massive increase in the price of RAM sticks. This is predicted to least for the next few years, with datacenters predicted to consume 70% of world's RAM production. On a positive side this doesn't necessary have to affect all VTube Studio users, provided they already have a decent amount of RAM already. The more worrying part are predictions regarding the rise of prices of GPUs, as they are also affected by the DRAM shortages (and companies specializing in DRAM production are prioritizing HBM memory, which results in lower availability of graphics cards with 12GB or 16GB of VRAM, which are preferable cards for VTubers).

Having recently moved to 16 GB graphics card and being constrained with streaming to 8 GB of VRAM for the last 4 years, I believe that that 8 GB is not enough for any streamer - not just Vtubers (The Callisto Protocol or Dead Space Remake have already been running out of VRAM back in 2022, early 2023 even on relatively low details). **Vtubers especially will have to deal with this problem, due to how VRAM expensive their models are. As such, I would highly advise implementation of DDS texture format and BC7 in PC version of VTube Studio to drastically reduce the VRAM usage of Live2D models and avoid the issues often present in cheaper DX5/BC3 texture compression or even cheaper DXT1/BC1.**

[ARGB32]: https://docs.unity3d.com/6000.3/Documentation/ScriptReference/ImageConversion.LoadImage.html

## Tradeoffs
Advantages:
* Textures in DDS files load significantly faster than PNG (on average 8.5x faster!) as they are ready to be loaded to RAM and then VRAM straight way.
* BC7 compressed textures take 4x less VRAM.
* BC7 textures do not have observable impact on performance (reportedly they may have slightly higher fetch time, but that is offset with lower bandwidth requirement so they often end up working better)
* Existing PNGs can be converted to DDS with BC7 by users - they do not require involvement of riggers or payed applications.
* (Likely) Already supported by every PC that can run VTube Studio due to compression requiring DX11 (or Vulkan when using DXVK on Linux).

Disadvantages:
* BC7 textures need to be prepared ahead of time due to how the compression scheme works.
* They are not mobile friendly, so a fallback for mobile is needed when using VNet.
* Certain models with low pixel density on textures may see some texture degradation (but these are not going to see massive benefits from using BC7 in the first place).
* They take more space on SSD/HDD.
* Support of BC7 on both sides of VNet will require some additional code (likely Gzip or sending original PNG file).


## Quality comparison 
To prove my point on why BC7 should be implement, I think it's best to start with visuals. It can't be just text all the time. The way I set every one of those captures is a maximum available zoom in VTube Studio - not necessary something a user will see, especially with models having variable pixel density, but it's there to prove a point.

My own model (shoutouts to Tsuno 💚):
{% include before-after.html
  before="/images/articles/vtubestudio_bc7/sui_rgba32.png"
  after="/images/articles/vtubestudio_bc7/sui_bc7.png"
  label-before="RGBA32 (2.66GB)"
  label-after="BC7 (0.67GB)"
  alt-before="RGBA32"
  alt-after="BC7"
  unique-class="slider-1"
%}
2.66GB vs 0.67GB

Akari:
{% include before-after.html
  before="/images/articles/vtubestudio_bc7/akari_rgba32.png"
  after="/images/articles/vtubestudio_bc7/akari_bc7.png"
  label-before="RGBA32 (85.33MB)"
  label-after="BC7 (21.33MB)"
  alt-before="RGBA32"
  alt-after="BC7"
  unique-class="slider-2"
%}
85.33 MB vs 21.33MB

If you look real closely, especially with edges, you can notice a small difference, but tradeoff with BC7 is generally worth it.

Now to show you this isn't the same image, I have manually increased the size 4x.
{% include before-after.html
  before="/images/articles/vtubestudio_bc7/sui_rgba32_4x.png"
  after="/images/articles/vtubestudio_bc7/sui_bc7_4x.png"
  label-before="RGBA32 (2.66GB)"
  label-after="BC7 (0.67GB)"
  alt-before="RGBA32"
  alt-after="BC7"
  unique-class="slider-3"
%}
2.66GB vs 0.67GB

As you can see with a model of this pixel density, the trade off is very much not noticeable, but we are saving almost 2 GB of VRAM!

## DDS vs PNG benchmarks
My way of testing it is to do at least 5 different loads of textures of different resolution and calculate an average. I have done it inside of VTube Studio by hooking into the load function. These times are in seconds are rounded to 3 decimal places.

| Texture | PNG-ARGB32 | DDS-BC7 | Speed |
| ----------- | :----: | :----: | -----: |
| Sui-16k (1) | 2.176s | 0.213s | 10.21x |
| Sui-16k (2) | 2.729s | 0.327s | 8.34x  |
| Nepeta-8k   | 0.419s | 0.052s | 8.05x  |
| Akari-4k    | 0.128s | 0.015s | 8.53x  | 

This is off topic a bit, but this also got me thinking on how PNG compression (deflate) affects load times in VTube Studio. Results are below:

| PNG Texture | Compression 9 | Compression 6 | Compression 1 | None  |
| ----------- | :----: | :----: | :----: | :----: |
| Akari-4k    | 0.244s | 0.252s | 0.210s | 0.175s |

Now the question that I have in my head - what was the original source PNG compression and what made it load so fast. I guess it's also worth noting that PNG compression is lossless, but even best case (original texture) loads 8.53x slower with Unity's LoadImage than from DDS.

## Implementation issues
My suggestion for implementation would be to implement support for BC1/BC3 and BC7. This isn't hard as the hardest part is reading DDS header and then telling Unity which Texture2D object to create based on it. However as far as I understand there are 2 things that need to be considered:
* Mobile support and compatibility
* VNet support

### Mobile compatibility
Here with my limited time I can only offer suggestions.
1. **Option 1** would be to do it in a similar way to my VTS-Memory-Compression hook, where VTube Studio checks whatever PNG file exists and once it is found, check whatever DDS file is present - if the latter is found, the latter is loaded, if not, PNG (or technically originally defined texture in json file) is loaded. With this approach, when sending a model to mobile application we can be sure that the file that the phone can handle is there (although that still doesn't prohibit sending a resolution that mobile phones can not handle - but this is an issue present currently as well). Another good thing about this approach is that we can be sure the user has access to original, lossless texture (you never can fully trust riggers) - BC1/BC3/BC7 are lossy after all.
2. **Option 2** and this is something to possibly explore, since I haven't been able to verify it - it's possible that despite BC1/BC3/BC7 not being supported on mobile, they will get automatically decompressed by Unity anyway using software, as [Unity documentation implies it][BC7Mobile]. But I personally think te first solution is safer.

[BC7Mobile]: https://docs.unity3d.com/6000.3/Documentation/ScriptReference/TextureFormat.BC7.html

### VNet
Since VNet as far as I know isn't available on Mobile, this leads to a safe assumption that any hardware running VTube Studio is capable nowadays of handling BC7 (and simpler) compression schemes.

However, since BCn compressed textures are generally bigger than PNGs, this posses an issue when sending them to external users. My suggestion for this would be a use of Gzip. To keep things performant, I would say that when joining VNet, user's textures should get compressed and stored in a temporary directory - this is due to how long a Gzip compression takes.

In my test (inside of Unity, so it might be faster in release) compression of 341MB DDS file took 3s, which is quite a significant amount of time. Of course, this an extreme case as the texture is 16k! Smaller textures were below 1s, thankfully!

On a positive note after compressing DDS with Gzip the resulting size was smaller than original PNG (but still bigger than PNG compressed manually with Compression level 9).


| PNG Texture | Source | Source with<br> Compression 9 | DDS | Gzip DDS |
| ----------- | :----: | :----: | :----: | :----: |
| Sui-16k (1) | 101.0MB | 41.3MB | 341MB  | 84.2 MB |
| Sui-16k (2) | 54.5MB | 22.7MB | 341MB  | 42.9 MB |
| Nepeta-8k   | 8.11MB | 5.13MB | 85.3MB | 5.99MB |
| Akari-4k    | 7.03MB | 5.24MB | 21.3MB | 5.06MB |

So worst case scenario, Gzipped texture is always smaller than all source textures I had. Bad news is that using PNG Compression on Level 9 results almost always in smaller files than Gzipped DDS, anyway.

## Conclusion
With how the things are nowadays, I believe DDS and BC7 offers significant advantages for VTubers who are limited to a single PC setup (which is vast majority of them) at almost no cost for the user (as long as the quality drop is not noticeable). Having used my own hook for VTube Studio I ***personally*** see it as a massive advantage for the users, which drastically reduces VRAM requirement of complex models, which then allows that VRAM space to be utilized by games instead, not to mention speeds up loadings of models!

Of course ultimately the decision on whatever to implement it remains to Denchi. But with hardware prices nowadays and how hard it is to get a good deal on a new graphics card that has 16 GB of VRAM, this could help many VTubers that are limited to 8 GB or 12 GB graphics cards play more modern games or play those games with higher amount of details.

## Sources and references
* [https://github.com/SuiMachine/VTS-Memory-Compression](https://github.com/SuiMachine/VTS-Memory-Compression)
* [https://www.reedbeta.com/blog/understanding-bcn-texture-compression-formats/](https://www.reedbeta.com/blog/understanding-bcn-texture-compression-formats/)
* [https://www.connburanicz.com/block-compression-bc1-bc7](https://www.connburanicz.com/block-compression-bc1-bc7)
* [https://www.gamedev.net/forums/topic/719663-are-there-any-hidden-costs-of-using-bc7-over-rgba32/](https://www.gamedev.net/forums/topic/719663-are-there-any-hidden-costs-of-using-bc7-over-rgba32/)
* [https://learn.microsoft.com/en-US/windows/win32/direct3ddds/dx-graphics-dds](https://learn.microsoft.com/en-US/windows/win32/direct3ddds/dx-graphics-dds)
