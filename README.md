# Ubuntu on macbook 12 (mid 2017)

> ## Here is a "noob tutorial" to install Ubuntu 17.04 on Macbook 12 version mid 2017 [MacBook10,1](https://en.wikipedia.org/wiki/MacBook_(Retina)#Technical_Specifications)

:bulb: First warning!

I tried this installation with a **windows bootcamp partition** already installed alongside
**BUT I COULD NOT BOOT ON IT ANYMORE AFTER** (though the datas of the partition were not overwritten).
I suspect more a boot reFind related error,
reFind displayed me that error: `Blinitializelibrary failed 0xc00000bb`

:bulb: Second warning!

Contrarly to what it is said [here](https://gist.github.com/roadrunner2/1289542a748d9a104e7baec6a92f9cd7) or [here](https://github.com/Dunedan/mbp-2016-linux) about WIFI stability on previous macbook models mine seem to behave natively pretty well. Letting `ping google.com` in a console show most of the time 30ms (on a poor connection) with no disconnection.

### Before starting

You need a usb hub with at least 3 ports available for the keyboard + the mouse + ubuntu live usb

### Install Ubuntu

In all cases grab the [ubuntu iso](http://releases.ubuntu.com) and install [unetbootin](https://unetbootin.github.io) (which i generally execute on my macOs partition)

As of 29/06/2017 ubuntu 17.04 ships **with 4.10 kernel version**

This is an important point because with kernel under 4.11 internal **SSD WON'T BE RECOGNIZED** and therefore **YOU WON'T BE ABLE TO INSTALL UBUNTU** on the ssd

:bulb: From there you have 3 options (from most simple to most tricky)

- 1) Wait for future (or if it is already released when you read this tutorial) Ubuntu artful aardvark to be released that will implement 4.11 kernel version then make a bootable usb like you would do with unetbootin

- 2) If you can already run a ~30 go Ubuntu system somewhere

Make an image of Ubuntu with kernel upgraded.

To do so download [isorespin.sh](http://www.cnx-software.com/2017/03/29/isorespin-sh-script-updates-ubuntu-iso-files-with-mainline-kernel)

(install propable dependencies)

```bash
sudo apt install xorriso
chmod +x isorespin.sh
./isorespin.sh -i [your iso] -k v4.11.7
```

Make the generated iso bootable with unetbootin.
Plug the usb, boot your mac holding "alt" stroke and choose EFI boot at the right.
Choose "Install ubuntu".

(I have also tried this method and it is i think the smartest.)

- 3) Classic install

Create the bootable ubuntu usb with unetbootin.
Plug the usb, boot your mac holding "alt" stroke and choose EFI boot at the right.
Choose "Try ubuntu".

Once the in the live session launch a terminal and execute the following commands

```bash
sudo su
modprobe nvme
echo 106b 2003 > /sys/bus/pci/drivers/nvme/new_id
```

:bulb: You should see the internal drives mounting.

Now you can initiate the install process from within the live session.
(If for some reason the process install is stuck execute again the above commands)

Once ubuntu installed you will have to **REBOOT IN RECOVERY MODE** (default option will give you unreadable screen, in that case just type exit, press enter and retry).

At next boot the drive won't be recognized again so you will have to enter the same command above (without sudo) in the `>initramfs` console.

Then hit CTRL+d to exit and it should manage to boot and mount the drive. Recall this step because you may have to do it again. (It is explained [here](https://www.debian-fr.org/t/probleme-dualboot-sur-macos/73496/15) for french people)

Once ubuntu rebooted with discs recognized you may **update the kernel to 4.11.7 version** in order to solve once for all the problem of internal ssd.
To do so launch a terminal and execute:

```bash
cd /tmp
wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.11.7/linux-headers-4.11.7-041107_4.11.7-041107.201706240231_all.deb
wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.11.7/linux-headers-4.11.7-041107-generic_4.11.7-041107.201706240231_amd64.deb
wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.11.7/linux-image-4.11.7-041107-generic_4.11.7-041107.201706240231_amd64.deb
sudo su 
dpkg -i *.deb
apt update
apt upgrade -y
reboot
```

### Solve the keyboard and touchpad recognition problem

- 1) Retrieve the drivers on Github

Launch a terminal and execute the following command

```bash
cd
sudo apt install git
git clone https://github.com/roadrunner2/macbook12-spi-driver
pushd macbook12-spi-driver
git checkout touchbar-driver-hid-driver
make
sudo mkdir /lib/modules/`uname -r`/custom/
sudo cp applespi.ko appletb.ko /lib/modules/`uname -r`/custom/
sudo depmod
popd
```

- 2) Set the proper dpi for the touchpad

`sudo nano /etc/udev/hwdb.d/61-evdev-local.hwdb`

and copy paste the content of the 61-evdev-local.hwdb file of this repository.
CTRL+o (to write)
CTRL+x (to exit)

- 3) Load the modules

`sudo modprobe intel_lpss_pci spi_pxa2xx_platform applespi appletb`

- 4) You'll want these loaded on boot, so rebuild the initramfs:

```bash
sudo su
echo 'add_drivers+="applespi intel_lpss_pci spi_pxa2xx_platform appletb"' >> /etc/initramfs-tools/modules
cd /boot
mv initrd.`img-uname -r`{,.orig}
mkinitramfs -o initrd.img-$(uname -r)
reboot
```

And voilà! Your keyboard and touchpad should be working at next login!

### Booting Ubuntu

:bulb: I noticed a strange demeanor (using the following install of ubuntu) depenging surely on how one reboots.

Indeed after several reboots in "ubuntu default mode" I start to lose touchpad and then keyboard capabilities
but it seems i can get them back (without recompiling anything) if I boot in recovery mode and without executing any commands (I don't know if it can be related?)

Also if one reboots letting default ubuntu selection one can change screen resolution (actually not the resolution itself but the range cursor below) and screen brightness
but in recovery mode it is not the case.

>Conclusion is to always boot in "recovery mode" in order to get keyboard and touchpad recognized and work with a "static" screen resolution and brightness.
