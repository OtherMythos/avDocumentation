_input
=======
The input namespace provides access to the simple input system used to detect key presses.

Example
^^^^^^^
.. code-block:: c

    if(_input.getKey(_KeyUp)) print("Wow key is up");

API
^^^

.. js:function:: getKey(keyId)

    Get the boolean value of the specified key.
