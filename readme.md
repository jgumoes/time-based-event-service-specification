## Preamble
 This document is still being created, and is not yet self-consistent. I haven't yet decided on a lot of details, or even if this is worth seeing through to completion. Maybe there's a reason this service doesn't exist? I need to go through all the examples, and outline what each use-case requires from this service. I also need to work out how to handle time changes, i.e. daylight savings, crossing timezones, the device having the wrong time, etc. The time-changing requirements mean that the peripheral implementation of this service will need to couple with an implementation of the [Device Time Service](https://www.bluetooth.com/specifications/specs/device-time-service-1-0/).

 # Abstract
 This pseudo-whitepaper describes a generic approach to setting time-based events such as alarms or time windows, that should allow for platform-agnostic libraries and copy-pasting into projects as required. I have included a few examples to demonstrate its usage and flexibility.

# Motivation
 I have several projects that involve changing device parameters with time of day (i.e. a lamp that always turns on at min brightness after midnight), as well as alarms with different functionalities. I also don't know when these events occur or how many events each device will handle, so I'm not keen on hard-coding the event times and values. The projects are all microcontrollers that will be interfaced with a mobile app, and will be able to connect to the internet but will use BLE as much as possible to avoid over-loading my router.
 
 It would be useful to have a C++ library that works on various microcontrollers but works with various file systems and communication protocols. It would also be useful to have a React Native library that can be imported to any project as required. I think having a clear technical specification will be useful for myself, and hopefully for others too.

 Another benefit of this specification is that, when combined with a publisher/subscriber protocol, it could allow for simplified synchronised alarms across a number of devices that each have their own widely-different time-based requirements.

 I should point out that there is a good reason why I'm targeting BLE instead of WiFi: my router does not play nicely with smart devices. The failure pattern is as follows:
  1. The router gets overwhelmed by the number of connected devices and temporarily reverts its credentials back to the factory defaults
  2.  the smart device only holds one set of credentials so it can no longer connect to the router
  3.  the smart device gives up trying to connect
  4.  the router is now connected to fewer devices, and, feeling less overwhelmed, it changes its credentials back to the user-defined ones
  5.  the smart device doesn't bother trying to connect after long periods of time, it just stays disconnected

Obviously you can program a device to hold multiple WiFi credentials, or when the re-connect threshold is reached increase to re-attempt interval to an hour instead of stopping any further attempts. However, this will result in more WiFi connections for the router, it will be constantly overwhelmed and the connections will be unstable. It would be far better to have devices over a BLE mesh network, with the smartphone acting as an entry point. It's worth point out, though, that the implementation can be communication-agnostic. The same source code could be used for WiFi or Thread, or ZigBee, etc.

# Overview / Theory of Operation

It's probably best to think of this service as relational key-value stores: 
 - eventIDs are keys to the events;
 - events contain a 1-to-1 relationship to actions;
 - actionIDs are keys to the actions;
 - many events can refer to single actions;
 - cascading events hold relations to other events;
 - many cascading events can relate to a single event of any type, but not vice-versa

## actionIDs and listeners
This specification aims to abstract alarm objects (i.e. time-based events) to allow for flexible runtime alterations. This is achieved by using actionIDs that point to unique listeners, and payloads which carry data through those listeners. A sensible implementation could use the actionIDs to reference a map of callback functions, that take the payload as an argument.

(note: the event logic can't be abstracted without a form of virtual machine, and I'm not putting a VM into a resource-constrained microcomputer, so the event logic will have to be hardcoded)

## Time-Based Events

These are essentially alarm objects that will trigger specified actions at specified times. They must contain a compatible actionID and payload, a self-identifying eventID, and, most importantly, a timestamp. What this timestamp looks like, will depend on the type of event the timestamp describes. Day of Week and Date of Month events contain an absolute timestamp, and Cascading Events contain a timestamp relative to the event that triggers the cascade.

### Event Types
All time-based functionality can be encapsulated in three types of events:

**Day Of Week Events**

These are events that trigger at a set time of day on set days of the week. i.e. an alarm clock might trigger at 8:00 am Monday to Friday.

**Date of Month Events**

These are events that trigger at a set time of day on a set date. They can trigger on a specific month (i.e. for birthdays, or auto-adjusting for daylight savings), or they can trigger on the same date every month.

**Cascading Events**

These are events that are set when another event gets triggered. They can be used to create time windows, add snooze functionality to headless alarm clocks, timeout the snooze functionality of headless alarm clocks, turn every lamp in a room into a synchronized Pomodoro timer, change the default brightness of a lamp depending on time of day, etc.

Cascading events are designed to be triggered when an absolute event is triggered, but a sensible implementation could allow for a user to trigger the cascade. They can be self-triggering for strict time intervals, they can set a mode to allow a user to reset the timer, and they can be non-repeating one-off timers.

An implementation must include the ability to cancel an upcoming event.

## Missed Events

The device may miss the exact event time for a number of reasons, such as a power outage or the OS rebooting. Some events should still trigger a few minutes late (i.e. alarm clocks), and some events should trigger regardless of how late they are (i.e. pet feeders).

The device should sequentially run through all missed events and trigger (or skip) them chronologically. Therefore, if an event changes a parameter, i.e. a light that changes its default brightness throughout the night, the event should always be triggered regardless of how late. The device will run through all the parameter-changing events until it lands at the appropriate one for the specific time (note: this could result in a noticeable wobbling. The developer could implement a short non-blocking delay before the parameter change is actioned, that resets every time the parameter is changed).

Each event can be given a timeout, or it can revert to the device default. The device default can also be set.

## Daylight Savings and Timezones

Daylight Savings and timezone changes need to be handled differently depending on the time event type and time. Daylight savings can change by 1 hour, and crossing the International Date Line will change the time by 24 hours, meaning that the device time can change by up to 25 hours within a few hours. Handling changes can get pretty complex, especially for 

Day of Week and Date of Month events shouldn't be affected in spring, as an event triggered immediately before the DST change will be reinitiated for a different day and therefore won't repeat. In autumn however

