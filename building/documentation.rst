Build the Documentation
=======================

This documentation is stored in the ``avDocumentation`` repository.
It can be built with the following steps.

For a Debian based system:

.. code-block:: bash

    sudo apt install python3-sphinx python3-sphinx-rtd-theme
    cd avDocumentation
    make html

The results of the build are placed in ``_build``.
