_camera
=======
The camera api gives access to functions regarding the camera.
There is a single camera in the engine, which makes it easier to call these functions, as no camera reference needs to be passed as an argument.

Example
^^^^^^^
.. code-block:: c

    _camera.setPosition(0, 0, 100);
    _camera.lookAt(0, 0, 0);

API
^^^

.. js:function:: setPosition(x, y, z)

    Set the position of the camera.

.. js:function:: lookAt(x, y, z)

    Set the camera to look at a specified position.
