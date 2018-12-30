World
=====

The world is arguably the largest set of components and systems in the engine.
It acts as a base in which all the gameplay elements of the game occur.
The world is a single class that deals with the encapsulation of a range of other components.
These other components include things such as The Entities, Physics similations, World Chunk construction and loading, as well as a number of other things.
There is only intended to be a single world at a time.
When creating a world, the user is able to supply a save file with which the world should construct itself.
The save file will be used to bring the world to a state identical to when the save file was originally created.

When the world is created, it will initialise all the other components which are involved in the world and begin loading content from there.

Creation
--------
The world is constructed as a singleton.
I made the decision to only allow a single world in the engine at a time, as it made no sense for there to be multiple.
Multiple worlds would lead to conflicting renderings, and general cluttered code.
Furthermore, there is no forseeable use case for the user wanting to create multiple worlds, and thus is pointless to support it in the code.

The singleton component is responsible for the creation and destruction of the world, and works by receiving requests from the user.

.. code-block:: c++

    World* w = WorldSingleton::createWorld();
    WorldSingleton::destroyWorld();

The reason for these decisions is flexibility.
Rather than creating a single world at the startup of engine, I realised it would be better to allow the user to specify when they wanted the world created.
Consider for example a menu system that's presented to the player on startup.
This menu would show a title screen, then allow them to select a save.
Up until they've selected the save there would be no need for the world, and now that a save has been selected the world can be constructed with that save in mind.
Now remember that the game will most likely want to have a 'quit to menu' option.
In this case, the world would be destroyed to make room for another.
The user might go to the menu screen and select another save, and the process would continue as before.

The world is therefore required to be very flexible in its creation and construction, as it could happen an unlimited ammount of times during runtime.
In order to create a new world, the old one must first be destroyed.

Access
------
The world is able to be accessed as a singleton, as you might expect.

.. code-block:: c++

    WorldSingleton::getWorld();

While this is possible, it should be noted it is **not the preferred way to access the world**.
Global access to a variable is generally not preferred, and should be avoided when possible.
This global access exists mainly for the script exposure, as much of the script api wishes to access the functionality of the world.
Without this exposure the script manager would need to be injected with the new pointers whenever the world changes.

The best way to communicate with the world would be by passing pointers, or by using the event system.
Much of the world will be encapsulated within itself, and thus can be programmed with pointers specific to that world instance, without the worry of having to keep them updated.

