---
layout: page
title: Lunacid - Ultra-widescreen fix
menu: none
permalink: projects/lunacid_ultrawidescreen/
---

This was a quick hack I have made, when I saw people complaining about the game not displaying UI correctly on ultra-widescreen monitors. All it essentially does is it switches the canvas scaling to "Expand" mode (see: [https://docs.unity3d.com/Packages/com.unity.ugui@1.0/manual/script-CanvasScaler.html](https://docs.unity3d.com/Packages/com.unity.ugui@1.0/manual/script-CanvasScaler.html)), so it the UI is no longer ending up outside of the screen space.

![images/lunacid_ultrawidescreen.jpg](../images/lunacid_ultrawidescreen.jpg)

Source code and release available at: [https://github.com/SuiMachine/Lunacid---Ultrawidescreen-fix](https://github.com/SuiMachine/Lunacid---Ultrawidescreen-fix)