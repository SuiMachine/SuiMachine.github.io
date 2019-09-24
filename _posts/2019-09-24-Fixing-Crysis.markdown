---
layout: post
title: "Fixing Crysis"
date:   2019-09-24 09:00:00 +0200
categories: ["reverse"]
---
A friend of mine asked me recently to make a load remover for him, as the one in LiveSplit repository wasn't quite working for him. This however turned out to be a more of a hustle then expected. Not only because finding a pointer path to anything that'd tell whatever loading is happening took many hours of looking up code in IDA, but also because - as it turned out - Crysis didn't age very well, when it comes to compatibility with modern systems... especially AMD based.

The following post is more or less explanation of what was wrong and how I hacked / "fixed" it. Both of my fixes can be found on [PCGamingWiki](https://pcgamingwiki.com/wiki/Crysis).

### 32bit and 3D Now!
Let's start off with a fix that I am more confident about, which is a fix for 32bit version of the game. As always, started with plugging IDA to main executable (cracked, since the Steam version of the game was screwing debugging process) and clicking run, seeing when it throws the error. Bam - ``CryRenderD3D10.dll:3800D501``.
<!--more-->

So I loaded CryRenderD3D10.dll to have initial analysis done of it by IDA, go to that address and here is what we see.
![2019-09-24 10_19_23-IDA-CryRenderD3D10.png](/images/articles/2019-09-24 10_19_23-IDA-CryRenderD3D10.png)

``pswapd``? ``pfmul``? ``pfnacc``? None of this sound familiar. But despite it, arguments in registers are fine, so why would that crash the game. A quick google.... I meam duck, duck, go-ing?

![2019-09-24 10_26_16-ipswapd.png](/images/articles/2019-09-24 10_26_16-ipswapd.png)

``AMD``, ``MMX``, ``3DNow!``? CPU: AMD K6-2, deprecated. Oh. Suddenly everything starts to make sense. A quick look at the features of Ryzen 5 in CPU-Z. No ``3DNow`` to be seen. So what we're dealing with, is game trying to execute instructions not supported by a CPU. But how do we proceed about fixing it?

Well, let's x-ref all instances in which D3D10 tries to use that function. There are only 4 - set breakpoints, see what happens.

![2019-09-24 10_19_23-IDA-CryRenderD3D10_2.png](/images/articles/2019-09-24 10_19_23-IDA-CryRenderD3D10_2.png)

Interesting. We have instruction loading a byte to ``cl`` from ``382D5110``, then doing bit compare against 4 and then doing a jump based on that.

Quick test - set breakpoint at ``mov cl, byte ptr dword_382D5110`` and launch the game.

Attach CheatEngine. Modify value in ``382D5110`` to be lower by for 4 (negate 3rd bit), let it continue further and... boom game works.

OK, so what's left to do is figure out where ``382D5110`` is actually set and modify it. Breakpoint at value, launch he game.

```*(_DWORD *)dword_382D5110 = (*(int (**)(void))(*(_DWORD *)dword_382C7A04 + 44))();```

Where is instruction from (382C7A04 + 44) located? CrySystem.dll at 365535F0:

![2019-09-24 10_19_23-IDA-CryRenderD3D10_3.png](/images/articles/2019-09-24 10_19_23-IDA-CryRenderD3D10_3.png)

And would you look at that - our bit which causes entire problem. All is left to do is nop few instructions so it's not set and problem mostly solved. Sure, it'd be interesting to see how ``v1 +24`` is set, but we can pretty much be entirely sure at this point, that it's just Crytek that assumed these instructions will be forever present in AMD CPUs... and sadly, they are not. Fortunately, alternative ones still work, so we can just negate that bit and use them instead. Patch done.

### 64bit and MXCSR
At this point in writing, I am really getting sick of it, but oh well - I started it, I might as well try and end it.

Just like before, start by setting IDA to debug Crysis, except a 64bit version. Launch. Error.
![2019-09-24 11_34_44-Warning.png](/images/articles/2019-09-24 11_34_44-Warning.png)

And here is how the section looks like:

![2019-09-24 11_36_21-IDA - CrySystem.png](/images/articles/2019-09-24 11_36_21-IDA - CrySystem.png)

Since the error happens at ``cvtsd2ss``, it'd be nice to check what it does.
> Convert Scalar Double-Precision Floating-Point Value to Scalar Single-Precision Floating-Point Value

So basically we are converting from double to float using xmm registers. But why the crash?

> When the conversion is inexact, the value returned is rounded according to the rounding control bits in the MXCSR register.

Which is interesting, cause by default it should not be happening, but it seems like some function in the thread managed to set MXCSR to non default behaviour and since it literally stays as long as something doesn't change it again, it may be pretty much anything in both game's libraries and even system libraries itself.

How do we work around it, then?

Firstly, what I did, is I added additional section in dll using Stud_PE.

Then I set a new value (just 4 bytes) in new section, which was equal to: ``0x1F80``

After that, I added modified instructions taken from original segment that was crashing the game, but expended it with additional instruction ``ldmxcsr``, right before ``cvtsd2ss``, which was loading our allocated ``0x1F80``.

Finally I set up a jump instruction to a new segment from original code, nop-ed not needed instructions and then I set up a jump back from a new code back to original.

![2019-09-24 11_36_21-IDA - CrySystem2.png](/images/articles/2019-09-24 11_57_36-IDA - CrySystem2.png)

Saved everything and we're done.

Game works and I can finally go to sleep.
