Slot Manager
============

The Slot Manager is the response to the problem of streaming a continuous open world.
A number of problems and issues arise when attempting to stream a large world.
These include:

 - Memory constraints
 - Floating point precision issues
 - Loading times

My solution of slots and chunks solves these problems.
There is a difference between the two.

A slot is a coordinate in the world.
A chunk is a piece of data that fits into a slot.
In order to stream an open world, you need to split it into smaller individual *chunks* of data.
This data is loaded in and presented in the world depending on where the player is at that moment.
As the player moves around, new chunks are loaded and inserted, and old ones are deleted.
In this sense the game gives the impression that the whole world is loaded at a single time, when infact it is only partly loaded.

Floating Point Precision
------------------------
A slot is a position in the world, and in my implementaton they exist to combat the problem of floating point precision.
Floating point precision is a fundamental problem in computer science, in which computers have trouble representing floating point numbers of a high magnitude.
The bigger the whole number gets, the less of a resolution the decimal number can have.
For a 3d simulation this can be detremental, and is a classic problem for simulations with a large world.
While problems might not present themselves at a closer position to the origin when the magnitude is small, in bigger areas further from the origin this becomes difficult to manage.
Ultimately it leads to peculiar results from the 3d render, or innacurate physics.
My design combats floating point issues by splitting the map into smaller sections, these sections are known as *slots*.

So slots would have a set size in world coordinates, say 500 units.
That means the origin would have a value of

``x:0 y:0``

The position 500 in world coordinates would be:

``x:1 y:1``

Because the size of a single slot is 500x500.
In between these slots is another measurement, which I represent as an OgreVector3.

``x:1 y:0 (100, 0, 100)``

This vector3 allows you to represent any position within slot space.
This coordinate system is known as *SlotPositions*.

The best thing about the SlotPosition system is that they function as an abstraction for the actual origin of the world.
This is what I used to remove the floating point problem.

Libraries like Ogre and Bullet want to know where to place things in hard floating point numbers, which are susceptable to these issues.
This means even with my coordinate system they need to map into actual positions which are within the safe range of values.
As a nieve approach I could just multiply the x and y values by the chunk size and have the position, however this would be just as useful as storing the position as an actual float value.
Instead the solution is to store a slot position to reference the origin of the world.
In this approach, none of the slot positions themselves actually change, but the floating point number they map to changes based on the origin of the world.

As the player moves around, I can move this origin as necessary.
Say they cross a threshold of 1000 world units distance from the origin, I would shift everything in the world to be positioned around where the player is, and set their current position as the origin.
This is a process known as origin shifting.
To the user there has been no change in any of the slot positions, as their actual values don't change.
However, when the engine is asked to convert a slot position to a float value this can be done taking the origin into account, thus producing a floating point position which is within the safe bounds!

So ultimately the SlotPositions are nothing more than a way to abstract the actual position in the world.
The user won't have to worry about where or what the origin of the world is, or origin shifting at all.
All the details of where things end up in the world can be handled by the engine.

Recipies and Chunks
-------------------
Recipies are a concept used by the slot manager to construct chunks.
A recipe is a collection of plain data structures which describe how to construct a chunk.
Recipies have nothing to do with an actual constructed chunk.

The reason for recipies centres around threading and blocking IO.
Much of the libraries used by the engine (bullet, ogre) have a preference of single threaded activity.
For instance, the process of creating an ogre scene node is not thread safe.
Therefore, the actual construction of the chunks needs to be done on the main thread.
This holds true for a number of other non thread safe components.
Recipies allow the threading system to load chunks into an intermediate state before their actual construction.
This helps prevent the problems posed by blocking IO, as slow file read speeds won't cause the main thread to hang.

When a chunk is requested to be constructed into the world, the recipe will first be loaded into memory before any construction happens.
Recipies are stored in memory until they are destroyed to make room for a newer recipe.
When a chunk is deconstructed, the recipe won't necessarily be destroyed along with it.
The slot manager operates by maintaining a maximum number of recipies that can be loaded at a time.
This way, if the chunk is later required to be reconstructed, and the recipe still exists in memory, then it won't have to be read in from a file again.

Scripts and loading chunks
--------------------------
The Slot Manager has been designed to ask few questions about what it's asked to load.
All of the actual logic of what needs to be loaded happens outside of the Slot Manager, with much of this being the ChunkRadiusLoader.
This is a class which calculates which chunks need to be loaded based on the radius from the player.
The Slot Manager itself simply does as it's told, only performing minimal sanity checks.

Much of the api for the Slot Manager has been intentionally not exposed to scripts.
This is because from a scripting point of view I felt the internal workings of the Slot Manager were too low level and complex.
For instance, the slot manager has no concept of chunk garbage collection, meaning if a script accidentally loads a chunk and forgets to unload it, it'll remain there for the rest of the engine runtime.

Scripts are able to tell the slot manager to load a recipe, which can be used for pre-loading areas, however there is no direct way to tell the manager to activate or construct chunks.
The intended way to control the loading of chunks is to alter the position of the player.
The player position is an abstract concept, and coupled with the entity system is easy to change.
The whole point of the slot manager is to stream the world as the player moves around it.
Components such as the chunk radius checks will react as the player position moves around and manage chunks accordingly.
This has a number of advantages, such as removing the possiblilty that scripts can activate a chunk which actually lies outside the floating point safe zone, as the safe zone will have moved as the player position moves.
In this process, by moving the player position, scripts can affect active chunks.

 - It means scripts don't have to deal with constructing and de-constructing chunks.
 - All logic of which scripts need to be loaded can be fed through the radius loader.

Lots of the more complex functionality is going to be implemented in c++ as compared to Squirrel.
Things like teleporting the player will be exposed to scripts, but actually implemented in the engine.
That means the teleportation code can take advantage of the Slot Manager's functionality with things like pre-creating a chunk.
In this way the scripts can use the flexiblity of the engine without having to expose complex system to the user.

Origin
------
The origin of the world is set in the top left corner of the coordinate system.
This means that positive x moves in the right direction, and positive z moves downwards from the slot positions.

Loading Procedure
-----------------
For much of its api the Slot Manager will perform a number of steps.
The Slot Manager uses a threaded approach for loading chunks, and because of this not all requests are completed immediately.
For instance, a request to construct a chunk might take a number of update ticks to actually display something on the screen.
This is a variable ammount of time depending on how fast the user's computer can load the content from disk.


The Slot Manager takes care of a lot of the heavy lifting for the user.
For instance you can call the activateChunk function on a chunk that has never been activated before.
The engine will deal with the procedure of loading and construction as well as activation.
This means the user doesn't have to make a call to construct chunk, and then check that the chunk is constructed first before calling activating it.

It is also entirely possible to call other api functions on chunks in between their loading cycle.
Say for example you requested to construct a chunk, and then during its loading cycle you then made a call to activate that chunk, the engine will switch the operation to perform on the chunk midway, and you will get an activated chunk in the end.
This way the user doesn't have to check if the chunk is done loading.

Chunks have a few different states that can be set before their load. These are mostly activate and construct.
A constructed chunk isn't necessarily visible, but its meshes and other content has been inserted into the world.
An activated chunk is visible, and by its nature has also been constructed.
The engine provides a number of api calls to influence how chunks are dealt with.

The recipe system implemented in the SlotManager uses a number of recipe slots.
Recipies will be loaded into these slots, when they're requested, and then replaced when the space they occupy is needed.
If more recipies are currently pending than there are recipe slots, the request will be pushed into a queue.
This queue will be depleated as recipe jobs complete.
Once a recipe has finished, and its construction requests are performed, a chunk request will be popped from the queue to take the recipe slot.
In this way hundreds of requests more than the size of the recipies list can be requested without any being lost.

The Slot Manager works by providing an output to the user as soon as it's ready and tries to be as simple as possible.

The procedure to construct a chunk is very similar to the above.
The only difference is the chunk completion request will be a construction request.
The activate request is similar because it also involves constructing the chunk.

Map Switching
-------------

The engine allows scripts to specify when to switch maps.
This is done like this:

.. code-block:: c

    _slotManager.setCurrentMap("mapName");

Maps are identified by their string name.

Switching maps involves the following procedure:

 - Destroys all tracked entities.
 - Destroys all chunks (meshes, physics shapes)

Interiors
---------

An interior is a special type of map which does not utilise the streamable world as described above.
Their purpose if an optimisation of the map switching that takes place when the player needs to go somewhere different.
They act as a pseudo map switch, where the map isn't really switched at all.
Instead the chunks of the current map are hidden, and the player is transported to a smaller scene.

The use case for these is if the player wanted to enter a house.
If the inside of the house was to be a different map, a lot of background work would need to be done so that the player could visit a relatively small area (destroying entities, destroying geometry, destroying physics shapes).
Interiors solve this problem.

So rather than a map switch happening, an interior can be activated which contains a simple scene like structure.
There are no chunks or slots within an interior, and everything is loaded up front.
This means that there is a limit on how large an interior can be, and they make no effort to address the floating point problem described above.
This is because interiors should never be that large.

The c++ and squirrel offers an api to set an interior as active.
