---
layout: post
title: "VirtualBricks in WSL2"
author: "Lorenzo Felletti"
---
# Virtualbricks in WSL2

| Version | Updated | Author | Contact |
|---|---|---|---|
| 2.0.0 | 2022-10-26 | Lorenzo Felletti | lorenzo.felletti2@unibo.it |

This is a guide to install *Virtualbricks* in *WSL2*. Virtualbricks is a frontend for the management of *Qemu* Virtual Machines (VMs) and *VDE* virtualized network devices (switches, channel emulators, etc.) used in the course of Infrastructures and Architectures for Space Networks at the University of Bologna.

In this guide I will assume that you already know what *WSL* (Windows Subsystem for Linux) is and that you have it already installed on your Windows machine.

The following links are a useful starting point if you haven't heard of or haven't installed WSL:
 - WSL | Wikipedia https://en.wikipedia.org/wiki/Windows_Subsystem_for_Linux
 - Install WSL | Microsoft Docs - https://docs.microsoft.com/en-us/windows/wsl/install.

I also strongly recccomend that you use the *Windows Terminal* as your terminal emulator. It is a very powerful terminal emulator that allows you to use multiple shells (cmd, powershell, wsl, etc.) in the same window. It is also very customizable and it is open source. You can find it on the *Microsoft Store*.

## Prerequisites
- You must be on Windows 11 Build 22000 or later (because we will use GUI apps in WSL2).
  - You can check your build number by running `winver` command in a *cmd* shell
- Ubuntu 20.04 LTS installed on WSL2

Update to the latest version of WSL2 by running in a *PowerShell* shell:
```PowerShell
wsl --update
wsl --set-default-version 2
wsl --shutdown
```

User prerequisites:
- knowing the basic principles of the shell. If you don't know them, you can find a good introduction [here](https://ubuntu.com/tutorials/command-line-for-beginners#1-overview)
- knowing how to edit a file from within the terminal (I suggest you starti with `nano`[<sup>1</sup>](#introduction-to-nano) if you're not practical with any)
- (useful) knowing how to pass files from Windows to WSL2 and viceversa[<sup>2</sup>](#passing-files-from-windows-to-wsl2-and-viceversa).

## Install Virtualbricks
> For the updated official guide, please refer to the professor slides on [virtuale](https://virtuale.unibo.it/).

To install Virtualbricks in Ubuntu 20.04 LTS, follow this steps:
1. Add following lines to your `/etc/apt/sources.list`:
    ```Bash
    # DTN Repository
    deb http://cnrl.deis.unibo.it/repo/ focal main
    deb-src http://cnrl.deis.unibo.it/repo/ focal main
    ```
2. Download and import the GPG Public Key for the repository, run the following comand:
   ```Bash
   wget -O - http://cnrl.deis.unibo.it/repo/signing_key.pub | sudo apt-key add -
   ```
3. Update the package list and install Virtualbricks:
   ```Bash
    sudo apt update
    sudo apt install virtualbricks
    ```

## Creating a Control Network
> The following steps are an adaptation of the steps from [here](http://cnrl.deis.unibo.it/control_network_en.php).
- Download this file: [control_network.sh](https://raw.githubusercontent.com/lorenzofelletti/lorenzofelletti.github.io/master/assets/control_network.sh)
- decompress the file using `tar jxvf vde_switch-init.tar.bz2`


## Importing a VB Project
> For the updated official guide, please refer to the professor slides on [virtuale](https://virtuale.unibo.it/).

- A VB project usually consists of a Linux image, common to all Virtual Machines, and a VB file.
  - Select and download the wanted project (e.g. the latest version of DTN2hops) from [here](http://cnrl.deis.unibo.it/VB_projects.php)
  - *Tip: save it in a new directory called `VB_projects`*
- Download the image required by the selected project (e.g. Debian 10) from here (the image is larger than 1GB) from [here](http://cnrl.deis.unibo.it/VB_images.php)
  - *Tip: save it in a new directory called `VB_images`*
- Decompress the image (but keep the compressed file for backup) with `tar -xvzf filename.tar.gz`
- Import the project in VB
  - In VB, select the *file* tab, and then the *import* command (check that you want to open the project)
  - You will be asked first to save (an optional) embedded image which is not present in our testbeds; skip this step by just pressing *Enter*
  - Then you must associate the name of the image used in the project with the full path of the (decompressed) image you have saved locally

## Troubleshooting


## Other useful resources
### Introduction to Nano
Nano is a simple text editor that is installed by default in Ubuntu. To open a file with nano, run `nano <filename>`. To save the file, press `Ctrl+O` and then `Enter`. To exit, press `Ctrl+X`.

A more detailed guide can be found [here](https://www.howtogeek.com/howto/42980/the-beginners-guide-to-nano-the-linux-command-line-text-editor/).

### Passing Files from Windows to WSL2 and Viceversa
The easiest way to pass files from Windows to WSL2 is to change directory to the target folder on the distro terminal and then run `explorer.exe .` to open the folder in Windows Explorer.<br/>
You can then drag and drop files from Windows to the distro terminal.
