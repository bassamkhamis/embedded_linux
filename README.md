# embedded_linux
**Hello All, In this Repo I will demonstrate how can build embedded linux from scratch**
## content 
- [Build compiler toolchain using Buildroot](#build-compiler-toolchain-using-Buildroot)
- [Combile bootloader using U-boot](#combile-bootloader-using-U-boot)
- [Compile openSBI firmware](#compile-openSBI-firmware)
- [Configure virtual SD card on linux Ubuntu](#configare-virtual-SD-card-on-linux-Ubuntu)
- [Build and compile root file system using busyBox](#build-and-compile-root-file-system-using-busyBox)
- [Compile kernel](#compile-kernel) 
- [Run linux on qemu](#Run-linux-on-qemu) 
- [References](#References) 

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

**This lines means first one save variables envirument in FAT, second one <virtio> name of SD card, <0:1> first device partition one**
**for additional configruation we can open menu and add some customer configruation by run this** `make menuconfig` **in folder of u-boot
   this will appear 
   ![](https://github.com/bassamkhamis/embedded_linux/blob/main/u-boot.png)
   
   finaly, run this
   
   ```
   export CROSS_COMBILE=riscv64-linux-
   make qemu-riscv64_smode_defconfig
   make -j6
   ```
   **NOTE:**
           1. CROSS_COMBILE=riscv64-linux- is store the prefix of your toolchain like riscv-linux-gcc
           2. -j6 build faster x6, run 5 jobs
   

## Compile openSBI firmware
**Required to start an OS/u-boot from the Firmware(M mode) to Supervisor(S mode) and it work for RISCV Machine, OpenSBI has menuconfig like u-bbot and buildroot you can configuare and combile it**
   
```
git clone https://github.com/riscv/opensbi.git
cd opensbi
git checkout v0.8
make PLATFORM=generic FW_PAYLOAD_PATH=../u-boot-2021.01/u-boot.bin   
```
***
   
## Compile kernel  
 **Download Linux sources from** `https://kernel.org` **Then we must set two environment variables for kernel**
   
```
 export CROSS_COMPILE=riscv64-linux-
 export ARCH=riscv  
```
**We can further customize the configuration, but for now will take default Linux kernel configuration for RISCV, we can find it by grep CLI**
   
   ```
 $ make help | grep defconfig
defconfig - New config with default from ARCH supplied defconfig
savedefconfig - Save current config as ./defconfig (minimal config)
alldefconfig - New config with all symbols set to default
olddefconfig - Same as oldconfig but sets new symbols to their
nommu_k210_defconfig - Build for nommu_k210
nommu_virt_defconfig - Build for nommu_virt
rv32_defconfig - Build for rv32
$ make defconfig
   ```
  **Then combile kerenl by** `make -j 8` **it realy take long time, after combile you can find output in** `arch/riscv/boot/Image`   
   
   
## Build and compile root file system using busyBox
**BusyBox is a software suite that provides several Unix utilities in a single executable**
**Download 1.33 version** `https://busybox.net` **then unzip it and run** ` make allnoconfig` **to reset, the configure is easy run** `make menuconfig` ** and select utilities that you want it to be in filesystem like**
   
1. In Settings →Build Options, enable Build static binary (no shared libs)
2. In Settings →Build Options, set Cross compiler prefix to riscv64-linux-
3. Then enable support for the following commands:
   
`ash, init, halt, mount, cat, mkdir, echo, ls, chmod,vi, ifconfig, rm`
   
 **Then run** 
   ```
   make -j 8
   make install
   _install
   
   ```
   **You must see this**
    ![](https://github.com/bassamkhamis/embedded_linux/blob/main/linux%20tree.png)   
   
   
   
   
   
## Configure virtual SD card on linux Ubuntu 
**Create SD virsual** `dd if=/dev/zero of=disk.img bs=1M count=128` * size =128 M, all bytes are filled by zeros
run this `cfdisk disk.img`  then selsect Dos and create two partins as follows:**
   
   1. A first 64 MB primary partition (type W95 FAT32 (LBA)), marked as bootable
   2. A second partition with remaining space (default type: Linux)
![](https://github.com/bassamkhamis/embedded_linux/blob/main/SD_format.png)
   
   **Now we can access this partitions in this disk image:**
   
   ```
   sudo losetup -f --show --partscan disk.img
    /dev/loop13

   ```
   **In my case loop13, may be different in your case, We can now format the partitions:**
   ```
   sudo mkfs.vfat -F 32 -n boot /dev/loop13p1
   sudo mkfs.ext4 -L rootfs /dev/loop13p2

   ```
   ** Create mount point and copy kernel and file system**
   
   ```
   mkdir /mnt/boot
   sudo mount /dev/loop2p1 /mnt/boot
   sudo cp linux-5.11-rc3/arch/riscv/boot/Image /mnt/boot
   sudo umount /mnt/boot
   ```
   **copy rootfile system**
   ```
   sudo mkdir /mnt/rootfs
   sudo mount /dev/loop2p2 /mnt/rootfs
   sudo rsync -aH _install/ /mnt/rootfs/
   sudo umount /mnt/rootfs
   
   ```
**Now we are ready to run**


## Run linux on qemu
**Using qemu-riscv64 **
   ```
  sudo qemu-system-riscv64 -m 2G -nographic -machine virt -smp 8 \
-bios opensbi/build/platform/generic/firmware/fw_payload.elf \
-drive file=disk.img,format=raw,id=hd0 \
-device virtio-blk-device,drive=hd0 \
-netdev tap,id=tapnet,ifname=tap2,script=no,downscript=no \
-device virtio-net-device,netdev=tapnet 
   ```
   
   ```
mount -t proc nodev /proc
mount -t sysfs nodev /sys
   ```
[viedo](https://drive.google.com/drive/folders/1BspgTYwvK53VdBDBNc1VzwBbZp8ZT3_X?usp=sharing)
   
## References
   1. [ENG.Keroles Shenouda slides](https://l.facebook.com/l.php?u=https%3A%2F%2Fdrive.google.com%2Fdrive%2Fmobile%2Ffolders%2F1rHnM5Q70x2f1bcMJORJjZJ_GvnDiNfo_%3Ffbclid%3DIwAR0wRfBsWWplRAI7yFiA8voYWkk0Y4GIXAQHpSWnl4hIFJer6bC4RXH4_WU&h=AT2d2Fc_XP7UkP5lTFiDTFRGz21onsxE5ZhHOBGVkYraZciENKc0AbZmKb7KCZaL0kAHHESP5_ga-hQbIYcg4u8BxQs_DH6mQ5zQPaJ7tUtgg2lauyIl-SOvJqCJfKzs4icKMQ)
   2. [Bootlin](https://riscv.org/news/2020/12/embedded-linux-from-scratch-in-45-minutes-on-risc-v-bootlin/)
   3. [channel ENG.Moatasem El Sayed](https://www.youtube.com/watch?v=8hP-oQ242B8&list=PLkH1REggdbJpFXAzQqpjZgV1oghPsf9OH&index=17)
