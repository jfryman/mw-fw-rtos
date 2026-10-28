# mw-fw-rtos

> **This is the MegaWiFi Rev.C fork** of [doragasu/mw-fw-rtos](https://gitlab.com/doragasu/mw-fw-rtos).
> The `rev-c` branch runs the cart link at **1.25 Mbaud** (`LSD_UART_BR` in `main/lsd.h`), because
> Rev.C carts ([jfryman/mw-cart](https://github.com/jfryman/mw-cart)) use a TI TL16C550D clocked at
> 20 MHz instead of the NXP SC16C550B at 24 MHz. That is the only change. Use this firmware **only**
> on Rev.C carts; stock carts need upstream's 1.5 Mbaud build. Pre-built ESP32-C3 images are on the
> [releases page](https://github.com/jfryman/mw-fw-rtos/releases). Flash over the cart's USB-C port:
> `esptool.py --chip esp32c3 -p <port> write_flash 0x0 mw-fw-rtos-merged.bin`.

Firmare for the ESP8266 WiFi module installed in MegaWiFi cartridges. This firmware talks directly to the MegaWiFi API (mw-api) running on the Genesis/MegaDrive console, allowing it to connect to WiFi access points, and to send and receive data through The Internet using standard TCP and UDP protocols. There are some additional goodies provided by the firmware, like flash read/write functions and SNTP time synchronization.

# Obtaining pre-built firmware files
Pre-built firmware archives can be found in [the package registry](https://gitlab.com/doragasu/mw-fw-rtos/-/packages). You can use a [MegaWiFi programmer](https://gitlab.com/doragasu/mw-prog) and the [MegaDrive Memory Administration client](https://gitlab.com/doragasu/mw-mdma-cli) to burn this files to the ESP8266 in a MegaWiFi cartridge. Go to the [Flashing section](#flashing) for more details. Typically you will only need to burn the latest version of the firmware once, because from that point, firmware updates can be installed Over-The-Air.

# Building

This firmware is based on [ESP8266\_RTOS\_SDK](https://github.com/espressif/ESP8266_RTOS_SDK). Follow the SDK instructions to install the toolchain and build the firmware. Currently this project builds against `release/v3.4` branch, so make sure to check it out. If you have Docker installed, the easiest way to build the firmware is using my `esp-dockerfiles` images. From the project diretory run:

```bash
$ docker run --rm -v$PWD:/home/esp/src -t registry.gitlab.com/doragasu/esp-dockerfiles/esp8266_rtos_sdk:release_v3.4 -c 'cd src && make -j$(nproc)'
```

The first time you run it, Docker will pull the image and perform the build. On Windows, you might have to replace `$PWD` with `%CD%`. I know, Windows is weird.

# Flashing

The recommended way of burning the firmware to the ESP8266 embedded on a MegaWiFi cartridge is using a MegaWiFi Programmer along with `mw-mdma-cli` tool. If you installed `mw-mdma-cli` tool and edited the path as instructed above, you can burn the firmware just by connecting the cart and the programmer to a PC. The first time you burn the firmware, you have to write the bootloader, the partitions and the firmware blob itself. If you built the firmware from sources, you can edit the `Makefile` to set `MDMAP` variable pointing to the path of your `mdma` client tool, then run:

```bash
$ make boot
$ make partitions
$ make cart
```

Alternatively, since version 1.5, you can download the firmware files from the [GitLab package registry here](https://gitlab.com/doragasu/mw-fw-rtos/-/packages), and burn them using `mdma` as follows (assuming `mdma` is in your path):

```bash
$ mdma -w bootloader/bootloader.bin
$ mdma -w partitions.bin:0x8000
$ mdma -w ota_data_initial.bin:0xe000
$ mdma -w mw-fw-rtos.bin:0x10000
```

When you need to update the firmware, you just need to run the last command, no need to burn the boot and partitions again.

# Status

This firmware, paired with the `mw-api` running on a Megadrive/Genesis, allows the console to:

* Configure and associate to access points (including neighbor scan functions).
* Store up to 3 network configurations.
* Store up to 3 gamertags.
* Use TCP and UDP for transport.
* Create both client and server sockets.
* Perform HTTP/HTTPS requests.
* Synchronize the date/time from NTP servers.
* Generate random numbers blazingly fast.
* Store and read custom data on non volatile flash (up to 16 megabits are available in addition to the standard 32 megabits of the cart).
* Integrate with GameJolt for online scoreboards, trophies, user and friend management, cloud storage, etc.

# Authors

This program has been written by doragasu. This is part of [MegaWiFi project](https://github.com/doragasu/mw).

# Contributions

Contributions are welcome. If you find a bug please open an issue, and if you have implemented a cool feature/improvement, please send a pull request.

# License

This program is provided with NO WARRANTY, under the [GPLv3 license](https://www.gnu.org/licenses/gpl-3.0.html). Make sure you also check `ESP8266_RTOS_SDK` for its license terms.

