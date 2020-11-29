User Entity Components
======================

To support the data-driven approach of the engine, the user is able to define their own custom components which can be assigned to entities.
These components can contain a number of user definable data types which can be get and set using the component api.

The user is able to define their components in the setup file like this:

.. code-block:: json

    "Components": {
        "health": ["int"],
        "statusAfflictions": ["int", "float", "bool", "float"],
        "coinAmount": ["int", "int", "int", "int"]
    }

When the components are defined, the user can access them like this:

.. code-block:: c

    enum Components{
        HEALTH,
        AFFLICTIONS,
        COIN
    };
    enum HealthVariables{
        VALUE,
    };

    local en = _entity.create(SlotPosition());
    _component.user[Components.HEALTH].add(en);

    _component.user[Components.HEALTH].set(en, HealthVariables.VALUE, 50);
    local health = _component.user[Components.HEALTH].get(en, HealthVariables.VALUE);

Variable types
--------------

The user has three data types to choose from, ``int``, ``float`` and ``bool``.
The user is allowed 4 variables per component. This limit is put in place for performance reasons.

Similarly, a maximum of 16 components can be defined.
Components are defined in the setup and cannot be changed during the execution of the engine.

Accessing components
--------------------

Components are id'd using a numeric identifier.

The user does not have to use enums, for example:

.. code-block:: c

    _component.user[0].set(en, 0, 50);

The above code is perfectly valid.
Components are id'd based on their index in the list when they were defined, and the same goes for their variables.
None of them are defined by string value, again for performance reasons, so the user is encouraged to use enums to make their code more descriptive.
