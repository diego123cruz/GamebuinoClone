# GamebuinoClone
Home made Gamebuino Clone

__NOTOC__
=== Overview ===
The bootloader is a small program that permanently resides in Gamebuino memory which allows you to load games via SD card or USB port. Under normal circumstances loading a new game into Gamebuino would require you to use an external programmer connected to the Gamebuino's ICSP port. The bootloader makes this process easier and faster by eliminating the need for a separate hardware programmer.

The Gamebuino will be delivered with a bootloader already installed. This bootloader is a modified version of the Arduino UNO's "Optiboot" but with some extra features added. The first modification is a feature that launches the game section sketch (LOADER.HEX) when you reset the device holding down the C button. The second addition is the ability for any sketch to load and launch other games on the SD card by calling directly into the boot loader. A final feature is the ability for a sketch to pass a 128-byte "page" of data to the boot loader and have it flash that data into any part of Flash memory and then return to the calling program. This last feature, when used in conjunction with an SD library effectively eliminates the size restriction on game sketches and allows for arbitrarily sized data and code and opens the possibility for self-modifying code.

These extra features have been added without removing existing functionality so in all other respects the bootloader should run normally, meaning it is compatible with the official Arduino software and will support IDE uploads and program verification. And of course ICSP programming still works normally for those users wishing to program Gamebuino (or replace the bootloader with another) using a hardware programmer.

=== Sources ===
* [https://code.google.com/p/optiboot/]: The firmware used by the official Arduino IDE for flashing Uno boards.
* [https://github.com/thseiler/embedded/tree/master/avr/2boots 2boots], an SD-enabled bootloader source code, the Gamebuino bootloader uses it's FAT16 and flashing code.

=== Launching Loader ===
If the the C button is held down while the unit is powered-up then the bootloader will search for a compiled sketch on the SD card called "LOADER.HEX". Note that the SD must be formatted with FAT16 and the filename must be upper case characters.If no SD card is present, or if the LOADER.HEX file can't be found or loaded then the bootloader will execute whatever program is already flashed into memory.

=== Flashing the Gamebuino Bootloader ===
Under normal operation the Gamebuino is programmed via the USB virtual COM port with the device selected as "Arduino UNO". However, if you wish to program it or upload the bootloader using an external hardware programmer  (e.g. another Arduino) then you must edit the boards.txt file in the Arduino package (typically loacted at C:\arduino-1.0.5\hardware\arduino\boards.txt) and add the following section:

 <nowiki>    gamebuino.name=Gamebuino
    gamebuino.upload.protocol=arduino
    gamebuino.upload.maximum_size=30592
    gamebuino.upload.speed=115200
    gamebuino.bootloader.low_fuses=0xff
    gamebuino.bootloader.high_fuses=0xda
    gamebuino.bootloader.extended_fuses=0x05
    gamebuino.bootloader.path=gamebuino_boot
    gamebuino.bootloader.file=gamebuino_boot.hex
    gamebuino.bootloader.unlock_bits=0x3F
    gamebuino.bootloader.lock_bits=0x0F
    gamebuino.build.mcu=atmega328p
    gamebuino.build.f_cpu=16000000L
    gamebuino.build.core=arduino
    gamebuino.build.variant=standard</nowiki>

Restart the Arduino IDE and you will see a new device type called "Gamebuino". (This change is required for external programming because the Gamebuino bootloader is larger than the original Optiboot bootloader and the fuse bits need to be modified to accommodate this).

=== Launching Games via Software ===
The boot loader can be invoked at any time to load and launch a game from the SD card. To do this simply add the following code to any sketch:

 <nowiki>    #define load_game (*((void(*)(const char* filename))(0x7ffc/2)))</nowiki>

A game can then be loaded and launched by calling this function, passing in an upper-case-only file name without the .HEX extension:

 <nowiki>    load_game("BLINK");</nowiki>

This causes a long-jump into the boot loader code causing it to load and launch BLINK.HEX from the SD card.

=== Self-Flashing with the Bootloader ===

The boot loader also allows the application to flash pages in application space which can be used for game patching or self-modifying code. The define to use this feature is as follows:

<pre>    #define write_flash_page (*((void(*)(prog_char * page, unsigned char * buffer))(0x7ffa/2)))</pre>

The "page" parameter is the Flash address of the page to be flashed, "buffer" is the address of a 128-byte buffer in SRAM containing the data to be flashed. (The "0x7ffa/2" is a fixed constant representing the address of the function call in the boot loader memory).

An example use of this function is to set up a large swap-buffer in PROGMEM which you then flash with a single-page buffer in SRAM:

<pre>    #define PAGE_SIZE(x) (((x) + (SPM_PAGESIZE-1)) & ~(SPM_PAGESIZE-1))
    unsigned char page_buffer[SPM_PAGESIZE];
    prog_char swap_buffer[PAGE_SIZE(the_actual_size_you_want)] PROGMEM __attribute__((aligned(SPM_PAGESIZE)));
    
    write_flash_page(&swap_buffer[some_128_aligned_offset], page_buffer);</pre>

Flashed pages can then be read using the regular C commands used to read PROGMEM memory.

=== Questions, suggestions, solutions ? ===
If you have any questions, ask them the in [http://gamebuino.com/forum/viewtopic.php?f=12&t=109 this thread].

Good luck !

=== Download ===

The current alpha version of the boot loader can be downloaded here (right click => save link as): [https://raw.githubusercontent.com/Rodot/Gamebuino/master/hardware/gamebuino/bootloaders/gamebuino_boot/gamebuino_boot.hex gamebuino_boot.hex]

All the bootloader files and examples are on [https://github.com/Rodot/Gamebuino/tree/master/hardware/gamebuino/bootloaders/gamebuino_boot Gamebuino's git].
