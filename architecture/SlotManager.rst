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

x:0 y:0

The position 500 in world coordinates would be:

x:1 y:1

Because the size of a single slot is 500x500.
In between these slots is another measurement, which I represent as an OgreVector3.

x:1 y:0 (100, 0, 100)

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
