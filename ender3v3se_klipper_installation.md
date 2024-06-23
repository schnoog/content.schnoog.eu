

## Update 23.06.2024
Original printer display is now working, thanks to [João Pedro Curti (jpcurti)](https://github.com/jpcurti) who forked [Clark Scheffs' (0xD34D)](https://github.com/0xD34D) Klipper branch and edited the display in.

#### If you have already installed Klipper from 0xD34Ds repo, please use the following instructions to update to jpcuris version and make your printer screen working:

[Update guide for already klippered V3 SE](https://schnoog.eu/hobbies/3dprinting/ender-3-v3-se-update-to-make-the-display-working "Update guide for already klippered V3 SE")

#### If you are new to the  Klipper Ende v3 SE story, just proceed.

*If you have any question, hints or other useful comments in regards to this "tutorial" please feel free to reach out for me at*

[Bluesky](https://bsky.app/profile/schnoog.eu),  [Reddit](https://reddit.com/user/Horror_Equipment_197/), [Github](https://github.com/schnoog/) or [Discord](https://discord.com/users/287545325118291968/)

## Preface
The day before Christmas

It was a week before Christmas 2023 when I received the note that I won an Ender-V3 SE. After some years of experience with the first version of the Ender-3 I was looking forward to have a 3D printer which comes with all the extras I mounted on my initial Ender-3 from summer 2020. Bed level sensor, silent stepper drivers, yes, even a Z-offset sensor. 

So I was really excited when I held the shipping package in my hands the day before Christmas.

After screwing everything together and doing the bed leveling I was able to print a Benchy only 20 minutes after I started to unpack the printer. What an experience. No piece of paper, no wild turning on the bed screws. It just worked out of the box.

But as soon as I started to print functional things, I also started to miss some things Marlin, the GPL licensed firmware for the printer, offers. So why not simple compile an own version of the firmware with the missing options (mainly pressure advance)  and install it on the printer? I mean I did that with the original Ender-3 again and again (initial version was the 1.x, Marlin just released the 2.x version at that time).

But unfortunately Creality has chosen to violate the GPL license and don't release their modifications to Marlin or the configurations used to compile it. 

(I have an email in which Creality states that the machine "doesn't support open source", it seems somebody in China really struggles to understand the GPL and the fact that the end user has the right to obtain a copy of the source code used to compile the firmware).

So compiling Marlin was not an option. But what could I do to have PA enabled on my printer?

And at that point I stumbled over a Discord discussion about Klipper on the Ender-3 V3 SE.

So I decided that I will use Klipper.

## Get ready to rumble
OK, decision made, but where to start?

I know Klipper requires a computer to run of. Since I use Octoprint for years, that requirement is easily met by my Raspberry PI 3B+ which is already connected to the printer.

It took not long until I noticed that the user 0xD34D ([0xD34D Github account](https://github.com/0xD34D/)) forked the Klipper firmware repository and made the changes required to get the V3 SE running. Including the bed leveling sensor (called PRTouch) as well as the Z-offset sensor (which is a load cell connected to an HX711 ADC).

And that's said repository:

[Klipper version for the Ender-3 V3 SE](https://github.com/0xD34D/klipper_ender3_v3_se)

However, in the meantime (mid of June 2024) jpcurti modified the version to bring the printer display back to life and published it in the following repo:

[Klipper version for the Ender-3 V3 SE with working display](https://github.com/jpcurti/ender3-v3-se-klipper-with-display) 

OK. That's the first step, but what now?

## The way to Klipper
The [github user dw-0](https://github.com/dw-0)  created a set of scripts to get Klipper installed, called kiauh. I used that lovely piece of software install Klipper on my Raspi.

So first I followed the steps on [https://github.com/dw-0/kiauh](https://github.com/dw-0/kiauh) to get kiauh installed.

Afterwards I installed everything using the following process.


#### 1. SSH into my Raspberry Pi as user pi


```
That shouldn't be a problem ;)
```


#### 2. Installation of git 


```
sudo apt-get update && sudo apt-get install git -y
```

#### 3. Installation of KIAUH 

```
cd ~ && git clone https://github.com/dw-0/kiauh.git
```

#### 4. Adding of jpcurtis' repository to KIAUHecho 

```
echo "https://github.com/Klipper3d/klipper" > ~/kiauh/klipper_repos.txt 

echo "https://github.com/jpcurti/ender3-v3-se-klipper-with-display" >> ~/kiauh/klipper_repos.txt
```

##### 5.1 Run KIAUH to install all the magic stuff
*(KIAUH expects you to enter the number / letter and confirm it with enter)* 


```
cd ~ && ./kiauh/kiauh.sh
```



##### 5.2. Change the repository

> **6** (Settings)  
**1** (Set custom Klipper repository)  
**1** (jpcurti/ender3-v3-se-klipper-with-display)  
**B** (Back)  
**B** (Back)  

##### 5.3. Install Klipper *(for the installation process just follow the on screen steps shown by KIAUH)*	

> **1** (Install - you may need to enter the password for the current use)  
**1** (Klipper)  
**2** (Moonraker)  
**3** or **4** (Mainsail or Fluidd, whatever you prefer)  
**B** (Back)  
**Q** (Quit)

##### 6. Configurating the new printer firmware	



```
cd ~/klipper
make menuconfig
```



With that tool you define what exactly should be compiled. So please do the following configuration:

**Micro-controller Architecture: STMicroelectronics STM32**

Processor model: **STM32F103**

Bootloader offset: **28KiB bootloader**

Communication interface: **Serial (on USART1 PA10/PA9)**

Activate the point "**Enable extra low-level configuration options**"

In the now expanded menue you have to activate "**Enable serial bridge**" and "**USART2**"

And leave menuconfig by "**Q**" and save the just made configuration.

#### 7. Compiling and installing the new printer firmware	


```
make
```



The "make" command starts the compiler. It may take a minute or two to complete it.

The compiled firmware is saved as **~/klipper/out/klipper.bin**

Copy this bin file to an empty (FAT32, 4096 assoc) SD card, and plug it into the powered off printer. Switch the printer on and wait for two minutes. 

(if the old Marlin GUI is shown on the display something went wrong. Try to rename the firmware file to f.e. firmw.bin (or any other max. 8.3 filename which is different to the last flashed firmware file name).

#### 8. Your printer is now running Klipper	 
 
#### 9. Configuration

Now it's time to configure the software (Klipper and moonraker) which is running on the computer attached to the printer.

Please be aware that many of the tutorials out there are outdated and don't reflect the current directory configurations.

I write here what worked for me.

##### Change into the directory with the settings

```
cd ~/printer_data/config
```
##### Now we pull some config files from Github	

```
wget "https://raw.githubusercontent.com/0xD34D/ender3-v3-se-klipper-config/main/prtouch.cfg"

curl "https://raw.githubusercontent.com/0xD34D/ender3-v3-se-klipper-config/main/printer-creality-ender3-v3-se-2023.cfg" > printer.cfg
```

##### Next step is adding the prtouch.cfg to the printer.cfg	echo 

```
'[include prtouch.cfg]' >> printer.cfg
```

##### And we also need to tell klipper, that a display is available	


```
echo '[e3v3se_display]' >> printer.cfg
echo 'language: english' >> printer.cfg
```


##### Checking the configuration

**Path to klippy**

 Please ensure that your moonraker.conf has the correct "klippy_uds_address" set. This should be

`/home/pi/printer_data/comms/klippy.sock`   (if your username is pi)

or

`~/printer_data/comms/klippy.sock`

##### Configure your GUI (mainsail, fluidd or octoprint)	
Installing the GUI is easily done with kiauh. Just run 

```
~/kiauh/kiauh.sh
```

select **1** for install and select your preferred option.

Please be aware that different to what most tutorials state, the serial console is located at 

`~/printer_data/comms/klippy.serial`

#### That's all?
At this point Klipper is installed on the V3 SE and on the control computer.  
Is it perfect?  
No it isn't.   
But it's already way better than the "closed source" (AKA GPL violation) original firmware.

#### But there's still something missing:

##### The printer display
~~The  display attached to the SE is not working with Klipper at the moment. Some people are currently (end of January 2024) in the process to reverse engineer the function of the display.~~ Thanks to the work of jpcurti the display is working now.

##### PRTouch V2
According to some half-hearted release of the source for the KE (sourc code of some parts still missing) Creality has developed a "different" approach for their PRTouch called bed level sensor.

Klipper is using the "old method" which is working quite fine. So no missing functionality but I think this small facts shouldn't be kept hidden.

#### Before you start printing
Ok, not you have klipper installed and the auto z-offset and bed leveling should be ready. 

But before you start printing and testing, place an old printbed (or anything else able to protect you current printbed) on the printer. 

**Neither any of the developer nor I are responsible if you crash printhead into your bed.**

First start with determining the z_offset by launching  

```
PRTOUCH_PROBE_ZOFFSET
```

 in the terminal of fluidd or mainsail.

But don't run it only once. Do it several time and look if the values reported are feasible or floating as crazy.

Also run 

```
PRTOUCH_ACCURACY SAMPLES=10 PROBE_SPEED=1
```

The output should have a rather low range (remember a standard layer height is 0.2mm.

Once everything looks good, run 

```
PRTOUCH_PROBE_ZOFFSET APPLY_Z_ADJUST=1
```

 and save the config afterwards.

Then run `PRTOUCH_PROBE_ZOFFSET` again, just to be sure. If everything looks rather stable, happy printing.

Yes, that may cost you 10 minutes, but saving a printbed may be worth it.

#### What's next? 
What? That's not all?

No, not really. So here are some remarks:

##### New commands added by 0xD34D
The following commands were added and can be called via the terminal (mainsail, fluidd or octoprint)



```
PRTOUCH_PROBE_ZOFFSET
```



This command determines the Z-Offset of the nozzle 



```
PRTOUCH_ACCURACY SAMPLES=10 PROBE_SPEED=1
```
This command tests the Z-offset probe for 10 times and returns the statistics of the accuracy of the sensor

##### Input shaping sensor

~~If one follows the procedure described on the Klipper page to make the input shaping sensor working, most likely it will fail.~~
~~To fix that, simply install the package "libopenblas-base" on your raspberry and try it again (sudo apt install libopenblas-base)~~

A recently (mid February 2024) accepted pull request on Klipper added all the required information to the official Klipper documentation. So one just have to follow the docs.

[https://github.com/Klipper3d/klipper/blob/master/docs/Measuring_Resonances.md](https://github.com/Klipper3d/klipper/blob/master/docs/Measuring_Resonances.md)


#### My "best" practise
As many hobby related things the workflow one choses is like a religion. There's a lot of grey area between wrong and right, and I just recently started using Klipper.

**Z-Offset:** The Z-offset (the difference in height between the nozzle and the PRTouch probe shouldn't change until one replaces the nozzle. That's why I decided to determine it once (and finetune it) and not do it on every print

**Mesh:** Here I decided to measure the mesh before every print. It only takes minute. Additionally I decided to use the addon "KAMP" which scales the mesh point down to the actual printing area

 **My Klipper configuration mess**
A small backup script I wrote saves the changes of my configs to my [(public) github repository: Schnoogs' Klipper Config Abyss](https://github.com/schnoog/klipperbackup)

 
### Troubleshooting - my experience (yet)
An user on reddit used the method above to install klipper. However, the connection with the serial interface failed.

The user contacted me for help.

What I found out:

His newly installed Raspbian failed to enumerate the /dev/serial/by-id  paths, so that the setting

serial: /dev/serial/by-id/usb-1a86_USB_Serial-if00-port0

in the printer config failed to define the connection. 

Quick solution

disconnect the printers USB cable, wait 5 seconds. 
plug the USB cable in, wait another 5 seconds.
execute the following command in your ssh terminal


```
find /dev/serial/by-path/ -lname '*'$(dmesg | grep USB | tail -n 1 | rev | cut -d " " -f 1 | rev)
```


the output is the path to the printers serial port, in my case `/dev/serial/by-path/platform-3f980000.usb-usb-0:1.3:1.0-port0`
change the printer.cfg to
[mcu]
serial: <the output the command gave>
  

How we solved it (the more complicated way):

disconnect the USB cable and wait for 5 seconds
plug the cable in again and wait another 5 seconds
Running the command "dmesg" from ssh 
 That should output something like 

[2395031.301884] ch341 1-1.3:1.0: ch341-uart converter detected
[2395031.305616] usb 1-1.3: ch341-uart converter now attached to ttyUSB1

That shows that the printers serial is attached to ttyUSB1 (in the users case ttyUSB0)

OK, so let's remember ttyUSB1

Running the command `ls -la /dev/serial/by-path/ | grep USB1`   (in the users case USB0) which gives the following output
> lrwxrwxrwx 1 root root 13 Apr 20 10:22 platform-3f980000.usb-usb-0:1.3:1.0-port0 -> ../../ttyUSB1

platform-3f980000.usb-usb-0:1.3:1.0-port0 in my case is the path-id of my printer (that depends on the USB port used on the pi, so this is individual to each setup)

**Glue the path together:**     /dev/serial/by-path/platform-3f980000.usb-usb-0:1.3:1.0-port0    
OK, now we have the full path for the serial console and just need to adjust our printer.cfg

by changing 



```
[mcu]
serial: /dev/serial/by-id/usb-1a86_USB_Serial-if00-port0
```



to



```
[mcu]
serial: /dev/serial/by-path/platform-3f980000.usb-usb-0:1.3:1.0-port0
```



and it works. 

However, this solution(s) come(s) with a caveat: You always need to use the same USB port on the raspberry pi. 

 
