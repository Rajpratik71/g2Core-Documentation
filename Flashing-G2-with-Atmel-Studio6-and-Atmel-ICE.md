_This page is for uploading an already compiled G2 binary to a target board using Atmel Studio 6.2 (or greater) and the Atmel ICE programmer/debugger. This requires Windows or a VM for AS6. Please see [Getting Started with G2](Getting-Started-with-G2) for information about other options._

###Step 1 - Get the TinyG2.bin file

**Option 1** - Compile your own using the instruction in [[Compiling G2 on Windows (Atmel Studio 6.2)]]

**Option 2** - Get the tinyg2.bin binary firmware files from http://synthetos.github.io/g2/

###Step 2 - Program TinyG2 onto the Due Using Atmel-ICE
Select the Tools / Device Programming menu. 
_I've omitted a lot of steps, but there are some things to watch for_
* Make sure the ribbon cable is plugged into the SAM port, not the AVR port
* Make sure the connection from the ribbon cable to the 10 pin micro connector on the board is properly polarozed (red wire by the white dot), and contacts both rows of pins. It's easy to mess this up as the connector is so small.
* In the programing dialog box select SWD, not JTAG
* Program the fuses (GPNVM Bits) to boot from FLASH (not ROM), BANK 0