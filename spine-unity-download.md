http://esotericsoftware.com/spine-unity-download

# Spine-Unity #

**[Forums](http://esotericsoftware.com/forum/viewforum.php?f=12)** | **[Documentation](http://esotericsoftware.com/spine-unity-docs)** | **[GitHub](http://esotericsoftware.com/git/spine-runtimes/tree/spine-unity)**

----------

## Getting Started ##
[Getting Started Documentation](http://esotericsoftware.com/spine-unity-gettingstarted)

## Download ##
**THE LATEST SPINE-UNITY RUNTIME UNITYPACKAGE:**  
**Spine-Unity 3.6 runtime unitypackage:**  
DOWNLOAD : http://esotericsoftware.com/files/runtimes/unity/spine-unity-3_6-2018-07-19.unitypackage  
Compatible with Spine 3.6.x and Unity 5.6-2018.2 (Last updated: UTC - 2018 Jul 19)  
For the changelog, see [3.6/CHANGELOG.md](https://github.com/EsotericSoftware/spine-runtimes/blob/3.6/CHANGELOG.md#c-1)

**Spine-Unity 3.7-beta runtime unitypackage:**  
DOWNLOAD : http://esotericsoftware.com/files/runtimes/unity/spine-unity-3_7-2018-07-19-beta.unitypackage  
Compatible with Spine 3.7.x and Unity 5.6-2018.2 (Last updated: UTC - 2018 Jul 19)  
For the changelog, see [3.7-beta/CHANGELOG.md](https://github.com/EsotericSoftware/spine-runtimes/blob/3.7-beta/CHANGELOG.md#c-1)

----------
## Installing in a new project
1. Create an empty Unity project.
1. Import the spine-unity.unitypackage you downloaded. (double-clicking on the unitypackage should open it in Unity).

If you are just getting started, there are more detailed steps in the [documentation section on Getting Started](http://esotericsoftware.com/spine-unity#Getting-Started).<br> For more options, see the [documentation section on Installing](http://esotericsoftware.com/spine-unity#Installing).

## Updating an existing project

- As with Unity updates, it is always recommended that you back up your whole Unity project before performing an update.
- Always check with your Lead Programmer and Technical Artist before updating your Spine runtime. Spine runtimes are source-available and designed to be user-modifiable based on varying project needs. Your project's Spine runtime may have been modified by your programmer. In this case, updating to the latest runtime also requires reapplying those modifications to the new version of the runtime.

### In-place Update (.unitypackage)
1. Open your Unity project.
2. Import the .unitypackage into your existing project by double-clicking on the unitypackage file or dragging it into Unity editor Project view.
3. The Import dialog will show which files are updated and will update them regardless of where you moved them in your project since you last imported.

>  - This functionality may not work correctly if your meta files were corrupted or replaced. In that case, you may have to do delete the old version of the runtime before importing the unitypackage.
>  - Much older versions of the unitypackage had inconsistent meta files for spine-c#. You may have to delete the older spine-csharp folder when updating. This otherwise works correctly.
>  - Occasionally, major refactoring involves removing or merging some files. Importing the .unitypackage will not delete those leftover files when you update so you may have to do that yourself. Check here for announcements regarding changed/leftover files: http://esotericsoftware.com/forum/Noteworthy-Spine-Unity-Topics-5924

For more options, see the [spine-unity documentation page section on updating](http://esotericsoftware.com/spine-unity#Updating-Your-Projects-SpineUnity-Runtime).

----------

Known Issues:
- You cannot drag SkeletonData Assets into the Hierarchy view if it is empty. You will need to drag them into Scene View, or create an empty GameObject before dragging into Hierarchy view.

Unity Limitations:
- Unity's physics components do not support dynamically assigned vertices so they cannot be used to mirror bone-weighted and deformed BoundingBoxAttachments. However, BoundingBoxAttachments' internal vertex data at runtime is still being deform correctly and can be used to perform hit detection with your own code.
- UnityEngine.UI.MaskableGraphic does not support using more than one texture, so uses of SkeletonGraphic are limited to skeletons with only one atlas page/texture.
At the cost of masking, an experimental version of SkeletonGraphic which splits it into multiple UI objects can be found here: http://esotericsoftware.com/files/runtimes/unity/spine-unity-experimental-SkeletonGraphicMultiObject-3_6-2018-04-19.unitypackage
- Spine animations are not imported as Unity.AnimationClip objects. Unity uses a different curve scheme and the set of features do not overlap well with Spine's features. Instead, Spine-Unity animates using core C# Spine.Animation objects which retain all of Spine's animation features. This means a few conveniences may be absent, such as playing back animations in edit mode using the Animation panel.
- Unity does not recognize arbitrary file types, so atlases need to be exported as `.atlas.txt`. Likewise, binary files need to be exported as `.skel.bytes`. For more information on importing Spine assets, see the [documentation section on Getting Started](http://esotericsoftware.com/spine-unity#Getting-Started)
- Unity has had a long-standing issue with multi-material/multi-submesh meshes, sorting and dynamic batching. If you have many duplicates of a skeleton that uses multiple materials, Unity's dynamic batching system will attempt to batch the similar materials but it breaks sorting within skeletons in the process of batching submeshes with similar materials. To work around this Unity bug, add a Sorting Group component to your Spine GameObject. However, it is still better for performance if your skeleton uses only one texture and material.

----------


###Compatibility###

- Runtimes cannot load exported binary file versions that are newer or older than the version it supports.
- Json exports are more stable and have better chances of being compatible with future versions, but may still break.
- If you want to avoid incompatibility issues with a set runtime version, you can choose a Spine editor version that matches that runtime specifically. This can be done in Spine's `Settings...` window.<br>
For more information, see [this forum topic](http://esotericsoftware.com/forum/Spine-editor-and-runtime-version-management-6534). 

**Fixed Bugs**
* (2018 Jul 17) Fixed a typo in the conditional compile for Unity 2017.2 and above.
* (2018 Jul 2) Fixed a bug where AtlasUtilities.GetRepackedAttachments would have misaligned arrays when your attachment list had non-renderable Attachments.
* (2018 Jun 7) Fixed an issue where a Spine-generated mesh would leak if you force-reinitialized a SkeletonRenderer/SkeletonAnimation/SkeletonAnimator.
* (2018 May 25) Fixed an issue where the first frame of a (Unity Timeline) SpineAnimationStateClip playing would not visually reflect the new animation despite having no mixing.
* (2018 May 24) Fixed a bug where skeleton previews were not working, or animation previews were playing too fast.
* (2018 May 10) Fixed a bug where clipping would function incorrectly with SkeletonRenderSeparator.
* (2018 Apr 12) Fix case where Tint Black was used and no attachments were rendered causing a NullReferenceException.
* (2018 Mar 27) Fix SpineAttributes not correctly detecting sibling or base properties, especially when array fields are involved.
* (2018 Mar 23) Fixed new tangents not being solved every frame.
* (2018 Mar 14) Fix DrawBoundingBox.
* (2018 Feb 8) Fix "none" item for SpineSlot attribute.
* (2018 Jan 12) [AnimationState] Fixed incorrect delay being calculated for non-loop animations.
* (2018 Jan 9) Fixed previous BoneFollower fix not applied to world-space Transform mode.
* (2018 Jan 8) Fixed BoneFollower adding redundant rotation when bone is locally flipped and BoneFollower is following local scale.
* (2018 Jan 6) Fixed SkeletonAnimator not caching clip info in existing lists in Unity 5.6. It caused memory allocation per frame.
* (2018 Jan 5) Fixed SkeletonAnimator's MecanimTranslator not clearing completely when calling Initialize.
* (2017 Dec 24) We've temporarily removed the included asmdef file until we can add and arrange the editor assembly definitions so that building for the target platform works.
* (2017 Dec 22) Fixed the BoundingBoxFollower inspector not correctly initializing the BoneFollower it creates through the Add BoneFollower button.
* (2017 Dec 15) Fixed a Clipping attachment bug where clipping would not stop if the end slot was empty or non-renderable (eg, had a BoundingBoxAttachment, PathAttachment or PointAttachment).
* (2017 Dec 7) Fixed handling of play-pause in the SkeletonDataAsset inspector.
* (2017 Nov 13) Fixed the Drag-and-Drop cursor tooltip "Create Spine GameObject" not showing up.
* (2017 Oct 31) Fixed a bug where repacking the same attachments twice would cause exceptions.
* (2017 Oct 31) Fixed a bug where AnimationState couldn't reset certain bone positions to setup pose when combining mix durations with zero-duration/immediate animation transitions.
* (2017 Oct 31) Fixed a bug where the default SkeletonGraphic shader wouldn't correctly apply the tint alpha.
* (2017 Oct 21) Fixed typo in SpriteVertexLighting.cginc. Choose "reimport" to refresh the shader compilation from cginc.
* (2017 Oct 20) Fixed Unity 2017.2 playModeStateChange API change.
* (2017 Oct 20) Fixed SkeletonGraphic TintBlack shader not using the correct vertex color alpha source.
* (2017 Oct 11) Fixed a bug where Deform animation timelines would blend from zeroes.
* (2017 Sep 30) Fixed a bug where the Tint and Tint Black shaders were ignoring the overall opacity/alpha from vertex colors and the Material property.
* (2017 Sep 28) Fixed a bug where PathConstraints were calculating incorrect spacing between bones when constraining multiple bones.
* (2017 Aug 23) Fixed a bug where destroying a GameObject with SlotBlendModes component would cause a NullReferenceException.
* (2017 Aug 23) Fixed AnimationState not mixing out properly in certain combinations of zero and nonzero mix durations.
* (2017 Jul 21) Fixed Triangulator causing incorrect ClippingAttachment clipping behavior.
* (2017 Jul 11) Fixed sample AtlasRegionAttacher script not checking for null returned atlas.
* (2017 Jul 11) Fixed SkeletonJson not handling empty end slots on ClippingAttachments.
* (2017 Jul 05) Fixed BoundingBoxFollower not taking the active skin into account.
* [See history...](http://esotericsoftware.com/spine-unity-history#Fixed-Bugs)


**Important notes when upgrading**
* (2017 October 16) The old AttachmentTools static class called `SpriteAtlasAttachmentExtensions` was renamed AtlasUtilities. Any methods that were called as statics rather than extension methods must now refer to that new name. Extension method calls will not need any code changes.
* (2017 June 11) SkeletonGraphic now allows you to override the mainTexture with a repacked texture using SkeletonGraphic.OverrideTexture.
* (2017 May 6) Sprite Shaders: When updating shaders from any source, especially ones that use cginc files, it may be necessary to restart Unity or right-click then choose `Reimport All` in the project panel so the cginc files are properly reimported. https://forum.unity3d.com/threads/is-there-a-way-to-force-shaders-to-update-when-cginc-files-are-updated.336041/
* (2017 May 6) Sprite Shaders: Some files from the older version of the Sprite shaders were removed. delete the whole `spine-unity/Modules/Shaders/Sprite` subfolder before upgrading to ensure no old unused files are left behind.
* (2017 May 6) Sprite Shaders: Updated to the latest source version. Some serialization may have changed. If you used these shaders, do go over your Material assets and make sure they have the correct settings: https://github.com/traggett/UnitySpriteShaders
* (2017 Mar 30) You now have the option to disable multi-mixing to get rid of the dipping issue. This can be set with the `AnimationState.MultipleMixing` bool property. In 3.5, it is enabled by default so updating your runtime will not result in unexpected changes in animation behavior. You can change the code so AnimationState sets the value to false by default to remove unwanted dipping at the cost of multiple mixing. This default may change in 3.6.
* (2017 Jan 24) Based on Unity Technology recommendations, whenever SkeletonUtility (or BoundingBoxFollower) generates PolygonColliders based on Spine Bounding Boxes, it will also add a Kinematic Rigidbody2D to that object. This means each of those GameObjects are their own immutable box2D body and cost significantly less computationally to move/animate.
* [See history...](http://esotericsoftware.com/spine-unity-history#Important-Notes-when-Upgrading)

----------
## Older Versions
**Spine-Unity 3.0 for Unity 4.6:**  
http://esotericsoftware.com/files/runtimes/unity/spine-unity-3_0_Unity-4_6.unitypackage  
(Last updated: UTC - 2017 May 9)

**Spine-Unity 3.5 for Unity 5.4-2017.1:**  
http://esotericsoftware.com/files/runtimes/unity/spine-unity-3_5_Unity-5_4.unitypackage  
(Last updated: UTC - 2017 June 18)

**Spine-Unity 3.6 for Unity 5.4:**  
http://esotericsoftware.com/files/runtimes/unity/spine-unity-3_6_Unity-5_4-2017-06-20.unitypackage  
(Last updated: UTC - 2017 June 20)