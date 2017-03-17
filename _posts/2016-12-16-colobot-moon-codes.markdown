---
layout: post
title:  "Colobot - minimum human input - Moon Codes"
date:   2016-12-06 01:44:00 +0200
categories: Colobot
---


![menu_02.png](/images/colobot/02_moon.png){: .alignLeft} This post is tied to my small YouTube series, which involves beating Colobot using minimum human input. To see the CBot codes, together with some notes click read more.
<br><br>
<!--more-->

> Mission 1 - Titanium ore

**Wheeled Grabber**
{% highlight cs %}
extern void object::DoIt()
{
	object item = radar(Titanium);
	object spaceship = radar(SpaceShip);
	goto(item.position);
	grab();
	goto(flatspace(spaceship.position, 10));
	drop();
	build(ResearchCenter);
	item = radar(PowerCell);
	goto(item.position);
	grab();
	while((item = radar(ResearchCenter)) == null)
	{
		wait(1);
	}
	goto(item.position);
	drop();
	item.research(ResearchWinged);
	
	item = radar(Titanium);
	goto(item.position);
	grab();
	goto(flatspace(spaceship.position, 10, 10, 50, 10));
	drop();
	build(BotFactory);
	
	item = radar(Titanium);
	goto(item.position);
	grab();
	item = radar(BotFactory);
	goto(item.position);
	drop();
	move(-5);
	file reader();
	reader.open("get_ore_solo.txt", "r");
	string program_read = "";
	while(not reader.eof())
	{
		program_read += reader.readln() + "\n";
	}
	item.factory(WingedGrabber, program_read);
	item = radar(PowerCell);
	goto(item.position);
	grab();
	item = radar(BotFactory);
	point tmp = item.position;
	tmp.x = tmp.x -5;
	tmp.y = tmp.y -5;
	goto(tmp);
	while((item = radar(WingedGrabber)) == null)
	{
		wait(1);
	}
	goto(item.position);
	drop();
	item = radar(SpaceShip);
	goto(item.position);
}
{% endhighlight %}

**get_ore_solo.txt** (located in: %USERPROFILE%\colobot\files\)
{% highlight cs %}
extern void object::DoIt()
{
	while(this.energyCell == null)
	{
		wait(1);
	}
	
	wait(5);
	move(-5);
	int i = 0;
	object item = radar(TitaniumOre, 0, 360, 20);
	object spaceship = radar(SpaceShip);
	for(i=0; i<4; i++)
	{
		item = radar(TitaniumOre, 0, 360, 20);
		gotoSafe(item.position);
		grab();
		item = radar(SpaceShip);
		goto(item.position);
		safedrop();
	}
	item.takeoff();
}

extern void safedrop()
{
	//We surpress the error
	errmode(0);
	try
	{
		while(drop() != 0)
		{
			turn(45);
		}
	}
	errmode(1);
}

extern void object::gotoSafe(point pos)
{
	//We surpress the error
	errmode(0);
	try
	{
		while(distance2d(this.position, pos) > 15)
		{
			turn(45);
			goto(pos);
		}
	}
	errmode(1);
}
{% endhighlight %}

> Mission 2 - Flying Drill #1 & Mission 3 - Flying Drill #2

{% highlight cs %}
extern void object::FlyAtTargets()
{
	while(radar(Target2)!=null)
	{
		object target = radar(Target2);
		
		if(this.temperature < 0.8)
		{
			jet(Throttle(this.position.z, this.velocity.z, target.position.z-1));
			PointAt(direction(target.position));
		}
		else
		{
			motor(0,0);
			jet(-0.4);
			while(temperature!=0)
			{
				wait(0.1);
			}
		}
		wait(0.1);
	}
	
	object spaceship = radar(SpaceShip);
	goto(spaceship.position);
}

extern float Throttle(float objSourceZ, float objSourceVelZ, float objTargetZ)
{
	if(objSourceZ +5 < objTargetZ)
	{
		return 1;
	}
	else if(objSourceZ -5 >objTargetZ)
	{
		return -1;
	}
	else
	{
		return (objTargetZ - objSourceZ-(objSourceVelZ/2))/5;
	}
}

extern void PointAt(float dir)
{
	if(dir < -10)
	{
		motor(1,0);
	}
	else if(dir > 10 )
	{
		motor(0,1);
	}
	else
	{
		if(dir < 0)
		{
			float val = (dir/10)+1;
			motor(1, val);
		}
		else
		{
			float val = -(dir/10)+1;
			motor(val,1);
		}
	}
}
{% endhighlight %}

> Mission 4 - Black Box

**WheeledGrabber**
{% highlight cs %}
extern void object::DoIt()
{
	//Initialize exchange class
	ex list();
	
	object item = radar(Titanium);
	object secondBot = radar(WingedGrabber);
	
	goto(item.position);
	grab();
	goto(buildSpace());
	drop();
	build(Converter);
	
	item = radar(TitaniumOre);
	goto(item.position);
	grab();
	item = radar(Converter);
	goto(item.position);
	drop();
	move(-4);
	turn(90);
	move(-5);
	wait(14);
	
	//This should be done more dynamically
	list.whGrabberNeedsEnergy = true;
	
	//Wait till you are refueled
	while(list.whGrabberNeedsEnergy)
	{
		wait(1);
	}
	//Get titanium ore, convert it, build radar station, go to a ship, set the flag that you're ready to leave
	item = radar(TitaniumOre);
	goto(item.position);
	grab();
	item = radar(Converter);
	goto(item.position);
	drop();
	move(-4);
	wait(14.8);
	move(4);
	grab();
	goto(buildSpace());
	drop();
	build(RadarStation);
	item = radar(SpaceShip);
	goto(item.position);
	list.whGrabberReadyToLeave = true;
}

extern point buildSpace()
{
	object item = radar(SpaceShip);
	flatspace(item.position, 5, 10, 100, 5);
}
{% endhighlight %}

**WingedGrabber**
{% highlight cs %}
extern void object::DoItW()
{
	object spaceShip = radar(SpaceShip);
	
	//Initialize exchange list that can be accessed by other bots
	ex list();
	
	//Find the other bot
	object secondBot = radar(WheeledGrabber);
	
	//Wait till the other bot doesn't send the "whGrabberNeedsEnergy" to true in our ex class
	while(!list.whGrabberNeedsEnergy)
	{
		wait(1);
	}
	
	//Find Titanium Cube, get it, build Power station
	object item = radar(Titanium);
	move(-5);
	goto(item.position);
	grab();
	goto(buildSpace());
	drop();
	build(PowerStation);
	
	//We store it, because it's useful
	object pwStation = radar(PowerStation);
	goto(pwStation.position);
	
	//Wait till we fully charged
	while(this.energyCell.energyLevel != 1)
	{
		wait(1);
	}
	
	//Go to 2nd bot, grab its powercell, to to PowerStation, charge it, return it to 2nd bot, set its flag to false, so it continues working
	goto(secondBot.position);
	grab();
	goto(pwStation.position);
	while(this.load.energyLevel != 1)
	{
		wait(1);
	}
	goto(secondBot.position);
	drop();
	list.whGrabberNeedsEnergy = false;
	
	//Go to blackbox, bring it to the ship
	object blackBox = radar(BlackBox);
	goto(blackBox.position);
	grab();
	basicGoto(getSpaceOnAShip(), 14);
	
	while(!list.whGrabberReadyToLeave)
	{
		wait(1);
	}
	spaceShip.takeoff();
}

extern void object::basicGoto(point whereTo, float altitude)
{
	//This is a basic function which makes the bot go in a straight line to specified point, since "goto()" does some checks and usually throws an error
	while(distance2d(this.position, whereTo) > 1)
	{
		if(this.temperature > 0.8)
		{
			jet(-0.3);
			motor(0,0);
			
			while(this.temperature != 0)
			{
				wait(1);
			}
		}
		
		turn(direction(whereTo));
		
		//handle Height
		if(this.position.z - (topo(this.position) + altitude) > 5)
		{
			jet(-1);
		}
		else if (this.position.z - (topo(this.position) + altitude) < -5)
		{
			jet(1);
		}
		else
		{
			jet((topo(this.position) + altitude)/this.position.z - 1);
		}
		
		//handle Position
		if(distance2d(this.position, whereTo) > 25)
		{
			motor(1,1);
		}
		else
		{
			float val = distance2d(this.position, whereTo) / 25;
			motor(val,val);
		}
		wait(0.2);
	}
	motor(0,0);
	jet(-0.3);
	wait(3);
}

extern point buildSpace()
{
	//Find a closes free flat space from a ship
	object item = radar(SpaceShip);
	flatspace(item.position, 5, 10, 100, 5);
}

public class ex
{
	//This is an exchange class, which is publically available for all bots. Issue is, it will crash the game if it hasn't be initialized in current session (so if you load and program of a bot tries to access it - the game will crash)
	public static bool whGrabberNeedsEnergy = false;
	public static bool whGrabberReadyToLeave = false;
}

point getSpaceOnAShip()
{
	object sp = radar(SpaceShip);
	
	//This basically creates a list of objects that can be on the ship
	int obj[];
	int i=0;
	obj[i++] = TitaniumOre;
	obj[i++] = UraniumOre;
	obj[i++] = Titanium;
	obj[i++] = PowerCell;
	obj[i++] = NuclearCell;
	obj[i++] = OrgaMatter;
	obj[i++] = TNT;
	obj[i++] = BlackBox;
	obj[i++] = KeyA;
	obj[i++] = KeyB;
	obj[i++] = KeyC;
	obj[i++] = KeyD;
	obj[i++] = WheeledGrabber;
	obj[i++] = TrackedGrabber;
	obj[i++] = WingedGrabber;
	obj[i++] = LeggedGrabber;
	obj[i++] = WheeledShooter;
	obj[i++] = TrackedShooter;
	obj[i++] = WingedShooter;
	obj[i++] = LeggedShooter;
	obj[i++] = WheeledSniffer;
	obj[i++] = TrackedSniffer;
	obj[i++] = WingedSniffer;
	obj[i++] = LeggedSniffer;
	obj[i++] = WheeledOrgaShooter;
	obj[i++] = TrackedOrgaShooter;
	obj[i++] = WingedOrgaShooter;
	obj[i++] = LeggedOrgaShooter;
	obj[i++] = Subber;
	obj[i++] = Recycler;
	obj[i++] = Shielder;
	obj[i++] = Thumper;
	obj[i++] = PhazerShooter;
	obj[i++] = Me;
	
	//This section specifies free spots on the ship (you know, the squares)
	point[] spaces;
	spaces[0].x = sp.position.x -3.5;
	spaces[0].y = sp.position.y -3.5;
	
	spaces[1].x = sp.position.x -3.5;
	spaces[1].y = sp.position.y;
	
	spaces[2].x = sp.position.x -3.5;
	spaces[2].y = sp.position.y + 3.5;
	
	spaces[3].x = sp.position.x;
	spaces[3].y = sp.position.y -3.5;
	
	spaces[4].x = sp.position.x;
	spaces[4].y = sp.position.y +3.5;
	
	spaces[5].x = sp.position.x + 3.5;
	spaces[5].y = sp.position.y - 3.5;
	
	spaces[6].x = sp.position.x + 3.5;
	spaces[6].y = sp.position.y;
	
	spaces[7].x = sp.position.x + 3.5;
	spaces[7].y = sp.position.y + 3.5;
	
	//We check if any of these is free by checking if any of listed objects is located in 2m radius
	for(i=0; i<8; i++)
	{
		bool taken = false;
		object item = search(obj, spaces[i]);
		if(distance2d(item.position, spaces[i]) > 2)
		    taken = false;
		else
		    taken = true;
		
		if(!taken)
		    return spaces[i];
	}
	
	//If all of them are taken, we return a point -1337, -1337
	point empty (-1337, -1337);
	return empty;
}
{% endhighlight %}