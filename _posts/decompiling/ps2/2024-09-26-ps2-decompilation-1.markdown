---
layout: post
title: PS2 Decompilation Pt. 1
date: 2024-09-26 15:51:00 -0600
categories: introduction devlog decompilation
---
Welcome!

In this series of tutorials, we will be going over how to reverse engineer and decompile a PS2 binary back to its equivalent
C or C++ code in a series of high-level and in-depth tutorials.

This first tutorial will go over how to source the PS2 executable
from an ISO rip and how to identify the compiler used in a PS2 binary.

A list of PS2 executables with debug symbols not stripped (present in .STABS section) can be found
[here](https://www.retroreversing.com/ps2-unstripped/).

## Ripping PS2 executable
Most RIPs of PS2 Games come in the ISO format, though other formats such as CDVD, IMG, CSO, ZSO, and GZ images exist.

This tutorial will only consider the ISO format though other formats can still be used.

You can mount the ISO file as a drive without any special tools. The SYSTEM.CNF file tells the PS2 what executable to execute when loading the disk. An example SYSTEM.CNF file can be found below

```ini
BOOT2 = cdrom0:\SLUS00000.123;1
VER = 1.00
VMODE = NTSC # can also be PAL
```
As you can see, the executable above is called SLUS00000.123. Some PS2 executables do not come with the `ELF` extension but are still valid `ELF` files.

Running `file` on the executable should output the following:
```
SLUS00000.123: ELF 32-bit LSB executable, MIPS, MIPS-III version 1 (SYSV), statically linked, not stripped
```

Copy this executable to a working folder, the assets and the like will not be necessary for this portion of the decompilation stage.

## Identify the compiler
Most PS2 binaries are compiled with Sony's proprietary GCC fork or with Metrowerks CodeWarrior (MWCC).

The easy method of identifying which compiler is used is to simply use
[Detect-It-Easy](https://github.com/horsicq/DIE-engine/releases) (DIE) which is an open source file identifier which has signatures for detecting OS/compiler/packers from an executable.

Dropping the executable into DIE should yield a similar result for GCC-compiled binaries:
![Detect-It-Easy with GCC 2.x](/assets/images/decompiling/ps2/part_1/Detect-It-Easy-GCC-2.png)

An alternative method is to pipe the output of `strings` from the executable into `grep` with `"gcc"` as the keyword which can be done with the following command:
`strings SLUS_00000.123 | grep "gcc"`

![Strings command output](/assets/images/decompiling/ps2/part_1/PS2-Strings-GCC.png)

For MWCC-compiled binaries, the process unfortunately isn't as straight-forward as dropping the executable into DIE.

DIE will still return GCC as the compiler used even if MWCC was used, so the above output may not be correct for MWCC binaries.

If the `.comment` section was not stripped from the binary, you can simply dump the `.comment` section and check for the string: `MW CodeWarrior` which can be found at the beginning of the `.comment` section 

## Installing Ghidra

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

# Foreword
In this tutorial, you learned how to source executables from an ISO rip, identify what compiler was used to build the executable, and install Ghidra with the Ghidra Emotion Engine Reloaded plugin, and are ready to start looking at a binary in Ghidra. In the next few tutorials, we will be setting up a matching decompilation using Splat

And that is all for this tutorial! Hope to see you in the next.