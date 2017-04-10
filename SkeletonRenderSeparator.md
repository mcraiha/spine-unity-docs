Documentation last updated for Spine-Unity for Spine 3.5.x (2017 April 11)
If this documentation doesn't cover some questions, please feel free to post in the official [Spine-Unity forums](http://esotericsoftware.com/forum/viewforum.php?f=3).

# SkeletonRenderSeparator
a Spine-Unity Rendering Module (Modules\SkeletonRenderSeparator)

Normally, Spine components that render a skeleton will use a single renderer to render the whole mesh for its skeleton. This prevents you from inserting/sandwiching other UnityEngine.Renderers (SpriteRenderer, MeshRenderer, LineRenderer, ParticleSystem, etc...) between its parts arbitrarily.

**SkeletonRenderSeparator** allows you to split your SkeletonRenderer or SkeletonAnimation render into two or more MeshRenderers instead of only one.

This means you can set their sorting layers and sorting order individually, allowing you to render things in between parts of your skeleton. This can be particles, Unity SpriteRenderers, parts of your level, other skeletons, anything that uses a UnityEngine.Renderer that you can sort. This can be useful for dynamically sorted special effects, if you need your character to ride a vehicle, or if just want them to hug a tree.

You can find a sample scene in the unitypackage: `Examples/Other Examples/SkeletonRenderSeparator.unity` 
![](/img/skeletonrenderseparator_p1.png)

## How to Use
### Step 0
Make sure you know your Skeleton's draw order. Find out which slot you want use to separate your Skeleton's render into parts. For convenience, label this slot clearly before exporting your Skeleton.

![](/img/skeletonrenderseparator_steps.gif)
Fig 1. Steps 1 to 3

### Step 1: Add the SkeletonRenderSeparator component
Select your Spine GameObject and right-click on your `SkeletonAnimation` or `SkeletonRenderer` and choose "Add Skeleton Render Separator". This will add the SkeletonRenderSeparator component to the same GameObject.

### Step 2: Assign Separators
The inspector will warn you if you have no separators.
Set the separator slots by choosing the correct slot(s) under "Separator Slot Names". The dropdown will show the list of slots in your Skeleton.
- This field actually directly manipulates the field serialized in your SkeletonRenderer/SkeletonAnimation.

### Step 3: Make sure you have enough Parts Renderers 
The inspector will warn you if you don't have enough renderers to render the parts. Click on the "Add the missing renderers (n)" button if this is the case. This will create GameObjects with SkeletonPartsRenderers on them, and add them to SkeletonRenderSeparator's partsRenderers list.

> Note: The SkeletonRenderSeparator only detects the currently required number of parts renderers. If at any point at runtime during animations, the render requires more because you changed the draw order, you may need to add one or two extra manually by clicking on `Add Parts Renderer`.

### Step 4: Set Sorting Layers and Orders
Each of these SkeletonPartsRenderers are now in their own GameObject. Each of them have their own sorting layer and order via their own MeshRenderers.
Select them in the hierarchy and set their properties in the inspector.

## Manipulation at runtime 
### Enabling and Disabling
By default, SkeletonRenderSeparator will take over SkeletonRenderer's mesh rendering task.
Likewise, if you disable SkeletonRenderSeparator, SkeletonRenderer should will take over rendering again.

In code, you can enable and disable this through:
```csharp
skeletonRenderSeparator.enabled = true; // separation is enabled.
skeletonRenderSeparator.enabled = false; // separation is disabled.
```
### Changing the separation point
The point of separation is not stored in SkeletonRenderSeparator. It is defined by the separator slots in SkeletonRenderer/SkeletonAnimation. If you want to manipulate this at runtime, you can call `Add`, `Remove` or `Clear` on SkeletonRenderer's `List<Slot> separatorSlots`.
```csharp
Spine.Slot mySlot = skeletonAnimation.skeleton.FindSlot("MY SPECIAL SLOT");
skeletonAnimation.separatorSlots.Clear();
skeletonAnimation.separatorSlots.Add(mySlot);
```

## Overview
- The SkeletonRenderSeparator takes rendering instructions from SkeletonRenderer — by subscribing to its `GenerateMeshOverride` event.
- It then splits up the instructions and distributes it among its SkeletonPartsRenderers. These are assigned in `SkeletonRenderSeparator.partsRenderers`.
- Each SkeletonPartsRenderer processes those instructions to generate a mesh based on a range of slots.
- The renderers are ordered according to Skeleton draw order. This is also similar to Unity's Sorting "Order in Layer" ordering. (This means the first SkeletonPartsRenderer is given the farthest attachments to render. And the last SkeletonPartsRenderer is given the nearest attachments.)
- Each SkeletonPartsRenderer can have multiple submeshes. This makes it compatible with multi-page atlases and multi-atlas setups and selective material overrides. This also allows the number of resulting parts to be predictable (always 1 + the number of separator slots chosen).
