# Examples

Below are some real and hypothetical use-cases for this service. Some are projects I am working on, some are purely hypothetical to ensure the service will work for a wide range of use-cases, and some are pure evil and should never be made in the real world.

Note that all of the functionalities described are inter-compatible. You could create a single device that implements every example below, simultaneously.

None of the examples are explicitly synchronous, despite my bragging that synchronised events are supported. The reason is simply that there is nothing to explain. Provided that a set of devices have the same time, and are programmed the same event, the event will trigger at the same time on all devices regardless of connection status at the time of the event.

## headless alarm lamp
*Description*: A smart lamp may be used as an alarm clock. It could have an alarm set for 8 am every weekday day and 10am on the weekends, that slowly increases the brightness over 30 minutes. This device could then be snoozed for 5 minutes, after which it will increase it's brightness over 5 minutes. Note that the end user will likely have set the alarm for 8:30/10:30, but the alarm-setting app will have reinterpreted it as full brightness by 8:30/10:30 over a 30 minute period.

```
{
  0x01, // set eventID to one cus why not
  0b00000110,  // no payload | user interaction required | event will repeat;
  0b01000001111100100000000000000000, // set an alarm for 8 am Monday to Friday
}

```

## Automatic Pet Feeder
*Description*: As I am writing this, I have just read an article complaining that the author's automated pet feeder is unreliable, as it frequently loses its WiFi signal and subsequently misses the communication instructing it to feed the dog. The article uses the pet feeder as an example of the increasingly unreliability of smart devices device operating over 2.4gHz WiFi. My big question, though, is if the user knows what time the pet feeder needs to open, why can't the pet feeder know what time it needs to open? Why  People use these devices to feed their animals when they're not around, they need to work reliably, even when there's no internet!

The pet feeder is a great example of why projects like this one are necessary: large companies tend to design products that work in laboratory environments with little thought to real-world limitations. If an end-user wants a product like a pet feeder, that will work as expected, but cannot afford an expensive high-end model, it is up to the end user to modify a cheap pet feeder to make it suitable for the real world.

## a smart-locking biscuit barrel
*Description*: <sup>(this is the bad one)</sup> A biscuit barrel could be equipped with a locking device, such that it can only be opened a set number of times a day. It could be given a time window between 8 am and 8 pm, with a limit of 2 hours between each opening. Between 8 pm and 9 pm it may be opened once more as desert, then it may not be opened again until 8 am the following day.

## bedtime indicator
*Description*: A decorative light source (i.e. LED lights covering a wall) could 

## smart-home timer
*Description*: A light connected to a smart-home system (i.e. google home) that can be set with the command "set smart light timer for 30 minutes"

## Strict Pomodoro timer
*Operation*: Uses two cascading events that trigger each other to switch between work and break modes. The payload dictates the time until the next event. The initial event and the cancellation are both user triggered.

## Flexible Pomodoro timer
*Description*: With the strict Pomodoro, it's easy to over-run each mode and fall behind in the next. It's preferable to manually start the next mode when ready.

*Operation*: The same as with the Strict Pomodoro timer, except the events don't trigger each other. Instead, they enable the user to trigger the next mode/event when they are ready. You could collect the timestamps when each mode is started, and use them to self-reflect on productivity.

## a faux-neon sign declaring "Pinch Punch First Of The Month"
*Operation*: Uses a repeating Date Of Month event to trigger an action that slowly flashes a light. Uses a second repeating Date Of Month event to terminate the action at noon. Note that it's the same actionID for both events. The payload dictates the speed of the flashing, with a value of 0 defaulting to off.

## automatic DST changeover

## Randomized non-consensual All Star
*Description*: You could have a device that plays random clips of Smash Mouth's All Star at random intervals during a day. If the device is well hidden and the clips short enough, this device could be incredibly difficult to locate.

*Operation*: Uses a self-destructing cascading event that triggers a short audio clip, and creates and sets a new self-destructing cascading event with a randomized time length. Uses a Day Of Week event to initiate the cascade, and another Day Of Week event to cancel the cascade.