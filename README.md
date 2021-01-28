![](./resources/official_armmbed_example_badge.png)
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

``` commandline
mbed import mbed-os-example-blockdevice
cd mbed-os-example-blockdevice
```

#### Compile the example

Invoke `mbed compile`, and specify the name of your platform and your favorite
toolchain (`GCC_ARM`, `ARM`, `IAR`). For example, for the ARM Compiler 5:

``` commandline
mbed compile -m K64F -t ARM
```

Your PC may take a few minutes to compile your code. At the end, you see the
following result:

``` commandline
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

``` commandline
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

``` commandline
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
Mbed-OS supports a variety of block device types, more information on supported devices can be found [here](https://os.mbed.com/docs/mbed-os/latest/apis/data-storage.html#Default-BlockDevice-configuration).

Each device is represented by a C++ class that inherits from the interface class [BlockDevice](https://os.mbed.com/docs/mbed-os/latest/apis/blockdevice-apis.html). These classes take their default configuration from the component configuration file. This may be found in `/mbed-os/storage/blockdevice/` under the path corresponding to the block device type—for example [mbed_lib.json](https://github.com/ARMmbed/mbed-os/blob/master/storage/blockdevice/COMPONENT_SPIF/mbed_lib.json). 

In this example, you can determine which block device is used by modifying the type of `bd` in main.cpp. For instance, if instead you wanted to use a SPIF block device you would declare `bd` as an SPIFBlockDevice instance. 
```diff
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
You may need to make modifications to the application configuration file if you're using a physical storage device that isn't included in your target's default configuration (check in `targets.json` for this). To do this, add your physical storage device as a component in `mbed_app.json` as follows:
``` json
   "target_overrides": {
         "K64F": {
             "target.components_add": ["SPIF"],
         }
     }
```
You can also modify the pin assignments for your component as follows:
``` json
   "target_overrides": {
         "K64F": {
             "target.components_add": ["SPIF"],
             "spif-driver.SPI_MOSI": "PC_12",
             "spif-driver.SPI_MISO": "PC_11",
             "spif-driver.SPI_CLK":  "PC_10",
             "spif-driver.SPI_CS":   "PA_15"
         }
     }
```
Alternatively, you may use the system's default block device `BlockDevice *bd = BlockDevice::get_default_instance()` but this will require more code changes to the example.
# Tested configurations

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
## License and contributions

The software is provided under Apache-2.0 license. Contributions to this project are accepted under the same license. Please see [contributing.md](CONTRIBUTING.md) for more info.

This project contains code from other projects. The original license text is included in those source files. They must comply with our license guide.


