# Pixels Communications Protocol

To communicate with a Pixels die, one has to connect to it using the Bluetooth
Low Energy protocol, and then retrieve the two Bluetooth characteristics that
the die is using to communicate with other Bluetooth devices (note: a Bluetooth
characteristic is like a wireless communication channel).

One of the two characteristics is used to send messages to the die and the other
one to receive messages from it.

See the open source Pixels [firmware](
    https://github.com/GameWithPixels/DiceFirmware/
) to learn more about the details of Bluetooth support with Pixels.

## Advertisement

The Pixels firmware is using the advertisement packets to send a bit of
information about its state.
Those packets include the following information:

- Die name (as set by the user, up to 13 characters)
- Pixel Id: a 32 bits number unique to each Pixel
- The total LED count
- The die visual design
- The roll state and face up
- Battery level: in percentage
- Whether the battery is charging
- The firmware build timestamp
- RSSI

Name and RSSI are part of Bluetooth standard for advertisement packets.
The rest of the data is stored in manufacturer data and the Pixels service data.
Since the space for advertising data is quite limited, we've packed the data as
much as we could.

### Manufacturer data

| Name                    | Size    | Description                       |
| ----------------------- | ------- | --------------------------------- |
| LED count               | 1 byte  | Number of LEDs                    |
| Design & Color          | 1 byte  | Physical look of the die          |
| Roll State              | 1 byte  | Current rolling state             |
| Current Face            | 1 byte  | Current face up (face index)      |
| Battery Information (*) | 1 byte  | Battery level and charging status |

(*) Battery Level:
* The most significant bit (MSB) is set to 1 when charging.
* The other 7 bits are the battery level in percentage.

### Service data

| Name            | Size    | Description                     |
| --------------- | ------- | ------------------------------- |
| Pixel Id        | 4 bytes | Unique identifier               |
| Build Timestamp | 4 bytes | Firmware build timestamp (UNIX) |

## Services & Characteristics

Pixels support three different services:

| Name         | UUID                                   |
| ------------ | -------------------------------------- |
| Information  | `180a`                                 |
| Pixels       | `6e400001-b5a3-f393-e0a9-e50e24dcca9e` |
| Nordic's DFU | `fe59`                                 |

Only the Information and Pixels services are advertised.

The Pixels service has two characteristics:

| Name   | UUID                                   |
| ------ | -------------------------------------- |
| Notify | `6e400001-b5a3-f393-e0a9-e50e24dcca9e` |
| Write  | `6e400002-b5a3-f393-e0a9-e50e24dcca9e` |

Those two characteristics allow for exchanging data messages between the die
and a connected device.

The "write" characteristic is used to send messages to the Pixel die whereas
the "notify" characteristics is used to receive messages from the die.

## Messages

Pixels messages are encoded in binary format (little endian).
As a general rule we try to keep the messages as compact as possible in order to
minimize the amount of data that is send over Bluetooth.
This is partly to save battery life on mobile devices as the Bluetooth radio
consumes a lot of power, and partly to keep communications latency as low as possible.

This document describes only the most useful messages.
For the complete list please refer to [bluetooth_messages.h](
    https://github.com/GameWithPixels/DiceFirmware/blob/main/src/bluetooth/bluetooth_messages.h
) in the firmware code.

The messages are made of two parts:
- The first byte contains the message id, it identifies the message type
- The rest contains the data of the message, it is empty for messages that have
no associated data

> *Note*:
>
> The tables below present the messages data fields, in the order their value appear
> in the message data.
>
> Multi-bytes values are serialized using little endianness.

### Messages Received By Pixels

These are messages send by the host device to the Pixels die.

#### WhoAreYou

After being connected to a die, the host usually sends a "WhoAreYou" message which requires the die to identify itself.
Sending this message is not mandatory.

| Field     | Size    | Description |
| --------- | ------- | ----------- |
| Id        | 1 byte  | Value: 1    |

This message has no data, so it's effectively just one byte with the value 0x01.

#### Blink

This message makes the die light up its LEDs according to the values of the fields.

| Field     | Size    | Description                                           |
| --------- | ------- | ----------------------------------------------------- |
| Id        | 1 byte  | Value: 29                                             |
| Count     | 1 byte  | Number of blinks                                      |
| Duration  | 2 bytes | Animation duration in milliseconds                    |
| Color     | 4 bytes | Color in 32 bits ARGB format (alpha value is ignored) |
| Face Mask | 4 bytes | Select which faces to light up                        |
| Fade      | 1 byte  | Amount of in and out fading (*)                       |
| Loop      | 1 byte  | Whether to indefinitely loop the animation            |

(*) Sharpness value:
- 0: sharp transition
- 255: maximum fading.

### Messages Send By Pixels

These are messages send by by the Pixels die to the host device, whether spontaneously or in response to another message.

#### IAmADie

Upon receiving a "WhoAreYou" message, the die will send back an "IAmADie" message which contains useful information about the die.

| Field           | Size    | Description                      |
| --------------- | ------- | -------------------------------- |
| Id              | 1 byte  | Value: 2                         |
| Led Count       | 1 byte  | Number of LEDs                   |
| Design & Color  | 1 byte  | Physical look of the die         |
| N/A             | 1 byte  |                                  |
| Data Set Hash   | 4 bytes | Internal                         |
| Pixel Id        | 4 bytes | Unique identifier                |
| Available Flash | 2 bytes | Unique identifier                |
| Build Timestamp | 4 bytes | Firmware build timestamp (UNIX)  |
| Roll State      | 1 byte  | Current rolling state            |
| Current Face    | 1 byte  | Current face up (face index)     |
| Battery Level   | 1 byte  | Battery level in percentage      |
| Battery State   | 1 byte  | Battery state (charging or else) |

#### RollState

The "RollState" message is send by the die every time it's roll state or current face up changes.

| Field           | Size    | Description                      |
| --------------- | ------- | -------------------------------- |
| Id              | 1 byte  | Value: 3                         |
| Roll State      | 1 byte  | Current rolling state            |
| Current Face    | 1 byte  | Current face up (face index)     |

#### BatteryLevel

The "BatteryLevel" message is send by the die every time it detects a battery level or a battery state change.

| Field           | Size    | Description                      |
| --------------- | ------- | -------------------------------- |
| Id              | 1 byte  | Value: 34                        |
| Battery Level   | 1 byte  | Battery level in percentage      |
| Battery State   | 1 byte  | Battery state (charging or else) |
