AvSetup File
============

The avSetup file is a crucial metadata file used by the engine.
It contains metadata which describes details of the project, and outlines how the engine should start itself.

The intention of the avSetup file is to allow customisation of how the engine loads itself.
Given its data driven approach, the engine would be capable of powering a number of different games without having to change the binary at all.
The intention of the avSetup file is to specify where exactly it should look to find this data.

Say for instance you were working on a modified version of a set of game files, and you didn't want anything you did to interfere with them.
You could place them somewhere separate from the engine install, all you would have to do to load these other files in, rather than the supplied ones would be to provide a different config file.

The config file is intended to represent a single game to be powered by the engine.
Multiple config files can be used to represent multiple games, and allow easy swapping between games, rather than having to re-compile things.
This also alleviates the trouble of having to hard code search paths into the engine binary, as these things can be fed in at runtime.

The avSetup file is also used by the testing framework to describe a single test.
It provides an easy way to encapsulate a single test case without having to modify the engine at all, as during startup the engine can just be pointed to a different file which represents a different test case.

Contents
--------
The file itself contains JSON data.
An example of a setup file is shown below

.. code-block:: json

    {
        "WindowTitle": "Empty",
        "CompositorBackground": "1 0 1 1",
        "SquirrelEntryFile": "squirrelEntry.nut",
        "DataDirectory": ".",
        "MapsDirectory": "../common/maps"
    }

In this example, the file specifies aspects of a simple project.
Each value is later parsed by the engine and used as part of setup.

All path values specified are relative to the setup file.

Basic setup file values
^^^^^^^^^^^^^^^^^^^^^^^

.. list-table::
   :widths: 50 50
   :header-rows: 1

   * - Value
     - Meaning
   * - WindowTitle
     - The title which will be set for the render window.
   * - DataDirectory
     - A path to the data directory. This path will be resolved relative to the setup file.
   * - CompositorBackground
     - A colour value which is used for the default background colour. Should be in the format '1 0 1 1', where each value represents one of RGBA.
   * - OgreResourcesFile
     - A path to a file containing ogre resource locations.
   * - SquirrelEntryFile
     - A path to the script file which will be executed at the start of the engine execution.
   * - MapsDirectory
     - A path to the directory which contains map data.
   * - SaveDirectory
     - A path to a directory which contains saveable data. This should be writable.
   * - WorldSlotSize
     - An integer value which outlines how big a slot in the world is.
   * - WindowResizable
     - A boolean specifying whether the user should be able to resize the window.
   * - WindowWidth
     - The initial width of the window.
   * - WindowHeight
     - The initial height of the window.
   * - DialogScript
     - A path to the script file which will be called for dialog events.
   * - UseDefaultActionSet
     - Boolean of whether the engine should setup the default action set.
   * - UserSettings
     - A json object containing a list of user settings.

Setup file values for testing
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

These values are related specifically to testing.
If the engine was built without testing capabilities, these settings are ignored.

.. list-table::
   :widths: 50 50
   :header-rows: 1

   * - Value
     - Meaning
   * - TestMode
     - True or false depending on whether this is a test.
   * - TestName
     - The name of the test.
   * - TestTimeout
     - An integer representing the number of seconds until the test should timeout. This is used to prevent a broken test never failing.
   * - TestTimeoutMeansFailure
     - True or false for how the engine should treat a timeout. Projects such as stress tests might not want to fail on test timeout.

User Settings
-------------

The user is able to set custom settings values for their project.
These values can later be read as required.

An example of this syntax is shown below:

.. code-block:: json

    "UserSettings": {
        "userInt": 100,
        "userFloat": 100.1,
        "userBool": true,
        "userBoolCase": false,
        "userString": "Some example text",
        "SecondSectionInt": 300
    }


These values can be read from scripts like this:

.. code-block:: c

    local userValue = _settings.getUserSetting("userValue");

The returned type depends on the value set in the file.
