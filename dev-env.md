# Development Environment

## Cross Compiler

Options:

- [ ] [Linaro](https://www.linaro.org/downloads/) *(Precompiled)*
  - `arm-eabi` for linux `x86_64` hosts: [gcc-linaro-7.5.0-2019.12-x86_64_arm-eabi.tar.xz](https://releases.linaro.org/components/toolchain/binaries/latest-7/arm-eabi/gcc-linaro-7.5.0-2019.12-x86_64_arm-eabi.tar.xz)
- [x] [GNU Arm Embedded Toolchain](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm/downloads) *(Precompiled)*
  - `arm-none-eabi-gcc` for linux `x86_64` hosts: [gcc-arm-none-eabi-10-2020-q4-major-x86_64-linux.tar.bz2](https://developer.arm.com/-/media/Files/downloads/gnu-rm/10-2020q4/gcc-arm-none-eabi-10-2020-q4-major-x86_64-linux.tar.bz2?revision=ca0cbf9c-9de2-491c-ac48-898b5bbc0443&la=en&hash=68760A8AE66026BCF99F05AC017A6A50C6FD832A)
- [ ] `crossbuild-essential-armhf` *(Precompiled - Package) (From Raspberry Pi Guide)*
- [ ] `aarch64-linux-gnu-gcc` *(Precompiled - Package)*
- [ ] `arm-linux-gnueabihf-gcc` *(Precompiled - Package)*
- [ ] `gcc-arm-linux-gnueabihf` *(Precompiled - Package)*
- [ ] `crosstool-ng` *(Build from Source)*


### GNU Arm Embedded Toolchain
#### Download Toolchain
The cross compiler is called `arm-none-eabi-gcc`, and we can download it from [the ARM developer website](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm/downloads). You can also download it from the terminal by grabbing the latest link from the website for the correct host (Linux, Windows, MacOS) and passing it to `wget`.

From the home directory run the command:
```bash
wget https://developer.arm.com/-/media/Files/downloads/gnu-rm/10-2020q4/gcc-arm-none-eabi-10-2020-q4-major-x86_64-linux.tar.bz2?revision=ca0cbf9c-9de2-491c-ac48-898b5bbc0443&la=en&hash=68760A8AE66026BCF99F05AC017A6A50C6FD832A
```
#### Extract Toolchain
Run the following command in order to extract the files:
```bash
tar -xvf gcc-arm-none-eabi-10-2020-q4-major-x86_64-linux.tar.bz2
```
Now the extracted files will be in the `gcc-arm-none-eabi-10-2020-q4-major` directory.

You can also remove the `.tar`:
```bash
rm gcc-arm-none-eabi-10-2020-q4-major-x86_64-linux.tar.bz2
```

#### Add toolchain to PATH
In order for the toolchain to be accessible from every directory we need to add it to our `$PATH` variable permantly.

We need to define the export to the `$PATH` variable in the shell configuration files:
- `.bashrc` if you are using `bash`.
- `.zshrc` if you are using `zsh`.

Run the following command, that adds the export command at then end of your shell configuration file (here `.zshrc` for example):
```bash
echo 'export PATH="$HOME/gcc-arm-none-eabi-10-2020-q4-major/bin:$PATH"' >> $HOME/.zshrc
```

Load the new `$PATH` into the current shell session using the `source` command:
```bash
source ~/.zshrc
```

Now you can type `arm-none-eabi-gcc` from everywhere:
```bash
$ arm-none-eabi-gcc -v
...
gcc version 10.2.1 20201103 (release) (GNU Arm Embedded Toolchain 10-2020-q4-major)
```

#### Using Toolchain
In order to compile a `hello_world.c` program for an ARM target you run:
```bash
arm-none-eabi-gcc --specs=nosys.specs hello_world.c -o hello_world
```
Where `--specs=nosys.specs` is allowing semi-hosting.
- `-ffreestanding`: A freestanding environment is an environment in which the
    standard library may not exist, and program startup may not necessarily be
    at main. The option `-ffreestanding` directs the compiler to not assume
    that standard functions have their usual definition.

- `-fPIC`: Position Independent Code means that the generated machine code is not
    dependent on being located at a specific address in order to work.

- `-nostdlib`: Don't use the C standard library. Most of the calls in the C
    standard library eventually interact with the operating system.
    We are writing a bare-metal program, and we don't have any underlying
    OS, so the C standard library is not going to work for us anyway.

```bash
arm-none-eabi-gcc -c boot.S -o boot
```

## QEMU

Find QEMU version:
```bash
qemu-system-arm -version
```

List available CPUs that QEMU can emulate for `versatilepb` machine:
```bash
qemu-system-arm -M versatilepb -cpu help
```

With option:
```bash
-gdb tcp::26000
```
QEMU allows gdb to connect to the virtual machine for remote debugging through
a tcp socket with port 26000.

With option: `-S`, QEMU waits for gdb to connect before it starts running.



## Serial Connection with Raspberry PI
In order to remotely control your Raspberry Pi with a console cable you will
need:
- Raspberry Pi
- USB to TTL serial cable
- SD Card with Raspbian OS installed

**Guides:**

- [Adafruit's Raspberry Pi Lesson 5. Using a Console Cable, pdf](https://cdn-learn.adafruit.com/downloads/pdf/adafruits-raspberry-pi-lesson-5-using-a-console-cable.pdf)
- [Adafruit's Raspberry Pi Lesson 5. Using a Console Cable, web](https://learn.adafruit.com/adafruits-raspberry-pi-lesson-5-using-a-console-cable/overview)
- [UART configuration, raspberrypi.org](https://www.raspberrypi.org/documentation/configuration/uart.md)


### Enable Serial Console (for Raspbian)
You can enable/disable the serial console with either editing `/boot/config.txt`
or `raspi-config` (which will edit `/boot/config.txt` for you).

#### Option 1 - Enabling in `/boot/config.txt`
- Insert your SD card into a computer
- Edit `config.txt` with a text editor
```
vim media/{your-SD-name}/boot/config.txt
```
- At the bottom, last line, add `enable_uart=1` (for Pi >= 3) on it's own line:
```
enable_uart=1
```
in condext with rest lines in the file
```
#...
[all]
#dtoverlay=vc4-fkms-v3d
enable_uart=1
```

#### Option 2 - Enabling via `raspi-config`
- Boot your Raspberry Pi and log in to a shell and run:
```
sudo raspi-config
```
- Go to *Interface Options*
- Go to *P6 Serial Port*
- Select *Yes* on *You would like a login shell to be accessible over serial?*
- After that, it should be enabled.
- When it asks you to reboot, press *Yes*.
- If it is not show, reboot it yourself.


### Drivers for Linux (for Host)
Linux Kernels 2.4.31 and above already have the PL2303 and CP210X USB driver for
the Console Lead built-in, so you should not need to install that.

But you need to install the `screen` command if not installed on your distro.

Install `screen`:
```
sudo apt-get install screen
```
Maybe we would need it later.
Install `minicom`:
```
sudo apt-get install minicom
```

### Connect the USB to TTL Leads
The Console lead has four female connections that can be plugged directly onto
the GPIO header of the Raspberry Pi.

Most USB console cables have 3.3V logic, so its safe to use with your Pi.

The connections will be the following:

- The **red** lead should be unconnected.
  (It was used to deliver power to the Pi, but for versions > 2 won't work)
- The **black** lead to `GND` (Physical `Pin 6`)
- The **white** lead to `TXD` (`GPIO14`) on the Pi (Physical `Pin 8`)
- The **green** lead to `RXD` (`GPIO15`) on the pI (Physical `Pin 10`)

After that you can power up your Raspberry Pi.

### Test & Configure
Open a terminal and find in which serial port the Pi is connected.
Usually, the device is called `/dev/ttyUSB0`.

It is best to run:
```
sudo dmesg
```
after plugging in and looking for hints on what the device is called.

After you found it (e.g. `/dev/ttyUSB0`) check the permissions:
```
ls -l /dev/ttyUSB0
```
Usually `root` user can read/write and the group `dialout` can read/write.

Make sure your user is in the `dialout` group, with:
```
id
```

If you do not see `dialout` listed, add yourself with the command:
```
sudo usermod -a -G dialout <username>
```

After you found it (e.g. `/dev/ttyUSB0`) connect to the Pi with:
```
sudo screen /dev/ttyUSB0 115200
```
or with `minicom`:
```
sudo minicom -b 115200 -o -D /dev/ttyUSB0
```

After logging in with your user account, you should have a shell for your Pi.

### Exit Screen
In order to exit from screen press the following keys:
- <kbd>Ctrl</kbd>+<kbd>a</kbd> and then <kbd>d</kbd>

### Exit Minicom
In order to exit from minicom press the following keys:
- <kbd>Ctrl</kbd>+<kbd>a</kbd> and then <kbd>z</kbd>
- Then press exit and reset key: <kbd>x</kbd>

## Running

### Running on Raspberry Pi Zero W

#### Compile OS
Compile your OS with `make` for the correct version of the Pi you are using.

#### Format SD Card
Install on the SD Card the Raspbian OS 32bit Lite version (no GUI), in order
to have the SD properly formatted for Pi to accept your OS.

##### Raspberry Pi Imager
- Choose OS: Raspbian Pi OS (other) -> Raspbian Pi OS 32bit Lite
- Choose SD Card: Choose your SD Card
- Write

##### Terminal
- Download from [Raspberry Pi site](https://www.raspberrypi.org/software/operating-systems/#raspberry-pi-os-32-bit) the Raspbian Pi OS 32bit Lite version.
- Insert SD Card on computer
- Unmount SD Card, if it mounted by the OS

```bash
sudo umount /dev/mmcblk0p*
```
or if inserted with a USB stick:
```bash
sudo umount /dev/sdb*
```

- Write Pi OS `*.img` to SD card device:
```bash
sudo dd if=raspbian.img of=/dev/mmcblk0 bs=4M status=progress
```
or if inserted with a USB stick (find your `/dev/sd*`, or else you could write to your hard drive):
```bash
sudo dd if=raspbian.img of=/dev/sdb bs=4M status=progress
```

#### Transfer `.img` to SD Card
- Copy the generated `kernel.img` (for Raspberry Pi Zero W) file to the `/boot`
  partition of your Raspberry Pi SD card
- Delete `kernel.img` as well as any other `kernel*.img` files that may be
  present on your SD card.
- **Delete `config.txt` from `/boot`.** It is not needed for Raspberry Pi Zero W, and produces garbage to the serial console.
- Make sure you left all other files in the boot partition untouched.

### Running on Raspberry Pi 4

#### Compile OS
Compile your OS with `make` for the correct version of the Pi you are using.

#### Format SD Card
Install on the SD Card the Raspbian OS 32bit Lite version (no GUI), in order
to have the SD properly formatted for Pi to accept your OS.

#### Transfer `.img` to SD Card
- Copy the generated `kernel8.img` (for Raspberry Pi 4) file to the `/boot` partition of your Raspberry Pi SD card
- Delete `kernel8.img` as well as any other `kernel*.img` files that may be present on your SD card.
- **Delete `config.txt` from `/boot`.** (?)
- Make sure you left all other files in the boot partition untouched.


### Serial Connection with Pi
After you had a successful serial connection to the Pi with the USB to TTL cable
with the SD card with the Raspbian OS, you can try the serial connection with
the Custom OS.

- Connect the USB-to-TTL serial cable to your Pi and your Computer.
- Open your terminal emulator as described in the Serial Connection part:
```
sudo screen /dev/ttyUSB0 115200
```
- Power on your Raspberry Pi.
- You should be able to see the Hello, world! message there.

**Notes:**

- It is important to **first** to the screen session, and **then** power on the Pi.
- Sometimes the serial console, might be a little laggy, it usually recovers after a restart.
- Sometimes it helps, to disconnect the cable entirely from the computer and reconnecting it before the other steps.
  - Check `dmesg` to see errors about the usb-ttl cable.


### Running on QEMU
For running on **Raspberry Pi Zero** *(Not working)*:
```bash
qemu-system-arm -cpu arm1176 -m 256 -M versatilepb -serial stdio -kernel kernel.img
```
For running on **Raspberry Pi 2**:
```bash
qemu-system-arm -m 256 -M raspi2 -serial stdio -kernel kernel.img
```
For running on **Raspberry Pi 3**:
```bash
qemu-system-aarch64 -m 256 -M raspi3 -serial stdio -kernel kernel8.img
```
For running on other Raspberry Pi's:
```bash
qemu-system-arm -cpu cortex-a7 -m 256 -M versatilepb -serial stdio -kernel kernel.img
```
Parameters:

- `-kernel`: this is the path to our kernel
- `-serial stdio`: redirects the machine's virtual serial port (`UART0`) to our
    host's `stdio`, so that everything sent to the serial line will be displayed
    and every key typed in the terminal will be received by the VM.
- `-m`: this sets the RAM limit
- `-cpu`: this sets the CPU type to match a Raspberry Pi
- `-M`: this sets the machine we are emulating. `versatilepb` is the 'ARM Versatile/PB' machine

`-cpu`: `cortex-a7` or `arm1176`
