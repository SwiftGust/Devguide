# Development Environment on Linux

We have standardized on Debian / Ubuntu LTS as the supported Linux distribution, but [boutique distribution instructions](starting-installing-linux-boutique.md) are available for Cent OS and Arch Linux.

## Permission Setup

> **Warning** Never ever fix permission problems by using 'sudo'. It will create more permission problems in the process and require a system reinstallation to fix them.

The user needs to be part of the group "dialout":


```sh
sudo usermod -a -G dialout $USER
```

And then you have to logout and login again, as this is only changed after a new login.

## Installation

Update the package list and install the following dependencies for all PX4 build targets. PX4 supports four main families:

  * NuttX based hardware: [Pixhawk](hardware-pixhawk.md), [Pixfalcon](hardware-pixfalcon.md)
  * Snapdragon Flight hardware: [Snapdragon](hardware-snapdragon.md)
  * Raspberry Pi hardware: [Raspberry Pi 2](hardware-pi2.md)
  * Host simulation: [jMAVSim SITL](simulation-sitl.md) and [Gazebo SITL](simulation-gazebo.md)

> **Info** Install the [Ninja Build System](http://dev.px4.io/starting-installing-linux-boutique.html#ninja-build-system) for faster build times than with Make. It will be automatically selected if installed.

```sh
sudo add-apt-repository ppa:george-edison55/cmake-3.x -y
# Currently getting java in Ubuntu does not work. Add this repository
sudo add-apt-repository ppa:openjdk-r/ppa
sudo apt-get update
sudo apt-get install python-argparse git-core wget zip \
    python-empy qtcreator cmake build-essential genromfs -y
# simulation tools
sudo apt-get install ant protobuf-compiler libeigen3-dev libopencv-dev openjdk-8-jdk openjdk-8-jre clang-3.5 lldb-3.5 -y
```

### NuttX based hardware

Ubuntu comes with a serial modem manager which interferes heavily with any robotics related use of a serial port (or USB serial). It can deinstalled without side effects:


```sh
sudo apt-get remove modemmanager
```

Update the package list and install the following dependencies. Packages with specified versions should be installed with this particular package version.


```sh
sudo add-apt-repository ppa:terry.guo/gcc-arm-embedded -y
sudo apt-get update
sudo apt-get install python-serial openocd \
    flex bison libncurses5-dev autoconf texinfo build-essential \
    libftdi-dev libtool zlib1g-dev \
    python-empy gcc-arm-none-eabi -y
```

If the resulting `gcc-arm-none-eabi` version produces build errors for PX4/Firmware master, please refer to [the bare metal installation instructions](http://dev.px4.io/starting-installing-linux-boutique.html#toolchain-installation) to install version 4.8 manually.

### Snapdragon Flight
Snapdragon Flight Kit uses Linux Linaro OS which is for ARM Architecture Cores.
To compile PX4 for Snapdragon Flight Kit, Also requires bunch of packages to cross-compile it's ARM core.

#### Toolchain installation


```sh
sudo apt-get install android-tools-adb android-tools-fastboot fakechroot fakeroot unzip xz-utils wget python python-empy -y
```


```sh
git clone https://github.com/ATLFlight/cross_toolchain.git
```

Get the Hexagon SDK 3.0 from QDN: https://developer.qualcomm.com/download/hexagon/hexagon-sdk-v3-linux.bin

This will require a QDN(Qualcomm Developer Network) login. You will have to register if you do not already have an account.

Now move the following files in the download folder of the cross toolchain as follows:


```sh
mv ~/Downloads/hexagon-sdk-v3-linux.bin cross_toolchain/downloads
```
Install the toolchain and SDK like this:


```sh
cd cross_toolchain
./installv3.sh
cd ..
```

Follow the instructions to set up the development environment. If you accept all the install defaults you can at any time re-run the following to get the environment setup. It will only install missing components.

After this the tools and SDK will have been installed to "$HOME/Qualcomm/...". Append the following to your ~/.bashrc:


```sh
export HEXAGON_SDK_ROOT="${HOME}/Qualcomm/Hexagon_SDK/3.0"
export HEXAGON_TOOLS_ROOT="${HOME}/Qualcomm/HEXAGON_Tools/7.2.12/Tools"
export PATH="${HEXAGON_SDK_ROOT}/gcc-linaro-4.9-2014.11-x86_64_arm-linux-gnueabihf_linux/bin:$PATH"
```

Load the new configuration:


```sh
source . ~/.bashrc
```

#### Sysroot Installation

A sysroot is required to provide the libraries and header files needed to cross compile applications for the Snapdragon Flight applications processor.

There are 2 sysroot options. The recommended is QRLinux because it provides access to all the proprietary libraries to access camera(Snapdragon has built in 4K front camera and downfacing optical flow cammera), etc on Snapdragon Flight.
(To download QRLinux, Requires support account of Intrisyc which asks Snapdragon Flight Kit's product serial)

If you do not have an Intrinsyc account, you can use stock Ubuntu Trusty (14.04) sysroot to build PX4 , but will not have access to the camera headers and libs.

#### QRLinux sysroot
Login to the Intrisyc support page and download QRLinux for Snapdragon
http://support.intrinsyc.com/attachments/download/690/Flight_3.1.1_qrlSDK.zip

copy/move the file to the ./download directory
```
cp ~/Downloads/Flight_qrlSDK.zip ./downloads
./qrlinux_sysroot.sh --clean

```
This will install:

ARMv7hf QRLinux sysroot [HEXAGON_ARM_SYSROOT]: ${HOME}/Qualcomm/qrlinux_v1.0_sysroot

#### Stock Ubuntu Trusty (14.04) sysroot

Create the Ubuntu Trusty (14.04) sysroot
```
export HEXAGON_SDK_ROOT=${HOME}/Qualcomm/Hexagon_SDK/3.0
./trusty_sysroot.sh
```

This will install:

ARMv7hf Ubuntu Trusty (14.04) sysroot [HEXAGON_ARM_SYSROOT]: ${HOME}/Qualcomm/ubuntu_14.04_armv7_sysroot
Append the following to your ~/.bashrc:

```
export HEXAGON_ARM_SYSROOT=${HOME}/Qualcomm/ubuntu_14.04_armv7_sysroot
```

Load the new configuration:


```sh
source ~/.bashrc
```

For more sysroot options see [Sysroot Installation](https://github.com/ATLFlight/cross_toolchain/blob/sdk3/README.md#sysroot-installation)

#### Update ADSP firmware
Before building, flashing and running code, you'll need to update the [ADSP firmware](advanced-snapdragon.html#updating-the-adsp-firmware).

#### References

There is a an external set of documentation for Snapdragon Flight toolchain and SW setup and verification:
[ATLFlightDocs](https://github.com/ATLFlight/ATLFlightDocs/blob/master/README.md)
And If you have Snapdragon Flight Kit, Refer these documents of
Basic Guides for User, Developers and How to use cameras, etc : [Intrinsyc Documents](http://support.intrinsyc.com/projects/snapdragon-flight/documents)

Messages from the DSP can be viewed using mini-dm.


```sh
$HOME/Qualcomm/Hexagon_SDK/3.0/tools/debug/mini-dm/Linux_Debug/mini-dm
```

### Raspberry Pi hardware
Developers working on Raspberry Pi hardware should download the RPi Linux toolchain from below. The installation script will automatically install the cross-compiler toolchain. If you are looking for the *native* Raspberry Pi toolchain to compile directly on the Pi, see [here](http://dev.px4.io/hardware-pi2.html#native-builds-optional)


```sh
git clone https://github.com/pixhawk/rpi_toolchain.git
cd rpi_toolchain
./install_cross.sh
```
You will be required to enter your password for toolchain installation to complete successfully.

You can pass a different path to the installer script if you wouldn't like to install the toolchain to the default location of ```/opt/rpi_toolchain```. Run ``` ./install_cross.sh <PATH>```. The installer will automatically configure required environment variables as well.

### Parrot Bebop
Developers working with the Parrot Bebop should install the RPi Linux Toolchain. Follow the
description under [Raspberry Pi hardware](raspberry-pi-hardware).

Next, install ADB.


``sh
sudo apt-get install android-tools-adb -y`
``

## Finishing Up

Now continue to run the [first build](starting-building.md)!
