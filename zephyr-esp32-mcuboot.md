# Setup Zephyr for esp32 with MCUboot

This writeup will describe how to setup an environment to build MCUboot for esp32 as well as an demo application which connects to a local Wi-Fi.

MCUboot support for esp32 in Zephyr was just recently added to upstream Zephyr PR[#37872](https://github.com/zephyrproject-rtos/zephyr/pull/37872). As of writing this is still a moving target so some details may be already outdated.

## Install required packages

Install the Zephyr requirements and SDK according to the official [Getting Started Guide](https://docs.zephyrproject.org/latest/getting_started/index.html).

I installed all Python requirements globally on my system to avoid issues with the ESP-IDF requirements.
Open a new terminal to check that the setup still works.

```
west init ~/zephyrproject
cd ~/zephyrproject
west update
```

Install the ESP-IDF [2].
```
west espressif install
export ZEPHYR_TOOLCHAIN_VARIANT="espressif"
export ESPRESSIF_TOOLCHAIN_PATH="${HOME}/.espressif/tools/xtensa-esp32-elf/esp-2020r3-8.4.0/xtensa-esp32-elf"
west espressif update
```

## Build MCUboot

```
cd bootloader/mcuboot/
git submodule update --init --recursive boot/espressif/hal/esp-idf
git submodule update --init --recursive ext/mbedtls

cd boot/espressif/hal/esp-idf
./install.sh
. ./export.sh
cd ../..

cd boot/espressif/
cmake -DCMAKE_TOOLCHAIN_FILE=tools/toolchain-esp32.cmake -DMCUBOOT_TARGET=esp32 -B build -GNinja
cmake --build build/
```


Convert elf to bin
```
esptool.py --chip esp32 elf2image --flash_mode dio --flash_freq 40m -o build/mcuboot_esp32.bin build/mcuboot_esp32.elf
```

## Build Zephyr application
Before building the sample application MCUboot support must be enabled.
```
echo "CONFIG_BOOTLOADER_MCUBOOT=y" >> zephyr/samples/boards/esp32/wifi_station/prj.conf
```

```
west build -b esp32 -p auto zephyr/samples/boards/esp32/wifi_station
```

## Flash to supported boards
Flash MCUboot
```
esptool.py -p /dev/ttyUSB0 -b 2000000 --before default_reset --after hard_reset --chip esp32 write_flash --flash_mode dio --flash_size detect --flash_freq 40m 0x1000 build/mcuboot_esp32.bin
```

To flash the Zephyr application just type
```
west flash
```
in the Zephyr workspace folder.
## References

 1. https://www.embarcados.com.br/primeiros-passos-com-esp32-utilizando-mcuboot-como-bootloader/
 2. https://docs.zephyrproject.org/2.7.0/boards/xtensa/esp32/doc/index.html
