# Spine-Unity #

**[Forums](http://esotericsoftware.com/forum/viewforum.php?f=12)** | **[Documentation](http://esotericsoftware.com/spine-unity-documentation)** | **[GitHub](http://esotericsoftware.com/git/spine-runtimes/tree/spine-unity)**

For the latest changes, see : http://esotericsoftware.com/spine-unity-download

----------

# Spine-Unity Changelog History #

## Fixed Bugs ##
* (2017 Jun 28) Fixed SkeletonRagdoll2D not working in some cases.
* (2017 Jun 28) Disabled hierarchy icons in play mode to skip unnecessarily heavy processing when GameObjects where rearranged/created/destroyed rapidly.
* (2017 Jun 28) Fixed a bug where the SkeletonAnimator inspector would not work.
* (2017 Jun 26) Fixed a bug where SkeletonDataAssets would report missing texture sources when TK2D is enabled.
* (2017 Jun 24) Fixed a bug where AnimationState was dip-compensating between tracks.
* (2017 Jun 22) Fixed a bug where TK2D had missing AttachmentLoader interface methods.
* (2017 Jun 22) Fixed a bug where unweighted meshes with DeformTimelines that have no keys at the start of the timeline would shrink during mixing.
* (2017 Jun 9) Fixed a bug where SkeletonAnimator would throw a KeyNotFoundException when its controller used a non-dummy AnimationClip.
* (2017 Jun 7) Handled an obsolete Handles.DotCap call for Unity 5.6 and higher.
* (2017 Jun 1) Fixed a bug where the "Spine/Skeleton Tint Black" shader ignored the texture alpha channel.
* (2017 May 29) Fixed a bug where SpineEditorUtilities would attempt to check and import external empty or un-findable text assets.
* (2017 May 6) Fixed a bug where example scripts used code for older Unity instead of newer.
* (2017 Apr 26) Fixed a bug where cloning VertexAttachments was copying the wrong array.
* (2017 Apr 14) Fixed a bug where cloning MeshAttachments and BoundingBoxAttachments returned the original rather than the clone.
* (2017 Mar 30) Fixed a bug where cloning MeshAttachments would throw exceptions if data was exported without nonessential data, such as when repacking skins.
* (2017 Mar 30) Fixed a bug where repacking skins multiple times would cause rotated source atlas regions to have the incorrect rotation.
* (2017 Mar 16) Fixed a bug where non-Spine jsons were reporting version errors.
* (2017 Mar 16) Fixed a bug where older versions of Unity wouldn't compile because of the use of DelayedTextField.
* (2017 Mar 16) Fixed a possible leak when destroying SkeletonRenderers (if Unity requires manual destroy on Mesh objects).
* (2017 Feb 18) Fixed a bug where the SkeletonDataAsset inspector would not load despite having a complete and correct TK2D Sprite Collection. This did not affect builds.
* (2017 Feb 18) Fixed a bug where AnimationState wouldn't raise events that were queued from within event callbacks.
* (2017 Feb 15) Fixed a bug causing a BoudingBoxFollower's generated PolygonCollider2Ds to not be cleared correctly.


## Important Notes when Upgrading ##
* (2017 October 16) The old AttachmentTools static class called `SpriteAtlasAttachmentExtensions` was renamed AtlasUtilities. Any methods that were called as statics rather than extension methods must now refer to that new name. Extension method calls will not need any code changes.
* (2017 June 11) SkeletonGraphic now allows you to override the mainTexture with a repacked texture using SkeletonGraphic.OverrideTexture.
* (2017 May 6) Sprite Shaders: When updating shaders from any source, especially ones that use cginc files, it may be necessary to restart Unity or right-click then choose `Reimport All` in the project panel so the cginc files are properly reimported. https://forum.unity3d.com/threads/is-there-a-way-to-force-shaders-to-update-when-cginc-files-are-updated.336041/
* (2017 May 6) Sprite Shaders: Some files from the older version of the Sprite shaders were removed. delete the whole `spine-unity/Modules/Shaders/Sprite` subfolder before upgrading to ensure no old unused files are left behind.
* (2017 May 6) Sprite Shaders: Updated to the latest source version. Some serialization may have changed. If you used these shaders, do go over your Material assets and make sure they have the correct settings: https://github.com/traggett/UnitySpriteShaders
* (2017 Mar 30) You now have the option to disable multi-mixing to get rid of the dipping issue. This can be set with the `AnimationState.MultipleMixing` bool property. In 3.5, it is enabled by default so updating your runtime will not result in unexpected changes in animation behavior. You can change the code so AnimationState sets the value to false by default to remove unwanted dipping at the cost of multiple mixing. This default may change in 3.6.
* (2017 Jan 24) Based on Unity Technology recommendations, whenever SkeletonUtility (or BoundingBoxFollower) generates PolygonColliders based on Spine Bounding Boxes, it will also add a Kinematic Rigidbody2D to that object. This means each of those GameObjects are their own immutable box2D body and cost significantly less computationally to move/animate.
* (2016 Nov 28) Some UnityEngine.Sprite.ToRegion methods have been moved to consolidate code. You will need to add `using Spine.Unity.Modules.AttachmentTools` to your using directives in your code.
* (2016 Oct 23) **Spine 3.5** The context menu for creating scene objects from the Project view/asset menu is disabled by default because it is not the recommended place for scene objects. You can now instantiate your skeletons by dragging and dropping the SkeletonData asset into the scene view, similar to Unity Sprites. See this topic for more details: http://esotericsoftware.com/forum/New-Feature-Drag-and-Drop-to-instantiate-6789
* (2016 Jul 13) **Spine 3.4** There were no base runtime feature and serialization changes between 3.3 and 3.4. There is no need to re-export your binaries and jsons if they were exported with Spine 3.3.
* (2016 Apr) The sample implementation module called `VertexHelperSpineMeshGenerator` has been removed. You can find the code here: [VertexHelperSpineMeshGenerator.cs](https://gist.github.com/pharan/f1e28ada3e2b7107b435200f37008c0f)
* (2016 Apr) The .unitypackage was exported with Text Serialization to improve the example scenes compatibility with version controlled projects (and Unity Collaborate).
* (2016 Apr) **Spine 3.3** involved some refactoring. If you are upgrading your runtime, it is recommended that you delete your spine-csharp folder before importing the new files. Some redundant/unused files like `FfdAttachment.cs` and `WeightedMeshAttachment.cs` may be left over and cause errors. (Because none of the `spine-csharp` files are Unity components, deleting or replacing it will not break any of your scenes or prefabs.)
* (2016 Apr) Most older jsons (especially with meshes) are not compatible with Spine 3.3 and higher. You will need to re-export your skeletons even if you used json. As usual, skeleton binaries are incompatible across Spine editor versions.
* (2016 Mar) Spine classes were moved into the `Spine.Unity` namespace. You will need to update your scripts that use Spine classes. Add `using Spine.Unity` and/or `using Spine.Unity.Modules`. Read more here: http://esotericsoftware.com/forum/Spine-Unity-namespaces-6025
* (2016 Mar) Old sample binary files were removed.
* (2016 Mar) SharpJson.cs was merged into Json.cs. Please delete SharpJson.cs when upgrading.