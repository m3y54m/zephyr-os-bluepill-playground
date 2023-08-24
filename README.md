# Getting started with Zephyr RTOS on BluePill board

## Project Initialization

Create a new project using PlatformIO with the following settings:

- Board: BluePill F103C8 (Generic)
- Framework: Zephyr RTOS

## Upload

### Using ST-LINK

There are several ways to upload the firmware to this board but the default way is to use the [ST-Link debugger](https://stm32-base.org/guides/connecting-your-debugger.html). 

In linux, you can use the following commands to install the ST-Link drivers:

```console
sudo apt -y install stlink-tools
sudo systemctl restart udev
```

Modify `platformio.ini` file to set `stlink` as default upload method:

```
upload_protocol = stlink
```

Now you can `Upload` the firmware to the board using [PlaformIO](https://platformio.org/).

You can also use `st-flash` tool for programming using ST-Link in linux:

```console
st-flash write firmware.bin 0x8000000
```

### Using UART and Embedded Bootloader

All the STM32 microcontrollers come with built-in bootloaders that burned in during production.

![memory-mapping](https://github.com/m3y54m/zephyr-os-bluepill-playground/assets/1549028/9bf5569e-b50e-4b1a-a864-6f612b9ceae4)


A couple of special MCU pins has to be set-up to proper logical values to enter the bootloader. The pins are named BOOT0 and BOOT1 on the STM32 microcontroller. Boot pins can select several modes of bootloader operation:

| BOOT1  | BOOT0  | Boot Mode         | Aliasing                                    |
| ------ | ------ | ----------------- | ------------------------------------------- |
| X      | 0      | Main Flash Memory | Main flash memory is selected as boot space |
| 0      | 1      | System Memory     | System memory is selected as boot space     |
| 1      | 1      | Embedded SRAM     | Embedded SRAM is selected as boot space     |

![boot-mode](https://github.com/m3y54m/zephyr-os-bluepill-playground/assets/1549028/61dce9f7-215b-49e4-8745-91ad020d334f)

As you can see, there are three cases:

- The first case is when the BOOT0 pin is tied to the ground, and BOOT1 is open or at a logical state of 0 after the reset program is executed from Main Flash Memory. Grounded BOOT pins are a standard configuration when executing programs in the field.
- The second case (BOOT1=0; BOOT0=1) means that after reset execution starts at System memory were built into bootloader resides. This is the case when we need to upload binaries via USART1.
- The third case means that program execution is performed in SRAM.

Read [this article](https://scienceprog.com/flashing-programs-to-stm32-embedded-bootloader/) to understand how you can use UART1 for programming this board: 

## Supported Features in Zephyr

The stm32\_min\_dev board configuration supports the following hardware
features:

| Interface | Controller | Driver/Component                     |
| --------- | ---------- | ------------------------------------ |
| NVIC      | on-chip    | nested vectored interrupt controller |
| SYSTICK   | on-chip    | system clock                         |
| UART      | on-chip    | serial port                          |
| GPIO      | on-chip    | gpio                                 |
| I2C       | on-chip    | i2c                                  |
| PWM       | on-chip    | pwm                                  |
| SPI       | on-chip    | spi                                  |
| USB       | on-chip    | USB device                           |
| ADC       | on-chip    | adc                                  |

Other hardware features are not supported by the Zephyr kernel.

### Connections and IOs

#### Default Zephyr Peripheral Mapping:

  - UART\_1 TX/RX: PA9/PA10
  - UART\_2 TX/RX: PA2/PA3
  - UART\_3 TX/RX: PB10/PB11
  - I2C\_1 SCL/SDA : PB6/PB7
  - I2C\_2 SCL/SDA : PB10/PB11
  - PWM\_1\_CH1: PA8
  - SPI\_1 NSS\_OE/SCK/MISO/MOSI: PA4/PA5/PA6/PA7
  - SPI\_2 NSS\_OE/SCK/MISO/MOSI: PB12/PB13/PB14/PB15
  - USB\_DC DM/DP: PA11/PA12
  - ADC\_1: PA0

#### System Clock

The on-board 8Mhz crystal is used to produce a **72Mhz** system clock with
PLL.

#### Serial Port

STM32 Minimum Development Board has 3 U(S)ARTs. The Zephyr console
output is assigned to UART_1. Default settings are **115200 8N1**.

#### On-Board LEDs

The board has one on-board LED that is connected to PC13.

## Pinout

![bluepill-pinout](https://github.com/m3y54m/zephyr-os-bluepill-playground/assets/1549028/9ed22ff4-b452-4d14-81eb-e22531671370)

## Schematics

![bluepill-schematic](https://github.com/m3y54m/zephyr-os-bluepill-playground/assets/1549028/5cc9dd1b-9f18-4a7a-9086-c50335ad85d4)


## USB to Serial Cable

![pl2303hxd-cable](https://github.com/m3y54m/zephyr-os-bluepill-playground/assets/1549028/148db7e0-329e-42b2-8abb-eb73640658fc)

USB to Serial (UART) cable is used to connect the board to a PC.

| Blue Pill | USB to Serial |
| --------- | ------------- |
| A9 (TX1)  | RXD           |
| A10 (RX1) | TXD           |
| G         | GND           |


### Access to Serial Port in Linux

In order to upload the compiled program to your board you should have access to serial ports. This is done by adding your user to `dialout` and `tty` groups:

```console
sudo usermod -a -G dialout $USER
sudo usermod -a -G tty $USER
```

You can verify if your user is added to `dialout` and `tty` groups using this command:

```console
groups $USER
```

Note: You should log out and log in or reboot your computer to apply the changes.


## Resources

- [Blue Pill - STM32F103C8T6](https://stm32-base.org/boards/STM32F103C8T6-Blue-Pill.html)
- [Getting Started with STM32F103C8T6 Blue Pill](https://www.electronicshub.org/getting-started-with-stm32f103c8t6-blue-pill/)
- [PlatformIO BluePill F103CB](https://docs.platformio.org/en/latest/boards/ststm32/bluepill_f103c8.html)
- [Connecting ST-LINK debugger](https://stm32-base.org/guides/connecting-your-debugger.html)
- [How to fix PlatformIO STM32 Error: libusb_open() failed with LIBUSB_ERROR_ACCESS](https://techoverflow.net/2021/09/22/how-to-fix-platformio-stm32-error-libusb_open-failed-with-libusb_error_access/)
- [Zephyr for BluePill Documentation](https://docs.zephyrproject.org/latest/boards/arm/stm32_min_dev/doc/index.html)
- [Zephyr Blinky Sample Application](https://github.com/zephyrproject-rtos/zephyr/blob/main/samples/basic/blinky/src/main.c)
- [Zephyr Tutorial for Beginners](https://github.com/maksimdrachov/zephyr-rtos-tutorial)
- [IoT RTOS Zephyr on cheap STM32 Minimum Development Board](https://embedjournal.com/iot-rtos-zephyr-stm32-minimum-system-development-board/)
- [Bootloader for STM32F103 boards, to use with the Arduino_STM32 repo and the Arduino IDE](https://github.com/rogerclarkmelbourne/STM32duino-bootloader/blob/master/bootloader_only_binaries/generic_boot20_pc13.bin)
- [DFU Bootloader for STM32 chips](https://github.com/devanlai/dapboot)
- [AN2606 Application note: STM32 microcontroller system memory boot mode](https://www.st.com/content/ccc/resource/technical/document/application_note/b9/9b/16/3a/12/1e/40/0c/CD00167594.pdf/files/CD00167594.pdf/jcr:content/translations/en.CD00167594.pdf)
- [Accessing Devices without Sudo](https://elinux.org/Accessing_Devices_without_Sudo)
- [Programming STM32F103 Blue Pill using USB Bootloader and PlatformIO](https://coytbarringer.com/programming-stm32f103-blue-pill-using-usb-bootloader-platformio/)
