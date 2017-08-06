
#### spine-unity
The information here may change over time as the implementations within Spine-Unity get updated, improved or fixed.
This contains intermediate-level documentation. If you're just starting out, try the [Getting Started](/Getting-Started.md) document.

Documentation last updated for Spine-Unity for Spine 3.6.x (2017 Aug 4)
If this documentation doesn't cover some questions, please feel free to post in the official [Spine-Unity forums](http://esotericsoftware.com/forum/viewforum.php?f=3). 


# Controlling Animation
![](/img/spine-runtimes-guide/spine-unity/animationcoding.png)

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
skeletonAnimation.TimeScale = 1.5f;
skeletonAnimation.Loop = true;
skeletonAnimation.AnimationName = "run";
```

### Left and Right Animations
It is not necessary for you to create separate left-facing and right-facing animations in Spine editor.
`SkeletonAnimation`'s `Skeleton` object has a FlipX property (and FlipY) you can set which horizontally flips the rendering and logical positions of bones of that skeleton.
```csharp
// If your character was facing right, this will cause it to face left.
skeletonAnimation.Skeleton.FlipX = true; 
```

However, if you opted to design your character asymmetrically but the movement is essentially the same in both directions, you can use Spine's Skins feature to have two skins— one facing right, and one facing left— and switch between those two as your character faces left and right.
```csharp
bool facingLeft = (facing != "right");  
skeletonAnimation.Skeleton.FlipX = facingLeft;
skeletonAnimation.Skeleton.SetSkin(facingLeft ? "leftSkin" : "rightSkin");
```


### For Typical Use
The class that actually controls the animation of the skeleton is **Spine.AnimationState**.
`SkeletonAnimation` is built around `AnimationState`. It stores a reference to its instance of `Spine.AnimationState` in `SkeletonAnimation.AnimationState`. This is instantiated on `Awake` so make sure to only access it on `Start` or any time after.

`AnimationState` is Spine’s common-case implementation of a Spine Animation State Machine. It is lower-level in a sense that it only manages updating, queuing, layering and mixing/crossfading animations. (It is not a "State Machine" in the Mecanim sense).

In code, you can play animations through `AnimationState` methods:
```csharp
// Plays the animation named “stand” once on Track 0.
skeletonAnimation.AnimationState.SetAnimation(0, “stand”, false);
// The last pose of a non-looping animation will persist until you clear it or replace it with another animation.

// Queues the animation named “run” to loop on Track 0 after the last animation is done.
skeletonAnimation.AnimationState.AddAnimation(0, “run”, true, 0f);
```

Mixing/crossfading options can be found on your `SkeletonDataAsset`'s inspector. There you'll find a field for "Default Mix" which is the default duration of a mix between two animations on the same track. You may also define specific mix durations for specific pairs of animations there.


#### TRACKS
Tracks are animation layers, for playing more than one Spine animation at once.

This is useful for when, for example, you have an animation for a character's body running, but also have an animation for the arm shooting a gun.

If you play an animation on track 0, and another animation on track 1, both animations will play at the same time and end or loop according to their own durations independently.

The animations keys of the animation playing on the higher track will override the keys of the lower track: higher track number, higher priority.

```csharp
// Play a running animation on track 0, and a shooting animation on track 1.
skeletonAnimation.AnimationState.SetAnimation(0, "run", true);
skeletonAnimation.AnimationState.SetAnimation(1, "shoot", false);
```

You can see a sample of this in the 

#### TRACKENTRY
When you call `SetAnimation` or` AddAnimation`, it returns a `TrackEntry` object.

This object represents the instance of an animation playback for the animation you just played or queued. It holds information about what animation it is, its elapsed time, its own timescale and several other properties.

`TrackEntry` lets you control the state of a playing or queued animation, and playback parameters not included in `SetAnimation` and `AddAnimation`.

If you choose not to keep the reference to the `TrackEntry` object when you called SetAnimation or AddAnimation, you can get the currently active TrackEntry by calling
```csharp
	var trackEntry = skeletonAnimation.AnimationState.GetCurrent(myTrackNumber); // null if no animation is playing.
```


###### Current Time (or Start Time)
The time the Animation is currently playing at can be changed at any point while it's playing.

For example, you can change the `.TrackTime` immediately after `SetAnimation` so the animation starts in the middle instead of the beginning. Note that time is in seconds. To convert Spine editor frames to seconds, you must divide by the dopesheet’s 30 fps tick marks: `AnimationTime = (frameNumber/30f)`.

```csharp
     // Play a "dance" animation starting from frame 10
     var trackEntry = skeletonAnimation.AnimationState.SetAnimation(0, "dance", false);
     trackEntry.AnimationTime = 10f/30f;

     // You can shorten the above code this way if you only need
     // to change one field, and don't need to store the trackEntry.
     skeletonAnimation.AnimationState.SetAnimation(0, "dance", false).TrackTime = 10f/30f;
```

> If you are doing this to Animations with animation events, make sure you also set `lastTime` to the same value as well.
> If `lastTime` is left as 0, all events between time 0 and `time` will be captured and raised/fired in the next Update.

You can also change the point when the TrackEntry animation ends by setting `.TrackEnd`.

###### Timescale
You can change the playback speed by setting that `TrackEntry`’s `.TimeScale`. This gets multiplied by the `SkeletonAnimation.TimeScale` and `AnimationState.TimeScale` to get the final timescale.

> You can set `TimeScale` to 0 so it pauses. Note that even if `TimeScale = 0` results in the skeleton not moving, the animation is still be applied to the skeleton every frame. Any changes you make to the skeleton will still be overridden in the normal updates.

Here is a sample helper method if you want to jump to a certain point in time.
```csharp
static public Spine.TrackEntry JumpToTime (SkeletonAnimation skeletonAnimation, int trackNumber, float time, bool skipEvents, bool stop) {
     if (skeletonAnimation == null) return null;
     return JumpToTime(skeletonAnimation.AnimationState.GetCurrent(trackNumber), time, skipEvents, stop);
}

static public Spine.TrackEntry JumpToTime (Spine.TrackEntry trackEntry, float time, bool skipEvents, bool stop) {
    if (trackEntry != null) {
		trackEntry.AnimationTime = time;
		if (skipEvents)
			trackEntry.AnimationLast = time;

		if (stop)
			trackEntry.TimeScale = 0;
	}
	return trackEntry;
}
```

###### TrackEntry-specific events.
You can subscribe to events just for that specific animation playback instance. See the list of events here: [Events Documentation](/Events.md).

## Spine.Animation

A `Spine.Animation` object corresponds to an individual "animation clip" animated in Spine. If a run animation called "run" was animated in Spine, you'll get a `Spine.Animation` object named "run".

Each `Spine.Animation` is a collection of `Timeline` objects. Each `Timeline` object is a collection of keys, which defines how a certain animatable value (scale, rotation, position, color, attachment, mesh vertices, IK, events, draw order) changes over time. Each `Timeline` has its own target: a `Bone` for scale, rotation and position; a `slot` for attachments and colors; a `MeshAttachment` for mesh deformation, and the appropriate parts of the skeleton for *draw order* and *events*.

In Spine runtimes, a `Spine.Animation` is applied to a skeleton with `Animation.Apply(...)`. This poses the `Skeleton`'s various parts only according to that Animation's `Timeline`s and keys. If a `Timeline` isn't there, the animatable value won't change. If a `Timeline` is there, it will overwrite whatever value is there with a new value.

This means that if you play two Animations at the same time using `Spine.AnimationState`'s track system, the `Timeline`s of the higher track will overwrite any changes that the lower track applied, but will not modify anything else.

This system allows the user to combine **different animations for different parts of the skeleton** and play them back independently of each other.

### Posing the Skeleton according to an animation frame/time.

You can call `Animation.Apply` directly if you want to pose a skeleton according to a certain point in time in that `Animation`. The Spine-Unity runtime also has a `Skeleton.PoseWithAnimation` (Unity-specific) extension method that allows you to do this according to an animation's name.

One of the parameters is always time. This is time in seconds. If you want to pose the skeleton according to what you see in the editor, you may need to call `skeleton.SetToSetupPose()` before calling `Animation.Apply` or `Skeleton.PoseWithAnimation` (Unity-specific)


## Animation Callbacks + Spine Events
Sometimes, you need your game to execute a piece of code based on events in and around animations, such as sounds and effects. **Spine.AnimationState** provides animation callbacks in the form of [C# events](https://msdn.microsoft.com/en-us/library/awbftdfh.aspx). You can use these to handle some basic points of animation playback such as the start of an animation, when it ends or is interrupted, when it completes or loops, or event that were user-created in Spine.

To learn more about Spine Events and AnimationState callbacks, see the [Events documentation](Events.md).

### Advanced Animation Control
Tracks and TrackEntry are just part of `Spine.AnimationState`, and `AnimationState` is included with the Spine runtimes just to get you going immediately. It's usable in most cases and performant for production.
However, Spine-Unity and Spine-C# can function even without `AnimationState`. For example, `SkeletonAnimator` uses `UnityEngine.Animator` instead of `Spine.AnimationState` and uses Mecanim to manage numbers and data for mixing, layering and queueing.

If you wanted to do bookkeeping of your animations a different way, whether to be optimized based on how you plan to use it, or to handle special cases just the way you want them, you can build a different system.

In this case, studying the [Official Spine Documentation](http://esotericsoftware.com/spine-using-runtimes) will come in handy.