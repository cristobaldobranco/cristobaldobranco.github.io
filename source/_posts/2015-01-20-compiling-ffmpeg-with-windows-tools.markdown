---
layout: post
title: "Compiling FFmpeg with Windows Tools"
date: 2015-01-20 11:34:12 +0100
comments: true
categories: [FFmpeg, Windows, Visual Studio]
---

FFmpeg is quite a powerful tool for handling video and audio files and streams. During work I came across the need to link the FFmpeg libraries statically with a Microsoft Visual Studio project. While there are current [pre-built versions] of the developer libraries for dynamic linking (i.e. DLLs), I did not come across the static libs. So I figured, I'll have to compile them myself. We required the libraries for 32bit and 64bit architectures. After experimenting a lot with MinGW and its GCC variant and trying to link the outputs to our Visual Studio project, I gave up on this method as I could not get the libraries to link with x86 and x64 configurations.

Basically this is what I did to get the libraries to work. The configure script provides a --toolchain=msvc switch which will tell make to use of the Windows compiler suite but this needs a few prerequisites:

* Git to get the latest version of FFmpeg
* MinGW to be able to process makefiles
* Windows compiler suite (as I only had Visual Studio 2008, which is too old because it does not support the compiler switch /Fi, I had to use the Windows SDK 7.1)
* C99-to-C89 wrapper to support the C99 standard
* YASM to process the assembly files of FFmpeg

Getting the Prerequisites
---

* Download [Git for Windows]
* Download [Windows SDK 7.1]
* Download [MinGW-get utitlity]
* Download [C99-to-C89 Converter & Wrapper] 
* Download [Yasm], get the 64bit version for general use

Setting Up the Environment
---

* Install it and put the bin folder into your PATH enviroment variable.
* Install Windows SDK
* Install MinGW-get utility, set PATH to point to it and in a command shell run:

```sh
mingw-get install msys
mingw-get install msys-coreutils
```

* Rename Yasm binary to yasm.exe and copy into the path of the compilers of the Windows SDK (for me this was C:\Program Files\Microsoft SDKs\Windows\v7.1\Bin and C:\Program Files\Microsoft SDKs\Windows\v7.1\Bin\x64). Be sure to place it in both the folders if you want to compile for 32bit and 64bit architectures.
* Copy c99conv.exe and c99wrap.exe from the C99-to-C89 Converter archive into the path of the compilers of the Windows SDK. Be sure to place it in both the folders if you want to compile for 32bit and 64bit architectures.

Get the FFmpeg Source
---
* Go to the drive location where you want to compile FFmpeg and in a command shell execute. This might take a little while depending on your connection, grab a coffee.

```sh
git clone git://source.ffmpeg.org/ffmpeg.git ffmpeg
```

Compiling the Libraries
---
* Open cmd.exe and change to the Windows SDK v7.1 installation folder
* Run either for x86 
```sh
SetEnv.cmd /x86 Release
```
or for x64
```sh
SetEnv.cmd /x64 Release
```
* Change to the MinGW/Msys folder and run msys.bat

These steps set up the Windows path variables to point to the correct Windows compiler (+Yasm+C99 converter that you copied there) and then start Msys (a bash shell for MinGW) with these paths

* In Msys run
```sh
which link.exe
```
If link.exe points to /bin/link.exe, your installation cannot see the Microsoft Linker. Run the following in Msys:
```sh
mv /bin/link.exe /bin/link.mingw.exe
```

* Change to the folder you checked out FFmpeg into and run for x86 compilation
```sh
./configure --toolchain=msvc --arch=x86
make clean && make
```
or 
```sh
./configure --toolchain=msvc --arch=x86_64
make clean && make
```

* This will take a longer while, go for lunch

Working with the Libraries
---
Congratulations, you are now the proud owner of your own FFmpeg tools compiled without any compatibility layers. All system accesses directly call the Windows runtime.

You will find several .a files in the subfolders of the source (e.g. libavcodec/libavcodec.a). These are the developer libraries that although they have the extension .a are compatible to (or actually the same format as) Windows .lib files. They can easily be linked with any Microsoft Visual Studio project that needs to interact with  FFmpeg. Finding the necessary include files for your application might get a little trickier but you should be able to extract a set of header files that your program needs.

[pre-built versions]: http://ffmpeg.zeranoe.com/builds/
[Git for Windows]:http://git-scm.com/downloads
[C99-to-C89 Converter & Wrapper]:https://github.com/libav/c99-to-c89/downloads
[Yasm]: http://yasm.tortall.net/Download.html
[Windows SDK 7.1]: http://www.microsoft.com/en-us/download/details.aspx?id=8279
[MinGW-get utitlity]:http://sourceforge.net/projects/mingw/files/latest/download?source=files


