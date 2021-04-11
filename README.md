# Tiva C Series CMake Example Project

**Tested in WSL2(Debian) Windows!**


## Compilation


# Create a build folder where all of the temporary files will be stored
mkdir build
cd build
cmake ..
make
make flash

# Configure the project


cmake .. -DCMAKE_BUILD_TYPE=Debug -DCMAKE_TOOLCHAIN_FILE:PATH="../cmake/arm-none-eabi-toolchain.cmake"

https://github.com/utzig/lm4tools or sudo apt-get install lm4flash

## Requirements

* TivaWare C Series SDK - [Download here](http://software-dl.ti.com/tiva-c/SW-TM4C/latest/index_FDS.html) (Use the Complete solution!).
* Flash Programmer - [Download here](http://www.ti.com/tool/LMFLASHPROGRAMMER).
* ARM version of the GCC - [Download here](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm/downloads) (Comes with C11 and C++14).
* CMake - [Download here](https://cmake.org/download/).
* Visual Studio Code - [Download here](https://code.visualstudio.com/).
* OpenOCD (optional, for debugging) - [Download here](https://gnutoolchains.com/arm-eabi/openocd/).
