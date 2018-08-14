---
layout: post
title:  "Installing Kolor GoPro VR Plugins on Adobe Premiere/After Effects CS5.5"
date:   2018-08-14 09:21:00 +0200
categories: Guides
---
I knew about Kolor GoPro VR plugins for a while. Having to 'flatten' one of the 360 videos recently, however, I was deeply disappointed to find installer not supporting CS5.5. After few minutes of experimentation however, it turned out it is possible to manually add them to plugins in CS5.5. Here is how.

1. Download [GoPro VR Plugins](http://www.kolor.com/gopro-vr-plugins/).
2. Download and install [7-zip](https://www.7-zip.org/).
3. Right click on downloaded __GoProVRPlugins_x64_x_x-x-x.exe__, select __7-Zip__ and __Extract to \"GoProVRPlugins_x64_x_x-x-x.exe\\\"__
![2018-08-14 09_26_17-GoProExtract.png](/images/articles/2018-08-14 09_26_17-GoProExtract.png)
<!--more-->
4. You'll see a folder with directories that look like this:
![2018-08-14 09_32_37-GoProVRPlugins_x64_102_2017-11-03](/images/articles/2018-08-14 09_32_37-GoProVRPlugins_x64_102_2017-11-03.png)
5. Check each of the directories.
6. For __After Effects__ plugins you look for files with extension __\*.aex__. Copy them to: ```Adobe After Effects CS5.5\Support Files\Plug-ins\Effects\```
7. For __Premiere__ plugins you look for files with extension __\*.prm__. Copy them to: ```Adobe Premiere Pro CS5.5\Plug-ins\Common\```
8. Now open the folder with a number one lower than the one in which you found your plugin files (e.g. for ```$_10_``` open ```$_9_```)
9. Copy dll libraries from that folder to ```After Effects\Support Files``` directory or ```Premiere Pro``` directory, depending on which program plugin files you've just copied.