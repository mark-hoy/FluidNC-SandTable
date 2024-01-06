# FluidNC-SandTable
FluidNC Downloading to a new ESP32-WROOM-32D board.

How to download and create a brand new ESP32 with FluidNC for use with Bart's old ESP32 Pen/Laser board on my M1 Mac Mini runnin OSX version 14.1.2.
http://wiki.fluidnc.com/en/installation#upgrading-firmware

You'll need a recent version of Python 3.6 or greater (according to the HOWTO-INSTALL.txt), I happen to have Python 3.11.4

```
$ python3 --version
Python 3.11.4
```

Sourced some ESP32 boards from Amazon that look exactly like the ones I got from Bart pre-loaded years ago
Also got some Lexar 64G microSDs (3 for less than $20). However for FluidNC, you will need to partition the card
and ONLY use 32G (also, don't use 2G or smaller cards as they aren't supported). See the end if you need to partition a card.

Docs on the reference board (in particular, locate the Boot button you might need it during the install if a software reboot fails).
https://docs.espressif.com/projects/esp-idf/en/v5.1/esp32/hw-reference/esp32/get-started-devkitc.html#get-started-esp32-devkitc-board-front

Step 1, download the latest release of FluidNC from https://github.com/bdring/FluidNC/releases which in my case is 3.7.12 (fluidnc-v3.7.12-posix.zip)
Also download the matching Web UI firmware file as this needs to be added by hand (at this point of development)
https://github.com/bdring/FluidNC/blob/main/FluidNC/data/index.html.gz

Basic steps:
<ol>
  -download the build and unpack it
</ol>
<ol>
  -change into the directory
</ol>
<ol>
-if it's a new microSD card, erase it (./erase.sh) and format (./install_fs.sh)
</ol>
<ol>
  -install WiFi version of FluidNC (./install_wifi.sh)
</ol>

Download and Unpack it

```
$ unzip ~/Downloads/fluidnc-v3.7.12-posix.zip 
Archive:  /Users/mark_hoy/Downloads/fluidnc-v3.7.12-posix.zip
 extracting: fluidnc-v3.7.12-posix/HOWTO-INSTALL.txt  
 extracting: fluidnc-v3.7.12-posix/common/boot_app0.bin  
 extracting: fluidnc-v3.7.12-posix/common/SecurityFusesOK.bin  
 extracting: fluidnc-v3.7.12-posix/common/SecurityFusesOK0.bin  
 extracting: fluidnc-v3.7.12-posix/wifi/bootloader.bin  
 extracting: fluidnc-v3.7.12-posix/wifi/littlefs.bin  
 extracting: fluidnc-v3.7.12-posix/wifi/index.html.gz  
 extracting: fluidnc-v3.7.12-posix/wifi/firmware.bin  
 extracting: fluidnc-v3.7.12-posix/wifi/partitions.bin  
 extracting: fluidnc-v3.7.12-posix/install-wifi.sh  
 extracting: fluidnc-v3.7.12-posix/bt/bootloader.bin  
 extracting: fluidnc-v3.7.12-posix/bt/firmware.bin  
 extracting: fluidnc-v3.7.12-posix/bt/partitions.bin  
 extracting: fluidnc-v3.7.12-posix/install-bt.sh  
 extracting: fluidnc-v3.7.12-posix/install-fs.sh  
 extracting: fluidnc-v3.7.12-posix/fluidterm.sh  
 extracting: fluidnc-v3.7.12-posix/checksecurity.sh  
 extracting: fluidnc-v3.7.12-posix/erase.sh  
 extracting: fluidnc-v3.7.12-posix/tools.sh  
 extracting: fluidnc-v3.7.12-posix/common/fluidterm.py  
 extracting: fluidnc-v3.7.12-posix/common/README-FluidTerm.md  
 extracting: fluidnc-v3.7.12-posix/common/README-ESPTOOL-SOURCE-v3.1.txt  
 extracting: fluidnc-v3.7.12-posix/common/esptool-source.zip  

$ cd fluidnc-v3.7.12-posix
```

Format the internal Flash of the ESP32. This will contain only critical files, typically things like: index.html.gz and config.yaml

```
$ ./erase.sh 
esptool.py --chip esp32 --baud 230400 dump_mem 0x3ff5a018 4 SecurityFuses.bin
esptool.py v3.1
Found 3 serial ports
Serial port /dev/cu.wlan-debug
Connecting........_____....._____....._____....._____....._____....._____....._____
/dev/cu.wlan-debug failed to connect: Failed to connect to ESP32: Timed out waiting for packet header
Serial port /dev/cu.usbserial-0001
Connecting........_____.
Chip is ESP32-D0WDQ6 (revision 1)
Features: WiFi, BT, Dual Core, 240MHz, VRef calibration in efuse, Coding Scheme None
Crystal is 40MHz
MAC: 0c:b8:15:c3:10:24
Uploading stub...
Running stub...
Stub running...
Changing baud rate to 230400
Changed.
Read 4 bytes
Done!
Hard resetting via RTS pin...
esptool.py --chip esp32 --baud 230400 erase_flash
esptool.py v3.1
Found 3 serial ports
Serial port /dev/cu.wlan-debug
Connecting........_____....._____....._____....._____....._____....._____....._____
/dev/cu.wlan-debug failed to connect: Failed to connect to ESP32: Timed out waiting for packet header
Serial port /dev/cu.usbserial-0001
Connecting........_
Chip is ESP32-D0WDQ6 (revision 1)
Features: WiFi, BT, Dual Core, 240MHz, VRef calibration in efuse, Coding Scheme None
Crystal is 40MHz
MAC: 0c:b8:15:c3:10:24
Uploading stub...
Running stub...
Stub running...
Changing baud rate to 230400
Changed.
Erasing flash (this may take a while)...
Chip erase completed successfully in 5.7s
Hard resetting via RTS pin...
```

Now create the internal flash file system, this will wipe out previous data

```
$ ./install-fs.sh 
esptool.py --chip esp32 --baud 230400 dump_mem 0x3ff5a018 4 SecurityFuses.bin
esptool.py v3.1
Found 3 serial ports
Serial port /dev/cu.wlan-debug
Connecting........_____....._____....._____....._____....._____....._____....._____
/dev/cu.wlan-debug failed to connect: Failed to connect to ESP32: Timed out waiting for packet header
Serial port /dev/cu.usbserial-0001
Connecting........__
Chip is ESP32-D0WDQ6 (revision 1)
Features: WiFi, BT, Dual Core, 240MHz, VRef calibration in efuse, Coding Scheme None
Crystal is 40MHz
MAC: 0c:b8:15:c3:10:24
Uploading stub...
Running stub...
Stub running...
Changing baud rate to 230400
Changed.
Read 4 bytes
Done!
Hard resetting via RTS pin...
esptool.py --chip esp32 --baud 230400 --before default_reset --after hard_reset write_flash -z --flash_mode dio --flash_freq 80m --flash_size detect 0x3d0000 wifi/littlefs.bin
esptool.py v3.1
Found 3 serial ports
Serial port /dev/cu.wlan-debug
Connecting........_____....._____....._____....._____....._____....._____....._____
/dev/cu.wlan-debug failed to connect: Failed to connect to ESP32: Timed out waiting for packet header
Serial port /dev/cu.usbserial-0001
Connecting........____
Chip is ESP32-D0WDQ6 (revision 1)
Features: WiFi, BT, Dual Core, 240MHz, VRef calibration in efuse, Coding Scheme None
Crystal is 40MHz
MAC: 0c:b8:15:c3:10:24
Uploading stub...
Running stub...
Stub running...
Changing baud rate to 230400
Changed.
Configuring flash size...
Auto-detected Flash size: 4MB
Flash will be erased from 0x003d0000 to 0x003fffff...
Compressed 196608 bytes to 117900...
Wrote 196608 bytes (117900 compressed) at 0x003d0000 in 5.3 seconds (effective 295.6 kbit/s)...
Hash of data verified.

Leaving...
Hard resetting via RTS pin...
```

Decide what version you want to install (WiFi or BlueTooth), in my case I want the WiFi version.

```
$ ./install-wifi.sh 
esptool.py --chip esp32 --baud 230400 dump_mem 0x3ff5a018 4 SecurityFuses.bin
esptool.py v3.1
Found 3 serial ports
Serial port /dev/cu.wlan-debug
Connecting........_____....._____....._____....._____....._____....._____....._____
/dev/cu.wlan-debug failed to connect: Failed to connect to ESP32: Timed out waiting for packet header
Serial port /dev/cu.usbserial-0001
Connecting....
Chip is ESP32-D0WDQ6 (revision 1)
Features: WiFi, BT, Dual Core, 240MHz, VRef calibration in efuse, Coding Scheme None
Crystal is 40MHz
MAC: 0c:b8:15:c3:10:24
Uploading stub...
Running stub...
Stub running...
Changing baud rate to 230400
Changed.
Read 4 bytes
Done!
Hard resetting via RTS pin...
esptool.py --chip esp32 --baud 230400 --before default_reset --after hard_reset write_flash -z --flash_mode dio --flash_freq 80m --flash_size detect 0x1000 wifi/bootloader.bin 0xe000 common/boot_app0.bin 0x10000 wifi/firmware.bin 0x8000 wifi/partitions.bin
esptool.py v3.1
Found 3 serial ports
Serial port /dev/cu.wlan-debug
Connecting........_____....._____....._____....._____....._____....._____....._____
/dev/cu.wlan-debug failed to connect: Failed to connect to ESP32: Timed out waiting for packet header
Serial port /dev/cu.usbserial-0001
Connecting........_____...
Chip is ESP32-D0WDQ6 (revision 1)
Features: WiFi, BT, Dual Core, 240MHz, VRef calibration in efuse, Coding Scheme None
Crystal is 40MHz
MAC: 0c:b8:15:c3:10:24
Uploading stub...
Running stub...
Stub running...
Changing baud rate to 230400
Changed.
Configuring flash size...
Auto-detected Flash size: 4MB
Flash will be erased from 0x00001000 to 0x00005fff...
Flash will be erased from 0x0000e000 to 0x0000ffff...
Flash will be erased from 0x00010000 to 0x001b8fff...
Flash will be erased from 0x00008000 to 0x00008fff...
Warning: some reserved header fields have non-zero values. This image may be from a newer esptool.py?
Compressed 17520 bytes to 12170...
Wrote 17520 bytes (12170 compressed) at 0x00001000 in 0.7 seconds (effective 207.6 kbit/s)...
Hash of data verified.
Compressed 8192 bytes to 47...
Wrote 8192 bytes (47 compressed) at 0x0000e000 in 0.1 seconds (effective 1262.4 kbit/s)...
Hash of data verified.
Compressed 1740592 bytes to 1039169...
Wrote 1740592 bytes (1039169 compressed) at 0x00010000 in 46.2 seconds (effective 301.1 kbit/s)...
Hash of data verified.
Compressed 3072 bytes to 129...
Wrote 3072 bytes (129 compressed) at 0x00008000 in 0.1 seconds (effective 351.4 kbit/s)...
Hash of data verified.

Leaving...
Hard resetting via RTS pin...
Starting fluidterm

--- Available ports:
---  1: /dev/cu.Bluetooth-Incoming-Port 'n/a'
---  2: /dev/cu.usbserial-0001 'CP2102 USB to UART Bridge Controller'
---  3: /dev/cu.wlan-debug   'n/a'
--- Enter port index or full name: 2
--- FluidTerm v1.2.0 on /dev/cu.usbserial-0001  115200,8,N,1 ---
--- Quit: Ctrl+] or Ctrl+Q | Upload: Ctrl+U | Reset: Ctrl+R | ClearScreen: Ctrl+W ---
.....

[MSG:INFO: FluidNC v3.7.12 https://github.com/bdring/FluidNC]
[MSG:INFO: Compiled with ESP32 SDK:v4.4.4]
[MSG:INFO: Local filesystem type is spiffs]
[MSG:WARN: Cannot open configuration file:config.yaml]
[MSG:INFO: Using default configuration]
[MSG:INFO: Axes: using defaults]
[MSG:INFO: Machine Default (Test Drive)]
[MSG:INFO: Board None]
[MSG:INFO: Stepping:RMT Pulse:4us Dsbl Delay:0us Dir Delay:0us Idle Delay:255ms]
[MSG:INFO: Axis count 3]
[MSG:INFO: Axis X (-1000.000,0.000)]
[MSG:INFO:   Motor0]
[MSG:INFO: Axis Y (-1000.000,0.000)]
[MSG:INFO:   Motor0]
[MSG:INFO: Axis Z (-1000.000,0.000)]
[MSG:INFO:   Motor0]
[MSG:INFO: Kinematic system: Cartesian]
[MSG:INFO: Using spindle NoSpindle]
[MSG:INFO: STA SSID is not set]
[MSG:INFO: AP SSID FluidNC IP 192.168.0.1 mask 255.255.255.0 channel 1]
[MSG:INFO: AP started]
[MSG:INFO: WiFi on]
[MSG:INFO: Captive Portal Started]
[MSG:INFO: HTTP started on port 80]
[MSG:INFO: Telnet started on port 23]

Grbl 3.7 [FluidNC v3.7.12 (wifi) '$' for help]
```

Now setup your WiFi. One way is by connecting to the built in WiFi access point at 192.168.0.1 with your laptop/cellphone and 
enter your WiFi SSID and Password (don't forget to hit the SET button once you enter info).

Via fluidterm.sh, reboot with Ctrl-R

```
$ ./fluidterm.sh 

--- Available ports:
---  1: /dev/cu.Bluetooth-Incoming-Port 'n/a'
---  2: /dev/cu.usbserial-0001 'CP2102 USB to UART Bridge Controller'
---  3: /dev/cu.wlan-debug   'n/a'
--- Enter port index or full name: 2
--- FluidTerm v1.2.0 on /dev/cu.usbserial-0001  115200,8,N,1 ---
--- Quit: Ctrl+] or Ctrl+Q | Upload: Ctrl+U | Reset: Ctrl+R | ClearScreen: Ctrl+W ---
.....

.
.
.
$sta/ssid=YOUR-SSID-HERE
ok
$sta/Password=YOUR-PASSWORD-FOR-WIFI-HERE
ok
$sta/Password
$Sta/Password=********
ok
Resetting MCU
ets Jun  8 2016 00:22:57

```
Upon reboot, there will be an error from a cleared system as there is no config.yaml on the file system yet, this is to be expected.

```

rst:0x1 (POWERON_RESET),boot:0x13 (SPI_FAST_FLASH_BOOT)
configsip: 0, SPIWP:0xee
clk_drv:0x00,q_drv:0x00,d_drv:0x00,cs0_drv:0x00,hd_drv:0x00,wp_drv:0x00
mode:DIO, clock div:1
load:0x3fff0030,len:1184
load:0x40078000,len:13220
ho 0 tail 12 room 4
load:0x40080400,len:3028
entry 0x400805e4

[MSG:INFO: FluidNC v3.7.12 https://github.com/bdring/FluidNC]
[MSG:INFO: Compiled with ESP32 SDK:v4.4.4]
[MSG:INFO: Local filesystem type is littlefs]
[MSG:WARN: Cannot open configuration file:config.yaml]
```

<h3>Reboot and ensure FluidNC connect to your WiFi Access Point</h3>

You should see the following after the reboot inside FluidTerm IF the Esp32 connects to your WiFi correctly

```
[MSG:INFO: Connecting to STA SSID:YOUR-SSID-HERE]
[MSG:INFO: Connecting.]
[MSG:INFO: Connecting..]
[MSG:INFO: Connected - IP is 192.168.1.222]
[MSG:INFO: WiFi on]
[MSG:INFO: Start mDNS with hostname:http://fluidnc.local/]
[MSG:INFO: SSDP Started]
[MSG:INFO: HTTP started on port 80]
[MSG:INFO: Telnet started on port 23]

Grbl 3.7 [FluidNC v3.7.12 (wifi) '$' for help]
```

You can now bring up the web page of FluidNC in your browser for configuration of the config.yaml file (or upload it via fluidterm.sh)
In my case, the IP address is: 192.168.1.222


<h3>Disk Partioning</h3>
I had a new microSD card and I wanted it wiped and a FAT32 partition installed onto it.
Don't do this is you have something on the microSD you care about.

An example of partitioning a 64G disk into two 32G Fat32 partitions.
FluidNC will only use partition ONE. It's CRITICAL you get the disk correct, otherwise you could wipe out something else by mistake.

Use the following to see the various disks available on your Mac.
```
$ diskutil list
```

Once you are absolutely sure of the disk (which in my case was a 64G on disk7) you can partition it into two parts, replace "diskX" with your disk info.

```
$ diskutil partitionDisk diskX MBR FAT32 one 32G FAT32 two 31G
Started partitioning on diskX
Unmounting disk
Creating the partition map
Waiting for partitions to activate
Formatting disk7s1 as MS-DOS (FAT32) with name one
512 bytes per physical sector
/dev/rdisk7s1: 62469440 sectors in 1952170 FAT32 clusters (16384 bytes/cluster)
bps=512 spc=32 res=32 nft=2 mid=0xf8 spt=32 hds=255 hid=2048 drv=0x80 bsec=62500000 bspf=15252 rdcl=2 infs=1 bkbs=6
Mounting disk
Formatting disk7s2 as MS-DOS (FAT32) with name two
512 bytes per physical sector
/dev/rdisk7s2: 61064032 sectors in 1908251 FAT32 clusters (16384 bytes/cluster)
bps=512 spc=32 res=32 nft=2 mid=0xf8 spt=32 hds=255 hid=62502912 drv=0x80 bsec=61093888 bspf=14909 rdcl=2 infs=1 bkbs=6
Mounting disk
Finished partitioning on disk7
/dev/disk7 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *63.3 GB    disk7
   1:                 DOS_FAT_32 ONE                     32.0 GB    disk7s1
   2:                 DOS_FAT_32 TWO                     31.3 GB    disk7s2
```
