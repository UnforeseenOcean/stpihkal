# Lovense

## Introduction

Lovense has been manufacturing toys since 2011.

## Bluetooth Details

While all lovense toys use the same protocol, they can communicate
over bluetooth differently, depending on when they were released.

### Bluetooth 2.0 Toys

The first toys released by Lovense used both Bluetooth 2.0 SPP
(emulating a serial port) and Bluetooth LE. This was most likely due
to the sparse mobile support of BTLE when they were released.

These toys include:

- Max
- Nora

When paired with a system via Bluetooth 2.0, these toys identify as a
serial port. These toys are also capable of using Bluetooth 4.0, as
outlined in the next section.

### Bluetooth LE Toys

Starting with the Lush, all toys released by Lovense use only
Bluetooth LE.

These toys have GATT characteristics to mimic the RX/TX setup of the
serial port style control of the old toys. The GATT service and
characteristic IDs differ between different toy firmware versions.

It's difficult to keep a current list of exact Lovense device names
and service/characteristic UUIDs, as they tend to change rapidly on
firmware updates. The following rules can be used for finding and
connecting to Lovense toys.

Lovense toy names always start with "LVS-". What comes after that
varies depending on when the toy was released. Early toys used names
involving the single character identifier, like "LVS-A011", while
newer toys use the full product name, like "LVS-Edge36". The last 2
numbers denote the firmware version the toy is running.

Lovense toys usually have one of 3 service ID formats:

```
0000fff0-0000-1000-8000-00805f9b34fb
6e400001-b5a3-f393-e0a9-e50e24dcca9e
5X300001-002Y-4bd4-bbd5-a6920e4c5653
```

The first two service IDs are static, and represent the service IDs
used by first and second generation Lovense toys. The 3rd service ID
can vary, with

- X being any number 0x0-0xf
- Y usually being 0x3 or 0x4

While some bluetooth APIs can wildcard services, others like
WebBluetooth require an exact service UUID to connect. For these
instances, it's recommended to just generate out all 32 variations of
the last service, for a total of 34 services, to use with the
optionalServices portion of a WebBluetooth connection filter.

To identify the type of toy after connecting, it is recommended to use
the [DeviceType;](lovense.md#get-device-information) message, outlined
below. This will return a device model identifier.

## Protocol

Commands for Lovense toys follows these rules:

- Commands and replies are strings, using semicolons to mark their end.
- All commands start with a command identifier word, then possibly
  either specifiers or levels, delimited by colons. e.g. "Vibrate:5;"
  would set vibration to 5.
- Replies are in the context of the command (i.e. sending "Battery;"
  will just return a number, like "85;"), but can still be colon
  delimited lists.
- Commands that do not return a context specific value will return
  "OK;" on success, "ERR;" on error.

### Command List

The following is the known command table for all toys. Anything send or
received over the serial port is in quotes to denote communication, but
should not be sent using quotes if you are implementing your own version
of this protocol. Commands with ":x" mean that the x should be replaced
with a number, the range of which is mentioned in the description.

#### Get Device Information

Returns toy model type, firmware version, and bluetooth MAC address,
as a colon delimited list

_Availability:_ All toys

_Command Format_
```
DeviceType;
```

_Return Example_
```
C:11:0082059AD3BD;
```

Denotes Nora toy, running v1.1 firmware, BT Addr of 00:82:05:9A:D3:BD

**Model Types:**

| Model | Type Letter |
| ----- | ----------- |
| Nora | A or C |
| Max | B |
| Ambi | L |
| Lush | S |
| Hush | Z |
| Domi | W |
| Edge | P |
| Osci | O |


#### Get Battery Level

Returns the battery level of the toy as an integer percentage from 0-100.

_Availability:_ All toys

_Command Format_
```
Battery;
```

_Return Example_
```
85;
```

Denotes 85% battery remaining.

#### Turn Off Power

Turns off power to the toy.

_Availability:_ All toys

_Command Format_
```
PowerOff;
```

_Return Example_
```
OK;
```

#### Device Status

Retreive the status of the toy. 

_Availability:_ All toys

_Command Format_
```
Status:1;
```

_Return Example_
```
2;
```

_Status Codes:_

- 2: Normal

#### Set Vibration Speed

Changes the vibration speed for the toy. Takes integer values from 0-20.

_Availability:_ All toys

_Command Format_
```
Vibrate:10;
```

Sets vibration speed to 10 (50%).

_Return Example_
```
OK;
```

#### Start Accelerometer Data Stream

Starts a stream of accelerometer data. Will send constantly until stop
command is sent. Incoming accelerometer data starts with the letter G,
followed by 3 16-bit little-endian numbers.

_Availability:_ Max, Nora

_Command Format_
```
StartMove:1;
```

_Return Example_
```
GEF008312ED00;
```

Denotes [0x00EF, 0x1283, 0x00ED] accelerometer readings.

#### Stop Accelerometer Data Stream

Stops stream of accelerometer data.

_Availability:_ Max, Nora

_Command Format_
```
StopMove:1;
```

_Return Example_
```
OK;
```

#### Change Rotation Direction

Changes the direction of rotation for the toy.

_Availability:_ Nora

_Command Format_
```
RotateChange;
```

_Return Example_
```
OK;
```

#### Set rotation speed

Changes the rotation speed of the Nora toy. Takes integer values from 0-20.

_Availability_: Nora

_Command Format_
```
Rotate:10;
```

Sets rotation speed to 10 (50%).

_Return Example_
```
OK;
```

#### Set Absolute Air Level

Changes the inflation level of the Max toy. Takes integer values from 0-5.

_Availability:_ Max

_Command Format_
```
Air:Level:3;
```

Sets air level to 3 (60%).

_Return Example_
```
OK;
```

#### Set Relative Inflation Level

Inflates relative to current level, i.e. if currently inflation level
is 3, and "Air:In:1;" is sent, will inflate to 4.

_Availability:_ Max

_Command Format_
```
Air:In:1;
```

Sets air level to 1 level more inflated than it was.

_Return Example_
```
OK;
```

#### Set Relative Deflation Level

Deflates relative to current level, i.e. if currently inflation level
is 3, and "Air:Out:1;" is sent, will deflate to 2.

_Availability:_ Max

_Command Format_
```
Air:Out:1;
```

Sets air level to 1 level deflated than it was.

_Return Example_
```
OK;
```

## Related Projects and Links

The applications and repositories below contain implementations of the
Lovense communications protocol, or have relevant information about
the hardware/firmware.

* Buttplug C# (All toys): [https://github.com/metafetish/buttplug-csharp](https://github.com/metafetish/buttplug-csharp)
* Buttplug JS (All toys): [https://github.com/metafetish/buttplug-js](https://github.com/metafetish/buttplug-js)
* lovesense-rs (Rust, Max/Nora only): [https://github.com/metafetish/lovesense-rs](https://github.com/metafetish/lovesense-rs)
* lovesense-py (Python, Max/Nora only): [https://github.com/metafetish/lovesense-py](https://github.com/metafetish/lovesense-py)
* Max/MSP patch (Max/MSP, Max/Nora only): [https://github.com/metafetish/lovesense-max](https://github.com/metafetish/lovesense-max)
* Node.js Library (Max/Nora only): [https://github.com/metafetish/lovesense-js](https://github.com/metafetish/lovesense-js)
* WebBluetooth JS Library/Demo (Hush only): [https://github.com/metafetish/lovesense-hush-js-demo](https://github.com/metafetish/lovesense-hush-js-demo)
