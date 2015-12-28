#### spine-unity
The information here may change over time as the implementations within Spine-Unity get updated, improved or fixed.

## How is Spine-Unity Put together?
A functioning Spine-Unity setup consists of:

 - Spine-C#
 - Spine-Unity
 - ... and Unity Engine itself.

### Spine-C# ###
**Spine-C#** is the C# port of the common core of all Spine runtimes

This common core contains the code for the classes and data structures you can see in Spine editor, including  `Skeleton`, `Bone`, `Slot`, `Skin`, `Attachment`, `Animation`. These are also the classes described in the [Official Spine Documentation](http://esotericsoftware.com/spine-using-runtimes). 

It deserializes  JSON (`.json`) and binary (`.skel.bytes` in Unity) skeleton data files into the SkeletonData object model.

It also includes the base implementation of `AnimationState`, Spine's basic, out-of-the-box Animation controller. (More on this below)

All of its classes are under the `Spine` namespace.

**Spine-C#** is shared between **Spine-Unity**, **Spine-XNA** and **Spine-MonoGame**— all C# runtimes. Any other potential Spine runtimes written in .NET/mono can use Spine-C# as its base. Because of this, it uses none of UnityEngine’s classes or functionality.

To maintain compatibility with Unity’s older Mono runtime and compiler, Spine-C# doesn’t use any of the latest C# language features.

### Spine-Unity
Spine-Unity is the layer around Spine-C# that allows it to work with `UnityEngine`, and work in Unity editor.

It contains definitions of Spine MonoBehaviours and Asset files, as well as various editor tools, automatic asset processing for user convenience, and code tools like [PropertyAttributes](http://docs.unity3d.com/ScriptReference/PropertyDrawer.html).

Spine-Unity mostly times and manages the functions that Spine-C# does according to [UnityEngine’s lifecycles and game loop](http://docs.unity3d.com/Manual/ExecutionOrder.html). 
But it also contains the main render code (in `.LateUpdate`) which turns Spine skeletons into visible meshes.

Spine-Unity also times and manages the deserialization and instantiation of Spine-C# classes in-editor and at runtime.

In scripting, this is relevant: all Spine-C# classes are instantiated on `Awake`. This means that without controlling Script Execution Order directly, instances of Skeleton and AnimationState are not guaranteed to exist once your script’s `Awake` is called.

You should cache or access references to `.state` or `.skeleton` on or after `Start`. This is in line with [Unity’s documentation for Awake](http://docs.unity3d.com/ScriptReference/MonoBehaviour.Awake.html).

### SkeletonUtility and Spine-Unity Extras
Spine-Unity also includes classes for optional functionality, and alternative workflows.

`SkeletonAnimator` allows users to use a Mecanim `AnimationController` to drive animation.
SkeletonUtility allows interfacing between the transforms of `Spine.Bones` and `UnityEngine.Transform`s.
This lets you drive `Bone` movement by controlling `UnityEngine.Transform`s, or let `GameObject`s follow `Bone` movements.

Several other smaller components are also included, which are ready-to-use for certain simple cases, but are really useful as sample implementations to be studied or modified.

///// LIST A FEW/////

## Controlling Animation
Spine Runtime Documentation details the guts of the Spine runtimes because that level of use is the expectation of most lower-level game frameworks. However, Spine-Unity is built differently.

##### FOR BEGINNERS
“Out of the box”, an instantiated GameObject with a SkeletonAnimation component on it is ready to go. Changing the Animation dropdown in the Inspector allows you to set the initial animation for that SkeletonAnimation object which starts playing immediately when the scene starts. Looping can be enabled and disabled, and the Time Scale can be changed.

These Inspector properties map to `.AnimationName`, `.loop` and `.timeScale`.

For beginner code, you can use skeletonAnimation.AnimationName property to change the playing animation with its name. It will start playing and loop if the skeletonAnimation.loop was set to true at the time it was changed.

##### FOR TYPICAL USE
The class that actually controls the animation of the skeleton is **Spine.AnimationState**.
`SkeletonAnimation` is built around `AnimationState`. It stores a reference to its instance in `.state`. This is instantiated on `Awake`.

`AnimationState` is Spine’s common-case implementation of a Spine Animation State Machine. It is lower-level in a sense that it only manages updating, queueing, layering and mixing/crossfading animations. (It is not a "State Machine" in the Mecanim sense).

In code, you can play animations through `AnimationState` methods.
```csharp
// Plays the animation named “stand” once on Track 0.
skeletonAnimation.state.SetAnimation(0, “stand”, false);

// Queues the animation named “run” to loop on Track 0 after the last animation is done.
skeletonAnimation.state.AddAnimation(0, “run”, true, 0f);
```

##### TRACKS
Tracks are animation layers, for playing more than one Spine animation at once.

This is useful for when, for example, you have an animation for a character's body running, but also have an animation for the arm shooting a gun.

If you play an animation on track 0, and another animation on track 1, both animations will play at the same time and end or loop according to their own durations independently.

The animations keys of the animation playing on the higher track will override the keys of the lower track: higher track number, higher priority.

```
// Play a running animation on track 0, and a shooting animation on track 1.
skeletonAnimation.state.SetAnimation(0, "run", true);
skeletonAnimation.state.SetAnimation(1, "shoot", false);
```

##### TRACKENTRY
When you call `SetAnimation` or` AddAnimation`, it returns a `TrackEntry` object.

This object represents the instance of an animation playback for the animation you just played or queued. It holds information about what animation it is, its elapsed time, its own timescale and several other properties.

`TrackEntry` lets you control the state of a playing or queued animation, and playback parameters not included in `SetAnimation` and `AddAnimation`.
For example, you can change the `.Time` immediately after `SetAnimation` so the animation starts in the middle instead of the beginning. Note that time is in seconds. To convert Spine editor frames to seconds, you must divide by the dopesheet’s 30 fps tick marks: `time = (frameNumber/30f)`.

```csharp
     // Play a "dance" animation starting from frame 10
     var trackEntry = skeletonAnimation.state.SetAnimation(0, "dance", false);
     trackEntry.Time = 10f/30f;

     // You can shorten the above code this way if you only need
     // to change one field, and don't need to store the trackEntry.
     skeletonAnimation.state.SetAnimation(0, "dance", false).Time = 10f/30f;
```

You can change the playback speed by setting that TrackEntry’s `.TimeScale`. This gets multiplied by the SkeletonAnimation’s timeScale and AnimationState's timeScale to get the final timescale.

You can also change the point where the animation ends by setting `.EndTime`.
You can subscribe to events just for that specific animation playback instance.

##### Animation Callbacks
**Spine.AnimationState** provides animation callbacks in the form of [C# events](https://msdn.microsoft.com/en-us/library/awbftdfh.aspx). You can use these to handle some basic points of animation playback.

(((ILLUSTRATION HERE FOR THE POINTS OF ANIMATION PLAYBACK)))

**AnimationState.Start** event is raised when an animation starts playing.
**AnimationState.Complete** event is raised when an animation reaches its full duration. It fires at the end with a non-looping animation. And it fires at the end of every loop iteration, for looping animations.
**AnimationState.End** event is raised when 

**WARNING:** NEVER subscribe to `End` with a method that calls `SetAnimation`. Since `End` is raised when an animation is interrupted, and `SetAnimation` interrupts any existing animation, this will cause an infinite recursion of End->Handle>SetAnimation->End->Handle->SetAnimation, causing Unity to freeze until a stack overflow happens.
******* List AnimationState Events here.

((( Sample code for how to subscribe to events. Include anonymous method and named method. )))


### Advanced Animation Control
Tracks and TrackEntry are just part of Spine.AnimationState, and AnimationState is included with the Spine runtimes just to get you going immediately. It's usable in most cases and performant for production.
However, Spine-Unity and Spine-C# can function even without AnimationState. For example, SkeletonAnimator uses `UnityEngine.Animator` instead of `Spine.AnimationState` and uses Mecanim to manage numbers and data for mixing, layering and queueing.

If you wanted to do bookkeeping of your animations a different way, whether to be optimized based on how you plan to use it, or to handle special cases just the way you want them, you can build a different system.

In this case, studying the [Official Spine Documentation](http://esotericsoftware.com/spine-using-runtimes) will come in handy.


## Spine-C# ##
### Spine-C# vs. Spine-libGDX
Because of the very similar syntax (and the language tradition) Spine-C# closely resembles Spine-libGDX(Java), except where C# and Java differ.

In Spine-C#:

 - The core library used in Spine-C# is [mscorlib](http://referencesource.microsoft.com/#mscorlib,namespaces)*. This includes Dictionary for maps, Array for array methods.
 - Methods are identified by **PascalCase** instead of **camelCase**. This is the C# language standard.
 - Properties and Auto-Properties are used instead of Java’s getter and setter methods.
   This is purely a syntax and API matter. Under the hood, properties are compiled into methods.
 - Events definition and subscription are done with C# events and delegates.

*For performance reasons, Spine-C# uses the included `ExposedList<T>` class instead of `System.Collections.Generic.List<T>` in ****hot running code.

> Written with [StackEdit](https://stackedit.io/).
