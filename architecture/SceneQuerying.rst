Scene Querying
==============

The engine allows scenes to be queried using raycasts.

Raycasting is a technique which sends a ray along a specific direction from a specific point, reporting back which objects it hits along the way.
This technique can be used for all sorts of functions from gameplay to scene editing.

.. Note::

    The user is only able to cast rays in the scene during a designated 'safe' period.
    This is the 'sceneSafeUpdate' function in the :ref:`Squirrel Entry File`.
    In this period, the scene cannot be manipulated (nodes moved, scaled. Entities moved etc).
    This is to ensure that the scene is clean and all transforms have been updated before rays are cast.

Casting a Ray in the Scene
--------------------------

The following snippet is a complete :ref:`Squirrel Entry File`.

.. code-block:: c

    function start(){
        print("Squirrel Entry file!");
    }

    function sceneSafeUpdate(){
        if(_input.getMouseButton(0)){
            local posX = _input.getMouseX().tofloat() / _window.getWidth();
            local posY = _input.getMouseY().tofloat() / _window.getHeight();

            local ray = _camera.getCameraToViewportRay(posX, posY);
            local result = _scene.testRayForSlot(ray);
            print(result);
        }
    }

This is a query of the scene, not of any physics worlds.

This code casts a ray from the current position of the camera in the scene in whatever direction it is facing.
This is useful for picking specific objects in the scene with the mouse.

Ray queries can only be called in the sceneSafeUpdate function.
This function guarantees that the scene has no dirty flags set, meaning rays can perform correctly.
Calling any functions that manipulate the scene tree will throw an error in this function.

Masked Raycasts
---------------

Raycast queries can use a masking system to limit the number of responses they return.
This can be useful if you need to exclude the number of items to be tested.

The following shows how to apply masks to objects.

.. code-block:: c

    local firstItem = _scene.createItem("cube");
    local secondItem = _scene.createItem("cube");

    //Give your objects a bitmask id.
    firstItem.setQueryFlags(1 << 6);
    secondItem.setQueryFlags(1 << 5);

Then, you can test the scene, asking only for items with the specific mask.

.. code-block:: c

    local foundPos = _scene.testRayForSlot(::createdRay, 1 << 6);

The query system is based around bitmask and.
This means that if a single bit of the object matches the mask the object will be considered by the raycast.
