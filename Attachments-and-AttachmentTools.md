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

## AttachmentTools
// TODO:
// Order:
// using Spine.Unity.Modules.AttachmentTools;
// Skin Operations
// GetRemappedClone
// Unity Sprite methods
// AtlasRegion methods