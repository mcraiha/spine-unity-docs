Documentation last updated for Spine-Unity for Spine 3.5.x (2017 April 11)
If this documentation doesn't cover some questions, please feel free to post in the official [Spine-Unity forums](http://esotericsoftware.com/forum/viewforum.php?f=3).

# Spine-Unity Coroutine Yield Instructions
This module is a set of Coroutine yield instructions. (requires Unity 5.3 or higher)

They use the `IEnumerator` interface and not the [CustomYieldInstruction](https://docs.unity3d.com/ScriptReference/CustomYieldInstruction.html) class from 5.3.
Despite this, the changes to allow this functionality were only introduced in Unity's Coroutine system in 5.3. So it is incompatible with older Unity versions.


## How to Use

Firstly, you can find [basic tutorials](https://unity3d.com/learn/tutorials/topics/scripting/coroutines) and [documentation](https://docs.unity3d.com/Manual/Coroutines.html) on Unity coroutines to learn about them.

Assuming you understand how to use Unity Coroutines, the following demonstrates how to use Spine-Unity's custom yield instructions.

Like Unity's built-in yield instructions, these yield instructions pause execution of a coroutine until certain conditions are met. 


### WaitForSpineEvent
```csharp
yield return new WaitForSpineEvent(skeletonAnimation.state, "spawn bullet");
```
Listens for when a `Spine.AnimationState` fires a `Spine.Event` (a named user-defined event in Spine editor).


### WaitForSpineAnimationComplete
```csharp
var track = skeletonAnimation.state.SetAnimation(0, "talk", false);
yield return new WaitForSpineAnimationComplete(track);
```
Listens for when a `Spine.TrackEntry` finishes playing, when it fires its `Complete` event.

### Tips
- **Memory allocation optimization**: Like Unity's built-in yield instructions, instances of these spine-unity yield instructions can be reused to prevent extra memory allocations.
- **WaitForSpineEvent constructor overloads**: Listening for a Spine.Event with a name string is great for fast prototyping. But when it's time to optimize, you can cache and pass it the Spine.Event's `Spine.EventData` reference. WaitForSpineEvent has a few constructor overloads that will allow you to do this.


## A note on Execution Order

At their core, these yield instructions **yield null** to the coroutine system.
This means they check pausing and continuing execution AFTER MonoBehaviour Update and BEFORE MonoBehaviour LateUpdate.
See more information in Unity's documentation: https://docs.unity3d.com/Manual/ExecutionOrder.html
![](img\spine-runtimes-guide\spine-unity\yieldinstructions-execution-order.png) 

These yield instructions are tied to Spine.AnimationState's functionality and rely on it to raise the necessary events.
This means that when the target AnimationState is not active or updated, it cannot notify the yield instruction, so the yield instruction will continue to block execution of your coroutine. 

`SkeletonAnimation` updates its AnimationState `state` inside its MonoBehaviour Update call stack.
If you disable the SkeletonAnimation component or its parent or ancestor GameObjects, you cause SkeletonAnimation, and its AnimationState, to not be updated. This will cause yield instructions relying on these to stop execution as well.

Likewise, remember the `Update`>`Coroutine (yield null)`>`LateUpdate` order.
This means disabling a SkeletonAnimation or its ancestor GameObjects inside a coroutine will cause the SkeletonAnimation to not execute its LateUpdate, and thus not update its mesh.

This will cause the mesh to reflect the state of the skeleton right before the coroutine execution, and not after. Users have report this causing artifacts and blinking parts when you also modify the skeleton state within the Coroutine when you re-enable GameObjects. 