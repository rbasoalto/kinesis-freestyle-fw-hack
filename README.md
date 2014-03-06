# Kinesis Freestyle Firmware Hacking

## Hardware

On the underside of the right half of the keyboard (the side where the USB cable comes out), remove the black label covering the FW Update connector. From the side we're looking, the pinout is:

    ---------------
    \  5   4   3  /
     \   2   1   /
      -----------

    1: VCC (3v3)
    2: EEPROM Write Protect (high = protected)
    3: SCL
    4: SDA
    5: GND

Connect the Bus Pirate accordingly, in I2C 100kHz mode.

## Read FW

Read the whole EEPROM with `[0xa0 0x00][0xa1 r:1025]` (disregard last byte).

## Write FW

To write the whole EEPROM, put pin 2 low (use BP aux pin), group in sets of 16 bytes, then issue these commands to the bus pirate:

    [ 0xa0 0x00 <byte> ... <byte> ] %:500
    [ 0xa0 0x10 <byte> ... <byte> ] %:500
    ...
    [ 0xa0 0xf0 <byte> ... <byte> ] %:500
    [ 0xa2 0x00 <byte> ... <byte> ] %:500
    [ 0xa2 0x10 <byte> ... <byte> ] %:500
    ...
    [ 0xa2 0xf0 <byte> ... <byte> ] %:500
    [ 0xa4 0x00 <byte> ... <byte> ] %:500
    ...
    [ 0xa4 0xf0 <byte> ... <byte> ] %:500
    [ 0xa6 0x00 <byte> ... <byte> ] %:500
    ...
    [ 0xa6 0xf0 <byte> ... <byte> ]

And that's all. Note the address bytes are 0xa0, 0xa2, 0xa4, 0xa6, each is repeated 16 times. The second byte is the address within that block, in 16 increments each (since we're giving 16 bytes in each command). In total, 16 bytes x 16 addrs per block x 4 blocks = 1024 bytes = THE WHOLE ADDRESS SPACE! See `write-mac-flash.txt` with an example.

## Software

I haven't played with SW yet. Some info can be found [in this blogpost](http://alvarop.com/2013/08/kinesis-freestyle-2-keyboard-mod-to-fix-media-keys/).
