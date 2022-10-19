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


