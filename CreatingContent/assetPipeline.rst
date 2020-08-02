Asset Pipeline
==============

One prime consideration for content creation is managing assets.
If no structured approach is created, managing the creation and lifetime of assets can quickly become uncontrollable.

A number of considerations exist:

 - Will there be a separation between project files and raw assets (i.e blender project to .mesh)
 - How will assets be stored and versioned.
 - What if something small needs to be changed?

One of the most common sources of slowdown is exporting assets to their correct location.
With the example of meshes, if done manually, the user would have to open the 3d modeller project file, click file>export, choose a directory, click export.
Doing this manually for even a small change is error prone and time consuming.
This concept could be extended to other types of resources such as textures.
For instance, the user might be creating assets in a vector format, but expected to rasterise that for the final project.
In this instance, the vector files should be used as the original source of the asset, not the rasterised image.

Regarding the question of what should be versioned, it is incorrect to version the final .mesh file.
Really this file is an output of the modeller application.
Instead, it should be the modelling application's project file that is versioned, and then the final .mesh file is produced from this.
Separating project files from their output helps reduce clutter.

The asset pipeline tool helps fulfil these desires.
It is a script which allows the user to automate the output of design tools such as 3d modellers to a format the engine can understand.
The entire process should be based around a single command line command.
The .blend and .svg files are versioned appropriately, and then the user can run this command to produce a full directory structure containing the assets which can be understood by the engine.
There is no need to open blender and manually export assets, as the tool does this by itself.
