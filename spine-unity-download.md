# Spine-Unity #

**[Forums](http://esotericsoftware.com/forum/viewforum.php?f=12)** | **[Documentation](http://esotericsoftware.com/spine-unity-documentation)** | **[GitHub](https://github.com/EsotericSoftware/spine-runtimes/tree/master/spine-unity)**

## Download ##
http://esotericsoftware.com/files/runtimes/unity/spine-unity.unitypackage
<br>Compatible with Spine 3.3.x (Last updated: 2016 June 29)

###Compatibility###

- Runtimes cannot load exported binary file versions that are newer or older than the version it supports.
- Json exports are more stable and have better chances of being compatible with future versions, but may still break.
- If you want to avoid incompatibility issues with a set runtime version, you can choose a Spine editor version that matches that runtime specifically. This can be done in Spine's `Settings...` window.<br>
For more information, see [this forum topic](http://esotericsoftware.com/forum/Spine-editor-and-runtime-version-management-6534). 

----------
## Installing in a new project
1. Create an empty Unity project.
1. Import the spine-unity.unitypackage you downloaded. (double-clicking on the unitypackage should open it in Unity).

If you are just getting started, there are more detailed steps in the [documentation section on Getting Started](http://esotericsoftware.com/spine-unity#Getting-Started).<br> For more options, see the [documentation section on Installing](http://esotericsoftware.com/spine-unity#Installing).

## Updating an existing project

- As with Unity updates, it is always recommended that you back up your whole Unity project before performing an update.
- Always check with your Lead Programmer and Technical Artist before updating your Spine runtime. Spine runtimes are source-available and designed to be user-modifiable based on varying project needs. Your project's Spine runtime may have been modified by your programmer. In this case, updating to the latest runtime also requires reapplying those modifications to the new version of the runtime.

### In-place Update (.unitypackage)
1. Open your Unity project.
2. Import the .unitypackage into your existing project by double-clicking on the unitypackage file or dragging it into Unity editor Project view.
3. The Import dialog will show which files are updated and will update them regardless of where you moved them in your project since you last imported.

>  - This functionality may not work correctly if your meta files were corrupted or replaced. In that case, you may have to do delete the old version of the runtime before importing the unitypackage.
>  - Much older versions of the unitypackage had inconsistent meta files for spine-c#. You may have to delete the older spine-csharp folder when updating. This otherwise works correctly.
>  - Occasionally, major refactoring involves removing or merging some files. Importing the .unitypackage will not delete those leftover files when you update so you may have to do that yourself. Check here for announcements regarding changed/leftover files: http://esotericsoftware.com/forum/Noteworthy-Spine-Unity-Topics-5924

For more options, see the [spine-unity documentation page section on updating](http://esotericsoftware.com/spine-unity#Updating-Your-Projects-SpineUnity-Runtime).