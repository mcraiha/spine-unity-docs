#### spine-unity
The information here may change over time as the implementations within Spine-Unity get updated, improved or fixed.
This contains intermediate-level documentation. If you're just starting out, try the [Getting Started](/Getting-Started.md) document.

Documentation last updated for Spine-Unity for Spine 3.6.x
If this documentation contains mistakes or doesn't cover some questions, please feel to comment below, open an issue or post in the official [Spine-Unity forums](http://esotericsoftware.com/forum/viewforum.php?f=3). 

# Attachments and Spine-Unity AttachmentTools
// TODO: Q: What are attachments? How do they work?
// TODO: Q: What do I need to understand so I can understand mix-and-match and use AttachmentTools with confidence?

### What are attachments?
In the Spine runtimes, **Attachments** are objects that can be placed in a slot to follow a bone; they generally follow a bone's transform (rotation, position, scale).

For the visible parts of a skeleton, there are 2 renderable attachment types:
1. **RegionAttachments**, rectangular image attachments that map to a rectangular region in an atlas, or any backing texture.
2. **MeshAttachments**, attachments that map their image/texture area to a deformable mesh.

> - Other non-rendered Attachment types exist such as **BoundingBoxAttachment**, **PathAttachment** and **PointAttachment**. They will be documented separately.
> - While **Skin Placeholders** exist in the Spine Editor, they are not actually Attachment objects at runtime. Skin Placeholders actually only correspond in the runtime to the key strings stored in Skin dictionaries. See the [Mix and Match documentation](/Mix-and-Match.md) or generic [Runtime Skins documentation](http://esotericsoftware.com/spine-runtime-skins) for more information.

### Where do I find Attachments?
Attachments loaded from a skeleton data file are deserialized and stored into Skin objects.
You can find these loaded skins and loaded Attachments in the SkeletonData. `Skeleton.Data.FindSkin(string)`

Even if you don't overtly define a Skin in Spine Editor, each skeleton data has a skin named "default". If you didn't put attachments in Skin Placeholders, they go into the **default skin**. `Skeleton.Data.DefaultSkin`

To get an Attachment from a skin, you need to know its skin key name and the index of the slot it belongs to.
```
SkeletonData skeletonData = skeleton.Data;
//Skin defaultSkin = skeletonData.DefaultSkin;
Skin foundSkin = skeletonData.FindSkin("goblin");
int slotIndex = skeletonData.FindSlotIndex("hand");
string keyName = "closed hand";
Attachment closedHandAttachment = foundSkin.GetAttachment(slotIndex, keyName);
```

For attachments you placed in Skin Placeholders, the name of the Skin Placeholder is the skin key name.
But for the default skin, the skin key name is the name of the attachment itself.

See the [Mix and Match documentation](/Mix-and-Match.md) or generic [Runtime Skins documentation](http://esotericsoftware.com/spine-runtime-skins) for more information.
   

// TODO: RegionAttachment and MeshAttachment
// TODO: Link to article about PMA. For now: http://esotericsoftware.com/forum/Premultiply-Alpha-3132

# AttachmentTools

AttachmentTools is a collection of utility methods and static classes that help manipulate Spine Attachment objects for Spine-Unity.

These are contained in the namespace:
```csharp
using Spine.Unity.Modules.AttachmentTools;
```

## Skin Utilities ##
**skeleton.UnshareSkin / skeleton.GetClonedSkin**  
Gets a duplicate of the skeleton's currently active skin, optionally including items from its default skin. This allows this new skin to be modified without affecting other skeleton instances.

If `unshareAttachments` or `cloneAttachments` is true, duplicates of the attachments will also be created. This allows the attachments to be modified without affecting other skeletons that use the original attachments.

In `skeleton.UnshareSkin`, this duplicate skin is assigned to the Skeleton's `.Skin`. If AnimationState is provided, the skeleton will be "refreshed" to use the new cloned skin.

**skin.GetClone**  
Creates a clone of a Skin. Attachments in this new skin are not clones.

**skin.CopyTo**  
Copies all the items in a Skin to a given destination Skin.

**skin.RemoveAttachment**  
Removes an item in a Skin.

**skin.GetRepackedSkin**  
Creates an populates a duplicate skin with cloned attachments that use a new packed texture. The new packed texture is comprised of all the regions used by the attachments of the original skin.
You can pass a shader, or a template material so that it can use the right material property and shader keyword values.


## AttachmentCloneExtensions ##
**renderableAttachment.GetRemappedClone**  
Creates a clone of the original attachment but mapped to a different texture.  
You can use a UnityEngine.Sprite, or an AtlasRegion (see `AtlasUtilities`) as the texture source.  
For MeshAttachments, this gives you the option to inherit the original mesh's animations.

**meshAttachment.GetLinkedMesh**  
Creates a new linked mesh based on the given MeshAttachment.  
You can use a Sprite or AtlasRegion (see `AtlasUtilities`) as the texture source.

## AtlasUtilities ##
These methods create a new AtlasRegion which attachments can use.

### ToAtlasRegion ###
**texture.ToAtlasRegion**  
Creates a new AtlasRegion based on a Texture2D. You can provide a shader or template Material to get shader properties and material properties from.  
This method creates its own AtlasPage to assign to the new AtlasRegion.

**texture.ToAtlasRegionPMAClone**  
Clones a Texture2D, applies PMA to it, and then creates an AtlasRegion that uses that texture.  
Some shaders (such as the Spine default shaders) require a premultiplied alpha texture by default. In those cases, use this method.  
This method creates its own AtlasPage to assign to the new AtlasRegion.

**material.ToSpineAtlasPage**  
Creates an AtlasPage based on a Material with a main texture.  
This AtlasPage can be reused when creating multiple new AtlasRegions as long as they share the same backing texture.

**sprite.ToAtlasRegion**  
Creates a new AtlasRegion based on a UnityEngine.Sprite.  
Passing a cached AtlasPage from `material.ToSpineAtlasPage` is recommended.

**sprite.ToAtlasRegionPMAClone**  
Creates a clone of the sprite as a new premultiplied alpha texture, then creates an AtlasRegion that uses that texture.
  
### Runtime Repacking ###
**AtlasUtilities.GetRepackedAttachments**  
Fills an outputAttachments list with new attachment objects based on the attachments in sourceAttachments, but mapped to a new single texture using the same material.  
This is useful for if you want an arbitrary set of renderable attachments that come from different texture sources to be repacked into a single texture. This minimizes engine draw calls which are especially important when you have many of the same skeleton being instantiated. This also minimizes dynamic sorting bugs in Unity.


### ToTexture/ToSprite ###
**atlasRegion.ToSprite**  
This creates a new UnityEngine Sprite based on a Spine.AtlasRegion.  
This is handy at runtime for when you want to render a region from the atlas using a SpriteRenderer.  
However, SpineAtlasAsset's inspector provides a button, which generates Sprite data from the Spine atlas, and have them available in edit mode just like normal Unity sprites. 

**atlasRegion.ToTexture**  
This creates a new Texture2D based on an AtlasRegion.  
This is mostly a utility method for longer more complex atlas-manipulation operations.

## AttachmentRegionExtensions ##
A set of extension methods for manipulating the source region for an Attachment.  
Note these do not apply when the skeleton uses TK2D.  

### renderableAttachment.GetRegion
Gets an AtlasRegion from a renderable attachment. This is otherwise done by getting its `.RendererObject` and casting it as an AtlasRegion.  
If the attachment is not renderable, the method will return null.

### renderableAttachment.SetRegion
Sets the AtlasRegion of a renderable attachment. This is useful when pulling AtlasRegions from other atlases, or creating new AtlasRegions from other sources.