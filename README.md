# embedded_linux
**Hello All, In this Repo I will demonstrate how can build embedded linux from scratch**
## content 
- [Build compiler toolchain using Buildroot](#build-compiler-toolchain-using-Buildroot)
- [Combile bootloader using U-boot](#combile-bootloader-using-U-boot)
- [Compile openSBI firmware](#compile-openSBI-firmware)
- [Configare virtual SD card on linux Ubuntu](#configare-virtual-SD-card-on-linux-Ubuntu)
- [Compile kernel](#compile-kernel) 
- [Build and compile root file system using busyBox](#build-and-compile-root-file-system-using-busyBox)
- [Run linux on qemu](#Run-linux-on-qemu) 

## Build compiler toolchain using Buildroot
**is open source and Can build a toolchain, a rootfs, a kernel, a bootloader in small time
and easy to configure using menuconfig or xconfig**

**First download** [Buildroot](https://buildroot.org/)
Then Extract it and inside it Run 

```
make menuconfig
```
**may need to install some utilities like bison and flex**

```
sudo apt-get update
sudo apt-get install flex
sudo apt-get install bison
```

**window like this will be appeared** 

![](https://github.com/bassamkhamis/embedded_linux/blob/main/Buildroot.png)
**try to configuare the follows:**
1. Target options select
   1. Target Architecture > RISCV
   2. Target Architecture Size > 64-bit
2. Toolchain
   1. Kernel Headers > Linux 5.17.x kernel headers
   2. GCC compiler Version > gcc 11.x

**Finaly Run** `make -j6`
**it will take long time to combile toolchain, we can find toochain inside output/images/riscv64-buildroot-linux-musl_sdk-buildroot.tar.gz, copy zip file inside sutible folder and unzibe it then run this script** `./relocate-sdk.sh`
**Done we have our new toolchain
NOTE: Don't forget add toolchain in path by** `export PATH=~/path of file/:$PATH`

## Combile bootloader using U-boot
 **Dwonload u-boot** 
 
 ```
 git clone https://source.denx.de/u-boot/u-boot.git
 cd u-boot
git checkout v2022.07
```
**U-Boot has many configruation files according to specific hardware you can find this file in configs directory we can use grep command line to find it**

```
ls configs/ | grep riscv
```

**we will use this ** `qemu-riscv64_smode_defconfig` **open add the follows**

```
 CONFIG_ENV_IS_IN_FAT=y
 CONFIG_ENV_FAT_INTERFACE="virtio"
 CONFIG_ENV_FAT_DEVICE_AND_PART="0:1"
```

**This lines means fist one save varibles envirument in FAT, second one <virtio> name of SD card, <0:1> first device partin one**
**for adtinal configruation we can open menual and add some customer configruation by run this** `make menuconfig` **in folder of u-boot
   this will appear 
   ![]()
   
   finaly, run this
   
   ```
   export CROSS_COMBILE=riscv64-linux-
   make qemu-riscv64_smode_defconfig
   make -j6
   ```
   **NOTE:
           1. CROSS_COMBILE=riscv64-linux- is store the prefix of your toolchain like riscv-linux-gcc
           2. -j6 build faster x6, run 5 jobs
   **
   


## Compile openSBI firmware
**TODO**


## Configare virtual SD card on linux Ubuntu 
**TODO**


## Compile kernel 

**TODO**

## Build and compile root file system using busyBox
**TODO**


## Run linux on qemu
**TODO**
