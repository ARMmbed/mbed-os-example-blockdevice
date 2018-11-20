# Getting started with the Mbed OS block device API

This example demonstrates how to use the Mbed OS block device API.

You can find more information about the Mbed OS block device and other related pieces of the Mbed OS storage stack [in the storage overview](https://os.mbed.com/docs/latest/reference/storage.html).

**Table of contents:**

1. [Hardware requirements](#hardware-requirements)
1. [Usage](#usage)
   - [Import the example](#import-the-example)
   - [Compile the example](#compile-the-example)
   - [Run the example](#run-the-example)
   - [Troubleshooting](#troubleshooting)
1. [Changing the block device](#changing-the-block-device)
1. [Tested configurations](#tested-configurations)

## Hardware requirements

This example uses a block device as storage. This can be either an external
block device (one of SPI flash, DataFlash or an SD card) or simulated on a
heap block device on boards with enough RAM.

By default, this example uses an instance of the SPIFBlockDevice, which requires
external SPI flash. The [changing the block device](#changing-the-block-device)
section describes how to change the file system or block device in the example.

## Usage

#### Import the example

Make sure you have an Mbed development environment set up. [Get started with Mbed OS](https://os.mbed.com/docs/latest/tutorials/your-first-program.html)
to set everything up.

From the command-line, import the example:

```
mbed import mbed-os-example-blockdevice
cd mbed-os-example-blockdevice
```

#### Compile the example

Invoke `mbed compile`, and specify the name of your platform and your favorite
toolchain (`GCC_ARM`, `ARM`, `IAR`). For example, for the ARM Compiler 5:

```
mbed compile -m K64F -t ARM
```

Your PC may take a few minutes to compile your code. At the end, you see the
following result:

```
[snip]
+--------------------------+-------+-------+-------+
| Module                   | .text | .data |  .bss |
+--------------------------+-------+-------+-------+
| Fill                     |   164 |     0 |  2136 |
| Misc                     | 54505 |  2556 |   754 |
| drivers                  |   640 |     0 |    32 |
| features/filesystem      | 15793 |     0 |   550 |
| features/storage         |    42 |     0 |   184 |
| hal                      |   418 |     0 |     8 |
| platform                 |  2355 |    20 |   582 |
| rtos                     |   135 |     4 |     4 |
| rtos/rtx                 |  5861 |    20 |  6870 |
| targets/TARGET_Freescale |  8382 |    12 |   384 |
| Subtotals                | 88295 |  2612 | 11504 |
+--------------------------+-------+-------+-------+
Allocated Heap: 24576 bytes
Allocated Stack: unknown
Total Static RAM memory (data + bss): 14116 bytes
Total RAM memory (data + bss + heap + stack): 38692 bytes
Total Flash memory (text + data + misc): 91947 bytes

Image: ./BUILD/K64F/ARM/mbed-os-example-blockdevice.bin
```

#### Run the example

1. Connect your Mbed Enabled device to the computer over USB.
1. Copy the binary file to the Mbed Enabled device.
1. Press the reset button to start the program.
1. Open the UART of the board in your favorite UART viewing program. For
   example, `screen /dev/ttyACM0`.
   
**Note:** The default serial port baud rate is 9600 bit/s.

Expected output:

```
--- Block device geometry ---
read_size:    1 B
program_size: 1 B
erase_size:   4096 B
size:         4194304 B
---
bd.read(0x20000410, 0, 15)
bd.read -> 0
--- Stored data ---
f8 ff ff ff 0f 00 00 00 00 00 00 00 00 00 00
---
bd.erase(0, 4096)
bd.erase -> 0
bd.program(0x20000410, 0, 15)
bd.program -> 0
bd.read(0x20000410, 0, 15)
bd.read -> 0
--- Stored data ---
48 65 6c 6c 6f 20 53 74 6f 72 61 67 65 21 00     Hello Storage!
---
bd.deinit()
bd.deinit -> 0
--- done! ---
```

You can also reset the board to see the data persist across boots:

```
--- Block device geometry ---
read_size:    1 B
program_size: 1 B
erase_size:   4096 B
size:         4194304 B
---
bd.read(0x20000410, 0, 15)
bd.read -> 0
--- Stored data ---
48 65 6c 6c 6f 20 53 74 6f 72 61 67 65 21 00     Hello Storage!
---
bd.erase(0, 4096)
bd.erase -> 0
bd.program(0x20000410, 0, 15)
bd.program -> 0
bd.read(0x20000410, 0, 15)
bd.read -> 0
--- Stored data ---
48 65 6c 6c 6f 20 53 74 6f 72 61 67 65 21 00     Hello Storage!
---
bd.deinit()
bd.deinit -> 0
--- done! ---
```

Try changing the string "Hello Storage!" to a message of your choice to see it
stored on the block device.

#### Troubleshooting

If you have problems, you can review the [documentation](https://os.mbed.com/docs/latest/tutorials/debugging.html)
for suggestions on what could be wrong and how to fix it.

## Changing the block device

In Mbed OS, a C++ classes that inherits from the [BlockDevice](https://os.mbed.com/docs/latest/reference/storage.html#block-devices)
interface represents each block device. You can change the filesystem in the
example by changing the class declared in main.cpp.

``` diff
-SPIFBlockDevice bd(
-        MBED_CONF_SPIF_DRIVER_SPI_MOSI,
-        MBED_CONF_SPIF_DRIVER_SPI_MISO,
-        MBED_CONF_SPIF_DRIVER_SPI_CLK,
-        MBED_CONF_SPIF_DRIVER_SPI_CS);
+SDBlockDevice bd(
+        MBED_CONF_SD_SPI_MOSI,
+        MBED_CONF_SD_SPI_MISO,
+        MBED_CONF_SD_SPI_CLK,
+        MBED_CONF_SD_SPI_CS);
```

**Note:** Most block devices require pin assignments. Double check that the
pins in `<driver>/mbed_lib.json` are correct. For example, to change the pins for the SD driver, open `sd-driver/config/mbed_lib.json`, and change your target platform to the correct pin-out in the `target_overrides` configuration.

Starting mbed-os 5.10 the SD, SPIF, DATAFLASH and QSPIF block devices are components under mbed-os. In order to add a component to the application use the "components_add" `target_overrides` configuration:

```
   "target_overrides": {
         ...
         "NUCLEO_F429ZI": {
             "components_add": ["SPIF"],
             "SPI_MOSI": "PC_12",
             "SPI_MISO": "PC_11",
             "SPI_CLK":  "PC_10",
             "SPI_CS":   "PA_15"
         },
         ...
     }
```

The components_add param can be "SPIF", "SD" , "DATAFLASH" or "QSPIF" depends on the block devices you need.

Mbed OS has several options for the block device:

- **SPIFBlockDevice** - Block device driver for NOR-based SPI flash devices that
  support SFDP. NOR-based SPI flash supports byte-sized read and writes, with an
  erase size of about 4kbytes. An erase sets a block to all 1s, with successive
  writes clearing set bits.

  ``` cpp
  SPIFBlockDevice bd(
          MBED_CONF_SPIF_DRIVER_SPI_MOSI,
          MBED_CONF_SPIF_DRIVER_SPI_MISO,
          MBED_CONF_SPIF_DRIVER_SPI_CLK,
          MBED_CONF_SPIF_DRIVER_SPI_CS);
  ```

- **QSPIFBlockDevice** - Block device driver for NOR-based Quad SPI flash devices that
  support SFDP, with QUAD SPI bus support for 4 bits per cycle (4 times the speed of standard SPI) 

  ``` cpp
  QSPIFBlockDevice bd(
          QSPI_FLASH1_IO0, 
          QSPI_FLASH1_IO1, 
          QSPI_FLASH1_IO2, 
          QSPI_FLASH1_IO3,
          QSPI_FLASH1_SCK, 
          QSPI_FLASH1_CSN, 
          QSPIF_POLARITY_MODE_0, 
          MBED_CONF_QSPIF_QSPI_FREQ);          
  ```

- **DataFlashBlockDevice** - Block device driver for NOR-based SPI flash devices
  that support the DataFlash protocol, such as the Adesto AT45DB series of
  devices. DataFlash is a memory protocol that combines flash with SRAM buffers
  for a programming interface. DataFlash supports byte-sized read and writes, with
  an erase size of about 528 bytes or sometimes 1056 bytes. DataFlash provides
  erase sizes with an extra 16 bytes for error correction codes (ECC), so a flash
  translation layer (FTL) may still present 512 byte erase sizes.
  
  ``` cpp
  DataFlashBlockDevice bd(
          MBED_CONF_DATAFLASH_SPI_MOSI,
          MBED_CONF_DATAFLASH_SPI_MISO,
          MBED_CONF_DATAFLASH_SPI_CLK,
          MBED_CONF_DATAFLASH_SPI_CS);
  ```

- **SDBlockDevice** - Block device driver for SD cards and eMMC memory chips. SD
  cards or eMMC chips offer a full FTL layer on top of NAND flash. This makes the
  storage well-suited for systems that require a about 1GB of memory.
  Additionally, SD cards are a popular form of portable storage. They are useful
  if you want to store data that you can access from a PC.
  
  ``` cpp
  SDBlockDevice bd(
          MBED_CONF_SD_SPI_MOSI,
          MBED_CONF_SD_SPI_MISO,
          MBED_CONF_SD_SPI_CLK,
          MBED_CONF_SD_SPI_CS);
  ```

- [**HeapBlockDevice**](https://os.mbed.com/docs/v5.6/reference/heapblockdevice.html) -
  Block device that simulates storage in RAM using the heap. Do not use the heap
  block device for storing data persistently because a power loss causes
  complete loss of data. Instead, use it fortesting applications when a storage
  device is not available.
  
  ``` cpp
  HeapBlockDevice bd(1024*512, 512);
  ```

Additionally, Mbed OS contains several utility block devices to give you better
control over the allocation of storage.

- [**SlicingBlockDevice**](https://os.mbed.com/docs/latest/reference/slicingblockdevice.html) -
  With the slicing block device, you can partition storage into smaller block
  devices that you can use independently.

- [**ChainingBlockDevice**](https://os.mbed.com/docs/latest/reference/chainingblockdevice.html) -
  With the chaining block device, you can chain multiple block devices together
  and extend the usable amount of storage.

- [**MBRBlockDevice**](https://os.mbed.com/docs/latest/reference/mbrblockdevice.html) -
  Mbed OS comes with support for storing partitions on disk with a Master Boot
  Record (MBR). The MBRBlockDevice provides this functionality and supports
  creating partitions at runtime or using preformatted partitions configured
  separately from outside the application.

- **ReadOnlyBlockDevice** - With the read-only block device, you can wrap a
  block device in a read-only layer, ensuring that user of the block device does
  not modify the storage.

- **ProfilingBlockDevice** - With the profiling block device, you can profile
  the quantity of erase, program and read operations that are incurred on a
  block device.

- **ObservingBlockDevice** - The observing block device grants the user the
  ability to register a callback on block device operations. You can use this to
  inspect the state of the block device, log different metrics or perform some
  other operation.

- **ExhaustibleBlockDevice** - Useful for evaluating how file systems respond to
  wear, the exhaustible block device simulates wear on another form of storage.
  You can configure it to expire blocks as necessary.

## Tested configurations

- K64F + Heap
- K64F + SD
- K64F + SPIF (requires shield)
- K64F + DataFlash (requires shield)
- UBLOX_EVK_ODIN_W2 \[1\] + Heap
- UBLOX_EVK_ODIN_W2 \[1\] + SD
- UBLOX_EVK_ODIN_W2 \[1\] + SPIF (requires shield)
- UBLOX_EVK_ODIN_W2 \[1\] + DataFlash (requires shield)
- NUCLEO_F429ZI + Heap
- NUCLEO_F429ZI + SD (requires shield)
- NUCLEO_F429ZI + SPIF (requires shield)
- NUCLEO_F429ZI + DataFlash (requires shield)
- DISCO_L475VG_IOT01A + QSPIF
- DISCO_L476VG + QSPIF
- DISCO_F413ZH + QSPIF
- DISCO_F469NI + QSPIF

\[1\]: Note: The UBLOX_EVK_ODIN_W2 SPI pins conflict with the default serial
pins. A different set of serial pins must be selected to use SPI flash with
serial output.

```c++
// Connect Tx, Rx, and ground pins to a separte board running the passthrough example:
// https://os.mbed.com/users/sarahmarshy/code/SerialPassthrough/file/2a3a62ee17fa/main.cpp/
Serial pc(TX, RX);   

pc.printf("...");    // Replace printf with pc.printf in the example
```

