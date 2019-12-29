Dialog System
=============

The engine contains an implementation of a customisable and flexibile dialog system.
It is built with simplicity in mind, as well as user configurability.
Ultimately the system is built for very simple dialog, but can also be used for rudimentary scripting.


Design
------
The dialog system for the avEngine contains two main parts to its implementation, the c++ implementation, and the user implementation.

Dialog scripts themselves are written in xml.
The content which needs to be displayed, as well as its order is contained within this file.
Dialog scripts make it easy to specify a number of common dialog requirements, such as player choice, variable dialog paths, spoken text per character, so on.

A system to compile these scripts and execute them is implemented in c++.
However, in order to keep this truly data driven, a second squirrel implementation is expected to be provided by the user.
The engine assumes that the user has their own desires in how dialog should be displayed to the player, and therefore actual implementation of this process is left to them.
The engine allows the user to define a script which contains the dialog implementation.
This script should implement callback functions, which get called when something in the dialog occurs.
Say for instance a new piece of text was spoken.
A callback function is simply called passing that string as a parameter.
The user's implementation could do anything with that really, from printing it to the screen, or just printing it to the console (or both).

They are able to create any sort of dialog box design or function.
Because of this, the uses of the dialog system are quite substantial.

Dialog Scripts
--------------

The dialog system is built around dialog scripts.
These are scripts which contain the actual dialog.
Here is an example.

.. code-block:: xml

    <Dialog_Script>
    <b id = "0" sz = "5">
        <ts>The dialog starts here!</ts> <!--This is a blocking tag, meaning the dialog will block until it's unblocked by the implementation.-->
        <ts>This is some dialog.</ts> <!--ts means just a direct string. In the future <t> will mean an id in the localisation system.-->
        <ts>This is some more.</ts>
        <jmp id = "1" /> <!--Switch to dialog block 1. Execution will continue until another blocking tag is reached.-->
        <ts>This is some more.</ts> <!--This piece of dialog will never be reached, unless the jmp tag was invalid in some way-->
    </b>

    <b id = "1">
        <ts>Some other bits of dialog.</ts>
        <!--We reach the end of the dialog here, as there's nothing else left to run.-->
    </b>
    </Dialog_Script>

This example shows a few things working together.

Dialog blocks
^^^^^^^^^^^^^

A section of the code which contains a sequence of 'tags'.
Essentially these are bits of dialog.
Blocks are executed from the top to the bottom, until the end is reached.
Once the end of any block is reached the end of the dialog is reached.

However, different tags do different things, and there are more examples than just printing dialog.

Dialog Tags
^^^^^^^^^^^

Tags are different 'entries' in a dialog block.
They're named tags after xml tags.
There are a number of different tags, each with a different purpose.

In the example above two tags are seen within a block, ``ts`` and ``jmp``.
``ts`` is a piece of text. ``jmp`` is a tag to jump to a different block.
So in this example above it would start at block 0, and print out the ``ts`` entries until it reaches the ``jmp`` command.
It would then jump to block 1 and execute from its beginning.
It would keep executing until it reaches the end of the blocks, in which case the dialog finishes.

There are significantly more tags in existence than given in this example, all of which do something different.
Their combination allows the user to create all sorts of dialog.

Blocking Tags
^^^^^^^^^^^^^

Certain tags block until the command to unblock the execution is given.
For instance, the system would want to give the user bits of dialog at a time, rather than all at once.
That way they would have time to read it.
So tags such as ``ts`` block the execution of the dialog system until the unblock function is called.
