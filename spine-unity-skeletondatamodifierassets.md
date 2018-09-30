http://esotericsoftware.com/spine-unity-skeletondatamodifierassets

Documentation last updated for Spine-Unity for Spine 3.7.x
If this documentation contains mistakes or doesn't cover some questions, please feel free to [open an issue](https://github.com/pharan/spine-unity-docs/issues) or post in the official [Spine-Unity forums](http://esotericsoftware.com/forum/viewforum.php?f=3).

----------

# SkeletonData Modifier Assets

**SkeletonData Modifier Assets** provide a way for users to add additional processing to SkeletonData after they are loaded from json or binary.

![](/img/spine-runtimes-guide/spine-unity/skeletondatamodifierasset-inspector-field.png)
SkeletonDataAsset's inspector shows a "Skeleton Data Modifier" list you can add assets to. In this screenshot, we have used "Default BlendModeMaterials" as an example asset added to the list.

## Included SkeletonDataModifierAssets

### BlendModeMaterials
**BlendModeMaterials** is an asset that holds references to materials that can be used to render attachments within slots that have the "Additive", "Multiply" and "Screen" blend modes assigned to them.

The Material references stored in BlendModeMaterials assets are used as templates to generate new Materials that use the appropriate texture needed by the loaded attachments. 

The spine-unity runtime comes packaged with a "Default BlendModeMaterials" asset. Using this included asset allows the attachments in slots with special blend modes to use the included default "Multiply" and "Screen" shaders: `Spine/Blend Modes/Skeleton PMA Multiply` and `Spine/Blend Modes/Skeleton PMA Screen`.

If you need to use different Materials or shaders or Materials with different settings, you can create new BlendModeMaterials assets using `Create>Spine>SkeletonData Modifiers>Blend Mode Materials`. Then assign your Material templates to your new BlendModeMaterials asset.

## Making Custom SkeletonDataModifierAssets
**SkeletonDataModifierAsset** is an abstract ScriptableObject class. You can create a new class using it as a base and implement its `void Apply (SkeletonData skeletonData)` method. Create an instance of your new class and add it to your SkeletonDataAsset as needed. `Apply(skeletonData)` will be called after the SkeletonData is loaded from its json or binary.