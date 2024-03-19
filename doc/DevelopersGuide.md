# Pixels Developer's Guide

This guide aims to provide a first introduction to software developers who
want to write code that connects to and communicates with Pixels dice.

Check our [website](https://gamewithpixels.com/) to learn more about Pixels dice.

Our dice may be integrated with different types of software:
websites, mobile apps, video games, desktop apps, etc.
For simplicity, in this document we just use the term "software" to refer to
any or all of those possibilities.

Our [libraries](https://github.com/orgs/GameWithPixels/repositories)
enable developers to communicate with Pixels dice without first needing to learn
about the Bluetooth protocol.
But we think it remains essential to understand the inherent challenges that
come with wireless communications in order to write reliable code and build
software with a great user experience.

With wireless communications, delays or failures may happen at any time. 
To keep the user engaged in such circumstances, the software should try to
present this information in a way that is integrated with the user experience.
We think that the user interface should be designed to gracefully handle
such problems as they will inevitably happen.

> *Note*:
>
> This document describes what we think are good practices when integrating Pixels dice
> in any given software.
> There are a few code snippets that illustrates some of most the basic features available
> in our libraries.
> Those example written in JavaScript using our
> [pixels-web-connect](
    https://github.com/GameWithPixels/pixels-js/tree/main/packages/pixels-web-connect
)
> package.
>
> Even though the syntax will vary from one language to another, our libraries
> have all a similar `Pixel` class and the general concepts of how to communicate
> with Pixels dice stay the same.

## The Challenge Of Wireless Communications

Using one of our Pixels library, connecting to a Pixel and retrieving its roll
state is quite straightforward:

```JavaScript
// Ask user to select a Pixel
const pixel = await requestPixel();
// Connect to die
await pixel.connect();
// Get last roll state
const rollState = pixel.rollState;
console.log(`Roll state: ${rollState.state}, face up: ${rollState.face}`);
```

However there are a number of challenges associated with those few lines of
code.

Like with any other wireless device, communicating with Pixels happens
asynchronously so this code might take longer than anticipated to execute.

In average, it takes up to a few seconds to connect to a Pixel and a couple
hundred milliseconds to get its roll state.
But there are a number of reasons such as interferences or the distance
between your device and the die that may slow down or break communications.

The above code might also throw an exception and never complete as wireless
communications are by nature unreliable.
The Bluetooth protocol is designed to help with that but there are a number
of things that may happen and for which failure is the only possible outcome.

Depending on the platform and the circumstances, it may take more than a few
seconds before the underlying Bluetooth stack gives up on a specific request
and throws an exception.

Here is a short list of the most common cause of communication failures:
- Device is taken out of range of by the user.
- Device is turned off or restarted by the user.
- Device ran out of battery.
- Too much wireless interferences.

Any of these issues might happen while the above code sample is being executed
and which will eventually result in an exception being thrown by one of the
asynchronous call made to the Pixels package.

> *Note:*
> 
> There is no need to pair Pixels dice with the operating system before
> attempting to connect to them.
> In fact pairing will fail on most OSes as this is not a supported feature.

## User Experience

In the above section we've seen that communicating with Pixels may sometimes
result in a long wait time (more than a few hundred milliseconds) or with an
outright failure to complete (in which case the code throws an exception).

Those outcomes should be infrequent but they will occur nonetheless.
In order to keep the user engaged, the user interface should be designed to
gracefully respond to such events.

We recommend that the user interface should some indication to the user
(like a spinner) when waiting for an asynchronous call to complete as well as
offering the user the option to either retry the operation or fallback to
another option (like a virtual roll).

Other unexpected events will inevitably happen while using Pixels dice.
For example the user might re-roll their die or roll the wrong one (in case
of a multi dice setup).

The overall quality of the user experience will depend as much on the quality
of the Pixels dice as on how the user interface responds to physical events.
It is one of the main challenges of integrating a physical device with a piece
of software.
For this link to feel true to the user, the software needs to reflect as much
as possible what is happening in the physical world, meaning that unexpected
rolls shouldn't be simply ignored by the software.

In the case of Pixels dice, the key is in handling and responding to connection
and roll events.

For example, displaying a dialog box stating that the die was disconnected and
requiring to choose between "reconnect" or "cancel" is not engaging for the user.
We suggest that this information should be presented in a more integrated way
as part of the user interface (more on this later).

It's also usually good practice to leave the possibility for the user to a do
a virtual roll at any moment.

## Reading Rolls

Pixels dice automatically send their roll state when connected.
The `Pixel` class offer two ways to access that value:
- Retrieve the last known roll state value: this is an immediate operation
that doesn't involve communicating with the die as the Pixel instance
keeps that data locally.
- Subscribe to roll events: it will notify your a subscriber of any new roll.

When waiting for a roll to happen, the preferred way is to subscribe
to the "roll" event and wait for it.

```JavaScript
const onRolled = (face) => {
    // Unsubscribe so further roll events are ignored
    pixel.removeEventListener("roll", onRolled);
    // Use the roll event
    console.log(`Rolled face: ${face}`);
};
// Add listener to get notified on rolls
pixel.addEventListener("roll", onRolled);
```

Depending on the use case, you may not want to wait indefinitely for a roll
and notify the user if they haven't rolled their die after a certain delay.

It also possible to get notified of roll state changes.
Possible states are "onFace", "handling", "rolling" and "crooked".

> *Note:*
> 
> When not connected to, Pixels dice advertise some information such as roll state,
> current face up and more.
>
> This makes it possible to read rolls without even connecting to a die.
> But there is an important caveat: as intended by Bluetooth protocol, this
> advertising data is not re-emitted when not received by the listening device(s).
>
> So this makes it possible for the listening device to miss some rolls.
> In contrast, when connected to a die, the Pixels software ensures that all roll
> events are always received.

## Connection Status

The `Pixel` class lets the developer check for the die connection status.

Or more precisely this is the *last know* connection status.
The die might already be disconnected from the phone or computer, but depending
on the circumstances there might be a delay before the Bluetooth stack
acknowledges that fact (such as in the case of an out of range die or a power loss).

It is also worth noting that even if the connection status was to be accurate
at the time of retrieval, the die might still disconnect the instant after the status
was checked.

As a consequence the following code may still result in the call to
`queryRssi()` throwing an exception.

```JavaScript
if (pixel.status === "ready") {
    // Querying RSSI may still fail (and throw an exception)
    const rssi = await pixel.queryRssi();
    console.log(`RSSI: ${rssi}`);
} else {
    // Die not connected
}
```

Therefore checking for the connection status is usually not a good practice.
It is best to always catch exceptions when invoking asynchronous methods of a
`Pixel` instance.

```JavaScript
try {
    // Querying RSSI
    const rssi = await pixel.queryRssi();
    console.log(`RSSI: ${rssi}`);
} catch {
    // Die not connected
}
```

## Connected Pixels

A Pixels die may be connected to only one device at a time.
When connected it won't be seen by a Bluetooth scan so in practice this is just as
if the die was turned off from the perspective of other devices
(or even other software running on the same device).

There is not automatic disconnection timeout:  once the software is connected to a Pixels
die, it will stay as such until a disconnection occurs (whether intended or not).
On modern phones, the app will continue running even when put into the background
and will stay connected to the dice.

A user might not realize that and try to connect to their dice using another software
but they won't be able to do so until the dice are disconnected from the first software.

To offer the best user experience we recommend that:
- The software clearly presents the information about the Pixels connection status.
- The user should always have the option to manually disconnect from a die.

Ideally the software would disconnect from the dice when put in the background so they
become available for other software to use.

## Upon Disconnection

As discussed, the connection with a die may fail at any moment for various reasons.
The software should be designed to handle such events gracefully so the user experience
stays optimal.

Showing an error message in a popup window is strongly discouraged.
Instead the software may simply update the information presented to the user to indicate
that the connection was lost.
It is important that this information is clearly presented so the user understands why
their rolls are no longer picked up by the software.

When the disconnection is not intended (not initiated by the software either automatically
or on the user's request) the software can attempt to automatically reconnect to the die.
It should also offer the possibility to do a virtual roll so the user has always the option
to continue without a physical die.

From a code perspective, an exception is thrown when trying to actively communicate
to a disconnected die.
But when listening for events such as the roll event, nothing special happens upon
disconnection.
We recommend that the software always listen to the die connection status event and
reacts upon receiving it.

```JavaScript
const onStatusChanged = (status) => {
    if (status === "disconnecting" || status === "disconnected") {
        // We are being disconnected
        console.log(`${pixel.name} is ${status}`);
    }
};
// Add listener to get notified on rolls
pixel.addEventListener("status", onStatusChanged);
```

## One Die, Many Dice

The decision to connect to one die or to several dice at a time has a major
impact on the UX design of a project integrating with Pixels.

Technically speaking connecting to multiple dice at a time is just a matter of
keeping track of several `Pixel` instances, one for each physical die that is
being connected to.

However from a UX perspective, dealing with multiple dice creates new
challenges. In particular the user interface should allow to mix Pixels dice
with virtual ones, as not every user might have all the required Pixels dice.

When waiting on the user to roll their dice:
- The user might roll their die one by one or all at a time. It is usually
  best to display the roll results as they come in rather than waiting for all
  them before displaying anything.
- Some die might disconnect while others have already been rolled, so it
  should be possible for the user to reconnect to any die or to roll a virtual
  die without loosing the rolls already made.
- The user might re-roll a die, accidentally or not, before rolling all the
  requested dice.

Regarding the last item (some dice being re-rolled), different situations may
call for different UX decisions.
It is preferable to ask the user if the re-roll was intended rather then force
the decision on them (which can be frustrating).

Regardless of the decision, we strongly recommend to inform the user which
dice have been re-rolled and whether or not the new roll was taken into account.

For example, one user may accidentally bump an already rolled die and not
realize it.
If no information is displayed, the user might think the system is not working
properly has the roll result displayed on screen will not match the physical die.

When multiple users are involved, we recommend that they are shown the same
information regarding roll results and re-rolls as the person actually rolling dice.
