# Virtualbricks in WSL2

| Version | Updated | Author | Contact |
|---|---|---|---|
| 2.0.2 | 2023-02-21 | Lorenzo Felletti | lorenzo.felletti2@unibo.it |

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
> The following steps are an adaptation of the steps described [here](http://cnrl.deis.unibo.it/control_network_en.php).
The **vde_switch** is just a *virtual switch*. It is one of the tools provided in the *VDE* package. In Debian and Ubuntu, the packet vde2 is already installed or it can be easily installed from official repositories. As a last resort, if in your
ditribution the packet is not included, it can be installed from the source code, by following the guide provided here
(https://github.com/virtualsquare/vde-2).

- Download this file: [control_network.sh](https://raw.githubusercontent.com/lorenzofelletti/lorenzofelletti.github.io/master/assets/control_network.sh)
- Decompress the file using `tar jxvf vde_switch-init.tar.bz2`
- Copy the file in a directory inside your home (e.g. `/home/USERNAME/mytdnlab`)
  - `cp vde_switch /home/USERNAME/mydtnlab`
  - *Note:* it is important that the directory is inside `/home/USERNAME/` (or at least `/home/`, if you really need it), otherwise the procedure does *not* work (for some reason)
- Assign the *execution* right to the file by running `sudo chmod +x /home/USERNAME/mydtnlab/vde_switch`
- Run `sudo adduser USERNAME kvm`.

### Vde_switch Execution Automation
At this point you have three options to handle the `vde_switch` execution:
1. If your WSL installation supports `systemd` or `init.d`, you can automate the task directly in Ubuntu
2. You can skip the task automation, and execute (*only once*) at every WSL reboot the following steps
    1. `cd /home/USERNAME/mydtnlab`
    2. `sudo ./vde_switch start`
3. If `systemd` or `init.d` are not supported, but you still want to automate the script execution, you can follow the same steps as for the `init.d` automation, and then the additional steps to create a Windows Task to handle the rest of the automation Windows-side.

> After you chose and completed one of the three options, check the network configuration by running `sudo ifconfig`

#### Steps if Init.d is Supported
1. `cd /etc/cron.d/`
2. Create a new file named vde_for_dtn (or whatever)
    - `sudo nano -c vde_for_dtn`
3. Write the following lines
    ```Text
    SHELL=/bin/sh
    HOME=/home/USERNAME/mydtnlab
    PATH=/home/USERNAME/mydtnlab
    @reboot root cd /home/USERNAME/mydtnlab && ./vde_switch start
    ```
4. Save and close
5. Run `sudo update-rc.d vde_switch defaults`
6. Run `sudo reboot now`.

#### (Additional) Steps to Create a Windows Task
> These steps should be followed **if and only if** you chose to follow the third option [here](#vde_switch-execution-automation). Do **not** follow them otherwise.
At this point, we have created the `cron` job, but `cron` is not started automatically on WSL start if `init.d` or `systemd` are not supported by your WSL installation.
To automate `cron` start, we have to create a Task in Windows, using the **Task Scheduler** (*Utilità di Pianificazione* in Italian)
1. Within Windows, go to the search bar and search ***Task Scheduler***
2. Run ***Task Scheduler*** as administrator
3. Click ***Task Scheduler Library*** on the left and then ***Create Task...*** on the right to create a new task
4. Use the following parameters to configure the task
    1. Under the ***General*** tab
        - ***Name*** the task whatever you want, like *WSL service cron start*
        - Choose the option ***Run whether user is logged or not***
        - Mark ***Do not store password*** and ***Run with highest privileges***
    2. Under the ***Triggers*** tab (*Attivazione* in Italian)
        - Click ***New...*** to add a new trigger for the task
        - In the ***Begin the task*** dropdown select ***At startup***
        - (*Recommended*) Within the ***Advanced settings*** check ***Delay task for 1 minute***
    3. Under the ***Actions*** tab
        - Click ***New...*** to add a new action for the task
        - Pick ***Start a program*** for the action type and then enter ***C:\Windows\System32\wsl.exe*** as the program to run
        - Set ***Add arguments (optional)*** to ***-u root service cron start***
    4. (*For laptops only*) Click ***Conditions***
        - Uncheck ***Start the task only if the computer is on AC power*** (*Avvia l'attività solo se il computer è alimentato da rete elettrica* in Italian)
5. Reboot Windows
6. Check whether `cron` is running
    - service cron status
7. Check whether `tap0` appears running
    - `ip a`
8. Run Virtualbricks
    - `sudo virtualbricks`
9. Enjoy!

## Importing a VB Project
> For the updated official guide, please refer to the professor slides on [virtuale](https://virtuale.unibo.it/).

- A VB project usually consists of a Linux image, common to all Virtual Machines, and a VB file.
  - Select and download the wanted project (e.g. the latest version of DTN2hops) from [here](http://cnrl.deis.unibo.it/VB_projects.php)
  - *Tip: save it in a new directory called `VB_projects`*
- Download the image required by the selected project (e.g. Debian 10) from here (the image is larger than 1GB) from [here](http://cnrl.deis.unibo.it/VB_images.php)
  - *Tip: save it in a new directory called `VB_images`*
- Decompress the image (but keep the compressed file for backup) with `sudo tar -xvzf filename.tar.gz`
- Import the project in VB
  - In VB, select the *file* tab, and then the *import* command (check that you want to open the project)
  - You will be asked first to save (an optional) embedded image which is not present in our testbeds; skip this step by just pressing `Enter`
  - Then you must associate the name of the image used in the project with the full path of the (decompressed) image you have saved locally

## Troubleshooting
### Virtualbricks GTK 3.0 Error
```Bash
sudo apt-get update
sudo apt-get install libgtk-3-dev
```

### Windows Task Not Starting (Missing tap0 Network Interface)
Sometimes, usually if you run WSL too fast after boot, the windows task doens't start, and so does not `cron`.
To avoid this problem, try to wait some seconds before starting WSL after a reboot.

To fix this if the problem happens:
1. Open WSL
2. Run `sudo service cron start`
3. Check if `tap0` is added by running `ip a`.

### Virtual switches not starting
Try with `sudo apt-get install vde-netemu`.

### Gdk-Message: Error reading events from display or Broken pipe
This error appears in a seemingly random manner.
To solve it when it happens:
1. open a powershell
2. run `wsl --shutdown`
3. then
    1. if you have configured `cron`
        1. `wsl -u root service cron start`
        2. open a new Ubuntu shell
    2. if you have *not* configured cron
        1. open a new Ubuntu shell
        2. `cd /home/USERNAME/mydtnlab`
        3. `sudo ./vde_switch start`

### Fixing the theme missing warning
You could see some warning about the missing of the theme icons.
If the missing icons are that of the yaru theme, then it's quite easy to fix this warnings:
```Bash
sudo apt install yaru-theme-icon
```
If the missing icons are relative to other themes, probably you can find a solution with some googling.

## Other useful resources
### Introduction to Nano
Nano is a simple text editor that is installed by default in Ubuntu. To open a file with nano, run `nano <filename>`. To save the file, press `Ctrl+O` and then `Enter`. To exit, press `Ctrl+X`.

Sometimes, you will need sudo privileges to edit some files.
If, by error you edited a file that needed sudo to be edited and now you don't know how to exit from nano, just press `Ctrl+X`, then type `N` when asked whether you want to save or not, and finally press `Enter` to exit *without* saving.

A more detailed guide can be found [here](https://www.howtogeek.com/howto/42980/the-beginners-guide-to-nano-the-linux-command-line-text-editor/).

### Passing Files from Windows to WSL2 and Viceversa
The easiest way to pass files from Windows to WSL2 is to change directory to the target folder on the distro terminal and then run `explorer.exe .` to open the folder in Windows Explorer.<br/>
You can then drag and drop files from Windows to the distro terminal.
