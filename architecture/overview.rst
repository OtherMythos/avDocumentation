Architecture Overview
=====================

This section of the documentation will focus on explaining the architecture of the engine.
I'll give an overview of the majour components of the engine, as well as try and explain the design decisions I made and why they were made.

The AV engine has been designed with a focus on simplicity and maintainability, as well as being fit for a specific use case.
This is not supposed to be a general purpose game engine, and therefore I've made some decisions about what to include and what not to include.
Many of these decisions are intended to support this idea of simplicity, and maintainability.
Bear in mind that this is intended to be a project completed by a single person, and a lot of design decisions have been based around me and my limited resources.

Bird's eye view
---------------
The engine is split into a number of components.

Outline of Components
---------------------
Much of the engine is based around an event messaging system, which is supplied by the event manager.
The purpose of this is to help with the decoupling of a number of the majour components.
Rather than components talking directly to the target component, they instead favour the use of the event system, which will broadcast the event to components which have subscribed.

The largest set of components in the engine is the world, which contains the actual logic of the game world.
Inside the world are a variety of other systems which work together to provide the streamable open world present in the gameplay.
The world itself is rather complex, and involves the entity system, physics system, level streaming and also provides support for deserialisation.
The world can be shutdown and restarted during the runtime of the game, as a new world save file is requested to be loaded.

Outside of the world, the other components focus on things that aren't directly coupled with the world.
These components rarely communicate with one another directly and instead use the event system.
Components such as the Input component deals with abstracting the details of inputs to game specific actions.
So rather than space key pressed, the input system deals with jump_input_sent.
A similar thing could be said for the window component, which acts as a system abstraction for output.
For instance, it contains an SDL window for desktop environments, and could be extended to support others implementations for output on other platforms.

Specifics of the majour engine components are discussed in more detail in their own sections.
