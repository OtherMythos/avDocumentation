Physics
=======
The world itself contains a number of bullet physics worlds.
Currently there are three bullet worlds, each intended to serve a different purpose.

 - The dynamics world
 - The Trigger world
 - The damage world

The dynamics world is a btDynamicWorld, which deals with simulating rigid bodies and dynamic movement in the world.
The other two are btCollisionWorlds, they only deal with detecting and processing collisions between shapes.

Dynamics World
--------------
The dynamics world is a btDynamicWorld.
This is a world that involves more conventional physics with rigid bodies and all that sort of thing.
It's used extensively for things like entity controllers and collision in general, and is of vital importance for the function of the engine.

The dynamics world is filled with a number of shapes at chunk load, for instance shapes to represent the terrain surface or other collidable things.
Much of these things are static, however it is quite possible for bodies to be inserted into the world which are not.

Entites use their rigid bodies to determine where they should be positioned, and when an entity has a rigidBodyComponent they will be re-positioned each frame to reflect the movements of their body.

Trigger World
-------------
The trigger world is for detecting when entities enter or leave certain trigger objects.
These trigger objects could be a variety of things, for instance:

 - Script callback triggers
 - Action triggers (Provides a button to press to do something)
 - Sound emission triggers
 - Ai trigger events

Essentually the trigger will just allow the user to place shapes about their world that are invisible to the player.
Currently, the system works by fitting a shape to the frong the player.
When this trigger collides with other shapes, the appropriate actions described by the colliding triggers can be performed,
although this could be extended to include support for other entities as well.

Damage World
------------
The damage world is for detecting when a shape moves into a damage dealing trigger.
This system is mostly used to process the damage to be dealt to entities, as they are really all that's in the world which has a concept of 'health'.

The purpose of having a separate world is to open up possibilities for more interesting forms of damage.
For instance, an area where if the player touches it they take damage (fire, poison swamp, etc)
All the damage world really cares about is the shapes.
It splits the shapes into two categories, senders and receivers.
Senders would be things like the static shapes which describe the fire on the ground.
Senders contain data about what sort of damage they deal, which ties more into the damage system.
Receivers reference an entity which is to be damaged when it intercepts a sender.
The damage will be calculated based on the data in the sender.

Damage senders are intended to be used in a number of ways.
This could include as static shapes, or as attached to something, for instance the blade of a weapon or a projectile.
In this way I'm able to take advantage of the full power of bullet to create even more realistic combat physics.

.. Note::
    I made the decision not to combine the damage world and the trigger world because I felt they conflicted too much in their intentions.
    Even though they're both the same bullet collision worlds their actual functionality differs in noticable ways.
    These different worlds function based on numbers of shapes, and the more shapes, the higher chance of multiple collisions needing to be dealt with each tick.

    Splitting the world up prevents this problem nicely and helps to make the code cleaner, as I don't have to take into account what type of object has just been collided with.

Usage
-----
Interracting with the physics worlds is done via entity components.
Components such as the rigidBodyComponent can be created to give that entity a presence in the dynamics world.
By doing this they will be taken into account when during any physics simulations, and in some cases this will effect them later on (rigidBody moving the entity around).

For some entity configurations, an entity having a shape in a physics world is necessary, for instance the player controller requires a shape in the physics world to detect collisions with things.
