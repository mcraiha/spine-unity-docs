> **Licensing**
> 
> This Spine runtime may only be used for personal or internal use, typically to evaluate Spine before purchasing. If you would like to incorporate a Spine Runtime into your applications, distribute software containing a Spine Runtime, or modify a Spine Runtime, then you will need a valid [Spine license](https://esotericsoftware.com/spine-purchase). Please see the [Spine Runtimes Software License](https://github.com/EsotericSoftware/spine-runtimes/blob/master/LICENSE) for detailed information.
> The Spine Runtimes are developed with the intent to be used with data exported from Spine. By purchasing Spine, `Section 2` of the [Spine Software License](https://esotericsoftware.com/files/license.txt) grants the right to create and distribute derivative works of the Spine Runtimes.

# Getting Started

## Installing
1. Download and install [Unity](http://unity3d.com/get-unity) (There's a free version.)
1. Create an empty Unity project.
1. Download the latest spine-unity.unitypackage: http://esotericsoftware.com/files/runtimes/unity/spine-unity.unitypackage.
1. Import the unitypackage (you can double-click on it and Unity will open it).
1. Go to the `Examples\Getting Started` folder in the Project panel. Open and examine those Unity Scene files in order. Make sure you read the text in the scene, check out the inspector and open the relevant sample scripts.

> The **Spine-Unity** runtime provides functionality to load, manipulate and render [Spine](http://esotericsoftware.com) skeletal animation data in [Unity](http://unity3d.com/) engine.
>  
> Spine-Unity works in Unity without the need for any other plugins. But it also works with [2D Toolkit](http://www.unikronsoftware.com/2dtoolkit/) and can render skeletons using TK2D's texture atlas system. To enable this, open Unity's `Preferences...` and under the `Spine` tab, you can enable TK2D.
>
> If you are not familiar with programming in C# and using Unity in general, we recommend watching the [official Unity Tutorials](http://unity3d.com/learn/tutorials) first. The [Interface Essentials](http://unity3d.com/learn/tutorials/topics/interface-essentials) and then [Scripting](http://unity3d.com/learn/tutorials/topics/interface-essentials) topics are a good place to start. Their Animation topic is not directly applicable to Spine-Unity so there's no need to learn that to know how to use Spine-Unity.
>
> Spine-Unity is built on top of Spine-C# ([spine-csharp](https://github.com/EsotericSoftware/spine-runtimes/tree/master/spine-csharp)).

## Sample Code
After you have imported the latest unitypackage, go to the `Examples\Getting Started` folder in the Project panel.

Open and examine those Unity Scene files in order.
Make sure you read the text in the scene, check out the inspector and open the relevant sample scripts.

Those will cover the basics of playing animations and posing characters.

Visit the [Spine-Unity forum](http://esotericsoftware.com/forum/viewforum.php?f=12) for more information.

## Bringing Your Spine Assets Into Your Project
### Exporting from Spine
1. After you have created your skeleton and animations, click on `Spine Menu`>`Export...` (`CTRL`+`E`). This opens the **Export window**.
1. Choose `JSON` on the upper-left of the Export window.
1. Check the `Create atlas` checkbox. (Checking `Nonessential data`, `Pretty print` are also recommended for beginners).
	1. Click on `Settings` beside the `Create atlas` checkbox. This opens the **Texture Packer Settings** window.
	1. On the lower-right, look for the textbox labeled `Atlas extension` and make sure it is set to `.atlas.txt`. (This is to work around Unity not accepting file extensions it doesn't recognize despite being plain text, but the Spine-Unity runtime can actually handle auto-renaming `.atlas` files. However, setting it to `.atlas.txt` minimizes problems and ambiguities with subsequent exports.)
	1. You're done with the Texture Packer Settings window. Click `OK` to close.
1. In the **Export window**, pick an output folder. (Recommendation: Make a new empty folder. Make sure you can find it.)
1. Click `Export`.
1. This will export three files:
	- a **.json** file that holds data of the skeleton.
	- a **.png** file which is the packed version of all your images in one texture.
	- a **.atlas.txt** file (libGDX atlas) that has data of where each image is in the packed texture.

> For __2D Toolkit__ users, Step 3 (packing a `.png` and `.atlas.txt`) is not necessary. Instead, you will have the appropriate field in your SkeletonDataAsset to assign a reference to `tk2dSpriteCollectionData`. To enable this, open Unity's `Preferences...` and under the `Spine` tab, you can enable TK2D.

### Importing into Unity
1. Make sure your Unity project is open.
	- It should already have a functioning Spine-Unity runtime in it.
1. Look for the folder where you exported your 3 files. (**json**, **.atlas.txt** and **.png**)
1. Drag the 3 files (or the folder containing them) into Unity. Drop them in Unity's **Project panel**.
	- This will cause the Spine-Unity runtime to process them and auto-generate the necessary Unity assets.
	- You will notice 3 new files.
		- a **_Material** asset that holds references to the shader and **.png** texture.
		- an **_Atlas** asset that holds a reference to the material and the **.atlas.txt**.
		- a **_SkeletonData** asset that holds a reference to the **json** and the **_Atlas** asset.
4. Right-click on the **_SkeletonData** asset and choose `Spine > Instantiate (SkeletonAnimation)`. This will instantiate a new Spine GameObject.
	- See the `Examples\Getting Started` sample scenes to learn more about Spine GameObjects.


> - **MANUAL ASSET SETUP** For advanced cases, you can create these three files yourself. The arrangement is noted in the bullet points of Step 3.
> - **CHANGING SHADERS** Using Spine-Unity's default shaders (`Spine/Skeleton` or `Spine/SkeletonLit`) requires textures that were saved with **Premultiplied Alpha**. This is the default setting in Spine's Texture Packer. Choosing other shaders may yield unexpected results and may require you to re-export the texture without premultiplied alpha. For more information on premultiply alpha, [check this forum topic](http://esotericsoftware.com/forum/Premultiply-Alpha-3132).
> - **TEXTURE SIZES.** Unity scales overly large images down by default. The Spine-Unity runtime automatically sets atlas maximum sizes to 2048x2048. If your textures are larger than this, it will cause atlas coordinates to be incorrect. Make sure the import settings are set appropriately, or decrease the maximum page width and height in your Spine Texture Packer settings.
> - **TEXTURE ARTIFACTS FROM COMPRESSION.** Unity's 2D project defaults import new images added to the project with the Texture Type "Sprite". This can cause artifacts when using the `Spine/Skeleton` shader. To avoid these artifacts, make sure the Texture Type is set to "Texture" and . Spine-Unity's automatic import will attempt to apply these settings but in the process of updating your textures, these settings may be reverted.


# Updating Your Project's Spine-Unity Runtime
Some Spine editor updates require that you update your Spine-Unity runtime so it reads and interprets exported Spine data correctly.

- As with Unity updates, it is always recommended that you back up your whole Unity project before performing an update.
- Always check with your Lead Programmer and Technical Artist before updating your Spine runtime. Spine runtimes are source-available and designed to be user-modifiable based on varying project needs. Your project's spine runtime may have been modified by your programmer. In this case, updating to the latest runtime also requires reapplying those modifications to the new version of the runtime.
- You have three options for updating your Spine-Unity runtime. An in-place update with Unity's `Import Package` dialog is the recommended option. In rare, complicated cases, you may have to delete your older version of spine-unity and then import the unitypackage.   

## In-place Update (.unitypackage)
1. Download the latest spine-unity.unitypackage: http://esotericsoftware.com/files/runtimes/unity/spine-unity.unitypackage.
2. Import the .unitypackage into your project by double-clicking on the unitypackage file or dragging it into Unity editor.
3. The Import dialog will show which files are updated and will update them regardless of where you moved them in your project since you last imported.

>  - This functionality may not work correctly if your meta files were corrupted or replaced. In that case, you may have to do delete the old version of the runtime before importing the unitypackage.
>  - Much older versions of the unitypackage had inconsistent meta files for spine-c#. You may have to delete the older spine-csharp folder when updating. This otherwise works correctly.
>  - Occasionally, some files may be removed and merged. Importing the .unitypackage will not delete those files so you may have to do that yourself. Check here for announcements regarding that: http://esotericsoftware.com/forum/Noteworthy-Spine-Unity-Topics-5924 

## Manual (.unitypackage)
1. Open an empty scene. (to be safe)
1. Delete the previous spine-unity and spine-csharp runtime from your Unity project.
1. Import the .unitypackage normally.

## Manual (zip from github)
1. Open an empty scene. (to be safe)
1. Download the zip file of the spine runtimes from the github repository: https://github.com/EsotericSoftware/spine-runtimes.
	1. Extract the files where you can find them. (You only need `spine-csharp` and `spine-unity`)
1. Look for the correct folders and replace them in your project.
	- These are the following folders you need
		- `spine-csharp/src` (this is named "spine-csharp" in the unitypackage)
		- `spine-unity/Assets/Gizmos`.
		- `spine-unity/Assets/spine-unity`
		- `spine-unity/Assets/Examples`(optional)

> - `Gizmos` is a [special folder](http://docs.unity3d.com/Manual/SpecialFolders.html) in Unity. It needs to be at the root of your assets folder to function correctly. (ie. `Assets/Gizmos`)
> - `spine-csharp` and `spine-unity` can be placed in any subfolder you want.
> - Sometimes, some files get moved around and you may get duplicate files. Unity will warn you when this happens. Make sure you delete the older versions whenever this happens.
> - Some files may have been removed and merged since the last version you used. Just copying the whole folder will not delete them. Check here for announcements regarding that: http://esotericsoftware.com/forum/Noteworthy-Spine-Unity-Topics-5924


----------


# Overall Structure
How is Spine-Unity Put together?
A functioning Spine-Unity setup consists of:

 - Spine-C#
 - Spine-Unity
 - ... and Unity Engine itself.

## Spine-C# ##
**spine-csharp** is the C# port of the common core of all Spine runtimes

This common core contains the code for the classes and data structures you can see in Spine editor, including  `Skeleton`, `Bone`, `Slot`, `Skin`, `Attachment`, `Animation`. These are also the classes described in the [Official Spine Documentation](http://esotericsoftware.com/spine-using-runtimes). 

It deserializes  JSON (`.json`) and binary (`.skel.bytes` in Unity) skeleton data files into the SkeletonData object model.

It also includes the base implementation of `AnimationState`, Spine's basic, out-of-the-box Animation controller. (More on this below)

All of its classes are under the `Spine` namespace. 

**spine-csharp** is shared between **Spine-Unity**, **Spine-XNA** and **Spine-MonoGame**— all C# runtimes. Any other potential Spine runtimes written in .NET/mono can use Spine-C# as its base. Because of this, it uses none of UnityEngine’s classes or functionality.

To maintain compatibility with Unity’s older Mono runtime and compiler, Spine-C# doesn’t use any of the latest C# language features.

### Stateful vs Stateless
The Spine core classes (defined in spine-csharp) are generally divided into *stateful-instance* and *stateless-data* classes:

Classes like `Skeleton`, `Bone`, `Slot`, `AnimationState` define *stateful* objects and can be modified without affecting other instances. You can flip a `Skeleton`, reposition a `Bone`, or modify a `Slot` or change the timeScale of an `AnimationState`, and it will only apply to that single instance.

Classes like `SkeletonData`, `Animation`, `Attachment`, `BoneData`, `SlotData` define *stateless* objects and, by default and by design, are shared across (and will affect) all Skeleton instances you create. (Each Spine Unity Spine GameObject creates one Skeleton). Most of the time, you won't need to (and probably shouldn't) modify these stateless objects.

As a rule of thumb:
- If you want to find *stateful* objects, access them through `Skeleton`. (eg, when you want to pose a skeleton or swap a slot attachment)
- If you want to find *stateless* objects, access them through `SkeletonData`. (eg, when you want information about an `Animation` like its duration, or an `Event`, or the setup/bind pose) 

This is just the intended design. If you are an advanced user and know your way around the runtime, you can define how "stateless" objects really are for your project's needs.

You can find this and more detailed information about the base runtime and Spine core classes [here](http://esotericsoftware.com/spine-using-runtimes).

### Differences between Spine-C# and Spine-libGDX
Because of the very similar syntax (and the language tradition) Spine-C# closely resembles Spine-libGDX(Java), except where C# and Java differ.

In Spine-C#:

 - The core library used in Spine-C# is [mscorlib](http://referencesource.microsoft.com/#mscorlib,namespaces)^. This includes `Dictionary` for maps, `Array` for array methods.
 - Methods and enum values are identified by **PascalCase** instead of **camelCase**. This is the C# language standard. (For serialization compatibility reasons, some enums are camelCase.)
 - Properties and Auto-Properties are used instead of Java’s getter and setter methods.
   This is purely a syntax and API matter. Under the hood, properties are compiled into methods and occasionally inlined by the Mono JIT compiler.
 - Events definition and subscription are done with C# events and delegates.

> ^ For performance reasons, Spine-C# uses the included `ExposedList(T)` class instead of `System.Collections.Generic.List(T)` in hot-running code.

## Spine-Unity
Spine-Unity is the layer around Spine-C# that allows it to work with `UnityEngine`, and be usable in Unity Editor through inspectors and windows.

It contains definitions of Spine MonoBehaviours and Asset files, as well as various editor tools, automatic asset processing for user convenience, and code tools like [PropertyAttributes](http://docs.unity3d.com/ScriptReference/PropertyDrawer.html).

Spine-Unity mostly times and manages the functions that Spine-C# does according to [UnityEngine’s lifecycles and game loop](http://docs.unity3d.com/Manual/ExecutionOrder.html). 
But it also contains the main render code (in `.LateUpdate`) which turns Spine skeletons into visible meshes.

Spine-Unity also times and manages the deserialization and instantiation of Spine-C# classes in-editor and at runtime.

In scripting, this is relevant: all Spine-C# classes are instantiated on `Awake`. This means that without controlling Script Execution Order directly, instances of Skeleton and AnimationState are not guaranteed to exist once your script’s `Awake` is called.

You should access (and cache) references to `.state` or `.skeleton` on or after `Start`. This is in line with [Unity’s documentation for Awake](http://docs.unity3d.com/ScriptReference/MonoBehaviour.Awake.html).

### SkeletonUtility and Spine-Unity Modules
Spine-Unity also includes classes for optional functionality, and alternative workflows.

`SkeletonAnimator` allows users to use a Mecanim `AnimationController` to drive animation.
SkeletonUtility allows interfacing between the transforms of `Spine.Bones` and `UnityEngine.Transform`s.
This lets you drive `Bone` movement by controlling `UnityEngine.Transform`s, or let `GameObject`s follow `Bone` movements.

Several other modules are also included in the `spine-unity\Modules\` folder, which are ready-to-use for certain simple cases, but are really useful as sample implementations to be studied or modified for more advanced uses.

You can freely remove unused modules from your project by deleting their specific folders without worrying about affecting the rest of Spine's functionality. Note: some modules may be dependent on another module. Try removing them one by one. Unity should warn you after compiling if some classes were dependent on those deleted files. 

# Controlling Animation
![](http://i.imgur.com/7qOlCn8.png)

## SkeletonAnimation
Normally, `Spine.AnimationState` manages keeping track of time, updating the skeleton, queueing, layering and mixing/crossfading animation. If you're used to "Mecanim" as having State Machines, `Spine.AnimationState` is a simpler, non-programmable kind, but flexible enough to serve as the base for most animation logic.

But `Spine.AnimationState` is a C# class. To make it usable in Unity, the `Spine.AnimationState` object is wrapped in a `MonoBehaviour` called `SkeletonAnimation`. You will find that `SkeletonAnimation` has a member field called `state`, a reference to the `Spine.AnimationState` object.  

`SkeletonAnimation` both manages the timing of the updates (through `Update`) and generates the Mesh object since it derives from the `SkeletonRenderer` class. This is the main component that's added to the `GameObject` when you Instantiate a Skeleton Data Asset into a "Spine GameObject". You could say that SkeletonAnimation is *the* Spine component.

## How to use SkeletonAnimation

### For Beginners
“Out of the box”, an instantiated `GameObject` with a `SkeletonAnimation` component on it is ready to go. Changing the Animation dropdown in the Inspector allows you to set the initial animation for that SkeletonAnimation object which starts playing immediately when the scene starts.

Looping can be enabled and disabled, and the Time Scale can be changed. These Inspector properties map to `.AnimationName`, `.loop` and `.timeScale`.

For beginner code, you can use `skeletonAnimation.AnimationName` property to change the playing animation with its name. It will start playing and loop if the `skeletonAnimation.loop` was set to true at the time it was changed.

```
skeletonAnimation.timeScale = 1.5f;
skeletonAnimation.loop = true;
skeletonAnimation.AnimationName = "run";
```

### Left and Right Animations
It is not necessary for you to animate left-facing and right-facing animations in Spine editor.
`SkeletonAnimation`'s `Skeleton` object has a FlipX property (and FlipY) you can set which horizontally flips the rendering and logical positions of bones of that skeleton.
```csharp
// If your character was facing right, this will cause it to face left.
skeletonAnimation.skeleton.FlipX = true; 
```

However, if you opted to design your character asymmetrically but the movement is essentially the same in both directions, you can use Spine's Skins feature to have two skins— one facing right, and one facing left— and switch between those two as your character faces left and right.
```csharp
bool facingLeft = (facing != "right");  
skeletonAnimation.skeleton.FlipX = facingLeft;
skeletonAnimation.skeleton.SetSkin(facingLeft ? "leftSkin" : "rightSkin");
```


### For Typical Use
The class that actually controls the animation of the skeleton is **Spine.AnimationState**.
`SkeletonAnimation` is built around `AnimationState`. It stores a reference to its instance of `Spine.AnimationState` in `SkeletonAnimation.state`. This is instantiated on `Awake` so make sure to only access it on `Start` or any time after.

`AnimationState` is Spine’s common-case implementation of a Spine Animation State Machine. It is lower-level in a sense that it only manages updating, queuing, layering and mixing/crossfading animations. (It is not a "State Machine" in the Mecanim sense).

In code, you can play animations through `AnimationState` methods:
```csharp
// Plays the animation named “stand” once on Track 0.
skeletonAnimation.state.SetAnimation(0, “stand”, false);

// Queues the animation named “run” to loop on Track 0 after the last animation is done.
skeletonAnimation.state.AddAnimation(0, “run”, true, 0f);
```

Mixing/crossfading options can be found on your `SkeletonDataAsset`'s inspector. There you'll find a field for "Default Mix" which is the default duration of a mix between two animations on the same track. You may also define specific mix durations for specific pairs of animations there.


#### TRACKS
Tracks are animation layers, for playing more than one Spine animation at once.

This is useful for when, for example, you have an animation for a character's body running, but also have an animation for the arm shooting a gun.

If you play an animation on track 0, and another animation on track 1, both animations will play at the same time and end or loop according to their own durations independently.

The animations keys of the animation playing on the higher track will override the keys of the lower track: higher track number, higher priority.

```csharp
// Play a running animation on track 0, and a shooting animation on track 1.
skeletonAnimation.state.SetAnimation(0, "run", true);
skeletonAnimation.state.SetAnimation(1, "shoot", false);
```

#### TRACKENTRY
When you call `SetAnimation` or` AddAnimation`, it returns a `TrackEntry` object.

This object represents the instance of an animation playback for the animation you just played or queued. It holds information about what animation it is, its elapsed time, its own timescale and several other properties.

`TrackEntry` lets you control the state of a playing or queued animation, and playback parameters not included in `SetAnimation` and `AddAnimation`.

If you choose not to keep the reference to the `TrackEntry` object when you called SetAnimation or AddAnimation, you can get the currently active TrackEntry by calling
```csharp
	var trackEntry = skeletonAnimation.state.GetCurrent(myTrackNumber); // null if no animation is playing.
```


###### Current Time (or Start Time)
The time the Animation is currently playing at can be changed at any point while it's playing.

For example, you can change the `.Time` immediately after `SetAnimation` so the animation starts in the middle instead of the beginning. Note that time is in seconds. To convert Spine editor frames to seconds, you must divide by the dopesheet’s 30 fps tick marks: `time = (frameNumber/30f)`.

```csharp
     // Play a "dance" animation starting from frame 10
     var trackEntry = skeletonAnimation.state.SetAnimation(0, "dance", false);
     trackEntry.Time = 10f/30f;

     // You can shorten the above code this way if you only need
     // to change one field, and don't need to store the trackEntry.
     skeletonAnimation.state.SetAnimation(0, "dance", false).Time = 10f/30f;
```

> If you are doing this to Animations with animation events, make sure you also set `lastTime` to the same value as well.
> If `lastTime` is left as 0, all events between time 0 and `time` will be captured and raised/fired in the next Update.

You can also change the point where the animation ends by setting `.EndTime`.

###### Timescale
You can change the playback speed by setting that `TrackEntry`’s `.TimeScale`. This gets multiplied by the `SkeletonAnimation.timeScale` and `AnimationState.timeScale` to get the final timescale.

> You can set timeScale to 0 so it pauses. Note that even if `timeScale = 0` results in the skeleton not moving, the animation is still be applied to the skeleton every frame. Any changes you make to the skeleton will still be overridden in the normal updates.

Here is a sample helper method if you want to jump to a certain point in time.
```csharp
static public Spine.TrackEntry JumpToTime (SkeletonAnimation skeletonAnimation, int trackNumber, float time, bool skipEvents, bool stop) {
     if (skeletonAnimation == null) return null;
     return JumpToTime(skeletonAnimation.state.GetCurrent(trackNumber), time, skipEvents, stop);
}

static public Spine.TrackEntry JumpToTime (Spine.TrackEntry trackEntry, float time, bool skipEvents, bool stop) {
    if (trackEntry != null) {
		trackEntry.time = time;
		if (skipEvents)
			trackEntry.lastTime = time;	// in pre-3.0. This ignores attachment keys too.

		if (stop)
			trackEntry.timeScale = 0;
	}
	return trackEntry;
}
```


###### TrackEntry-specific events.
You can subscribe to events just for that specific animation playback instance. See the list of events here: [Events Documentation](http://esotericsoftware.com/spine-unity-events).

## Spine.Animation

A `Spine.Animation` object corresponds to an individual "animation clip" animated in Spine. If a run animation called "run" was animated in Spine, you'll get a `Spine.Animation` object named "run".

Each `Spine.Animation` is a collection of `Timeline` objects. Each `Timeline` object is a collection of keys, which defines how a certain animatable value (scale, rotation, position, color, attachment, mesh vertices, IK, events, draw order) changes over time. Each `Timeline` has its own target: a `Bone` for scale, rotation and position; a `slot` for attachments and colors; a `MeshAttachment` for mesh deformation, and the appropriate parts of the skeleton for *draw order* and *events*.

In Spine runtimes, a `Spine.Animation` is applied to a skeleton with `Animation.Apply(...)`. This poses the `Skeleton`'s various parts only according to that Animation's `Timeline`s and keys. If a `Timeline` isn't there, the animatable value won't change. If a `Timeline` is there, it will overwrite whatever value is there with a new value.

This means that if you play two Animations at the same time using `Spine.AnimationState`'s track system, the `Timeline`s of the higher track will overwrite any changes that the lower track applied, but will not modify anything else.

This system allows the user to combine **different animations for different parts of the skeleton** and play them back independently of each other.

### Posing the Skeleton according to an animation frame/time.

You can call `Animation.Apply` directly if you want to pose a skeleton according to a certain point in time in that `Animation`. The Spine-Unity runtime also has a `Skeleton.PoseWithAnimation` extension method that allows you to do this according to an animation's name.

One of the parameters is always time. This is time in seconds. If you want to pose the skeleton according to what you see in the editor, you may need to call `skeleton.SetToSetupPose()` before calling `Animation.Apply` or `Skeleton.PoseWithAnimation`

#### One Animation After Another (pre-3.0)
This also means that without **auto-reset logic**, playing animations one after another will not necessarily cause it to look like what the animations like in Spine editor. Instead, playing a sequence of animations will result in later animations inheriting the values and bone poses of the previous animation.

In 3.0, Spine.AnimationState will handle auto-reset logic itself and give you the option to use it so it can behave like it does in Spine editor, or if you want the original free-rein mode for advanced uses.


## Animation Callbacks + Spine Events
**Spine.AnimationState** provides animation callbacks in the form of [C# events](https://msdn.microsoft.com/en-us/library/awbftdfh.aspx). You can use these to handle some basic points of animation playback.

Spine.AnimationState raises events:
 - `Start` when an animation starts playing,
 - `End` when an animation is cleared or interrupted,
 - `Complete` when an animation completes its full duration,
 - `Event` when a user-defined event is detected.

**WARNING:**
> NEVER subscribe to `End` with a method that calls `SetAnimation`. Since `End` is raised when an animation is interrupted, and `SetAnimation` interrupts any existing animation, this will cause an **infinite recursion** of End->Handle>SetAnimation->End->Handle->SetAnimation, causing Unity to freeze until a stack overflow happens.

To learn more about Spine Events and AnimationState callbacks, see the [Events documentation](http://esotericsoftware.com/spine-unity-events).

### Advanced Animation Control
Tracks and TrackEntry are just part of `Spine.AnimationState`, and `AnimationState` is included with the Spine runtimes just to get you going immediately. It's usable in most cases and performant for production.
However, Spine-Unity and Spine-C# can function even without `AnimationState`. For example, `SkeletonAnimator` uses `UnityEngine.Animator` instead of `Spine.AnimationState` and uses Mecanim to manage numbers and data for mixing, layering and queueing.

If you wanted to do bookkeeping of your animations a different way, whether to be optimized based on how you plan to use it, or to handle special cases just the way you want them, you can build a different system.

In this case, studying the [Official Spine Documentation](http://esotericsoftware.com/spine-using-runtimes) will come in handy.

# Rendering
In general, Spine’s systems are built to take advantage of the way modern game engines use 3D graphics. This is especially true for its advanced features like Mesh Deformation and Weights.

It does this by interfacing with frameworks and engines “in 3D terms”; in terms of meshes and vertices, and texture mapping and UVs. So every image part is either defined by a polygon comprised of several triangles, or a rectangle made up of two triangles. This is not unusual even if the engine is mainly for 2D graphics.

![](http://i.imgur.com/XMsoUww.jpg)

The basic idea is this:
You have mesh made of triangles. This is shaped based on the Spine skeleton/model.
You have texture data. This comes from the atlas texture.
And you have shader programs or blending instructions.

You give all that to the game engine, the game engine gives it to the GPU, and then it renders. The textures are mapped to the mesh, and rendered based on the shader and blend mode.

![](http://i.imgur.com/URrFWUW.jpg)

Spine-Unity rendering uses Unity’s `MeshRenderer` and `MeshFilter` components, and Material assets. These are the same components that Unity uses to render 3D models.

The `SkeletonRenderer class` (base class of `SkeletonAnimation`) builds the mesh in its `LateUpdate` method. It generates arrays for vertices, vertex colors, triangle indexes and UVs, and puts all that inside a `UnityEngine.Mesh` object.
The `Mesh` goes into the `MeshFilter` component, which is used by the `MeshRenderer`.
The `MeshRenderer` renders the mesh at the appropriate point in [Unity’s game loop](http://docs.unity3d.com/Manual/ExecutionOrder.html).

`MeshFilter` and `MeshRenderer` have a [Retained Mode](https://en.wikipedia.org/wiki/Retained_mode) API (as opposed to an Immediate Mode API); that is, mesh and materials don't need to be passed to them every frame for the visibility to persist. This means that while `SkeletonRenderer`/`SkeletonAnimation` can update the mesh every frame, disabling the updates at runtime (by disabling `SkeletonAnimation` or `SkeletonRenderer`) will not destroy the visible mesh.

> **Advanced use: Frame skipping per skeleton**
> This also means that, after the initial mesh generation step, you can control and skip mesh updates to implement specific frameskipping and staggered-update behavior. This can be used to minimize updates on less important game elements, or achieve a stylistic frame-by-frame look to the animations.

## Materials
Spine-Unity also uses Materials to store information about what texture, shader and material properties should be used. The Materials are assigned through the `AtlasAsset`.

The materials array of the `MeshRenderer` is managed by the `SkeletonRenderer` every frame depending on what `AtlasAssets` it needs to use. Modifying that array directly does not work like a typical Unity setup.

### So many Materials
You may notice that your MeshRenderer has a lot of Materials— particularly, more materials than you actually set in your `AtlasAsset`.

This is not unusual if you have more than one atlas page. The renderer's material array is not supposed to reflect the order and number of items in your AtlasAsset. SkeletonRenderer lays out a Material array based on what materials it needs to render Spine Attachments.

If all attachments share one material, SkeletonRenderer only puts one Material in MeshRenderer.

If some attachments require material A and some material B, a Material array is laid out according to the order the materials are needed. This is based on the order the attachments are drawn and which attachments can be found in which Material's texture.

If the order goes:
> Attachment from A
> 
> Attachment from A
> 
> Attachment from B
> 
> Attachment from A

The resulting material array will be:
> Material A (for the first two)
> 
> Material B (for the third)
> 
> Material A (for the fourth)

In other words, the more the attachments alternate between coming from A and coming from B, the more materials there will be in the material array, and each item in the material array signifies it needing to switch materials.

The Dragon example demonstrates this:
![](http://i.imgur.com/2aHrOfh.png)

More materials in this array means more [draw calls](http://docs.unity3d.com/Manual/DrawCallBatching.html), which can adversely affect performance if the same skeleton is instantiated several times. If you only have one of this skeleton, it is probably not a cause for significant slowdown.

To learn more about how to arrange atlas regions in your Spine atlases, see this page: [Spine Texture Packer: Folder Structure](http://esotericsoftware.com/spine-texture-packer#Folder-structure)

### Setting Material Properties Per Instance.
Likewise, using MeshRenderer.material and changing values there will not work.

The `Renderer.material` property generates a copy just for that renderer, but that material will immediately be overwritten by SkeletonRenderer's render code.   

On the other hand, `Renderer.sharedMaterial` will modify the original Material. And if you spawn more than one Spine GameObject that uses that Material, the changes you apply to it will be applied to all instances.

In this case, Unity's [Renderer.SetPropertyBlock](http://docs.unity3d.com/ScriptReference/Renderer.SetPropertyBlock.html) is a useful method. Remember that `SkeletonRenderer`/`SkeletonAnimation` uses a `MeshRenderer`. Setting that MeshRenderer's material property block allows you to change property values just for that renderer.

```csharp
MaterialPropertyBlock mpb = new MaterialPropertyBlock();
mpb.SetColor("_FillColor", Color.red); // "_FillColor" is a named property on a hypothetical shader. 
GetComponent<MeshRenderer>().SetPropertyBlock(mpb);
```

> **Notes on optimization**
> - using Renderer.SetPropertyBlock allows renderers with the same material to be batched correctly even if their material properties are changed by different MaterialPropertyBlocks.
> - You do need to call `SetPropertyBlock` whenever you change or add a property value to your MaterialPropertyBlock. But you can keep that MaterialPropertyBlock as part of your class so you don't have to keep instantiating a new one whenever you want to change a property.
> - When you need to set a property frequently, you can use the static method: `Shader.PropertyToID(string)` to cache the int ID of that property instead of using the string overload of MaterialPropertyBlock's setters.  

## Sprite Texture Rendering/Z Ordering
![](http://i.imgur.com/xqDt3Sn.jpg)

Spine typically* uses alpha-blending to render your Spine model parts.

When you draw, paint and export parts for your Spine project, you export them as PNG files which encode pixels as RGBA. Red, Green, Blue and Alpha. In the file, each pixel in the alpha channel can have a value from 0 to 255, just like the RGB channels, representing 256 levels of transparency/opacity for each pixel.

This transparency/semi-transparency, defined by the alpha channel, presents classical problems for depth/render order. These problems have many imperfect solutions and cause many of the weird 3D artifacts we see today even in our AAA games.

But in very standard fashion, ie. just like in most 2D alpha-blended sprite rendering cases including Unity’s own `Sprite`/`SpriteRenderer` system, Spine-Unity uses a shader that does not use the 3D Z-buffer nor alpha testing to determine what should be rendered in front and behind.

Within one mesh, it just renders things in the order defined by the order of the triangles of the mesh, drawing one thing on top of another. This order is what Spine uses to control Slot Draw Order.

Between meshes, Spine-Unity uses many of Unity’s render order systems to determine what sprite/mesh should be on top of which. Using the standard Spine-Unity setup, here is typical behavior for it:

- Mesh data is passed to Unity as a whole.
- It is rendered by the GPU triangle-by-triangle.
- Whole meshes are rendered in an order determined by many factors:
   - Distance from the camera. (Camera determines [what kind of distance; planar or perspective](http://docs.unity3d.com/ScriptReference/Camera-transparencySortMode.html).)
   - Renderer Sorting Layer/Sorting Order. ([All UnityEngine.Renderers have these sorting properties](https://unity3d.com/learn/tutorials/modules/beginner/2d/sorting-layers).)
   - Shader’s ShaderLab Queue Tag (defaults to the “Transparent” queue like other sprites)
   - Material.renderQueue. (does nothing until you set it. It just overrides the shaderlab queue tag).
   - [Camera depth](http://docs.unity3d.com/ScriptReference/Camera-depth.html). (For multi-camera setups.)

For the most part, you won’t need to and shouldn’t touch Material.renderQueue and the shader’s queue tag.

*You can opt to use a different shader to get different kinds of blending, by changing the shader on the Material. Handling and working around the resulting blending behavior will likewise be up to you.

### Sorting Layer and Order
The **Sorting Layer** and **Sorting Order** properties found in `SkeletonRenderer`/`SkeletonAnimation`’s Inspector actually modifies the `MeshRenderer`’s [sorting layer](http://docs.unity3d.com/ScriptReference/Renderer-sortingLayerID.html) and [sorting order](http://docs.unity3d.com/ScriptReference/Renderer-sortingOrder.html) properties.

Despite being hidden in MeshRenderer’s inspector, these properties are actually serialized/stored normally as part of `MeshRenderer` and not part of SkeletonRenderer.

### Sorting via Camera Distance
If you keep all your renderers in the same sorting layer and order, they will be subject to the other sorting schemes mentioned above.

If the queue tag is also the same (if you're using the same material and shader), then you should be able to control sorting of your Spine GameObjects according to camera distance.

Also remember that [perspective cameras have sorting modes](http://docs.unity3d.com/ScriptReference/Camera-transparencySortMode.html). 

### So can I ever render anything between parts of my Spine skeleton?
Sometimes, you need your character to ride a bicycle, or lift a rock hug an Ionic column. In Spine terms, this means you have to render some parts behind a thing, and some parts in front of a thing.

In Unity, meshes are rendered as a whole. So how do you get one to render behind and in front?

Please see [SkeletonRenderSeparator](https://github.com/pharan/spine-unity-docs/blob/master/SkeletonRenderSeparator.md)

### I lowered the alpha to fade out my skeleton. Why are the overlaps showing?
This is how realtime mesh rendering works anywhere. The opacity is applied right when the triangles are drawn, not after.

There are probably many solutions to this. Some workarounds too.
You’ll find many good examples of workarounds from studying how various 2D games do it. Go out and play some of your favorites!

## How does Spine-Unity determine how big my model will be?
Spine editor uses a 1 pixel:1 unit scale. Meaning, that if you just include an image into your skeleton without any rotation or scaling, 1 pixel in that image will be 1 unit tall and 1 unit wide in Spine.

In Unity, 1 unit is 1 meter. This is defined by Unity’s default physics values and constraints (both 2D and 3D). It’s generally not a good idea to use the 1 pixel : 1 unit ratio because of this. Instead, Unity’s own sprites default to scaling down by 1/100; meaning 100 pixels in your image will be 1 Unity unit long.

For convenience, when you import your Spine data into Unity, the scale is set at a default of 0.01 to match Unity’s sprites.

![](http://i.imgur.com/4YHiEUF.png)

If you want to set your skeleton’s base scale to something higher or lower, you can change this value in your Skeleton Data Asset’s inspector. This value is a multiplier used when your `SkeletonData` is loaded. Changing this value at runtime will not do anything after the `SkeletonData` has already been loaded.

If you want to dynamically change/animate your skeleton’s visual scale (like a balloon) in gameplay, you can do that by just changing `gameObject.transform.localScale`.

## Flipping Horizontally or Vertically
Accessing `Skeleton.FlipX` and `Skeleton.FlipY` can allow you to flip the skeleton horizontally or vertically.
It’s usually a good idea to make all your skeletons face one direction so that your flipping logic can be consistent throughout your code. But if not, you can always add an extra `bool` to your controller and [negate your intended flip value with boolean logic](http://stackoverflow.com/questions/13983198/negate-a-boolean-based-on-another-boolean).

You may have learned to flip your 2D sprites in Unity setting a negative scale on the Transform, or rotating it along the y-axis by 180 degrees.
Both of these work for purely visual purposes. But they have their own side effects to keep in mind:

 - Nonuniform scale can cause a mesh to bypass Unity’s batching system. This means it will have its own (set of) draw calls for each instance. So it’s okay for your main character. It can be detrimental if you have nonuniform scale for dozens of skeletons.
 - Rotating causes normals to rotate with the mesh. For lighting 2D sprites, this means they’ll point in the wrong direction.
 - Rotating X or Y may also cause Unity’s 2D Colliders to behave unpredictably.
 - Negative scale can cause attached physics Colliders and some script logic to behave with unexpected results.

