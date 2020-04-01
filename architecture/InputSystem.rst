Input System
=============

The engine contains quite sophisticated support for input devices.
As well as the traditional keyboard and mouse, it also contains support for game controllers.
This support is implemented by taking an action orientated approach.

Overview
--------

To use the system, there are a few steps that should be followed.

Firstly the actions should be defined:

.. code-block:: c

    _input.setActionSets({
        "actionSet" : {
            "StickPadGyro" : {
                "Move": "#Action_Move",
                "Camera": "#Action_Camera"
            },
            "AnalogTrigger":{
                "Fire":"#Action_Fire"
            },
            "Buttons" : {
                "Jump": "#Action_Jump",
                "Reload": "#Action_Reload",
            }
        }
    });

Secondly, they should be mapped to controller or keyboard inputs:

.. code-block:: c

    //Obtain action handles
    ::JumpHandle <- _input.getButtonActionHandle("Jump");
    ::ReloadHandle <- _input.getButtonActionHandle("Reload");
    ::FireHandle <- _input.getTriggerActionHandle("Fire");
    ::MoveHandle <- _input.getAxisActionHandle("Move");
    ::CameraHandle <- _input.getAxisActionHandle("Camera");

    //Map keyboard input
    _input.mapKeyboardInput(0, ::JumpHandle);
    _input.mapKeyboardInput(10, ::FireHandle);
    _input.mapKeyboardInputAxis(50, 51, 52, 53, ::MoveHandle); //Designate four keys to represent the axis directions.

    //Map controller input
    _input.mapControllerInput(0, ::JumpHandle); //A Button
    _input.mapControllerInput(1, ::ReloadHandle); //B Button
    _input.mapControllerInput(0, ::FireHandle); //Left Trigger
    _input.mapControllerInput(0, ::MoveHandle); //Left Stick
    _input.mapControllerInput(1, ::MoveHandle); //Right Stick

Finally, the contents of these actions can be queried.

.. code-block:: c

    //Returns bool values representing a button press.
    local hasJumped = _input.getButtonAction(::JumpHandle);
    local shouldReload = _input.getButtonAction(::ReloadHandle);

    //Returns a float between 0 and 1 of how far pressed the trigger is.
    local fireVelocity = _input.getTriggerAction(::FireHandle);

    //Returns a single float each between -1 and 0, with 0 being the middle.
    local moveX = _input.getAxisActionX(::MoveHandle);
    local moveY = _input.getAxisActionY(::MoveHandle);


Mapping Keyboards
-----------------

While the items within the action sets are named after parts of a controller, the keyboard can still be used to send inputs.
This includes buttons, triggers and axises.
However it should be noted that the keyboard keys send very rigid values.
For instance, when mapping a trigger, the value set will be either 0 or 1. There is no in-between.
A similar thing occurs for axises.

Querying Specific Devices
-------------------------

Devices are the name given to a source of input, for instance a game controller.
The engine supports up to four devices at once.
Controllers can be dynamically added and removed during runtime.

Each device is given an id between 0 and 3.
The keyboard has its own id.
As one device is freed up, it's id can be re-used.

Querying a device looks like this:

.. code-block:: c

    _input.getButtonAction(::AButton, 0); //Query device 0
    _input.getButtonAction(::AButton, 1); //Query device 1
    //Given how it's checking a different device, one might return true and one might return false.
    _input.getButtonAction(::AButton, _KEYBOARD_INPUT_DEVICE); //Query the keyboard.

    _input.getButtonAction(::AButton); //Not specifying a device will default to the any device.

The Any Device
--------------

The Any Device is the name given to the device which contains data from all devices.
Say for instance the user doesn't want to query specific devices for their input.
If creating a single player game, they might instead want to listen for any input on a controller or the keyboard.
In this case, the any device facilitates this.
Any input pressed on any device will be contributed to the any device, with some notes.

Firstly, a button on the any device will return true if any of its contributing devices are true.
That is, if the same button is pressed on two controllers, querying the associate action from the any device will return true.
Then, if one of the controllers has its button released, it will still return true, as long as a single button is pressed.

At the moment axises and triggers are based on which device most recently set a value.
In future that might change.
