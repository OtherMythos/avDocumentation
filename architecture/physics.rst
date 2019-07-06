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

Interracting with the physics world can be done via a number of methods.
Much of the api is exposed to scripts, as they are able to flexibly create things like physics shapes, bodies, add and remove them from the world an so on.
Created rigid bodies can be attached to entities and meshes, allowing a visual representation of these bodies in the world.

An example of usage would be the following:

.. code-block:: c

    //Create a shape for the rigid body. This can be used to create an unlimited number of rigid bodies.
    local shape = _physics.getCubeShape(1, 1, 1);
    local body = _physics.dynamics.createRigidBody(shape);
    //Add it to the world. It's only considered as part of the simulation once this has happened.
    _physics.dynamics.addBody(body);

    //Attach it to an entity via a component. The entity will now be moved as the body moves.
    ::entity <- _entity.create(SlotPosition());
    _component.rigidBody.add(entity, body);


Threading
=========

.. Note::
    The following is a work in progress design.

Threading the bullet world would provide a substantial performance optimisation.

I am proposing to have an entire thread devoted to processing and updating the bullet world.
This would happen each frame, and would be responsible for updating the world and synching it up with the main thread.

Why not update the world in a job?
----------------------------------

I am speculating problems caused by the mis-match of job functionality.
Jobs are intended to be used for things which are largely 'set and forget'.
They have no deadline.
For example, loading in chunks of the map.
This would be finished when it's finished, and there should be no reliance on timing.
If it takes a year to load the chunk in then so be it (that would never happen though).
The problem would be that if the job stack was full of jobs which take a year to complete, something like the physics would become blocked.
We would want the physics to update each frame, but this is not reliable in the job system, as jobs are processed when a worker thread becomes available.

So a dedicated thread becomes the only solution.
Physics processing is such an integral part of the engine operation that it's worth giving it a devoted thread.

How is it going to be thread safe?
----------------------------------

This is obviously the main question.
Bullet by itself is not thread safe, and pretty much the entire api cannot be used in a multi-threaded manner.
My dedicated thread is going to update all three bullet worlds each frame. One of them is a dynamics world, and the rest are simple collision worlds.
I'll first describe my plan for the collision worlds as they are simpler.

Collision Worlds
================

The collision worlds simply determine collisions between objects.
Rather than stepping the world, you instead just check for collisions.
Shapes can be moved around, and the output of the collision check are collision manifolds, which describe which shapes are colliding.
That's all the main thread is interested in.

When the check for collisions is happening in a thread, nothing can change in the world.
So this means the main thread cannot have direct access to it at all.
However, the main thread will still want to insert things, or alter variables and so on.
For this reason, a work around is required.

The main thread communicates with the worlds through a command queue.
These commands are queued up until the main thread is able to process them.
This will happen at the start of the physics world's update procedure, before the world is updated.
These commands might be something like:
 - Create a physics object
 - Apply force to an object
 - Remove an object

These commands can be queued up at any point by any thread. They will be processed when the physics thread is able.
So this means writes to the physics worlds will only ever happen when it is safe.

These writing similarities are shared between all three worlds, including the dynamics world.
For the collision worlds, reading is much simpler.
As previously mentioned, the only value you might ever want to read is the collision manifolds.
The main thread only cares about the collisions, and as long as this is delivered to the main thread in a suitable manner there is no further requirements.

Dynamics World
--------------

The dynamics world is inherently more complex.
As part of the world update, ridgid bodies can move on their own accord.
These changes to the shapes need to be updated and synchronised with the main thread.

A common usage for ridgid bodies is attaching them to entities, to act as a controller object to navigate the world.
So therefore, if the ridgid body moves, the entity should move along with it.
This sort of information needs to be synchornised with the main thread.
That in itself isn't that big of a problem.

The biggest problem is wanting to do things like ray-casting in the dyanmics world.
Ultimately, ray casting is a necessity for a number of reasons.
It's useful for writing things like character controllers, or programming ai.

However, this is largely just speculation.
At this stage of the project, I can't be sure how important ray casting is going to be.
A problem I'm conerned of is locking causing problems.

I need to have a world which is always in a ready state.
That way it can be queried.

I feel like there might be a better way to do this.
For instance, I could queue up by ray casts.
Or register that I want it done each frame.
This way they could be done in batches when the world is ready.
The problem is that I'm not entirely sure at this point how valuable ray casts are going to be (although I can imagine quite a bit).
A queued system would help a lot though.
There's going to be a point in the physics world where the update starts.
If I request a ray immediately after that, it would hit a lock, and the entire thread would become redundant.
The main thread would have to wait for it to finish, and that's a problem.
These rays need to be easy to call.

So:
Determine the timeline for this (can I have the dynamics world and the collision world, rather than two collision worlds).
Can I just have a lock around the write to the collision world.
In my immediate mind, two collision worlds sounds like it might be best, but that's a lot for just raycasting.


--
I sort of came to the conclusion that rays are going to be more complex to support.
I can have a system where rays are requested and performed each frame.
For something like entities this could be done per ridgid body.
So for instance, project a ray from this position downwards.
The the results can be used later on.
But the point is, they're done pre-emptively.

The thing is, right now I need to just start on the basics.
I feel like ray casting is going to become a bigger thing further down the line, but this is a prototype.
I need to be able to add things to the world, and see them function.
Ray casting is really just a sort of enhancement for later on.

So I need to write the api to add shapes to the world, and then determine their position.
This would be having a class (cube), where you can just set it up with a shape.
Later the physics would output something that moves the cube position.
If I can see a moving cube that's good.
