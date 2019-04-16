_entity
=======
The Entity namespace provides access to the Entity Manager.
Operating on and effecting entities largely happens in the component namespace or through the EntityClass.

Example
^^^^^^^
.. code-block:: c

    local en = _entity.create(SlotPosition());
    en.setPosition(SlotPosition(0, 0, 100, 100, 100));
    _component.mesh.add(en, "ogrehead2.mesh");

API
^^^

.. js:function:: create(SlotPosition)

    Create an entity at the specified position.
