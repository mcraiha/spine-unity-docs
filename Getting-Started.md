> **Licensing**
> 
> This Spine runtime may only be used for personal or internal use, typically to evaluate Spine before purchasing. If you would like to incorporate a Spine Runtime into your applications, distribute software containing a Spine Runtime, or modify a Spine Runtime, then you will need a valid [Spine license](https://esotericsoftware.com/spine-purchase). Please see the [Spine Runtimes Software License](https://github.com/EsotericSoftware/spine-runtimes/blob/master/LICENSE) for detailed information.
> The Spine Runtimes are developed with the intent to be used with data exported from Spine. By purchasing Spine, `Section 2` of the [Spine Software License](https://esotericsoftware.com/files/license.txt) grants the right to create and distribute derivative works of the Spine Runtimes.

# Getting Started with Spine-Unity

## Installing
Adding Spine-Unity into your Unity project:

1. Download and install [Unity](http://unity3d.com/get-unity) (There's a free version.)
1. Create an empty Unity project.
1. Download the latest spine-unity.unitypackage: http://esotericsoftware.com/spine-unity-download
1. Import the unitypackage (you can double-click on it and Unity will open it).
1. Go to the `Examples\Getting Started` folder in the Project panel. Open and examine those Unity Scene files in order. Make sure you read the text in the scene, check out the inspector and open the relevant sample scripts.

> The **Spine-Unity** runtime provides functionality to load, manipulate and render [Spine](http://esotericsoftware.com) skeletal animation data in [Unity](http://unity3d.com/) engine.
>  
> Spine-Unity works in Unity without the need for any other plugins. But it also works with [2D Toolkit](http://www.unikronsoftware.com/2dtoolkit/) and can render skeletons using TK2D's texture atlas system. To enable this, open Unity's `Preferences...` and under the `Spine` tab, you can enable TK2D.
>
> If you are not familiar with programming in C# and using Unity in general, we recommend watching the [official Unity Tutorials](http://unity3d.com/learn/tutorials) first. The [Interface Essentials](http://unity3d.com/learn/tutorials/topics/interface-essentials) and then [Scripting](http://unity3d.com/learn/tutorials/topics/interface-essentials) topics are a good place to start. Their Animation topic is not directly applicable to Spine-Unity so there's no need to learn that to know how to use Spine-Unity.
>
> Spine-Unity is built on top of Spine-C# ([spine-csharp](https://github.com/EsotericSoftware/spine-runtimes/tree/master/spine-csharp)).

## Bringing Your Spine Assets Into Your Project
### Exporting from Spine
![](/img/spine-runtimes-guide/spine-unity/spine-unity-export-settings.png)

1. After you have created your skeleton and animations, click on `Spine Menu`>`Export...` (`CTRL`+`E`). This opens the **Export window**.  
	![](/img/spine-runtimes-guide/spine-unity/spinemenu-export-s.png)    
1. Choose `JSON` on the upper-left of the Export window.  
 	![](/img/spine-runtimes-guide/spine-unity/exportwindow-json-s.png)  
1. Check the `Create atlas` checkbox. (Checking `Nonessential data`, `Pretty print` are also recommended for beginners).  
	![](/img/spine-runtimes-guide/spine-unity/exportwindow-createatlas-s.png)    
	1. Click on `Settings` beside the `Create atlas` checkbox. This opens the **Texture Packer Settings** window.
	1. On the lower-right, look for the textbox labeled `Atlas extension` and make sure it is set to `.atlas.txt`. (This is to work around Unity not accepting file extensions it doesn't recognize despite being plain text, but the Spine-Unity runtime can actually handle auto-renaming `.atlas` files. However, setting it to `.atlas.txt` minimizes problems and ambiguities with subsequent exports.)
	1. You're done with the Texture Packer Settings window. Click `OK` to close.  
	
1. In the **Export window**, pick an output folder. (Recommendation: Make a new empty folder. Make sure you can find it.)  
	![](/img/spine-runtimes-guide/spine-unity/exportwindow-json-outputfolder-s.png)  
1. Click `Export`.  
	![](/img/spine-runtimes-guide/spine-unity/exportwindow-exportbutton-s.png)
1. This will export three files:  
	- ![](/img/spine-runtimes-guide/spine-unity/three-exported-files.png)
	- a **.json** file that holds data of the skeleton.
	- a **.png** file which is the packed version of all your images in one texture.
	- a **.atlas.txt** file (libGDX atlas) that has data of where each image is in the packed texture.

For more information on ideal export settings, see the [Export Settings](Export-Settings.md) documentation.

> For __2D Toolkit__ users, Step 3 (packing a `.png` and `.atlas.txt`) is not necessary. Instead, you will have the appropriate field in your SkeletonDataAsset to assign a reference to `tk2dSpriteCollectionData`. To enable this, open Unity's `Preferences...` and under the `Spine` tab, you can enable TK2D.

### Importing into Unity

Step 1-3: Open both Unity and your folder. Drag the 3 files into Unity.  
![](/img/spine-runtimes-guide/spine-unity/explorer-to-unity-project.gif)

Step 4: Drag the SkeletonData Asset into scene view or the hierarchy panel.  
![](/img/spine-runtimes-guide/spine-unity/drag-and-drop-instantiate.gif)

Specifically:
1. Make sure your Unity project is open. It should already have a functioning Spine-Unity runtime in it.
1. Open the folder where you exported your 3 files. (**json**, **.atlas.txt** and **.png**)
1. Drag the 3 files (or the folder containing them) into Unity. Drop them in Unity's **Project panel**.
	- This will cause the Spine-Unity runtime to process them and auto-generate the necessary Unity assets.
	- You will notice 3 new files.
		- a **_Material** asset that holds references to the shader and **.png** texture.
		- an **_Atlas** asset that holds a reference to the material and the **.atlas.txt**.
		- a **_SkeletonData** asset that holds a reference to the **json** and the **_Atlas** asset.
4. Drag the **_SkeletonData** asset into the Scene View or the Hierarchy panel and choose `Instantiate (SkeletonAnimation)`.  

5. See the `Examples\Getting Started` sample scenes to learn more about Spine GameObjects.

> - **MANUAL ASSET SETUP** For advanced cases, you can create these three files yourself. The arrangement is noted in the bullet points of Step 3.
> - **CHANGING SHADERS** Using Spine-Unity's default shaders (`Spine/Skeleton` or `Spine/SkeletonLit`) requires textures that were saved with **Premultiplied Alpha**. This is the default setting in Spine's Texture Packer. Choosing other shaders may yield unexpected results and may require you to re-export the texture without premultiplied alpha. For more information on premultiply alpha, [check this forum topic](http://esotericsoftware.com/forum/Premultiply-Alpha-3132).
> - Older versions of Spine-Unity did not do drag-and-drop instantiation. If you have an older version, right-click on the **_SkeletonData** asset and choose `Spine > Instantiate (SkeletonAnimation)`. This will instantiate a new Spine GameObject.
> - Due to a limitation in Unity Editor APIs, dragging the asset into an empty hierarchy panel will not work. In this case, just drag the asset into Scene View.
> - **TEXTURE SIZES.** Unity scales overly large images down by default. The Spine-Unity runtime automatically sets atlas maximum sizes to 2048x2048. If your textures are larger than this, it will cause atlas coordinates to be incorrect. Make sure the import settings are set appropriately, or decrease the maximum page width and height in your Spine Texture Packer settings.
> - **TEXTURE ARTIFACTS FROM COMPRESSION.** Unity's 2D project defaults import new images added to the project with the Texture Type "Sprite". This can cause artifacts when using the `Spine/Skeleton` shader. To avoid these artifacts, make sure the Texture Type is set to "Texture" and . Spine-Unity's automatic import will attempt to apply these settings but in the process of updating your textures, these settings may be reverted.

### Using Skeleton Binary instead of json.
If you want to use Skeleton Binary files, choose Binary instead of JSON on the upper left of the Spine Export window when exporting.
![](/img/spine-runtimes-guide/spine-unity/spine-unity-export-skel-bytes.png)

- **EXTENSION:** Make sure the Extension is set to `.skel.bytes`. Otherwise, Unity will not recognize it as a binary file and Spine-Unity can't read it.
- All the steps mentioned above that apply to `.json` also apply to `.skel.bytes`. Spine-Unity will automatically recognize it and ingest it into "_SkeletonData".
- While the Spine Skeleton Binary format offers loading time improvements, the format is less stable across feature-update versions of the runtime. It's best to have a batch setup ready if you intend to update your runtime. For more info on command line exports, see: http://esotericsoftware.com/spine-export#Command-line


-----------------------------------------------------

# Updating Your Project's Spine-Unity Runtime
Major Spine editor updates require that you update your Spine-Unity runtime so it reads and interprets exported Spine data correctly.

- As with Unity updates, it is always recommended that you back up your whole Unity project before performing an update.
- Always check with your Lead Programmer and Technical Artist before updating your Spine runtime. Spine runtimes are source-available and designed to be user-modifiable based on varying project needs. Your project's spine runtime may have been modified by your programmer. In this case, updating to the latest runtime also requires reapplying those modifications to the new version of the runtime.
- You have three options for updating your Spine-Unity runtime. An in-place update with Unity's `Import Package` dialog is the recommended option. In rare, complicated cases, you may have to delete your older version of spine-unity and then import the unitypackage.   

## In-place Update (.unitypackage)
1. Download the latest spine-unity.unitypackage: http://esotericsoftware.com/files/runtimes/unity/spine-unity.unitypackage.
2. Import the .unitypackage into your project by double-clicking on the unitypackage file or dragging it into Unity editor.
3. The Import dialog will show which files are new or updated and will update them regardless of where you moved them in your project since you last imported.  
![](/img/spine-runtimes-guide/spine-unity/unitypackage-update.png)


>  - This functionality may not work correctly if your meta files were corrupted or replaced. In that case, you may have to delete the old version of the runtime before importing the unitypackage.
>  - Much older versions of the unitypackage had inconsistent meta files for spine-c#. You may have to delete the older spine-csharp folder when updating. This otherwise works correctly.
>  - Occasionally, some files may be removed and merged. Importing the .unitypackage will not delete those files so you may have to do that yourself. Check here for announcements regarding that: http://esotericsoftware.com/forum/Noteworthy-Spine-Unity-Topics-5924 

## Manual (.unitypackage)
1. Open an empty scene. (to be safe)
1. Delete the previous spine-unity and spine-csharp runtime from your Unity project.
1. Import the .unitypackage normally.

## Manual (zip from github)
1. Open an empty scene. (to be safe)
1. Download the zip file of the spine runtimes from the github repository: https://github.com/EsotericSoftware/spine-runtimes.
	1. Extract the files where you can find them. (You only need `spine-csharp` and `spine-unity`)
1. Look for the correct folders and replace them in your project.
	- These are the following folders you need
		- `spine-csharp/src` (this is named "spine-csharp" in the unitypackage)
		- `spine-unity/Assets/Gizmos`.
		- `spine-unity/Assets/spine-unity`
		- `spine-unity/Assets/Examples`(optional)

> - `Gizmos` is a [special folder](http://docs.unity3d.com/Manual/SpecialFolders.html) in Unity. It needs to be at the root of your assets folder to function correctly. (ie. `Assets/Gizmos`)
> - `spine-csharp` and `spine-unity` can be placed in any subfolder you want.
> - Sometimes, some files get moved around and you may get duplicate files. Unity will warn you when this happens. Make sure you delete the older versions whenever this happens.
> - Some files may have been removed and merged since the last version you used. Just copying the whole folder will not delete them. Check here for announcements regarding that: http://esotericsoftware.com/spine-unity-download/

## Sample Code
The basic animation-playing code will look like this:
```csharp
// Sample written for for Spine 3.6
void Start () {
	var skeletonAnimation = GetComponent<SkeletonAnimation>();

	skeletonAnimation.AnimationState.SetAnimation(0, "run", true);
	// The first parameter is the track number. You can have many animations playing on top of one another on different tracks at the same time.
	// The second parameter is the animation name. For optimization stages, non-string API also exists.
	// The third parameter is for making your animation loop.
}
```

For more sample code, the Spine-Unity unitypackage comes with a set of sample scenes and scripts. 
Once you have imported the latest unitypackage, find the `Examples\Getting Started` folder in the Project panel.

Open and examine those Unity Scene files in order.
Make sure you read the text in the scene, check out the inspector and open the relevant sample scripts.

Those will cover the basics of playing animations and posing characters.

For more detailed information on how to control animations, see the [Controlling Animation](Animation.md) documentation.

Visit the [Spine-Unity forum](http://esotericsoftware.com/forum/viewforum.php?f=12) for more information.
