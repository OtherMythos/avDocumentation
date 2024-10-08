Squirrel Overview
=================

Squirrel is a scripting language library used by the av engine.
It is leveraged to provide a flexible and powerful facility for data driven interaction with the engine.
A number of functionalities of the engine are exposed through the scripting language.
It allows the implementation of gameplay aspects without the need to touch any of the c++ code or recompile the engine.

The engine is entirely built around squirrel as a means for implementing functionality and heavily encourages its use.

APIs and Functionality
----------------------
The engine exposes apis for squirrel to call.
These act as handles to c++ functions, which allows c++ functions to be called by scripts.

The api has been designed in a way which is easy to use but is still powerful in its capabilities.
The apis are exposed to squirrel using a namespace approach.
Api calls will look something like this:

.. code-block:: c

    _mesh.createMesh("ogrehead2.mesh");
    _camera.setPosition(0, 0, 0);
    local e = _entity.create();

As you can see, the api wraps individual calls behind namespaces such as ``_mesh`` or ``_camera``.
The intention of this is to group the function calls together into their own individual namespaces.
This would prevent me from having to otherwise write something like ``_meshCreateMesh``, which looks ugly.

.. tip::
    Each namespace is prefixed by an ``_``. The purpose of this is to make it clear that this is indeed a function call rather than anything else.

    It also helps to prevent name collisions. Consider this example:

    .. code-block:: c

        local mesh = _mesh.createMesh("");

    In that example a mesh is created with the name mesh. That's quite a common name, but now consider that I didn't include the ``_``.

    .. code-block:: c

        local mesh = mesh.createMesh("");

    We now have a collision, as mesh has been re-assigned to mean a reference to a mesh. The entire namespace is now broken.
    The ``_`` character helps make sure this sort of situation never arises.

.. warning::
    You might be thinking that calling:

    .. code-block:: c

        local _mesh = _mesh.createMesh("");

    Would do the same thing, and you'd be right! At the moment there is no protection against overriding the namespace, so please try and avoid it.
    Not to mention that if one script does it the whole thing will break for other scripts as well. So don't do that.

All of the apis documented in the further sections follow a similar approach.
You can refer to the title of the page to find out which namespace those functions fall under.

Engine Specific Types
---------------------

SlotPosition
^^^^^^^^^^^^

The engine exposes a few common built in types to squirrel.
These can be used within squirrel as classes, and constructed in a way that is easy to understand.
For example:

.. code-block:: c

    local position = SlotPosition(1, 2, 10, 20, 30);

The above line of code will create an instance of a SlotPosition.
This SlotPosition behaves identically to the SlotPosition found in the c++.
It follows the same rules of overflow and underflow, as well as the origin respecting conversion.

.. code-block:: c

    local first = SlotPosition(1, 2); //Just the slot coordinates.
    local second = SlotPosition(3, 4, 50, 60, 70); //Slot coordinates and a position.

    local third = first + second;
    third.toVector3(); //Translate relative to the origin.

Vector3
^^^^^^^

Most of the time the engine does not provide a means to represent a vector.
This is purley for the purpose of efficiency.
Wrapping functionality around a class can become quickly convoluted, and for something simple like a vector, an array for representation is much more efficient.
The SlotPosition requires its own class and container because in reality a slot position is a much more complex data object than a vector.
Sanity checks and shifting is necessary for SlotPositions, while not necessary for vectors.

Furthermore, the engine often times will take plain values for function parameters rather than something like a vector3 object.
This is again for the sake of efficiency, as providing three floats to represent a vector3 is much more efficient than providing a wrapper class.

.. code-block:: c

    local result = SlotPosition(1, 2, 10, 20, 30); //Here there is no separator between the slot positions and the actual positions.
    local vec = first.toVector3(); //Returns an array.

    print("x: " + vec[0]);
    print("y: " + vec[1]);
    print("z: " + vec[2]);

Squirrel Entry File
-------------------

The first script executed is the squrrel entry file. This script is responsible for the startup of the engine, and is therefore very important.

The entry file is provided based on information in the ``avSetup.cfg`` file.
For more information please see :ref:`squirrel-entry-file`.

Scripted States
---------------

Scripted states are a way in which the user can specify scripts to run.
Say for instance the user was in a situation where they had a script they wanted run each frame for a little while.
For instance, if the weather in the game became rainy.
If in this situation some sort of water based entity should start spawning, how would that be implemented?

Scripted states solve this problem.
With a scripted state you can specify a script to be run each frame.
The user has complete control over when the state starts and when it ends.
The script will be run until the state is ended.
In the case of the rain example, the state would be started when the rain starts, the script would create monsters as it runs, and then stop when it ends.

Scripts work by providing a name for the state and a script to run.
This script has three functions which are run, ``start()``, ``update()`` and ``end()``.
These functions are expected to be defined within the provided script file.
Start is called once when the state begins, end is called once when it ends.
Update is called each update tick until the state is ended.

So in the above described example, update would generate random numbers to decide when to generate monsters,
and end would kill them all off horribly, because they can't survive without the rain.
If code for start is not needed, this function can be omitted. The same applies to the other functions.

There are a number of places where states become useful, for instance boss fights might want a state to run until the boss ends.
This state might print things to the screen, or generate hazards for the player.

If the player was in one of those levels where if you walk into an area you get found out, a state could be made to keep track of whether the player has gone there yet.

The flexibility of being able to specify a script for the task makes the system very powerful.
As a matter of fact, the squirrel entry file is being run as a state, meaning you can define the start, update and end functions in your script and have them run.
This is a state known as the ``EngineState``, and is the only state which cannot be ended manually.
It will be started on engine start, and ended on engine shutdown.
If there are any states left running during engine shutdown, they will be ended.

Script File Types
-----------------

The c++ has two different ways to compile and execute a squirrel file. These are the ``Script`` class and the ``CallbackScript`` class.
These two classes are used to compile squirrel files and expose them to the c++, but they do this in different ways.
The Script class compiles scripts as a single executable.
The Callback script compiles them into multiple closures, which can be individually executed on request.
The Callback script class is used to perform the various callback operations that the engine performs.
