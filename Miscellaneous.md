#### spine-unity
The information here may change over time as the implementations within Spine-Unity get updated, improved or fixed.
This contains intermediate-level documentation. If you're just starting out, try the [Getting Started](/Getting-Started.md) document.

Documentation last updated for Spine-Unity for Spine 3.5.x (2017 May 15)
If this documentation doesn't cover some questions, please feel free to post in the official [Spine-Unity forums](http://esotericsoftware.com/forum/viewforum.php?f=3). 

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

// These are for Spine classes.
using Spine;
using Spine.Unity;

public class MyFirstSpineScript : MonoBehaviour {
//....
}
```

To learn the basics of Unity scripting, see the official Unity Tutorials Scripting Section: https://unity3d.com/learn/tutorials/s/scripting

For more in-depth information on using namespaces in C#, see the official Microsoft documentation: https://docs.microsoft.com/en-us/dotnet/articles/csharp/programming-guide/namespaces/using-namespaces 
 

### How do I create a Spine GameObjects from code?
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

### What if I want to create the skeletonDataAsset from code as well?
SkeletonDataAsset and AtlasAsset provide static functions to create new instances at runtime. This sometimes makes sense if you have on-demand assets where having the extra assets would just be unnecessary data.

See the code below:
```
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
		// Create the AtlasAsset (needs atlas text asset and textures, and materials/shader);
		runtimeAtlasAsset = AtlasAsset.CreateRuntimeInstance(atlasText, textures, materialPropertySource, true);

		// Create SkeletonDataAsset (needs json or binary asset file, and an AtlasAsset) 
		runtimeSkeletonDataAsset = SkeletonDataAsset.CreateRuntimeInstance(skeletonJson, runtimeAtlasAsset, true);

		// Create SkeletonAnimation (needs a valid SkeletonDataAsset)
		runtimeSkeletonAnimation = SkeletonAnimation.NewSkeletonAnimationGameObject(runtimeSkeletonDataAsset);
	}

}
```

// TODO: Topic on http://esotericsoftware.com/forum/Sorting-Error-on-Start-8442