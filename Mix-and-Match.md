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
A **Skin** is just a container for Attachments; and you can retrieve specific attachments by name (`string`).
For example, this is a theoretical "Red character" skin.
// TODO: visual representation of the internal skin dictionary.

At runtime and during animations, Spine.Skeleton uses the active and base skins as the source for what Attachments to use.
The values in the skin are used whenever an animation has attachment keys, or when you call `Skeleton.SetToSetupPose()` or `Slot.SetToSetupPose()`.

Normally, in Spine, you can define Skins as a fixed set of attachments to be used at runtime.
In Spine, you create a skin, add skin placeholders in slots, and then add attachments to skin placeholders.
This Skin can then be used at runtime, is part of SkeletonData, and is shared across all instances of skeletons using that SkeletonData.

// TODO: screenshot of skins and skin placeholders.

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

#### Option 1: Add variations in Spine.
**You can put source attachments plainly in slots.**
This can be handy if you have a very limited number of customizations, and don't want the hassle of extra code.

```csharp
// TODO: sample code for finding attachments in the default skin
```

**You can put them in Skins and Skin Placeholders.**
This can be helpful if you have a finite number of equips, and want to combine attachments in reasonable sets.
For example, you want to create skins for the boots: "leather boots", "iron boots", "power boots" so they can store both the left and right boots together.

A set of helpful extension methods can be found by scoping to the AttachmentTools namespace.
```
using Spine;
using Spine.Unity;
using Spine.Unity.Modules.AttachmentTools; // Add this on top of your .cs file.
```

If you group your equips or customization items into skins, you can create new skins by combining existing ones. 
```csharp
// TODO: sample code for combining skins.
```


#### Option 2: Use template attachments in Spine, and generate Attachments from UnityEngine.Sprites in Unity.
This can be handy if you have an arbitrarily large number of equips and customization, or ones that can come from anywhere- DLC, game updates, etc...

A set of helpful extension methods can be found by scoping to the AttachmentTools namespace.
```
using Spine;
using Spine.Unity;
using Spine.Unity.Modules.AttachmentTools; // Add this on top of your .cs file.
```

```
// TODO: sample code for creating Attachments from UnityEngine.Sprites, or using UnityEngine.Sprites as backing images to linked meshes
```

### Runtime Repacking

> See the **Mix and Match** example scene and the **MixAndMatch.cs** sample script that comes with the Spine-Unity unitypackage for implementation sample.


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


 

