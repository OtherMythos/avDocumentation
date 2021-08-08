Generating Meshes
=================

Producing meshes for use in the engine is an important consideration.

The avEngine makes use of Ogre3D, and as such all meshes are provided in Ogre's mesh format.
Due to this, all of Ogre's tools and documentation can be used to produce suitable meshes.

This document will explain how to setup a suitable environment for mesh creation, and explain how to export them to the engine.

Blender
^^^^^^^

Blender is the recommended application for 3d modelling.
The ogre project provides a plugin named blender2ogre, which can be downloaded from their repositories.
When installed this will enable an export option in blender to export to ogre.

In order to install this project in Linux, copy the directory ``io_ogre`` from their repository into the blender addons folder.
Consult Blender's documentation to find this location, but in linux this path is generally:

.. code-block:: bash

    ~/.config/blender/2.83/scripts/addons

``2.83`` changes depending on what version of blender you have. If this path doesn't exist for you you can create it.

It is recommended to create the ``io_ogre`` directory as a symlink, so that if you ever pull some changes from the blender2ogre repository they will be updated automatically.

.. code-block:: bash

    ln -s ~/blender2ogre/io_ogre ~/.config/blender/2.83/scripts/addons/io_ogre

The blender2ogre addon requires use of ogre's mesh tool.
This is a binary tool included with your ogre distribution that allows exporting of xml files to ogre's mesh format.

The purpose of the xml format is to provide an intermediate file type, where vertices, faces and materials are described in plain text.
These files have a ``.mesh.xml`` extension.

Ogre can't read these formats directory however. It is only able to read a compressed binary ``.mesh`` file.
The blender2ogre extension produces an xml file, and then uses the ogreMeshTool to produce the final .mesh format.

In order to use the extension, edit the file ``io_ogre/config.py``.
Find the line containing the string ``OGRETOOLS_XML_CONVERTER`` and replace it with the path to your ogreMeshTool.
The avEngine uses ogre 2.x, so the binary file will be called ogreMeshTool, not OgreXmlConverter.

The tool will be included in your built ogre distribution, under the path ``ogre/build/bin/OgreMeshTool``
Before using it in blender it is worth checking that you can run the converter without any issues.
If you have ogre installed on your path, you shouldn't need to change this config value.

Back in blender, check the new extension is loaded, by opening edit>preferences>addons, and search for ogre.
Enable this extension, and test it out by exporting a mesh.
If you have problems with the mesh exporter blender should alert you of this.

If you still have problems with the meshTool path, blender does allow you to set it using their gui, inside the extensions window.
Expand the options for the mesh tool, and you should see it there.

Every created skeleton animation is named 'my_animation'
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Name your animation whatever you want in the hierarchy view on the left.

Switch to the dope sheet view, then the action editor.
Ensure your current track is selected in the options.
There should be an option somewhere on the left with a snowflake with the word 'stash' next to it.

Pressing this will register your animation in the NLA stack.

avTools asset pipeline
^^^^^^^^^^^^^^^^^^^^^^

The asset pipeline created for the engine includes support for automatic exporting of blender files to .mesh format.

Before using it you should setup your default settings in the blender ogre exporter window.

 - Set swap axis to x-zy (ogre default)
 - Disable export materials
 - Disable export scene
