_world
======
Interface for creating and destroying the world.

The engine provides flexibility as to when the user wishes to create a world, and by default one is not created until the ``createWorld()`` function is called from a script.
The world itself is stored as a singleton, meaning there can only be one instance of it at a time.
If you wish to create a new world, the old one must be destroyed first.

Example
^^^^^^^

.. code-block:: c

    _world.createWorld();
    _world.destroyWorld();

API
^^^

.. js:function:: createWorld()

    :returns: True if the world was created successfully, false if not.

    Creates a new world. This function will fail and no world will be created if one already exists.

.. js:function:: destroyWorld()

    :returns: True if the world was destroyed successfully, false if not.

    Destroy the world. This function will fail if no world exists.
