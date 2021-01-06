# Xiaomi-daisy-pmOS
All the code in this repository is distributed under the GNU GPL 2.0 license, read [here](https://github.com/NotLugozzi/Xiaomi-daisy-pmOS/blob/main/LICENSE) for further details
## all of my work has been moved in the official postmarketOS gitlab repository so i won't be updating this one here anymore, check their wiki and their repo for further details
### Port of postmarketOS to the xiaomi mi a2 lite(daisy)
This projet aims to run a fully featured installation of alpine linux on the xiaomi mi a2 lite. It isn't mature enough to be daily driven as it only runs the 
[weston](https://youtu.be/JLhaONV8zBw) demo and [plasma mobile](https://github.com/NotLugozzi/Xiaomi-daisy-pmOS/blob/main/images/71884710-0a75-4978-be57-b59c9a149f9f.png) 

## Current features:
#### works:
- [x] Building
- [x] Installing
- [x] Booting
- [x] Weston interface
- [x] SSH to the device
- [x] Wayland now knows the screen resolution
- [x] Touchscreen support
- [x] Charging and thermal controls(P)(error with charging-sdl, it works fine when booted into plasma. from dmesg the kernel thinks the cpu temp is -22° c)
- [x] Usb networking(P)(doesn't work on windows, i'm going to try on my ubuntu laptop at some point)
- [x] Plasma mobile(P)(scaling issues)
- [x] Wireless connectivity(P)(The firmware blobs now can be loaded and they dont bootloop the phone, nmcli sees the the drivers but i can't connect yet - we're going in the right direction)
#### broken:
- [ ] usb OTG( i dont have really intention to port this right now, i'll work on it after i made the system stable enough to be used as a phone, then i'll work on that and on anbox)
- [ ] camera
- [ ] Mainline pmOS kernel

# Installation
First of all you'll need a linux system(or vm), a Mi A2 Lite with an unlocked bootloader and a couple hours..
#### Install pmbootstrap
You'll need this to compile the kernel and the necessary files
```python
pip3 install --user pmbootstrap
```
after you've installed it, run
```python
pmbootstrap init
```
After it says it finished downlading the repo copy the linux-xiaomi-daisy-2, device-xiaomi-daisy-2 and firmware-xiaomi-daisy-2 folders in your pmbootstrap environment (/home/user/.local/var/pmbootstrap/cache_git/pmaports/device/community/)
At this point you can select weston or [plasma mobile](https://github.com/NotLugozzi/Xiaomi-daisy-pmOS#plasma) as the interface. You can also allow the proprietary wifi drivers
#### Building the kernel
After setting pmbootstrap up run this 3 commands, the first will validate checksums for all the necessary patches and for the kernel itself, the second one will verify the device specific package and the last one will check the firmware specific files. If they don't give any errors we can move to actually building The Kernel
```python
pmbootstrap checksum linux-xiaomi-daisy-2
pmbootstrap checksum device-xiaomi-daisy-2
pmbootstrap checksum firmware-xiaomi-daisy-2
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
Just follow the installation process and keep an eye on the log window. At this point you have 2 options: either flashing both the rootfs and the kernel from pmbootstrap(reccomended - you'll find instructions after the install command, just remember to append **--partition userdata** to the rootfs flash command) or moving the rootfs to your host, flashing that with fastboot, with this command:
```
fastboot flash userdata path/to/xiaomi-daisy-2.img
```
And then connecting your phone to the vm and flashing the kernel on that. Keep in mind that installing it to userdata is necessary and it will wipe all your data. The first boot can take up to a minute because it has to initiallize some stuff. you'll get the pmos boot screen and the weston demo desktop:
![boot](https://github.com/NotLugozzi/Xiaomi-daisy-pmOS/blob/main/images/pmOS%20succesful%20boot.jpg)
Just to be sure everything works, run dmesg on the linux host and check if you get this output:
![dmesg](https://github.com/NotLugozzi/Xiaomi-daisy-pmOS/blob/main/images/dmesg.png).
                          If your screen is black after installing plasma mobile, do not worry - it's a known issue with the framebuffer mode in the de, read (here)[https://github.com/NotLugozzi/Xiaomi-daisy-pmOS#plasma] for the temporary fix - you'll have to do this only on the first boot - or after reflashing the rootfs.
## Experimental Features. 
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
## Plasma
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
and then restart lightdm
```
sudo rc-service lightdm restart
```
This is how your kwinwrapper should look like:
![kwin](https://github.com/NotLugozzi/Xiaomi-daisy-pmOS/blob/main/images/kwinwrapper.png)
## Contributing
If you want to contribute to the development of this project, edit the files and do a pr, if you're doing a lot of modifications to the kernel it's recommended to fork the repo to your account and merge once you've done your changes. In both cases remember to give a short desctiption about the changes you made so that i can merge without having to go thru all your changes  


*As my friend said after i showed them this:: that's what 0 pussy does to a mf*
