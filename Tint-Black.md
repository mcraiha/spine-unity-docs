#### spine-unity
The information here may change over time as the implementations within Spine-Unity get updated, improved or fixed.
This contains intermediate-level documentation. If you're just starting out, try the [Getting Started](/Getting-Started.md) document.

Documentation last updated for Spine-Unity for Spine 3.6.x
If this documentation contains mistakes or doesn't cover some questions, please feel to comment below, open an issue or post in the official [Spine-Unity forums](http://esotericsoftware.com/forum/viewforum.php?f=3). 

![](/img/spine-runtimes-guide/spine-unity/tint-black-demo.gif)  

# Tint Black in Spine-Unity
Tint Black is a color blending feature where a rendered color's darkness level is replaced by a specified color. As the rendered color gets darker, it moves towards the Black Tint color instead of towards black.

This has a number of features, including allowing a tinted texture to be brighter than its original color, compared to ordinary multiply color blending; and allowing the texture to be filled with a solid color by setting the multiply tint and the black tint to the same color.

This color blending is implemented in shader code.

## Per-Material Black Tinting
Spine-Unity comes with a shader called `Spine/Skeleton Tint`.  
This shader has two Material properties for color:

"Tint Color" (`"_Color"`)  
"Black Point" (`"_Black"`)  

![](/img/spine-runtimes-guide/spine-unity/skeleton-tint-shader-color-properties.png)  

By setting "_Color" and "_Black" through the Material APIs ([Material](https://docs.unity3d.com/ScriptReference/Material.html), or [MaterialPropertyBlock](https://docs.unity3d.com/ScriptReference/MaterialPropertyBlock.html)) you can control the Renderer's multiply tint and black tint.  

> Note: The alpha channel on the black tint color does nothing.

## Spine-Animated Per-Slot Black Tinting
The Spine editor features an optional Tint Black option for slots, allowing you to set the black tint for each slot.  
See: http://esotericsoftware.com/spine-attachments#Tint-black

For this to work, your skeleton's setup must have the following setup:
1.  Enable Tint Black in the SkeletonAnimation's inspector under "Advanced...":  
![](/img/spine-runtimes-guide/spine-unity/skeletonanimation-inspector-tintblack.png)
2.  Set the Material's shader to `Spine/Skeleton Tint Black`  
![](/img/spine-runtimes-guide/spine-unity/skeleton-tint-black-material-shader.png)

In the Unity Editor, the changes may not all be applied together so you may need to enter and exit play mode to force a new mesh to be generated according to the settings.

When the setup is incorrect, the mesh may be somewhat rainbow-colored, as in the case where Tint Black was not enabled on the component. In this case, make sure the setup is correct and you try to enter and exit play mode to refresh the meshes.

### SkeletonGraphic tint Black
For **SkeletonGraphic**, steps are slightly different.
1. You still need to enable Tint Black in the SkeletonGraphic's inspector under `Advanced...`.
2. SkeletonGraphic's shader is `Spine/SkeletonGraphic Tint Black`. Since UI materials can be recycled (as textures are set via MaterialPropertyBlocks), just set the Material of the SkeletonGraphic to the bundled `SkeletonGraphicTintBlack` material.
![](/img/spine-runtimes-guide/spine-unity/skeletongraphictintblack-material.png)
3. Select the Canvas and enable the Additional Shader Channels for UV0 and UV1, labeled "TexCoord1" and "TexCoord2"    
![](/img/spine-runtimes-guide/spine-unity/unity-canvas-texcoord1-texcoord2.png)

## Additional Notes
In Unity, Slot colors are pushed to the Unity mesh via its vertex color buffer. UnityEngine's Mesh type doesn't allow arbitrary extra buffers so the extra color data from the second color (black tint) is pushed via the UV0 and UV1 buffers.

The effect of this is that meshes that use per-slot black tint also need the memory for the extra buffers, and need also need to have those extra buffers be updated every frame. This may have a noticeable performance cost when applied to large, complex skeletons or large numbers of skeletons. 