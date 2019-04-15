Testing Overview
================

The avEngine employs a range of testing methods to ensure built in quality (fancy term).

There are three majour types of test that the engine supports:
 - C++ unit tests
 - Squirrel integration tests
 - Squirrel unit tests

These test types are explained in more detail below.

Unit Tests
----------

The unit tests are c++ unit tests written to test components of the api.
The practice of unit testing is pretty common. They're just tests to test simple functionality of parts of the engine.

The unit tests are written using the Google testing framework.
This includes GTest and GMock.

The unit tests live in the engine repository under ``avEngine/test/unit``, and have their own cmake build system, so can be built separately from the engine.

Squirrel Integration Tests
--------------------------

These tests are a bit involved.
Integration testing is the practice of taking the individual blocks of software present in unit tests and testing them as a whole.

The integration tests themselves are actually written in the same squirrel scripting language as the scripts for the main engine, and support for them is built right into the engine.
A wide range of aspects of the engine can be tested with this flexible system, and it has its own api to perform assertions and test related actions.

Usage
^^^^^

Code I've written to test things lives in the ``avTests`` repository.
The structure of a test is a simple data directory similar to what would be distributed with an actual game.
The only difference here is that the ``avSetup.cfg`` file contains an entry like this:

.. code-block:: c

    TestMode	True
    TestName	SlotManagerActivatesChunk

This entry tells the engine that this data directory is a test, and it should start with testing mode enabled.
Testing mode changes some functionality of the engine to allow for tests to be written.
For instance, it enables the ``_test`` namespace which is where the test related script functionality lives.
However, it should not be enabled unless the user is actually intending to write a test.

From there the test has access to extra information that the engine would not normally provide, to allow checking for changes in the engine.

For instance:

.. code-block:: c

    local num = _test.slotManager.getChunkListSize();
    _test.assertEqual(1, num);
    _test.assertTrue(true == true);
    _test.assertFalse("Blah" == "BlahBlah");

That test would be able to check the contents of the slot manager chunk list and make assertions based on it.
These methods of querying are reliant on them being implemented in c++, however the function set that the engine has will grow as more tests are created.

Running Tests
^^^^^^^^^^^^^

Tests are run the exact same way that the user would run any data directory.

.. code-block:: c

    ./av ~/Documents/avTests/integration/SlotManagerTests/SlotManagerActivatesChunk/avSetup.cfg

The engine has the capability to run these tests automatically, and the result of the test will be printed to the terminal when done.

Running Multiple Tests
^^^^^^^^^^^^^^^^^^^^^^

Tests are organised into a hierarchy, which goes

 - Test Projects - Large groups of tests that try and fulfil a task (integration tests, stress tests)
 - Test Plans - Groups of tests that test a section of the code (SlotManagerTests)
 - Test Cases - Individual tests.

The engine only allows the running of individual test cases, and it has no knowledge of the upper two layers.
This is purely to keep things simple on the engine front.

Instead I have developed a python test runner application that can run entire test plans for the user, and provide the complete results at the end.
This script can be found in the ``avTools`` repository.
Information on its use can be found by running:

.. code-block:: bash

    ./testRunner.py --help

Squirrel Unit Tests
--------------------------

Squirrel unit tests are used to address the issues caused by exposing c++ functionality to squirrel.
It is very easy to break something that previously worked when manipulating the stack, and this has the potential to destroy scripts that previously worked.

The Squirrel Unit tests, as they have been dubbed, are used to test that squirrel exposed functions work and act as they always have.
