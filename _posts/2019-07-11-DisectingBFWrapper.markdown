---
layout: post
title: "Dissecting junky Star Wars: Battlefront patch"
date:   2019-07-11 13:00:00 +0200
categories: ["reverse"]
---
Today I got news that Star Wars: Battlefront (original one, not the EA's cashcow) got updated via both GOG Galaxy and Steam, but it no longer functions as well as it should. Having created my own version of widescreen fix for it, I thought I'd try and dissect the new fix, Disney/GOG came up with. First off, changelog:

> Hotfix Update 1    
> 10 JUL @ 3:16AM by DISNEY    
> Minor bug fixes and optimized performance, the most important fixes are:    
> * Framerate capped at 60FPS, which will address the Crouch & Vehicle Flipping issues in the game that were noticed by players with high framerate monitors.
> * Fixed a number of the on/off video settings in setup not being able to be toggled.
> * Fixed incorrect aspect ratio in the 3d game world for widescreen monitors.
<!--more-->

So we've got a fix for players not being able to toggle multiple options in video settings menu. That's good. How it's done, not really sure (GOG version that I have access to via PCGamingWiki, still doesn't seem to have standalone patch). The remaining 2 ones, I have some clue, as in - they are done via sort of DLL wrapper, which is **winmm.dll** built specifically for this game. How this works is basically:
1. You start the game.
1. Game starts loading dynamic libraries - first in folder with game's executable and **if none is found**, then it starts looking for them in C:\Windows\System32 or C:\Windows\SystemWOW64 (the 2nd one is for 32bit libraries, not the first one). I think that if none is found, then it also checks folders specified under system's environment variables, but that last part I am not so sure of, so don't quote me on that.
1. Either way, it loads winmm.dll, which normally is Windows library and located under System32 directory. I write normally, because here it actually loads a wrapper DLL made for the game, which is located in a folder with the game.
1. Once wrapper DLL gets loaded, it performs its own code, but since game refers to functions that should be handled by legit DLL from System32 / SystemWOW64, it  also loads DLL from System32 / SystemWOW64 and then sets all of the calls for its own extern factions to point to the legit DLL.

With such a trick, wrapper DLL has native access to all memory of the game and can pretty much perform any code needed within the process itself (including patching OP codes in RAM), whilst also providing all of the functions of original DLL. Many of third party game fixes use it, as it's more convenient for end-user just to put DLL and not having to worry about running some program manually each time a game starts! It's pretty much the way to go, when you have to patch some function and don't have original source code to rebuild the game's exe.

### Problems with widescreen fix

And so we end up with current aspect ratio fix, which really isn't all that different than the one **I** wrote based on **ivanproff LUA's one**. The following is Pseudo-code generated (and then cleaned by me) from a function of 1st thread created in the "official" wrapper in today's patch.

{% highlight cpp %}
void WidescreenFix()
{ 
  char processWindowText[256];
  int readBuffer = 0;
  int writeBuffer = 0;
  
  while ( true )
  {
    do
    {
      Sleep(3000);
      HWND processWindowHandle = GetForegroundWindow();
      GetWindowTextA(processWindowHandle, processWindowText, 256);
    }
    while ( !strstr(processWindowText, "Star Wars Battlefront") );
	
    GetForegroundWindow();
    int processID = GetCurrentProcessId();
    HANDLE openHandleForProcess = OpenProcess(0x1FFFFFu, 0, processID);
    ReadProcessMemory(openHandleForProcess, (LPCVOID)0x728F70, &readBuffer, 0x4, 0);
    ReadProcessMemory(openHandleForProcess, (LPCVOID)0x728F74, &writeBuffer, 0x4, 0);
    if ( readBuffer > 0 )
    {
      writeBuffer = (readBuffer >> 1) + (readBuffer >> 2);
      WriteProcessMemory(openHandleForProcess, (LPVOID)0x728F74, &writeBuffer, 0x4, 0);
    }
  }
}
{% endhighlight %}

So let's pick apart some issues with it.

#### Getting process handle
First issue I have with it, instantly - is how it obtains the process handle. It's mostly done in 2 parts. First the function tries to obtain the WindowHandle for a window with a title / header "Star Wars Battlefront".

{% highlight cpp %}
do
{
  Sleep(3000);
  HWND processWindowHandle = GetForegroundWindow();
  GetWindowTextA(processWindowHandle, processWindowText, 256);
}
while ( !strstr(processWindowText, "Star Wars Battlefront") );
{% endhighlight %}

That's some really backward way of obtaining window handle. What happens here is, every 3 seconds, the function asks Windows to return it handle of a window currently in focus. Once it receives it, it asks Windows again, to provide it with a header of that window (in the form of char array). Finally, it compares the result it gets with "Star Wars Battlefront" to figure out whatever the window currently in foreground is from "Star Wars Battlefront" or not. There are multiple issues with it:
1. It requires 2 calls to Windows (minor)
1. It assumes that the window it gets is actually the game. That doesn't actually have to be the case, as having a folder with a name "Star Wars Battlefront" opened, will also return true! (Yes, I tested that)
1. Finally, there is also a slight chance that 2015's Battlefront game has the same window header, although knowing DICE - it probably doesn't, as they often list date, time and other nonsense in it.

{% highlight cpp %}
GetForegroundWindow();
processID = GetCurrentProcessId();
{% endhighlight %}

Having obtained window handle, the function will proceed to... get Foreground Window handle again, except this time it won't even assign it to variable. Just..... delet this!

Then it will ask Windows to get it process ID for a current process. Except, it isn't ID of process currently in foreground (thankfully, as the last thing we'd want is the game trying to write into something like Explorer.exe), it's ID of a process the function executes from. So, to some extend all of the lines before it were pointless, unless we treat them as safety checks to make sure game is loaded etc. but no... just no.

#### Hello? Yes, this is me. How are you... me?
After all that nonsense, the function finally managed to obtained process ID and we can proceed to correct aspect ratio. In case of this game, it's actually quite simple, as all we need to do is read Resolution With, divide it by 4, multiply by 3 and write it into Resolution Height. It's probably also why Ivan managed to figure it out, without his programming knowledge. Sometimes all you need is an idea and a bit of luck to make wonders.

So that's what this function d.............
{% highlight cpp %}
HANDLE openHandleForProcess = OpenProcess(0x1FFFFFu, 0, processID);
ReadProcessMemory(openHandleForProcess, (LPCVOID)0x728F70, &readBuffer, 0x4, 0);
ReadProcessMemory(openHandleForProcess, (LPCVOID)0x728F74, &writeBuffer, 0x4, 0);
if ( readBuffer > 0 )
{
  writeBuffer = (readBuffer >> 1) + (readBuffer >> 2);
  WriteProcessMemory(openHandleForProcess, (LPVOID)0x728F74, &writeBuffer, 0x4, 0);
}
{% endhighlight %}
No, of course not. Instead of reading directly from memory, what we get in here, is we have a function that asks Windows for Open Handle, which allows it to Read and Write to memory (even though, it really doesn't need it, cause it has that access!) and that utilize it, by asking WinAPI again, each time it needs to read or write something. Totally pointless!
1. It's way slower than doing reading / writing directly.
1. It's meant to be use for reading / writing into external processes!
1. Read of from address 0x728F74 is utterly pointless, as the result gets immediately overridden by ``(readBuffer >> 1) + (readBuffer >> 2);`` so wtf.

At least the content of writeBuffer is correct. It's a weird way of doing it, using bit offset operations, but it actually gives the same results. Huh.

#### Loops make everything worse
Finally there is a matter of loops. While a lot of this code is pretty trash, what totally makes it awful is the fact it repeats each 3 seconds... in its entirety! So that's:
1. Get process handle
1. Get window header text.
1. Compare char arrays.
1. Get foreground window and scrap it (of course!)
1. Get current process.
1. Open new handle for read/write,
1. Ask Windows to read and then write again.

Repeat until process is killed.

Additional note, something I realized really late on, but I do know from my Trainer classes in C#. **Those Read/Write handles.... they are supposed to be closed**. So that's a new read/write handle opened and **never** closed... each 3 seconds as the application runs.

I will also try and have a look at 2nd thread and its function, but having got up over 17 hours ago, I'll just get some sleep for now.