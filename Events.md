#### spine-unity
The information here may change over time as the implementations within Spine-Unity get updated, improved or fixed.

Documentation last updated for Spine-Unity for Spine 3.5.x (2017 April 11)
If this documentation doesn't cover some questions, please feel free to post in the official [Spine-Unity forums](http://esotericsoftware.com/forum/viewforum.php?f=3). 

----------

![](/img/spine-runtimes-guide/spine-unity/events.png)
# Spine Events & AnimationState callbacks

**Spine.AnimationState** provides functionality for animation callbacks in the form of [C# events](https://msdn.microsoft.com/en-us/library/awbftdfh.aspx). You can use these to handle some basic points of animation playback.

> **For the novice programmer**: [Callbacks](https://en.wikipedia.org/wiki/Callback_%28computer_programming%29) mean you can tell the system to inform you when *something* specific happens by giving it a method to call when that something happens. [Events](https://en.wikipedia.org/wiki/Event_%28computing%29) are the meaningful points in program execution— in this case, that you can subscribe to/handle with your own code by providing a function/method for the event system to call.
> 
> When the event happens, the process of calling the function/method you provided is called "raising" or "firing" the event. Most C# documentation will call it "raising". But Spine documentation will call it "firing". Those mean the same thing. 
>  
> The structure and syntax for callback functionality varies from language to language. See the sample code at the bottom for examples of C# syntax.

![](/img/spine-runtimes-guide/spine-unity/callbackchart.png) 
Fig 1. Chart of Events raised without mixing/crossfading.


Spine.AnimationState raises the following events:
 - **Start** is raised when an animation starts playing,
	 - This applies to right when you call `SetAnimation`.
	 - I can also be raised when a queued animation starts playing.
 - **End** is raised an animation is cleared (or interrupted),
	 - This applies to when you call `SetAnimation` before the current animation has a chance to finish.
	 - This is also raised when you clear the track using `ClearTrack` or `ClearTracks`.
	 - During a mix/crossfade, end is raised after a mix is completed.
	 - **NEVER** handle the End event with a method that calls SetAnimation. See the warning below.
	 - Note that by default, non-looping animation TrackEntrys no longer stop at the duration of the animation. Instead, the last frame continues to be applied indefinitely until you clear it or some other animation replaces it. See below for more information on how to make this behave differently.
 - **Dispose** is raised for TrackEntries when AnimationState disposes of a TrackEntry (at the end of its life cycle).
	 - Runtimes like spine-libgdx and spine-csharp pool TrackEntry objects to avoid unnecessary GC pressure. This is particularly important in Unity which has an old and less efficient garbage collection implementation.
	 - It is important to clear all your references to TrackEntries when they are disposed since they could later contain data or raise events that you did not intend to read or observe.
 - **Interrupt** is raised when a new animation is set and a current animation is still playing.
	 - This is raised when an animation starts mixing/crossfading into another animation.
 - **Complete** is raised an animation completes its full duration,
	 - This is raised when a non-looping animation finishes playing, whether or not a next animation is queued.
	 - This is also raised every time a looping animation finishes an loop.
 - **Event** is raised whenever *ANY* user-defined event is detected.
	 - These are events you keyed in animations in Spine editor. They are purple keys. A purple icon can also be found in the Tree view.
	 - To distinguish between different events, you need to check the `Spine.Event e` parameter for its `Name`. (or `Data` reference).
	 - This is useful for when you have to play sounds according to points the animation like footsteps. It can also be used to synchronize or signal non-Spine systems according to Spine animations, such as Unity particle systems or spawning separate effects, or even game logic such as timing when to fire bullets (if you really want to).
	 - Each TrackEntry has an `EventThreshold` property. This defines at what point within a crossfade user events stop being raised. See **Events During Mixing** below for more information.


At the junction where an animation completes playback, and a queued animation will start, the events are raised in this order: `Complete`, `End`, `Start`.

> **WARNING:**
> NEVER subscribe to `End` with a method that calls `SetAnimation`. Since `End` is raised when an animation is interrupted, and `SetAnimation` interrupts any existing animation, this will cause an infinite recursion of End->Handle>SetAnimation->End->Handle->SetAnimation, causing Unity to freeze until a stack overflow happens.

### Events During Mixing

//TODO: ILLUSTRATION OF EVENTS RAISED WITHIN A MIX/CROSSFADE GOES HERE

The standard AnimationState implementation treats events differently when it does mixing.

When you have a mix time set (or `Default Mix` on your Skeleton Data Asset), there is a span of time where the next animation starts being mixed with an increasing alpha, and the previous animation is still being applied to the skeleton.

We can call this span of time, the "crossfade" or "mix".

By default, a TrackEntry stops raising user events right when TrackEntry starts fading out, with the default EventThreshold value of `0`. If you set it to `0.5`, user events stop being raised halfway through the crossfade/mix. If you set it to `1`, events will continue to be raised up until the very last frame of the crossfade/mix. Setting this or the default value to the appropriate value for your animations is important, as you may have overlapping animations that have the same animations and shouldn't raise the same ones, or want events to still be raised even if the animation has been interrupted. 

## Sample Code

Here is a sample `MonoBehaviour` that subscribes to `AnimationState`'s events. Read the comments to see what's going on.
```csharp
// Sample written for for Spine 3.5
using UnityEngine;
using Spine;
using Spine.Unity;

// Add this to the same GameObject as your SkeletonAnimation
public class MySpineEventHandler : MonoBehaviour {

	// The [SpineEvent] attribute makes the inspector for this MonoBehaviour
	// draw the field as a dropdown list of existing event names in your SkeletonData.
	[SpineEvent] public string footstepEventName = "footstep"; 

	void Start () {
		var skeletonAnimation = GetComponent<SkeletonAnimation>();
		if (skeletonAnimation == null) return;	// told you to add this to SkeletonAnimation's GameObject.

		// This is how you subscribe via a declared method. The method needs the correct signature.
		skeletonAnimation.AnimationState.Event += HandleEvent;

		skeletonAnimation.AnimationState.Start += delegate (TrackEntry trackEntry) {
			// You can also use an anonymous delegate.
			Debug.Log(string.Format("track {0} started a new animation.", trackEntry.TrackIndex));
		};

		skeletonAnimation.AnimationState.End += delegate {
			// ... or choose to ignore its parameters.
			Debug.Log("An animation ended!");
		};
	}

	void HandleEvent (TrackEntry trackEntry, Spine.Event e) {
		// Play some sound if the event named "footstep" fired.
		if (e.Data.Name == footstepEventName) {			
			Debug.Log("Play a footstep sound!");
		}
	}
}
```

## Advanced

Since the Spine runtimes are source-available and fully modifiable in your project, you can of-course define and raise your own events in AnimationState or in whatever version of it you make. See the official [Spine-Unity forums](http://esotericsoftware.com/forum/viewforum.php?f=3) for more information.

 