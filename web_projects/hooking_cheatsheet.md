---
layout: page
title: Hooking cheat sheet
menu: none
permalink: web_projects/hooking_cheatsheet/
---
> __thiscall hook
{% highlight cs %}
template<class Out, class In>
Out type_pun(In x)
{
	union {
		In a;
		Out b;
	};
	a = x;
	return b;
};

CameraHook::CameraHook()
{
	baseModule = GetModuleHandle(NULL);
	UnprotectModule(baseModule);
	fsub449E10 = (sub449E10)((uintptr_t)baseModule + 0x49E10);
	HookJmpTrampoline((DWORD)baseModule + 0x4AFB0, type_pun<void*>(&CameraHook::hk_SetCameraProperties), 0x39);
}
{% endhighlight cs %}