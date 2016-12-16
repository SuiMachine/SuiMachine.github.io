---
layout: post
title:  "Colobot - minimum human input - Earth Codes"
date:   2016-12-05 08:41:00 +0200
categories: colobot
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
	while(!researched(ResearchTracked))
	{
		wait(1);
	}
	file handle();
	handle.open("getCubeToShip.txt", "r");
	
	string o_str = "";
	while(not handle.eof())
	{
		o_str += handle.readln() + "\n";
	}
	
	item.factory(TrackedGrabber, o_str);
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
	object item = radar(BlackBox);
	goto(item.position);
	grab();
	item = radar(SpaceShip);
	goto(item.position);
	item.takeoff();
}
{% endhighlight %}

