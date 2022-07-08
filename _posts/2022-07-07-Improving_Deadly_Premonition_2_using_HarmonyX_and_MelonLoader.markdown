---
layout: post
title:  "Improving Deadly Premonition 2 using HarmonyX and MelonLoader"
date:   2022-07-07 10:00:00 +0200
---

At the moment of writing, Deadly Premonition 2's PC port was released almost a month ago now. I started working on it almost the moment I heard about how bad the PC port is, and worked on it pretty much continously for over 3 weeks, either investingating what could be improved, actually experimenting (either in Unity 2018.4.16) or just trying to play through the game to see if anything major gets broken by my changes.

Now with my playthroughh finished - I'll be moving away from active developement and moving over to more of support. This does not mean, no further updates for my hack gets released - I have started working professionally with Unity about a year and half ago - there is still quite a lot for me to learn about it - so if I learn anything that could be retrofitted to the hack, I probably will. Nevertheless, the developement will slow down significently now.

This is a bit of a write up on what changes I have made and how I have made them. The current version of the hack can be [here](https://github.com/SuiMachine/Deadly-Premonition-2---Sui-s-hack/releases)

Il2cpp and what it implies
---
One of the issues and complaints leveled against Unity a lot of the times is that the performance of Unity is really bad, due to it using C#. This is true to some extend, but these claims are generaly overexagurated. Unity in the right hands can perform really fast, even when using Managed code (see [Warhammer 40,000: Mechanicus](https://store.steampowered.com/app/673880/Warhammer_40000_Mechanicus/), [Art of Rally](https://store.steampowered.com/app/550320/art_of_rally/) or [Overload](https://store.steampowered.com/app/448850/Overload/)).

Unity itself doesn't even use .NET and instead relies on their own fork of Mono, which allows to easily develop games that are targetting multiple platforms. By default, Unity will compile the game to use Managed libraries - these libraries don't contain code that CPU can understand and instead containing Common Intermediate Language code (at least to my understanding), in also known as IL code in binary form. When the application is run and a call to a method is made, this code is then procesed at runtime to a code that CPU can execute (Just-in-time compilation - JIT). This has a few problems. 

One of those (although extremely beneficial for modding) is that Managed libraries are easy to read with programs like ilSpy or dnSpy, which can convert IL code to a "lowered" C# code - while you can't obtain the original code using it, the form of code that can be obtained is easily readable (for a programmer) and still close to original code. This representation lacks many of C#'s high-level features and variable names used inside C#'s methods, but neverthless allows to have a pretty good look at original code.

[![originalvsil.png](/images/articles/deadlypremonition2/originalvsil.png)](/images/articles/deadlypremonition2/originalvsil.png)

For comparision, in case C/C++ the code is compiled almost directly to a form that CPU can understand and stripped of all of its variable names, as CPU doesn't need them to its operations. The code is often optimised to be run fast and libraries are loaded at applications startup into RAM - thanks to that approach, if provided proper data and CPU stack, these methods/functions can be executed instantly, which matters very much for applications where CPU performance is imporant - such as video games. The downside is that - this often requires programmers to be very careful about memory allocation, as there are very few protections, that prevent access violation (where a CPU tries to access a part of unallocated memory) or memory leaks (where object is created in memory, but the pointer that point to it is lost, without dealoacting the memory space) - the latter of course has many modern and fast workarouds now (as for the former - I don't know, as I am not have spent enough time with C/C++).

Unity Technologies is fully aware of the overhead requires to run JIT has on high-fidality video games and to work around it without straight up forcing everyone to use different language, they have came up with a solution, which - in many cases - can be implemented into video games just in few clicks known as il2cpp (Intermediate Language To C++). As the full name implies, this solution adds additional process when building the game in Unity, which takes IL libaries, converts them to a C++ code and then uses it to build native binary libraries. Such libraries - as far as I know - always run faster (as long as the limiting factor is the CPU) and limit the amount stuttering possible - this is almost instantly noticable, when you try loading a level that was build with Mono compilation vs. Il2cpp compilation. Il2cpp is not without its flaws, as sometimes it can affect final builds stability, but - having worked with a project that use it for compilation and how it improved so games that had performance issues (such as [Underworld Ascendent](https://store.steampowered.com/app/692840/Underworld_Ascendant/)) - I am extremely impressed with results it can produce.

Don't get me wrong, there is still some garbage collection happening in the background (which has to happen using Unity's main thread!), but thanks to it (and many additional optimisations and components like DOTS), Unity can bring its performance very close to the level of its competition. Il2cpp also ads a layer of security - as the original Il code is no longer available for people (like me) to easily read and understand. It's not the end of the world, as you can still use some tools to find out addresses where native code is located and try to reverse engineer it to original form, but that is a long, frustrating and time consuming process.

Deadly Premonition 2 is one of those games that use Il2cpp. MelonLoader comes with a bunch of tools and libraries to obtain Unhollowed libraries - this is useful, as they provide a point to which code can be hooked by name (instead of address or signature), not to mention access to variable names and general class structure (although that one has some additional garbage information, which is requires for Il2cpp). This is what I used to fix some of the games issues, improve graphics etc with my hack.

HarmonyX and code hooking
---
HarmonyX is a fantastic library that can be used to hotpatch the games method. It structure is a bit weird and hard to understand, but after some time getting used to it, its hard not to love it. First step required is of course installing MelonLoader (as it comes with HarmonyX and Unhollower) and running the game, so Managed libraries can be generated from Unity. These libraries can then be added as references in Visual Studio or Rider.

Let's say for example, we want to prevent the game from forcing a resolution at start up - something I have done. As long as the it's not an extern call, class constructor or override of method inhereted from other class, the process with HarmonyX is fairly straingforward. If you have worked with Unity, you know that the display method will be set using ``Screen.SetResolution()`` - there is a couple of them, but with a process of elimination we can figure out which one exactly is used. In Deadly Premonition 2's this one is used:

```cs
Screen.SetResolution(int width, int height, bool fullscreen)
```

So to hook it, we create a class with a method and paramters like this:
```cs
[HarmonyPatch]
public static class ScreenHook
{
    [HarmonyPrefix]
    [HarmonyPatch(typeof(Screen), "SetResolution", new Type[] { typeof(int), typeof(int), typeof(bool) })]
    public static bool SetResolution1()
    {
        return false;
    }
}
```

I am not fully sure if the first ``[HarmonyPatch]`` attribute is needed, but the second (right above SetResolution1) certainly is. What it does it essentially tells Harmony that we want to find a class ``Screen`` and in it find a method ``SetResolution``, which takes arguments ``int``, ``int`` and ``bool`` and we want to hook it (that last part is not needed if only one method under the same name exists).

What ``[HarmonyPrefix]`` does it tells HarmonyX, that we want to hook it in the way, that we execute the content of ``SetResolution1()`` before the original code is executed. Alternatively, we can execute the code after the original method's code is executed by using a attribute ``[HarmonyPostfix]``.

Now the confusing part about HarmonyX's syntax is that bool does not refer to a type returned by original method. Instead it refers to "whatever we want original method to be executed or not". If we return true, the original method gets executed, if we return false, it gets skipped. This is only possible when using HarmonyPrefix. And so with these few lines, we prevent the game from applying the original resolution.

For more information, you can see [HarmonyX's wiki](https://harmony.pardeike.net/articles/intro.html).

What's great, is that despite the methods having to be static, it is possible to obtain a reference to original object by adding ``__instance`` as method's attribute, as well as the original attributes by using their names:

```cs
[HarmonyPatch]
public static class ScreenHook
{
    [HarmonyPrefix]
    [HarmonyPatch(typeof(Screen), "SetResolution", new Type[] { typeof(int), typeof(int), typeof(bool) })]
    public static bool SetResolution1(Screen __instance, int width, int height, bool fullscreen)
    {
        return false;
    }
}
```

We can also access the class' fields (even if they are private!), by providing a reference type of the field and its name, with \_\_\_ as its name prefix. Example, if we have a class:
```cs
public class CameraMoverMainMenu : MonoBehaviour
{
    private Transform CarCameraTargetTransform;
    private float CarChoooserZoomLevel = 13f;
    public Transform CareerTransform;
    public Transform CarUnlockedTransform;
    private bool HasInitialized;
    private float InitialZoomLevel;
    private Transform RallyCompleteCameraTarget;
    private Vector3 StartingPosition;
    private Quaternion StartingRotation;
    private Camera TheCamera;


    private void Start() {}
    private void Init() {}
    public void SetCarCameraTarget(Transform t) {}
    public void ZoomIn() {}
    public void ZoomOut(bool showDiorama) {}
}
```

We can easily hook it with a code:

```cs
[HarmonyPatch]
public static class SomeOtherClassNameIrrelevant
{
    [HarmonyPrefix]
    [HarmonyPatch(typeof(CameraMoverMainMenu), "ZoomIn")]
    public static bool Hook(CameraMoverMainMenu __instance, bool showDiorama, ref Transform ___RallyCompleteCameraTarget)
    {
        return false;
    }
}
```

Not only we obtained a reference to an object instance, we also have a value of argument the method was just called with and we have a value of the private field of the class... and we can prevent original code from being executed - all with high level code!

Monobehaviours with Il2cpp
---
When not using Entity Component System, Unity generally operates by processing monobehaviours from GameObjects, which are paired with Transforms to exist somewhere in the scene. Transform describes its position in hierarchy in object hierarchy (moving parent of the transform, moves the child transform, while preserving its local position, local rotation and local scale in relation to its parent), but also gives access to its potion and world rotation, as well as few other useful features (like Matrix4x4), massivily simplifying math that is needed to be done when writing code.

[![unity_game_object.png](/images/articles/deadlypremonition2/unity_game_object.png)](/images/articles/deadlypremonition2/unity_game_object.png)

GameObject describes elements such us - which Unity layer is used (useful for handling collisions and rendering), whatever it is "Active" or or not, which scene it belongs to etc. To GameObject, scripts (MonoBehaviours) can be attached, to perform actions, which are then handled by Unity in update cycles, see [Execution Order](https://docs.unity3d.com/Manual/ExecutionOrder.html).

When using MelonLoader (or BepInEx), plugin named [Unity Explorer](https://github.com/sinai-dev/UnityExplorer) to have a look and manipulate game objects in the scene. It gives great insight in how the prefabs used by developers and game world was constructed, what mono behaviours (components) are attached to it etc.
[![unityexplorer.png](/images/articles/deadlypremonition2/unityexplorer.png)](/images/articles/deadlypremonition2/unityexplorer.png)

With MelonLoader (especially when combined with HarmonyX) it is also possible to add new components that will execute additional code. With Il2cpp however, there are a few additional steps that need to be done. Let's just say we have a basic Monobehaviour that moves us 0.1 up each frame.

```cs
public class ExampleBehaviour : MonoBehaviour
{
    private void Update()
    {
        this.transform.position += Vector3.up * 0.1f;
    }
}
```

And lets just use HarmonyX to attach it to ``CharaControl`` to the correct object when it gets Initiated
```cs
[HarmonyPatch]
public static class CharaControlHook
{
    [HarmonyPostfix]
    [HarmonyPatch(typeof(CharaControl), "Init")]
	public static void CharaControlHookInit(CharaControl __instance)
    {
        //Check if we haven't already added it
        if(__instance.GetComponent<ExampleBehaviour>() == null)
        {
            __instance.gameObject.AddComponent<ExampleBehaviour>();
        }
    }
}
```

While this will likely work with Mono Compiled game, it will fail with Il2cpp compiled one. To make it work, we need to add few more things to MonoBehaviour. First thing is ``[RegisterTypeInIl2Cpp]`` attribute and then we need a constructor with a pointer.
```cs
[RegisterTypeInIl2Cpp]
public class ExampleBehaviour : MonoBehaviour
{
    public ExampleBehaviour(IntPtr ptr) : base(ptr) { }

    private void Update()
    {
        this.transform.position += Vector3.up * 0.1f;
    }
}
```

There are a few other quirks. ``obj.GetType()`` won't work for example and ``UnhollowerRuntimeLib.Il2CppType.Of<T>()`` or ``obj.GetIl2CppType()`` have to be used. Coroutines require running using ``MelonCoroutines.Start(__coroutine__here__);``. A good point of reference is [MelonLoader's wiki](https://melonwiki.xyz/#/modders/il2cppdifferences).

Fixing stuff - FixedUpdates and Character interpolation
---
Deadly Premonition 2 has planty of things to fix - one, already mentioned, which relates to resolution, I have already covered. The other common complaint was the character movement which seems to be stuttering. This is a common problem with many Unity games, as the developers opted to handle its movement using FixedUpdate - the characteristing of which is that, it is not tied with the renderered framerate and instead suppose to run a fixed amount of time per second. The amount of updates per second can be configured in Unity, but most games use a value of 50 times a second (delta time of 0.02) and so does DP2. This isn't too much of a problem on Switch (which runs badly as it is), but on PC, where obtaing a framerate of 60 or even higher is pretty much a standard - this ends up being noticable. If you need a graphical examples, I recommend checking "[Fix jittery camera movement in Unity with Rigidbody Interpolate](https://blog.terresquall.com/2021/12/fix-jittery-camera-movement-in-unity-with-rigidbody-interpolate/)" at Terresquall Blog's or "[Timesteps and Achieving Smooth Motion in Unity](https://www.kinematicsoup.com/news/2016/8/9/rrypp5tkubynjwxhxjzd42s3o034o8)" at Kinematic Soup's blog.

To fix that issue, I have decided to write 2 mono behaviours. One, which gets attached to objects that needs to be interpolated, which writes down their position in fixed updates and one which gets attached to the camera, which actually does the interpolation, by disabling rigid bodies, storing their original position and rotation before the frame gets rendered in ``OnPreCull`` method, then after the frame gets rendered, moving them back to their original positions, restoring velocities and state in ``OnPostRender`` (this only works when monobehaviour is attached to a gameobject that has a camera). It also allowed me to easily control for situations, where not to use interpolation (for example during cutscenes), by simply setting flags inside the component responsible for interpolation inside the camera. There was one major bug related to it (and one I was not able to fix), which caused York to start hovering at times and prevent him from falling, which I bypassed by just adding a gravity acceleration if Y velocity was 0.

Fixing stuff  - Controller prompts + keyboard and mouse support
---
Another element that needed fixing was gamepad prompts. The original bindings are just confusing and counter intuitive, as the "Cancel" button for menu is by default bound to Xbox's A, instead of "B" like it is the unspoken standard (or occasionally X). Since the game uses SteamInput, it is possible to just flip those via Steam, but that leaves us with a situation where prompts are no incorrect. So, to fix that issue, issue I had to hook a component responsible for displaying prompts. I was originally expecting the game just to use standard Unity UI (as that has been introduced I think in 2014 to Unity), but - for whatever reason and to my surprise - it turned out the game uses a third party component NGUI. This was a problem, as to replace those prompts, I needed to find an older version of NGUI (as the most recent version of NGUI uses different atlasses), but also not the "Free" version, which turned out to be way too old. I managed to do so and after preparing proper atlases in Unity 2018.

[![ngui.png](/images/articles/deadlypremonition2/ngui.png)](/images/articles/deadlypremonition2/ngui.png)

And then marking the atlas, material and gameobject to be an [asset bundle](https://docs.unity3d.com/Manual/AssetBundles-Workflow.html) and generating asset bundle for each of the controller layout. Then it was all a matter or making sure these asset bundles get loaded using my hack / mod, instantiating a proper game object, tagging it to not be destroyed on scene change and setting a static reference.
```cs
void Awake()
{
    if (SuisHackMain.Settings.Entry_Other_Prompts.Value != "" || SuisHackMain.Settings.Input_Override.Value == ExposedSettings.InputType.KeyboardAndMouse)
    {
        var assetBundlePath = SuisHackMain.Settings.Entry_Other_Prompts.Value;
        this.ReplacementTypeUsed = ReplacementType.Basic;
        if (SuisHackMain.Settings.Input_Override.Value == ExposedSettings.InputType.KeyboardAndMouse)
        {
            this.ReplacementTypeUsed = ReplacementType.KeyboardAndMouse;
            assetBundlePath = "keyboard";
        }

        string path = Path.Combine(Path.Combine(Application.streamingAssetsPath, "prompts"), assetBundlePath);
        SuisHackMain.loggerInst.Msg("Trying to load replacement prompts using bundle: " + path);

        var assetBundle = AssetBundle.LoadFromFile(path);
        if (assetBundle == null)
        {
            SuisHackMain.loggerInst.Error("Failed to load asset bundle!");
            return;
        }

        //Load game object
        var assets = assetBundle.LoadAsset("prompts_go");
        if (assets != null)
        {
            var cast = assets.TryCast<GameObject>();
            if (cast != null)
            {
                if (cast.GetComponent<UIAtlas>() != null)
                {
                    var instanitate = GameObject.Instantiate<GameObject>(cast);
                    instanitate.transform.SetParent(this.transform);
                    instanitate.transform.localPosition = Vector3.zero;
                    instanitate.transform.localRotation = Quaternion.identity;
                    Atlas = instanitate.GetComponentInChildren<UIAtlas>();
                    
                    switch (ReplacementTypeUsed)
                    {
                        case ReplacementType.KeyboardAndMouse:
                            Cache = new PromptCacheKeyboard(Atlas);
                            break;
                        default:
                            Cache = new PromptCacheBasic(Atlas);
                            break;
                    }
                }
                else
                {
                    SuisHackMain.loggerInst.Error("Atlas Game object was found, but UIAtlas was null. Crap!");
                    return;
                }
            }
            else
            {
                SuisHackMain.loggerInst.Error("Atlas game object cast failed - object might be invalid.");
                return;
            }
        }
        else
            SuisHackMain.loggerInst.Error("Failed to find Xbox atlas... fuck!");

        if (Atlas != null)
            SuisHackMain.loggerInst.Msg("Replacement atlas loaded correctly!!");
    }
}
```

Then just hooking the proper component to check the static reference if there is a replacement and use it if it exists.
```cs
public static void UISpriteOn(UISprite __instance)
{
    if (GlobalReplacementAtlas.Instance != null && __instance.mSpriteName != null)
    {
        switch (__instance.mSpriteName)
        {
            case "NX_Cont_Button_A":
                GlobalReplacementAtlas.Instance.Replace(__instance, GamepadKeyIcons.A);
                break;
            case "common_intaract_A":
                GlobalReplacementAtlas.Instance.Replace(__instance, GamepadKeyIcons.A2);
                break;
            case "NX_Cont_Button_B":
                GlobalReplacementAtlas.Instance.Replace(__instance, GamepadKeyIcons.B);
                break;
            case "NX_Cont_Button_X":
                GlobalReplacementAtlas.Instance.Replace(__instance, GamepadKeyIcons.X);
                break;
            case "NX_Cont_Button_Y":
                GlobalReplacementAtlas.Instance.Replace(__instance, GamepadKeyIcons.Y);
                break;
		}
	}
}
```

I did use a manual patching in this case. As using ``[HarmonyPatch]`` attributes causes a class to be automatically patched and I needed patching to happen depending on whatever settings are set.
```cs
public static void Initialize()
{
    if (SuisHackMain.Settings.Input_Override.Value == ExposedSettings.InputType.SteamInput)
    {
        var source = typeof(UISprite).GetMethod(nameof(UISprite.OnInit), System.Reflection.BindingFlags.Instance | System.Reflection.BindingFlags.Public);
        var target = typeof(GamepadPrompts).GetMethod(nameof(GamepadPrompts.UISpriteOn), System.Reflection.BindingFlags.Static | System.Reflection.BindingFlags.Public);

        if (source == null)
        {
            SuisHackMain.loggerInst.Msg("Failed to find source method for GamepadPrompts");
            return;
        }

        if (target == null)
        {
            SuisHackMain.loggerInst.Msg("Failed to find target method for GamepadPrompts");
            return;
        }

        SuisHackMain.harmonyInst.Patch(source, postfix: new HarmonyMethod(target));
        SuisHackMain.loggerInst.Msg("Initialized GamepadPrompts patch");
    }
}
```

I hooked the UISprite in similar way, when using Keyboard and Mouse hook. And whilst I am not entirely happy with keyboard and mouse support, as - without original code - it required me to:
* Write a binary patcher to modify "globalgamemanagers", which contains Unity Axis definitions
* Hook SteamInput to prevent it from reading gamepad and instead when methods are called, check which input is used and then pass it to either Unity's ``Input.GetKey(KeyCode key)`` or ``Input.GetAxis(string axis)`` methods.
I tried also prevent SteamInput from initiating, but sadly it seems like for the calls to be handled properly, it has to be running, which means that if you really want to use Keyboard and mouse to play Deadly Premonition 2, you need a controller that SteamInput detects. Thankfully, it can be a virtual controller and it will work (tested with [vXbox](https://sourceforge.net/projects/vjoy-controller/)).

[![map.png](/images/articles/deadlypremonition2/map.png)](/images/articles/deadlypremonition2/map.png)

To make PC input at least slightly more bareable, I do try and detect game state (namely whatever the player is in menu, in normal gameplay, looking at the map etc.) and then swap input and keyboard/mouse prompts on the fly. For example - when looking at the map, on the controller, you'd normally be using left stick to move the pointer (in the middle of the screen) and press A (or B) to set a waypoint. If translated directly to keyboard and mouse, that would mean using a map with WSAD and pressing E to set a waypoint marker. Instead, what I do is I swap inputs, so that mouse is used for moving the cursor and left mouse button is used for setting up a waypoint. Mouse wheel up and down can be used then to zoom in/zoom out the map (in addition to numpad + / numpad -).

Fixing stuff  - Wires geometry
---
One of the things I have noticed quite early when playing the game, is that electrical wires seems to  disappear. At first I thought this was an issue with LOD Groups, but at further investigation it turned out to be something more trivial. You see, rendering elements is a complicated matter and one of the ways to speed it up, is to use more basic shapes to figure out whatever the elements (or their shadows) are visible on the screen. And some of the most basic ways to check, is to essentially put an entire model inside of a box, as the math requires to check if something is inside of the box or if any of the boxes faces are visible on the screen is fast and straightfoward.

One issue is that wires are often not of a very unique lenghts - more or less the amount of lenght needed to connect point A with B (with a bit of an overhead to account for thermal expansion, poles moving slightly further and closer to each other in the wind etc.). So if you don't want to create seperate meshes and store them in game files for each connection between the poles, what can be used is either a fancy vertex shader that is going to strech one mesh (model) between 2 points or bones. Bones are nowadays an old technology - it started to be used in video games around 2000s (Half-Life famously prouded itself with having skeletal animations) and whilst the intention behind was generally to use it for organic shapes, nothing is stopping developers for applying it to elements like wires. This is exactly what the develoeprs of Deadly Premonition 2 did, as each wire is one and the same model, that is being transformed using a single bone going from point A to B.

The problem is, Skinned meshes that are used to renderer models that are being transformed by bones need to have their bounding set up in a way, that during entire transformation, the model is inside that bounding box. Below is an example of how UnityChan's bounding box looks like for the part that encapsulates her skin (arm, part of chest and neck).

[![bounding_box.png](/images/articles/deadlypremonition2/bounding_box.png)](/images/articles/deadlypremonition2/bounding_box.png)

If I were to modify the models bounding box and in a way that it does no longer encapsulete the models arms, what will happen the moment, the camera does not see a models bounding box, it will stop rendering the model as well. This can be bypassed (although in not a very efficient way), by toggling "Update when Offscreen" in Skinned Mesh Renderer component. Instead I have decided to fix it the correct way, by simplying calculating the proper bounding box. I knew that the one of the ends of the wire is always rendered correctly and having a quick look at its bounding box coordinates, it turned out it has local position 0,0,0 (X,Y,Z). So all that was needed to do is calculate a box that encapsulates entire mesh from local position 0,0,0 to B and Unity makes operations like this very easy, thanks to its [InverseTransformPoint](https://docs.unity3d.com/ScriptReference/Transform.InverseTransformPoint.html) method as that automatically handles object orientations and scale in relation to each other.

```cs
public void OnEnable()
{
    var skinnedMeshes = GetComponentsInChildren<SkinnedMeshRenderer>(true);
    for (int i = 0; i < skinnedMeshes.Length; i++)
    {
        var newBound = new Bounds();

        var skinnedMesh = skinnedMeshes[i].transform;
        var objectsToEnapsulateParent = skinnedMesh.transform.parent;
        
        for (int j = 0; j < objectsToEnapsulateParent.childCount; j++)
        {
            var childToEncapsulate = objectsToEnapsulateParent.GetChild(j);
            if (childToEncapsulate == skinnedMesh.transform)
                continue;
                
            var point = skinnedMesh.transform.InverseTransformPoint(childToEncapsulate.transform.position);
            newBound.Encapsulate(point);
        }
        
        skinnedMeshes[i].localBounds = newBound;
    }
}
```

And there we go:
[![geometry_improvements.png](/images/articles/deadlypremonition2/geometry_improvements.png)](/images/articles/deadlypremonition2/geometry_improvements.png)

The only problem was that scripts used when rendering wires don't exactly have anything that could be hooked that is exacuted once, so instead I opted out to create a new game object, if the scene ends up being open world, which has a script that iterates over root objects in the scene and checks if it should be fixed and if so attaches a custom component that fixes it (which also gives me a way of knowing it was already fixed and shouldn't be fixed again). The check happens 1 game object per frame, to not introduce performance issues, so there is a slight chance, that if a chunk of terrain gets loaded, it will take a few seconds for the wires that player can see to be fixed.

What I also did (although it wasn't a bug), which is part of geometry improvements option, is I modified characters LOD size, so they now get rendered at longer distance, before getting culled (as originally they were culled around 20 metres from the player and that is already on PCs LOD Bias being 2.0, meaning, the % of the screen they have to take is now a half of the original!).

Not fixed geometry
---
Sadly it isn't the end of issues with geometry rendering with this game. One of the issues that are quite noticable is tree imposters that are rendered way too close to the camera and whilst the LOD Bias pushes them back, it seems that from a certain distance, regardless of settings imposters will be used. These imposters are just flat, low-resolution textured planes - they are extremely noticable, they don't cast any shadows and are something I was hoping to improve but I was not able to.

[![incorrect_instanced_geometry.png](/images/articles/deadlypremonition2/incorrect_instanced_geometry.png)](/images/articles/deadlypremonition2/incorrect_instanced_geometry.png)

The other element I was hoping to fix and I was able to was the way the game renders distance fances. For those, from a certain distance, it seems [Graphics.DrawMeshInstanced](https://docs.unity3d.com/ScriptReference/Graphics.DrawMeshInstanced.html) is used (as that's what Matrix4x4 arrays would indicate), which allow to render multiple instances of meshes without having to spawn that mesh in the world and communicate with GPU for each draw call. If I am correct, that probably means that the developers had some process developed for level designers that would generate Matrix4x4 arrays for each terrain chunk in places, where fences and walls were spawned. Unfortunetly, what seems to have happened, is that there was some bug with calculating their position and now, when they are used (only for distant models), all of their geometry is either somewhere off to the side or stuck in the ground. At the example above you can see a fence with pillars that is drawn correctly, using a mesh renderer and left of it - what I assume - is a mesh rendered with DrawMeshInstanced method, which ends up in the ground. Of course if you approach it, then it gets swapped to normal mesh renderer and is rendered exactly where it should be, but nevertheless it is noticable and it is a bug that developers left.

There are also examples of terrain that is not connected between terrain chunks correctly, which causes gaps in geometry. This could possbily be fixed in runtime, if the developers were actually using terrain, but what this game does (despite me referring to terrain) is actually using mesh renderer. Unity has its terrain system - it had it for a very long time and that system does allow for setting up neighbours, to make sure terrain mesh doesn't fall appart in places where terrains connect with each other. And the developer used it, but instead of outputing it to a game, they opted out to use an external component that converts the terrain into a mesh - so not much I can do here.

[![filtering.png](/images/articles/deadlypremonition2/filtering.png)](/images/articles/deadlypremonition2/filtering.png)

There is also an issue with road textures. Which are low resolution and seem to be unfiltered. I could possibly replace those in runtime, if a good and free replacement were to be found.

Options and lifting the game way past Switch limitations
---
I will try to keep this section short - god knows, I already have enough of writting this and I am not even done.

One of the issues with this port is that it essentially gives not settings to the player, trying to use a fixed 1920x1080 resolution with a custom PC quality preset. Unity is a flexible engine and it can change settings at the fly and as such a huge part of my hack was exposing those settings to the player and making sure they get applied correctly.

To allow for easy configuration, I used Unity's GUILayout, which is a part of Unity's ImGui module. This way, instead of having to design custom canvases and UI with options, I could just create a game object with a proper script and ImGui would automatically do the rest of the job, as I wrote which options to expose.

First few options exposed were static settings under [QualitySettings](https://docs.unity3d.com/Manual/class-QualitySettings.html). These included:
* Vsync count (and an option to set desired frame available if Vsync is disabled under Application.targetFrameRate).
* Texture resolution (which allows for lowering the texture resolution by half on really weak machines)
* Anisotropic settings
* Shadow Resolution
* Shadow Distance
* Shadow Cascades amount and their distances (using 4 instead of 2 makes a massive difference)
[![shadowcascades.png](/images/articles/deadlypremonition2/shadowcascades.png)](/images/articles/deadlypremonition2/shadowcascades.png)
* LOD Bias (which pushes the distance at which LODs are getting switched or culled)

After that, the next in line was trying to change the game's anti-aliasing. This a bit harder, as anti-aliasing in deferred rendering path has to be a post process or straight up super sampling (no MSAA allowed) and as such it is set seperatly for each camera assuming the camera has a [PostProcessLayer](https://docs.unity3d.com/Packages/com.unity.postprocessing@2.0/api/UnityEngine.Rendering.PostProcessing.PostProcessLayer.html) componenet attached.

With that I could easily swap the game's default FXAA to way better SMAA or TAA (although the latter causes issues with one of the phantom shaders and as such I recommend not using it, especially if you are a person suffering from epylepsy) and since I already had a reference to a camera, also extend its far clip plane (although it doesn't do much due to the distance at which terrain chunks get loaded).

The next step was finding a component used for ambient occlusion, writing code to hook it and allow player to change its settings as well. The game uses HBAO (Horizon Based Ambient Occlusion), but it is set by default to use Fastest profile, but for most PCs even its Highest setting won't be a problem. However it does affect the way it is calculated so I also exposed its intensity multiplayer, so users can adjust it.

[![hbao.png](/images/articles/deadlypremonition2/hbao.png)](/images/articles/deadlypremonition2/hbao.png)

Then I implemented an override for resolution of render textures used to create planar reflections, as the original resolution is 512x512 and can produce quite pixelated resolutions. The maximum resolution for planar reflections is now 2048x2048.

[![planarreflections.png](/images/articles/deadlypremonition2/planarreflections.png)](/images/articles/deadlypremonition2/planarreflections.png)

The next step was finding a way to hook Post Process Volume used by the camera and modify it, based on player needs. With this I was able to expose a "edge find" filter used, which can now be disabled by the player and screen space reflections, which can now be enabled and modified by the player to make quite a drastic difference in some scenes.

[![ssr.png](/images/articles/deadlypremonition2/ssr.png)](/images/articles/deadlypremonition2/ssr.png)

However, in some it doesn't make almost any difference at all, as it it dependent on materials used. It will also won't introduce any sort of water reflection, as builtin render pipeline has a limitation, where transparent surfaces can not receive SSR (or any depth based post process effect pretty much).

Finally, as the last addition in my experiements, I decided to see what happens if you force shadows. This performance heavy option, that I named "Light Improvements" has 3 presets.
* Disabled - which doesn't modify any lights and leaves them as it is
* Minor - which adds shadows to York's flashlight
* All - which adds shadows to all of the light sources used in closed scenes and adds shadows to vehicles during the night. At times, it also modifies some renderers to actually allow for casting shadows.
[![light_improvements.png](/images/articles/deadlypremonition2/light_improvements.png)](/images/articles/deadlypremonition2/light_improvements.png)

As a side note, I was experimenting with enabling shadows of lamps that are used at night, but the performance impact was too drastic and the result was glitchy, so I decided to leave open world light as it is.

What should have been done when porting
---
Aside from at least some of the improvements made in my hack, which original port should have had by default, there are areas which I was able to improve. One - mentioned earlier being of course issues with tree imposters and fences at the distance, not to mention ground. The area needing most work - at least graphically in the game - is still open world.

As mentioned earlier, if the developers have not opted out to convert all terrains to meshes and instead left them, there was a chance, their mesh could have been improved (in part thanks to Unity alone, increasing mesh density of terrain, when camera gets closer). Furthermore, some foliage really should have been added. Unity's terrain can populatate terrain with grass models and small bushes automatically, when paiting the terrain, so I assume it was originally there (at least at some point) and was cut for performance sake when making cuts for Nintendo Switch.

A proper keyboard and mouse support on PC is pretty much a must, with a very few games (like [Brothers - A Tale of Two Sons](https://store.steampowered.com/app/225080/Brothers__A_Tale_of_Two_Sons/)) having any excuse to not provide it. DP2 is certainly not one of those games - we know a good keyboard and mouse support could have provided and with actual access to a source code - it would probably not take more than 2 weeks of single person's work.

If anything, with my hack - I hope I managed to at least prove that Unity is a very flexible engine and drastic shortcomings that you usually see, are either the fault of developers or (more often than not) incussificent resources.

And yes, I could have probably converted the camera to work in VR - but that would be stupid. See my [Art of Rally VR Hack](https://github.com/SuiMachine/Art-Of-Rally-VR-Hack) or [Raicuparta's guide](https://www.youtube.com/watch?v=Gt_kIrmTl44).