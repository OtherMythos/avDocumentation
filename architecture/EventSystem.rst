Event System
============

Events are used in the engine to keep track of when something of interest happens.
The engine allows scripts to take advantage of this system to call code when they need to.
There are two types of event in the engine, system events and user events.

System events are provided by the engine, and are utility things like 'WorldCreated', 'InputDeviceAdded', 'MapChanged', to name a few.

User events are arbuitrary events which can be defined by the user.
Any integer id can be given to reference the event, and some arbuitrary data can be provided.
The user would want to use user events for things like 'PlayerDied' or 'TankDestroyed'.

Subscribing to System Events
----------------------------

.. code-block:: c

    function windowResize(id, data){
        print("window resize");
        print(data.width);
        print(data.height);
    }

    _event.subscribe(_EVENT_SYSTEM_WINDOW_RESIZE, windowResize);
    //Unsubscribe that function from that event.
    _event.unsubscribe(_EVENT_SYSTEM_WINDOW_RESIZE, windowResize);

System events come with a pre-set value.
The data provided by the callback function changes depending on the event.

Only one function can be assigned as the receiver for each event type.
Attempting to set a second will override the previous.

Subscribing to User Events
----------------------------

User events are subscribed and transmitted like this:

.. code-block:: c

    enum EVENT{
        PLAYER_WEAPON_CHANGED
    };

    function weaponChanged(id, value){
        __playerHealth.setText("Weapon changed to: " + value);
    }

    _event.subscribe(EVENT.PLAYER_WEAPON_CHANGED, weaponChange);
    _event.unsubscribe(EVENT.PLAYER_WEAPON_CHANGED, weaponChange);

    _event.transmit(EVENT.PLAYER_WEAPON_CHANGED, _currentWeapon);

The enum is not necessary, although it is quite useful for code quality.
Unlike system events, any number of functions can be assigned to a user event type.

During transmission, any data type can be passed as the second option, including null.