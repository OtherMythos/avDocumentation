Entity Manager
==============
The Entity Manager is the component responsible for managing entities in the world.
The entity system is based on an EntityComponentSystem, which is provided by the library EntityX.
Quite simply, an Entity Component System is an architectural approach to dealing with entities. It favours composition of entities rather than a strict and ridgid entity hierarchy.
Entities are tied together by a list of components which represent simple aspects of the entity.
Entities are a combination of components, which can be combined to create more complex behaviour.
Components do nothing but store data, and the components themselves are manipulated by systems, which provide the functionality.

The entitiy manager is responsible for managing a lot of the core functionalities of the entity system, such as creating, destroying and moving entities.
It abstracts away a lot of the details of entityX, although doesn't try and hide them altogether.
It handles other things such as maintaining the correct composition of entities.
By this I mean making sure the entities have the expected components for what they need, and that these are managed correctly.
Some parts of maintaining the entities involves some book keeping, and the entity manager makes sure this all gets done correctly.

.. tip::
    The entityManager contains the entityx world and data structures, and should be used whenever the user needs to alter these in any way.

    Altering things like the position component of an entity manually will lead to some vital book keeping not being done correctly.

The engine does use entityX entity handles and ids throughout its functionality.
From a design point of view I didn't have a problem with this approach, as compared to trying to write my own sort of handle into the entity manager to represent entities.
This means the engine is entirely dependent on entityX and not just the EntityManager.
However this approach leads to faster querying of entities and their components.
