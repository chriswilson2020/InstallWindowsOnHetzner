# InstallWindowsOnHetzner
How to install Windows server edition on a Hetzner dedicated server

Hetzner has multiple ways of installing windows server editions on dedicated nodes, I'll try to cover some because neither Hetzner nor anyone made any sensible fucking posting about this and its tiresome. 


List of methods 
- Install using Rescue system and VNC 

Some stuff you need to know before you proceed because I wont re-re explain these:<br>
<br>
**Robot** = hetzner web panel to manage servers<br>
**Rescue system** = Is available under robot > servers > click on server you have > rescue (usually third option from the left)<br>
**KVM** = A KVM switch is a hardware device that allows a user to control computers from one or more sets of keyboards, video monitors, and mice. Basically with kvm you can access the server straight from the BIOS, like you were actually there. For the nub nubs -  It's a bios level RDP to speak.<br>
**VNC viewer** = Get one for YOUR computer (not for your server) [from here](https://www.realvnc.com/en/connect/download/viewer/)<br>

Here goes 

## Method 1 
Installing Windows server image on hetzner using Rescue system and KVM

- Go to [this site](https://robot.your-server.de/server)
- Click server and from the list of servers if you have, click on your server 
- Click *rescue* and you will see this
![img1](https://i.imgur.com/Riqz6Nc.png)
  - or this
    ![image](https://github.com/TheArchivists/InstallWindowsOnHetzner/assets/9638895/1ca0bd3c-61c1-4f7b-8ff9-62cc0a3e45b3)
- From here click *activate rescue system*
- Now the page will give you ssh logins to rescue system, copy these somewhere safe we will need them later 
- Now go to *reset* section which is just to the left of *rescue* tab 
![img2](https://i.imgur.com/02uYdZY.png)
  - for dedicated<br>
    ![image](https://github.com/TheArchivists/InstallWindowsOnHetzner/assets/9638895/d6d19f7b-c052-4912-a159-7c6a7d6eb0ad)
- Click *order an automatic hardware reset* - and then click send, this is like hard rebooting your server
- Now, when the server reboots it will reboot once into the rescue system mode

### How will you know its up?
We need to ssh into the server and you will know if its up cause ssh will respond, install putty and connect to the IP of the server with port 22 and the logins that we saved earlier. 

Beep boop, you are in on ssh in rescue system. 

### Let's get to real work from this point 
- Download and extract the portable qemu-kvm. /tmp folder is enough for this portable qemu-kvm. Just run this<br>
`wget -qO- /tmp https://github.com/AnimeKaizoku/InstallWindowsOnHetzner/raw/main/vkvm.tar.gz | tar xvz -C /tmp`

- If you dedicated server has a hard drive that is more than 2TB then you need to use the UEFI version too, run this one only if you have a drive that is bigger than 2tb in size (if you have multiple drives of 2tb you can ignore this step)<br>
`wget -qO- /tmp https://github.com/AnimeKaizoku/InstallWindowsOnHetzner/raw/main/uefi.tar.gz | tar -xvz -C /tmp`

### Okay, we are setup on our emulator, lets download a windows ISO and what better place than Hetzner itself? 

Hetzner has 2 types of mirrors - Internal and external, the internal one is accessible without login from *inside the hetzner network* and external one is public. 
We will use the external one to see the path of the image we want and use the internal url (they both have same structure)

Visit [this site](http://download.hetzner.com/bootimages/)<br>
The logins are<br>
Username: `hetzner`<br>
Password: `download`<br>

To save you the time I have the external url's here 

[Win Server STD CORE 2016 64Bit](http://download.hetzner.com/bootimages/windows/SW_DVD9_Win_Server_STD_CORE_2016_64Bit_English_-4_DC_STD_MLF_X21-70526.ISO)<br>
[Win Server STD CORE 2019 64Bit](http://download.hetzner.com/bootimages/windows/SW_DVD9_Win_Server_STD_CORE_2019_1809.11_64Bit_English_DC_STD_MLF_X22-51041.ISO)<br>
[Win Server STD CORE 2022 64Bit](http://download.hetzner.com/bootimages/windows/SW_DVD9_Win_Server_STD_CORE_2022_2108.15_64Bit_English_DC_STD_MLF_X23-31801.ISO)<br>

#### If link is incorrect, go to https://download.hetzner.com/bootimages/windows/ and find correct name of .ISO file

There are 3 images, I am using 2022 

Now close your eyes and run these commands in serial order<br>
```
cd /tmp
wget http://mirror.hetzner.de/bootimages/windows/SW_DVD9_Win_Server_STD_CORE_2022_2108.15_64Bit_English_DC_STD_MLF_X23-31801.ISO
cd /
```


### Now we run the qemu kun by running this command
**Do not hit enter until you read the VNC section first you sick fucks**

For servers with no 2TB+ NVME
```
/tmp/qemu-system-x86_64 -net nic -net user,hostfwd=tcp::3389-:3389 -m 10000M -localtime -enable-kvm -cpu core2duo,+nx -smp 2 -usbdevice tablet -k en-us -cdrom /tmp/SW_DVD9_Win_Server_STD_CORE_2022_2108.15_64Bit_English_DC_STD_MLF_X23-31801.ISO -hda /dev/nvme0n1 -boot once=d -vnc :1
```

For servers with a 2TB+ NVME and UEFI
```
/tmp/qemu-system-x86_64 -bios /tmp/uefi.bin -net nic -net user,hostfwd=tcp::3389-:3389 -m 10000M -localtime -enable-kvm -cpu core2duo,+nx -smp 2 -usbdevice tablet -k en-us -cdrom /tmp/SW_DVD9_Win_Server_STD_CORE_2022_2108.15_64Bit_English_DC_STD_MLF_X23-31801.ISO -hda /dev/nvme0n1 -boot once=d -vnc :1
```

For servers with no 2TB+ HDD
```
/tmp/qemu-system-x86_64 -net nic -net user,hostfwd=tcp::3389-:3389 -m 10000M -localtime -enable-kvm -cpu core2duo,+nx -smp 2 -usbdevice tablet -k en-us -cdrom /tmp/SW_DVD9_Win_Server_STD_CORE_2022_2108.15_64Bit_English_DC_STD_MLF_X23-31801.ISO -hda /dev/sda -boot once=d -vnc :1
```

For servers with a 2TB+ HDD and UEFI
```
/tmp/qemu-system-x86_64 -bios /tmp/uefi.bin -net nic -net user,hostfwd=tcp::3389-:3389 -m 10000M -localtime -enable-kvm -cpu core2duo,+nx -smp 2 -usbdevice tablet -k en-us -cdrom /tmp/SW_DVD9_Win_Server_STD_CORE_2022_2108.15_64Bit_English_DC_STD_MLF_X23-31801.ISO -hda /dev/sda -boot once=d -vnc :1
```

## This is the VNC section
Once that is ready and before you hit enter on that ssh command lets open VNC viewer on your computer so we can connect to this RDP session that we just made 
Put the IP of the server in the connection field like this
yourIP:1

Where yourIP is ofc your server IP<br>
Example: [*YOUR IP HERE*]:1


Now hit enter on the ssh, then hit connect on the VNC viewer
Hit connect, vnc will connect you to an rdp session where you will load the OS - in some cases, because its loading the OS like its from a pendrive you might get a *press and key to boot from disk/cd* so if you run the ssh command first you might miss this prompt and the ISO might not boot. 
This is why i said to prepare VNC first. 

From here for HDD < 2TB its just a normal windows install, do that yada yada, install the os
If you have a HDD or NVME > 2TB I reccomend selecting repair windows and just making sure you have the drive set up as GPT you can do that following the screenshot below:
<img width="776" alt="Screenshot 2024-02-11 at 12 17 22" src="https://github.com/chriswilson2020/InstallWindowsOnHetzner/assets/73828727/502d82e4-b507-49fb-adb7-9196de6447fd">

Login to the server 

Go to my computer properties once your server is up > advanced system settings > remote > and tick the following 

![img3](https://i.imgur.com/BdmEbaL.png)

Save, then click reboot and then try to connect. 
Done, **you are splendid**
