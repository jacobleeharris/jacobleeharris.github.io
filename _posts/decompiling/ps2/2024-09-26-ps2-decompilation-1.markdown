---
layout: post
title: PS2 Decompilation part 1
date: 2024-09-26 15:51:00 -0600
categories: introduction devlog decompilation
---
Welcome!

In this series of tutorials, we will be going over how to reverse engineer and decompile a PS2 binary back to its equivalent
C or C++ code in a series of high-level and in-depth tutorials.

This first tutorial will go over setting up a PS2 development toolchain to aid us in developing executables for the PS2
and reverse engineering them with a set of tools that are freely available.

# Setting up the PS2 toolchain
Note: A C/C++ toolchain must already be installed to continue with the following instructions.

### Step 1: Install prerequisites

The prerequisites for compiling the PS2 toolchain are:
`gcc`, `make`, `cmake`, `patch`, `git`, `texinfo`, `flex`, `bison`, `gettext`, `wget`, `gsl`, `gmp`, `zlib`, `mpfr`, `mpc`

The README.MD provides instructions on how to install these packages for a variety of distributions.

### Step 2: Setup environment variables

Copy the following code into `~/.bashrc`:
```bash
export PS2DEV=/usr/local/ps2dev
export PS2SDK=$PS2DEV/ps2sdk
export GSKIT=$PS2DEV/gsKit
export PATH=$PATH:$PS2DEV/bin:$PS2DEV/ee/bin:$PS2DEV/iop/bin:$PS2DEV/dvp/bin:$PS2SDK/bin
```

After modifying your login script, make sure to `source` the login script to ensure these
environment variables are set. Alternatively, log out and log back in.

```bash
source ~/.bashrc
```

Now create the standard directories (to which the toolchain will be installed into):
```bash
sudo mkdir -p $PS2DEV
sudo chown -R $USER: $PS2DEV
```
The space between these arguments is intended and will not change the ownership of the directory without the space in
between `$USER` and `$PS2DEV`.


### Step 3: Clone the toolchain

The toolchain is hosted on GitHub at the [ps2dev](https://github.com/ps2dev/ps2dev) repo.

Clone with the following command:
```bash
git clone https://github.com/ps2dev/ps2dev.git
```
This command will clone the `ps2dev` repository into a folder named `ps2dev` in the current directory.

If you clone with SSH, the toolchain may fail to build as it can require a password to clone repos over SSH
and input will not be allowed during build.

Alternatively, you can use the Docker image provided on [Docker Hub](https://hub.docker.com/r/ps2dev/ps2dev).
Using the Docker image allows you to build with the same exact OS, dependencies, and requirements on any OS
without the need to manually configure build steps each time.

On Linux-like systems, you can pull the Docker image with the following command:
```bash
sudo docker pull ps2dev/ps2dev
```

On Windows, you can download [Docker Desktop](https://www.docker.com/products/docker-desktop/) and issue the above command
without the `sudo` portion in Command Prompt / PowerShell.

After which, the Docker image will be cloned and built which will allow you to have a PS2DEV environment
on any system with minimal steps.

### Step 4: Compile the toolchain

Navigate to the directory in which you cloned the `ps2dev` repo and open a terminal there.

Execute the build-all shell script with the command `./build-all.sh`

This may take a while! If the build fails, you can retry the build by re-executing the above command.

# Compiling a test program

Create a new folder named `test` in a destination of your choosing.
This folder will be the project we are using to test our toolchain and ensure it is working as expected.

Inside the folder, create a file named `Makefile` and a `main.c` source file which will hold the source code
for our program.

Copy the following code to the `Makefile` with a text editor:
```makefile
TOOLCHAIN = mips64r5900el-ps2-elf
CC        = $(TOOLCHAIN)-gcc
CXX       = $(TOOLCHAIN)-g++

all:      main.elf

main.elf: main.c
    $(CC) main.c -o main.elf

```

In the `main.c` file, add the following code:
```c
#include <stdio.h>

int main(int argc, char** argv) {
    puts("Hello, World!");
    
    // Infinite loop
    // This is to allow the user to monitor the output
    // in the serial console of an emulator.
    for (;;) {

    }
}
```

If you used the Docker container from before, you will need to start the Docker instance which can be done with the following command:
```bash
sudo docker run -it -w /test -v $(pwd):/test ps2dev/ps2dev sh
```
This will open the Docker image with the current working folder mounted at `/test` in the Docker image which allows you to share code
written between the host and the container.

Once inside the Docker container, you will need to add some dependencies to be able to build C/C++ code:
```bash
apk add make mpfr mpc mpc1
```
which is enough to install the proper dependencies.

Afterwards, running `file` on `main.elf` should produce the following results if you used the correct toolchain:
```bash
main.elf: ELF 32-bit LSB executable, MIPS, N32 MIPS-III version 1 (SYSV), statically linked, with debug_info, not stripped
```

# Installing Ghidra

You can download the latest version of Ghidra from [GitHub releases](https://github.com/NationalSecurityAgency/ghidra/releases).
After downloading the ZIP file, you can extract it in the destination of choice. I prefer using `/opt/` for installations
such as this.

After downloading Ghidra and setting it up with the proper Java version (Java 17 is required as of writing), you can download and install the
[ghidra-emotionengine-reloaded](https://github.com/chaoticgd/ghidra-emotionengine-reloaded) plugin which allows one to load PS2 binaries into
Ghidra, demangle symbols for you, fix relocation issues, and other utilities that will benefit in reverse engineering PS2 games.

After opening Ghidra, you can install the Ghidra plugin by clicking on File -> Install Extensions -> Add Extension and selecting the extension
in the file locator window that opens up.

Here are some pictures to show exactly that:
![Ghidra Install Extensions](/assets/images/decompiling/ps2/part_1/GhidraInstallExtensions.png)

![Ghidra Select Extension](/assets/images/decompiling/ps2/part_1/GhidraSelectExtension.png)

After following the above steps, you will have the Ghidra Emotion Engine Reloaded plugin installed and ready to go alongside Ghidra
and the PS2 toolchain.

In the next tutorial, we will finally begin reverse engineering our test program with Ghidra and identifying commonalities between PS2
executables and how they work in practice.