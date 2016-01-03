
#### spine-unity
The information here may change over time as the implementations within Spine-Unity get updated, improved or fixed.

# How does rendering work?
In general, Spine’s systems are built to take advantage of the way modern game engines use 3D graphics. This is especially true for its advanced features like FFD (Free-form Deformation) and Weights.

It does this by interfacing with frameworks and engines “in 3D terms”; in terms of meshes and vertices, and texture mapping and UVs. So every image part is either defined by a polygon comprised of several triangles, or a rectangle made up of two triangles. This is not unusual even if the engine is mainly for 2D graphics.

![](http://i.imgur.com/XMsoUww.jpg)

The basic idea is this:
You have mesh made of triangles. This is shaped based on the Spine skeleton/model.
You have texture data. This comes from the atlas texture.
And you have shader programs or blending instructions.

You give all that to the game engine, the game engine gives it to the GPU, and then it renders. The textures are mapped to the mesh, and rendered based on the shader and blend mode.

![](http://i.imgur.com/URrFWUW.jpg)

Spine-Unity rendering uses Unity’s `MeshRenderer` and `MeshFilter` components, and Material assets. These are the same components that Unity uses to render 3D models.

The `SkeletonRenderer class` (base class of `SkeletonAnimation`) builds the mesh in its `LateUpdate` method. It generates arrays for vertices, vertex colors, triangle indexes and UVs, and puts all that inside a `UnityEngine.Mesh` object.
The `Mesh` goes into the `MeshFilter` component, which is used by the `MeshRenderer`.
The `MeshRenderer` renders the mesh at the appropriate point in [Unity’s game loop](http://docs.unity3d.com/Manual/ExecutionOrder.html).

`MeshFilter` and `MeshRenderer` have a [Retained Mode](https://en.wikipedia.org/wiki/Retained_mode) API (as opposed to an Immediate Mode API); that is, mesh and materials don't need to be passed to them every frame for the visibility to persist. This means that while `SkeletonRenderer`/`SkeletonAnimation` can update the mesh every frame, disabling the updates at runtime (by disabling `SkeletonAnimation` or `SkeletonRenderer`) will not destroy the visible mesh. 

#### Materials
Spine-Unity also uses Materials to store information about what texture, shader and material properties should be used. The Materials are assigned through the `AtlasAsset`.

The materials array of the `MeshRenderer` is managed by the `SkeletonRenderer` every frame depending on what `AtlasAssets` it needs to use. Modifying that array directly does not work like a typical Unity setup.

### Sprite Texture Rendering/Z Ordering
![](http://i.imgur.com/xqDt3Sn.jpg)

Spine typically* uses alpha-blending to render your Spine model parts.

When you draw, paint and export parts for your Spine project, you export them as PNG files which encode pixels as RGBA. Red, Green, Blue and Alpha. In the file, each pixel in the alpha channel can have a value from 0 to 255, just like the RGB channels, representing 256 levels of transparency/opacity for each pixel.

This transparency/semi-transparency, defined by the alpha channel, presents classical problems for depth/render order. These problems have many imperfect solutions and cause many of the weird 3D artifacts we see today even in our AAA games.

But in very standard fashion, ie. just like in most 2D alpha-blended sprite rendering cases including Unity’s own `Sprite`/`SpriteRenderer` system, Spine-Unity uses a shader that does not use the 3D Z-buffer nor alpha testing to determine what should be rendered in front and behind.

Within one mesh, it just renders things in the order defined by the order of the triangles of the mesh, drawing one thing on top of another. This order is what Spine uses to control Slot Draw Order.

Between meshes, Spine-Unity uses many of Unity’s render order systems to determine what sprite/mesh should be on top of which. Using the standard Spine-Unity setup, here is typical behavior for it:

- Mesh data is passed to Unity as a whole.
- It is rendered by the GPU triangle-by-triangle.
- Whole meshes are rendered in an order determined by many factors:
   - Distance from the camera. (Camera determines what kind of distance, planar or absolute.)
   - Renderer Sorting Layer/Sorting Order. (All UnityEngine.Renderers have these sorting properties.)
   - Shader’s ShaderLab Queue Tag (defaults to the “Transparent” queue like other sprites)
   - Material.renderQueue. (does nothing until you set it. It just overrides the shaderlab queue tag).
   - Camera order. (For multi-camera setups.)

For the most part, you won’t need to and shouldn’t touch Material.renderQueue and the shader’s queue tag.

The **Sorting Layer** and **Sorting Order** properties found in `SkeletonRenderer`/`SkeletonAnimation`’s Inspector actually modifies the `MeshRenderer`’s sorting layer and sorting order properties. Despite being hidden in MeshRenderer’s inspector, these properties are actually serialized normally.

*You can opt to use a different shader to get different kinds of blending, by changing the shader on the Material. Handling and working around the resulting blending behavior will likewise be up to you.

#### So can I ever render anything between parts of my Spine skeleton?
Sometimes, you need your character to ride a bicycle, or lift a rock hug an Ionic column. In Spine terms, this means you have to render some parts behind a thing, and some parts in front of a thing.

In Unity, meshes are rendered as a whole. So how do you get one to render behind and in front?

The answer to this question may change in the future.

But for now, you can use the Spine-Unity SkeletonUtility function called "Submesh Renderers". Its use will be covered separately.

But basically, what it does is allow your `SkeletonRenderer`’s mesh to be split by a slot (things above it and things below it). Those resulting pieces will then be rendered by their own MeshRenderers in their own `GameObject`s. Because they become separate renderers, you can set their sorting order individually.

#### I lowered the alpha to fade out my skeleton. Why are the overlaps showing?
This is how realtime mesh rendering works anywhere. The opacity is applied right when the triangles are drawn, not after.

There are probably many solutions to this. Some workarounds too.
You’ll find many good examples of workarounds from studying how various 2D games do it. Go out and play some of your favorites!

#### How does Spine-Unity determine how big my model will be?
Spine editor uses a 1 pixel:1 unit scale. Meaning, that if you just include an image into your skeleton without any rotation or scaling, 1 pixel in that image will be 1 unit tall and 1 unit wide in Spine.

In Unity, 1 unit is 1 meter. This is defined by Unity’s default physics values and constraints (both 2D and 3D). It’s generally not a good idea to use the 1 pixel : 1 unit ratio because of this. Instead, Unity’s own sprites default to scaling down by 1/100; meaning 100 pixels in your image will be 1 Unity unit long.

For convenience, when you import your Spine data into Unity, the scale is set at a default of 0.01 to match Unity’s sprites.

![](http://i.imgur.com/4YHiEUF.png)

If you want to set your skeleton’s base scale to something higher or lower, you can change this value in your Skeleton Data Asset’s inspector. This value is a multiplier used when your `SkeletonData` is loaded. Changing this value at runtime will not do anything after the `SkeletonData` has already been loaded.

If you want to dynamically change/animate your skeleton’s visual scale (like a balloon) in gameplay, you can do that by just changing `gameObject.transform.localScale`.

#### Flipping Horizontally or Vertically
Accessing `Skeleton.FlipX` and `Skeleton.FlipY` can allow you to flip the skeleton horizontally or vertically.
It’s usually a good idea to make all your skeletons face one direction so that your flipping logic can be consistent throughout your code. But if not, you can always add an extra `bool` to your controller and [negate your intended flip value with boolean logic](http://stackoverflow.com/questions/13983198/negate-a-boolean-based-on-another-boolean).

You may have learned to flip your 2D sprites in Unity setting a negative scale on the Transform, or rotating it along the y-axis by 180 degrees.
Both of these work for purely visual purposes. But they have their own side effects:

 - Nonuniform scale can cause a mesh to bypass Unity’s batching system. This means it will have its own (set of) draw calls for each instance. So it’s okay for your main character. It can be detrimental if you have nonuniform scale for dozens of skeletons.
 - Rotating causes normals to rotate with the mesh. For lighting 2D sprites, this means they’ll point in the wrong direction.
 - Rotating X or Y may also cause Unity’s 2D Colliders to behave unpredictably.
 - Negative scale can cause attached physics Colliders and some script logic to behave with unexpected results.
