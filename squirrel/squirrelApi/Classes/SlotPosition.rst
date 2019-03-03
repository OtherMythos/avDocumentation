SlotPosition
============

The SlotPosition namespace is an extension of the c++ SlotPosition.
Many of the functions in the api return or take a SlotPosition as an argument.

SlotPositions themselves are relatively complicated, and therefore require their own wrapper class.
The SlotPositions in Squirrel follow the same rules and structure of the c++ version.

Example
^^^^^^^

.. code-block:: c

    local first = SlotPosition(1, 2);
    local second = SlotPosition(3, 4, 50, 60, 70);

    local third = first + second;
    third.toVector3(); //Translate relative to the origin.

API
^^^

.. js:function:: SlotPosition(chunkX, chunkY, posX, posY, posZ);

    :param SQInteger chunkX: The chunkX.
    :param SQInteger chunkY: The chunkY.
    :param SQFloat posX: The position x.
    :param SQFloat posY: The position y.
    :param SQFloat posZ: The position z.
    :returns: A constructed SlotPosition.

    Constructs a SlotPosition.


.. js:function:: SlotPosition(chunkX, chunkY);

    :param SQInteger chunkX: The chunkX.
    :param SQInteger chunkY: The chunkY.
    :returns: A constructed SlotPosition.

    Constructs a SlotPosition, setting the position coordinates to their default of 0.


.. js:function:: Operator+

    :returns: A new SlotPosition containing the result of the addition operation on two other SlotPositions.

    Constructs a new SlotPosition from the two used in the operation.

    .. code-block:: c

        local third = first + second;


.. js:function:: Operator-

    :returns: A new SlotPosition containing the result of the subtraction operation on two other SlotPositions.

    Constructs a new SlotPosition from the two used in the operation.

    .. code-block:: c

        local third = first - second;

.. js:function:: toVector3

    :returns: An array containing the origin respecting vector.

    This function does not modify the contents of the SlotPosition at all.
