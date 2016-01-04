### spine-unity
The information here may change over time as the implementations within Spine-Unity get updated, improved or fixed.
This document is currently based off Spine-Unity's README.

##### Licensing

This Spine runtime may only be used for personal or internal use, typically to evaluate Spine before purchasing. If you would like to incorporate a Spine Runtime into your applications, distribute software containing a Spine Runtime, or modify a Spine Runtime, then you will need a valid [Spine license](https://esotericsoftware.com/spine-purchase). Please see the [Spine Runtimes Software License](https://github.com/EsotericSoftware/spine-runtimes/blob/master/LICENSE) for detailed information.

The Spine Runtimes are developed with the intent to be used with data exported from Spine. By purchasing Spine, `Section 2` of the [Spine Software License](https://esotericsoftware.com/files/license.txt) grants the right to create and distribute derivative works of the Spine Runtimes.

----------

# Getting Started

Spine-Unity is built on top of Spine-C# ([spine-csharp](https://github.com/EsotericSoftware/spine-runtimes/tree/master/spine-csharp)).

The **Spine-Unity** runtime provides functionality to load, manipulate and render [Spine](http://esotericsoftware.com) skeletal animation data in [Unity](http://unity3d.com/) engine.

While Spine-Unity works in Unity without the need for any other plugins, it also works with [2D Toolkit](http://www.unikronsoftware.com/2dtoolkit/) and can render skeletons using TK2D's texture atlas system.

## Overview

A Spine skeleton GameObject (a `GameObject` with a `SkeletonAnimation` component on it) can be used throughout Unity like any other GameObject. It renders through `MeshRenderer`.

`SkeletonUtility` allows other GameObjects to interact with the Spine skeleton, to control bones in the skeleton, be controlled by the skeleton, attach colliders, etc.

For advanced uses and specific optimization cases, Spine skeletons can be "baked" into native Unity animation assets. Since Unity's animation feature-set does not overlap with Spine's perfectly, baked assets have many limitations and removed features. For most uses, baking is not necessary.

The [Spine Unity Features Tutorial](http://esotericsoftware.com/forum/Unity-Feature-Tutorials-4839) forum thread has many videos on how to use spine-unity.

## Quick installation

Download and run this Unity package:

[spine-unity.unitypackage](http://esotericsoftware.com/files/runtimes/unity/spine-unity.unitypackage)

In the `Assets/Examples/Scenes` folder you will find many example scenes that demonstrate various spine-unity features.

## Manual installation

You can also choose to setup and run from the Git files:

1. Download the Spine Runtimes source using [git](https://help.github.com/articles/set-up-git) or by downloading it [as a zip](https://github.com/EsotericSoftware/spine-runtimes/archive/master.zip).
2. Spine-Unity requires both `spine-csharp` and `spine-unity`.
	- Copy the contents of `spine-csharp/src` to `Assets/spine-csharp` in your Unity project directory.
	- Copy the contents of `spine-unity/Assets/` to `Assets/` in your Unity project directory. Including `Gizmos` and `spine-unity` and `Examples` if you want them.


> - `Gizmos` is a [special folder](http://docs.unity3d.com/Manual/SpecialFolders.html) in Unity. It needs to be at the root of your assets folder to function correctly. (ie. `Assets/Gizmos`
- `spine-csharp` and `spine-unity` can be placed in any subfolder you want.

----------

## Importing skeleton data

1. When you export (and pack) from Spine editor, you will get 3 files:
	- a **JSON file** (`.json`) which stores the skeleton.
	- a **packed texture image** (`.png`) which stores the image parts. 
	- and its **libGDX atlas file** (`.atlas.txt`) which lists where each image is within the packed texture.
2. Add your `.json`, `.atlas.txt` and `.png` into your Unity project. Make sure they're in the same folder*.
	- You can do this through **Unity's Project View**: Drag and drop a folder containing the `.json`, `.atlas.txt` and `.png` files exported from Spine directly into the Unity Project view.
	- ... or you can opt to do this through **Windows File Explorer** or **OSX Finder**. Move or copy your `.json`, `.atlas.txt` and `.png` files into your Unity project's `Assets` folder.
2. Spine-Unity will automatically detect the `.json` and `.atlas.txt` and attempt to generate the necessary Spine-Unity assets.
3. To start using your Spine assets, right-click on the SkeletonDataAsset (the asset with the orange Spine logo on it) and choose `Spine > Instantiate(SkeletonAnimation)`. This will add a GameObject with a `SkeletonAnimation` component on it.
	-  If you are more familiar with Mecanim, you may choose `Spine > Instantiate(Mecanim)` instead.
4. For more info on how to control the animation, see the [Spine-Unity Animation Control documentation](https://github.com/pharan/spine-unity-docs/blob/master/Animation.md).

*Some advanced setups include sharing atlases between skeletons. See the forums on how to import this.  

For __2D Toolkit__ users, packing a `.png` and `.atlas.txt` is not necessary. Instead, you will have the appropriate field in your SkeletonDataAsset to assign a reference to `tk2dSpriteCollectionData`. 

----------


## Example Scenes

To run the examples:

1. Download the Spine Runtimes source using [git](https://help.github.com/articles/set-up-git) or by downloading it [as a zip](https://github.com/EsotericSoftware/spine-runtimes/archive/master.zip).
1. Copy the contents of `spine-csharp/src` to `spine-unity/Assets/spine-csharp`.
1. Open an example scene file from `spine-unity/Assets/examples/` using Unity 4.3.4 or above.

#### Spineboy
This shows the spineboy skeleton. The included Spineboy script shows the basics of controlling SkeletonAnimation through code.

There is also a cube which follow's Spineboy's hand using the `BoneFollower` script. This script allows a Unity Transform to follow the position and rotation of a Spine bone.
##### Spineboy Movement
This shows the spineboy skeleton controlled through a condensed SkeletonAnimation controller setup: `SpineboyController`.

You can make Spineboy run left and right, and he will react to mouse clicks.

#### Goblins
This shows a male and female goblin that use the same skeleton and animations. Click to change the skin from male to female and back. It has a single atlas page, so is drawn with just 1 draw call. It uses the `Skeleton Lit` shader for vertex lighting. The [Goblins.cs](https://github.com/EsotericSoftware/spine-runtimes/blob/master/spine-unity/Assets/examples/goblins/Goblins.cs) script manipulates the head bone after the animation is applied.
#### Dragon
This shows the dragon skeleton. The flying animation has many image changes. It also shows shadow rendering. This example uses a multi-page atlas so has a high number of draw calls.

#### Raptor
The raptor skeleton demonstrates animated Free-form deformation, skinning (its legs and tail) and IK constrants (on its feet, and spineboy's hands).

##### Raptor Animated Physics
This playful version of the raptor scene demonstrates a SkeletonUtility setup, where certain bones are driven by a chain of rigidbodies to simulate physics. Here, the raptor's tail, neck and arms are bouncing according to Box2D physics. Note how Unity's default Time Settings can cause a different update rate between physics (`FixedUpdate`) and animation (`Update`). You may have to handle this yourself if you choose to integrate physics into your animations.  Certain rigidbody interpolation settings and timestep sizes can improve the quality.

##### Raptor GroundConstraint
This version of the Raptor scene demonstrates the primary use of `SkeletonUtilityGroundConstraint`. By controlling the position of the foot IK targets and checking colliders, you can allow your character to visually plant their feet on uneven terrain.
#### Eyes
The Eyes scene demonstrates a simple case of programmatic control of skeletons. Here, eyes can follow a target Unity Transform. Move the box around and see the eyes follow it. This is done through `SkeletonUtilityEyeConstraint`.

### SpineGauge
This is a simple scene with a script that demonstrates using a Spine skeleton for a visual healthbar or arbitrary gauge. It does this using `Animation.Apply`, a relatively lower-level Spine API.
Using this technique, the gauge animation can be made as complicated as the Spine allows, give more control of the movement and visuals to the animator, or remove the need for extra logic for UI alignment.


## Notes

- This slightly outdated [spine-unity tutorial video](http://www.youtube.com/watch?v=x1umSQulghA) may still be useful.
- Atlas images should use **Premultiplied Alpha** when using the shaders that come with spine-unity (`Spine/Skeleton` or `Spine/SkeletonLit`).
- **TEXTURE SIZES.** Unity scales large images down by default if they exceed 1024x1024. This can cause atlas coordinates to be incorrect. To fix this, make sure to set import settings in the Inspector for any large atlas image you have so Unity does not scale it down.
- **TEXTURE ARTIFACTS FROM COMPRESSION.** Unity's 2D project defaults import new images added to the project with the Texture Type "Sprite". This can cause artifacts when using the `Spine/Skeleton` shader. To avoid these artifacts, make sure the Texture Type is set to "Texture". Spine-Unity's automatic import will attempt to apply these settings but in the process of updating your textures, these settings may be reverted.
