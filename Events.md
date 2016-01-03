#### spine-unity
The information here may change over time as the implementations within Spine-Unity get updated, improved or fixed.

## Coding for Spine Events & AnimationState callbacks

(((ILLUSTRATION HERE FOR THE POINTS OF ANIMATION PLAYBACK)))

**Spine.AnimationState** provides animation callbacks in the form of [C# events](https://msdn.microsoft.com/en-us/library/awbftdfh.aspx). You can use these to handle some basic points of animation playback.

Spine.AnimationState raises events:
 - **Start** is raised when an animation starts playing,
	 - 
 - **End** is raised an animation is cleared or interrupted,
	 - 
 - **Complete** is raised an animation completes its full duration,
	 - 
 - **Event** is raised a user-defined event is detected.
	 - 

**WARNING:** NEVER subscribe to `End` with a method that calls `SetAnimation`. Since `End` is raised when an animation is interrupted, and `SetAnimation` interrupts any existing animation, this will cause an infinite recursion of End->Handle>SetAnimation->End->Handle->SetAnimation, causing Unity to freeze until a stack overflow happens.
******* List AnimationState Events here.

((( Sample code for how to subscribe to events. Include anonymous method and named method. )))

// work in progress...  