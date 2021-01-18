---
title: Getting Started with Embedded Systems in Visual Studio Code
date: 2019-04-29 16:10:00
tags: 
    - vscode
    - stm32
    - embedded systems
category: 
    - Tutorials
img: /images/screenshot-vscode-tutorial-1.png
---

In the [previous post](https://shineyruan.github.io/2019/03/15/vscode-tutorials/) we have known that Visual Studio Code is a lightweight editor for small coding projects. However, you might still wonder how to actually take advantage of this small but powerful editor. In this post, we will see together how to develop embedded system projects in Visual Studio Code.

<!-- more -->

# VS Code and STM32

VS Code has an open-source solution for STM32 application developments. This solution is based on an extension called [Platform IO](https://platformio.org/). This extension does no more than integrating the ARM-GCC toolchain as well as OpenOCD debugger with Python scripts for automatic compiling, uploading and debugging. It provides a variety of popular STM32 frameworks, such as Arduino, Standard Peripheral Library, STM32Cube, etc. It also supports almost all types of popular STM32 development boards in the market. With this extension, we can easy code, compile and upload codes to the MCU all in VS Code. In a word, it is a combination of VS Code's excellent coding autocomplete feature and the open-source ARM-GCC toolchain.

![vscode-stm32](/images/screenshot-vscode-tutorial-2.png)

## Adding a customized framework to Platform IO core

Although Platform IO has provided a large number of framework for a variety of boards, certain types of boards may lack support from some framework. Take my development experience for an example, I use STM32 value-line discovery (STM32VLDISCOVERY) board for project use, while the IDE does not offer a standard peripheral library framework support for it, which means that I cannot use standard peripheral library function calls in my project development. So how can I modify the core and add the support manually? Here comes the trick.

1. Install the **Platform IO IDE** from VS Code's extension page.

![vscode-stm32-marketplace](/images/screenshot-vscode-tutorial-3.png)

2. Install the **STM32 official support** from the extension home page in VS Code.

![vscode-stm32-pio](/images/screenshot-vscode-tutorial-4.png)

3. Clone the source from [my personal GitHub repository](https://github.com/shineyruan/STM32-PlatformIO-Support).
4. If Platform IO IDE is installed successfully, there should be a root folder called `.platformio` in your computer. Find it out.
5. Copy everything in folder `stm32vldiscovery-spl` from the GitHub repository you just cloned into the `.platformio` folder.
6. Done!

Some explanations for my modification:

* In `packages/framework-spl`, there are two folders:
  * `platformio/ldscripts` contains the linker script for STM32VLDISCOVERY board;
  * `stm32` contains the STM32F10x standard peripheral library from the [STMicroelectronics official website](https://www.st.com/en/embedded-software/stsw-stm32054.html).
* In `platforms/ststm32`, there are also two folders:
  * `boards` contains a JSON file for the STM32VLDISCOVERY board. In the file, `spl` framework and `stm32vldiscovery.cfg` board config file (already in the original Platform IO core) have been added, and the required defined macro `STM32F10X_MD_VL` has been added for the board;
  * `builders/frameworks` contains a Python script for the standard peripheral library framework. In the file, standard peripheral library support for STM32F10x devices has been added.

Happy coding!