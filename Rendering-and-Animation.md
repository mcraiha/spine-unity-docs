
#### spine-unity
The information here may change over time as the implementations within Spine-Unity get updated, improved or fixed.
This contains intermediate-level documentation. If you're just starting out, try the [Getting Started](/Getting%20Started.md) document.

Documentation last updated for Spine-Unity for Spine 3.5.x (2017 April 11)
If this documentation doesn't cover some questions, please feel free to post in the official [Spine-Unity forums](http://esotericsoftware.com/forum/viewforum.php?f=3). 


# Controlling Animation
![](/img/animationcoding.png)

## SkeletonAnimation
Normally, `Spine.AnimationState` manages keeping track of time, updating the skeleton, queueing, layering and mixing/crossfading animation. If you're used to "Mecanim" as having State Machines, `Spine.AnimationState` is a simpler, non-programmable kind, but flexible enough to serve as the base for most animation logic.

But `Spine.AnimationState` is a C# class. To make it usable in Unity, the `Spine.AnimationState` object is wrapped in a `MonoBehaviour` called `SkeletonAnimation`. You will find that `SkeletonAnimation` has a member field called `state`, a reference to the `Spine.AnimationState` object.  

`SkeletonAnimation` both manages the timing of the updates (through `Update`) and generates the Mesh object since it derives from the `SkeletonRenderer` class. This is the main component that's added to the `GameObject` when you Instantiate a Skeleton Data Asset into a "Spine GameObject". You could say that SkeletonAnimation is *the* Spine component.

## How to use SkeletonAnimation

### For Beginners
“Out of the box”, an instantiated `GameObject` with a `SkeletonAnimation` component on it is ready to go. Changing the Animation dropdown in the Inspector allows you to set the initial animation for that SkeletonAnimation object which starts playing immediately when the scene starts.

Looping can be enabled and disabled, and the Time Scale can be changed. These Inspector properties map to `.AnimationName`, `.loop` and `.timeScale`.

For beginner code, you can use `skeletonAnimation.AnimationName` property to change the playing animation with its name. It will start playing and loop if the `skeletonAnimation.loop` was set to true at the time it was changed.

```
skeletonAnimation.TimeScale = 1.5f;
skeletonAnimation.Loop = true;
skeletonAnimation.AnimationName = "run";
```

### Left and Right Animations
It is not necessary for you to create separate left-facing and right-facing animations in Spine editor.
`SkeletonAnimation`'s `Skeleton` object has a FlipX property (and FlipY) you can set which horizontally flips the rendering and logical positions of bones of that skeleton.
```csharp
// If your character was facing right, this will cause it to face left.
skeletonAnimation.Skeleton.FlipX = true; 
```

However, if you opted to design your character asymmetrically but the movement is essentially the same in both directions, you can use Spine's Skins feature to have two skins— one facing right, and one facing left— and switch between those two as your character faces left and right.
```csharp
bool facingLeft = (facing != "right");  
skeletonAnimation.Skeleton.FlipX = facingLeft;
skeletonAnimation.Skeleton.SetSkin(facingLeft ? "leftSkin" : "rightSkin");
```


### For Typical Use
The class that actually controls the animation of the skeleton is **Spine.AnimationState**.
`SkeletonAnimation` is built around `AnimationState`. It stores a reference to its instance of `Spine.AnimationState` in `SkeletonAnimation.AnimationState`. This is instantiated on `Awake` so make sure to only access it on `Start` or any time after.

`AnimationState` is Spine’s common-case implementation of a Spine Animation State Machine. It is lower-level in a sense that it only manages updating, queuing, layering and mixing/crossfading animations. (It is not a "State Machine" in the Mecanim sense).

In code, you can play animations through `AnimationState` methods:
```csharp
// Plays the animation named “stand” once on Track 0.
skeletonAnimation.AnimationState.SetAnimation(0, “stand”, false);

// Queues the animation named “run” to loop on Track 0 after the last animation is done.
skeletonAnimation.AnimationState.AddAnimation(0, “run”, true, 0f);
```

Mixing/crossfading options can be found on your `SkeletonDataAsset`'s inspector. There you'll find a field for "Default Mix" which is the default duration of a mix between two animations on the same track. You may also define specific mix durations for specific pairs of animations there.


#### TRACKS
Tracks are animation layers, for playing more than one Spine animation at once.

This is useful for when, for example, you have an animation for a character's body running, but also have an animation for the arm shooting a gun.

If you play an animation on track 0, and another animation on track 1, both animations will play at the same time and end or loop according to their own durations independently.

The animations keys of the animation playing on the higher track will override the keys of the lower track: higher track number, higher priority.

```csharp
// Play a running animation on track 0, and a shooting animation on track 1.
skeletonAnimation.AnimationState.SetAnimation(0, "run", true);
skeletonAnimation.AnimationState.SetAnimation(1, "shoot", false);
```

#### TRACKENTRY
When you call `SetAnimation` or` AddAnimation`, it returns a `TrackEntry` object.

This object represents the instance of an animation playback for the animation you just played or queued. It holds information about what animation it is, its elapsed time, its own timescale and several other properties.

`TrackEntry` lets you control the state of a playing or queued animation, and playback parameters not included in `SetAnimation` and `AddAnimation`.

If you choose not to keep the reference to the `TrackEntry` object when you called SetAnimation or AddAnimation, you can get the currently active TrackEntry by calling
```csharp
	var trackEntry = skeletonAnimation.AnimationState.GetCurrent(myTrackNumber); // null if no animation is playing.
```


###### Current Time (or Start Time)
The time the Animation is currently playing at can be changed at any point while it's playing.

For example, you can change the `.Time` immediately after `SetAnimation` so the animation starts in the middle instead of the beginning. Note that time is in seconds. To convert Spine editor frames to seconds, you must divide by the dopesheet’s 30 fps tick marks: `AnimationTime = (frameNumber/30f)`.

```csharp
     // Play a "dance" animation starting from frame 10
     var trackEntry = skeletonAnimation.AnimationState.SetAnimation(0, "dance", false);
     trackEntry.AnimationTime = 10f/30f;

     // You can shorten the above code this way if you only need
     // to change one field, and don't need to store the trackEntry.
     skeletonAnimation.AnimationState.SetAnimation(0, "dance", false).Time = 10f/30f;
```

> If you are doing this to Animations with animation events, make sure you also set `lastTime` to the same value as well.
> If `lastTime` is left as 0, all events between time 0 and `time` will be captured and raised/fired in the next Update.

You can also change the point where the animation ends by setting `.EndTime`.

###### Timescale
You can change the playback speed by setting that `TrackEntry`’s `.TimeScale`. This gets multiplied by the `SkeletonAnimation.TimeScale` and `AnimationState.TimeScale` to get the final timescale.

> You can set timeScale to 0 so it pauses. Note that even if `TimeScale = 0` results in the skeleton not moving, the animation is still be applied to the skeleton every frame. Any changes you make to the skeleton will still be overridden in the normal updates.

Here is a sample helper method if you want to jump to a certain point in time.
```csharp
static public Spine.TrackEntry JumpToTime (SkeletonAnimation skeletonAnimation, int trackNumber, float time, bool skipEvents, bool stop) {
     if (skeletonAnimation == null) return null;
     return JumpToTime(skeletonAnimation.AnimationState.GetCurrent(trackNumber), time, skipEvents, stop);
}

static public Spine.TrackEntry JumpToTime (Spine.TrackEntry trackEntry, float time, bool skipEvents, bool stop) {
    if (trackEntry != null) {
		trackEntry.AnimationTime = time;
		if (skipEvents)
			trackEntry.AnimationLast = time;

		if (stop)
			trackEntry.TimeScale = 0;
	}
	return trackEntry;
}
```


###### TrackEntry-specific events.
You can subscribe to events just for that specific animation playback instance. See the list of events here: [Events Documentation](/Events.md).

## Spine.Animation

A `Spine.Animation` object corresponds to an individual "animation clip" animated in Spine. If a run animation called "run" was animated in Spine, you'll get a `Spine.Animation` object named "run".

Each `Spine.Animation` is a collection of `Timeline` objects. Each `Timeline` object is a collection of keys, which defines how a certain animatable value (scale, rotation, position, color, attachment, mesh vertices, IK, events, draw order) changes over time. Each `Timeline` has its own target: a `Bone` for scale, rotation and position; a `slot` for attachments and colors; a `MeshAttachment` for mesh deformation, and the appropriate parts of the skeleton for *draw order* and *events*.

In Spine runtimes, a `Spine.Animation` is applied to a skeleton with `Animation.Apply(...)`. This poses the `Skeleton`'s various parts only according to that Animation's `Timeline`s and keys. If a `Timeline` isn't there, the animatable value won't change. If a `Timeline` is there, it will overwrite whatever value is there with a new value.

This means that if you play two Animations at the same time using `Spine.AnimationState`'s track system, the `Timeline`s of the higher track will overwrite any changes that the lower track applied, but will not modify anything else.

This system allows the user to combine **different animations for different parts of the skeleton** and play them back independently of each other.

### Posing the Skeleton according to an animation frame/time.

You can call `Animation.Apply` directly if you want to pose a skeleton according to a certain point in time in that `Animation`. The Spine-Unity runtime also has a `Skeleton.PoseWithAnimation` extension method that allows you to do this according to an animation's name.

One of the parameters is always time. This is time in seconds. If you want to pose the skeleton according to what you see in the editor, you may need to call `skeleton.SetToSetupPose()` before calling `Animation.Apply` or `Skeleton.PoseWithAnimation`

#### One Animation After Another (pre-3.0)
This also means that without **auto-reset logic**, playing animations one after another will not necessarily cause it to look like what the animations like in Spine editor. Instead, playing a sequence of animations will result in later animations inheriting the values and bone poses of the previous animation.

In 3.0, Spine.AnimationState will handle auto-reset logic itself and give you the option to use it so it can behave like it does in Spine editor, or if you want the original free-rein mode for advanced uses.


## Animation Callbacks + Spine Events
**Spine.AnimationState** provides animation callbacks in the form of [C# events](https://msdn.microsoft.com/en-us/library/awbftdfh.aspx). You can use these to handle some basic points of animation playback.

Spine.AnimationState raises events:
 - `Start` when an animation starts playing,
 - `End` when an animation is cleared or interrupted,
 - `Complete` when an animation completes its full duration,
 - `Event` when a user-defined event is detected.

**WARNING:**
> NEVER subscribe to `End` with a method that calls `SetAnimation`. Since `End` is raised when an animation is interrupted, and `SetAnimation` interrupts any existing animation, this will cause an **infinite recursion** of End->Handle>SetAnimation->End->Handle->SetAnimation, causing Unity to freeze until a stack overflow happens.

To learn more about Spine Events and AnimationState callbacks, see the [Events documentation](Events.md).

### Advanced Animation Control
Tracks and TrackEntry are just part of `Spine.AnimationState`, and `AnimationState` is included with the Spine runtimes just to get you going immediately. It's usable in most cases and performant for production.
However, Spine-Unity and Spine-C# can function even without `AnimationState`. For example, `SkeletonAnimator` uses `UnityEngine.Animator` instead of `Spine.AnimationState` and uses Mecanim to manage numbers and data for mixing, layering and queueing.

If you wanted to do bookkeeping of your animations a different way, whether to be optimized based on how you plan to use it, or to handle special cases just the way you want them, you can build a different system.

In this case, studying the [Official Spine Documentation](http://esotericsoftware.com/spine-using-runtimes) will come in handy.

# Rendering
In general, Spine’s systems are built to take advantage of the way modern game engines use 3D graphics. This is especially true for its advanced features like Mesh Deformation and Weights.

It does this by interfacing with frameworks and engines “in 3D terms”; in terms of meshes and vertices, and texture mapping and UVs. So every image part is either defined by a polygon comprised of several triangles, or a rectangle made up of two triangles. This is not unusual even if the engine is mainly for 2D graphics.

![](/img/render_spineunity_p1.jpg)

The basic idea is this:
You have mesh made of triangles. This is shaped based on the Spine skeleton/model.
You have texture data. This comes from the atlas texture.
And you have shader programs or blending instructions.

You give all that to the game engine, the game engine gives it to the GPU, and then it renders. The textures are mapped to the mesh, and rendered based on the shader and blend mode.

![](/img/render_spineunity_p2.jpg)

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
![](/img/render_spineunity_alternatingmaterials.png)

More materials in this array means more [draw calls](http://docs.unity3d.com/Manual/DrawCallBatching.html), which can adversely affect performance if the same skeleton is instantiated several times. If you only have one of this skeleton, it is probably not a cause for significant slowdown.

To learn more about how to arrange atlas regions in your Spine atlases, see this page: [Spine Texture Packer: Folder Structure](http://esotericsoftware.com/spine-texture-packer#Folder-structure)

### Setting Material Properties Per Instance.
Likewise, using MeshRenderer.material and changing values there will not work.

The `Renderer.material` property generates a copy just for that renderer, but that material will immediately be overwritten by SkeletonRenderer's render code.   

On the other hand, `Renderer.sharedMaterial` will modify the original Material. And if you spawn more than one Spine GameObject that uses that Material, the changes you apply to it will be applied to all instances.

In this case, Unity's [Renderer.SetPropertyBlock](http://docs.unity3d.com/ScriptReference/Renderer.SetPropertyBlock.html) is a useful method. Remember that `SkeletonRenderer`/`SkeletonAnimation` uses a `MeshRenderer`. Setting that MeshRenderer's material property block allows you to change property values just for that renderer.

```csharp
MaterialPropertyBlock mpb = new MaterialPropertyBlock();
mpb.SetColor("_FillColor", Color.red); // "_FillColor" is a named property on a hypothetical shader. 
GetComponent<MeshRenderer>().SetPropertyBlock(mpb);
```

// TODO: Small section on clearing the property block

> **Notes on optimization**
> - using Renderer.SetPropertyBlock allows renderers with the same material to be batched correctly even if their material properties are changed by different MaterialPropertyBlocks.
> - You do need to call `SetPropertyBlock` whenever you change or add a property value to your MaterialPropertyBlock. But you can keep that MaterialPropertyBlock as part of your class so you don't have to keep instantiating a new one whenever you want to change a property.
> - When you need to set a property frequently, you can use the static method: `Shader.PropertyToID(string)` to cache the int ID of that property instead of using the string overload of MaterialPropertyBlock's setters.  

## Sprite Texture Rendering/Z Ordering
![](/img/render_spineunity_sortingface.jpg)

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
   - Shader’s ShaderLab Queue Tag (defaults to the “Transparent” queue like other sprites)
   - Material.renderQueue. (does nothing until you set it. It just overrides the shaderlab queue tag).
   - [Camera depth](http://docs.unity3d.com/ScriptReference/Camera-depth.html). (For multi-camera setups.)

For the most part, you won’t need to and shouldn’t touch Material.renderQueue and the shader’s queue tag.

*You can opt to use a different shader to get different kinds of blending, by changing the shader on the Material. Handling and working around the resulting blending behavior will likewise be up to you.

### Sorting Layer and Order
The **Sorting Layer** and **Sorting Order** properties found in `SkeletonRenderer`/`SkeletonAnimation`’s Inspector actually modifies the `MeshRenderer`’s [sorting layer](http://docs.unity3d.com/ScriptReference/Renderer-sortingLayerID.html) and [sorting order](http://docs.unity3d.com/ScriptReference/Renderer-sortingOrder.html) properties.

Despite being hidden in MeshRenderer’s inspector, these properties are actually serialized/stored normally as part of `MeshRenderer` and not part of SkeletonRenderer.

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

![](/img/skeletonDataScale.png)

If you want to set your skeleton’s base scale to something higher or lower, you can change this value in your Skeleton Data Asset’s inspector. This value is a multiplier used when your `SkeletonData` is loaded. Changing this value at runtime will not do anything after the `SkeletonData` has already been loaded.

If you want to dynamically change/animate your skeleton’s visual scale (like a balloon) in gameplay, you can do that by just changing `gameObject.transform.localScale`.

## Flipping Horizontally or Vertically
Accessing `Skeleton.FlipX` and `Skeleton.FlipY` can allow you to flip the skeleton horizontally or vertically.
It’s usually a good idea to make all your skeletons face one direction so that your flipping logic can be consistent throughout your code. But if not, you can always add an extra `bool` to your controller and [negate your intended flip value with boolean logic](http://stackoverflow.com/questions/13983198/negate-a-boolean-based-on-another-boolean).

You may have learned to flip your 2D sprites in Unity setting a negative scale on the Transform, or rotating it along the y-axis by 180 degrees.
Both of these work for purely visual purposes. But they have their own side effects to keep in mind:

 - Nonuniform scale can cause a mesh to bypass Unity’s batching system. This means it will have its own (set of) draw calls for each instance. So it’s okay for your main character. It can be detrimental if you have nonuniform scale for dozens of skeletons.
 - Rotating causes normals to rotate with the mesh. For lighting 2D sprites, this means they’ll point in the wrong direction.
 - Rotating X or Y may also cause Unity’s 2D Colliders to behave unpredictably.
 - Negative scale can cause attached physics Colliders and some script logic to behave with unexpected results.

