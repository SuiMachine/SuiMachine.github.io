---
layout: post
title:  "A bit of work on AVP2 Custom Launcher"
date:   2016-10-09 04:05:00 +0200
categories: developement
---
Since I created this pointless blog, I thought I may write something, so that it's not just showing off WebP being superior to Gif. In the past 3 days, I've spent plenty of time learning how to do code injection and reworking my AVP2 Custom Launcher to use that method.

The point was to finally fix the field of view in cutscenes (while my Custom Launcher could do that for gameplay, the cutscenes remained stretched from 4:3). I also reminded myself that I once "accidently" found a value that is used for scaling background image in menu and decided to revisit it as well.

I quickly figured out how to calculate a proper value for menu background depending on aspect ratio (I used common vertical to horizontal fov functions) and added it to Custom Launcher.  The far bigger issue was figuring out how to scale displayed FOV with injected code, but I managed to finally sort it out as well using a combination of ASM and C++.

Here are some screenshots:

![menu_02.png](/images/avp2customlauncher/menu_01.png)

<!--more-->

![menu_02.png](/images/avp2customlauncher/menu_02.png)



And here is finally fixed cutscene:

![cutscenefov.png](/images/avp2customlauncher/cutscenefov.png)

![cutscenefov2.png](/images/avp2customlauncher/cutscenefov2.png)
