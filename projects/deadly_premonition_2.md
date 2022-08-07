---
layout: page
title: Deadly Premonition 2&#58; A Blessing in Disguise - Sui's Hack
menu: none
permalink: projects/deadly_premonition_2/
---

The work on this hack started essentially 2 days after the release of PC port. Going by the developer's and publisher's prior games, I knew, that if a patch is going to be made, it won't address many of the games issues and so I started my work early hoping to elevate this game's bare-bones port to entirely new level. 

One of the first things I addressed was the game's counter intuitive controls, as the developer opted out to use "B" on Xbox controllers as "accept" button and "A" on Xbox controllers as "Cancel" - the exact opposite of the current standard. My solution to this was to export the game's textures, build new asset bundles with keys flipped and then write the MelonLoader plugin that will load them and replace them at runtime. This proved to be more difficult than expected, due to the old version of NGUI this game uses, but I was ultimately successful at doing so. All it was left is to flip these keys using SteamInput, which the game uses either way.

After that, I proceeded to adress the game's issues with 50 times a second character updates, which were noticable on PCs. Following video showcases some of this problem and how an interpolatation helped me adress it.
<center><iframe width="940" height="528" src="https://www.youtube.com/embed/mZc9j-SmA0s" frameborder="0" allowfullscreen></iframe></center>
It doesn't solve all issues, as the pseudo-physics clothes simulation is done per frame - but that issue affects both interpolated and non interpolated character movement.

Another issue that I have managed to adress was the an issue affecting its wires and rendering bounds, as shown as video below:
<center><iframe width="940" height="528" src="https://www.youtube.com/embed/VtjgaIj6LB8" frameborder="0" allowfullscreen></iframe></center>

After that I have proceeded to work on improving the game's graphics and eventually worked on adding a basic mouse and keyboard support.

For my write up about the game's issues and how I proceeded to solve some of them, see: [Improving Deadly Premonition 2 using HarmonyX and MelonLoader](../../../2022/07/07/Improving_Deadly_Premonition_2_using_HarmonyX_and_MelonLoader.html) blog entry.

Source code and release available at: [https://github.com/SuiMachine/Deadly-Premonition-2---Sui-s-hack](https://github.com/SuiMachine/Deadly-Premonition-2---Sui-s-hack)