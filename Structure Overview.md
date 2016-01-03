#### spine-unity
The information here may change over time as the implementations within Spine-Unity get updated, improved or fixed.
This contains intermediate-level documentation. If you're just starting out, try the [Getting Started](https://github.com/pharan/spine-unity-docs) document.  

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

## SkeletonAnimation
Spine Runtime Documentation details the guts of the Spine runtimes because that level of use is the expectation of most lower-level game frameworks. AND for people who want to implement/port the Spine runtime to their engine of choice.

Normally, `Spine.AnimationState` manages keeping track of time, updating the skeleton, queueing, layering and mixing/crossfading animation. If you're used to "Mecanim" as having State Machines, `Spine.AnimationState` is a simpler, non-programmable kind, but flexible enough to serve as the base for most animation logic.

But `Spine.AnimationState` is a C# class. To make it usable in Unity, the `Spine.AnimationState` object is wrapped in a `MonoBehaviour` called `SkeletonAnimation`. You will find that `SkeletonAnimation` has a member field called `state`, a reference to the `Spine.AnimationState` object.

It's through `state` that you can play or queue animations.
In code, you can play animations through `AnimationState` methods.

```csharp
// Plays the animation named “stand” once on Track 0.
skeletonAnimation.state.SetAnimation(0, “stand”, false);

// Queues the animation named “run” to loop on Track 0 after the last animation is done.
skeletonAnimation.state.AddAnimation(0, “run”, true, 0f);

```  

`SkeletonAnimation` both manages the timing of the updates (through `Update`) and generates the Mesh object since it derives from the `SkeletonRenderer` class. This is the main component that's added to the `GameObject` when you Instantiate a SkeletonDataAsset into a "Spine GameObject". You could say that SkeletonAnimation is *the* Spine component.

For more information on SkeletonAnimation, see the [Animation documentation](https://github.com/pharan/spine-unity-docs/blob/master/Animation.md).


## Spine-C# ##
### Spine-C# vs. Spine-libGDX
Because of the very similar syntax (and the language tradition) Spine-C# closely resembles Spine-libGDX(Java), except where C# and Java differ.

In Spine-C#:

 - The core library used in Spine-C# is [mscorlib](http://referencesource.microsoft.com/#mscorlib,namespaces)*. This includes Dictionary for maps, Array for array methods.
 - Methods and enum values are identified by **PascalCase** instead of **camelCase**. This is the C# language standard.
 - Properties and Auto-Properties are used instead of Java’s getter and setter methods.
   This is purely a syntax and API matter. Under the hood, properties are compiled into methods.
 - Events definition and subscription are done with C# events and delegates.

*For performance reasons, Spine-C# uses the included `ExposedList<T>` class instead of `System.Collections.Generic.List<T>` in ****hot running code.
