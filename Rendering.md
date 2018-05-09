#### spine-unity
The information here may change over time as the implementations within Spine-Unity get updated, improved or fixed.
This contains intermediate-level documentation. If you're just starting out, try the [Getting Started](/Getting-Started.md) document.

Documentation last updated for Spine-Unity for Spine 3.6.x
If this documentation contains mistakes or doesn't cover some questions, please feel to open an issue or post in the official [Spine-Unity forums](http://esotericsoftware.com/forum/viewforum.php?f=3). 


# Rendering
In general, Spine’s systems are built to take advantage of the way modern game engines use 3D graphics. This is especially true for its advanced features like Mesh Deformation and Weights.

It does this by interfacing with frameworks and engines “in 3D terms”; in terms of meshes and vertices, and texture mapping and UVs. So every image part is either defined by a polygon comprised of several triangles, or a rectangle made up of two triangles. This is not unusual even if the engine is mainly for 2D graphics.

![](/img/spine-runtimes-guide/spine-unity/render_spineunity_p1.jpg)

The basic idea is this:
You have mesh made of triangles. This is shaped based on the Spine skeleton/model.
You have texture data. This comes from the atlas texture.
And you have shader programs or blending instructions.

You give all that to the game engine, the game engine gives it to the GPU, and then it renders. The textures are mapped to the mesh, and rendered based on the shader and blend mode.

Because of this system, some things that you may assume to be simple to do in programs like Photoshop or Flash may not be as straightforward to do in Unity or Spine (or 3D game engines in general).

![](/img/spine-runtimes-guide/spine-unity/render_spineunity_p2.jpg)

Spine-Unity rendering uses Unity’s `MeshRenderer` and `MeshFilter` components, and Material assets. These are the same components that Unity uses to render 3D models.

The `SkeletonRenderer class` (base class of `SkeletonAnimation`) builds the mesh in its `LateUpdate` method. It generates arrays for vertices, vertex colors, triangle indexes and UVs, and puts all that inside a `UnityEngine.Mesh` object.
The `Mesh` goes into the `MeshFilter` component, which is used by the `MeshRenderer`.
The `MeshRenderer` renders the mesh at the appropriate point in [Unity’s game loop](http://docs.unity3d.com/Manual/ExecutionOrder.html).

`MeshFilter` and `MeshRenderer` have a [Retained Mode](https://en.wikipedia.org/wiki/Retained_mode) API (as opposed to an Immediate Mode API); that is, mesh and materials don't need to be passed to them every frame for the visibility to persist. This means that while `SkeletonRenderer`/`SkeletonAnimation` can update the mesh every frame, disabling the updates at runtime (by disabling `SkeletonAnimation` or `SkeletonRenderer`) will not destroy the visible mesh.

> **Advanced use: Frame skipping per skeleton**
> This also means that, after the initial mesh generation step, you can control and skip mesh updates to implement specific frameskipping and staggered-update behavior. This can be used to minimize updates on less important game elements, or achieve a stylistic frame-by-frame look to the animations.

## Materials
Spine-Unity also uses Materials to store information about what texture, shader and material properties should be used. The Materials are assigned through the `AtlasAsset`.

The materials array of the `MeshRenderer` is managed by the `SkeletonRenderer` every frame depending on what `AtlasAssets` it needs to use. Modifying that array directly does not work like a typical Unity setup.

### So many Materials
You may notice that your MeshRenderer has a lot of Materials— particularly, more materials than you actually set in your `AtlasAsset`.

This is not unusual if you have more than one atlas page. The renderer's material array is not supposed to reflect the order and number of items in your AtlasAsset. SkeletonRenderer lays out a Material array based on what materials it needs to render Spine Attachments.

If all attachments share one material, SkeletonRenderer only puts one Material in MeshRenderer.

If some attachments require material A and some material B, a Material array is laid out according to the order the materials are needed. This is based on the order the attachments are drawn and which attachments can be found in which Material's texture.

// TODO: Make illustrate this in an image.
If the order goes:
> Attachment from A
> 
> Attachment from A
> 
> Attachment from B
> 
> Attachment from A

The resulting material array will be:
> Material A (for the first two)
> 
> Material B (for the third)
> 
> Material A (for the fourth)

In other words, the more the attachments alternate between coming from A and coming from B, the more materials there will be in the material array, and each item in the material array signifies it needing to switch materials.

The Dragon example demonstrates this: 
![](/img/spine-runtimes-guide/spine-unity/render_spineunity_alternatingmaterials.png)

More materials in this array means more [draw calls](http://docs.unity3d.com/Manual/DrawCallBatching.html), which can adversely affect performance if the same skeleton is instantiated several times. If you only have one of this skeleton, it is probably not a cause for significant slowdown.

To learn more about how to arrange atlas regions in your Spine atlases, see this page: [Spine Texture Packer: Folder Structure](http://esotericsoftware.com/spine-texture-packer#Folder-structure)

### Setting Material Properties Per Instance
![](/img/spine-runtimes-guide/spine-unity/materialpropertyblock-demo.gif)  
See the "Per Instance Material Properties" sample scene in the unitypackage for example component code.  

SkeletonRenderer uses Materials based on the Material reference stored in the Attachments at load time.
This means using MeshRenderer.material and changing values on the Material object you get will not work.

Normally, `Renderer.material` property generates a copy just for that renderer. But in SkeletonAnimation, that material will immediately be overwritten by SkeletonRenderer's render code.

On the other hand, `Renderer.sharedMaterial` will modify the original Material. And if you spawn more than one Spine GameObject that uses that Material, the changes you apply to it will be applied to all instances.

This is where Unity's [Renderer.SetPropertyBlock](http://docs.unity3d.com/ScriptReference/Renderer.SetPropertyBlock.html) is useful. `SkeletonRenderer`/`SkeletonAnimation` uses a `MeshRenderer`. Setting that MeshRenderer's material property block will allow you to change property values *just for that renderer* while not affecting others.

**To set a specific renderer's material property values**, you need to have a `UnityEngine.MaterialPropertyBlock` object (you can create it using `new`, and cache the value in your class). Then call the renderer's `SetPropertyBlock` method with your MaterialPropertyBlock as the argument.

```csharp
MaterialPropertyBlock mpb = new MaterialPropertyBlock();
mpb.SetColor("_FillColor", Color.red); // "_FillColor" is a named property on a hypothetical shader. 
GetComponent<MeshRenderer>().SetPropertyBlock(mpb);
```

When using MaterialPropertyBlock, keep in mind that while it is a class/reference type, **you will need to pass it to SetPropertyBlock every time you make changes to it** in order to see the changes applied to the renderer.

**To remove the renderer's material property values**, you need to clear the MaterialPropertyBlock and then call SetPropertyBlock using the empty MaterialPropertyBlock.
```csharp
MaterialPropertyBlock mpb = this.cachedMaterialPropertyBlock; // assuming you had cached a MaterialPropertyBlock with values in it.
mpb.Clear(); // Clear the property values from the MaterialPropertyBlock
GetComponent<Renderer>().SetPropertyBlock(mpb); // set the renderer property block using the empty MaterialPropertyBlock.
```

#### CustomMaterialOverride and CustomSlotMaterial
SkeletonRenderer allows you to override Materials on specific slots or override the resulting materials.

To replace an an original material with your new material at runtime for that instance of SkeletonRenderer, you would use SkeletonRenderer.CustomMaterialOverride. For example:
```
skeletonAnimation.CustomMaterialOverride[originalMaterial] = newMaterial; // to enable the replacement.
skeletonAnimation.CustomMaterialOverride.Remove(originalMaterial); // to disable that replacement.
``` 
With this code, when the SkeletonAnimation would have used originalMaterial, it would use newMaterial instead.


To use a replacement Material just on a specific slot, you would use SkeletonRenderer.CustomSlotMaterial. For example:
```
skeletonAnimation.CustomSlotMaterial[slot] = newMaterial; // to enable the replacement.
skeletonAnimation.CustomSlotMaterial.Remove(slot); // to disable that replacement.
```


> **Notes on optimization**
> - using Renderer.SetPropertyBlock allows renderers with the same material to be batched correctly even if their material properties are changed by different MaterialPropertyBlocks.
> - You do need to call `SetPropertyBlock` whenever you change or add a property value to your MaterialPropertyBlock. But you can keep that MaterialPropertyBlock as part of your class so you don't have to keep instantiating a new one whenever you want to change a property.
> - When you need to set a property frequently, you can use the static method: `Shader.PropertyToID(string)` to cache the int ID of that property instead of using the string overload of MaterialPropertyBlock's setters.  

## Sprite Texture Rendering/Z Ordering
![](/img/spine-runtimes-guide/spine-unity/render_spineunity_sortingface.jpg)

Spine typically* uses alpha-blending to render your Spine model parts.

When you draw, paint and export parts for your Spine project, you export them as PNG files which encode pixels as RGBA. Red, Green, Blue and Alpha. In the file, each pixel in the alpha channel can have a value from 0 to 255, just like the RGB channels, representing 256 levels of transparency/opacity for each pixel.

This transparency/semi-transparency, defined by the alpha channel, presents classical problems for depth/render order. These problems have many imperfect solutions and cause many of the weird 3D artifacts we see today even in our AAA games.

But in very standard fashion, ie. just like in most 2D alpha-blended sprite rendering cases including Unity’s own `Sprite`/`SpriteRenderer` system, Spine-Unity uses a shader that does not use the 3D Z-buffer nor alpha testing to determine what should be rendered in front and behind.

Within one mesh, it just renders things in the order defined by the order of the triangles of the mesh, drawing one thing on top of another. This order is what Spine uses to control Slot Draw Order.

Between meshes, Spine-Unity uses many of Unity’s render order systems to determine what sprite/mesh should be on top of which. Using the standard Spine-Unity setup, here is typical behavior for it:

- Mesh data is passed to Unity as a whole.
- It is rendered by the GPU triangle-by-triangle.
- Whole meshes are rendered in an order determined by many factors:
   - Distance from the camera. (Camera determines [what kind of distance; planar or perspective](http://docs.unity3d.com/ScriptReference/Camera-transparencySortMode.html).)
   - Renderer Sorting Layer/Sorting Order. ([All UnityEngine.Renderers have these sorting properties](https://unity3d.com/learn/tutorials/modules/beginner/2d/sorting-layers).)
   - [Sorting Group](https://docs.unity3d.com/Manual/SortingGroup.html) components on the current GameObject or on any of the parents GameObjects.
   - Shader’s ShaderLab Queue Tag (defaults to the “Transparent” queue like other sprites)
   - Material.renderQueue. (does nothing until you set it. It just overrides the shaderlab queue tag).
   - [Camera depth](http://docs.unity3d.com/ScriptReference/Camera-depth.html). (For multi-camera setups.)

For the most part, you won’t need to and shouldn’t touch Material.renderQueue and the shader’s queue tag.

*You can opt to use a different shader to get different kinds of blending, by changing the shader on the Material. Handling and working around the resulting blending behavior will likewise be up to you.

### Sorting Layer and Order
The **Sorting Layer** and **Sorting Order** properties found in `SkeletonRenderer`/`SkeletonAnimation`’s Inspector actually modifies the `MeshRenderer`’s [sorting layer](http://docs.unity3d.com/ScriptReference/Renderer-sortingLayerID.html) and [sorting order](http://docs.unity3d.com/ScriptReference/Renderer-sortingOrder.html) properties.

Despite being hidden in MeshRenderer’s inspector, these properties are actually serialized/stored normally as part of `MeshRenderer` and not part of SkeletonRenderer.

So while you can see them on SkeletonAnimation's Inspector, you can access them through code like this:
```csharp
GetComponent<MeshRenderer>().sortingOrder = 1; // Change the Order in Layer to 1. 
```

### Sorting via Camera Distance
If you keep all your renderers in the same sorting layer and order, they will be subject to the other sorting schemes mentioned above.

If the queue tag is also the same (if you're using the same material and shader), then you should be able to control sorting of your Spine GameObjects according to camera distance.

Also remember that [perspective cameras have sorting modes](http://docs.unity3d.com/ScriptReference/Camera-transparencySortMode.html). 

### So can I ever render anything between parts of my Spine skeleton?
Sometimes, you need your character to ride a bicycle, or lift a rock hug an Ionic column. In Spine terms, this means you have to render some parts behind a thing, and some parts in front of a thing.

In Unity, meshes are rendered as a whole. So how do you get one to render behind and in front?

Please see [SkeletonRenderSeparator](https://github.com/pharan/spine-unity-docs/blob/master/SkeletonRenderSeparator.md)

// TODO: Short Section on [Sorting Groups](https://docs.unity3d.com/560/Documentation/Manual/SortingGroup.html) (Unity 5.6)

### I lowered the alpha to fade out my skeleton. Why are the overlaps showing?
This is how realtime mesh rendering works anywhere. The opacity is applied right when the triangles are drawn, not after.

To apply the lowered alpha to the whole skeleton after it has been rendered at full alpha, you would need to temporarily render it to a different target first, then draw the contents of that target to the screen. The implementation for this is highly dependent on your setup.

There are many solutions and workarounds for this.
You’ll find many good and creative examples of workarounds from studying how various 2D games do it. Go out and play some of your favorites!

## How does Spine-Unity determine how big my model will be?
Spine editor uses a 1 pixel:1 unit scale. Meaning, that if you just include an image into your skeleton without any rotation or scaling, 1 pixel in that image will be 1 unit tall and 1 unit wide in Spine.

In Unity, 1 unit is 1 meter. This is defined by Unity’s default physics values and constraints (both 2D and 3D). It’s generally not a good idea to use the 1 pixel : 1 unit ratio because of this. Instead, Unity’s own sprites default to scaling down by 1/100; meaning 100 pixels in your image will be 1 Unity unit long.

For convenience, when you import your Spine data into Unity, the scale is set at a default of 0.01 to match Unity’s sprites.

![](/img/spine-runtimes-guide/spine-unity/skeletonDataScale.png)

If you want to set your skeleton’s base scale to something higher or lower, you can change this value in your Skeleton Data Asset’s inspector. This value is a multiplier used when your `SkeletonData` is loaded. Changing this value at runtime will not do anything after the `SkeletonData` has already been loaded.

If you want to dynamically change/animate your skeleton’s visual scale (like a balloon) in gameplay, you can do that by just changing `gameObject.transform.localScale`.

## Flipping Horizontally or Vertically
Accessing `Skeleton.FlipX` and `Skeleton.FlipY` can allow you to flip the skeleton horizontally or vertically.
It’s usually a good idea to make all your skeletons face one direction so that your flipping logic can be consistent throughout your code. But if not, you can always add an extra `bool` to your controller and [negate your intended flip value with boolean logic](http://stackoverflow.com/questions/13983198/negate-a-boolean-based-on-another-boolean).

You may have learned to flip your 2D sprites in Unity setting a negative scale on the Transform, or rotating it along the y-axis by 180 degrees.
Both of these work for purely visual purposes. But they have their own side effects to keep in mind:

 - Nonuniform scale can cause a mesh to bypass Unity’s batching system. This means it will have its own (set of) draw calls for each instance. So it’s okay for your main character. It can be detrimental if you have nonuniform scale for dozens of skeletons.
 - Rotating causes normals to rotate with the mesh. For lighting 2D sprites, this means they’ll point in the wrong direction unless your shader uses viewspace normals.
 - Rotating X or Y may cause Unity’s 2D Colliders to behave unpredictably.
 - Negative scale can cause attached physics Colliders and some script logic to behave with unexpected results.

