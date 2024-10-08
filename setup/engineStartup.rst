Engine Startup
==============

When the engine comes to start itself up there are a number of steps it goes through.
Much of this can be configures to suit the needs of the user, and the engine itself takes a data driven approach when deciding how the setup should happen.

Things the engine needs to take into account would include:
 - Where are the data files for the game?
 - Where are the files necessary for libraries to function (HLMS shaders for instance)?
 - Where should things like save files be stored?
 - Other useful settings like window titles.

Bear in mind that many of these files and requirements fall into different categories.
For instance, Ogre's HLMS shaders are necessary for ogre, but have nothing to do with the game files themselves.
Therefore they shouldn't be stored with the game files.
Furthermore, save files are written by the engine, but have nothing to do with the game files either, so they should target a separate directory.
The engine makes lots of distinctions like this and allows the user to specify their exact settings.

The steps taken in the startup to determine settings are outlined below:
 - Start the ``Base`` class.
 - Determine the Master Directory.
 - Check for an ``avSetup.cfg`` file.
 - Read in the settings from the avSetup file.

Directories
-----------
The engine has a number of specified directories (folders) which it expects to find in order to execute correctly.
Some are more important than others.

Av directories:
 - Master Directory
 - Data Directory
 - Save Directory

Master Directory
^^^^^^^^^^^^^^^^
The Master Directory is the most important of the lot, as without it the engine will immediately terminate.
It is the directory in which the engine expects to find its engine files and config files.
It is the directory which contains the avSetup file.

The master directory is the only hard coded directory which can't be changed by config or settings.
On Linux and Windows the master directory is the present working directory (pwd), which is just the directory in which the engine is ran from.
On MacOS, the Master Directory is set to be the Resources directory inside the app bundle.

Data Directory
^^^^^^^^^^^^^^
The Data directory is the directory which actually contains the game files.
I like to refer to these as the data files, which is why it's called the data directory.
The Data Directory is entirely built up of game files, which should be read only.

For more information please see :ref:`data-directory`.

Save Directory
^^^^^^^^^^^^^^
The Save Directory is the directory in which the serialiser will write its contents to.
This should be the only directory which the engine writes to.

Engine and Game files
---------------------
There is a big difference between the purpose and categories of files.
Some are engine specific files, while some others are game specific.

Engine files are used for the execution of the engine, and are universal to all games powered by the engine.
An example would be the HLMS shader files for Ogre.

Game files are files that the engine loads in to represent the game.

The difference between the file types means the same binary engine can power a number of different games without the need to rely on ridgidly defined paths in the engine code.

avSetup File
------------
The ``avSetup.cfg`` file is a game file. There is intended to be one per project built with the engine.

The file is an important file in the setup of the engine.
It is responsible for outlining many of the basic options used by the engine, such as paths to other resource directories, and info such as a window title.

More details of this file are provided in the section <section>

Ogre HLMS files
---------------
The Ogre HLMS files are a group of files necessary for the correct operation of Ogre.
They're used as building blocks to generate shaders.
They are engine files, and are therefore defined as part of the code.
They are expected to be in the master directory under the name of ``Hlms``, and are copied into the master directory by the build system from the ogre directory.
If the engine can't find them it will abort.
