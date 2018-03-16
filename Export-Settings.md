#### spine-unity
The information here may change over time as the implementations within Spine-Unity get updated, improved or fixed.

Documentation last updated for Spine-Unity for Spine 3.6.x (2018 Mar 16)
If this documentation contains mistakes or doesn't cover some questions, please feel to open an issue or post in the official [Spine-Unity forums](http://esotericsoftware.com/forum/viewforum.php?f=3). 

----------

# Recommended Spine Export Settings for Spine-Unity #
### Export Settings for Spine-Unity ###
![](/img/spine-runtimes-guide/spine-unity/spine-unity-export-settings.png)  
Spine-Unity is capable of loading both Json and Binary files.  
- **Json** is best during development. While we always recommend freezing your Spine editor version to the specific runtime version your game is using in development, we recommend exporting with json during development as it is more resilient to minor updates in runtime versions.  
Json also makes it easier to debug, test and handle errors.  

- **Nonessential data** allows you to more fully rehydrate a Spine project from a json export in cases of data loss.
- **Pretty Print** adds whitespace (spaces, tabs and line endings) to the json to make the text more humanly readable and cleaner to track by version control (such as git). This makes it easy to debug problems in the export and make manual changes for testing, and keep track of changes.

- **Binary** is best for performance and release-testing and when you are closer to release. This is the point in your development where it is usually not a good idea to update any more of its plugins and extensions (such as Spine-Unity). Binary export allows faster loading times on the runtime side, and smaller skeleton data sizes.
  


### Texture Packer Settings for Spine-Unity ###
![](/img/spine-runtimes-guide/spine-unity/spine-unity-texture-packer-settings.png)  
**Regions**
If you are using **Spine Professional** and using meshes, we recommend leaving `Strip Whitespace` OFF.
Whitespace stripping causes problems with mesh UV coordinates.  
*The best practice is to pre-trim your images before starting to set up your skeleton in Spine.*

**Pages**
`Power of two` and `Square` used to be requirements when you have PVRTC compressed images and intend to target iOS.
It's not clear if this is still the case, but by default, Spine-Unity will set your image settings to uncompressed (this means uncompressed at runtime) on import so it comes out clean.

**Output**
For beginners, and quick testing, leave `Premultiply Alpha` on. This is compatible with the bundled default `Spine/Skeleton` shader. This also allows it to be compatible with per-slot additive blending.

If you want to use `Sprites/Default` or `Sprites/Diffuse`, leave Premultiply Alpha unchecked and Bleed checked.

If you are using specialized shaders, you should know if it uses Straight alpha or Premultiplied alpha blending and check or uncheck this box accordingly. If you turn `Premultiply Alpha` off, you probably want to check Bleed.

If you plan to implement custom shaders, using Straight Alpha (Premultiply Alpha off) will likely make them easier to implement.

For more information on export settings and Shader compatibility, see the [Shaders documentation](Shaders.md).

For more info, check this thread: http://esotericsoftware.com/forum/Premultiply-Alpha-3132
//TODO: Move Premultiply Alpha information into this doc.

**Options**
Unity does not recognize custom file types even if they are plain text, so we recommend setting the extension to `.atlas.txt`.
For the most part, The Spine-Unity importer will detect `.atlas` files and automatically rename them to `.atlas.txt` for you. **But to save it a step and prevent any mixups when updating assets, set the Atlas extension specifically to .atlas.txt.** 
