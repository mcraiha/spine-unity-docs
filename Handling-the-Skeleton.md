#### spine-unity
The information here may change over time as the implementations within Spine-Unity get updated, improved or fixed.
This contains intermediate-level documentation. If you're just starting out, try the [Getting Started](/Getting-Started.md) document.

Documentation last updated for Spine-Unity for Spine 3.6.x
If this documentation contains mistakes or doesn't cover some questions, please feel to open an issue or post in the official [Spine-Unity forums](http://esotericsoftware.com/forum/viewforum.php?f=3). 

# Interacting with Skeleton and Bone properties through code

## Skeleton and Bones
Every `SkeletonAnimation` has a `Skeleton` property. This holds the state of a Skeleton including all its Constraints and Bones. You can access these using code that looks like the following:

// TODO: Diagram of what objects "store" what.

```csharp
using UnityEngine;
using Spine;
using Spine.Unity;

public class SkeletonHandlingSample : MonoBehaviour {

	void Start () {
		var skeletonAnimation = GetComponent<SkeletonAnimation>();
		Skeleton skeleton = skeletonAnimation.Skeleton; // stores skeleton state

		Bone torsoBone = skeleton.FindBone("torso"); // stores bone state
		BoneData torsoBoneData = torsoBone.Data; // stores bone setup pose info

		Slot torsoSlot = skeleton.FindSlot("torso"); // stores slot state
		SlotData torsoSlotData = torsoSlot.Data // stores slot setup pose info

		// Constraints
		
		
	}

}
``` 

// TODO: Summarize and demonstrate using Spine Attributes that help with picking skeleton object names.
// CONSIDER: Move this to its own section? This will also be used by mix and match.
Spine-Unity comes with some property drawers and property drawer attributes that allow you to pick Bones, Slots, or other Skeleton objects. 

```csharp
using UnityEngine;
using Spine;
using Spine.Unity;

public class SkeletonHandlingSample : MonoBehaviour {
	
	[SpineBone]
	public string myBone;

	[SpineSlot]
	public string mySlot;

	[SpineAttachment]
	public string myAttachment;

	void Start () {
		var skeletonAnimation = GetComponent<SkeletonAnimation>();
		Skeleton skeleton = skeletonAnimation.Skeleton; // stores skeleton state

		Bone torsoBone = skeleton.FindBone(myBone); // stores bone state
		BoneData torsoBoneData = torsoBone.Data; // stores bone setup pose info

		Slot torsoSlot = skeleton.FindSlot(mySlot); // stores slot state
		SlotData torsoSlotData = torsoSlot.Data // stores slot setup pose info
	}

}
```

## Lifecycle
![](/img/spine-runtimes-guide/spine-unity/spine-unity-skeletonanimation-updates.png)  
In the SkeletonAnimation component, AnimationState holds the state of all currently playing and queued animations.
Every `Update`, the AnimationState is updated so that the animations progress forward in time. And then the new frame is applied to the Skeleton as a new pose.

Your scripts may run before or after SkeletonAnimation's Update.  
If your code takes Skeleton or bone values before SkeletonAnimation's Update, your code will read values from the previous frame instead of the current one.

To make sure you get the current values, you can either change Script Execution Order to run AFTER SkeletonAnimation's Update or use one of SkeletonAnimation's various event callbacks.

### SkeletonAnimation Update Callbacks
- `SkeletonAnimation.UpdateLocal` is raised after the animations for the frame is updated and applied to the skeleton's local values. Use this if you need to read or modify bone local values. 
- `SkeletonAnimation.UpdateComplete` is raised after world values are calculated for all bones in the Skeleton. SkeletonAnimation makes no further operations in Update after this. Use this if you only need to read bone world values. Those values may still change if any of your scripts modify them after SkeletonAnimation's Update.
- `SkeletonAnimation.UpdateWorld` is raised after the world values are calculated for all the bones in the Skeleton. If you subscribe to this event, it will call `skeleton.UpdateWorldTransform` a second time. Depending on the complexity of your skeleton or what you are doing, this may be unnecessary, or wasteful. Use this event if you need to modify bone local values based on bone world values. This is useful for implementing custom constraints in Unity code.
 

For more information on on Unity's MonoBehaviour Lifecycle, see: https://docs.unity3d.com/Manual/ExecutionOrder.html

