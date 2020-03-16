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

Referencing variables
---------------------

The dialog system has support for reading variable values during execution.
This functionality is provided by the value registry system.

Two variable registries are known to the dialog manager.
These are global and local registries.
Global variables are those contained within the global value registry.
Local variables are stored within a value registry owned by the dialog manager.
Each time a dialog execution ends, the local registry is cleared.

Setting values in the registry looks like this:

.. code-block:: c

    _registry.set("playerName", "Wallace"); //Global
    _dialogSystem.registry.set("dogName", "Gromit"); //Local

The local registry is intended to be populated by scripts before the start of the dialog.
For instance, a script might determine where it wants the player to walk to after a certain point.
Then it can be passed to the system during execution.
Rather than having to call scripts throughout the dialog to determine things, they can just be batched at the start of the dialog.
Furthermore, this helps to prevent pollution of the global namespace.
If a value is only important to the current dialog, it can be set and forgotten about, knowing it will automatically be deleted when exection is finished.

The user is recommended to prefer local variables over global variables.

Variables are referenced in dialog scripts using either the $$ or ## notifiers, with $ being global and # being local.
A variable name is written as, $globalVarName$ or #localVarName#, with the value in between the notifiers being the variable name.
Here is an example of its usage:

.. code-block:: xml

    <Dialog_Script>
        <b id = "0">
            <ts>Hey $playerName$. How's it going?</ts>
            <ts>I've got #numApples# apples since I went and punched that apple tree.</ts>

            <ts>Well, see you later.</ts>
            <actorChangeDirection a="#targetActor#" d="#targetActorDirection#"/>
            <actorMoveTo a="$aId$" x="$x$" y="$y$" z="$z$"/>
        </b>
    </Dialog_Script>

In the above example, you can see instances of both variables in dialog text as well as in tag attributes.
Text tags can contain variables at any point in the text, and any number of them can be included.
Tag attributes only allow a single variable in the input at a time, and expects the first and final values of the string to be $ or #.

The dialog system performs error checking for variables requested by the user.
For instance, if the tag ``jmp`` requires an int for its attribute ``id``, and it receives a string instead, the dialog execution will abort with an error.
Similarly, if the system cannot find a variable with that name, it will also abort.

Tag Types
---------

Script Tags
^^^^^^^^^^^

The dialog system allows the user to execute squirrel scripts from within the execution of a dialog script.
Script tags are a non-blocking tag.
Below is an example:

.. code-block:: xml

    <Dialog_Script>
    <script path = "res://iceCreamScript.nut" id = "0"/>
    <b id = "0">
        <ts>I love rum and raisin ice cream!</ts>
        <script id = "0" func = "givePlayerIceCream" v1="$playerId$" v2="20" v3="#iceCreamType#" v4="40"/>
        <script id = "0" func = "printSomething"/>
    </b>
    </Dialog_Script>

Contents of ``iceCreamScript.nut``

.. code-block:: c

    function givePlayerIceCream(playerId, amount, iceCreamType, cost){
        print(playerId + " Got some icecream!");
    }

    function printSomething(){
        print("This is a function with no arguments!");
    }

Calling squirrel scripts requires a few things.
Firstly, the script must be pre-defined somewhere outside of a block.
This is done so that multiple tags can share the same script file.

Secondly, calling a script simply requires referencing the id of the pre-defined script, as well as a function name.

Parameters can be passed to scripts, however these are optional.
Up to four parameters are allowed.
These parameters are able to make use of the dialog registry system by passing global or local variables to the scripts.
