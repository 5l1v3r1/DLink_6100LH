# DLink 6100LH

Update: I was about to use tcl and expect to write a brute-force scripts for find the pin-code but it was never neded.

#### Hardware Used:

  * Arduino SA Uno R3 (CDC ACM)
  * DLINK 6100LH IPCam
  * System: Gentoo Linux

#### About/Info/Reason 

Why I disassembled this dlink camera is because I did not have the password and in the Dlinks app you can only add the camera by connecting to their wifi so I disassembled the device and hoped that it would be easy to get root access and solve this the problem.

It was easy done and there is three screws to disassemble this device and three contacts to release, txd/rxd/ gnd I found within 1 seconds, to the left of the micro-usb input, see pictures below.

However, I logged in to the access point from the camera since password is printed in stdout when reading serial console and now I could finally use the camera because it is quite good for the cheap price you get it for in 1080p, **I THOUGHT!**

When I reached the last setting, the dlink / camera also requires a PIN code and now I have to hack the device anyway and bypass the login (update: this is now done, see more below)

### Some photos taken during the process

![Screenshot](.preview/0.jpg)
![Screenshot](.preview/1.jpg)
![screenshot](.preview/2.jpg)

#### The chinese characters, I tried google the characters without any luck and didnt spend more time in digging deeper but Im sure D-Link doing as everyone else, buying some cheap cameras from china, putting their own awful logo on it and re-selling it for 10x higher price then in china. We all can belive how things are but proof is allways neded. What a fucking joke, Boycott all IoT and american companies for your own safety and go buy the IoT devices from asia instead, no reason to pay 10x when you dont need.

![Screenshot](.preview/3.jpg)

#### And we all *nix n3rds is well aware of 2034, odd odd its on the board on this camera. See upper left corner :) 

![Screenshot](.preview/2034.jpg)

#### STdin/STdout 

* We read serial console with a while loop and writing to a file, with some simple bash tricks without using screen or other tools, we can write to file and grep what we want as we want, see below:

* Script for read serial communication

      #!/bin/bash

      while true; 
        do tty=/dev/ttyACM0
        exec 4<$tty 5>$tty
        stty -F $tty 115200 -echo >&5;
        read r <&4;
        echo "$r"; 
      done |tee dc_6200lh.txt

* Without the while loop things wont be monitored and print everything, you will read everything one line by another and it sucks. Especially if you wanna bruteforce the login then we want see what happens in another window. For read username and password like above the loop is required, however it is an awesome way to read serial communications without using more advanced language, tcl is awesome for this otherwise and for everyone that is new to this and curios about this topic:

```tcl
set baud 115200
if { $argc >= 1 } {
 set tty [lindex $argv 0]
}
if { $argc >= 2 } {
 set baud [lindex $argv 1]
}
spawn screen /dev/ttyACM0 $baud
send \r
expect {
      "DCS-6100LH login: " {
           send admin\r
           expect "Password: "
           send $pin-codes\r
       }
       "#" 
    }
interact
```

And for bruteforce this, we know the pin-code is a length of six numbers so IF the pin-code wasnt in serial output I would just generate numbers from 00000-999999 and in a for loopp, really simple and basic tcl and shell knowledge needed, however since the serial communication replies within milliseconds after a failed login we would hack this pin-code within few monutes. My slow i7 cpu I generats numbers from 000000 to 999999 in 26 seconds, but for stay on the safe side we could add a timeout every 500ms in the code above for not missing any exit status for know when pin code would ahve been found, so something below would done the job for us:

```sh
for pin-codes in $(seq -w 000000 99999); do
....tcl code above
.......**set timeout -1**
done
```

Now you have a great start for getting started if you wanna brute-force a device via serial communication, actually I made a quick google search on this and there was nothing found really but its really easy to fix, tcl **pwnz** when we using serial communications for 'expect(ing)' and automate logins, for example.

##### Grep Password

* We can read the dc_6200lh.txt in realtime, and also we can grep what we want - **Nice**! 

      tail -f dc_6200lh.txt |egrep -i "Wifi_ap_pwd"
 
##### And here we go, pin-code found:

    tail -f dc_6200lh.txt|egrep -o 'user=admin,pass=......' 

![Screenshot](.preview/get_pin.gif)
   
* Catching, **done**.  No hacks needed and no brute-force needed.

#### See Full Boot Process

Right click on the video and press open image on a new tab for get 1080p resolution for readable text.

![Screenshot](.preview/4.gif)


#### There is nothing more to add about this device that is interesting that you cant figure out yourself, btw, default ip and opened ports: 

    IP: 192.168.0.20 
    
    Ports: 
    554
    8080
    7000
    6000
  
## U-Boot:

    U-Boot 2016.11 (Jul 01 2020 - 17:46:25 +0800)
    mips-linux-uclibc-xgcc (Realtek RSDK-6.4.1 Build 3029) 6.4.1 20180425
    GNU ld (Realtek RSDK-6.4.1 Build 3029) 2.27.90.20161222
    => help     
    ?       - alias for 'help'
    base    - print or set address offset
    bdinfo  - print Board Info structure
    boot    - boot default, i.e., run 'bootcmd'
    bootd   - boot default, i.e., run 'bootcmd'
    bootelf - Boot from an ELF image in memory
    bootm   - boot application image from memory
    bootvx  - Boot vxWorks from an ELF image
    cmp     - memory compare
    coninfo - print console devices and information
    cp      - memory copy
    crc32   - checksum calculation
    dma     - dma copy
    editenv - edit environment variable
    env     - environment handling commands
    fatinfo - print information about filesystem
    fatload - load binary file from a dos filesystem
    fatls   - list files in a directory (default /)
    fatsize - determine a file's size
    fephy   - fephy read/write
    go      - start application at address 'addr'
    help    - print command description/usage
    iminfo  - print header information for application image
    imls    - list all images found in flash
    imxtract- extract a part of a multi-image
    md      - memory display
    mm      - memory modify (auto-incrementing address)
    mmc     - MMC sub system
    mmcinfo - display MMC info
    mw      - memory write (fill)
    nfs     - boot image via network using NFS protocol
    nm      - memory modify (constant address)
    ping    - send ICMP ECHO_REQUEST to network host
    printenv- print environment variables
    reset   - Perform RESET of the CPU
    run     - run commands in an environment variable
    saveenv - save environment variables to flash
    setenv  - set environment variables
    setethaddr- set eth address
    setipaddr- set ip address
    sf      - SPI flash sub-system
    sleep   - delay execution for some time
    tftpboot- boot image via network using TFTP protocol
    tftpput - TFTP put command, for uploading files to a server
    tftpsrv - act as a TFTP server and boot the first received file
    update  - update xxx
    version - print monitor, compiler and linker version

    => printenv 
    baudrate=57600
    bootaddr=0xBC000000 + 0x50000
    bootargs=console=ttyS1,115200 root=/dev/mtdblock3 rts-quadspi.channels=dual mtdparts=18030000.spic:16384k@0(global),320k@0k(boot),2304k@320k(kernel),3584k@2624k(rootfs),7744k@6208k(userdata),2048k@13952k(userdata2),384k@16000k(userdata3)
    bootcmd=bootm 0xbc050000
    bootdelay=2
    ethact=r8168#0
    loadaddr=0x80010000
    netretry=yes
    stderr=serial
    stdin=serial
    stdout=serial

Access UBOOT by hit any key before 5 seconds when it boots, and from u-boot menu you should read on some wiki how to use the commands but printenv or 'pr' will print the interesting things, to edit bootargs for example you just tneet to type: 
 
    setenv bootargs console=ttyS0,115200............... 
   
Bypass login by add the below line to bootargs but it is really not needed since password is the pin-code we can grab as on the video provided above and yeah so  are the default username wich is: admin:

    setenv bootargs $(BOOTARGS) init=/bin/sh 
    boot

I do not recommending anyone to change bootargs cause its useless as said. If you get stuck in boot after you played with bootargs and get a kernel panic for some reason - **DON'T PANIC ;)** - Restart the device and enter CFE boot and reset default by:

    env default -a
    saveenv
    reset
   
## Sources:

[Device Info](https://www.dlink.com/en/products/dcs-6100lh-compact-full-hd-wi-fi-camera)

## Changelog

[Versions changelog](CHANGELOG.md).

## Authors: 

* **wuseman <wuseman@nr1.nu\>** 

## License

This project is licensed under the GNU General Public License v3.0 - see the [LICENSE.md](LICENSE.md) file for details

### Contact

  If you have problems, questions, ideas or suggestions please contact me on *_wuseman@nr1.nu_  - For faster contact visit Libera irc network or the webchat and type '/msg wuseman hi!' in the input bar and I will reply to you ASAP.
  
  Enter Libera's network via your own client 'chat.libera.chat:+6697 or use their new web client [here](https://web.libera.chat/).

#### Happy Hacking! 
