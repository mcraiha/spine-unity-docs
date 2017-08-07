#### spine-unity
The information here may change over time as the implementations within Spine-Unity get updated, improved or fixed.
This may contain intermediate-level documentation. If you're just starting out, try the [Getting Started](/Getting-Started.md) document.

Documentation last updated for Spine-Unity for Spine 3.6.x
If this documentation contains mistakes or doesn't cover some questions, please feel to open an issue or post in the official [Spine-Unity forums](http://esotericsoftware.com/forum/viewforum.php?f=3). 

# SkeletonGraphic and Unity UI

The Unity UI (UnityEngine.UI) uses a Canvas+CanvasRenderer system to sort and manage its renderable objects. Built-in renderable UI components such as `Text`, `Image`, and `RawImage`, rely on `CanvasRenderer` to function correctly.

> You can learn more about Unity UI here:
> https://docs.unity3d.com/Manual/UISystem.html
> https://unity3d.com/learn/tutorials/topics/user-interface-ui

Specific canvas settings cause UI to be scoped and drawn in a certain way or in a certain order and, by default, rendered without any connection to a Camera object.

![](/img/spine-runtimes-guide/spine-unity/skeletongraphic-unity-ui-specific.png)
Putting objects like MeshRenderers (default cubes), or SpriteRenderers (Sprites), under a UI will not render in a Canvas. `SkeletonAnimation` will behave in the same way.

Because of this, Spine-Unity comes with **SkeletonGraphic**, a version of `SkeletonAnimation` that builds on the Unity UI System and likewise relies on CanvasRenderer. SkeletonGraphic particularly subclasses `UnityEngine.UI.MaskableGraphic`.

![](/img/spine-runtimes-guide/spine-unity/skeletongraphic-inspector.png)
SkeletonGraphic Inspector and Scene View

Using MaskableGraphic allows SkeletonGraphic to live in Unity UI, be masked and rendered correctly and in an expected order.

But because it uses MaskableGraphic, it also has MaskableGraphic's **single-material limitation**.
This means Skeletons used in UI are best packed with single-texture/single-page atlases, rather than multi-page atlases.
// TODO: Link to SkeletonGraphicMulti implementation.

> TIP: You can make a SkeletonGraphic's RectTransform fit its current pose's dimensions by choosing "Match RectTransform with Mesh Bounds" in its inspector context menu.


## BoneFollowerGraphic
BoneFollowerGraphic is the BoneFollower analogue for SkeletonGraphic: it will cause a Transform to follow a bone's position and rotation around. This component is most performant if the Transform is an immediate child of the SkeletonGraphic Transform.
Because of the difference in systems, the normal BoneFollower will not work with SkeletonGraphic. Instead, you need to use BoneFollowerGraphic.

You can add a BoneFollowerGraphic GameObject by choosing it in the context menu, just like SkeletonAnimation.
![](/img/spine-runtimes-guide/spine-unity/skeletongraphic-bonefollower.png)