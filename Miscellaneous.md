#### spine-unity
The information here may change over time as the implementations within Spine-Unity get updated, improved or fixed.
This contains intermediate-level documentation. If you're just starting out, try the [Getting Started](/Getting-Started.md) document.

Documentation last updated for Spine-Unity for Spine 3.5.x (2017 May 15)
If this documentation contains mistakes or doesn't cover some questions, please feel to open an issue or post in the official [Spine-Unity forums](http://esotericsoftware.com/forum/viewforum.php?f=3). 

# Miscellaneous Topics and FAQ

### Unity says SkeletonAnimation (or another Spine class) doesn't exist.
The console will usually give you an error that looks like this if you have missing using directive on top of your code.
```
Assets/MyFirstSpineScript.cs(12,9): error CS0246: The type or namespace name `SkeletonAnimation' could not be found. Are you missing `Spine.Unity' using directive?
```

This is because Spine classes are defined in the `Spine` and `Spine.Unity` and namespaces.
To tell the compiler which namespaces to use to look for Spine classes, you need to add using directives.

In Unity code, your script should look like this on top:
```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

// These namespaces are for Spine classes.
using Spine;
using Spine.Unity;

public class MyFirstSpineScript : MonoBehaviour {
//....
}
```

To learn the basics of Unity scripting, see the official Unity Tutorials Scripting Section: https://unity3d.com/learn/tutorials/s/scripting

For more in-depth information on using namespaces in C#, see the official Microsoft documentation: https://docs.microsoft.com/en-us/dotnet/articles/csharp/programming-guide/namespaces/using-namespaces 
 

### How do I create a Spine GameObject from code?
Sometimes, it may make sense to create Spine GameObjects from code instead of creating prefabs.

To do that, use the `NewSkeletonAnimationGameObject` static function. This is actually a convenience method for `SkeletonRenderer.NewSpineGameObject<SkeletonAnimation>()`. Calling the latter will return the same thing.

```csharp
//SkeletonDataAsset skeletonDataAsset; // A reference to your asset
var newSkeletonAnimation = SkeletonAnimation.NewSkeletonAnimationGameObject(skeletonDataAsset);
```

If you want to create a SkeletonGraphic from code, you need to do use `SkeletonGraphic.NewSkeletonGraphicGameObject` instead.
```csharp
//SkeletonDataAsset skeletonDataAsset; // A reference to your asset
//Transform canvasTransform;
var newSkeletonGraphic = SkeletonGraphic.NewSkeletonGraphicGameObject(skeletonDataAsset, canvasTransform);
```

### What if I want to create the SkeletonDataAsset from code as well?
SkeletonDataAsset and AtlasAsset provide static functions to create new instances at runtime. This sometimes makes sense if you have on-demand assets where having the extra assets would just be unnecessary data.

See the code below:
```csharp
using UnityEngine;

using Spine;
using Spine.Unity;

public class RuntimeInstantiationSamples : MonoBehaviour {

	public TextAsset skeletonJson;
	public TextAsset atlasText;
	public Texture2D[] textures;
	public Material materialPropertySource;

	AtlasAsset runtimeAtlasAsset;
	SkeletonDataAsset runtimeSkeletonDataAsset;
	SkeletonAnimation runtimeSkeletonAnimation;

	void CreateRuntimeAssetsAndGameObject () {
		// 1. Create the AtlasAsset (needs atlas text asset and textures, and materials/shader);
		// 2. Create SkeletonDataAsset (needs json or binary asset file, and an AtlasAsset)
		// 3. Create SkeletonAnimation (needs a valid SkeletonDataAsset)

		runtimeAtlasAsset = AtlasAsset.CreateRuntimeInstance(atlasText, textures, materialPropertySource, true);		
		runtimeSkeletonDataAsset = SkeletonDataAsset.CreateRuntimeInstance(skeletonJson, runtimeAtlasAsset, true);		
		runtimeSkeletonAnimation = SkeletonAnimation.NewSkeletonAnimationGameObject(runtimeSkeletonDataAsset);
	}

}
```

### Can I make a specific Attachment render using a different Material?
Yes. After being loaded, each renderable attachment stores a reference to its AtlasRegion and the Material needed to render it.  

After your SkeletonData is loaded, you can change the Attachment's AtlasRegion properties to point to a different AtlasPage and Material.

`(MeshAttachment/RegionAttachment)renderableAttachment`.`(AtlasPage)RendererObject`.`(AtlasPage)page`.`(Material)RendererObject`

**Sample MonoBehaviour Code:**
```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

using Spine;
using Spine.Unity;
using Spine.Unity.Modules.AttachmentTools;

public class ChangeAttachmentMaterialExample : MonoBehaviour {

	public Material customMaterial;

	public SkeletonDataAsset skeletonDataAsset;
	[SpineSlot] public string slotName;
	[SpineSkin] public string skinName;
	[SpineAttachment(skinField = "skinNAme", slotField = "slotName")]
	public string attachmentName;

	void Start () {
		Attachment myAttachment = FindAttachment(skeletonDataAsset, skinName, slotName, attachmentName);
		//if (myAttachment == null) return;
		SetAttachmentRegionMaterial(myAttachment, customMaterial);
	}

	static public Attachment FindAttachment (SkeletonDataAsset skeletonDataAsset, string skinName, string slotName, string attachmentName) {
		if (skeletonDataAsset == null) throw new System.ArgumentNullException("skeletonDataAsset");

		var skeletonData = skeletonDataAsset.GetSkeletonData(true);
		//if (skeletonData == null) return null;
		var skin = skeletonData.FindSkin(skinName);
		//if (skin == null) return null;
		int slotIndex = skeletonData.FindSlotIndex(slotName);
		//if (slotIndex <= 0) return null;
		var attachment = skin.GetAttachment(slotIndex, attachmentName);

		return attachment;
	}

	// This method will affect all skeletons using this Attachment.
	static public void SetAttachmentRegionMaterial (Attachment attachment, Material material) {
		if (attachment == null) return;

		var atlasRegion = GetAttachmentAtlasRegion(attachment);
		if (atlasRegion == null) return;

		var atlasPage = material.ToSpineAtlasPage(); // extension from Spine.Unity.Modules.AttachmentTools
		atlasRegion.page = atlasPage;
	}

	static public AtlasRegion GetAttachmentAtlasRegion (Attachment attachment) {
		AtlasRegion atlasRegion = null;

		var regionAttachment = attachment as RegionAttachment;
		if (regionAttachment != null) {
			atlasRegion = (AtlasRegion)regionAttachment.RendererObject;
		} else {
			var meshAttachment = attachment as MeshAttachment;
			if (meshAttachment != null)
				atlasRegion = (AtlasRegion)meshAttachment.RendererObject;
		}

		return atlasRegion;
	}

}
```

### I want to change the attachment's material but not affect other Skeletons.
SkeletonAnimation.CustomSlotMaterials is a dictionary where you can assign special materials to slots.  
By giving it a slot reference and a material reference, it will use that material whenever it needs to render that slot.

```csharp
GetComponent<SkeletonAnimation>().CustomSlotMaterials.Add((Slot)mySlot, (Material)myMaterial);
``` 