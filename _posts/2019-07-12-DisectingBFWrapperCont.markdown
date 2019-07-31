---
layout: post
title: "Dissecting junky Star Wars: Battlefront patch (part 2)"
date:   2019-07-12 13:00:00 +0200
categories: ["reverse"]
---
Yesterday I wrote my piece on new Battlefront patch, where I focused on aspect ratio fixing function used by 1st thread of the wrapper. Today, I decided to continue dissecting it further and focus on second thread's function, which injects basic FPS limiter into D3D9's Present method.

### IDirect3DDevice9::Present
To start off, let's see what that function does. Its documentation can be found at [https://docs.microsoft.com/en-us/windows/win32/api/d3d9/nf-d3d9-idirect3ddevice9-present](https://docs.microsoft.com/en-us/windows/win32/api/d3d9/nf-d3d9-idirect3ddevice9-present).
> IDirect3DDevice9::Present method, presents the contents of the next buffer in the sequence of back buffers owned by the device.

What this means is, it basically tells API to switch to what would in movies be next frame. Method has following parameters:
{% highlight cpp %}
HRESULT Present(
  const RECT    *pSourceRect,
  const RECT    *pDestRect,
  HWND          hDestWindowOverride,
  const RGNDATA *pDirtyRegion
);
{% endhighlight %}

With possible 2 return values of either ``D3D_OK`` and ``D3DERR_DEVICEREMOVED`` (although there are some other errors it can return as well).

<!--more-->
Method parameters are as follows.
* ``pSourceRect`` - pointer to rectangle describing size of source ''frame''. This paremeter must be **NULL** if unless swap swap effect for buffers was set to **D3DSWAPEFFECT_COPY** when creating swap chain.
* ``pDestRect`` - pointer to rectangle describing coordinates and size inside of the window. If set to **NULL**, a size of the window is used.
* ``hDestWindowOverride`` - handle to a window whose area is used for presentation. If set to **NULL**, hDeviceWindow member of **D3DPRESENT_PARAMETERS** is used.
* ``pDirtyRegion`` - pointer to **RGNDATA struct** which can describe only which pixels to use when swapping buffers (used for optimisation). The value must be NULL unless swap chain was created with **D3DSWAPEFFECT_COPY**.

So in easiest case, we're looking at a function which returns HRESULT and has parameters set to NULL, NULL, NULL, NULL and at worst has all 4 parameters used. What makes this method so useful is that, a Thread Sleep can be implemented before invoking it together with counter as a framelimitter.

### Injection
Now that in theory we know how the FPS limiter is made, let's have a look at how the injection is done. First thing the thread does is it obtains a handle of d3d9.dll inside of the process using:
{% highlight cpp %}
hModuleHandle = GetModuleHandleA("d3d9.dll");
{% endhighlight %}

Then we a basic signature scan (sigscan):
{% highlight cpp %}
offset = 0;
do
{
  currentMask = 'x';
  currentByte = (char *)hModuleHandle + offset;
  i = 0;
  while ( currentMask != 'x' || *currentByte == signature[i] )
  {
    currentMask = mask[i++];
    ++currentByte;
    if ( !currentMask )
    {
      address = (int)hModuleHandle + offset;
      goto LABEL_7;
    }
  }
  ++offset;
}
while ( offset < 0x128000 );
{% endhighlight %}

While it may seem confusing, sigscan is actually quite simple. What the function does is basically it starts of from the first byte of d3d9.dll and continues iterating through all of the bytes until the limit is reached (``while ( offset < 0x128000 )``) or signature is found. When scanning for signature, a byte mask is also used to differentiate which bytes are must be as declared and which may don't matter.

To illustrate it better, here is the signature function is looking for:    
``0xC7, 0x6, 0x0, 0x0, 0x0, 0x0, 0x89, 0x86, 0x0, 0x0, 0x0, 0x0, 0x89, 0x86``    
and its mask:    
``xx????xx????xx``

Combine the two and you get something like this:    
``0xC7, 0x6, ?, ?, ?, ?, 0x89, 0x86, ?, ?, ?, ?, 0x89, 0x86``

Meaning that only bytes which are masked with **x** are meaningful and all of the ones marked with **?** can be literally anything.

Either way, as long as the result function should obtain the location of D3D9::Present function, **assuming the signature is correct and consistent between all d3d9.dll in all modern operating systems**. I am not entirely sure about that last part.

Finally, we get to hooking, which is done copying part of the original functions code to a new place in memory (at least I think that's how it's done in here - might be, that a stub of a function is just defined in wrapper dll). Then function sets assembly instruction jmp with an address to our injected function (instead of the wrapper dll). Then modifies our injected function to point jump back to few assembly procedures that were copied from a function we were hooking and finally at the end of those, a jump back to original function is set, so that the rest is executed normally.
