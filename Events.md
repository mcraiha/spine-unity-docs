#### spine-unity
The information here may change over time as the implementations within Spine-Unity get updated, improved or fixed.

----------

![](http://i.imgur.com/x9sd6yd.png)
## Coding for Spine Events & AnimationState callbacks

**Spine.AnimationState** provides animation callbacks in the form of [C# events](https://msdn.microsoft.com/en-us/library/awbftdfh.aspx). You can use these to handle some basic points of animation playback.

> **For the novice programmer**: "Callbacks"/"Events" mean you can tell the system to inform you when *something* specific happens. It lets you know when that *something* happens by calling a method you gave it prior. The structure and syntax for this varies from language to language. See the sample code at the bottom for examples of C# syntax.

![](http://i.imgur.com/kzv0qRA.png)

Spine.AnimationState raises events:
 - **Start** is raised when an animation starts playing,
	 - This applies to right when you call `SetAnimation`.
	 - I can also be raised when a queued animation starts playing.
 - **End** is raised an animation is cleared or interrupted,
	 - This applies to when you call `SetAnimation` before the current animation has a chance to finish.
	 - This is also raised when you clear the track using `ClearTrack` or `ClearTracks`.
	 - **NEVER** handle the End event with a method that calls SetAnimation. See the warning below. 
 - **Complete** is raised an animation completes its full duration,
	 - This is raised when a non-looping animation finishes playing, whether or not a next animation is queued.
	 - This is also raised every time a looping animation finishes an loop.
 - **Event** is raised a user-defined event is detected.
	 - These are events you keyed in animations in Spine editor.
	 - This is useful for when you have to play sounds according to points the animation like footsteps. It can also be used to synchronize or signal non-Spine systems according to Spine animations, such as Unity particle systems or spawning separate effects, or even game logic such as timing when to fire bullets (if you really want to).


At the junction where an animation completes playback, and a queued animation will start, the events are raised in this order: `Complete`, `End`, `Start`.

**WARNING:**
> NEVER subscribe to `End` with a method that calls `SetAnimation`. Since `End` is raised when an animation is interrupted, and `SetAnimation` interrupts any existing animation, this will cause an infinite recursion of End->Handle>SetAnimation->End->Handle->SetAnimation, causing Unity to freeze until a stack overflow happens.
******* List AnimationState Events here.

----------

### Sample Code

Here is a sample MonoBehaviour that subscribes to AnimationState's callbacks. Read the comments to see what's going on.
```csharp
using UnityEngine;
using System.Collections;

// Add this to the same GameObject as your SkeletonAnimation
public class MySpineControllerThing : MonoBehaviour {

	// The [SpineEvent] attribute makes the inspector for this MonoBehaviour
	// draw the field as a dropdown list of existing event names in your SkeletonData.
	[SpineEvent] public string footstepEventName = "footstep"; 

	void Start () {
		var skeletonAnimation = GetComponent<SkeletonAnimation>();
		if (skeletonAnimation == null) return;	// told you to add this to SkeletonAnimation's GameObject.

		// This is how you subscribe via a declared method. It needs the correct signature.
		skeletonAnimation.state.Event += HandleEvent;
		
		// You can also use an anonymous delegate.
		skeletonAnimation.state.Start += delegate (Spine.AnimationState state, int trackIndex) {
			Debug.Log(string.Format("track {0} started a new animation.", trackIndex));
		};

		// ... or choose to ignore its parameters.
		skeletonAnimation.state.End += delegate {
			Debug.Log("An animation ended!");
		};
	}

	void HandleEvent (Spine.AnimationState state, int trackIndex, Spine.Event e) {
		// Play some sound if the event named "footstep" fired.
		if (e.Data.Name == footstepEventName) {			
			Debug.Log("Play a footstep sound!");
		}
	}
}
```