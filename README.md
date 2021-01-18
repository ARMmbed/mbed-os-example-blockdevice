![](./resources/official_armmbed_example_badge.png)
# Block Device API Mbed OS example

The example project is part of the [Arm Mbed OS Official Examples](https://os.mbed.com/code/). It contains an application that demonstrates how to use a block device as storage on supported [Mbed boards](https://os.mbed.com/platforms/).

You can build the project with all supported [Mbed OS build tools](https://os.mbed.com/docs/mbed-os/latest/tools/index.html). However, this example project specifically refers to the command-line interface tool [Arm Mbed CLI](https://github.com/ARMmbed/mbed-cli#installing-mbed-cli).
(Note: To see a rendered example you can import into the Arm Online Compiler, please see our [import quick start](https://os.mbed.com/docs/mbed-os/latest/quick-start/online-with-the-online-compiler.html#importing-the-code).)



1. [Install Mbed CLI](https://os.mbed.com/docs/mbed-os/latest/quick-start/offline-with-mbed-cli.html).
1. From the command-line, import the example: `mbed import mbed-os-example-blockdevice`
1. Change the current directory to where the project was imported: `cd mbed-os-example-blockdevice`
1. [Changing the block device](#changing-the-block-device)
1. [Tested configurations](#tested-configurations)

## Application functionality
This example demonstrates how to read and write data into a storage device. This can be either an external block device (one of SPI flash, DataFlash or an SD card) or simulated on a heap block device on boards with enough RAM.

By default, an instance of the SDBlockDevice is used, this requires an external SD card. The [changing the block device](#changing-the-block-device) section describes how to change this. 

#### Building and running

1. Connect a USB cable between the USB port on the board and the host computer.
1. Run the following command to build the example project and program the microcontroller flash memory:

    ```bash
    mbed compile -m <TARGET> -t <TOOLCHAIN> --flash
    ```

Your PC may take a few minutes to compile your code.

Alternatively, you can manually copy the binary to the board, which you mount on the host computer over USB. The binary is located at `./BUILD/<TARGET>/<TOOLCHAIN>/mbed-os-example-blockdevice.bin`.

Depending on the target, you can build the example project with the `GCC_ARM`, or `ARM` toolchain. Run the command below to determine which toolchain supports your target:
```bash
mbed compile -S
```
Once you have programmed your board, open the UART of the board in your preferred UART viewing program. To launch the serial console provided by Mbed, run the command `mbedtools sterm`. If you don't see any output in the console press the reset button to restart the program.

**Note:** The default serial port baud rate is 9600 bit/s.

## Expected output

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

If you see garbled text from the read operations check that your SD card is securely fixed in the port. If there's no SD card mounted the application returns garbage characters.  
## Troubleshooting
 If you have problems, you can review the [documentation](https://os.mbed.com/docs/latest/tutorials/debugging.html) for suggestions on what could be wrong and how to fix it.

## Changing the block device
Mbed-OS supports a variety of block device types, more information on supported devices can be found [here](https://os.mbed.com/docs/mbed-os/latest/apis/data-storage-concepts.html).

Each device is represented by a C++ class that inherits from the interface class [BlockDevice](https://os.mbed.com/docs/mbed-os/latest/apis/blockdevice-apis.html). These classes take their default configuration from the component configuration file. This may be found in `/mbed-os/storage/blockdevice/` under the path corresponding to the block device type—for example [mbed_lib.json](https://github.com/ARMmbed/mbed-os/blob/master/storage/blockdevice/COMPONENT_SPIF/mbed_lib.json). 

In this example, you can determine which block device is used by modifying the type of `bd` in main.cpp. For instance, if instead you wanted to use a SPIF block device you would declare `bd` as an SPIFBlockDevice instance. 
```diff
-SDBlockDevice bd(
-        MBED_CONF_SD_SPI_MOSI,
-        MBED_CONF_SD_SPI_MISO,
-        MBED_CONF_SD_SPI_CLK,
-        MBED_CONF_SD_SPI_CS);
+SPIFBlockDevice bd(
+        MBED_CONF_SPIF_DRIVER_SPI_MOSI,
+        MBED_CONF_SPIF_DRIVER_SPI_MISO,
+        MBED_CONF_SPIF_DRIVER_SPI_CLK,
+        MBED_CONF_SPIF_DRIVER_SPI_CS);
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

``` c++
// Connect Tx, Rx, and ground pins to a separte board running the passthrough example:
// https://os.mbed.com/users/sarahmarshy/code/SerialPassthrough/file/2a3a62ee17fa/main.cpp/
Serial pc(TX, RX);   

pc.printf("...");    // Replace printf with pc.printf in the example
```
### License and contributions

The software is provided under Apache-2.0 license. Contributions to this project are accepted under the same license. Please see [contributing.md](CONTRIBUTING.md) for more info.

This project contains code from other projects. The original license text is included in those source files. They must comply with our license guide.
