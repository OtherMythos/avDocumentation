.. AV Engine Documentation documentation master file, created by
   sphinx-quickstart on Wed Dec  5 21:01:25 2018.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

AV Engine Documentation
===================================================

Welcome to the documentation for the AV Engine.
Feel free to look around! It's all very exciting.

The AV engine is a game engine written in C++ that I've been writing in my free time, as part of a larger project.
The eventual goal and purpose of the engine is to support an as of yet unrealeased game.
This documentation aims to serve as a means for me to document how everything works and fits together, as well as how to use the functions of the engine itself.

The engine is intended to support a game with features such as:
 - 3D graphics
 - A streamable open world
 - Explorative style RPG
 - Top down style gameplay where the camera follows the player
 - Extensibility via scripts and mods
 - Cross Platform

Features of the engine include:
 - Implementation of a streamable open world
 - Realtime physics from bullet
 - An Entity Component System provided by EntityX
 - A heavy focus on scripting and data driven content
 - Support for serialisation
 - Abstraction of platform specifics

As well as this I've tried to architect the engine as cleanly as possible, both as a learning experience and to increase its future maintainability.
The engine tries to be as data driven as it can, while still trying to be as optimised as possible.

Much of the engine is based on external libraries. These include:
 - Ogre3D
 - Bullet Physics
 - EntityX
 - Squirrel (Scripting Language)
 - SDL for desktop windowing
 - A few smaller utility libraries

Here's a friendly list of topics (unless I don't like you, in which case it's an unfriendly list):

.. toctree::
    :maxdepth: 2
    :caption: Contents:
    :name: sec-Architecture

    architecture/index
    CreatingContent/index
    setup/index
    squirrel/index
    Test/index
