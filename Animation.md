
#### spine-unity
The information here may change over time as the implementations within Spine-Unity get updated, improved or fixed.
This contains mostly intermediate-level documentation. If you're just starting out, try the [Getting Started](https://github.com/pharan/spine-unity-docs/blob/master/Getting%20Started.md) document.

----------

![](http://i.imgur.com/7qOlCn8.png)
# Animations and Controlling Animation


## SkeletonAnimation
Normally, `Spine.AnimationState` manages keeping track of time, updating the skeleton, queueing, layering and mixing/crossfading animation. If you're used to "Mecanim" as having State Machines, `Spine.AnimationState` is a simpler, non-programmable kind, but flexible enough to serve as the base for most animation logic.

But `Spine.AnimationState` is a C# class. To make it usable in Unity, the `Spine.AnimationState` object is wrapped in a `MonoBehaviour` called `SkeletonAnimation`. You will find that `SkeletonAnimation` has a member field called `state`, a reference to the `Spine.AnimationState` object.  

`SkeletonAnimation` both manages the timing of the updates (through `Update`) and generates the Mesh object since it derives from the `SkeletonRenderer` class. This is the main component that's added to the `GameObject` when you Instantiate a Skeleton Data Asset into a "Spine GameObject". You could say that SkeletonAnimation is *the* Spine component.

## How to use SkeletonAnimation

### FOR BEGINNERS
“Out of the box”, an instantiated `GameObject` with a `SkeletonAnimation` component on it is ready to go. Changing the Animation dropdown in the Inspector allows you to set the initial animation for that SkeletonAnimation object which starts playing immediately when the scene starts.

Looping can be enabled and disabled, and the Time Scale can be changed. These Inspector properties map to `.AnimationName`, `.loop` and `.timeScale`.

For beginner code, you can use `skeletonAnimation.AnimationName` property to change the playing animation with its name. It will start playing and loop if the `skeletonAnimation.loop` was set to true at the time it was changed.

### FOR TYPICAL USE
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
For example, you can change the `.Time` immediately after `SetAnimation` so the animation starts in the middle instead of the beginning. Note that time is in seconds. To convert Spine editor frames to seconds, you must divide by the dopesheet’s 30 fps tick marks: `time = (frameNumber/30f)`.

```csharp
     // Play a "dance" animation starting from frame 10
     var trackEntry = skeletonAnimation.state.SetAnimation(0, "dance", false);
     trackEntry.Time = 10f/30f;

     // You can shorten the above code this way if you only need
     // to change one field, and don't need to store the trackEntry.
     skeletonAnimation.state.SetAnimation(0, "dance", false).Time = 10f/30f;
```

You can change the playback speed by setting that `TrackEntry`’s `.TimeScale`. This gets multiplied by the `SkeletonAnimation.timeScale` and `AnimationState.timeScale` to get the final timescale.

You can also change the point where the animation ends by setting `.EndTime`.
You can subscribe to events just for that specific animation playback instance.

## Spine.Animation

A `Spine.Animation` object corresponds to an individual "animation clip" animated in Spine. If a run animation called "run" was animated in Spine, you'll get a `Spine.Animation` object named "run".

Each `Spine.Animation` is a collection of `Timeline` objects. Each `Timeline` object is a collection of keys, which defines how a certain animatable value (scale, rotation, position, color, attachment, mesh vertices, IK, events, draw order) changes over time. Each `Timeline` has its own target: a `Bone` for scale, rotation and position; a `slot` for attachments and colors; a `MeshAttachment` for free-form deformation/mesh vertices, and the appropriate parts of the skeleton for *draw order* and *events*.

In Spine runtimes, a `Spine.Animation` is applied to a skeleton with `Animation.Apply(...)`. This poses the `Skeleton`'s various parts only according to that Animation's `Timeline`s and keys. If a `Timeline` isn't there, the animatable value won't change. If a `Timeline` is there, it will overwrite whatever value is there with a new value.

This means that if you play two Animations at the same time using `Spine.AnimationState`'s track system, the `Timeline`s of the higher track will overwrite any changes that the lower track applied, but will not modify anything else.

This system allows the user to combine **different animations for different parts of the skeleton** and play them back independently of each other. 

#### One Animation After Another
This also means that without **auto-reset logic**, playing animations one after another will not necessarily cause it to look like what the animations like in Spine editor. Instead, playing a sequence of animations will result in later animations inheriting the values and bone poses of the previous animation.


In the inspector, `SkeletonAnimation` has a **Generic Auto-reset checkbox** under "Advanced". Check that box if you want to enable it. You can also set the public bool property `SkeletonAnimation.Autoreset` to `true`.

## Animation Callbacks + Spine Events
**Spine.AnimationState** provides animation callbacks in the form of [C# events](https://msdn.microsoft.com/en-us/library/awbftdfh.aspx). You can use these to handle some basic points of animation playback.

Spine.AnimationState raises events:
 - `Start` when an animation starts playing,
 - `End` when an animation is cleared or interrupted,
 - `Complete` when an animation completes its full duration,
 - `Event` when a user-defined event is detected.

**WARNING:**
> NEVER subscribe to `End` with a method that calls `SetAnimation`. Since `End` is raised when an animation is interrupted, and `SetAnimation` interrupts any existing animation, this will cause an **infinite recursion** of End->Handle>SetAnimation->End->Handle->SetAnimation, causing Unity to freeze until a stack overflow happens.

To learn more about Spine Events and AnimationState callbacks, see the [Events documentation](https://github.com/pharan/spine-unity-docs/blob/master/Events.md).

----------

### Advanced Animation Control
Tracks and TrackEntry are just part of `Spine.AnimationState`, and `AnimationState` is included with the Spine runtimes just to get you going immediately. It's usable in most cases and performant for production.
However, Spine-Unity and Spine-C# can function even without `AnimationState`. For example, `SkeletonAnimator` uses `UnityEngine.Animator` instead of `Spine.AnimationState` and uses Mecanim to manage numbers and data for mixing, layering and queueing.

If you wanted to do bookkeeping of your animations a different way, whether to be optimized based on how you plan to use it, or to handle special cases just the way you want them, you can build a different system.

In this case, studying the [Official Spine Documentation](http://esotericsoftware.com/spine-using-runtimes) will come in handy.