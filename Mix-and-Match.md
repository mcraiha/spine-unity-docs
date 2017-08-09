#### spine-unity
The information here may change over time as the implementations within Spine-Unity get updated, improved or fixed.
This contains intermediate-level documentation. If you're just starting out, try the [Getting Started](/Getting-Started.md) document.

Documentation last updated for Spine-Unity for Spine 3.6.x
If this documentation contains mistakes or doesn't cover some questions, please feel to open an issue or post in the official [Spine-Unity forums](http://esotericsoftware.com/forum/viewforum.php?f=3). 

# Equips, Customization, and Mix-and-Match
Spine has a **Skins** feature, which allows you to create image variations of a certain skeleton or character.
For example, Spine comes with an example skeleton with two skins. It allows you to use it as a female goblin or a male goblin. 

However, sometimes, you need to vary the skeleton part by part, rather than the whole thing at the same time.
This is what you would need for an *equip system*, or a *character customization system*. 

While Skins in the editor don't allow you to preview this yet, this is doable with Skins in Spine-Unity.

## Can Skins really help me?
The things you put in slots, such as *images* and *meshes*, are called **Attachments**.
A **Skin** is just a container for Attachments; and you can retrieve specific attachments by name & slot index (`string` and `int`).

For example, this is a representation of a "Red character" skin.
//TODO: Replace this with the json format?
```
Skin name: Red character

| SLOT#      NAME                ATTACHMENT OBJECT           |            
|--------|-------------------|-------------------------------|
|  4     |   "left hand"     |   red left glove              |
|  12    |   "right hand"    |   red right glove             |
|  11    |   "mouth"         |   mouth with fangs mesh       |
|  1     |   "cape"          |   red cape mesh               |
|  ...   |   ...             |   ...                         |
|  ...   |   ...             |   ...                         |
 ------------------------------------------------------------
```

At runtime and during animations, Spine.Skeleton uses the active and base skins as the source for what Attachments to use.
The values in the skin are used whenever an animation has attachment keys, or when you call `Skeleton.SetToSetupPose()` or `Slot.SetToSetupPose()`.

Normally, in Spine, you can define Skins as a fixed set of attachments to be used at runtime.
In Spine, you create a skin, add skin placeholders in slots, and then add attachments to skin placeholders.
This Skin can then be used at runtime, is part of SkeletonData, and is shared across all instances of skeletons using that SkeletonData.
![](/img/spine-runtimes-guide/spine-unity/mixandmatch-spinetree.png)
//TODO: Add a side-by-side with the json format for better clarity about how it translates to actual accessible data at runtime.

### Runtime Skins
Skins normally work as mentioned above, but skins can also be created and manipulated at runtime. We can call these "runtime skins".
By creating skins at runtime, you can set what attachments are in its collection programmatically.
```csharp
void UseRuntimeSkin () {
	Skin newSkin = new Skin("my new skin");
	newSkin.AddAttachment("armor", goldArmorAttachment); // Gold Armor
	newSkin.AddAttachment("hair", diamond hair); // Diamond Hair

	Skeleton skeleton = GetComponent<SkeletonAnimation>().Skeleton;	
	skeleton.SetSkin(newSkin);
	skeleton.SetSlotsToSetupPose();
}
```
With the above example, as long as you have "armor" and "hair" active, your skeleton will start using the Gold Armor and the Diamond Hair.

But where do you get Attachments to add to your custom skin?
That's up to you, and depending on what works best for your game's setup.

#### Option 1: Add variations in Spine
Adding variations in Spine and having the variations live inside the atlas is best for when you have a limited number of customizations. It will save you the potential repacking step or issues with using multiple texture atlases. 

**You can put source attachments plainly in slots.**
This can be handy if you have a very limited number of customizations, and don't want the hassle of managing Skins and Skin Placeholders in Spine.

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Spine;
using Spine.Unity;
using Spine.Unity.Modules.AttachmentTools; // A set of helpful extension methods for mix and match

public class MixAndMatchExample : MonoBehaviour {

	[SpineSlot] public string mouthSlotName = "mouth";
	[SpineAttachment(slotField:"mouthSlotName")] public string chosenMouthName = "lipstick mouth";

	[SpineSlot] public string gunSlotName = "gun";
	[SpineAttachment(slotField:"gunSlotName")] public string chosenGunName = "laser pistol";

	void Combine () {
		SkeletonAnimation skeletonAnimation = GetComponent<SkeletonAnimation>();
		Skeleton skeleton = skeletonAnimation.skeleton;

		int mouthSlot = skeleton.FindSlotIndex(mouthSlotName);
		int gunSlot = skeleton.FindSlotIndex(gunSlotName);

		// Get the original attachments
		var chosenMouth = skeleton.GetAttachment(mouthSlot, chosenMouthName);
		var chosenGun = skeleton.GetAttachment(gunSlot, chosenGunName);

		// Create a new skin, and add those attachments to it.
		Skin newSkin = new Skin("my new skin");
		newSkin.AddAttachment(mouthSlot, "mouth", chosenMouth);
		newSkin.AddAttachment(gunSlot, "gun", chosenGun);
		
		// Set and apply the Skin to the skeleton.
		skeleton.SetSkin(newSkin);
		skeleton.SetSlotsToSetupPose();
		skeletonAnimation.Update(0);
	}
}
```

**You can put them in Skins and Skin Placeholders.**
This can be helpful if you have a finite number of equips, and want to combine attachments in reasonable sets.
For example, you want to create skins for the boots: "leather boots", "iron boots", "power boots" so they can store both the left and right boots together.

If you group your equips or customization items into skins, you can create new skins by combining existing ones. 
```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Spine;
using Spine.Unity;
using Spine.Unity.Modules.AttachmentTools; // A set of helpful extension methods for mix and match.

public class MixAndMatchExample : MonoBehaviour {

	[SpineSkin] public string inspectedArmorName = "gold armor";
	[SpineSkin]	public string inspectedGlovesName = "red gloves";

	void Combine () {
		SkeletonAnimation skeletonAnimation = GetComponent<SkeletonAnimation>();
		Skeleton skeleton = skeletonAnimation.skeleton;
		SkeletonData skeletonData = skeleton.Data;

		// Get the source skins.
		var goldArmorSkin = skeletonData.FindSkin(inspectedArmorName);
		var redGlovesSkin = skeletonData.FindSkin(inspectedGlovesName);

		// Create a new skin, and append those skins to it.
		Skin myEquipsSkin = new Skin("my new skin");
		myEquipsSkin.Append(goldArmorSkin);
		myEquipsSkin.Append(redGlovesSkin);

		// Set and apply the Skin to the skeleton.
		skeleton.SetSkin(myEquipsSkin);
		skeleton.SetSlotsToSetupPose();
		skeletonAnimation.Update(0);
	}
}
```

#### Option 2: Use template attachments in Spine, and generate Attachments from UnityEngine.Sprites in Unity.
This can be handy if you have an arbitrarily large number of equips and customization, or ones that can come from anywhere- DLC, game updates, etc...

1. The first step is to **create a skeleton in Spine and use a set of images that are supposed to be templates for your eventual equips**.
Those template images will need to be the right height and width because they will form the bases of your actual equips.
If they can't all fit into one size, you may want to use more than one template. (eg. instead of just one "sword" template, you can have templates for "short sword", "wide sword", "dagger", "long sword", etc...)
For best results, you should animate with those template images in Spine. Create a skin called "template" and add your template swords and armor and whatever images into Skin Placeholders. (see: [Spine User Guide - Skins](http://esotericsoftware.com/spine-skins)). 
// TODO: Create and link to a sample skeleton with templated equips.

2. The next step is to **create those actual equip/customization images with the same height and width as your template images**.
You will import these individual (unpacked) images into your Unity project as Sprites.
**Requirements:**
You can pack these Sprites using Unity's packer, but they need to use **Full Rect** packing mode.
If you plan to use these Sprites with runtime repacking (see the next section), then you will need to use "Read/Write Enabled". 
   
3. Then you write your C# code that can combine use your template skin. This will vary as your equip or character customization system depends on how your game works.

> See the **Mix and Match** example scene and the **MixAndMatch.cs** sample script that comes with the Spine-Unity unitypackage for implementation sample.

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Spine;
using Spine.Unity;
using Spine.Unity.Modules.AttachmentTools; // A set of helpful extension methods for mix and match.

// TODO: sample code for creating Attachments from UnityEngine.Sprites, or using UnityEngine.Sprites as backing images to linked meshes
// Currently, there is sample code that comes with the Spine-Unity unitypackage. See Mix and Match sample scene and MixAndMatch.cs
```

### Runtime Repacking

> See the **Mix and Match** example scene and the **MixAndMatch.cs** sample script that comes with the Spine-Unity unitypackage for implementation sample.
//TODO: Mention useful [SpineAttributes] that can be used to find slots, template attachments, etc...

//TODO: Summary of what attachments are.
//TODO: Describe Skins basic function.
//TODO: Describe how Skins are just dictionaries, and how Skeleton pulls data from it.
//TODO: Describe how Skins can be generated at runtime.

//TODO: Describe some functions of AttachmentTools.
// - summary of how attachments relate to backing textures in Unity.
// - how you can clone existing/template attachments.
// - how you can replace the texture backing an attachment
// - how you can replace the texture backing an attachment with a Sprite
// - how you can create new attachments from Sprites.
// - required settings for this to work correctly
//   - Rect Packing. Read/Write Enabled. Match PMA parameter of calls. 


 

