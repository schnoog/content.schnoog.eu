
# Ender V3 SE Klipper Installation

This is an easy step-by-step tutorial explaining how to install Klipper on a raspberry pi to control an Ender V3 SE 3D printer

## Preconditions
* You have a raspberry pi (or any other linux machine with an up to date distribution)
* You are able to log into your PI with SSH
* You have an SD Card (FAT32, 4096 assoc)
* And yes, your raspberry needs to be able to pull content from the internet

## Some remarks for later points  
In frame of the installation you will use KIAUH  
**KIAUH expects you to enter the number / letter and confirm it with enter**  
You also will use `make menuconfig`   
**You can control it with your cursor keys and space bar**


## Let's rock

SSH into my Raspberry Pi as user pi

Command | Comment
--- | ---
`sudo apt-get update && sudo apt-get install git -y` | Install git
`cd ~ && git clone https://github.com/dw-0/kiauh.git` | Installation of KIAUH 
`echo "https://github.com/Klipper3d/klipper" > ~/kiauh/klipper_repos.txt`  | Adding of jpcurtis' repository to KIAUH
`echo "https://github.com/jpcurti/ender3-v3-se-klipper-with-display" >> ~/kiauh/klipper_repos.txt` | 
`cd ~ && ./kiauh/kiauh.sh` | Run KIAUH

### In KIAUH

Change the repository

> **6** (Settings)  
**1** (Set custom Klipper repository)  
**1** (jpcurti/ender3-v3-se-klipper-with-display)  
**B** (Back)  
**B** (Back)  

Install Klipper *(for the installation process just follow the on screen steps shown by KIAUH)*	

> **1** (Install - you may need to enter the password for the current use)  
**1** (Klipper)  
**1** ([Python 3.x]  (recommended))  
**1** (Number of Klipper instances to set up)  
Now Klipper will be installed. That may take a minute or two  
**2** (Moonraker)  
**Y** (Install Moonraker? (Y/n))  
**3** or **4** (Mainsail or Fluidd, whatever you prefer)  
**B** (Back)  
**Q** (Quit)

Configurating the new printer firmware	

Command | Comment
--- | ---
``` cd ~/klipper  && make menuconfig ``` | Start the confgurator

Now make the following settings:

Setting | Value
--- | --- 
Micro-controller Architecture | **STMicroelectronics STM32**
Processor model | **STM32F103**
Bootloader offset | **28KiB bootloader**
Communication interface | **Serial (on USART1 PA10/PA9)**
Enable extra low-level configuration options | **activated**
Enable serial bridge | **activated** 
USART2 | **activated**

And leave the configurator
> **Q** (Quit (prompts for save))  
**Y** (Yes)

Compiling and installing the new printer firmware	

Command  | Comment
--- | ---
``` make ``` | That actually compiles the firmware

The compiled firmware is saved as **~/klipper/out/klipper.bin**

**Copy this bin file to an empty (FAT32, 4096 assoc) SD card and give it a random short file name (f.e. abc.bin abe.bin and so on), and plug it into the powered off printer. Switch the printer on and wait for two minutes.** 

Configuration

Command | Comment
--- | ---
``` cd ~/printer_data/config ``` | Change into the config directory
``` wget "https://raw.githubusercontent.com/0xD34D/ender3-v3-se-klipper-config/main/prtouch.cfg" ``` | Download the prtouch config to start with
``` curl "https://raw.githubusercontent.com/0xD34D/ender3-v3-se-klipper-config/main/printer-creality-ender3-v3-se-2023.cfg" > printer.cfg ``` | Download the template config
``` echo '[include prtouch.cfg]' >> printer.cfg ``` | Include the prtouch config
``` echo '[e3v3se_display]' >> printer.cfg ```  | Tell klipper that there's a display
``` echo 'language: english' >> printer.cfg ``` | And which language it should use

Open `moonraker.conf` in your favorite editor and ensure that klippy_uds_address in set correctly

Setting | Value
--- | ---
klippy_uds_address | `/home/pi/printer_data/comms/klippy.sock` 
   
or  if your username isn't pi 

Setting | Value
--- | ---
klippy_uds_address | `~/printer_data/comms/klippy.sock`

Installing the GUI 
 
Command | Comment 
--- | --- 
`cd ~ && ./kiauh/kiauh.sh` | Run KIAUH

> **1** for install and select your preferred option. either  
**3** for Mainsail or  
**4** for Fluidd

Please be aware that different to what most tutorials state, the serial console is located at 

`~/printer_data/comms/klippy.serial`

## Acknowledgements
Many thanks to 
* [Clark Scheffs' (0xD34D)](https://github.com/0xD34D)  for bringing the V3 SE Bed Sensor to life
* [João Pedro Curti (jpcurti)](https://github.com/jpcurti)  for making the display work 


*If you have any question, hints or other useful comments in regards to this "tutorial" please feel free to reach out for me at*

[Bluesky](https://bsky.app/profile/schnoog.eu),  [Reddit](https://reddit.com/user/Horror_Equipment_197/), [Github](https://github.com/schnoog/) or [Discord](https://discord.com/users/287545325118291968/)

