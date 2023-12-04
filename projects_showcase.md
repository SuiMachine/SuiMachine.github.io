---
layout: page
title: Projects
menu: main
permalink: /projects/
---
<link rel="stylesheet" href="{{ base }}/css/projects.css">

<div class="gametable-container">
{% include project_button.html name="Art of Rally - VR hack" description="BepInEx 6, HarmonyX and OpenVR based hack that I used as a learning project for converting a flatscreen game to VR. The hack doesn't use any VR controls and requires the player to be sitting and using keyboard and mouse, gamepad or a steering wheel to play. VR controllers are not supported." url="art_of_rally" color="#FE7950A0" %}
{% include project_button.html name="Deadly Premonition 2 - Sui's Hack" description="MelonLoader and HarmonyX based hack designed to help mitagate the game's terrible port, exposing its settings, improving graphics and fixing some of its bugs. It also adds a simplistic, but playable keyboard and mouse support, controller rumble and many other options." url="deadly_premonition_2" color="#C03027A0" %}
{% include project_button.html name="DwarfHeim - Project Rebellion" description="BepInEx 6 and HarmonyX based hack that restores the minimum functionality to the game after the publisher carelessly turned off its API server barely a year after the game's release." url="dwarfheim-project_rebellion" color="#1D2150A0" %}
{% include project_button.html name="White Day: A Labyrinth Named School - Sui's Hack" description="BepInEx 5 and HarmonyX based hack that introduces settings for anisotropic filtering, supports high-FPS framerates and mouse inversion" url="white_day" color="#212B2BA0" %}
</div>

<h2>Archival:</h2>
<div class="gametable-container">
{% include project_button.html name="Lunacid - Ultra-widescreen fix" description="A simplistic BepInEx + HarmonyX based hack that used to modify the game's canvas behaviour to make UI usable on ultrawidescreen monitors made in an hour or so. This hack is now obsolete thanks to the developer implementing a proper ultrawidescreen support to the game." url="lunacid_ultrawidescreen" color="#333948A0" %}
</div>
