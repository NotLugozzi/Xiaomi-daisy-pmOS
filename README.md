# Xiaomi-daisy-pmOS

### working(ish) port of postmarketOS to the xiaomi mi a2 lite(daisy)
 This port, although heavily taken from [this](https://github.com/matthew-5pl/pmos-xiaomi-daisy), has some modifications and updates to get you up and running with a somewhat funcional pmOS install

## List of what works and what doesnt:
#### works:
- [x] building
- [x] installing
- [x] booting
- [x] Wayland compositor and the Weston interface(kinda broken - i'll explain later)
- [x] SSH in the device
#### broken/partial support:
- [ ] Touchscreen support
- [ ] Usb networking
- [ ] Wireless connectivity - this includes mobile data, wifi and bluetooth
- [ ] Charging and thermal controls
- [ ] usb OTG
- [ ] mainline pmos kernel

Now this might seem like a long list of things that dont work _yet_ but i realized that [xiaomi-vince](https://wiki.postmarketos.org/wiki/Xiaomi_Redmi_Note_5_Plus_(xiaomi-vince)) has the same exact hardware and a lot of the things i'm working on are already a thing on that device, so i'll take _inspiration_ from that pm port

## How to compile and install 
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
and set it up like this
![pmbs_init](https://github.com/NotLugozzi/xiaomi-daisy-pmos/blob/main/images/pmbootstrap%20init.png)
#### Build the kernel
After setting it up run this command, it'll validate checksums for all the necessary patch files and for the kernel itself. If this doesnt give an error we can move to actually building linux and the device specific packages
```python
pmbootstrap checksum linux-xiaomi-daisy-2
```
We're going to compile 2 things, the kernel itself and some device specific packages; at this point you should open another terminal window running **pmbootstrap log** to check the compile job in real time. The first command we'll run is this
```python
pmbootstrap build linux-xiaomi-daisy-2
```
go grab a coffee and some snacks as this can take up to 3 hours on older hardware - hopefully you'll only have to compile once, without any errors or stuff like that. After you've compiled the kernel, run this command for the specific packages
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
and then connecting your phone to the vm and flashing the kernel on that. Personally i use the second method and it works great, no matter what you choose, the moment you turn the phone on you'll have a _drum roll........_ black screen :c - i'm currently working on a fix for that and i'll keep you updated but for now you'll have to ssh into the phone and run this 2 commands to get your screen running:
```python
sudo su
cat /sys/class/graphics/fb0/modes > /sys/class/graphics/fb0/mode
```
if you dont want to do that but you still want to check if the phone is working properly you should run dmesg and check if you get this output:
![dmesg](https://github.com/NotLugozzi/Xiaomi-daisy-pmOS/blob/main/images/dmesg.png)
##### Experimental Features
As of now this port includes a working driver for the gpu and openGL. to test it out you'll need to edit a file using nano/vi **(sudo nano /etc/profile.d/xdg_runtime_dir.sh)**. you'll need to add this to make wayland work properly:
```
if test -z "${XDG_RUNTIME_DIR}"; then
  export XDG_RUNTIME_DIR=/tmp/$(id -u)-runtime-dir
  if ! test -d "${XDG_RUNTIME_DIR}"; then
    mkdir "${XDG_RUNTIME_DIR}"
    chmod 0700 "${XDG_RUNTIME_DIR}"
  fi
fi
```
This problem should be solved as soon as i'm able to merge the xiaomi-vince port on my current work. i'll update the repo as soon as something new happens
