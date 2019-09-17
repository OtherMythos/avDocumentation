.. _data-directory:

==============
Data Directory
==============

The data directory is one of the fundamental places in which data should be stored.
Files in the data directory are read only, and are used as actual resources for the execution of the game.

The data directory is a loose term for a collection of files.
The engine has some fundamental contents which the engine looks for, and are built into the code.

For instance:
 - The squirrel entry file. Default ``squirrelEntry.nut``
 - The Ogre resources file. Default ``OgreResources.cfg``
 - The maps directory. Default ``maps``

The exact location of these files and directories can also be specified directly in the ``avSetup.cfg`` file, however it is much more convenient for the user to just specify the path to the data directory.
The engine will then search for these files based on the above defaults.
If any of these files aren't found the engine won't crash, however it will start in a state where certain things might not work.
For instance, if no squirrel entry file is provided the engine will launch but do nothing.

.. Note::

    The data directory is automatically assumed to exist in the master directory, unless otherwise stated in the ``avSetup.cfg`` file.

    If you want the data directory to be inside the same directory as the avSetup.cfg file, you must specify that in the file.
    For example:

    .. code-block:: c

        DataDirectory	.

    All paths within the setup file are resolved relative to the setup file's path.

.. _squirrel-entry-file:

Squirrel Entry File
-------------------

The squirrel entry file is the first squirrel script that gets executed by the engine.
The engine itself isn't hard coded to run any other script files unless told to.
The purpose of the entry file is to tell the engine how it should start up, and acts like the 'main' function in other programming languages.

In an actual game it would be used to do things like setup the world, load in save state and kick off the actual game execution.
However, in something like a test it can just be used to setup state and assert a pass or failure value.
Conversely, if the user just wanted to see the engine do something, the user could just make it load a model and move the camera to be able to see it.

Ogre Resources File
-------------------

The ogre resources file is a file that registers the ogre resource locations.
Ogre itself has its own resource manager that deals with things like textures and meshes.
The user can supply these resource locations with this file.
It is a simple config file which contains various paths.

.. code-block:: c

    [General]
    FileSystem=/home/edward/Documents/avData/meshes
    FileSystem=../resourcesDirectory

In this example I provide an absolute path and a relative path to directories.
The engine accepts both. Relative paths are resolved relative to the location of the ogre resources file, **not** the data directory.
This is more manageable for tests.

.. _maps-directory:

The Maps Directory
------------------
The maps directory is a directory which contains maps.
These maps are contained inside of a directory structure which in itself describes them.
