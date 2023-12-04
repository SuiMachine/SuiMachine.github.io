---
layout: page
title: DwarfHeim - Project Rebellion
menu: none
permalink: projects/dwarfheim-project_rebellion/
---

This one I started out of spite. On October 31, 2023, the publisher of DwarfHeim announced that the API server behind the game will be shutting down a month later.
I do not appriciate it when publishers or developers enforce heavy custom API dependancy (instead of utilizing long running solutions like Steamworks / GOG Galaxy) without thinking about a backup plan in case the game doesn't sell.
But that wouldn't be enough to get me ralied up - what got my pissed off was the way it was announced.

![images/dwarfheim_project_rebellion_news.png](../images/dwarfheim_project_rebellion_news.png)

There is just something very ignorant about claiming that no source code is available for a game that was literally built with Mono - publishers who make such statements either have no idea what they are talking about and how Unity games they publish work or they are fully aware and just can't be bothered to hire a decent programmer for like 2 weeks of work to fully strip down the API requirement from the game - in either case, publishers like this should not be supported!


And so... I started working on figuring out which API responses have to be faked directly within the game to make a singleplayer running. 10 hours later initial release of Project Rebellion was released allowing players to play the tutorial and at least one of the maps in the game that they have paid for that the publisher couldn't be bothered to even support for more at least 2 years.

<center><iframe width="940" height="528" src="https://www.youtube.com/embed/6vGEfCLwU0M" frameborder="0" allowfullscreen></iframe></center>

Source code and release available at: [https://github.com/SuiMachine/DwarfHeim---Project-Rebellion](https://github.com/SuiMachine/DwarfHeim---Project-Rebellion)
