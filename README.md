# reMarkable 2 pogo pins reverse engineering

## Hardware
The reMarkable 2 tablet has two USB interfaces, one of which is in the form of USB-C at the bottom, while the other is in the form of 5 pins on the bottom left side. 
The USB interfaces are driven by a [MAX77818](https://www.analog.com/media/en/technical-documentation/data-sheets/MAX77818.pdf) capable of USB back power (OTG) which is however only wired to the pogo pins. 

From the bottom-up the pins of the pogo correspond to the following: GND | D- | D+ | ID | VCC

## The ID Pin
This pin is capable of talking single wire serial. 
The keyboard connected to it talks at 115200 baud. 
Pretty much default everything.

## Structure of a serial frame sent via ID pin
Byte[1]: Direction
- 2E (keyboard to device)
- 3A (device to keyboard)

Byte[2]: Data length (n)

Byte[n]: Data

Byte[1]: CRC Checksum

## Keyboard connected flow
- Tablet notices (pull-down/pull-up?) on pogo pins
- Tablet requests register read (cmd 20) -> keyboard responds success, reading "B<0x0c>rMkeyboard01"
- Tablet requests register read (batched) (cmd 20) -> keyboard responds success with data requested (content explained in later commit).
- Tablet requests an auth code (cmd 09) -> keyboard responds with the key "@O8eO77%o^4*1GE@oeodd#WMa%8Kr6v@" and is zero-terminated.
- Tablet goes into "app mode" (cmd 04) -> keyboard acknowledges.

This entire negotiation happens within 30ms.
