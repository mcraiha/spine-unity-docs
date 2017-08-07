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
// TODO: Illustration of SkeletonAnimation's life cycle and MonoBehaviour methods relative to Unity's Order of Execution/game loop.

Documentation on Unity's MonoBehaviour Lifecycle. https://docs.unity3d.com/Manual/ExecutionOrder.html

