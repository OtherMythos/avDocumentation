World
=====

The world is arguably the largest set of components and systems in the engine.
It acts as a base in which all the gameplay elements of the game occur.
The world is a single class that deals with the encapsulation of a range of other components.
These other components include things such as The Entities, Physics similations, World Chunk construction and loading, as well as a number of other things.
There is only intended to be a single world at a time.
When creating a world, the user is able to supply a save file with which the world should construct itself.
The save file will be used to bring the world to a state identical to when the save file was originally created.

When the world is created, it will initialise all the other components which are involved in the world and begin rendering from there.

