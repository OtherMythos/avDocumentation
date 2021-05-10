Build the Engine
================

Building the engine from source involves a number of steps.
A correct environment needs to be setup so the build can complete properly.
The project provides a few solutions to this.

Dockerised Build
----------------

A container is available to build the engine dependencies on Linux.
This container is based on Ubuntu, and is generally intended to build the appimage.
The produced built dependencies may not be suitable for a different host system, for instance Fedora.

This is stored in the ``avBuild`` repo.
The user must ensure their host operating system has docker installed and can run containers.

The dependencies can be built with the following script:

.. code-block:: bash

    #You only need to build the container once.
    ./build.sh
    #Where ~/buildDir is your directory.
    ./start.sh ~/buildDir

Local Build
-----------

Building without the container is also possible.
This is recommended if attempting to build the engine as something other than an appimage.

In the ``avBuild`` repository is the ``linuxBuild`` scripts directory.

In there ``build.sh`` will perform the same steps as within the docker container.

.. code-block:: bash

    ./linuxBuild/build.sh ~/buildDir