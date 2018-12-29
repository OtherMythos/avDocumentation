_mesh
=====
The mesh interface is used to create simple meshes and insert them into the ogre scene.
They are not intended for gameplay usage, but rather for simple debugging.
If the user wishes to see 'something on the screen' this will be a useful interface for them.

Meshes do not interract with the world in any way, and each mesh sits on a separate scene node from the root.

Example
^^^^^^^

.. code-block:: c

    local s = _mesh.createMesh("ogrehead2.mesh");
    _mesh.setPosition(s, 0, 30, 0);
    _mesh.destroyMesh(s);

API
^^^

.. js:function:: createMesh(meshName)

    :param String meshName: The name of the ogre mesh resource to be created.
    :returns: A User Data object representing the mesh.

    Create a new mesh in the ogre scene and add it to the origin.

.. js:function:: destroyMesh(mesh)

    :param UserData mesh: The mesh to destroy.

    Destroy a mesh in the scene.

.. js:function:: setPosition(mesh, x, y, z)

    :param mesh: The mesh to set the position of.

    Set the position of a mesh in the scene.
