Physics
=======

The engine provides support for physics features, provided by Bullet physics.
This allows the implementation of both dynamic, realtime physics and collision detection.
The engine's implementation of Bullet is provided in a threaded manner, meaning good performance should be possible.

Much fo the emphasis for the physics implementation has been shifted to C++, meaning Squirrel scripts should aim to setup their physics simulation desires and let the engine fulfil them.
For instance, the collision system allows the user to specify a number of flags to determine what counts as a collision, meaning less logic has to be performed by Squirrel.
This allows the workload to be shifted onto the physics thread, and better performance to be achieved as a whole.

The engine provides two types of physics worlds to the user:

 - Dynamics world
 - Collision world

Dynamics World
--------------
The dynamics world implements a realtime dynamic physics simulation.
In this sense, bodies are inserted into a world, and interact with one another dynamically, similar to how they would in the real world.
The user is able to create bodies with specified shapes, and attach them to meshes or entities.
The attached objects will be positioned as the rigid body moves throughout the world.
Scripts have the ability to create rigid bodies, and assign them according to their needs.

Collision World
---------------
The engine implements bullet collision as collision worlds.
Collision worlds have no concept of dynamic movement. They care only about which objects are intersecting one another.
The user is able to have multiple worlds at a single time, depending on their needs for collision.
A maximum of 4 worlds is allowed, and they are specified at startup in the setup file.
These worlds employ a system of sender and receiver objects, which allow events to be passed to scripts as required.
Similarly to the dynamics world, a number of of methods are used to allow the user to specify exactly what they wish to be notified of, meaning much of the work can be shifted back to the engine and C++ code.

Script Details
--------------
There is a number of technical details regarding the script api which should be kept in mind.

Dynamics Overview
^^^^^^^^^^^^^^^^^
The following example shows how to create a rigid body, insert it into the world, and cause it to move an entity or mesh around.

.. code-block:: c

    //Create an entity
    local en = _entity.create(SlotPosition());

    //Create a cube shape with diameters 1x1x1.
    local shape = _physics.getCubeShape(1, 1, 1);
    //Create a rigid body using this shape.
    local body = _physics.dynamics.createRigidBody(shape);
    _physics.dynamics.addBody(body);

    //Assign the rigid body to the entity by providing it as a component.
    _component.rigidBody.add(en, body);
    //The user can also assign to a mesh.
    ::mesh <- _mesh.create("cube");
    mesh.attachRigidBody(body);
    //NOTE a rigid body can only be attached to one object at a time.

Collision Overview
^^^^^^^^^^^^^^^^^^
Creating objects in the collision world can be performed with the following snippet.

.. code-block:: c

    function emptyCallback(){
        print("I'm called on collision!");
    }

    //Tables are used to contain construction info, and passed to the construction functions.
    local senderTable = {
        "func" : emptyCallback
        "id" : 1, //An arbuitrary id to be assigned to the object.
        "type" : _COLLISION_PLAYER, //The type of object which this is.
        "event" : _COLLISION_INSIDE //The events it should send upon.
    };
    local receiverInfo = {
        "type" : _COLLISION_PLAYER, //For a collision to occur, this needs to match up with the type of the sender.
    };

    //Create a shape the same as the physics world.
    local cubeShape = _physics.getCubeShape(1, 1, 1);

    //When calling the collision world, the world is referenced by this id in the function.
    local receiver = _physics.collision[0].createReceiver(receiverInfo, cubeShape);
    local sender = _physics.collision[0].createSender(senderTable, cubeShape);

    //Attach the objects to the world.
    _physics.collision[0].addObject(sender);
    _physics.collision[0].addObject(receiver);

Collision senders and receivers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The collision world uses the concept of senders and receivers to perform collisions.
Only if a sender and receiver collide can an event occur.

As an example, if the user wanted to build a damage dealing projectile system, this could be easily fulfilled with the collision world.
The user might create a sender which sends fire damage. It has no need to move, but sits there and waits for a receiver, attached to the player, to touch it.
When the callback occurs, the script can apply the damage to the object.

Or, if creating a trigger system, the user might want a function to run when the player gets close enough to a door.
In this case a sphere sender would be placed by the door, waiting for the player receiver to approach it.

Scripting objects are reference counted
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
This means that when script objects such as shapes, rigidBodies and collision sender and receivers run out of references, they are destroyed.
The intention of this is to help avoid memory leaks, and make destruction of objects as simple as possible.

Collision objects which are attached to entities or meshes will gain a reference, and can be obtained later.
However, objects which are not attached and lost all references will be destroyed.

The following snippet demonstrates this:

.. code-block:: c

    //If never used for anything, this shape would be destroyed once this variable loses scope.
    local shape = _physics.getCubeShape(1, 1, 1);

    //A body has been created which references the shape.
    local body = _physics.dynamics.createRigidBody(shape);
    //Previously the shape would have been destroyed when set to something else. Now it survives while the body survives.
    shape = 0;

    //A mesh is created and the body is attached to it. The body's lifespan extends until the mesh is destroyed.
    ::mesh <- _mesh.create("cube");
    mesh.attachRigidBody(body);
    //Resetting the body will not destroy it as it has these references.
    body = 0;
    //The body now has no references. Both it and its shape are destroyed. If the shape was used by other bodies it would survive.
    mesh.detachRigidBody();

The user must specify how many collision worlds they wish to use
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Information about the collision worlds is specified in the setup file.
If the user does not specify any of this information, the engine assumes the user does not wish to use any collision worlds.

Collision world object properties
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The engine provides the user with access to a number of object properties which can be set during collision object creation.
Once set they cannot be changed.

An example is shown below.

.. code-block:: c

    function emptyCallback(){
        print("I'm called on collision!");
    }

    //Tables are used to contain construction info, and passed to the construction functions.
    local senderTable = {
        "func" : emptyCallback,
        "id" : 1, //An arbuitrary id to be assigned to the object.
        "type" : _COLLISION_PLAYER | _COLLISION_ENEMY | _COLLISION_OBJECT,
        "event" : _COLLISION_INSIDE | _COLLISION_ENTER | _COLLISION_LEAVE
    };
    local receiverInfo = {
        "type" : _COLLISION_PLAYER | _COLLISION_ENEMY //For a collision to occur, this needs to match up with the type of the sender.
    };

    local receiver = _physics.collision[0].createReceiver(receiverInfo, _physics.getCubeShape(1, 1, 1));
    local sender = _physics.collision[0].createSender(senderTable, _physics.getCubeShape(1, 1, 1));
    _physics.collision[0].addObject(sender);
    _physics.collision[0].addObject(receiver);

These parameters are provided as part of the table input.
The possible parameters are:

.. list-table::
   :widths: 50 50
   :header-rows: 1

   * - Key
     - Value
   * - type
     - The type that this sender or receiver is. This is used to refine collisions. This input is a bit mask, so an object can have multiple types.
   * - func
     - Either a squirrel closure or a string representing a function to be called when a collision occurs. If a string is provided, the "path" field is also expected to be populated.
   * - path
     - A res path to a squirrel script. If populated, the engine will use this path to determine which script to call, expecting a function with the same name as previously provided in the "func" parameter.
   * - id
     - A numeric id, provided by the user. This id is not used by the engine, and is purely a means to identify an object.

When specifying a receiver object, only the type value is used.

Reduce the number of callbacks used
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Callback scripts can be loaded once and shared between multiple objects.
For best performance, limit the number of different scripts to as few as possible.

As an example, the user might be using a collision world to perform trigger events.
The most efficient way to meet this requirement is this:

.. code-block:: c

    function playerEnter(id){
        switch(id){
            case 0:
                print("Player reached door");
                break;
            case 1:
                print("Player reached gate");
                break;
        }
    }

    function playerLeave(id){
        print("do something");
    }

In this example, the same function is used, and the sender id is used to determine how to handle the event.
This approach is much more memory efficient than assigning a unique function for each sender.

The user might assign a different script for the colliders per chunk of the game.
If colliders share the same closure and script, callback script and closure information can be shared in memory.
Generally the user should prefer to specify a script and function rather than a callback, as it is better structured for larger projects.


Threading details
-----------------

Physics in the engine is implemented using threads.
The user should be aware of this fact, as there is a chance they will meet issues as a result of the latency between the worker thread and the main thread.
Script functions to add or remove bodies, as well as functions such as set body position are subject to a delay between threads.
This delay is often only a single frame between a request being made, and it being fulfilled.

Functionality such as getting the position of objects can be performed from the main thread, as a copy is made of useful data such as this.
