---
sidebar_position: 3
---
# Smooth Interfaces

Smooth Interactions is a VRChat toolkit for what it says in the name, high quality native VR interactions.
Ever wanted to actually turn a dial, or pull a lever in your VRC world? Now you can.

And, even though the focus of this toolkit is VR, we provide a no-effort desktop/mobile fallback, so nobody is left out.

## Included Prefabs:

Interaction Examples:
* Dial
* Wheel
* Lever
* Slider

Required in scene:
* Interaction Station
    - Used to lock a non-VR player in place, so we can use intercept move inputs

## Setup

To set up a scene to work with smooth interfaces, all you need to do is drop in a `Interaction Station` prefab to allow non-VR interactions to work.
After that, all prefabs should be easy to drop-in and start using right away!

## Events

For basic usage, smooth interfaces doesn't require any scripting, we have custom Udon compatible UnityEvents set up on all our scripts!

As an example, you could use a lever to drive a door animation without touching either U# or Graph:
![Example showing an animator driven by lever events](./images/DoorOpenExample.jpg)

For more info on how these events work, check out our docs on [Trigger Seeker Events](../trigger-seeker/events.md)

## Performance

All of our scripts are "reactive" and only run code when triggered by an outside event, such as an interaction. 

In fact if you do a search on our codebase there's not a single Update() or LateUpdate() in any of our scripts, so no matter how many you add to your world it has no impact on your CPU.
