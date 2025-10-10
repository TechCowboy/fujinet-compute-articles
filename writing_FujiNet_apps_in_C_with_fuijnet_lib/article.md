# Writing Cross-Platform FujiNet Code using C and fujinet-lib.

## What is fujinet-lib?

Those of us working on FujiNet have tried to make it as easy to write FujiNet applications as possible. We've done this not only by trying to integrate with each target computer's environment as closely as possible for native application development, we have tried to foster a consistent cross-platform development methodology by using proven languages like C, and the fujinet-lib library, which exposes all FujiNet functionality in a consistent manner no matter the target platform.

FujiNet-lib is a library of functions for the C programming language. It provides every function needed to access the network device, as well as to the FujiNet control device in order to easily interact with it from any C program using the same function names and parameters, no matter the target. Leveraging this library, while providing a clean seperation of system specific functions such as screen, user input, and common logic, will ensure that your program can be easily ported to all existing and future FujiNet platforms.

The FujiNet team chose C as the language we write most of our programs in, because it is reasonably fast, it is portable enough while still allowing access to system specific features, and because there are compilers available for every single retrocomputing and retrogaming platform in existence with mature runtime and system libraries. In addition, the compilers run on our modern computers with much more memory and processing power to allow us to build something to load directly on a target system with nothing more than a FujiNet device. Once the pre-requisites are installed, it's very efficient.

## Is it required to be used to write FujiNet applications?

No. It is merely a fast, efficient way to write a program that can be brought to many different platforms. This article intends to introduce the library that makes using the FujiNet from C very easy, but it's not an explicit limitation imposed by the FujiNet or its developers.

## Which platforms can be targeted using fujinet-lib?

* Atari 8-bit, using the CC65 compiler. (https://cc65.github.io/)
* Apple II, using the CC65 compiler.
* Commodore 8-bit (VIC20, C16/PLUS4, 64, 128), using the CC65 compiler.
* COLECO Adam using the Z88DK compiler. (https://z88dk.org/)
* TRS-80 Color Computer using the CMOC compiler (http://sarrazip.com/dev/cmoc.html)

**Note**: C compilers such as CC65 and Z88DK can target multiple machine types.

## What you need.

If you aren't familiar with the C language, the book "The ANSI C Programming Language (2nd Edition)" by Kernighan and Richie is still very relevant. A copy can be found on the Internet Archive, here: https://archive.org/details/the-ansi-c-programming-language-by-brian-w.-kernighan-dennis-m.-ritchie.org/mode/2up There are also plenty of great online tutorials teaching the basics of C, such as https://learn-c.org/

You will want an operating environment that can run GNU Make, a copy of 'git' to retrieve needed code from GitHub, your favorite text editor, and one or more of the compilers mentioned above. While this article will focus on installing the environment into a Linux environment, Windows users can install and use the Windows Services for Linux (Ubuntu) environment and follow along as well.

So to summarize:

* Some book on the C Language.
* A form of Linux, WSL2 works too.
* GNU Make
* git
* One or more of the compilers listed above.
* Your favorite text editor.

## Installing the foundation of the development environment.

We need to build a few tools. This will ensure that what we have is not only the newest, but that it fits our environment well, and that it can easily be updated when newer versions become available.

If you're running Windows, you will need to ensure WSL2 is installed, and that you can use sudo. If you haven't installed WSL, you can do so from PowerShell or CMD using the command 'wsl --install'

If you're running MacOS, you will at least need the XCode command line tools, which can be installed with the 'xcode-select --install' command.

The following set of instructions are intended to be executed in a Linux shell, which, even under windows can be started using WSL2. The shell prompt looks similar to:

```
$
```

