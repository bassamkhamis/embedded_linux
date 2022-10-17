# embedded_linux
**Hello All, In this Repo I will demonstrate how can build embedded linux from scratch**
## content 
1. Build compiler toolchain using Buildroot
2. combile bootloader using U-boot
3. compile openSBI firmware
4. configare virtual SD card on linux Ubuntu 
5. compile kernel 
6. Build and compile root file system using busyBox
7. Run linux on 

## Buildroot
**is open source and Can build a toolchain, a rootfs, a kernel, a bootloader in small time
and easy to configure using menuconfig or xconfig**

**First download** [Buildroot](https://buildroot.org/)
Then Extract it and inside it Run 

'

make menuconfig

'

**may need to install some utilities like bison and flex**

'''

sudo apt-get update
sudo apt-get install flex
sudo apt-get install bison
'''

**window like this will be appeared** 

![](https://github.com/bassamkhamis/embedded_linux/blob/main/Buildroot.png)
