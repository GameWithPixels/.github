# Welcome!

This is the right place to start looking for information on how to integrate
Pixels dice with an app or a website.

![Two Pixels dice lighting up with multiple colors.](images/pixels_header.jpg)

*Two Pixels dice lighting up with multiple colors.*

# What Are Pixels Dice?

Pixels are full of LEDs, smarts and no larger than regular dice, they can be
customized to light up when and how you desire.
Check our [website](https://gamewithpixels.com/) for more information.

All Pixels dice come with Bluetooth connectivity which enables a software developer
to write code to connect to them from a mobile device or a computer.
Once connected to a Pixel, an app or a website may read rolls and program colorful
animations to be played on the dice LEDs.

## An Open Source Ecosystem

The Pixels software ecosystem is open source.
We [publish](https://github.com/orgs/GameWithPixels/repositories)
the source code of our Pixels app, libraries and firmware.

Communications with a device such as a computer or a phone is made using
Bluetooth Low Energy which is an open standard for short-range radio frequency
communication.

This open knowledge and reliance on the Bluetooth standard enables developers
to write code that communicates with Pixels using any combination of language
and system they wish as long as the underlying platform supports Bluetooth
Low Energy 4.2 or greater.

# Community

If you have any question please join us on our
[Discord server](https://discord.com/invite/9ghxBYQFYA).

This is the best place to reach out to us and also get help from the community.

All our Pixels software is open source and published on GitHub.
We are open to contributions via pull requests.

If you find a bug with one of our package, please open an issue on the corresponding
GitHub repository.

# How To Start

We provide official support for a growing selection of platforms and will add more
based on community feedback.
Those libraries encapsulate the details of the Bluetooth protocol and Pixels message
serialization so the developer only need to write a few lines of code without prior
knowledge to start communicating with Pixels dice.

We currently provide packages for the following platforms/environments:
* Web - [link](https://github.com/GameWithPixels/pixels-js/tree/main/packages/pixels-web-connect)
* React Native (iOS, Android) - [link](https://github.com/GameWithPixels/pixels-js/tree/main/packages/react-native-pixels-connect)
* Unity (iOS, Android & Windows) - [link](https://github.com/GameWithPixels/PixelsUnityPlugin)
* Swift (iOS & MacOS) - [link](https://github.com/GameWithPixels/swift-pixels-library)

We are also currently working on supporting these ones:
* Kotlin/Java (Android)
* .NET (Windows)
* C++ (Windows)
* Python (Windows & Raspberry Pi)
* Unity (MacOS)

Let us know on Discord if your looking for support on other platforms!
We'll prioritize our development based on community feedback.

Before jumping into programming please make sure to read this developer's
[guide](doc/DevelopersGuide.md).
It aims at pointing developers in the right direction when starting to write code to
communicate with Pixels dice.

Although we recommend starting with one of our official package, it is entirely possible,
to write custom software that connects to Pixels dice on any platform.
See these Pixels communication protocol [specifications](doc/CommunicationsProtocol.md)
to learn more about how to communicate with Pixels without using one of our library.

# Going Further

The code architecture of the Pixels libraries is described in this [document](
    doc/InternalArchitecture.md
).

The Pixels dice run a bootloader and a custom firmware.
Their respective repositories may be found [here](
    https://github.com/GameWithPixels/DiceBootloader
) and [there](
    https://github.com/GameWithPixels/DiceFirmware/
).
