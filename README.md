# Xiaomi-daisy-pmOS
All the code in this repository is distributed under the GNU GPL 2.0 license, read [here](https://github.com/NotLugozzi/Xiaomi-daisy-pmOS/blob/main/LICENSE) for further details
### working(ish) port of postmarketOS to the xiaomi mi a2 lite(daisy)
 This projet aims to run a fully featured installation of alpine linux on the xiaomi mi a2 lite. It isn't mature enough to be daily driven as it only runs the [weston](https://youtu.be/JLhaONV8zBw) demo as of now - We are very sorry for the slow pace of the development, its a team of 2 porting to a brand new device and getting a basic framebuffer demo in less than 4 days has been a pretty good thing. 

## List of what works and what doesnt:
#### works:
- [x] Building
- [x] Installing
- [x] Booting
- [x] Weston interface
- [x] SSH to the device
- [x] Wayland now knows the screen resolution
- [x] Charging and thermal controls(P)
- [x] Touchscreen support(P)
- [x] Usb networking(P)
- [x] Plasma mobile(P)
#### broken/partial support:
- [ ] Wireless connectivity - this includes mobile data, wifi and bluetooth
- [ ] usb OTG
- [ ] camera
- [ ] Mainline pmOS kernel

# Installation
First of all you'll need a linux system(or vm), a Mi A2 Lite with an unlocked bootloader and a couple hours..
#### Install pmbootstrap
You'll need this to compile the kernel and the necessary files
```python
pip3 install --user pmbootstrap
```
after you've installed it, copy the linux-xiaomi-daisy-2 and the device-xiaomi-daisy-2 folders in your pmbootstrap environment (/home/user/.local/var/pmbootstrap/cache_git/pmaports/device/community/), now from the terminal run
```python
pmbootstrap init
```
and set it up. At this point you can select weston or [plasma mobile](https://github.com/NotLugozzi/Xiaomi-daisy-pmOS#plasma) as the interface and you allow the proprietary drivers
#### Building the kernel
After setting pmbootstrap up run this 2 commands, the first will validate checksums for all the necessary patches and for the kernel itself, the second one will verify the device specific package. If they don't give any errors we can move to actually building The Kernel
```python
pmbootstrap checksum linux-xiaomi-daisy-2
pmbootstrap checksum device-xiaomi-daisy-2
```
We're going to compile 2 files, the kernel itself and some device specific packages; at this point you should open another terminal window running **pmbootstrap log** to check the compile job in real time. The first command we'll run is this
```python
pmbootstrap build linux-xiaomi-daisy-2
```
go grab a coffee and some snacks as this can take up to 3 hours on older hardware - hopefully you'll only have to compile once, without any errors or stuff like that - if you had errors you should try running **pmbootstrap kconfig check**. After you've compiled the kernel, run this command for the specific packages
```python
pmbootstrap build device-xiaomi-daisy-2
```
We're very very close to booting, only a couple of commands left; one of them will generate the boot images we'll need:
```python
pmbootstrap install
```
Just follow the installation process and keep an eye on the log window _that i hope you still have running._ At this point you have 2 options: either flashing both the rootfs and the kernel from pmbootstrap(reccomended if you have a _actual pc_ and not a vm) or moving the rootfs to your host, flashing that with fastboot, with this command:
```
fastboot flash system path/to/xiaomi-daisy-2.img
```
and then connecting your phone to the vm and flashing the kernel on that. Personally i use the second method and it works great, no matter what you choose, The first boot will probably take a minute or so because it has to initiallize some stuff. you'll get the pmos boot screen and (hopefully)the weston demo desktop:
![boot](https://github.com/NotLugozzi/Xiaomi-daisy-pmOS/blob/main/images/pmOS%20succesful%20boot.jpg)
Just to be sure everything works, run dmesg and check if you get this output:
![dmesg](https://github.com/NotLugozzi/Xiaomi-daisy-pmOS/blob/main/images/dmesg.png)
##### Experimental Features. 
if you want to start demos from the ssh shell, edit the runtime dir script using nano **(sudo nano /etc/profile.d/xdg_runtime_dir.sh)**. You'll need to add this:
```
if test -z "${XDG_RUNTIME_DIR}"; then
  export XDG_RUNTIME_DIR=/tmp/$(id -u)-runtime-dir
  if ! test -d "${XDG_RUNTIME_DIR}"; then
    mkdir "${XDG_RUNTIME_DIR}"
    chmod 0700 "${XDG_RUNTIME_DIR}"
  fi
fi
```
The power button isnt currently implemented in the device package, you'll have to edit buttons.conf to make it work. using nano edit this file **/etc/triggerhappy/triggers.d/buttons.conf** and add the lines according to what you need:
```
KEY_VOLUMEDOWN+KEY_POWER 1 reboot 
```
```
KEY_VOLUMEUP+KEY_POWER 1 poweroff 
```
```
KEY_POWER 1 poweroff 
```
```
KEY_VOLUMEUP 1 poweroff 
```
### Plasma
If you want to use the plasma mobile DE you'll need to edit **/usr/bin/kwinwrapper** first of all you'll need to comment out the default startup line
```
#startplasma-wayland --xwayland --libinput --lockscreen --inputmethod maliit-keyboard --exit-with-session=/usr/lib/libexec/startplasma-waylandsession
```
and then add this to the end of the file:
```
export KWIN_COMPOSE=QPainter
export GALLIUM_DRIVER=softpipe
export LIBGL_ALWAYS_SOFTWARE=1

startplasma-wayland \
        --framebuffer \
        --xwayland \
        --libinput \
        --inputmethod maliit-server \
        --exit-with-session=/usr/lib/libexec/startplasma-waylandsession

```
This is how your kwinwrapper should look like:
![kwin](https://github.com/NotLugozzi/Xiaomi-daisy-pmOS/blob/main/images/kwinwrapper.png)
## Contributing
If you want to contribute to the development of this project, edit the files and do a pr, if you're doing a lot of modifications to the kernel it's recommended to fork the repo to your account and merge once you've done your changes. In both cases remember to give a short desctiption about the changes you made so that i can understand what you changed



