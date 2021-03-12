Sequence Animations
===================

Sequence Animations are a type of animation the user can create to bring interesting effects to their projects.
It is intended to be a generic, keyframe based approach to animating.
Users can define their animations in an easily edited XML format.
The design of the animation system is intended to be flexible and simple, and many different objects can be animated at once.

Sequence animations can be used to create anything from cutscenes to simple position and scale animations for gameplay.

Overview
--------

Below is an example of an animation defined as part of an animation file.

.. code-block:: xml

    <AnimationSequence>
        <data>
            <firstNode type='SceneNode'/>
            <secondNode type='SceneNode'/>
        </data>
        <animations>
            <animMultiPos repeat='false' end='120'>
                <t type='transform' target='0'>
                    <k t='0' position='-10, -10, 0'/>
                    <k t='120' position='0, 0, 0'/>
                </t>
                <t type='transform' target='1'>
                    <k t='0' position='10, 10, 0'/>
                    <k t='120' position='0, 0, 0'/>
                </t>
            </animMultiPos>
        </animations>
    </AnimationSequence>


Firstly, data is defined for the animation.
Each piece of data has a target type, in this case a SceneNode.
When the user goes to create their animation instance in code, they will supply it with these objects.

Secondly, the animations are created inside the 'animations' tag.
Animations are made up of 'tracks'.
Tracks effect a single piece of animation data, which is defined in the target attribute.

From there, keyframes are defined within tracks, each with a time value and a piece of data.
The pieces of data, for instance 'position' are what are actually animated.

With this multi-track system, the user can animate a number of objects as part of the same animation.

Initiating an animation is done like this:

.. code-block:: c

    //Provide the path to the animation script.
    _animation.loadAnimationFile("res://AnimationScript.xml");

    //Create the animation info. It must match what was defined for the animation.
    //This same info object can be used to construct many animation instances.
    local animationInfo = _animation.createAnimationInfo([firstNode, secondNode]);
    //Provide the name of the animation from the file and the info.
    //The animation is reference counted and destroyed when the references reach 0.
    local currentAnim = _animation.createAnimation("animMultiPos", animationInfo);

A component also exists to assign an animation to an entity.
With this the animation will persist while the entity exists.

.. code-block:: c

    _component.animation.add(entity, animationInstance);

