#### spine-unity
The information here may change over time as the implementations within Spine-Unity get updated, improved or fixed.
This contains intermediate-level documentation. If you're just starting out, try the [Getting Started](/Getting-Started.md) document.

Documentation last updated for Spine-Unity for Spine 3.5.x (2017 April 14)
If this documentation doesn't cover some questions, please feel free to post in the official [Spine-Unity forums](http://esotericsoftware.com/forum/viewforum.php?f=3). 


# Unity Shaders
[Shaders](https://en.wikipedia.org/wiki/Shader) are sets of instructions for the GPU regarding how to render objects and process data given to it.

In Unity, shaders are handled as [Shader Assets](https://docs.unity3d.com/Manual/class-Shader.html) (`.shader`  text files), and contain more information than what is typically considered a "shader" in other frameworks. 

Unity shaders are written in two languages:
// TODO: Illustration of Cg within ShaderLab context.

**ShaderLab**
The [Unity ShaderLab language](https://docs.unity3d.com/Manual/SL-Shader.html) defines:
- ["material properties" you can set in the inspector](https://docs.unity3d.com/Manual/SL-Properties.html) like texture assets, numbers and colors
- and [Subshaders](https://docs.unity3d.com/Manual/SL-SubShaderTags.html) and [Passes](https://docs.unity3d.com/Manual/SL-Pass.html) that define, for [different](https://docs.unity3d.com/Manual/SL-PassTags.html) [situations](https://docs.unity3d.com/Manual/SL-SubShaderTags.html):
  - graphics API calls ([blend modes](https://docs.unity3d.com/Manual/SL-Blend.html), [depth](https://en.wikipedia.org/wiki/Z-buffering) settings, [stencil](https://en.wikipedia.org/wiki/Stencil_buffer) settings, [backface culling](https://en.wikipedia.org/wiki/Back-face_culling), etc.)
  - other pre-built Unity graphics instructions ([GrabPass](https://docs.unity3d.com/Manual/SL-GrabPass.html), [UsePass](https://docs.unity3d.com/Manual/SL-UsePass.html))
  - and wraps the actual code for (Cg) shader programs

**Cg**
[Cg](http://http.developer.nvidia.com/CgTutorial/cg_tutorial_chapter01.html) is the industry standard, lower-level shader language that determines what actual operations and calculations the GPU needs to do on pixels/fragments, vertices, lights and texture samplers and the like.

Cg is the same code you would see used in other game engines and frameworks.

# Unity Materials


In Unity, a shader for a renderer is chosen by setting the [Materials](https://docs.unity3d.com/Manual/Materials.html) on that renderer.

// TODO: Illustration of a Material's two parts. (Shader + Material Properties like texture, sliders, etc).

Unity Materials combine a **Unity Shader** with a set of **Material Property values**. For 3D meshes, this commonly means that a Material defines whether a flat square looks like a rocky ground, or a brick wall, or metal panels. But for 2D meshes acting as sprites, a Material stores where to find the image data (the main texture), and how to render it (the shader, and the various material property values).

# Unity Shaders for Spine-Unity
Spine-Unity comes with a default shader— `Spine/Skeleton`— which is used by the auto-ingester by default when it generates a Material for your Spine texture atlas.

`Spine/Skeleton` is fairly short and typical, and has the following characteristics:
- Premultiply Alpha blending
- No depth buffer reading or writing, no lighting
- no backface culling
- no fog
- Uses vertex colors to tint the texture via multiply.
- Has a "ShadowCaster" pass so it can cast shadows in Unity's system.
- Material Properties:
	- Main Texture
	- Shadow alpha cutoff

Because it is a typical premultiply-alpha blended shader, you easily have the option of using other 2D shaders.
If you opt to use another shader, different characteristics could be accomodated for:

**Backface culling**
The only strict requirement for Spine is **disabling backface culling**, which is typical for 2D shaders.

> Most 3D shaders will have backface culling enabled. With Spine meshes, this will cause some triangles to become invisible when parts are scaled negatively or if you flip your skeleton direction.

**Depth and Lighting**
Not writing to the depth buffer is typical of an alpha-blended 2D sprite shader, as things are layered on top of each other primarily according to Unity's Renderer sorting scheme (see [Rendering](Rendering.md) for more details rather than the depth buffer. `Spine/Skeleton` shares this characteristic with Unity's own `Sprites/Default` shader.

> If you opt to use a shader that does use the depth buffer, you may want to add z-spacing to your SkeletonAnimation to prevent Z-Fighting, especially if you have lighting. You can do this by expanding the `Advanced...` foldout on your SkeletonAnimation's Inspector and sliding `Z Spacing` away from 0.
> Note that using the depth buffer may cause unwanted results around semitransparent areas, including antialiased edges.

**Vertex Colors**
You can set and animate colors in Spine. In Spine-Unity, this information is included in the generated mesh as **vertex colors**.
The renderer then uses the vertex colors as a multiply tint; simply, `texture color * vertex color` for all channels.

> Note that the SkeletonAnimation will have `PMA Vertex Colors` enabled by default. This makes it compatible with premultiply-alpha shaders. Leaving it enabled while using a straight alpha shader may cause color tints that are more transparent to become blacker than intended.
> Likewise, `PMA Vertex Colors` allows SkeletonAnimation to apply additive attachments by zeroing the submitted vertex color alpha for a certain slot.
> If you are not using a premultiply alpha shader, be sure to disable PMA Vertex Colors under the `Advanced...` foldout of SkeletonAnimation's inspector.

**Premultiply Alpha Blending**
Premultiply Alpha Blending ("PMA") is a type of alpha blending scheme that assumes that you have multiplied the alpha channel by the RGB values beforehand. PMA is the default but it not required for rendering Spine skeletons normally.

> Unity cannot detect PMA settings automatically. If you want to use a non-PMA shader (such as `Sprites/Default`), you must make sure the texture is saved without premultiply alpha, and you must make sure PMA Vertex Colors is disabled under the SkeletonAnimation's `Advanced...` foldout in the inspector.  


// TODO:
- Default assumptions and implementation (that it's nothing special)
	- Premultiplied Alpha
	- Depth
	- Vertex Colors
- You can use Unity's default Sprite shader
- Other options, requirements for it to work.

# Visual Shader Editors
If you are new to writing shaders, you can begin by using node-based visual shader editors. A few examples are [ShaderForge](https://www.assetstore.unity3d.com/en/#!/content/14147) and [Amplify Shader Editor](https://www.assetstore.unity3d.com/en/#!/content/68570) (still in beta at the time of writing).

These shader editors will generate Unity shader code text (a `.shader` file) that you can open, read, understand, edit and compare with changes you make in the nodes.

But remember that most of these editors are aimed at making 3D shaders. Not everything in shaders for 3D meshes will work for 2D sprites. You have to keep the characteristics mentioned above in mind for your shader to work correctly with Spine skeleton rendering and Unity sprites in general.

# Example


# Related Links
- [Rendering](https://github.com/pharan/spine-unity-docs/blob/master/Rendering.md)
  - 