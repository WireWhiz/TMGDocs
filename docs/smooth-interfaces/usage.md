---
sidebar_position: 1
---
# Usage 

## Phantom Interactables
The entire toolkit is based around a system we call "phantom interactables".

Put every interactable prefab has 3 parts:
* A root that holds the main phantom interactable script
* A visuals object, this is the visible part that moves. Only a mesh without colliders or other objects.
* A "phantom grip" object, this is a clone of the visuals object, but with a PhantomGrip script, VRCPickup, and everything needed for that VRCPickup to work.

The theory is pretty simple, the phantom grip object is invisible, and when you pick it up the interactable scripts will move the visuals object to match 
the grip object, but with added constraints. In the case of pancake fallbacks, we lock the player in place and then they can use their platform's specific 
move input to control the interactable.

![Screenshot of the interaction refs section of most smooth interfaces scripts](./images/interactionRefs.jpg)

All our scripts start with this section, handle visuals and handle grip are described more below in the section on setting up custom prefabs.

Interaction station just needs to be set to a reference of any instance of an "Interaction Station". We use this to lock 
the player in place while interacting if they're not in VR. If this isn't set, the interactable falls back to working like it would in VR, but 
like, super awkward because you're playing on a phone. These can all share the same station, or all have their own, just needs to be set to one.

:::tip
### How does the fallback mode know what wasd keys to map to what directions?
The movement input is automatically rotated based on the axis or normal of whatever interactable you're using, relative to the camera. So a horizontal 
slider will use the left/right inputs, and a vertical slider will use the up/down inputs, and looking at it diagonally will use a mix of both. 

With any rotational interactable, if the axis is straight up, it will use left/right inputs to rotate it clockwise/counterclockwise, looking at perpendicular to the axis will use up/down inputs. Looking at it from the front or back will use the forward/backward inputs. 

This is all done automatically, so you don't have to worry about it.
:::

## Flat Screen Settings

Any time you see "flat screen" it's referring to a setting for players not in VR. This is usually degrees or meters per second to move 
controls when intercepting the usual move input.

## Increments + Feedback settings 

Most of our scripts have an "increment" setting, this setting serves two purposes: 

1. Snapped/Rounded input 

All scripts with an increment setting, have an "Incremented" public property that returns the current value of a gizmo, but 
instead of being continuous it's snapped to the last increment. This is especially useful for dials you want an integer output from.

2. Audio & Haptic feedback

Additionally, every time a gizmo is moved past an increment it'll trigger both haptics, and a sound if set in the respective "Feedback" section of a script.

## Custom Prefabs & Visuals

First start with a model you want to rig and add one of the gizmo scripts (dial, wheel, lever, slider) to the base, the pivot of rotational gizmos should 
somewhere along the axis of rotation, if not you can parent it to a new empty GameObject and use that empty as the pivot. 

For all the gizmos but the lever, the rotation of this object doesn't matter, but the for the lever the forward direction must be aligned with the lever, and the x direction must be aligned with the axis of rotation.

Now clone that visual portion of the model so we have a copy for later, and then remove all colliders from the original object, set this (or the new GameObject root) as the "handleVisuals" object on the interactable script. 

Then take the clone you made before and add a phantom grip script to it. Set the "HandleGrip" property on the root interaction script to this object, and the phantom grip script properties should set themselves automatically. 

Add a collider if the object doesn't have one already, and set it to be a trigger. 

Now we want this object to be invisible, BUT we want to keep the mesh renderer as we outlines to work correctly on hover. The solution for this is to remove all materials from the mesh renderer, this will allow pickup highlights to work correctly, but it will be invisible otherwise.

Also, double-check the automatically added ridgidbody is set to kinematic, otherwise you'll be able to throw the invisible handle, and you definitely won't be able to find it again. (This should be automatically validated, but always better safe then sorry)

Reference the existing prefabs for what settings to use for the VRCPickup. 

The most important one is that an explicit grip pos must be set for wheel and lever gizmos. For dials and sliders its doesn't matter, in fact, with sliders they feel much better without an explicit grip pos set. Just depends on if you want them to retain the inital grip pos offset, or snap to the center of the hand.
