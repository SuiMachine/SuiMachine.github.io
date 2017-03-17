---
layout: post
title:  "Colobot - minimum human input - Earth Codes"
date:   2016-12-05 08:41:00 +0200
categories: Colobot
---


![menu_02.png](/images/colobot/01_earth.png){: .alignLeft} This post is tied to my small YouTube series, which involves beating Colobot using minimum human input. To see the CBot codes, together with some notes click read more.
<br><br>
<!--more-->

> Mission 2 - Building
{% highlight cs %}
extern void object::DoIt()
{
	object item = radar(Titanium);
	goto(item.position);
	grab();
	goto(flatspace(this.position, 10, 0, 100, 10));
	drop();
	build(ResearchCenter);
	item = radar(Titanium);
	goto(item.position);
	grab();
	goto(flatspace(this.position, 10, 0, 100, 10));
	drop();
	build(BotFactory);
}
{% endhighlight %}

> Mission 3 - Departure

**Wheeled Grabber**
{% highlight cs %}
extern void object::DoIt()
{
	object item = radar(PowerCell);
	goto(item.position);
	grab();
	item = radar(ResearchCenter);
	goto(item.position);
	drop();
	item.research(ResearchTracked);
	item = radar(Titanium);
	goto(item.position);
	grab();
	item = radar(BotFactory);
	goto(item.position);
	drop();
	move(-5);
	
	//Wait till the Tracks for Bots are researched
	while(!researched(ResearchTracked))
	{
		wait(1);
	}
	
	//Prapre reading the code from a file
	file handle();
	handle.open("getCubeToShip.txt", "r");
	
	//Read the file till the end
	string o_str = "";
	while(not handle.eof())
	{
		o_str += handle.readln() + "\n";
	}
	
	//Tell the factory to construct TrackedGrabber and send it the code we've read
	item.factory(TrackedGrabber, o_str);
	
	//Get the PowerCell and place it in TrackedGrabber when it's built
	item = radar(PowerCell);
	goto(item.position);
	grab();
	while((item = radar(TrackedGrabber)) == null)
	{
		wait(1);
	}
	goto(item.position);
	drop();
	item = radar(SpaceShip);
	goto(item.position);
}
{% endhighlight %}

**getCubeToShip.txt** (located in: %USERPROFILE%\colobot\files\)
{% highlight cs %}
extern void object::GetBlackBox()
{
	//Basic goto function - nothing to explain. Get Blackbox to a ship
	object item = radar(BlackBox);
	goto(item.position);
	grab();
	item = radar(SpaceShip);
	goto(item.position);
	item.takeoff();
}
{% endhighlight %}

