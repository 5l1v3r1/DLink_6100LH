# DLink 6100LH

Update: I was about to use tcl and expect to write a brute-force scripts for find the pin-code but it was never neded.

#### Hardware Used:

  * Arduino SA Uno R3 (CDC ACM)
  * DLINK 6100LH IPCam
  * System: Gentoo Linux

#### About/Info/Reason 

Why I disassembled this dlink camera is because I did not have the password and in the Dlinks app you can only add the camera by connecting to their wifi so I disassembled the device and hoped that it would be easy to get root access and solve this the problem.

It did not take much, three screws to disassemble this device and txd / rxd / gnd I found before I he flashed at the bottom of the usb port there were 3 inputs, the password was found directly in the bootlog.

I logged in to the access point from the camera and now I can finally use the camera because it is quite good for the cheap price you get it for in 1080p, **I THOUGHT!**

When I reached the last setting, the dlink / camera also requires a PIN code and now I have to hack the device anyway and bypass the login, I will re-update this page when I succeed, I never googling before I succeed, thats cheat ;)

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
      read reply <&4;echo "$reply"; 
    done |tee dc_6200lh.txt


##### Grep Password

* We can read the dc_6200lh.txt in realtime, and also we can grep what we want - Nice! 

    tail -f dc_6200lh.txt |egrep -i "Wifi_ap_pwd"
 
##### And here we go, pin-code found:

![Screenshot](.preview/get_pin.gif)
   
    tail -f dc_6200lh.txt|egrep -o 'user=admin,pass=......' 

* Catching, done, no hacks needed and no brute-force needed.

#### Full Boot Process

Right click on the video and press open image on a new tab for get 1080p resolution for readable text.

![Screenshot](.preview/4.gif)


