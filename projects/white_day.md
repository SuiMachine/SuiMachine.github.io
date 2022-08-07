---
layout: page
title: White Day&#58; A Labyrinth Named School - Sui's Hack
menu: none
permalink: projects/white_day/
---

This hack started developement early in 2022. The initial goal was to just add options for 120, 144 and 240 FPS cap to game settings, but it quickly turned out that the game's port requires more attantion.

Thankfully, due to the game being mono compiled, I was able to fairly accurately reconstruct its movement spliting it between FixedUpdate (for movement alone) and Update (for rotations) and then interpolate FixedUpdates in order to work around the issues that make the player unable to move with high-framerate updates.

Furthermore, I reworked camera rotations in order to allow for Vertical Axis Inversion to work with mouse input as it only worked with gamepad up until now.

Here are 2 examples of issues that appear with high-FPS - first off, crouching being broken around 120 fps:
![images/white_day_120_crouching.webp](../images/white_day_120_crouching.webp)

And then normal walking also being broken around 240fps:
![images/white_day_240_walking.webp](../images/white_day_240_walking.webp)

**Note**: To see above images a web browser capable of displaying WebP images is required.

Source code and release available at: [https://github.com/SuiMachine/White-Day---Sui-s-Hack](https://github.com/SuiMachine/White-Day---Sui-s-Hack)