# reMarkable 2 pogo pins reverse engineering

## Hardware
The reMarkable 2 tablet has two USB interfaces, one of which is in the form of USB-C at the bottom, while the other is in the form of 5 pins on the bottom left side. 
The USB interfaces are driven by a [MAX77818](https://www.analog.com/media/en/technical-documentation/data-sheets/MAX77818.pdf) capable of USB back power (OTG) for which the back power capable bus is wired to the pogo pins. 
![pins](https://github.com/pabloaul/rm2-pogo-com/assets/35423980/1daf98c5-a366-467d-9aa9-9fcda3983a65)


## The ID Pin
This pin is capable of talking single wire serial. 
The keyboard connected to it talks at 115200 baud. 
Pretty much default everything.

## Structure of a serial frame sent via ID pin
![Packet Structure](https://github.com/pabloaul/rm2-pogo-com/assets/35423980/e2cf386e-0075-4bc8-94dc-5746810920ad)

- Direction:
  - 2E (keyboard to device)
  - 3A (device to keyboard)
- Length:   amount of bytes in the data part + 1
- Data:     refer to data structure
- Checksum: inversion of sum of all data bytes + 1

## Data Structure
Data content usually consists of a command and arguments given to it.
Possible commands are:
- 04 : enter "app mode"
- 09 : request auth key from peripheral (sent from device to keyboard, no arguments)
- 20 : read register (batching possible, arguments are which registers to read)
- 40 : keep-alive (sent from keyboard to device, every 0.5s on no activity, no arguments)
- 51 : keystroke (sent from keyboard to device, contains key pressed and key counter)

## Known registers
- 02 : firmware version (2x uint8_t) 
- 04 : device_class (uint32_t)
- 05 : serial number (char[15])
- 06 : firmware_start_address (uint32_t)
- 07 : device_name (12 bytes)
- 10 : keyboard_layout (uint8_t)
- 11 : language (1 byte)
- 12 : rm_serial_number (33 bytes)

## Keyboard connection flow
- Tablet notices (pull-down/pull-up?) on pogo pins
- Tablet requests register read (cmd 20) -> keyboard responds success, reading "rMkeyboard01"
- Tablet requests register read (cmd 20, batched) -> keyboard responds success with data requested.
- Tablet requests an auth code (cmd 09) -> keyboard responds with the key "@O8eO77%o^4*1GE@oeodd#WMa%8Kr6v@" and is zero-terminated.
- Tablet goes into "app mode" (cmd 04) -> keyboard acknowledges.
![init](https://github.com/pabloaul/rm2-pogo-com/assets/35423980/907e60ae-b951-4206-802d-f7f9d35d2549)
This entire negotiation happens within 30ms.

## Related projects
![Jonas - USB Keyboard to pogo adapter using a Pi Pico](https://github.com/jonasoberschweiber/remarkable-keyboard-adapter/)
![Dudlushka - Resin printed pogo adapter and circutry using Arduino](https://github.com/Dudlushka/Remarkable_TypeFolio_Pretender)
