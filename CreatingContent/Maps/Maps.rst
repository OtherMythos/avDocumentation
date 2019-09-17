Maps
====

Maps are used in the engine to define content for the world.
Maps are a fundamental part of the world, and are defined in a data driven way.

Much of the definition of maps are done within the file system, in the :ref:`maps-directory`.

Directory Structure
===================

Map Name Directory
^^^^^^^^^^^^^^^^^^

The existence of maps is based on the existence of directories within the maps directory.
Say for instance the user had the map 'overworld' currently set, the engine would start searching for that maps content within the path:

.. code-block:: c

    {MapsDirectory}/Overworld

The {MapsDirectory} would be replaced with the path to the maps directory.
Essentually if there is no directory within the maps directory with that exact name, the engine will load just blank chunks.

Chunks
^^^^^^

Chunks themselves are then defined within the map name directory.
The name of the directory depends on the coordinates of this chunk, for example ``00010001``.
That directory would represent the chunk x:1, y:1.

At the moment there are four digits to represent each coordinate, with the x axis being the first four, and the y the second.
Similarly to the map name directory, if no directory is found for that chunk a blank chunk will be inserted.

File Structure
==============

If a chunk is requested for load and the engine finds the correct directory structure for the requested chunk, it will then start loading the files within.
These files are what actually define the content of the chunk.
These are:

 - ``meshes.txt`` - Static meshes which never move through the lifetime of the chunk, for instance these might be things like rocks, foliage, and so on.
 - ``bodies.txt`` - Static physics shapes which don't move. This can be used to define physics data for things like the meshes in the meshes file.

As well as this, within the chunk directory is the ``terrain`` directory, which is used to define terrain.

Terrain Directory
=================

The terrain directory exists within the chunk directory. It is used to define a piece of terrain that should exist within that specific chunk.

Within the terrain directory a few files will exist

 - ``height.png`` - An image used to represent the height of the terrain. This should be a grayscale image. This file is necessary for loading terrain. Without it no terrain will be created.
 - ``shadow.png`` - A pre-baked shadow map.
 - ``datablock.material.json`` - The datablock is the material that's applied to the terrain. It defines things like blend blend layer textures, colours and other things. The datablock is slightly more complex than the other files, so it has a further section below.

Determining whether or not that piece of terrain will be loaded or not is based on a few factors.

 - Whether this directory exists in the first place. If the terrain directory isn't within the chunk directory no terrain will be loaded. So if you don't want terrain in your chunks don't create this directory.
 - Whether or not the heightmap exists. If it doesn't the terrain won't be loaded.

Terrain Datablock
^^^^^^^^^^^^^^^^^

Datablocks are used by ogre to represent 'materials'.
The terrain datablock is a material which is applied to a terrain to define things like blend textures, blend maps and so on.
The user doesn't have to define a datablock, as if one isn't found a default will just be used.

Defining a datablock does not depend on a filename at all.
The datablock file could be named anything, as long as it ends with the ``.material.json`` extension. The important part is what's inside the files.

An example of file contents would be:

.. code-block:: json

    {
        "Terra": {
            "TerraTerrainMap00000000": {
                "diffuse": {
                    "value": [
                        1.0,
                        1.0,
                        1.0
                    ]
                }
            }
        }
    }

The file contents is just a json.
The most important part of this file is that it is contained within the ``Terra`` object and has a name referencing the chunk.
The Terra object makes sure that this datablock is defined within the terra hlms system, and is necessary for the datablock to be loaded correctly.
So in the example here the datablock has the name ``TerraTerrainMap00000000``.
Ogre relies on datablocks having unique names, and this even includes across hlms systems.
To avoid conflicts between the engine, the datablock is prefixed with the ``Terra`` string.

The engine works by generating a name from the chunk coordinates. It then tries to find a loaded datablock with this name.
If for whatever reason this cannot be found it will simply use the default.
The breakdown of a datablock name is this:

 - ``Terra`` - Always there. Used as a prefix to avoid conflicts between other hlms'es.
 - Map Name - The name of the map. In the example above the map is named 'TerrainMap'.
 - Chunk Coordinates - The same as the coordinates for the chunk directory.

If the datablock is named anything else it won't be applied to the terrain.

.. Note::

    Internally the engine creates a new ogre resource group in the terrain directory when the load begins.
    This is used to avoid conflicts between images such as ``height.png``.

    However, datablocks don't follow this convention.
    Instead if two datablocks have the same name an assertion is thrown.
    Furthermore, datablocks aren't unloaded when the resource group is unloaded.
    This makes it easy to run into assertion errors due to conflicting datablock names if chunks are often being loaded and unloaded, and the names were configured incorrectly.

    The engine is setup to try and destroy the datablock name specified above.
    So if the names have been setup correctly, the destruction of the datablocks will happen correctly.

.. Warning::

    Do not define multiple datablocks in the terrain directory!
    Any datablocks in the terrain directory will be loaded when the resource group is created, but not destroyed when the group is unloaded.
    It is possible to iterate through datablocks in a certain group and destroy them, however this is a costly operation.
    The engine does not do this for the sake of efficiency, and instead assumes that the user defined their names correctly.

    If you create any other sort of datablock (pbs or terra) you will run into assertions if the terrain directory is loaded twice.

    So don't do that.
