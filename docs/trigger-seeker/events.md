
# Events

When putting a scene together, you don't always want to write an entire script to connect two objects together. Sometimes you just want to hook existing scripts together. That's what events are for.

TriggerSeeker Events are a recreation of UntiyEvents that are compatible with Udon.

The UI looks a little messy at first, but if you're familiar with traditional UnityEvents or CyanTrigger you'll pick it up quickly.

## Fields
![Annotated Event UI Screenshot](./images/WheelRotationExampleAnnotated.jpg)

### #1 GameObject reference

This is used to collect the list of functions/events for #2

### #2 Fuction/Event

What function or event to call.

#### We support:
1. Most native Unity functions exposed to Udon
2. Custom UdonEvents
3. UdonSharp Methods (Including parameters)

:::tip
If you've used any form of Udon scripting, you're probbably going to notice quickly that `UdonBehaviour.SendCustomEvent()` does *NOT* show up when selecting a function.

This is because custom events show up as functions in that same dropdown, right at the top. They're grabbed directly from the compiled output of any program executed by an UdonBehaviour, so we support events from ALL program sources, including 3rd party ones like CyanTrigger. However, arguments are only supported for UdonSharpProgramAssets.

Tired of typing in an exact event name every time? You'll never need too in a trigger seeker event.
:::

### #3 Value Source

When a function or event accepts input, you can choose either to use a custom value, or an event argument.

In this example, `OnValueChanged(Single)` has one argument representing a new value for a wheel gismo's rotation, so we may want to pass that to a function instead of an unchanging custom variable. In this case, I set the `rotation` variable of an UdonGraph to that changed value, before calling the `ReportWheelRotation` custom event.

:::tip
`Single` is CSharp's internal name for `float`
:::

### #4 Value input

Here's where you type in those custom values, if you so choose.

Types currently supported:

* `object`
* `bool`
* `int`
* `float`
* `string`
* `Vector2`
* `Vector3`
* `Vector4`
* `UnityEngine.Object`

For `object` parameters you get an extra dropdown to choose which one of the types listed above you want to see UI for.

:::note
The function/event dropdown will only show functions that accept types supported by our UI, so if a function you want isn't showing up, it likely requires and unsupported type. Let us know if this becomes an issue, and we can add more.
:::

### #5 and #6

The secret to the way our events work, is a small custom Udon compiler. (Yes, an entire compiler just for an Event UI.)

The asset reference (field #5 in the diagram) is the UdonProgramAsset used to generate the event's code. 

Any TriggerSeekerEvent property on a script is secretly an entire hidden udon behaviour on the same game object, set to use that program asset.

The delete button (field #6) deletes that hidden UdonBehaviour, disabling the event.

## Using TriggerSeekerEvents in custom scripts

Adding a TriggerSeekerEvent to a script is as easy as 3 lines of code, doesn't even require a custom editor.

To add a public event property, create a public UdonBehaviour, and give it a `[TriggerSeekerEvent]` attribute:

```cs
[TriggerSeekerEvent("MyEvent")]
public UdonBehaviour onMyEvent;
```

:::note
The "MyEvent" string in the attribute sets what name to use to create program assets when events are enabled, but outside of that has no effect. The variable name `onMyEvent` is what's used in the UI.
:::


Then when you want to call the event use `EventUtil.Invoke(event)` like this:

```cs
public void TriggerEvent() {
    EventUtil.Invoke(onMyEvent);
}
```

And that's it! Any functions you add in the inspector will now be called by the `TriggerEvent()` function you just defined. You can put `EventUtil.Invoke(event)` anywhere you want in your USharp script and it'll work fine.

### Events with arguments

Now what if you want to pass a value with your event? It's as easy as adding a list of types to the TriggerSeekerEvent attribute.

Lets say we were creating a chat app, and we wanted an event for every time we recieve a message. We could define an event for that like this:

```cs
// Arg 1: message sender 
// Arg 2: message text
[TriggerSeekerEvent("MyEvent", typeof(string), typeof(string))]
public UdonBehaviour onMessage;
```

Then to send a message to the event we'd still call invoke, but this time we give it the data to send as well:

```cs
public void SendMessage(string sender, string message)
{
    EventUtil.Invoke(onMessage, sender, message);
}
```

So a complete example might look like this:

```cs
using TriggerSeeker;

public class MessageRoomClient : UdonSharpBehaviour {
    // Arg 1: message sender 
    // Arg 2: message text
    [TriggerSeekerEvent("MyEvent", typeof(string), typeof(string))]
    public UdonBehaviour onMessage;
    
    private void Start() {
        SendMessage("admin", "Welcome to the chatroom!");
    }

    public void SendMessage(string sender, string message)
    {
        EventUtil.Invoke(onMessage, sender, message);
    }
}
```


:::note
Events support any number of arguments, though performance will drop after 5 or so.

Refer to the list of types compatible with UI above for what types can be passed to events.
:::

:::warning
Event arguments are NOT checked by `EventUtil.Invoke()`, it's up to you to make sure you match both the 
number and types of parameters to what you defined in the [TriggerSeekerEvent] attribute, or you're gonna get some wonky bugs.
:::
