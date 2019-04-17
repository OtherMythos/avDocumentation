Entity Manager
==============

.. note::

    The following is a re-design of the old entity system.

    It is not fully implemented yet!


The Entity Manager is the part of the engine responsible for managing entities in the world.
The entity system is based on an EntityComponentSystem, which is provided by the library EntityX.
Quite simply, an Entity Component System is an architectural approach to dealing with entities. It favours composition of entities rather than a strict and ridgid entity hierarchy.
Entities are tied together by a list of components which represent simple aspects of the entity.
Entities are a combination of components, which can be combined to create more complex behaviour.
Components do nothing but store data, and the components themselves are manipulated by systems, which provide the functionality.

The Entity Manager itself ties together a number of parts of the engine.
It aims to be the governing body over entities, and plays a vital role in their creation, destruction and modification.
Outside of the entity part of the engine, the fact that the entity component system is implemented by entity x is unknown.
I aim to have no includes from entity x outside of the entity system.
This is intentional, as it means I can fully lock down the functionality of the system.

Functions of the Entity Manager
-------------------------------

The Entity Manager itself performs a number of jobs.

These jobs include:
 - Creating and destroying entities, and performing the necessary bookeeping
 - Positioning and moving entities
 - Coordinating the operation of the systems.

There are a number of functions which are delegated off to other parts of the system, such as managing the entity chunks.
From the perspective of the user working outside the entity orientated part of the engine, there is no mention of entity x.
This helps to prevent interference from parts of the engine which shouldn't be interfering.
For example, in the prototype of this system, I did much of my work by passing around entity handles.
However with a handle you have complete access to the entity and its components.
With this I could have destroyed the position component, or directly altered it without letting the entity manager know.
In this style of working, there were no solid guarantees as to how things operated.

In this new system, there is no direct access to the entities or their components.
Everything is passed through designated c++ functions which serve to facilitate the manipulation of components.

Creating Entities
^^^^^^^^^^^^^^^^^

Entities are created with a single component by default.
This is the position component, which is a simple component which contains a slot position representing where the entity resides in the world.
All entities in the system have a position, and the user doesn't have to worry about creating it either, as that is handled by the engine on entity creation.

There is no direct access to the position component, instead the user must use the supplied ``setPosition()`` function in the entity manager.
This will take care of all the necessary bookeeping when moving an entity (such as moving meshes to reflect the new position, and so on).

Iterating all entities with a position component is an effective way to iterate all entities in the world.

Component Manipulation
^^^^^^^^^^^^^^^^^^^^^^

Component manipulation is an important part of the entity process.
Components contain plain data, but this data needs to be read and written to.
As I don't directly expose these components to the user, how does this work?

Instead, I have a series of static functions implemented in c++ which deal with altering the components.
If a health component had an int health value, I would have a static c++ class which contains functions to get and set the health value.
This process would be repeated for each component the engine offers.
In this way I don't offer direct access to the components, but still allow them to be read and modified.

The disadvantage of this approach is the complexity.
There is an increased ammount of code that needs to be written for each component.
Furthermore the lack of direct access could be potentially seen as a performance issue, as external functions need to be called to do simple things.

However, there are some majour benefits to this approach.
The first and most substantial is the ability to entirely police the modifications to the components.
As mentioned previously, some components are there for the benefit of the engine, rather than the user, for instance the position component.
In the old system there was nothing to stop the user from modifying the values in this component to whatever they wanted.
However, the engine often wanted to do some vital bookeeping whenever an entity was moved (moving meshes, moving physics shapes).
Because of this direct access I wasn't able to guarantee that the positions in the component matched everything else.

Consider another example, which would be the health component as mentioned before.
It contains a single health value, but surely if this value was less than or equal to 0 the entity should be destroyed?
That was how I wanted the system to work, but how would it know this?
As before I was just editing the value in the entity.
If I wanted to check if the entity had recently died, I would have to either include the checks where the value was set, or have a system that checks the value each frame.
Both are untidy solutions.
However, with an exposed c++ function I can do all the checks there, as there will only ever be one place that sets the value of the component.

This is also a majour benefit for the scripting system, as now it has a single place to call to do simple manipulations.
Previously the scripts would have had to contain their own checks per exposed function, which would have been a duplication of code.

The Player Component
^^^^^^^^^^^^^^^^^^^^

The player component is an important component for the engine.
The engine itself has no solid definition of what the player should be, or how it should act, but the position component does help it keep track of which entity represents the player.
In and of itself the position component is an empty component which does nothing.
It is mearly used by the engine to designate the player.

There can only be one entity at a time with the player component.
The user has no direct access to the component, except for the ``setPlayer()`` function in the entity manager.
This will give the player component to that entity, and remove it from any other entity which had it until then.

Entity Tracking
---------------

Entity Tracking is the method I use to manage entity lifetime.
Say for instance an entity was created in the world, when would I want it to be destroyed?
As the engine supports an open world game, that would most likely be when the player goes far enough away from that entity.
It is important to make sure that entities are managed, as in some cases the engine can add them to the world automatically.
It would make sense therefore that they can also be removed automatically.

That is the solution that entity tracking provides.
Entities are sorted into entity chunks.
These chunks are very similar to slots in the Slot Manager, in that they represent sections of the world.
An Entity Chunk is essentually a section of the map, that contains a list of which entities reside within it.
The player has a load radius, and if they move far enough away from that chunk, it is destroyed.

Chunks are created lazily, meaning that if there are no entities in that part of the map a chunk won't be created.
If an entity is inserted into an area of the map that has no entities, a chunk will be created to contain it.
If the final entity in a chunk is removed, the chunk and its list won't be destroyed until it goes out of range.
If the map switches, all tracked entities are destroyed.

Upon entity creation, the entity manager asks the user whether they want this entity to be tracked or not.
Essentually this decision boils down to, 'Do you want me to deal with the entity's deletion, or do you want to be responsible for it?'
If the entity is not tracked by the engine, it will not be deleted automatically.
That becomes the user's job.

The engine preferes to manage entities itself though.
Entities created as part of a chunk creation are tracked by default.

The EntityManager exposes an api to track and untrack entities.
The squirrel scripts also have access to an api to track and untrack entities by an eId.
Really, the only time an entity should be untracked is when they're being used for something specific in a script, such as a cutscene.

If an entity is untracked, it will not disappear until something manually destroys it.
Furthermore, it will persist engine serialisation, meaning you might be stuck with it forever.

So please make sure that if you come to untrack an entity it is eventually destroyed.
