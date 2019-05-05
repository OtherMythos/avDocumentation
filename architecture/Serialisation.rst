Serialisation
=============

In and of itself, the process of serialising (saving) a game world is a complex procedure.
It involves a number of considerations at the engine level, as well as careful planning as to what should be serialised and what shouldn't.

Serialisation in the context of games is being able to save the state of the game to disk at a given point so that it can be recovered to that state on demand.
While seeming like a simple operation, there are a number of things that needs to be considered.

 - What actually needs to be serialised and what can be left out?
 - What procedure does the game need to be aware of when starting itself up?
 - How do I make sure nothing gets duplicated (An entity was serialised, but the engine might automatically create another on map load)?
 - How do we make sure that the state the game was in when saved is identical to what would be re-created?
 - How do we deal with changes in the world data (maps) or the engine (new version supports new things)?

What needs to be serialised?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The absolute minimum possible should be serialised.
This is not just to save space on the user's hard disk.
Serialisation poses an interesting problem in what the right ammount is.

I could write some code to essentually dump the memory of the engine to disk.
On startup I would just load that into memory and carry on from where I was.

However, that would quickly become unmanageable.
What if during development something in the game was changed, i.e the contents of a map file?
Well if that map got caught up in the serialisation and was just dumped from disk then it wouldn't change.

What if some of the data is pointless to serialise?
For instance, I could serialise every shape in the physics world, but what if some of that data was nothing but static shapes defined by the map that will never change?
If they don't change what's the point of serialising them.
Furthermore, not serialising them solves the problem described above, where if something is changed in the map data, the save would update to reflect that.

Ultimately, the absolute minimum should be serialised.

Startup and duplication
^^^^^^^^^^^^^^^^^^^^^^^

On startup the game needs to be aware of what was previously saved.
Otherwise it could run into an issue of things being duplicated.
Say for instance that I had serialised a group of entities that exist in part of the map.
Something would have had to create them in the first place for them to be serialised.
This would most likely have been one of the procedures in the world, like creating entities as part of a chunk load.

The problem then would be that when this map is visited the same group of entities would be created again, which might be a duplication of what was created by loading the serialised data.
The solution in that case would be to remember which chunks were loaded at the point of serialisation, and only insert the entities described in the serialised data.

Accounting for edge cases
^^^^^^^^^^^^^^^^^^^^^^^^^

What if the player saves in a state where an entity moved somewhere strange i.e they moved somewhere as part of a cutscene.
You would expect that when the save is loaded, the entity would still be in its strange place.

That makes this an edge case, which can sometimes become difficult to account for.

Methods of Serialisation
------------------------

As described above, serialisation is a complex problem, and solutions change depending on the needs of the game.
In some games it is perfectly acceptable to designate where the player is allowed to save, and this allows the developers more control over what actually gets saved.
However if a game wants to offer more flexibility as to when the user can save, the developers have to be aware of more potential issues.

Really there are two sides to the serialisation spectrum, this being serialising everything to be sure, or not serialising anything but the bare essentials.
While the two have their benefits, really the best solution is somewhere in the middle.

Serialisation in the engine
---------------------------

The engine is required to support two methods of serialisation, manual saves and auto saves.
Realistically, the two try and achieve a similar goal, however they do this in different ways.

An autosave is sort of like a service provided by the engine.
Rather than the player having to worry about saving their progress, the engine can be made to do this automatically just to be sure.
Autosaves are carried out at specific 'checkpoints' by the user, and would always be done transparently.
For instance, during a fast travel, when approaching a momentus part of the story, or after having achieved something like first arrival in a new location.

Manual saves are saves which are performed manually by the user.
This would see them going into a menu and asking for the game to be saved.
The important thing to remember here is that the user has control of when this happens.
You might want to take complete control away from the player i.e they can't save mid way through a boss fight, however this provides a lot more flexiblity than the fixed point nature of the autosaves.

Autosaving
^^^^^^^^^^

By its nature, autosaves are performed without the request of the player.
They exist more for convenience and safety, meaning that even if the player forgets to manually save their game, something will be saved none the less.
This puts the autosaving procedure in an interesting place.
It needs to happen without the player realising, and it doesn't need to be fully accurate.
Let's say for instance that the player has just fast-travelled somewhere.
In the process of doing this the engine would have cleared its state anyway, so there's not much state about the world to remember other than 'this is where the player is going to go'.
In most cases the checkpoint system would work similarly to this.

The information about the world that can be assumed is assumed, and as little information as information is serialised.
The majour benefit of doing this is speed and simplicity.
Realistically, the player won't be particularly bothered about the exact position of entities in an autosave, because they didn't make it directly.
If the player was being attacked by entities when reaching the checkpoint, these entities would not be serialised.
In this case, if the player was to load this save, they wouldn't be exactly as they were before the save was made (i.e not being chased by entities).
This is not a problem though, as they didn't physically make the save they most likely won't notice.

This makes the autosaving procedure much simpler, and means that it can be completed very quickly.

Manual saves
^^^^^^^^^^^^

Manual saves are a more rigorous form of save.
They are requested by the user, and can be performed in a much wider array of places than their counterpart.
They essentually take a snapshot of the game as it was when the save happened.
In order to achieve this, it does a few things differently to the auto saves.

It still tries to limit what is serialised as much as possible (static meshes, physics shapes), but some things are still serialised.
Entities are serialised, and loaded back into the engine in the state they were in before.
This includes serialisation of their components and other relevant data.

Given this increased flexibility, the engine has to be careful to avoid the problems described further above.
