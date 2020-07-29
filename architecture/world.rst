World
=====

The world is a large set of components and systems in the engine.
It acts as a base in which gameplay elements can occur, for instance, physics, entities and 3d objects.
To make a game with the engine, the user does not necessarily have to create a world, however many common requirements for game development are provided by the world.

The world is a single class that deals with the encapsulation of a range of other components.
These components include but are not limited to:

 - Physics
 - Entities
 - Streamable content such as chunks

Only a single world can exist at a time.

The world supports implementation of save states (serialisation).
When creating a world, the user is able to supply a save file with which the world should construct itself.
The save file will be used to bring the world to a state identical to when the save file was originally created.

When the world is created, it will initialise all the other components which are involved in the world and begin loading content from there.

Creation and lifetime
--------
A world can be constructed using the script function:

.. code-block:: c

    _world.createWorld();

It can be destroyed using the script function:

.. code-block:: c

    _world.destroyWorld();

The user is given the choice of when (if at all) to construct the world.
Rather than creating a single world at the startup of engine, a better solution is to allow the user to specify when they wanted the world created.
Consider for example a menu system that's presented to the player on startup.
This menu would show a title screen, then allow them to select a save.
Up until they've selected the save there would be no need for the world, and now that a save has been selected the world can be constructed with that save in mind.
Now remember that the game will most likely want to have a 'quit to menu' option.
In this case, the world would be destroyed to make room for another.
The user might go to the menu screen and select another save, and the process would continue as before.

The world is therefore required to be flexible in its creation and construction, as it could happen an unlimited amount of times during runtime.
In order to create a new world, the old one must first be destroyed.

Access
------
Some script functions are reliant on the world for their operation.
If the ``_world.createWorld()`` function hasn't been called by the time these functions are called, an error will be thrown.

For instance:

.. code-block:: c

    //No world exists at this point.
    local e = _entity.create();
    //calling this function here will throw an error.

The script api documentation describes in more detail which functions are reliant on the existence of the world.

..
    TODO mention the save states. I'm a bit unsure how they work at this point so I won't write about it.
